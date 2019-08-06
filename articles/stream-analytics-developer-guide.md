<properties 
	pageTitle="Stream Analytics Key Concepts | Azure" 
	description="Get guidance on the key components of a Stream Analytics job, including supported inputs and outputs, job configuration details, and exposed metrics." 
	services="stream-analytics" 
	documentationCenter="" 
	authors="jeffstokes72" 
	manager="paulettm" 
	editor="cgronlun"/>

<tags 
	ms.service="stream-analytics" 
	ms.devlang="na" 
	ms.topic="article" 
	ms.tgt_pltfrm="na" 
	ms.workload="data-services" 
	ms.date="04/28/2015" 
	ms.author="jeffstok"/>


# Azure Stream Analytics key concepts 

Azure Stream Analytics is a fully managed service providing low-latency, highly available, scalable, complex event processing over streaming data in the cloud. In the preview release, Stream Analytics enables customers to set up streaming jobs to analyze data streams, and allows customers to drive near real-time analytics.  


Targeted scenarios of Stream Analytics:

- Perform complex event processing on high-volume and high-velocity data   
- Collect event data from globally distributed assets or equipment, such as connected cars or utility grids 
- Process telemetry data for near real-time monitoring and diagnostics 
- Capture and archive real-time events for future processing

For more information, see [Introduction to Azure Stream Analytics](stream.analytics.introduction). 

Stream Analytics jobs are defined as one or more input sources, a query over the incoming stream data, and an output target.  


## Inputs

### Data stream

Each Stream Analytics job definition must contain at least one data-stream input source to be consumed and transformed by the job. [Azure Blob storage](azure.blob.storage) and [Azure Event Hubs](azure.event.hubs) are supported as data-stream input sources. Event Hubs input sources are used to collect event streams from multiple different devices and services, while Blob storage can be used an input source for ingesting large amounts of data. Because blobs do not stream data, Stream Analytics jobs over blobs will not be temporal in nature unless the records in the blob contain timestamps.

### Reference data
Stream Analytics also supports a second type of input source: reference data. This is auxiliary data used for performing correlation and lookups, and the data here is usually static or infrequently changing. [Azure Blob storage](azure.blob.storage) is the only supported input source for reference data. Reference data source blobs are limited to 50MB in size.

To enable support for refreshing reference data the user needs to specify a list of blobs in the input configuration using the {date} and {time} tokens inside the path pattern. The job will load the corresponding blob based on the date and time encoded in the blob names using UTC time zone.

For example if the job has a reference input configured in the portal with the path pattern such as: /sample/{date}/{time}/products.csv where the date format is “YYYY-MM-DD” and the time format is “HH:mm” than the job will pick up a file named /sample/2015-04-16/17:30/products.csv at 5:30 PM on April 16th 2015 UTC time zone (which is equivalent to 10:30 AM on April 16th 2015 using PST time zone).


### Serialization
To ensure correct behavior of queries, Stream Analytics must be aware of the serialization format being used on incoming data streams. Currently supported formats are JSON, CSV, and Avro for streaming data and CSV or JSON for reference data.

### Generated properties
Depending on the input type used in the job, some additional fields with event metadata will be generated. These fields can be queried against just like other input columns. If an existing event has a field that has the same name as one of the properties below, it will be overwritten with the input metadata.

<table border="1">
	<tr>
		<th></th>
		<th>Property</th>
		<th>Description</th>
	</tr>
	<tr>
		<td rowspan="4" valign="top"><strong>Blob</strong></td>
		<td>BlobName</td>
		<td>The name of the input blob that the event came from.</td>
	</tr>
	<tr>
		<td>EventProcessedUtcTime</td>
		<td>The date and time that the blob record was processed.</td>
	</tr>
	<tr>
		<td>BlobLastModifiedUtcTime</td>
		<td>The date and time that the blob was last modified.</td>
	</tr>
	<tr>
		<td>PartitionId</td>
		<td>The zero-based partition ID for the input adapter.</td>
	</tr>
	<tr>
		<td rowspan="3" valign="top"><b>Event Hub</b></td>
		<td>EventProcessedUtcTime</td>
		<td>The date and time that the event was processed.</td>
	</tr>
	<tr>
		<td>EventEnqueuedUtcTime</td>
		<td>The date and time that the event was received by Event Hubs.</td>
	</tr>
	<tr>
		<td>PartitionId</td>
		<td>The zero-based partition ID for the input adapter.</td>
	</tr>
</table>

###Partition(s) with slow or no input data
When reading from input sources that have multiple partitions, and one or more partitions lag behind or do not have data, the streaming job needs to decide how to handle this situation in order to keep events flowing through the system. Input setting ‘Maximum allowed arrival delay’ controls that behavior and is set by default to wait for the data indefinitely, which means events’ timestamps will not be altered, but also that events will flow based on the slowest input partition, and will stop flowing if one or more input partitions do not have data. This is useful if the data is distributed uniformly across input partitions, and time consistency among events is critical. User can also decide to only wait for a limited time, ‘Maximum allowed arrival delay’ determines the delay after which the job will decide to move forward, leaving the lagging input partitions behind, and acting on events according to ‘Action for late events’ setting, dropping their events or adjusting their events’ timestamps if data arrives later. This is useful if latency is critical and timestamp shift is tolerated, but input may not be uniformly distributed.

