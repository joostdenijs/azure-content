<properties 
	pageTitle="Move data to SQL Server on Azure| Azure" 
	description="Move data to SQL Server on Azure" 
	metaKeywords="" 
	services="machine-learning" 
	solutions="" 
	documentationCenter="" 
	authors="msolhab" 
	manager="paulettm" 
	editor="cgronlun" />

<tags 
	ms.service="machine-learning" 
	ms.workload="data-services" 
	ms.tgt_pltfrm="na" 
	ms.devlang="na" 
	ms.topic="article" 
	ms.date="05/12/2015" 
	ms.author="fashah;garye;mohabib;bradsev" /> 

#Move data to SQL Server on Azure

This document covers the movement of data from flat files (csv/tsv) or an on-premise SQL Server to a SQL Server on Azure


<table>

<tr>
<td><b>SOURCE</b></td>
<td colspan="2" align="center"><b>DESTINATION</b></td>
</tr>

<tr>
  <td></td>
  <td><b>SQL Server VM on Azure</b></td>
</tr>

<tr>
  <td><b>Flat File</b></td>  
  <td>
    1. <a href="#insert-tables-bcp">Command line bulk copy utility (BCP) </a><br>
    2. <a href="#insert-tables-bulkquery">Bulk Insert SQL Query </a><br>
    3. <a href="#sql-builtin-utilities">Graphical Built-in Utilities in SQL Server </a>
  </td>
</tr>
<tr>
  <td><b>On-Premise SQL Server</b></td>
  <td>
    1. <a href="#export-flat-file">Export to a flat File </a><br>
    2. <a href="#sql-migration">SQL Database Migration Wizard </a> <br>    
    3. <a href="#sql-backup">Database backup and restore </a> <br>
  </td>
</tr>
</table>

Note that this document assumes that SQL commands are executed from SQL Server Management Studio or Visual Studio Database Explorer.



## <a name="sqlonazurevm"></a>Moving your data to a SQL Server VM on an Azure

This section documents the process of moving data to a SQL Server VM on Azure. If you haven't set up the SQL Server VM, provision a new SQL Server virtual machine for advanced analytics as described in [Set up an Azure SQL Server virtual machine as an IPython Notebook server for advanced analytics](machine-learning-data-science-setup-sql-server-virtual-machine.md). 

This document describes moving data from the following data sources: 
  
