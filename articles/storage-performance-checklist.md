<properties 
	pageTitle="Microsoft Azure Storage Performance and Scalability Checklist" 
	description="A checklist of proven practices for use with Azure Storage in developing performant applications." 
	services="storage" 
	documentationCenter="" 
	authors="tamram" 
	manager="adinah" 
	editor=""/>

<tags 
	ms.service="storage" 
	ms.workload="storage" 
	ms.tgt_pltfrm="na" 
	ms.devlang="na" 
	ms.topic="article" 
	ms.date="02/26/2015" 
	ms.author="tamram"/>

# Microsoft Azure Storage Performance and Scalability Checklist

## Overview
Since the release of the Microsoft Azure Storage services, Microsoft has developed a number of proven practices for using these services in a performant manner, and this article serves to consolidate the most important of them into a checklist-style list. The intention of this article is to help application developers verify they are using proven practices with Azure Storage and to help them identify other proven practices they should consider adopting. This article does not attempt to cover every possible performance and scalability optimization — it excludes those that are small in their impact or not broadly applicable. To the extent that the application’s behavior can be predicted during design, it’s useful to keep these in mind early on to avoid designs that will run into performance problems.  

Every application developer using Azure Storage should take the time to read this article, and check that his or her application follows each of the proven practices listed below.  

## Checklist
This article organizes the proven practices into the following groups. Proven practices applicable to:  

-	All Azure Storage services (blobs, tables, queues, and files)
-	Blobs
-	Tables
-	Queues  

The Azure Files service is currently in Preview, and we will add proven practices here in the future.  