###Partition(s) with out of order events
When streaming job query uses the TIMESTAMP BY keyword, there are no guarantees about the order in which the events will arrive to input, Some events in the same input partition may be lagging, parameter ‘Maximum allowed disorder within an input’ causes the streaming job to act on events that are outside of the order tolerance, according to ‘Action for late events’ setting, dropping their events or adjusting their events’ timestamps.

### Additional resources
For details on creating input sources, see [Azure Event Hubs developer guide](azure.event.hubs.developer.guide) and [Use Azure Blob Storage](azure.blob.storage.use).  



## Query
The logic to filter, manipulate and process incoming data is defined in the query of Stream Analytics jobs. Queries are written in the Stream Analytics query language, an SQL-like language that is largely a subset of standard Transact-SQL syntax with some specific extensions for temporal queries.

### Windowing
Windowing extensions allow aggregations and computations to be performed over subsets of events that fall within some period of time. Windowing functions are invoked through the **GROUP BY** statement. For example, the following query counts the events received per second: 

	SELECT Count(*) 
	FROM Input1 
	GROUP BY TumblingWindow(second, 1) 

### Execution steps
For more complex queries, the standard SQL clause **WITH** can be used to specify a temporary named result set. For example, this query uses **WITH** to perform a transformation with two execution steps:
 
	WITH step1 AS ( 
		SELECT Avg(Reading) as avr 
		FROM temperatureInput1 
		GROUP BY Building, TumblingWindow(hour, 1) 
	) 

	SELECT Avg(avr) AS campus_Avg 
	FROM step1 
	GROUP BY TumblingWindow (day, 1) 

To learn more about the query language, see [Azure Stream Analytics Query Language Reference](stream.analytics.query.language.reference). 

## Output
The output target is where the results of the Stream Analytics job will be written to. Results are written continuously to the output target as the job processes input events. The following output targets are supported:

