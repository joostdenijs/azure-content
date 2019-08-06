Follow these steps to install and run MongoDB on a virtual machine running Windows Server.

> [AZURE.IMPORTANT] MongoDB security features, such as authentication and IP address binding, are not enabled by default. Security features should be enabled before deploying MongoDB to a production environment.  See [Security and Authentication](http://www.mongodb.org/display/DOCS/Security+and+Authentication) for more information.

1. After you've connected to the virtual machine using Remote Desktop, open Internet Explorer from the **Start** menu.
2. Select the **Tools** button in the upper right corner.  In **Internet Options**, select the **Security** tab, and then select the **Trusted Sites** icon, and finally click the **Sites** button. Add *http://\*.mongodb.org* to the list of trusted sites.
3. Go to [Downloads- MongoDB] [MongoDownloads].
4. Find the most recent release in the **Production Release (Recommended)** section and click the ***2008+** link in the Windows 64-bit column.  Click **Save As** and save the zip file to the desktop.
5. Right-click on the zip file and select **Extract All...**  Specify "C:\" and click **Extract**.  After the files have been extracted, you may wish to rename the install folder to something simpler.  "MongoDB", for example.
6. Create MongoDB data and log directories in the data disk (drive **F:**, for example) you created in the steps above. From **Start**, select **Command Prompt** to open a command prompt window.  Type:

		C:\> F:
		F:\> mkdir \MongoData
		F:\> mkdir \MongoLogs

7. To run the database, run: 

		F:\> C:
		C:\> cd \MongoDB\bin
		C:\my_mongo_dir\bin> mongod --dbpath F:\MongoData\ --logpath F:\MongoLogs\mongolog.log

	All log messages will be directed to the *F:\MongoLogs\mongolog.log* file as mongod.exe server starts and preallocates journal files. It may take several minutes for MongoDB to preallocate the journal files and start listening for connections.

8. To start the MongoDB administrative shell, open another command window from **Start** and type the following:

		C:\> cd \my_mongo_dir\bin  
		C:\my_mongo_dir\bin> mongo  
		>db  
		test  	  
		> db.foo.insert( { a : 1 } )  
		> db.foo.find()  
		{ _id : ..., a : 1 }  
		> show dbs  
		...  
		> show collections  
		...  
		> help  

	The database is created by the insert.
9. (Optional) mongod.exe has support for installing and running as a Windows service. To install mongod.exe as a service, run the following from the command prompt:

		C:\mongodb\bin>mongod --logpath "c:\mongodb\logs\logfile.log" --logappend --dbpath "c:\data" --install 

	This creates a service named "Mongo DB" with a description of "Mongo DB". The **--logpath** option must be used to specify a log file, since the running service will not have a command window to display output.  The **--logpath** option specifies that a restart of the service will cause output to append to the existing log file.  The **--dbpath** option specifies the location of the data directory. For more service-related command line options, see [Service-related command line options] [MongoWindowsSvcOptions].
10. Now that MongoDB is installed and running, you'll need to open a port in Windows Firewall so you can remotely connect to MongoDB.  From the **Start** menu, select **Administrator Tools** and then **Windows Firewall with Advanced Security**. 

11. In the left pane, select **Inbound Rules**.  In the **Actions** pane on the right, select **New Rule...**.
	
	![Windows Firewall][Image1]

	In the **New Inbound Rule Wizard**, select **Port** and then click **Next**.
	
	![Windows Firewall][Image2]

	Select **TCP** and then **Specific local ports**.  Specify a port of "27017" (the port MongoDB listens on) and click **Next**.

	![Windows Firewall][Image3]

	Select **Allow the connection** and click **Next**.

	![Windows Firewall][Image4]

	Click **Next** again.
	
	![Windows Firewall][Image5]

	Specify a name for the rule, such as "MongoPort", and click **Finish**.

	![Windows Firewall][Image6]
	
12. If you didn't configure an endpoint for MongoDB when you created the virtual machine, you can do it now. You need both the firewall rule and the endpoint to be able to connect to MongoDB remotely. In the Management Portal, click **Virtual Machines**, click the name of your new virtual machine, and then click **Endpoints**.

	![Endpoints][Image7]
13. Click **Add Endpoint** at the bottom of the page. Select **Add Endpoint** and click **Next**.
	
	![Endpoints][Image8]

14. Add an endpoint with name "Mongo", protocol **TCP**, and both **Public** and **Private** ports set to "27017". This will allow MongoDB to be accessed remotely.

	![Endpoints][Image9]

[MongoDownloads]: http://www.mongodb.org/downloads

[MongoWindowsSvcOptions]: http://www.mongodb.org/display/DOCS/Windows+Service


[Image1]: ./media/install-and-run-mongo-on-win2k8-vm/WinFirewall1.png
[Image2]: ./media/install-and-run-mongo-on-win2k8-vm/WinFirewall2.png
[Image3]: ./media/install-and-run-mongo-on-win2k8-vm/WinFirewall3.png
[Image4]: ./media/install-and-run-mongo-on-win2k8-vm/WinFirewall4.png
[Image5]: ./media/install-and-run-mongo-on-win2k8-vm/WinFirewall5.png
[Image6]: ./media/install-and-run-mongo-on-win2k8-vm/WinFirewall6.png
[Image7]: ./media/install-and-run-mongo-on-win2k8-vm/WinVmAddEndpoint.png
[Image8]: ./media/install-and-run-mongo-on-win2k8-vm/WinVmAddEndpoint2.png
[Image9]: ./media/install-and-run-mongo-on-win2k8-vm/WinVmAddEndpoint3.png
