<properties 
	pageTitle="Use Azure Premium Storage with SQL Server on Virtual Machines" 
	description="This article provides guidance for how to start using Azure Premium Storage with SQL Server running on Azure Virtual Machines. This includes examples of new deployments and migrations of existing deployments of SQL Server on IaaS." 
	services="virtual-machines" 
	documentationCenter="" 
	authors="rothja" 
	manager="jeffreyg"
	editor=""/>

<tags
	ms.service="virtual-machines"
	ms.devlang="na"
	ms.topic="article"
	ms.tgt_pltfrm="vm-windows-sql-server"
	ms.workload="infrastructure-services" 
	ms.date="04/29/2015"
	ms.author="jroth"/>

# Use Azure Premium Storage with SQL Server on Virtual Machines

## Overview

[Azure Premium Storage](storage-premium-storage-preview-portal.md) is the next generation of storage that provides low latency and high throughput IO. It works best for key IO intensive workloads, such as SQL Server on IaaS [Virtual Machines](http://azure.microsoft.com/services/virtual-machines/). This article provides planning and guidance for migrating a Virtual Machine running SQL Server to use Premium Storage. This includes Azure infrastructure (networking, storage) and guest Windows VM steps. The example in the [Appendix](#appendix-migrating-a-multisite-alwayson-cluster-to-premium-storage) shows a full comphrensive end to end migration of how to move larger VMs to take advantage of improved local SSD storage with PowerShell..

It is important to understand the end-to-end process of utilizing Azure Premium Storage with SQL Server on IAAS VMs. This includes:

- Identification of the prerequisites to use Premium Storage.
- Examples of deploying SQL Server on IaaS to Premium Storage for new deployments.
- Examples of migrating existing deployments, both stand-alone servers and deployments using SQL AlwaysOn Availability Groups.
- Possible migration approaches.
- Full end-to-end example showing Azure, Windows, and SQL Server steps for  the migration of an existing AlwaysOn implementation.

For more background information on SQL Server in Azure Virtual Machines, see [SQL Server in Azure Virtual Machines](virtual-machines-sql-server-infrastructure-services.md).

## Prerequisites for Premium Storage

There are several prerequisites for using Premium Storage.

### Machine Size

For using Premium Storage you will need to use DS series Virtual Machines (VM). If you have not used DS Series machines in your cloud service before, you must delete the existing VM, keep the attached disks, and then create a new cloud service before recreating  the VM as DS* role size. For more information on Virtual Machine sizes, see [Virtual Machine and Cloud Service Sizes for Azure](https://msdn.microsoft.com/library/azure/dn197896.aspx).

### Cloud Services

You can only use DS* VMs with Premium Storage when they are created in a new cloud service. If you are using SQL Server AlwaysOn in Azure, the AlwaysOn Listener will refer to the Azure Internal or External Load Balancer IP address that is associated with a cloud service. This article focuses on how to migrate while maintaining availability in this scenario.

> [AZURE.NOTE] A DS* Series must be the first VM that is deployed to the new Cloud Service.

### Regional VNETS

For  DS* VMs you must configure the Virtual Network (VNET) hosting your VMs to be regional. This “widens” the VNET is to allow the larger VMs to be provisioned in other clusters and allow communication between them. In the following screenshot, the highlighted Location shows regional VNETs, whereas the first result shows a “narrow” VNET.
 
![RegionalVNET][1]

You can raise a Microsoft support ticket to migrate to a regional VNET, Microsoft will make a change, then to complete the migration to regional VNETs, change the property AffinityGroup in the network configuration. First export the Network Configuration in PowerShell, and then replace the **AffinityGroup** property in the **VirtualNetworkSite** element with a **Location** property. Specify `Location = XXXX` where `XXXX` is an Azure region. Then import the new configuration.

For example, considering the following VNET configuration:

    <VirtualNetworkSite name="danAzureSQLnet" AffinityGroup="AzureSQLNetwork">
    <AddressSpace>
      <AddressPrefix>10.0.0.0/8</AddressPrefix>
      <AddressPrefix>172.16.0.0/12</AddressPrefix>
    </AddressSpace>
    <Subnets>
    ...
    </VirtualNetworkSite>

To move this to a regional VNET in West Europe, change the configuration to the following:

    <VirtualNetworkSite name="danAzureSQLnet" Location="West Europe">
    <AddressSpace>
      <AddressPrefix>10.0.0.0/8</AddressPrefix>
      <AddressPrefix>172.16.0.0/12</AddressPrefix>
    </AddressSpace>
    <Subnets>
    ...
    </VirtualNetworkSite>
    
### Storage Accounts

You will need to create a new storage account that is configured for Premium Storage. Note that the use of Premium Storage is set at the storage account, not on individual VHDs, however when using a DS* Series VM you can attach VHD’s from Premium and Standard Storage accounts. You may consider this if you do not want to place the OS VHD on to the Premium Storage account.

The following **New-AzureStorageAccountPowerShell** command with the "Premium_LRS" **Type** creates a Premium Storage Account:

    $newstorageaccountname = "danpremstor" 
    New-AzureStorageAccount -StorageAccountName $newstorageaccountname -Location "West Europe" -Type "Premium_LRS"   

### VHDs Cache Settings

The main difference between creating disks that are part of a Premium Storage account is the disk cache setting. For SQL Server Data volume disks it is recommended that you use ‘**Read Caching**’. For Transaction log volumes, the disk cache setting should be set to ‘**None**’. This is different from the recommendations for Standard Storage accounts.

Once the VHDs have been attached, the cache setting cannot be altered. You would need to detach and reattach the VHD with an updated cahce setting.

### Windows Storage Spaces

You can use [Windows Storage Spaces](https://technet.microsoft.com/library/hh831739.aspx) as you did with previous Standard Storage, this will allow you to migrate a VM that is already utilizing Storage Spaces. The example in [Appendix](#appendix-migrating-a-multisite-alwayson-cluster-to-premium-storage) (step 9 and forward) demonstrates the Powershell code to extract and import a VM with multiple attached VHDs. 

Storage Pools were used with Standard Azure storage account to enhance throughput and reduce latency. You might find value in testing Storage Pools with Premium Storage for new deployments, but they do add additional complexity with storage setup. 

#### How to find which Azure Virtual Disks Map to Storage Pools

As there are different cache setting recommendations for attached VHDs, you might decide to copy the VHDs to a Premium Storage account. However, when you reattach them to the new DS series VM, you might need to alter the cache settings. It is simpler to apply the Premium Storage recommended cache settings when you have separate VHDs for the SQL Data files and log files (rather than a single VHD that contains both). 

> [AZURE.NOTE] If you have SQL Server data and log files on the same volume, the caching option you choose depends on the IO access patterns for your database workloads. Only testing can demonstrate which caching option is best for this scenario.

However if you are using Windows Storage Spaces which are made up of multiple VHDs you will need to look at your original scripts to identify which attached VHDs are in what specific pool, so you can then set the cache settings accordingly for each disk.

If you do not have original script available to show you which VHDs map to the storage pool, you can use the following steps to determine the disk/storage pool mapping.

For each disk, use the following steps:

1. Get list of disks attached to VM with the **Get-AzureVM** command:

    Get-AzureVM -ServiceName <servicename> -Name <vmname> | Get-AzureDataDisk

1. Note the Diskname and LUN.

	![DisknameAndLUN][2]

1. Remote desktop into the VM. Then go to **Computer Management** | **Device Manager** | **Disk Drives**. Look at the  properties of each of the ‘Microsoft Virtual Disks’

	![VirtualDiskProperties][3]

1. The LUN number here is a reference to the LUN number you specify when attaching the VHD to the VM.
1. For the ‘Microsoft Virtual Disk’ go to the **Details** tab, then in the **Property** list, go to **Driver Key**. In the **Value**, note the **Offset**, which is 0002 in the following screenshot. The 0002 denotes the PhysicalDisk2 that the storage pool references.

	![VirtualDiskPropertyDetails][4]

2. For each storage pool dump out the associated disks:

    Get-StoragePool -FriendlyName AMS1pooldata | Get-PhysicalDisk

	![GetStoragePool][5]
 
Now you can use this information to associate attached VHDs to Physical Disks in Storage Pools.

Once you have mapped VHDs to Physical Disks in Storage Pools you can then detach and copy them over to a Premium Storage account, then attach them with the correct cache setting. Please see the example in the [Appendix](#appendix-migrating-a-multisite-alwayson-cluster-to-premium-storage), steps 8 through 12. These steps show how to extract a VM-attached VHD disk configuration to a CSV file, copy the VHDs, alter the disk configuration cache settings, and finally redeploy the VM as a DS series VM with all the attached disks.

### VM Storage Bandwidth and VHD Storage Throughput 

The amount of storage performance depends on the DS* VM size specified and the VHD sizes. The VMs have different allowances for the number of VHDs that can be attached and the maximum bandwidth they will support (MB/s). For the specific bandwidth numbers, see [Virtual Machine and Cloud Service Sizes for Azure](https://msdn.microsoft.com/library/azure/dn197896.aspx).

Increased IOPS are achieved with larger disk sizes. You should consider this when you think about your migration path. For details, [see the table for IOPS and Disk Types](storage-premium-storage-preview-portal.md#scalability-and-performance-targets-when-using-premium-storage). 

Finally, consider that VMs have different maximum disk bandwidths they will support for all disks attached. Under high load, you could saturate the maximum disk bandwidth available for that VM role size. For example a Standard_DS14 will support up to 512MB/s; therefore, with three P30 disks you could saturate the disk bandwidth of the VM. But in this example, the throughput limit could be exceeded depending on the mix of read and write IOs.

## New Deployments

The next two sections demonstrate how you can deploy SQL Server VMs to Premium Storage. As mentioned before, you do not necessarily need to place the OS disk onto Premium storage. You might choose to do this if you are intending to place any intensive IO workloads on the OS VHD. 

The first example demonstrates utilizing existing Azure Gallery Images. The second example shows how to use a custom VM image that you have in an existing Standard storage account.

> [AZURE.NOTE] These examples assume that you have already created a Regional VNET.

### Create a new VM with Premium Storage with Gallery Image

The example below shows how to place the OS VHD onto premium storage and attach Premium Storage VHDs. However, you can also place the OS disk in a Standard Storage account and then attach VHDs that reside in a Premium Storage account. Both scenarios are demonstrated.

    $mysubscription = "DansSubscription"
    $location = "West Europe"
    
    #Set up subscription 
    Set-AzureSubscription -SubscriptionName $mysubscription 
    Select-AzureSubscription -SubscriptionName $mysubscription -Current  

#### Step 1: Create a Premium Storage Account


    #Create Premium Storage account, note Type
    $newxiostorageaccountname = "danspremsams" 
    New-AzureStorageAccount -StorageAccountName $newxiostorageaccountname -Location $location -Type "Premium_LRS"  

 
#### Step 2: Create a New Cloud Service

    $destcloudsvc = "danNewSvcAms" 
    New-AzureService $destcloudsvc -Location $location 


#### Step 3: Reserve a Cloud Service VIP (Optional)
    #check exisitng reserved VIP
    Get-AzureReservedIP
    
    $reservedVIPName = “sqlcloudVIP” 
    New-AzureReservedIP –ReservedIPName $reservedVIPName –Label $reservedVIPName –Location $location 

#### Step 4: Create a VM Container
    #Generate storage keys for later 
    $xiostorage = Get-AzureStorageKey -StorageAccountName $newxiostorageaccountname 
    
    ##Generate storage acc contexts 
    $xioContext = New-AzureStorageContext –StorageAccountName $newxiostorageaccountname -StorageAccountKey $xiostorage.Primary   
    
    #Create container
    $containerName = 'vhds'
    New-AzureStorageContainer -Name $containerName -Context $xioContext

#### Step 5: Placing OS VHD on Standard or Premium Storage
    #NOTE: Set up subscription and default storage account which will be used to place the OS VHD in
    
    #If you want to place the OS VHD Premium Storage Account
    Set-AzureSubscription -SubscriptionName $mysubscription -CurrentStorageAccount  $newxiostorageaccountname  
    
    #If you wanted to place the OS VHD Standard Storage Account but attach Premium Storage VHDs then you would run this instead:
    $standardstorageaccountname = "danstdams" 
    
    Set-AzureSubscription -SubscriptionName $mysubscription -CurrentStorageAccount  $standardstorageaccountname

#### Step 6: Create VM
    #Get list of available SQL Server Images from the Azure Image Gallery.
    $galleryImage = Get-AzureVMImage | where-object {$_.ImageName -like "*SQL*2014*Enterprise*"} 
    $image = $galleryImage.ImageName 
    
    #Set up Machine Specific Information
    $vmName = "dsDan1"
    $vnet = "dansvnetwesteur"
    $subnet = "SQL"
    $ipaddr = "192.168.0.8"
    
    #Remember to change to DS series VM
    $newInstanceSize = "Standard_DS1"
    
    #create new Avaiability Set 
    $availabilitySet = "cloudmigAVAMS"
    
    #Machine User Credentials
    $userName = "myadmin"
    $pass = "mycomplexpwd4*"
    
    #Create VM Config
    $vmConfigsl = New-AzureVMConfig -Name $vmName -InstanceSize $newInstanceSize -ImageName $image  -AvailabilitySetName $availabilitySet  ` | Add-AzureProvisioningConfig -Windows ` -AdminUserName $userName -Password $pass | Set-AzureSubnet -SubnetNames $subnet | Set-AzureStaticVNetIP -IPAddress $ipaddr
    
    #Add Data and Log Disks to VM Config
    #Note the size specified ‘-DiskSizeInGB 1023’, this will attach 2 x P30 Premium Storage Disk Type
    #Utilising the Premium Storage enabled Storage account
    
    $vmConfigsl | Add-AzureDataDisk -CreateNew -DiskSizeInGB 1023 -LUN 0 -HostCaching "ReadOnly"  -DiskLabel "DataDisk1" -MediaLocation "https://$newxiostorageaccountname.blob.core.windows.net/vhds/$vmName-data1.vhd"
    $vmConfigsl | Add-AzureDataDisk -CreateNew -DiskSizeInGB 1023 -LUN 1 -HostCaching "None"  -DiskLabel "logDisk1" -MediaLocation "https://$newxiostorageaccountname.blob.core.windows.net/vhds/$vmName-log1.vhd"
    
    #Create VM
    $vmConfigsl  | New-AzureVM –ServiceName $destcloudsvc -VNetName $vnet ## Optional (-ReservedIPName $reservedVIPName)  
    
    #Add RDP Endpoint
    $EndpointNameRDPInt = "3389"
    Get-AzureVM -ServiceName $destcloudsvc -Name $vmName | Add-AzureEndpoint -Name "EndpointNameRDP" -Protocol "TCP" -PublicPort "53385" -LocalPort $EndpointNameRDPInt  | Update-AzureVM
    
    #Check VHD storage account, these should be in $newxiostorageaccountname 
    Get-AzureVM -ServiceName $destcloudsvc -Name $vmName | Get-AzureDataDisk
    Get-AzureVM -ServiceName $destcloudsvc -Name $vmName |Get-AzureOSDisk
     

### Create a new VM to Use Premium Storage with a Custom Image

This scenario demonstrates where you have existing customized images that reside in a Standard Storage account. As mentioned if you want to place the OS VHD on Premium Storage you will need to copy the image that exists in the Standard Storage account and transfer them to a Premium Storage before it can be used. If you have an image on premise, you can also use this method to copy that directly to the Premium Storage account.

#### Step 1: Create Storage Account
    $mysubscription = "DansSubscription"
    $location = "West Europe"
    
    #Create Premium Storage account
    $newxiostorageaccountname = "danspremsams" 
    New-AzureStorageAccount -StorageAccountName $newxiostorageaccountname -Location $location -Type "Premium_LRS"  
    
    #Standard Storage account
    $origstorageaccountname = "danstdams" 
 
#### Step 2 Create Cloud Service
    $destcloudsvc = "danNewSvcAms" 
    New-AzureService $destcloudsvc -Location $location 

 
#### Step 3: Use Existing Image
You can use an existing image. Or, you can [take an image of an existing machine](virtual-machines-capture-image-windows-server.md). Note the machine you image does not have to be DS* machine. Once you have the image, the following steps show how to copy it to the Premium Storage account with the **Start-AzureStorageBlobCopy** PowerShell commandlet. 

    #Get storage account keys:
    #Standard Storage account
    $originalstorage =  Get-AzureStorageKey -StorageAccountName $origstorageaccountname
    #Premium Storage account
    $xiostorage = Get-AzureStorageKey -StorageAccountName $newxiostorageaccountname
    
    #Set up contexts for the storage accounts:
    $origContext = New-AzureStorageContext  –StorageAccountName $origstorageaccountname -StorageAccountKey $originalstorage.Primary
    $destContext = New-AzureStorageContext  –StorageAccountName $newxiostorageaccountname -StorageAccountKey $xiostorage.Primary  
 
#### Step 4: Copy Blob between Storage Accounts
    #Get Image VHD from Portal
    $myImageVHD = "dansoldonorsql2k14-os-2015-04-15.vhd"
    $containerName = 'vhds'
    
    #Copy Blob between accounts
    $blob = Start-AzureStorageBlobCopy -SrcBlob $myImageVHD -SrcContainer $containerName `
    -DestContainer vhds -Destblob "prem-$myImageVHD" `
    -Context $origContext -DestContext $destContext  

#### Step 5: Regularly check copy status:
    $blob | Get-AzureStorageBlobCopyState 

#### Step 6: Add Image disk to Azure disk Repository in Subscription
    $imageMediaLocation = $destContext.BlobEndPoint+"/"+$myImageVHD 
    $newimageName = "prem"+"dansoldonorsql2k14"
    
    Add-AzureVMImage -ImageName $newimageName -MediaLocation $imageMediaLocation
 
> [AZURE.NOTE] You may find that even though the status reports as success, you could still get a disk lease error. In this case, wait about 10 minutes.

#### Step 7:  Build the VM
Here you are building the VM from your image and attaching two Premium Storage VHDs:

    $newimageName = "prem"+"dansoldonorsql2k14"
    #Set up Machine Specific Information
    $vmName = "dansolchild"
    $vnet = "westeur"
    $subnet = "Clients"
    $ipaddr = "192.168.0.41"
    
    #This will need to be a new cloud service
    $destcloudsvc = "danregsvcamsxio2"
    
    #Use to DS Series VM
    $newInstanceSize = "Standard_DS1"
    
    #create new Avaiability Set 
    $availabilitySet = "cloudmigAVAMS3"
    
    #Machine User Credentials
    $userName = "myadmin"
    $pass = "theM)stC0mplexP@ssw0rd!”
     
    
    #Create VM Config
    $vmConfigsl2 = New-AzureVMConfig -Name $vmName -InstanceSize $newInstanceSize -ImageName $newimageName  -AvailabilitySetName $availabilitySet  ` | Add-AzureProvisioningConfig -Windows ` -AdminUserName $userName -Password $pass | Set-AzureSubnet -SubnetNames $subnet | Set-AzureStaticVNetIP -IPAddress $ipaddr
    
    $vmConfigsl2 | Add-AzureDataDisk -CreateNew -DiskSizeInGB 1023 -LUN 0 -HostCaching "ReadOnly"  -DiskLabel "DataDisk1" -MediaLocation "https://$newxiostorageaccountname.blob.core.windows.net/vhds/$vmName-Datadisk-1.vhd" 
    $vmConfigsl2 | Add-AzureDataDisk -CreateNew -DiskSizeInGB 1023 -LUN 1 -HostCaching "None"  -DiskLabel "LogDisk1" -MediaLocation "https://$newxiostorageaccountname.blob.core.windows.net/vhds/$vmName-logdisk-1.vhd" 
     
    
    
    $vmConfigsl2 | New-AzureVM –ServiceName $destcloudsvc -VNetName $vnet 

## Existing Deployments that do not use AlwaysOn Availability Groups

> [AZURE.NOTE] For existing deployments, first see the [Prerequisites](#prerequisites-for-premium-storage) section of this topic.

There are different considerations for SQL Server deployments that do not use AlwaysOn Availbility Groups and those that do. If you are not using AlwaysOn and have an existing standalone SQL Server, you can upgrade to Premium Storage by using a new cloud service and storage account. Consider the following options:

- **Create a new SQL Server VM**. You can create a new SQL Server VM that uses a Premium Storage account, as documented in New Deployments. Then backup and restore your SQL Server configuration and user databases. The application will need to be updated to reference the new SQL Server if it is being accessed internally or externally. You would need to copy all ‘out of db’ objects as if you were doing a Side by Side (SxS) SQL Server migration. This includes objects such as logins, certificates, and linked servers.
- **Migrate an existing SQL Server VM**. This will require taking the SQL Server VM offline, then transferring it to a new cloud service, which includes copying all of its attached VHDs to the Premium Storage account. When the VM comes online, the application will reference the server host name as before. Be aware that the size of the existing disk will affect the performance characteristics. For example, a 400 GB disk gets rounded up to a P20. If you know that you do not require that disk performance, then you could recreate the VM as a DS Series VM, and attach Premium Storage VHDs of the size/performance specification you require. Then you could detach and reattach the SQL DB files.

> [AZURE.NOTE] When copying the VHD disks you should beaware of the size, depending on the size will mean what Premium Storage Disk type they fall into, this determines disk performance specification. Azure will round up to the nearest disk size, so if you have a 400GB disk, this will be rounded up to a P20. Depending on your existing IO requirements of the OS VHD, you might not need to migrate this to a Premium Storage account.

If your SQL Server is accessed externally, then the cloud service VIP will change. You will also have to update endptoints, ACLs, and DNS settings.

## Existing Deployments that use AlwaysOn Availability Groups

> [AZURE.NOTE] For existing deployments, first see the [Prerequisites](#prerequisites-for-premium-storage) section of this topic.

Intially in this section we will look at how AlwaysOn interacts with Azure Networking. We will then break down migrations in to two scenarios: migrations where some downtime can be tolerated and migrations where you must achieve minimal downtime.

On-premises SQL Server AlwaysOn Availability Groups use a Listener on-premises that registers a virtual DNS name along with an IP address that is shared between one or more SQL Servers. When clients connect they are routed through the listener IP to the Primary SQL Server. This is the server that owns the AlwaysOn IP resource at that time.
 
![DeploymentsUseAlwaysOn][6]

In Microsoft Azure you can have only one IP address assigned to a NIC on the VM, so in order to achieve the same layer of abstraction as on-premises, Azure utilizes the IP address that is assigned to the Internal/External Load Balancers (ILB/ELB). The IP resource that is shared between the servers is set to the same IP as the ILB/ELB. This is published in the DNS, and client traffic is passed through the ILB/ELB to the Primary SQL Server replica. The ILB/ELB knows which SQL Server is primary since it uses probes to probe the AlwaysOn IP resource. In the previous example, it probes each node that has an endpoint referenced by the ELB/ILB, whichever responds is the Primary SQL Server. 

> [AZURE.NOTE] The ILB and ELB are both assigned to a particular Azure cloud service, therefore any cloud migration in Azure will most likely mean that the Load Balancer IP will change. 

### Migrating AlwaysOn deployments that can allow some downtime

There are two strategies to migrate AlwaysOn deployments that allow for some downtime:

1. **Add more secondary replicas to an existing AlwaysOn Cluster**
1. **Migrate to a new AlwaysOn Cluster**

#### 1. Add more Secondary Replicas to an Existing AlwaysOn Cluster

One strategy is to add more secondaries to the AlwaysOn Availability Group. You need to add these into a new cloud service and update the listener with the new load balancer IP. 

##### Points of Downtime:

- Cluster Validation.
- Testing AlwaysOn failovers for New Secondaries.

If you are using Windows Storage Pools within the VM for higher IO throughput, then these will be taken offline during a Full Cluster Validation. The validation test is required when you add nodes to the cluster. The time it takes to run the test can vary, so you should test this in your representative test environment to get an approximate time of how long this will take. 

You should provision time where you can perform manual failover and chaos testing on the newly added nodes to ensure AlwaysOn High Availability functions as expected.

![DeploymentUseAlwaysOn2][7]

> [AZURE.NOTE] You should stop all instances of SQL Server where the Storage Pools are used before the Validaion runs. 
##### High Level Steps

1. Create two new SQL Servers in new cloud service with attached Premium Storage.
1. Copy over FULL backups and restore with **NORECOVERY**.
1. Copy over ‘out of user DB’ dependent objects, such as logins etc.
1. Create new a new Internal Load Balancer (ILB) or use an External Load Balancer (ELB), and then set up Load Balanced Endpoints on both new nodes.
> [AZURE.NOTE] Check all Nodes have the correct Endpoint configuration before you continue

1. Stop User/Application Access to the SQL Server (if using Storage Pools).
1. Stop SQL Server Engine Services on All Nodes (if using Storage Pools).
1. Add new Nodes to cluster and run full validation. 
1. Once Validation is successful, start all SQL Server Services.
1. Backup Transaction logs, and restore user databases.
1. Add new nodes into the AlwaysOn Availability Group and place replication into **Syncronous**. 
1. Add the IP address resource of the new Cloud Service ILB/ELB through PowerShell for AlwaysOn based on the Multi-site example in the [Appendix](#appendix-migrating-a-multisite-alwayson-cluster-to-premium-storage). In Windows clustering, set the **Possible owners** of the **IP Address** resource to the new nodes old. See the ‘Adding IP Address Resource on Same Subnet’ section of the [Appendix](#appendix-migrating-a-multisite-alwayson-cluster-to-premium-storage).
1. Failover to one of the new nodes.
1. Make the new nodes Auto Failover Partners and test failovers.
1. Remove original nodes from Availability Group.

##### Advantages

- New SQL Servers can be tested (SQL Server and Application) before they are added to AlwaysOn.
- You can change the VM size and customize the storage to your exact requirements. However, it would be beneficial to keep all the SQL file paths the same.
- You can control when the transfer of the DB backups to the Secondary Replicas are started. This differs from using Azure **Start-AzureStorageBlobCopy** commandlet to copy VHDs, because that is an asynchronous copy.

##### Disadvantages
- When using Windows Storage Pools, there is Cluster downtime during the Full Cluster Validation for the new additional nodes. 
- Depending on the SQL Server Version and the existing number of secondary replicas, you might not be able to add more secondary replicas without removing existing secondaries.
- There could be long SQL data transfer time while setting up the secondaries.
- There is additional cost during migration while you have new machines running in paralell.

#### 2. Migrate to a New AlwaysOn Cluster

Another strategy is to create a brand new AlwaysOn Cluster with brand new nodes in new cloud service and then redirect the clients to use it.

##### Points of Downtime

There is downtime when you transfer applications and users to the new AlwaysOn listener. The downtime depends on:

- The time taken to restore final transaction log backups to databases on new servers.
- The time taken to update client applications to use new AlwaysOn listener.

##### Advantages

- You can test the actual production environment, SQL Server, and OS build changes.
- You have the option to customize the storage and to potentially reduce size of VM. This could result in cost reduction.
- You can update your SQL Server build or version during this process. You can also upgrade the Operating System.
- The previous AlwaysOn Cluster can act as a solid rollback target.

##### Disadvantages

- You need to change the DNS name of the listener if you want both AlwaysOn clusters running simultaneously. This adds administration overhead during the migration as client application strings must reflect the new Listener name.
- You must implement a synchronization mechanism between the two environments to keep them as close as possible to minimize the final synchronization requirements before migration.
- There is added cost during migration while you have the new environment running.

### Migrating AlwaysOn Deployments for Minimal Downtime

There are two strategies for migrating AlwaysOn deployments for minimal downtime:

1. **Utilize an Existing Secondary: Single-Site**
1. **Utilize Existing Secondary Replica(s): Multi-Site**

#### 1. Utilize an Existing Secondary: Single-Site

One strategy for minimal downtime is to take an existing cloud secondary and remove it from the current cloud service. Then copy the VHDs to the new Premium Storage account, and create the VM in the new cloud service. Then update the listener in clustering and failover.

##### Points of Downtime

- There is downtime when you update the final node with the Load Balanaced endpoint.
- Your client reconnection might be delayed depending on your client/DNS configuration.
- There is additional downtime if you choose to take the AlwaysOn Cluster group offline to swap out the IP addresses. You can avoid this by using an OR dependancy and Possible Owners for the added IP Address resource. See the ‘Adding IP Address Resource on Same Subnet’ section of the [Appendix](#appendix-migrating-a-multisite-alwayson-cluster-to-premium-storage).

> [AZURE.NOTE] When you want the added node to partake in as an AlwaysOn Failover Partner, you need to add an Azure Endpoint with a reference to the Load Balanced Set. When you run the **Add-AzureEndpoint** command to do this, current connections to remain open, but new connections to the listener will not be able to be established until the load balancer has updated. In testing this was seen to last 90-120seconds, this should be tested. 

##### Advantages

- No extra cost incurred during migration.
- A one-to-one migration.
- Reduced complexity.
- Allows for increased IOPS from Premium Storage SKUs. When the disks are detached from the VM and copied to the new cloud service, a 3rd party tool can be used to increase the VHD size, which provides higher throughputs. For increasing VHD sizes, see this [forum discussion](https://social.msdn.microsoft.com/Forums/azure/4a9bcc9e-e5bf-4125-9994-7c154c9b0d52/resizing-azure-data-disk?forum=WAVirtualMachinesforWindows).

##### Disadvantages

- There is a temporary loss of HA and DR during migration.
- As this is a 1:1 migration, you will have to use a minimum VM size that will support your number of VHDs, so you might not be able to downsize your VMs.
- This scenario would use the Azure **Start-AzureStorageBlobCopy** commandlet, which is asynchronous. There is no SLA on copy completion. The time of the copies varies, while this depends on wait in queue it will also depend on the amount of data to transfer. The copy time increases if the transfer is going to another Azure data center that supports Premium Storage in another region. If you just have 2 nodes, consider a possible mitigation in case the copy takes longer than in testing. This could include the following ideas.
	- Add a temporary 3rd SQL Server node for HA before the migration with agreed downtime.
	- Run the migration outside of Azure scheduled maintenance.
	- Ensure you have configured your cluster quorum correctly.  

##### High Level Steps

This document does not demonstrate a complete end to end example, however the [Appendix](#appendix-migrating-a-multisite-alwayson-cluster-to-premium-storage) provides details that can be leveraged to perform this.
 
![MinimalDowntime][8]

- Gather disk configuration, and remove the node (do not delete attached VHDs).
- Create a Premium Storage account and copy VHDs from the Standard Storage account
- Create new cloud service and redeploy the SQL2 VM in that cloud service. Create the VM using the copied original OS VHD and attaching the copied VHDs.
- Configure ILB / ELB and add Endpoints.
- Update Listener by either:
	- Taking the AlwaysOn Group offline and updating the AlwaysOn Listener with new ILB / ELB IP address. 
	- Or adding the IP addess resource of new Cloud Service ILB/ELB through PowerShell into Windows clustering. Then set the Possible owners of the IP Address resource to the migrated node, SQL2, and set this as OR dependency in the Network Name. See the ‘Adding IP Address Resource on Same Subnet’ section of the [Appendix](#appendix-migrating-a-multisite-alwayson-cluster-to-premium-storage).
- Check DNS configuration/propogation to the clients.
- Migrate SQL1 VM, and go through steps 2 – 4.
- If using steps 5ii, then add SQL1 as a Possible Owner for the added IP Address Resource
- Test failovers.

#### 2. Utilize Existing Secondary Replica(s): Multi-Site

If you have nodes in more than one Azure datacenter (DC) or if you have a hybrid environment, then you can use an AlwaysOn configuration in this environment to minimize downtime.

The approach is to change the AlwaysOn synchronization to Synchronous for the on-premises or secondary Azure DC, and then failover over to that SQL Server. Then copy the VHDs to a Premium Storage account, and redeploy the machine into a new cloud service. Update the listener, and then fail back. 

##### Points of Downtime

The downtime consists of the time to failover to the alternative DC and back. It also depends on your client/DNS configuration and your client reconnection may be delayed.
Consider the following example of a hybrid AlwaysOn configuration:

![MultiSite1][9]

##### Advantages

- You can utilize existing infrastructure.
- You have the option to pre-upgrade the Azure storage on the DR Azure DC first.
- The DR Azure DC storage can be reconfigured.
- There is a minimum of two failovers during migration, excluding test failovers.
- You do not need to move SQL Server data with backup and restore.

##### Disadvantages

- Depending on client access to SQL Server, there might be increased latency when SQL Server is running in an alternative DC to the application.
- The copy time of VHDs to Premium storage could be long. This might affect your decision on whether to keep the node in the Availability Group. Consider this for when log intensive work loads are running during the migrationis required, since the Primary node will have to keep the unreplicated transactions in its transaction log. Therefore this could grow significantly.
- This scenario would use the Azure **Start-AzureStorageBlobCopy** commandlet, which is asynchronous. There is no SLA on completion. The time of the copies varies, while this depends on wait in queue, it will also depend on the amount of data to transfer. Therefore you just have one node in your 2nd data center, you should take mitigation steps in case the copy takes longer than in testing. This could include the following ideas.
	- Add a temporary 2nd SQL node for HA before the migration with agreed downtime.
	- Run the migration outside of Azure scheduled maintenance. 
	- Ensure you have configured your cluster quorum correctly. 

This scenario assumes that you have documented your install and know how the storage is mapped in order to make changes for optimal disk cache settings.

##### High Level Steps
![Multisite2][10]

- Make the on-premises / alternate Azure DC the SQL Server Primary, and make it the other Auto Failover Partner (AFP). 
- Gather disk configuration information from SQL2, and remove the node (do not delete attached VHDs).
- Create a Premium Storage account and copy VHDs from the Standard Storage account.
- Create a new cloud service and create the SQL2 VM with its Premiums Storage disks attached.
- Configure ILB / ELB and add Endpoints.
- Update the AlwaysOn Listener with new ILB / ELB IP address and test failover.
- Check the DNS configuration.
- Change the AFP to SQL2, and then migrate SQL1 and go through steps 2 – 5.
- Test failovers.
- Switch the AFP back to SQL1 and SQL2

## Appendix: Migrating a Multisite AlwaysOn Cluster to Premium Storage 

The remainder of this topic provides a detailed example of converting a multi-site AlwaysOn cluster to Premium storage. It also converts the Listener from using an external load balancer (ELB) to an internal load balancer (ILB).

### Environment

- Windows 2k12 / SQL 2k12
- 1 DB Files on SP
- 2 x Storage Pools per Node

![Appendix1][11]
 
### VM:
 
In this example we are going to demonstrate moving from an ELB to ILB. ELB was available before ILB, so this shows how to switch to this during the migration.

![Appendix2][12]

### Pre Steps: Connect to Subscription

    Add-AzureAccount
    
    #Set up subscription
    Get-AzureSubscription 

#### Step 1: Create New Storage Account and Cloud Service
    $mysubscription = "DansSubscription"
    $location = "West Europe"
    
    #Storage accounts
    #current storage account where the vm to migrate resides
    $origstorageaccountname = "danstdams"
    
    #Create Premium Storage account
    $newxiostorageaccountname = "danspremsams" 
    New-AzureStorageAccount -StorageAccountName $newxiostorageaccountname -Location $location -Type "Premium_LRS"  
    
    #Generate storage keys for later
    $originalstorage =  Get-AzureStorageKey -StorageAccountName $origstorageaccountname
    $xiostorage = Get-AzureStorageKey -StorageAccountName $newxiostorageaccountname
    
    #Generate storage acc contexts
    $origContext = New-AzureStorageContext  –StorageAccountName $origstorageaccountname -StorageAccountKey $originalstorage.Primary
    $xioContext = New-AzureStorageContext  –StorageAccountName $newxiostorageaccountname -StorageAccountKey $xiostorage.Primary  
    
    #Set up subscription and default storage account
    Set-AzureSubscription -SubscriptionName $mysubscription -CurrentStorageAccount $origstorageaccountname
    Select-AzureSubscription -SubscriptionName $mysubscription -Current 
    
    #CREATE NEW CLOUD SVC
    $vnet = "dansvnetwesteur"
    
    ##Existing cloud service
    $sourceSvc="dansolSrcAms"
    
    ##Create new cloud service
    $destcloudsvc = "danNewSvcAms" 
    New-AzureService $destcloudsvc -Location $location 

#### Step 2: Increase the Permitted Failures on Resources <Optional>
On certain resources that belong to your AlwaysOn Availability Group there are limits on how many failures that can occur in a period, where the cluster service will attempt to restart the resource group. It is recommended you increase this whilst you are walking through this procedure, since if you don’t manually failover and trigger failovers by shutting down machines you can get close to this limit.

It would be prudent to double the failure allowance, to do this in Failover Cluster Manager, go to the Properties of the AlwaysOn resource group:
 
![Appendix3][13]

Change the Maximum Failures to 6.

#### Step 3: Addition IP Address Resource for Cluster Group <Optional>

If you have only one IP address for the Cluster Group and this is aligned to the cloud subnet, beware, if you accidentally take offline all cluster nodes in the cloud on that network then the Cluster IP resource and Cluster Network Name will not be able to come online. In the event of this it will prevent updates to other cluster resources. 

#### Step 4: DNS Configuration

To implement a smooth transition depends on how DNS is being utilised and updated.
When AlwaysOn is installed, it creates a Windows Cluster Resource group, if you open Failover Cluster Manager, you will see that at a minimum it will have three resources, the two that the document refers to are:

- Virtual Network Name (VNN) – This is the DNS name that client connect to when wanting to connect to SQL Servers via AlwaysOn.
- IP Address Resource – This is the IP address that associated with the VNN, you can have more than one, and in a multisite configuration you will have an IP address per site/subnet.

When connecting to SQL Sevrer, the SQL Server Client driver will retrieve the DNS records associated with the listener and try to connect to each AlwaysOn associated IP address, below we discuss some factors that can influence this.

The number of concurrent DNS records that are associated with the listener name depends not only on the number of IP addresses associated, but the ‘RegisterAllIpProviders’setting in Failover Clustering for the AlwaysON VNN resource.

When you deploy AlwaysOn in Azure there are different steps to create the Listener and IP Addresses, you have to manually configure the ‘RegisterAllIpProviders’ to 1, this is different to an on premise AlwaysOn deployment where it is already set to 1. 

If ‘RegisterAllIpProviders’ is 0, then you will only see one DNS record in DNS associated with the Listener:

![Appendix4][14]

If ‘RegisterAllIpProviders’ is 1:
 
![Appendix5][15]

The code below will dump out the VNN settings and set it for you, please note, for the change to take effect you will need to take the the VNN offline and turn it back online, this taking the Listener offline causing client connectivity disruption. 

    ##Always On Listener Name
    $ListenerName = "Mylistener"
    ##Get AlwaysOn Network Name Settings
    Get-ClusterResource $ListenerName| Get-ClusterParameter
    ##Set RegisterAllProvidersIP 
    Get-ClusterResource $ListenerName| Set-ClusterParameter RegisterAllProvidersIP  1 

In a later migration step you will need to update the AlwaysOn listener with an updated IP address that will reference a load balancer, this will involve an IP Address resource removal and addition. After the IP update, you need to ensure the new IP address has been updated in DNS Zone and that the clients are updating their local DNS cache. 

If you clients reside in a different network segment and reference a different DNS server, you need to consider what happens about DNS Zone Transfer during the migration, as the application reconnect time will be constrained by at least the Zone Transfer Time of any new IP addresses for the listener. If you are under time constraint here, you should discuss and test forcing an incremental zone transfer with your Windows teams, and also put the DNS host record to a lower Time To Live (TTL), so the clients update. For more information, see [Incremental Zone Transfers](https://technet.microsoft.com/library/cc958973.aspx) and [Start-DnsServerZoneTransfer](https://technet.microsoft.com/library/jj649917.aspx).

By default the TTL for DNS Record that is associated with the Listener in AlwaysOn in Azure is 1200 seconds. You may wish to reduce this if you are under time constraint during your migration to ensure the clients update their DNS with the updated IP address for the listener. You can see and modify the configuration by dumping out the configuration of the VNN:

    $AGName = "myProductionAG"
    $ListenerName = "Mylistener"
    #Look at HostRecordTTL
    Get-ClusterResource $ListenerName| Get-ClusterParameter
    
    #Set HostRecordTTL Examples
    Get-ClusterResource $ListenerName| Set-ClusterParameter -Name "HostRecordTTL" 120 

Please note, the lower the ‘HostRecordTTL’, a higher amount of DNS traffic will occur.

##### Client Application Settings

If your SQL client application supports the .Net 4.5 SQLClient, then you can use ‘MULTISUBNETFAILOVER=TRUE’ keyword, this is recommended to be applied as it allows for faster connection to SQL AlwaysOn Availability Group during failover. It enumerates through all IP addresses associated with the AlwaysOn listener in parallel, and performs a more aggressive TCP connection retry speed during a failover.

For more information regarding the settings above, please see [MultiSubnetFailover Keyword and Associated Features](https://msdn.microsoft.com/library/hh213080.aspx#MultiSubnetFailover). Also see [SqlClient Support for High Availability, Disaster Recovery](https://msdn.microsoft.com/library/hh205662(v=vs.110).aspx).

#### Step 5: Cluster Quorum Settings

As you are going to be taking out at least one SQL Server down at a time, you should modify the cluster quorum setting, if using File Share Witnes (FSW) with 2 nodes, you should set the quorum to allow node majority and utilise dynamic voting, and this is to allow for a single node to remain standing.


    Set-ClusterQuorum -NodeMajority  

For more information on managing and configuring the cluster quorum, please see [Configure and Manage the Quorum in a Windows Server 2012 Failover Cluster](https://technet.microsoft.com/en-us/library/jj612870.aspx).

#### Step 6: Extract Existing Endpoints and ACLs
    #GET Endpoint info
    Get-AzureVM -ServiceName $destcloudsvc -Name $vmNameToMigrate | Get-AzureEndpoint
    #GET ACL Rules for Each EP, this example is for the AlwaysOn Endpoint
    Get-AzureVM -ServiceName $destcloudsvc -Name $vmNameToMigrate | Get-AzureAclConfig -EndpointName "myAOEndPoint-LB"  

Save these to a text file.

#### Step 7: Change Failover Partners and Replication Modes

If you have more than 2 SQL Servers, you should change the failover of another secondary in another DC or on premise to ‘Synchronous’ and make it an Automatic Failover Partner (AFP), this is so you maintain HA whilst you are making changes. You can do this through TSQL of modify though SSMS:
 
![Appendix6][16]

#### Step 8: Remove Secondary VM from Cloud Service

You should be planning to migrate a cloud secondary node first, if this is currently primary, you should initiate a manual failover.

    $vmNameToMigrate="dansqlams2"
    
    #Check machine status 
    Get-AzureVM -ServiceName $sourceSvc -Name $vmNameToMigrate 
    
    #Shutdown secondary VM
    Get-AzureVM -ServiceName $sourceSvc -Name $vmNameToMigrate | stop-AzureVM
    
    
    #Extract disk configuration
    
    ##Building Existing Data Disk Configuration
    $file = "C:\Azure Storage Testing\mydiskconfig_$vmNameToMigrate.csv"
    $datadisks = @(Get-AzureVM -ServiceName $sourceSvc -Name $vmNameToMigrate | Get-AzureDataDisk )
    Add-Content $file “lun, vhdname, hostcaching, disklabel, diskName”
    foreach ($disk in $datadisks)
    {
      $vhdname = $disk.MediaLink.AbsolutePath -creplace  "/vhds/"
      $disk.Lun, , $disk.HostCaching, $vhdname, $disk.DiskLabel,$disks.DiskName 
    # Write-Host "copying disk $disk"
    $adddisk = "{0},{1},{2},{3},{4}" -f $disk.Lun,$vhdname, $disk.HostCaching, $disk.DiskLabel, $disk.DiskName 
    $adddisk | add-content -path $file
    }
    
    #Get OS Disk
    $osdisks = Get-AzureVM -ServiceName $sourceSvc -Name $vmNameToMigrate | Get-AzureOSDisk ## | select -ExpandProperty MediaLink
    $osvhdname = $osdisks.MediaLink.AbsolutePath -creplace  "/vhds/"
    $osdisks.OS, $osdisks.HostCaching, $osvhdname, $osdisks.DiskLabel, $osdisks.DiskName
    $addosdisk = "{0},{1},{2},{3},{4}" -f $osdisks.OS,$osvhdname, $osdisks.HostCaching, $osdisks.Disklabel , $osdisks.DiskName 
    $addosdisk | add-content -path $file
    
    #Import disk config
    $diskobjects  = Import-CSV $file
    
    #Check disk config, make sure below returns the disks associated with the VM
    $diskobjects 
    
    #Identify OS Disk
    $osdiskimport = $diskobjects | where {$_.lun -eq "Windows"}
    $osdiskforbuild = $osdiskimport.diskName
    
    #Check machibe is off
    Get-AzureVM -ServiceName $sourceSvc -Name  $vmNameToMigrate 
    
    #Drop machine and rebuild to new cls
    Remove-AzureVM -ServiceName $sourceSvc -Name $vmNameToMigrate 

#### Step 9: Change Disk Caching Settings in CSV file and Save

For data volumes these should be set to READONLY.

For TLOG volumes these should be set to NONE.
 
![Appendix7][17]

#### Step 10: Copy VHDS
    #Ensure you have created the container for these:
    $containerName = 'vhds'
    
    #Create container
    New-AzureStorageContainer -Name $containerName -Context $xioContext 
    
    ####DISK COPYING####
    #Get disks from csv, get settings for each VHDs and copy to Premium Storage accoun
    ForEach ($disk in $diskobjects)
       {
       $lun = $disk.Lun
       $vhdname = $disk.vhdname
       $cacheoption = $disk.HostCaching
       $disklabel = $disk.DiskLabel
       $diskName = $disk.DiskName
       Write-Host "Copying Disk Lun $lun, Label : $disklabel, VHD : $vhdname has cache setting : $cacheoption"
    
       #Start async copy
       Start-AzureStorageBlobCopy -srcUri "https://$origstorageaccountname.blob.core.windows.net/vhds/$vhdname" `
    -SrcContext $origContext `
    -DestContainer $containerName `
    -DestBlob $vhdname `
    -DestContext $xioContext
       }
     
 

You can check the copy status of the VHDs to the Premium Storage account:

    ForEach ($disk in $diskobjects)
       {
       $lun = $disk.Lun
       $vhdname = $disk.vhdname
       $cacheoption = $disk.HostCaching
       $disklabel = $disk.DiskLabel
       $diskName = $disk.DiskName
      
       $copystate = Get-AzureStorageBlobCopyState -Blob $vhdname -Container $containerName -Context $xioContext
    Write-Host "Copying Disk Lun $lun, Label : $disklabel, VHD : $vhdname, STATUS = " $copystate.Status 
       }
 
![Appendix8][18]

Wait until all these are recorded as success.

For information for individual blobs:

    Get-AzureStorageBlobCopyState -Blob "blobname.vhd" -Container $containerName -Context $xioContext 

#### Step 11: Register OS Disk

    #Change storage account
    Set-AzureSubscription -SubscriptionName $mysubscription -CurrentStorageAccount $newxiostorageaccountname 
    Select-AzureSubscription -SubscriptionName $mysubscription -Current 
    
    #Register OS disk
    $osdiskimport = $diskobjects | where {$_.lun -eq "Windows"}
    $osvhd = $osdiskimport.vhdname
    $osdiskforbuild = $osdiskimport.diskName
    
    #Registering OS disk, but as XIO disk
    $xioDiskName = $osdiskforbuild + "xio"
    Add-AzureDisk -DiskName $xioDiskName -MediaLocation  "https://$newxiostorageaccountname.blob.core.windows.net/vhds/$osvhd"  -Label "BootDisk" -OS "Windows"

#### Step 12: Import Secondary into New Cloud Service

The code below also uses the added option here you can import the machine and use the retainable VIP.

    #Build VM Config
    $ipaddr = "192.168.0.5"
    #Remember to change to XIO
    $newInstanceSize = "Standard_DS13"
    $subnet = "SQL"
    
    #Create new Avaiability Set 
    $availabilitySet = "cloudmigAVAMS"
    
    #build machine config into object
    $vmConfig = New-AzureVMConfig -Name $vmNameToMigrate -InstanceSize $newInstanceSize -DiskName $xioDiskName -AvailabilitySetName $availabilitySet  ` | Add-AzureProvisioningConfig -Windows ` | Set-AzureSubnet -SubnetNames $subnet | Set-AzureStaticVNetIP -IPAddress $ipaddr
    
    #Reload disk config
    $diskobjects  = Import-CSV $file
    $datadiskimport = $diskobjects | where {$_.lun -ne "Windows"}
    
    ForEach ( $attachdatadisk in $datadiskimport) 
       {
    $label = $attachdatadisk.disklabel
    $lunNo = $attachdatadisk.lun
    $hostcach = $attachdatadisk.hostcaching
    $datadiskforbuild = $attachdatadisk.diskName
    $vhdname = $attachdatadisk.vhdname
    
    ###Attaching disks to a VM during a deploy to a new cloud service and new storage account is different from just attaching VHDs to just a redeploy in a new cloud service
    $vmConfig | Add-AzureDataDisk -ImportFrom -MediaLocation "https://$newxiostorageaccountname.blob.core.windows.net/vhds/$vhdname" -LUN $lunNo -HostCaching $hostcach -DiskLabel $label
    
    }
    
    #Create VM
    $vmConfig  | New-AzureVM –ServiceName $destcloudsvc –Location $location -VNetName $vnet ## Optional (-ReservedIPName $reservedVIPName)
 
#### Step 13: Create ILB on New Cloud Svc, Add Load Balanced Endpoints and ACLs
    #Check for existing ILB
    GET-AzureInternalLoadBalancer -ServiceName $destcloudsvc
    
    $ilb="sqlIntIlbDest"
    $subnet = "SQL"
    $IP="192.168.0.25"
    Add-AzureInternalLoadBalancer -ServiceName $destcloudsvc -InternalLoadBalancerName $ilb –SubnetName $subnet –StaticVNetIPAddress $IP
    
    #Endpoints
    $epname="sqlIntEP"
    $prot="tcp"
    $locport=1433
    $pubport=1433
    Get-AzureVM –ServiceName $destcloudsvc –Name $vmNameToMigrate  | Add-AzureEndpoint -Name $epname -Protocol $prot -LocalPort $locport -PublicPort $pubport -ProbePort 59999 -ProbeIntervalInSeconds 5 -ProbeTimeoutInSeconds 11  -ProbeProtocol "TCP" -InternalLoadBalancerName $ilb -LBSetName $ilb -DirectServerReturn $true | Update-AzureVM
    
    #SET Azure ACLs or Network Security Groups & Windows FWs 
     
    #http://msdn.microsoft.com/en-us/library/azure/dn495192.aspx
    
    ####WAIT FOR FULL AlwaysOn RESYNCRONISATION!!!!!!!!!#####

####Step 14: Update AlwaysOn 
    #Code to be executed on a Cluster Node
    $ClusterNetworkNameAmsterdam = "Cluster Network 2" # the azure cluster subnet network name
    $newCloudServiceIPAmsterdam = "192.168.0.25" # IP address of your cloud service 
    
    $AGName = "myProductionAG"
    $ListenerName = "Mylistener" 
    
    
    Add-ClusterResource "IP Address $newCloudServiceIPAmsterdam" -ResourceType "IP Address" -Group $AGName -ErrorAction Stop |  Set-ClusterParameter -Multiple @{"Address"="$newCloudServiceIPAmsterdam";"ProbePort"="59999";SubnetMask="255.255.255.255";"Network"=$ClusterNetworkNameAmsterdam;"OverrideAddressMatch"=1;"EnableDhcp"=0} -ErrorAction Stop 
    
    #set dependancy and NETBIOS, then remove old IP address
    
    #set NETBIOS, then remove old IP address
    Get-ClusterGroup $AGName | Get-ClusterResource -Name "IP Address $newCloudServiceIPAmsterdam" | Set-ClusterParameter -Name EnableNetBIOS -Value 0 
     
    #set dependency to Listener (OR Dependency) and delete previous IP Address resource that references:
    
    #Make sure no static records in DNS 
    
![Appendix9][19]

Now remove the old cloud service IP Address.

![Appendix10][20]
 
#### Step 15: DNS Update Check

You should now check DNS Servers on your SQL Server client networks and make sure that clustering has added the extra host record for the added IP address. If those DNS servers have not updated, consider forcing a DNS Zone transfer and ensure that the clients in there subnet are able to resolve to both AlwaysOn IP Addresses, this is so you do not need to wait for automatic DNS replication. 

#### Step 16: Reconfigure AlwaysOn 

At this point you wait for the secondary that node that was migrated to fully resynchronise with the on-premise node and switch to synchronous replication node and make it the AFP.  

#### Step 17: Migrate Second Node
    $vmNameToMigrate="dansqlams1"
    
    Get-AzureVM -ServiceName $sourceSvc -Name $vmNameToMigrate 
    
    #Get endpoint information
    $endpoint = Get-AzureVM -ServiceName $sourceSvc  -Name $vmNameToMigrate | Get-AzureEndpoint
    
    #Shutdown VM
    Get-AzureVM -ServiceName $sourceSvc -Name $vmNameToMigrate | stop-AzureVM
    
    #Get disk config
    
    #Building Existing Data Disk Configuration
    $file = "C:\Azure Storage Testing\mydiskconfig_$vmNameToMigrate.csv"
    $datadisks = @(Get-AzureVM -ServiceName $sourceSvc -Name $vmNameToMigrate | Get-AzureDataDisk )
    Add-Content $file “lun, vhdname, hostcaching, disklabel, diskName”
    foreach ($disk in $datadisks)
    {
      $vhdname = $disk.MediaLink.AbsolutePath -creplace  "/vhds/"
      $disk.Lun, , $disk.HostCaching, $vhdname, $disk.DiskLabel,$disks.DiskName 
    # Write-Host "copying disk $disk"
    $adddisk = "{0},{1},{2},{3},{4}" -f $disk.Lun,$vhdname, $disk.HostCaching, $disk.DiskLabel, $disk.DiskName 
    $adddisk | add-content -path $file
    }
    
    #Get OS Disk
    $osdisks = Get-AzureVM -ServiceName $sourceSvc -Name $vmNameToMigrate | Get-AzureOSDisk ## | select -ExpandProperty MediaLink
    $osvhdname = $osdisks.MediaLink.AbsolutePath -creplace  "/vhds/"
    $osdisks.OS, $osdisks.HostCaching, $osvhdname, $osdisks.DiskLabel, $osdisks.DiskName
    $addosdisk = "{0},{1},{2},{3},{4}" -f $osdisks.OS,$osvhdname, $osdisks.HostCaching, $osdisks.Disklabel , $osdisks.DiskName 
    $addosdisk | add-content -path $file
    
    #Import disk config
    $diskobjects  = Import-CSV $file
    
    #Check disk configuration
    $diskobjects 
    
    #Identify OS Disk
    $osdiskimport = $diskobjects | where {$_.lun -eq "Windows"}
    $osdiskforbuild = $osdiskimport.diskName
    
    #Check machine is off
    Get-AzureVM -ServiceName $sourceSvc -Name  $vmNameToMigrate 
    
    #Drop machine and rebuild to new cls
    Remove-AzureVM -ServiceName $sourceSvc -Name $vmNameToMigrate 

#### Step 18: Change Disk Caching Settings in CSV file and Save

For data volumes these should be set to READONLY.

For TLOG volumes these should be set to NONE.
 
![Appendix11][21]

#### Step 19: Create New Independent Storage Account for Secondary Node
    $newxiostorageaccountnamenode2 = "danspremsams2" 
    New-AzureStorageAccount -StorageAccountName $newxiostorageaccountnamenode2 -Location $location -Type "Premium_LRS"  
    
    #Reset the storage account src if node 1 in a different storage account
    $origstorageaccountname2nd = "danstdams2"
    
    #Generate storage keys for later
    $xiostoragenode2 = Get-AzureStorageKey -StorageAccountName $newxiostorageaccountnamenode2
    
    #Generate storage acc contexts
    $xioContextnode2 = New-AzureStorageContext  –StorageAccountName $newxiostorageaccountnamenode2 -StorageAccountKey $xiostoragenode2.Primary  
    
    #Set up subscription and default storage account
    Set-AzureSubscription -SubscriptionName $mysubscription -CurrentStorageAccount $newxiostorageaccountnamenode2
    Select-AzureSubscription -SubscriptionName $mysubscription -Current 

#### Step 20: Copy VHDS
    #Ensure you have created the container for these:
    $containerName = 'vhds'
    
    #Create container
    New-AzureStorageContainer -Name $containerName -Context $xioContextnode2  
    
    ####DISK COPYING####
    ##get disks from csv, get settings for each VHDs and copy to Premium Storage accoun
    ForEach ($disk in $diskobjects)
       {
       $lun = $disk.Lun
       $vhdname = $disk.vhdname
       $cacheoption = $disk.HostCaching
       $disklabel = $disk.DiskLabel
       $diskName = $disk.DiskName
       Write-Host "Copying Disk Lun $lun, Label : $disklabel, VHD : $vhdname has cache setting : $cacheoption"
      
       #Start async copy
       Start-AzureStorageBlobCopy -srcUri "https://$origstorageaccountname2nd.blob.core.windows.net/vhds/$vhdname" `
	    -SrcContext $origContext `
	    -DestContainer $containerName `
	    -DestBlob $vhdname `
	    -DestContext $xioContextnode2
       }
    
    #Check for copy progress
    
    #check induvidual blob status
    Get-AzureStorageBlobCopyState -Blob "danRegSvcAms-dansqlams1-2014-07-03.vhd" -Container $containerName -Context $xioContext
     
    
You can check the VHD copy status for all VHDs:
    ForEach ($disk in $diskobjects)
       {
       $lun = $disk.Lun
       $vhdname = $disk.vhdname
       $cacheoption = $disk.HostCaching
       $disklabel = $disk.DiskLabel
       $diskName = $disk.DiskName
      
       $copystate = Get-AzureStorageBlobCopyState -Blob $vhdname -Container $containerName -Context $xioContextnode2
    Write-Host "Copying Disk Lun $lun, Label : $disklabel, VHD : $vhdname, STATUS = " $copystate.Status 
       }
     
![Appendix12][22]

Wait until all these are recorded as success.

For information for individual blobs:
    #Check induvidual blob status
    Get-AzureStorageBlobCopyState -Blob "danRegSvcAms-dansqlams1-2014-07-03.vhd" -Container $containerName -Context $xioContextnode2

#### Step 21: Register OS Disk
    #change storage account to the new XIO storage account
    Set-AzureSubscription -SubscriptionName $mysubscription -CurrentStorageAccount $newxiostorageaccountnamenode2
    Select-AzureSubscription -SubscriptionName $mysubscription -Current 
    
    #Register OS disk
    $osdiskimport = $diskobjects | where {$_.lun -eq "Windows"}
    $osvhd = $osdiskimport.vhdname
    $osdiskforbuild = $osdiskimport.diskName
    
    #Registering OS disk, but as XIO disk
    $xioDiskName = $osdiskforbuild + "xio"
    Add-AzureDisk -DiskName $xioDiskName -MediaLocation  "https://$newxiostorageaccountnamenode2.blob.core.windows.net/vhds/$osvhd"  -Label "BootDisk" -OS "Windows" 
    
    #Build VM Config
    $ipaddr = "192.168.0.4"
    $newInstanceSize = "Standard_DS13"
    
    #Join to existing Avaiability Set 
    
    #Build machine config into object
    $vmConfig = New-AzureVMConfig -Name $vmNameToMigrate -InstanceSize $newInstanceSize -DiskName $xioDiskName -AvailabilitySetName $availabilitySet  ` | Add-AzureProvisioningConfig -Windows ` | Set-AzureSubnet -SubnetNames $subnet | Set-AzureStaticVNetIP -IPAddress $ipaddr
    
    #Reload disk config
    $diskobjects  = Import-CSV $file
    $datadiskimport = $diskobjects | where {$_.lun -ne "Windows"}
    
    ForEach ( $attachdatadisk in $datadiskimport) 
       {
    $label = $attachdatadisk.disklabel
    $lunNo = $attachdatadisk.lun
    $hostcach = $attachdatadisk.hostcaching
    $datadiskforbuild = $attachdatadisk.diskName
    $vhdname = $attachdatadisk.vhdname
    
    ###This is different to just a straight cloud service change
    #note if you do not have a disk label the command below will fail, populate as required.
    $vmConfig | Add-AzureDataDisk -ImportFrom -MediaLocation "https://$newxiostorageaccountnamenode2.blob.core.windows.net/vhds/$vhdname" -LUN $lunNo -HostCaching $hostcach -DiskLabel $label
    
    }
    
    #Create VM
    $vmConfig  | New-AzureVM –ServiceName $destcloudsvc –Location $location -VNetName $vnet -Verbose

#### Step 22: Add Load Balanced Endpoints and ACLs
    #Endpoints
    $epname="sqlIntEP"
    $prot="tcp"
    $locport=1433
    $pubport=1433
    Get-AzureVM –ServiceName $destcloudsvc –Name $vmNameToMigrate  | Add-AzureEndpoint -Name $epname -Protocol $prot -LocalPort $locport -PublicPort $pubport -ProbePort 59999 -ProbeIntervalInSeconds 5 -ProbeTimeoutInSeconds 11  -ProbeProtocol "TCP" -InternalLoadBalancerName $ilb -LBSetName $ilb -DirectServerReturn $true | Update-AzureVM
    
    
    #STOP!!! CHECK in portal or Machine Endpoints through powershell that these Endpoints are created!
    
    #SET ACLs or Azure Network Security Groups & Windows FWs 
     
    #http://msdn.microsoft.com/en-us/library/azure/dn495192.aspx

#### Step 23: Test Failover

You should now let the migrated node synchronise with the on-premise AlwaysOn node, place it in to synchronous replication mode and wait until it is synchronised. Then failover from on-prem to the first node migrated, which is the AFP. Once that has worked, change the last migrated node to the AFP.

You should test failovers between all nodes and run though chaos tests to ensure failovers work as expected and in a timely manor.

#### Step 24: Put back cluster quorum settings / DNS TTL / Failover Pntrs / Sync Settings 
##### Adding IP Address Resource on Same Subnet

If you have only 2 SQL Servers and want to migrate them to a new cloud service, but want to keep them on the same subnet, you can avoid taking the listener offline to delete the original AlwaysOn IP Address and add the New IP Address. If you are migrating the VMs to another subnet you will not need to do this as there will be an additional cluster network that will reference that subnet.

Once you have brought up the migrated secondary and added in the new IP Address resource for the new cloud service before failover the existing Primary, you should take these steps within the Cluster Failover Manager:

To add in IP Address, see the [Appendix](#appendix-migrating-a-multisite-alwayson-cluster-to-premium-storage), step 14.

1. For the current IP Address resource change the possible owner to ‘Existing Primary SQL Server’, in the example below, ‘dansqlams4’:

	![Appendix13][23]

1. For the new IP Address resource  change the possible owner to ‘Migrated secondary SQL Server’, in the example below, ‘dansqlams5’:

	![Appendix14][24]

1. Once this is set you can failover, and when the last node is migrated the Possible Owners should be edited so that node is added as a Possible Owner:

	![Appendix15][25]

## Additional Resources
- [Azure Premium Storage](storage-premium-storage-preview-portal.md)
- [Virtual Machines](http://azure.microsoft.com/services/virtual-machines/)
- [SQL Server in Azure Virtual Machines](virtual-machines-sql-server-infrastructure-services.md)

<!-- IMAGES -->
[1]: ./media/virtual-machines-sql-server-use-premium-storage/1_VNET_Portal.png
[2]: ./media/virtual-machines-sql-server-use-premium-storage/2_Diskname_Lun.png
[3]: ./media/virtual-machines-sql-server-use-premium-storage/3_Virtual_Disk_Properties.png
[4]: ./media/virtual-machines-sql-server-use-premium-storage/4_Virtual_Disk_Properties_Details.png
[5]: ./media/virtual-machines-sql-server-use-premium-storage/5_Get_Storage_Pool.png
[6]: ./media/virtual-machines-sql-server-use-premium-storage/6_Deployments_Always_On.png
[7]: ./media/virtual-machines-sql-server-use-premium-storage/7_Add_More_Secondaries.png
[8]: ./media/virtual-machines-sql-server-use-premium-storage/8_Use_Existing_Secondary_Single_Site.png
[9]: ./media/virtual-machines-sql-server-use-premium-storage/9_Use_Existing_Secondary_Multi_Site.png
[10]: ./media/virtual-machines-sql-server-use-premium-storage/9_Use_Existing_Secondary_Multi_Site_b.png
[11]: ./media/virtual-machines-sql-server-use-premium-storage/10_Appendix_01.png
[12]: ./media/virtual-machines-sql-server-use-premium-storage/10_Appendix_02.png
[13]: ./media/virtual-machines-sql-server-use-premium-storage/10_Appendix_03.png
[14]: ./media/virtual-machines-sql-server-use-premium-storage/10_Appendix_04.png
[15]: ./media/virtual-machines-sql-server-use-premium-storage/10_Appendix_05.png
[16]: ./media/virtual-machines-sql-server-use-premium-storage/10_Appendix_06.png
[17]: ./media/virtual-machines-sql-server-use-premium-storage/10_Appendix_07.png
[18]: ./media/virtual-machines-sql-server-use-premium-storage/10_Appendix_08.png
[19]: ./media/virtual-machines-sql-server-use-premium-storage/10_Appendix_09.png
[20]: ./media/virtual-machines-sql-server-use-premium-storage/10_Appendix_10.png
[21]: ./media/virtual-machines-sql-server-use-premium-storage/10_Appendix_11.png
[22]: ./media/virtual-machines-sql-server-use-premium-storage/10_Appendix_12.png
[23]: ./media/virtual-machines-sql-server-use-premium-storage/10_Appendix_13.png
[24]: ./media/virtual-machines-sql-server-use-premium-storage/10_Appendix_14.png
[25]: ./media/virtual-machines-sql-server-use-premium-storage/10_Appendix_15.png