- Azure Event Hubs - Choose Event Hubs as an output target for scenarios when multiple streaming pipelines need to be composed together, such as issuing commands back to devices.
- Azure Blob storage - Use Blob storage for long-term archival of output or for storing data for later processing.
- Azure Table storage - Azure Table storage is a structured data store with fewer constraints on the schema. Entities with different schema and different types can be stored in the same Azure table. Azure Table storage can be used to store data for persistence and efficient retrieval. For more information, see [Introduction to Azure Storage](storage.introduction.md) and [Designing a Scalable Partitioning Strategy for Azure Table Storage](https://msdn.microsoft.com/library/azure/hh508997.aspx).
- Azure SQL Database - This output target is appropriate for data that is relational in nature or for applications that depend on content being hosted in a database.


## Scale jobs

A Stream Analytics job can be scaled through configuring streaming units, which define the amount of processing power a job receives. Each streaming unit corresponds to roughly 1MB/second of throughput. Each subscription has a quota of 12 streaming units per region to be allocated across jobs in that region.

For details, see [Scale Azure Stream Analytics jobs](stream.analytics.scale.jobs).


## Monitor and troubleshoot jobs

### Regional monitoring Storage account

To enable job monitoring, Stream Analytics requires you to designate an Azure Storage account for monitoring data in each region that contains Stream Analytics jobs. This is configured at the time of job creation.  

### Metrics
The following metrics are available for monitoring the usage and performance of Stream Analytics jobs:

- Errors - Number of error messages incurred by a Stream Analytics job.
- Input events - Amount of data received by the Stream Analytics job, in terms of event count.
- Output events - Amount of data sent by the Stream Analytics job to the output target, in terms of event count.
- Out-of-order events - Number of events received out of order that were either dropped or given an adjusted timestamp, based on the out-of-order policy.
- Data conversion errors - Number of data conversion errors incurred by a Stream Analytics job.

### Operation logs
The best approach to debugging or troubleshooting a Stream Analytics job is through Azure operation logs. Operation logs can be accessed in the **Management Services** section of the portal. To inspect logs for your job, set **Service Type** to **Stream Analytics** and **Service Name** to the name of your job.


## Manage jobs 

### Start and stop jobs
When starting a job, you're prompted to specify a **Start Output** value, which determines when this job will start producing resulting output. If the associated query includes a window, the job will begin picking up input from the input sources at the start of the window duration required, in order to produce the first output event at the specified time. There are three options: **Job Start Time**, **Custom**, and **Last Stopped Time**. The default setting is **Job Start Time**. For cases when a job has been stopped temporarily, the best practice is to choose **Last Stopped Time** for the **Start Output** value in order to resume the job from the last output time and avoid data loss. For the **Custom** option, you must specify a date and time. This setting is useful for specifying how much historical data in the input sources to consume or for picking up data ingestion from a specific time. 

### Configure jobs
You can adjust the following top-level settings for a Stream Analytics job:

- **Start output** - Use this setting to specify when this job will start producing resulting output. If the associated query includes a window, the job will begin picking up input from the input sources at the start of the window duration required in order to produce the first output event at the specified time. There are two options, **Job Start Time** and **Custom**. The default setting is **Job Start Time**. For the **Custom** option, you must specify a date and time. This setting is useful for specifying how much historical data in the input sources to consume or for picking up data ingestion from a specific time, such as when a job was last stopped. 
- **Out of order policy** - Settings for handling events that do not arrive to the Stream Analytics job sequentially. You can designate a time threshold to reorder events within by specifying a tolerance window and also determine an action to take on events outside this window: **Drop** or **Adjust**. **Drop** will drop all events received out of order, and **Adjust** will change the System.Timestamp of out-of-order events to the timestamp of the most recently received ordered event. 
- **Late arrival policy** - When reading from input sources that have multiple partitions, and one or more partitions lag behind or do not have data, the streaming job needs to decide how to handle this situation in order to keep events flowing through the system. Input setting ‘Maximum allowed arrival delay’ controls that behavior and is set by default to wait for the data indefinitely, which means events’ timestamps will not be altered, but also that events will flow based on the slowest input partition, and will stop flowing if one or more input partitions do not have data. This is useful if the data is distributed uniformly across input partitions, and time consistency among events is critical. User can also decide to only wait for a limited time, ‘Maximum allowed arrival delay’ determines the delay after which the job will decide to move forward, leaving the lagging input partitions behind, and acting on events according to ‘Action for late events’ setting, dropping their events or adjusting their events’ timestamps if data arrives later. This is useful if latency is critical and timestamp shift is tolerated, but input may not be uniformly distributed.
- **Locale** - Use this setting to specify the internationalization preference for the Stream Analytics job. While timestamps of data are locale neutral, settings here impact how the job will parse, compare, and sort data. For the preview release, only **en-US** is supported.

### Status

The status of Stream Analytics jobs can be inspected in the Azure portal. Running jobs can be in one of three states: **Idle**, **Processing**, or **Degraded**. The definition for each of these states is below:

- **Idle** - No input bytes have been seen since the job was created or in the in the last 2 minutes. If a job is in the **Idle** state for a long period of time, it is likely that the input exists but there are no raw bytes to process.
- **Processing** - A nonzero amount of filtered input events has been successfully consumed by the Stream Analytics job. If a job is stuck in the **Processing** state without producing output, it is likely that the processing time window is large or the query logic is complicated.
- **Degraded** - This state indicates that a Stream Analytics job is encountering one of the following errors: input/output communication errors, query errors, or retry-able run-time errors. To distinguish what type of error(s) the job is encountering, view the operation logs.


## Get support
For further assistance, try our [Azure Stream Analytics forum](https://social.msdn.microsoft.com/Forums/en-US/home?forum=AzureStreamAnalytics). 


## Next steps

- [Introduction to Azure Stream Analytics](stream-analytics-introduction.md)
- [Get started using Azure Stream Analytics](stream-analytics-get-started.md)
- [Scale Azure Stream Analytics jobs](stream-analytics-scale-jobs.md)
- [Azure Stream Analytics Query Language Reference](https://msdn.microsoft.com/library/azure/dn834998.aspx)
- [Azure Stream Analytics Management REST API Reference](https://msdn.microsoft.com/library/azure/dn835031.aspx)



<!--Image references-->
[5]: ./media/markdown-template-for-new-articles/octocats.png
[6]: ./media/markdown-template-for-new-articles/pretty49.png
[7]: ./media/markdown-template-for-new-articles/channel-9.png


<!--Link references-->
[azure.blob.storage]: http://azure.microsoft.com/documentation/services/storage/
[azure.blob.storage.use]: http://azure.microsoft.com/documentation/articles/storage-dotnet-how-to-use-blobs/

[azure.event.hubs]: http://azure.microsoft.com/services/event-hubs/
[azure.event.hubs.developer.guide]: http://msdn.microsoft.com/library/azure/dn789972.aspx

[stream.analytics.query.language.reference]: http://go.microsoft.com/fwlink/?LinkID=513299
[stream.analytics.forum]: http://go.microsoft.com/fwlink/?LinkId=512151

[stream.analytics.introduction]: stream-analytics-introduction.md
[stream.analytics.get.started]: stream-analytics-get-started.md
[stream.analytics.developer.guide]: stream-analytics-developer-guide.md
[stream.analytics.scale.jobs]: stream-analytics-scale-jobs.md
[stream.analytics.limitations]: stream-analytics-limitations.md
[stream.analytics.query.language.reference]: http://go.microsoft.com/fwlink/?LinkID=513299
[stream.analytics.rest.api.reference]: http://go.microsoft.com/fwlink/?LinkId=517301