|Done|	Area|	Category|	Question
|----|------|-----------|-----------
||All Services|	Scalability Targets|[Is your application designed to avoid approaching the scalability targets?](#subheading1)
||All Services|	Networking|	[Do client side devices have sufficiently high bandwidth and low latency to achieve the performance needed?](#subheading2)
||All Services|	Networking|	[Do client side devices have a high enough quality link?](#subheading3)
||All Services|	Networking|	[Is the client application located "near" the storage account?](#subheading4)
||All Services|	Content Distribution|	[Are you using a CDN for content distribution?](#subheading5)
||All Services|	Direct Client Access|	[Are you using SAS and CORS to allow direct access to storage instead of proxy?](#subheading6)
||All Services|	Caching|	[Is your application caching data that is repeatedly used and changes rarely?](#subheading7)
||All Services|	Caching|	[Is your application batching updates (caching them client side and then uploading in larger sets)?](#subheading8)
||All Services|	.NET Configuration|	[Have you configured your client to use a sufficient number of concurrent connections?](#subheading9)
||All Services|	.NET Configuration|	[Have you configured .NET to use a sufficient number of threads?](#subheading10)
||All Services|	.NET Configuration|	[Are you using .NET 4.5 or later, which has improved garbage collection?](#subheading11)
||All Services|	Parallelism|	[Have you ensured that parallelism is bounded appropriately so that you don’t overload either your client capabilities or the scalability targets?](#subheading12)
||All Services|	Tools|	[Are you using the latest version of Microsoft provided client libraries and tools?](#subheading13)
||All Services|	Retries|	[Are you using an exponential backoff retry policy for throttling errors and timeouts?](#subheading14)
||All Services|	Retries|	[Is your application avoiding retries for non-retryable errors?](#subheading15)
||Blobs|	Scalability Targets|	[Is your application staying within the bandwidth or operations scalability target for a single blob?](#subheading16)
||Blobs|	Copying Blobs|	[Are you copying blobs in an efficient manner?](#subheading17)
||Blobs|	Copying Blobs|	[Are you using AzCopy for bulk copies of blobs?](#subheading18)
||Blobs|	Copying Blobs|	[Are you using Azure Import/Export to transfer very large volumes of data?](#subheading19)
||Blobs|	Use Metadata|	[Are you storing frequently used metadata about blobs in their metadata?](#subheading20)
||Blobs|	Uploading Fast|	[When trying to upload one blob quickly, are you uploading blocks in parallel?](#subheading21)
||Blobs|	Uploading Fast|	[When trying to upload many blobs quickly, are you uploading blobs in parallel?](#subheading22)
||Blobs|	Correct Blob Type|	[Are you using page blobs or block blobs when appropriate?](#subheading23)
||Tables|	Scalability Targets|	[Are you approaching the scalability targets for entities per second?](#subheading24)
||Tables|	Configuration|	[Are you using JSON for your table requests?](#subheading25)
||Tables|	Configuration|	[Have you turned Nagle off to improve the performance of small requests?](#subheading26)
||Tables|	Tables and Partitions|	[Have you properly partitioned your data?](#subheading27)
||Tables|	Hot Partitions|	[Are you avoiding append-only and prepend-only patterns?](#subheading28)
||Tables|	Hot Partitions|	[Are your inserts/updates spread across many partitions?](#subheading29)  
||Tables|	Query Scope|	[Have you designed your schema to allow for point queries to be used in most cases, and table queries to be used sparingly?](#subheading30)
||Tables|	Query Density|	[Do your queries typically only scan and return rows that your application will use?](#subheading31)
||Tables|	Limiting Returned Data|	[Are you using filtering to avoid returning entities that are not needed?](#subheading32)
||Tables|	Limiting Returned Data|	[Are you using projection to avoid returning properties that are not needed?](#subheading33)
||Tables|	Denormalization|	[Have you denormalized your data such that you avoid inefficient queries or multiple read requests when trying to get data?](#subheading34)
||Tables|	Insert/Update/Delete|	[Are you batching requests that need to be transactional or can be done at the same time to reduce round-trips?](#subheading35)
||Tables|	Insert/Update/Delete|	[Are you avoiding retrieving an entity just to determine whether to call insert or update?](#subheading36)
||Tables|	Insert/Update/Delete|	[Have you considered storing series of data that will frequently be retrieved together in a single entity as properties instead of multiple entities?](#subheading37)
||Tables|	Insert/Update/Delete|	[For entities that will always be retrieved together and can be written in batches (e.g. time series data), have you considered using blobs instead of tables?](#subheading38)
||Queues|	Scalability Targets|	[Are you approaching the scalability targets for messages per second?](#subheading39)
||Queues|	Configuration|	[Have you turned Nagle off to improve the performance of small requests?](#subheading40)
||Queues|	Message Size|	[Are your messages compact to improve the performance of the queue?](#subheading41)
||Queues|	Bulk Retrieve|	[Are you retrieving multiple messages in a single "Get" operation?](#subheading41)
||Queues|	Polling Frequency|	[Are you polling frequently enough to reduce the perceived latency of your application?](#subheading42)
||Queues|	Update Message|	[Are you using UpdateMessage to store progress in processing messages, avoiding having to reprocess the entire message if an error occurs?](#subheading43)
||Queues|	Architecture|	[Are you using queues to make your entire application more scalable by keeping long-running workloads out of the critical path and scale then independently?](#subheading44)


##<a name="allservices"></a>All Services
This section lists proven practices that are applicable to the use of any of the Azure Storage services (blobs, tables, queues, or files).  

###<a name="subheading1"></a>Scalability Targets
Each of the Azure Storage services has scalability targets for capacity (GB), transaction rate, and bandwidth. If your application approaches or exceeds any of the scalability targets, it may encounter increased transaction latencies or throttling. When a Storage service throttles your application, the service begins to return “503 Server busy” or “500 Operation timeout” error codes for some storage transactions. This section discusses both the general approach to dealing with scalability targets and bandwidth scalability targets in particular. Later sections that deal with individual storage services discuss scalability targets in the context of that specific service:  

-	[Blob bandwidth and requests per second](#subheading16)
-	[Table entities per second](#subheading24)
-	[Queue messages per second](#subheading39)  

####<a name="sub1bandwidth"></a>Bandwidth Scalability Target for All Services
At the time of writing, the bandwidth targets in the US for a geo-redundant storage (GRS) account are 10 gigabits per second (Gbps) for ingress (data sent to the storage account) and 20 Gbps for egress (data sent from the storage account). For a locally redundant storage (LRS) account, the limits are higher – 20 Gbps for ingress and 30 Gbps for egress.  International bandwidth limits may be lower and can be found on our [scalability targets page](http://msdn.microsoft.com/library/azure/dn249410.aspx).  For more information on the storage redundancy options, see the links in [Useful Resources](#sub1useful) below.  

####What to do when approaching a scalability target
If your application is approaching the scalability targets for a single storage account, consider adopting one of the following approaches:  

-	Reconsider the workload that causes your application to approach or exceed the scalability target. Can you design it differently to use less bandwidth or capacity, or fewer transactions?
-	If an application must exceed one of the scalability targets, you should create multiple storage accounts and partition your application data across those multiple storage accounts. If you use this pattern, then be sure to design your application so that you can add more storage accounts in the future for load balancing. At time of writing, each Azure subscription can have up to 100 storage accounts.  Storage accounts also have no cost other than your usage in terms of data stored, transactions made, or data transferred.
-	If your application hits the bandwidth targets, consider compressing data in the client to reduce the bandwidth required to send the data to the storage service.  Note that while this may save bandwidth and improve network performance, it can also have some negative impacts.  You should evaluate the performance impact of this due to the additional processing requirements for compressing and decompressing data in the client. In addition, storing compressed data can make it more difficult to troubleshoot issues since it could be more difficult to view stored data using standard tools.
-	If your application hits the scalability targets, then ensure that you are using an exponential backoff for retries (see [Retries](#subheading14)).  It’s better to make sure you never approach the scalability targets (by using one of the above methods), but this will ensure your application won’t just keep retrying rapidly, making the throttling worse.  

####Useful Resources
The following links provide additional detail on scalability targets:
-	You can view the current scalability targets on the page [Azure Storage Scalability and Performance Targets on MSDN](http://msdn.microsoft.com/library/azure/dn249410.aspx).
-	You can learn more about storage redundancy options in the blog post [Azure Storage Redundancy Options and Read Access Geo Redundant Storage](http://blogs.msdn.com/b/windowsazurestorage/archive/2013/12/11/introducing-read-access-geo-replicated-storage-ra-grs-for-windows-azure-storage.aspx)– see the next section for more details.
-	For current information about pricing for Azure services, see [Azure pricing at-a-glance](http://azure.microsoft.com/pricing/overview/).  

###Networking
While the API calls matter, often the physical network constraints of the application have a significant impact on performance. The following describe some of limitations users may encounter.  

####Client Network Capability
#####<a name="subheading2"></a>Throughput
For bandwidth, the problem is often the capabilities of the client. For example, while a single storage account can handle 10 Gbps or more of ingress (see [bandwidth scalability targets](#sub1bandwidth)), the network speed in a “Small” Azure Worker Role instance is only capable of approximately 100 Mbps. Larger Azure instances have NICs with greater capacity, so you should consider using a larger instance or more VM’s if you need higher network limits from a single machine. If you are accessing a Storage service from an on premises application, then the same rule applies: understand the network capabilities of the client device and the network connectivity to the Azure Storage location and either improve them as needed or design your application to work within their capabilities.  

#####<a name="subheading3"></a>Link Quality
As with any network usage, be aware that network conditions resulting in errors and packet loss will slow effective throughput.  Using WireShark or NetMon may help in diagnosing this issue.  

#####Useful Resources
For more information about virtual machine sizes and allocated bandwidth, see [Best Practices for the Design of Large-Scale Services on Azure Cloud Services](http://msdn.microsoft.com/library/azure/dn197896.aspx) on MSDN.  

####<a name="subheading4"></a>Location
In any distributed environment, placing the client near to the server delivers in the best performance. For accessing Azure Storage with the lowest latency, the best location for your client is within the same Azure region. For example, if you have an Azure Web Site that uses Azure Storage, you should locate them both within a single region (for example, US West or Asia Southeast). This reduces the latency and the cost — at the time of writing, bandwidth usage within a single region is free.  

If your client applications are not hosted within Azure (such as mobile device apps or on premises enterprise services), then again placing the storage account in a region near to the devices that will access it, will generally reduce latency. If your clients are broadly distributed (for example, some in North America, and some in Europe), then you should consider using multiple storage accounts: one located in a North American region and one in a European region. This will help to reduce latency for users in both regions. This approach is usually easier to implement if the data the application stores is specific to individual users, and does not require replicating data between storage accounts.  For broad content distribution, a CDN is recommended – see the next section for more details.  

###<a name="subheading5"></a>Content Distribution
Sometimes, an application needs to serve the same content to many users (e.g. a product demo video used in the home page of a website), located in either the same or multiple regions. In this scenario, you should use a Content Delivery Network (CDN) such as Azure CDN, and the CDN would use Azure storage as the origin of the data. Unlike an Azure Storage account that exists in a single region and that cannot deliver content with low latency to other regions, Azure CDN uses servers in multiple data centers around the world. Additionally, a CDN can typically supports much higher egress limits than a single storage account.  

For more information about Azure CDN, see [Azure CDN](http://azure.microsoft.com/services/cdn/).  

###<a name="subheading6"></a>Using SAS and CORS
When you need to authorize code such as JavaScript in a user’s web browser or a mobile phone app to access data in Azure Storage, one approach is to use an application in web role as a proxy: the user’s device authenticates with the web role, which in turn authenticates with the storage service. In this way, you can avoid exposing your storage account keys on insecure devices. However, this places a big overhead on the web role because all the data transferred between the user’s device and the storage service must pass through the web role. You can avoid using a web role as a proxy for the storage service by using Shared Access Signatures (SAS), sometimes in conjunction with Cross-Origin Resource Sharing headers (CORS). Using SAS, you can allow your user’s device to make requests directly to a storage service by means of a limited access token. For example, if a user wants to upload a photo to your application, your web role can generate and send to the user’s device a SAS token that grants permission to write to a specific blob or container for the next 30 minutes (after which the SAS token expires).   

Normally, a browser will not allow JavaScript in a page hosted by a website on one domain to perform specific operations such as a “PUT” to another domain. For example, if you host a web role at “contosomarketing.cloudapp.net,” and want to use client side JavaScript to upload a blob to your storage account at “contosoproducts.blob.core.windows.net,” the browser’s “same origin policy” will forbid this operation. CORS is a browser feature that allows the target domain (in this case the storage account) to communicate to the browser that it trusts requests originating in the source domain (in this case the web role).  

Both of these technologies can help you avoid unnecessary load (and bottlenecks) on your web application.  

####Useful Resources
For more information about SAS, see [Shared Access Signatures, Part 1: Understanding the SAS Model](../storage-dotnet-shared-access-signature-part-1/).  

For more information about CORS, see [Cross-Origin Resource Sharing (CORS) Support for the Azure Storage Services on MSDN](http://msdn.microsoft.com/library/azure/dn535601.aspx).  

###Caching
####<a name="subheading7"></a>Getting Data
In general, getting data from a service once is better than getting it twice. Consider the example of an MVC web application running in a web role that has already retrieved a 50MB blob from the storage service to serve as content to a user. The application could then retrieve that same blob every time a user requests it, or it could cache it locally to disk and reuse the cached version for subsequent user requests. Furthermore, whenever a user requests the data, the application could issue GET with a conditional header for modification time, which would avoid getting the entire blob if it hasn’t been modified. You can apply this same pattern to working with table entities.  

In some cases, you may decide that your application can assume that the blob remains valid for a short period after retrieving it, and that during this period the application does not need to check if the blob was modified. 

Configuration, lookup, and other data that are always used by the application are great candidates for caching.  

For an example of how to get a blob’s properties to discover the last modified date using .NET, see [Set and Retrieve Properties and Metadata](http://msdn.microsoft.com/library/azure/hh225342.aspx) on MSDN. For more information about conditional downloads, see [Conditionally Refresh a Local Copy of a Blob](http://msdn.microsoft.com/library/azure/dd179371.aspx) on MSDN.  

####<a name="subheading8"></a>Uploading Data in Batches
In some application scenarios, you can aggregate data locally, and then periodically upload it in a batch instead of uploading each piece of data immediately. For example, a web application might keep a log file of activities: the application could either upload details of every activity as it happens as a table entity (which requires many storage operations), or it could save activity details to a local log file, and then periodically upload all activity details as a delimited file to a blob. If each log entry is 1KB in size, you can upload thousands in a single “Put Blob” transaction (you can upload a blob of up to 64MB in size in a single transaction). Of course, if the local machine crashes prior to the upload, you will potentially lose some log data: the application developer must design for the possibility of client device or upload failures.  If the activity data needs to be downloaded for timespans (not just single activity), then blobs are recommended over tables. 

###.NET Configuration
If using the .NET Framework, this section lists several quick configuration settings that you can use to make significant performance improvements.  If using other languages, check to see if similar concepts apply in your chosen language.  

####<a name="subheading9"></a>Increase default connection limit
In .NET, the following code increases the default connection limit (which is usually 2 in a client environment or 10 in a server environment) to 100. Typically, you should set the value to approximately the number of threads used by your application.  

	ServicePointManager.DefaultConnectionLimit = 100; //(Or More)  

You must set the connection limit before opening any connections.  

For other programming languages, see that language’s documentation to determine how to set the connection limit.  

For additional information, see the post [Web Services: Concurrent Connections](http://blogs.msdn.com/b/darrenj/archive/2005/03/07/386655.aspx) on MSDN.  

####<a name="subheading10"></a>Increase ThreadPool Min Threads if using synchronous code with Async Tasks
This code will increase the thread pool min threads:  

	ThreadPool.SetMinThreads(100,100); //(Determine the right number for your application)  

For more information, see [ThreadPool.SetMinThreads Method](http://msdn.microsoft.com/library/system.threading.threadpool.setminthreads(v=vs.110).aspx) on MSDN.  

####<a name="subheading11"></a>Take advantage of .NET 4.5 Garbage Collection
Use .NET 4.5 or later for the client application to take advantage of performance improvements in server garbage collection.   

For more information, see the article [An Overview of Performance Improvements in .NET 4.5](http://msdn.microsoft.com/magazine/hh882452.aspx) on MSDN.  

###<a name="subheading12"></a>Unbounded Parallelism
While parallelism can be great for performance, be careful about using unbounded parallelism (no limit on the number of threads and/or parallel requests) to upload or download data, using multiple workers to access multiple partitions (containers, queues, or table partitions) in the same storage account or to access multiple items in the same partition. If the parallelism is unbounded, your application can exceed the client device’s capabilities or the storage account’s scalability targets resulting in longer latencies and throttling.  

###<a name="subheading13"></a>Storage Client Libraries and Tools
Always use the latest Microsoft provided client libraries and tools. At the time of writing, there are RTM libraries available for .NET, Windows Phone, Windows Runtime, and Java, as well as preview libraries in other languages like node.js and C++. In addition, Microsoft has released PowerShell cmdlets and cross-platform command-line tools developed in Node.js for working with Azure Storage. Microsoft actively develops these tools with performance in mind, keeps them up to date with the latest service versions, and ensures they handle many of the proven performance practices internally.  

###Retries
####<a name="subheading14"></a>Throttling/ServerBusy
In some cases, the storage service may throttle your application or may simply be unable to serve the request due to some transient condition and return a “503 Server busy” message or “500 Timeout”.  This can happen if your application is approaching any of the scalability targets, or if the system is rebalancing your partitioned data to allow for higher throughput.  The client application should typically retry the operation that causes such an error: attempting the same request later can succeed. However, if the storage service is throttling your application because it is exceeding scalability targets, or even if the service was unable to serve the request for some other reason, aggressive retries usually make the problem worse. For this reason, you should use an exponential back off (the client libraries default to this behavior). For example, your application may retry after 2 seconds, then 4 seconds, then 10 seconds, then 30 seconds, and then give up completely. This behavior results in your application significantly reducing its load on the service rather than exacerbating any problems.  

Note that connectivity errors can be retried immediately, because they are not the result of throttling and are expected to be transient.  

####<a name="subheading15"></a>Non-Retryable Errors
The client libraries are aware of which errors are retry-able and which are not. However, if you are writing your own code against the storage REST API, remember there are some errors that you should not retry: for example, a 400 (Bad Request) response indicates that the client application sent a request that could not be processed because it was not in an expected form. Resending this request will result the same response every time, so there is no point in retrying it. If you are writing your own code against the storage REST API, be aware of what the error codes mean and the proper way to retry (or not) for each of them.  

####Useful Resources
For more information about storage error codes, see [Status and Error Codes](http://msdn.microsoft.com/library/azure/dd179382.aspx) on the Microsoft Azure web site.  

##Blobs
In addition to the proven practices for [All Services](#allservices) described previously, the following proven practices apply specifically to the blob service.  

###Blob-Specific Scalability Targets
####<a name="subheading16"></a>Bandwidth and operations per Blob
You can read or write to a single blob at up to a maximum of 60 MB/second (this is approximately 480 Mbps which exceeds the capabilities of many client side networks (including the physical NIC on the client device). In addition, a single blob supports up to 500 requests per second. If you have multiple clients that need to read the same blob and you might exceed these limits, you should consider using a CDN for distributing the blob.  

For more information about target throughput for blobs, see [Azure Storage Scalability and Performance Targets](http://msdn.microsoft.com/library/azure/dn249410.aspx) on MSDN.  

###Copying and Moving Blobs
####<a name="subheading17"></a>Copy Blob
The storage REST API version 2012-02-12 introduced the useful ability to copy blobs across accounts: a client application can instruct the storage service to copy a blob from another source (possibly in a different storage account), and then let the service perform the copy asynchronously. This can significantly reduce the bandwidth needed for the application when you are migrating data from other storage accounts because you do not need to download and upload the data.  

One consideration, however, is that, when copying between storage accounts, there is no time guarantee on when the copy will complete. If your application needs to complete a blob copy quickly under your control, it may be better to copy the blob by downloading it to a VM and then uploading it to the destination.  For full predictability in that situation, ensure that the copy is performed by a VM running in the same Azure region, or else network conditions may (and probably will) affect your copy performance.  In addition, you can monitor the progress of an asynchronous copy programmatically.  
 
Note that copies within the same storage account itself are generally completed quickly.  

For more information, see [Copy Blob on MSDN](http://msdn.microsoft.com/library/azure/dd894037.aspx).  

####<a name="subheading18"></a>Use AzCopy
The Azure Storage team has released a command line tool “AzCopy” that is meant to help with bulk transferring many blobs to, from, and across storage accounts.  This tool is optimized for this scenario, and can achieve high transfer rates.  Its use is encouraged for bulk upload, download, and copy scenarios.  You can learn more about it and download it [here](storage-use-azcopy.md).  

####<a name="subheading19"></a>Azure Import/Export Service
For very large volumes of data (more than 1TB), the Azure Storage offers the Import/Export service, which allows for uploading and downloading from blob storage by shipping hard drives.  You can put your data on a hard drive and send it to Microsoft for upload, or send a blank hard drive to Microsoft to download data.  You can read more about it [here](storage-import-export-service.md).  This can be much more efficient than uploading/downloading this volume of data over the network.  

###<a name="subheading20"></a>Use metadata
The blob service supports head requests, which can include metadata about the blob. For example, if your application needed the EXIF data out of a photo, it could retrieve the photo and extract it. To save bandwidth and improve performance, your application could store the EXIF data in the blob’s metadata when the application uploaded the photo: you can then retrieve the EXIF data in metadata using only a HEAD request, saving significant bandwidth and the processing time needed to extract the EXIF data each time the blob is read. This would be useful in scenarios where you only need the metadata, and not the full content of a blob.  Note that only 8KB of metadata can be stored per blob (the service will not accept a request to store more than that), so if the data does not fit in that size, you may not be able to use this approach.  

For an example of how to get a blob’s metadata using .NET, see [Set and Retrieve Properties and Metadata](http://msdn.microsoft.com/library/azure/hh225342.aspx) on MSDN.  

###Uploading Fast
To upload blobs fast, the first question to answer is: are you uploading one blob or many?  Use the below guidance to determine the correct method to use depending on your scenario.  

####<a name="subheading21"></a>Uploading one large blob quickly
To upload a single large blob quickly, your client application should upload its blocks or pages in parallel (being mindful of the scalability targets for individual blobs and the storage account as a whole).  Note that the official Microsoft-provided RTM Storage Client libraries (.NET, Java) have the ability to do this.  For each of the libraries, use the below specified object/property to set the level of concurrency:  

-	.NET: Set ParallelOperationThreadCount on a BlobRequestOptions object to be used.
-	Java/Android: Use BlobRequestOptions.setConcurrentRequestCount()
-	Node.js: Use parallelOperationThreadCount on either the request options or on the blob service.
-	C++: Use the blob_request_options::set_parallelism_factor method.   

####<a name="subheading22"></a>Uploading many blobs quickly
To upload many blobs quickly, upload blobs in parallel. This is faster than uploading single blobs at a time with parallel block uploads because it spreads the upload across multiple partitions of the storage service. A single blob only supports a throughput of 60 MB/second (approximately 480 Mbps). At the time of writing, a US based LRS account supports up to 20 Gbps ingress which is far more than the throughput supported by an individual blob.  [AzCopy](#subheading18) performs uploads in parallel by default, and is recommended for this scenario.  

###<a name="subheading23"></a>Choosing the correct type of blob
Azure Storage supports two types of blob: *page* blobs and *block* blobs. For a given usage scenario, your choice of blob type will affect the performance and scalability of your solution. Block blobs are appropriate when you want to upload large amounts of data efficiently: for example, a client application may need to upload photos or video to blob storage. Page blobs are appropriate if the application needs to perform random writes on the data: for example, Azure VHDs are stored as page blobs.  

For more information, see [Understanding Block Blobs and Page Blobs](http://msdn.microsoft.com/library/azure/ee691964.aspx) on MSDN.  

##Tables
In addition to the proven practices for [All Services](#allservices) described previously, the following proven practices apply specifically to the table service.  

###<a name="subheading24"></a>Table-Specific Scalability Targets
In addition to the bandwidth limitations of an entire storage account, tables have the following specific scalability limit.  Note that the system will load balance as your traffic increases, but if your traffic has sudden bursts, you may not be able to get this volume of throughput immediately.  If your pattern has bursts, you should expect to see throttling and/or timeouts during the burst as the storage service automatically load balances out your table.  Ramping up slowly generally has better results as it gives the system time to load balance appropriately.  

####Entities per Second (Account)
The scalability limit for accessing tables is up to 20,000 entities (1KB each) per second for an account.  In general, each entity that is inserted, updated, deleted, or scanned counts toward this target.  So a batch insert that contains 100 entities would count as 100 entities.  A query that scans 1000 entities and returns 5 would count as 1000 entities.  

####Entities per Second (Partition)
Within a single partition, the scalability target for accessing tables is 2,000 entities (1KB each) per second, using the same counting as described in the previous section.  

###Configuration
This section lists several quick configuration settings that you can use to make significant performance improvements in the table service:  

####<a name="subheading25"></a>Use JSON
Beginning with storage service version 2013-08-15, the table service supports using JSON instead of the XML-based AtomPub format for transferring table data. This can reduce payload sizes by as much as 75% and can significantly improve the performance of your application.   

For more information, see the post [Microsoft Azure Tables: Introducing JSON](http://blogs.msdn.com/b/windowsazurestorage/archive/2013/12/05/windows-azure-tables-introducing-json.aspx) and [Payload Format for Table Service Operations](http://msdn.microsoft.com/library/azure/dn535600.aspx) on MSDN. 

####<a name="subheading26"></a>Nagle Off
Nagle’s algorithm is widely implemented across TCP/IP networks as a means to improve network performance. However, it is not optimal in all circumstances (such as highly interactive environments). For Azure Storage, Nagle’s algorithm has a negative impact on the performance of requests to the table and queue services, and you should disable it if possible.  

For more information, see our blog post [Nagle’s Algorithm is Not Friendly towards Small Requests](http://blogs.msdn.com/b/windowsazurestorage/archive/2010/06/25/nagle-s-algorithm-is-not-friendly-towards-small-requests.aspx), which explains why Nagle’s algorithm interacts poorly with table and queue requests, and shows how to disable it in your client application.  

###Schema
How you represent and query your data is the biggest single factor that affects the performance of the table service. While every application is different, this section outlines some general proven practices that relate to:  

-	Table design
-	Efficient queries
-	Efficient data updates  

####<a name="subheading27"></a>Tables and partitions
Tables are divided into partitions. Every entity stored in a partition shares the same partition key and has a unique row key to identify it within that partition. Partitions provide benefits but also introduce scalability limits.  

-	Benefits: You can update entities in the same partition in a single, atomic, batch transaction that contains up to 100 separate storage operations (limit of 4MB total size). Assuming the same number of entities to be retrieved, you can also query data within a single partition more efficiently than data that spans partitions (though read on for further recommendations on querying table data).
-	Scalability limit: Access to entities stored in a single partition cannot be load-balanced because partitions support atomic batch transactions. For this reason, the scalability target for an individual table partition is lower than for the table service as a whole.  

Because of these characteristics of tables and partitions, you should adopt the following design principles:  

-	Data that your client application frequently updated or queried in the same logical unit of work should be located in the same partition.  This may be because your application is aggregating writes, or because you want to take advantage of atomic batch operations.  Also, data in a single partition can be more efficiently queried in a single query than data across partitions.
-	Data that your client application does not insert/update or query in the same logical unit of work (single query or batch update) should be located in separate partitions.  One important note is that there is no limit to the number of partition keys in a single table, so having millions of partition keys is not a problem and will not impact performance.  For example, if your application is a popular website with user login, using the User Id as the partition key could be a good choice.  

####Hot Partitions
A hot partition is one that is receiving a disproportionate percentage of the traffic to an account, and cannot be load balanced because it is a single partition.  In general, hot partitions are created one of two ways:  

#####<a name="subheading28"></a>Append Only and Prepend Only patterns
The “Append Only” pattern is one where all (or nearly all) of the traffic to a given PK increases and decreases according to the current time.  An example is if your application used the current date as a partition key for log data.  This results in all of the inserts going to the last partition in your table, and the system cannot load balance because all of the writes are going to the end of your table.  If the volume of traffic to that partition exceeds the partition-level scalability target, then it will result in throttling.  It’s better to ensure that traffic is sent to multiple partitions, to enable load balance the requests across your table.  

#####<a name="subheading29"></a>High-Traffic Data
If your partitioning scheme results in a single partition that just has data that is far more used than other partitions, you may also see throttling as that partition approaches the scalability target for a single partition.  It’s better to make sure that your partition scheme results in no single partition approaching the scalability targets.  

####Querying
This section describes proven practices for querying the table service.  

#####<a name="subheading30"></a>Query Scope
There are several ways to specify the range of entities to query.  The following is a discussion of the uses of each.  

In general, avoid scans (queries larger than a single entity), but if you must scan, try to organize your data so that your scans retrieve the data you need without scanning or returning significant amounts of entities you don’t need.  

######Point Queries
A point query retrieves exactly one entity. It does this by specifying both the partition key and row key of the entity to retrieve. These queries are very efficient, and you should use them wherever possible.  

######Partition Queries
A partition query is a query that retrieves a set of data that shares a common partition key. Typically, the query specifies a range of row key values or a range of values for some entity property in addition to a partition key. These are less efficient than point queries, and should be used sparingly.  

######Table Queries
A table query is a query that retrieves a set of entities that does not share a common partition key. These queries are not efficient and you should avoid them if possible.  

#####<a name="subheading31"></a>Query Density
Another key factor in query efficiency is the number of entities returned as compared to the number of entities scanned to find the returned set. If your application performs a table query with a filter for a property value that only 1% of the data shares, the query will scan 100 entities for every one entity it returns. The table scalability targets discussed previously all relate to the number of entities scanned, and not the number of entities returned: a low query density can easily cause the table service to throttle your application because it must scan so many entities to retrieve the entity you are looking for.  See the section below on [denormalization](#subheading34) for more information on how to avoid this.  

#####Limiting the Amount of Data Returned
######<a name="subheading32"></a>Filtering
Where you know that a query will return entities that you don’t need in the client application, consider using a filter to reduce the size of the returned set. While the entities not returned to the client still count toward the scalability limits, your application performance will improve because of the reduced network payload size and the reduced number of entities that your client application must process.  See above note on [Query Density](#subheading31), however – the scalability targets relate to the number of entities scanned, so a query that filters out many entities may still result in throttling, even if few entities are returned.  

######<a name="subheading33"></a>Projection
If your client application needs only a limited set of properties from the entities in your table, you can use projection to limit the size of the returned data set. As with filtering, this helps to reduce network load and client processing.  

#####<a name="subheading34"></a>Denormalization
Unlike working with relational databases, the proven practices for efficiently querying table data lead to denormalizing your data. That is, duplicating the same data in multiple entities (one for each key you may use to find the data) to minimize the number of entities that a query must scan to find the data the client needs, rather than having to scan large numbers of entities to find the data your application needs.  For example, in an e-commerce website, you may want to find an order both by the customer ID (give me this customer’s orders) and by the date (give me orders on a date).  In Table Storage, it is best to store the entity (or a reference to it) twice – once with Table Name, PK, and RK to facilitate finding by customer ID, once to facilitate finding it by the date.  

####Insert/Update/Delete
This section describes proven practices for modifying entities stored in the table service.  

#####<a name="subheading35"></a>Batching
Batch transactions are known as Entity Group Transactions (ETG) in Azure Storage; all the operations within an ETG must be on a single partition in a single table. Where possible, use ETGs to perform inserts, updates, and deletes in batches. This reduces the number of round trips from your client application to the server, reduces the number of billable transaction (an ETG counts as a single transaction for billing purposes and can contain up to 100 storage operations), and enables atomic updates (all operations succeed or all fail within an ETG). Environments with high latencies such as mobile devices will benefit greatly from using ETGs.  

#####<a name="subheading36"></a>Upsert
Use table **Upsert** operations wherever possible. There are two types of **Upsert**, both of which can be more efficient than a traditional **Insert** and **Update** operations:  

-	**InsertOrMerge**: Use this when you want to upload a subset of the entity’s properties, but aren’t sure whether the entity already exists. If the entity exists, this call updates the properties included in the **Upsert** operation, and leaves all existing properties as they are, if the entity does not exist, it inserts the new entity. This is similar to using projection in a query, in that you only need to upload the properties that are changing. 
-	**InsertOrReplace**: Use this when you want to upload an entirely new entity, but you aren’t sure whether it already exists. You should only use this when you know that the newly uploaded entity is entirely correct because it completely overwrites the old entity. For example, you want to update the entity that stores a user’s current location regardless of whether or not the application has previously stored location data for the user; the new location entity is complete, and you do not need any information from any previous entity. 

#####<a name="subheading37"></a>Storing Data Series in a Single Entity
Sometimes, an application stores a series of data that it frequently needs to retrieve all at once: for example, an application might track CPU usage over time in order to plot a rolling chart of the data from the last 24 hours. One approach is to have one table entity per hour, with each entity representing a specific hour and storing the CPU usage for that hour. To plot this data, the application needs to retrieve the entities holding the data from the 24 most recent hours.  

Alternatively, your application could store the CPU usage for each hour as a separate property of a single entity: to update each hour, your application can use a single **InsertOrMerge Upsert** call to update the value for the most recent hour. To plot the data, the application only needs to retrieve a single entity instead of 24, making for a very efficient query (see above discussion on [query scope](#subheading30)). 

#####<a name="subheading38"></a>Storing structured data in blobs
Sometimes structured data feels like it should go in tables, but ranges of entities are always retrieved together and can be batch inserted.  A good example of this is a log file.  In this case, you can batch several minutes of logs, insert them, and then you are always retrieving several minutes of logs at a time as well.  In this case, for performance, it’s better to use blobs instead of tables, since you can significantly reduce the number of objects written/returned, as well as usually the number of requests that need made.  

##Queues
###<a name=subheading39"></a>Scalability Limits
A single queue can process approximately 2,000 messages (1KB each) per second (each AddMessage, GetMessage, and DeleteMessage count as a message here). If this is insufficient for your application, you should use multiple queues and spread the messages across them.  

You can view the current scalability targets on the page [Azure Storage Scalability and Performance Targets](http://msdn.microsoft.com/library/azure/dn249410.aspx) on MSDN.  

###<a name=subheading40"></a>Nagle Off
See the section on table configuration that discusses the Nagle algorithm — the Nagle algorithm is generally bad for the performance of queue requests, and you should disable it.  

###<a name=subheading41"></a>Message Size
Queue performance and scalability decreases as message size increases. You should place only the information the receiver needs in a message.  

###<a name=subheading42"></a>Batch Retrieval
You can retrieve up to 32 messages from a queue in a single operation. This can reduce the number of roundtrips from the client application, which is especially useful for environments, such as mobile devices, with high latency.  

###<a name=subheading43"></a>Queue Polling Interval
Most applications poll for messages from a queue, which can be one of the largest sources of transactions for that application. Select your polling interval wisely: polling too frequently could cause your application to approach the scalability targets for the queue. However, at 200,000 transactions for $0.01 (at the time of writing), a single processor polling once every second for a month would cost less than 15 cents so cost is not typically a factor that affects your choice of polling interval.  

For up to date cost information, see [Storage Pricing Details](http://azure.microsoft.com/pricing/details/storage/).  

###<a name=subheading44"></a>UpdateMessage
You can use **UpdateMessage** to increase the invisibility timeout or to update state information of a message. While this is powerful, remember that each **UpdateMessage** operation counts towards the scalability target. However, this can be a much more efficient approach than having a workflow that passes a job from one queue to the next, as each step of the job is completed. Using the **UpdateMessage** operation allows your application to save the job state to the message and then continue working, instead of re-queuing the message for the next step of the job every time a step completes.  

For more information, see the article [How to: Change the contents of a queued message](../storage-dotnet-how-to-use-queues/#change-contents).  

###<a name=subheading45"></a>Application architecture
You should use queues to make your application architecture scalable. The following lists some ways you can use queues to make your application more scalable:  

-	You can use queues to create backlogs of work for processing and smooth out workloads in your application. For example, you could queue up requests from users to perform processor intensive work such as resizing uploaded images.
-	You can use queues to decouple parts of your application so that you can scale them independently. For example, a web front-end could place survey results from users into a queue for later analysis and storage. You could add more worker role instances to process the queue data as required.  

##Conclusion
This article discussed some of the most common, proven practices for optimizing performance when using Azure Storage. We encourage every application developer to assess their application against each of the above practices and consider acting on the recommendations to get great performance for their applications that use Azure Storage.