1. [From flat files](#filesource_to_sqlonazurevm) 
2. [From an On-Premise SQL Server](#sqlonprem_to_sqlonazurevm)


### <a name="filesource_to_sqlonazurevm"></a>File source

If your data is in a flat file (arranged in a row/column format), it can be moved to SQL Server VM on Azure via the following methods:

1. [Command line bulk copy utility (BCP)](#insert-tables-bcp) 
2. [Bulk Insert SQL Query ](#insert-tables-bulkquery)
3. [Graphical Built-in Utilities in SQL Server (Import/Export, SSIS)](#sql-builtin-utilities)


### <a name="insert-tables-bcp"></a>Command line bulk copy utility (BCP)

BCP is a command line utility installed with SQL Server and is one of the quickest ways to move data. It works across all three SQL Server variants (On-premise SQL Server, SQL Azure and SQL Server VM on Azure). 

> [AZURE.NOTE] **Where should my data be for BCP?**  
> While it is not required, having files containing source data located on the same machine as the target SQL server allows for faster transfers (network speed vs local disk IO speed). You can move the flat files containing data to the machine where SQL Server is installed using various file copying tools such as [AZCopy](storage-use-azcopy.md), [Azure Storage Explorer](https://azurestorageexplorer.codeplex.com/) or windows copy/paste via Remote Desktop Protocol (RDP).

1. Ensure that the database and the tables are created on the target SQL Server database. Here is an example of how to do that using the `Create Database` and `Create Table` commands:

		CREATE DATABASE <database_name>
	
		CREATE TABLE <tablename>
		(
			<columnname1> <datatype> <constraint>,
			<columnname2> <datatype> <constraint>,
			<columnname3> <datatype> <constraint>
		) 
	
2. Generate the format file that describes the schema for the table by issuing the following command from the command line of the machine where bcp is installed.

	`bcp dbname..tablename format nul -c -x -f exportformatfilename.xml -S servername\sqlinstance -T -t \t -r \n` 

3. Insert the data into the database using the bcp command as follows. This should work from the command line assuming that the SQL Server is installed on same machine:

	`bcp dbname..tablename in datafilename.tsv -f exportformatfilename.xml -S servername\sqlinstancename -U username -P password -b block_size_to_move_in_single_attemp -t \t -r \n`

> **Optimizing BCP Inserts** Please refer the following article ['Guidelines for Optimizing Bulk Import'](https://technet.microsoft.com/library/ms177445%28v=sql.105%29.aspx) to optimize such inserts.

#### <a name="insert-tables-bulkquery-parallel"></a>Parallelizing Inserts for Faster Data Movement

If the data you are moving is large, you can speed things up by simultaneously executing multiple BCP commands in parallel in a PowerShell Script.

> [AZURE.NOTE] **Big data Ingestion** 
> To optimize data loading for large and very large datasets, partition your logical and physical database tables using multiple filegroups and partition tables. For more information about creating and loading data to partition tables, see  [Parallel Load SQL Partition Tables](machine-learning-data-science-parallel-load-sql-partitioned-tables.md).


The sample PowerShell script below demonstrate parallel inserts using bcp :
	
	$NO_OF_PARALLEL_JOBS=2

	 Set-ExecutionPolicy RemoteSigned #set execution policy for the script to execute
	 # Define what each job does
	   $ScriptBlock = {
		   param($partitionnumber)

		   #Explictly using SQL username password
		   bcp database..tablename in datafile_path.csv -F 2 -f format_file_path.xml -U username@servername -S tcp:servername -P password -b block_size_to_move_in_single_attempt -t "," -r \n -o path_to_outputfile.$partitionnumber.txt

			#Trusted connection w.o username password (if you are using windows auth and are signed in with that credentials)
			#bcp database..tablename in datafile_path.csv -o path_to_outputfile.$partitionnumber.txt -h "TABLOCK" -F 2 -f format_file_path.xml  -T -b block_size_to_move_in_single_attempt -t "," -r \n 
	  }
	

	# Background processing of all partitions
	for ($i=1; $i -le $NO_OF_PARALLEL_JOBS; $i++)
	{
	  Write-Debug "Submit loading partition # $i"
	  Start-Job $ScriptBlock -Arg $i	  
	}
	

	# Wait for it all to complete
	While (Get-Job -State "Running")
	{
	  Start-Sleep 10
	  Get-Job
	}
	
	# Getting the information back from the jobs
	Get-Job | Receive-Job
	Set-ExecutionPolicy Restricted #reset the execution policy


### <a name="insert-tables-bulkquery"></a>Bulk Insert SQL Query

[Bulk Insert SQL Query](https://msdn.microsoft.com/library/ms188365) can be used to import data into the database from row/column based files (the supported types are covered [here](https://msdn.microsoft.com/library/ms188609)). 

Here are some sample commands for Bulk Insert are as below:  

1. Analyze your data and set any custom options before importing to make sure that the SQL Server database assumes the same format for any special fields such as dates. Here is an example of how to set the date format as year-month-day (if your data contains the date in year-month-day format):

		SET DATEFORMAT ymd;	
	
2. Import data using bulk import statements:

    	BULK INSERT <tablename>
    	FROM	
    	'<datafilename>'
    	WITH 
    	(
		FirstRow=2,
    	FIELDTERMINATOR =',', --this should be column separator in your data
    	ROWTERMINATOR ='\n'   --this should be the row separator in your data
    	)
 	  

### <a name="sql-builtin-utilities"></a>Built-in Utilities in SQL Server

You can use SQL Server Integrations Services (SSIS) to import data into SQL Server VM on Azure from a flat file. 
SSIS is available in two studio environments. For details, see [Integration Services (SSIS) and Studio Environments](https://technet.microsoft.com/library/ms140028.aspx):

- For details on SQL Server Data Tools, see [Microsoft SQL Server Data Tools](https://msdn.microsoft.com/data/tools.aspx)  
- For details on the Import/Export Wizard, see [SQL Server Import and Export Wizard](https://msdn.microsoft.com/library/ms141209.aspx)

### <a name="sqlonprem_to_sqlonazurevm"></a>Moving Data from On-Premise SQL Server

Data can be moved from an on premise SQL Server as follows:

1. [Export to Flat File](#export-flat-file) 
2. [SQL Database Migration Wizard](#sql-migration)
3. [Database backup and restore](#sql-backup)

We describe each of these below:

#### <a name="export-flat-file"></a>Export to Flat File

Various methods can be used to bulk export data from an On-Premise SQL Server as documented [here](https://msdn.microsoft.com/library/ms175937.aspx). This document will cover the Bulk Copy Program (BCP) as an example. Once data is exported into a flat file, it can be imported to another SQL server using bulk import. 

1. Export the data from on-premise SQL Server to a File using the bcp utility as follows

	`bcp dbname..tablename out datafile.tsv -S	servername\sqlinstancename -T -t \t -t \n -c`

2. Create the database and the table on SQL Server VM on Azure using the `create database` and `create table` for the table schema exported in step 1.

3. Create a format file for describing the table schema of the data being exported/imported. Details of the format file are described [here](https://msdn.microsoft.com/library/ms191516.aspx).

	Format file generation when running BCP from the SQL Server machine 

		bcp dbname..tablename format nul -c -x -f exportformatfilename.xml -S servername\sqlinstance -T -t \t -r \n 

	Format file generation when running BCP remotely against a SQL Server 

		bcp dbname..tablename format nul -c -x -f  exportformatfilename.xml  -U username@servername.database.windows.net -S tcp:servername -P password  --t \t -r \n
		
	
4. Use any of the methods described in section [Moving Data from File Source](#filesource_to_sqlonazurevm) to move the data in flat files to a SQL Server.

#### <a name="sql-migration"></a>SQL Database Migration Wizard

[SQL Server Database Migration Wizard](http://sqlazuremw.codeplex.com/) provides a  user-friendly way to move data between two SQL server instances. It allows the user to map the data schema between sources and destination tables, choose column types and various other functionality. It uses bulk copy (BCP) under the covers. A screenshot of the welcome screen for the SQL Database Migration wizard is shown below.  

![SQL Server Migration Wizard][2]

#### <a name="sql-backup"></a>Database backup and restore

SQL Server supports: 

1. [Database backup and restore functionality](https://msdn.microsoft.com/library/ms187048.aspx) (both to a local file or bacpac export to blob) and [Data Tier Applications](https://msdn.microsoft.com/library/ee210546.aspx) (using bacpac). 
2. Ability to directly create SQL Server VMs on Azure with a copied database or copy to an existing SQL Azure database. For more details, see [Use the Copy Database Wizard](https://msdn.microsoft.com/library/ms188664.aspx). 

A screenshot of the Database backup/restore options from SQL Server Management Studio are shown below.

![SQL Server Import Tool][1]


[1]: ./media/machine-learning-data-science-move-sql-server-virtual-machine/sqlserver_builtin_utilities.png
[2]: ./media/machine-learning-data-science-move-sql-server-virtual-machine/database_migration_wizard.png

