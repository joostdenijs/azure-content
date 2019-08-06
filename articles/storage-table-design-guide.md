<properties
   pageTitle="Azure Storage Table Design Guide | Microsoft Azure"
   description="Design Scalable and Performant Tables in Azure Table Storage"
   services="storage"
   documentationCenter="na"
   authors="Tamram" 
   manager="adinah"
   editor=""/>

<tags
   ms.service="storage"
   ms.devlang="na"
   ms.topic="article"
   ms.tgt_pltfrm="na"
   ms.workload="storage"
   ms.date="02/26/2015"
   ms.author="tamram"/>

# Azure Storage Table Design Guide: Designing Scalable and Performant Tables

## Overview

To design scalable and performant tables you must consider a number of factors such as performance, scalability, and cost. If you have previously designed schemas for relational databases, these considerations will be familiar to you, but while there are some similarities between the Azure Table service storage model and relational models, there are also many important differences. These differences typically lead to very different designs that may look counter-intuitive or wrong to someone familiar with relational databases, but which do make good sense if you are designing for a NoSQL key/value store such as the Azure Table service. Many of your design differences will reflect the fact that the Table service is designed to support cloud-scale applications that can contain billions of entities (rows in relational database terminology) of data or for datasets that must support very high transaction volumes: therefore, you need to think differently about how you store your data and understand how the Table service works. A well designed NoSQL data store can enable your solution to scale much further (and at a lower cost) than a solution that uses a relational database. This guide helps you with these topics.  

## About the Azure Table service

This section highlights some of the key features of the Table service that are especially relevant to designing for performance and scalability. If you are new to Azure Storage and the Table service, first read [Introduction to Microsoft Azure Storage](storage-introduction.md) and [How to use Table Storage from .NET](storage-dotnet-how-to-use-tables.md) before reading the remainder of this article. Although the focus of this guide is on the Table service, it will include some discussion of the Azure Queue and Blob services, and how you might use them along with the Table service in a solution.  

What is the Table service? As you might expect from the name, the Table service uses a tabular format to store data. In the standard terminology, each row of the table represents an entity, and the columns store the various properties of that entity. Every entity has a pair of keys to uniquely identify it, and a timestamp column that the Table service uses to track when the entity was last updated (this happens automatically and you cannot manually overwrite the timestamp with an arbitrary value). The Table service uses this last modified timestamp (LMT) to manage optimistic concurrency.  

>[AZURE.NOTE] The Table service REST API operations also return an **ETag** value that it derives from the LMT. In this document we will use the terms ETag and LMT interchangeably because they refer to the same underlying data.  

The following example shows a simple table design to store employee and department entities. Many of the examples shown later in this guide are based on this simple design.  

<table>
<tr>
<th>PartitionKey</th>
<th>RowKey</th>
<th>Timestamp</th>
<th></th>
</tr>
<tr>
<td>Marketing</td>
<td>00001</td>
<td>2014-08-22T00:50:32Z</td>
<td>
<table>
<tr>
<th>FirstName</th>
<th>LastName</th>
<th>Age</th>
<th>Email</th>
</tr>
<tr>
<td>Don</td>
<td>Hall</td>
<td>34</td>
<td>donh@contoso.com</td>
</tr>
</table>
</tr>
<tr>
<td>Marketing</td>
<td>00002</td>
<td>2014-08-22T00:50:34Z</td>
<td>
<table>
<tr>
<th>FirstName</th>
<th>LastName</th>
<th>Age</th>
<th>Email</th>
</tr>
<tr>
<td>Jun</td>
<td>Cao</td>
<td>47</td>
<td>junc@contoso.com</td>
</tr>
</table>
</tr>
<tr>
<td>Marketing</td>
<td>Department</td>
<td>2014-08-22T00:50:30Z</td>
<td>
<table>
<tr>
<th>DepartmentName</th>
<th>EmployeeCount</th>
</tr>
<tr>
<td>Marketing</td>
<td>153</td>
</tr>
</table>
</td>
</tr>
<tr>
<td>Sales</td>
<td>00010</td>
<td>2014-08-22T00:50:44Z</td>
<td>
<table>
<tr>
<th>FirstName</th>
<th>LastName</th>
<th>Age</th>
<th>Email</th>
</tr>
<tr>
<td>Ken</td>
<td>Kwok</td>
<td>23</td>
<td>kenk@contoso.com</td>
</tr>
</table>
</td>
</tr>
</table>


So far, this looks very similar to a table in a relational database with the key differences being the mandatory columns, and the ability to store multiple entity types in the same table. In addition, each of the user-defined properties such as **FirstName** or **Age** has a data type, such as integer or string, just like a column in a relational database. Although unlike in a relational database, the schema-less nature of the Table service means that a property need not have the same data type on each entity. To store complex data types in a single property, you must use a serialized format such as JSON or XML. For more information about the table service such as supported data types, supported date ranges, naming rules, and size constraints, see [Understanding the Table Service Data Model](http://msdn.microsoft.com/library/azure/dd179338.aspx) on MSDN.

As you will see, your choice of **PartitionKey** and **RowKey** is fundamental to good table design. Every entity stored in a table must have a unique combination of **PartitionKey** and **RowKey**. As with keys in a relational database table, the **PartitionKey** and **RowKey** values are indexed to create a clustered index that enables fast look-ups; however, the Table service does not create any secondary indexes so these are the only two indexed properties (some of the patterns described later show how you can work around this apparent limitation).  

A table is made up of one or more partitions, and as you will see, many of the design decisions you make will be around choosing a suitable **PartitionKey** and **RowKey** to optimize your solution. A solution could consist of just a single table that contains all your entities organized into partitions, but typically a solution will have multiple tables. Tables help you to logically organize your entities, help you manage access to the data using access control lists, and you can drop an entire table using a single storage operation.  

### Table partitions  
The account name, table name and **PartitionKey** together identify the partition within the storage service where the table service stores the entity. As well as being part of the addressing scheme for entities, partitions define a scope for transactions (see [Entity Group Transactions](#entity-group-transactions) below), and form the basis of how the table service scales. For more information on partitions see [Azure Storage Scalability and Performance Targets](http://msdn.microsoft.com/library/azure/dn249410.aspx).  

In the Table service, an individual node services one or more complete partitions and the service scales by dynamically load-balancing partitions across nodes. If a node is under load, the table service can *split* the range of partitions serviced by that node onto different nodes; when traffic subsides, the service can *merge* the partition ranges from quiet nodes back onto a single node.  

For more information about the internal details of the Table service, and in particular how the service manages partitions, see the paper [Microsoft Azure Storage: A Highly Available
Cloud Storage Service with Strong Consistency](http://blogs.msdn.com/b/windowsazurestorage/archive/2011/11/20/windows-azure-storage-a-highly-available-cloud-storage-service-with-strong-consistency.aspx).  

### Entity Group Transactions
In the Table service, Entity Group Transactions (EGTs) are the only built-in mechanism for performing atomic updates across multiple entities. EGTs are also referred to as *batch transactions* in some documentation. EGTs can only operate on entities stored in the same partition (share the same partition key in a given table), so anytime you need atomic transactional behavior across multiple entities you need to ensure that those entities are in the same partition. This is often a reason for keeping multiple entity types in the same table (and partition) and not using multiple tables for different entity types. A single EGT can operate on at most 100 entities.  If you submit multiple concurrent EGTs for processing it is important to ensure  those EGTs do not operate on entities that are common across EGTs as otherwise processing can be delayed.

EGTs also introduce a potential trade-off for you to evaluate in your design: using more partitions will increase the scalability of your application because Azure has more opportunities for load balancing requests across nodes, but this might limit the ability of your application to perform atomic transactions and maintain strong consistency for your data. Furthermore, there are specific scalability targets at the level of a partition that might limit the throughput of transactions you can expect for a single node: for more information about the scalability targets for Azure storage accounts and the table service, see [Azure Storage Scalability and Performance Targets](http://msdn.microsoft.com/library/azure/dd179338.aspx) on MSDN. Later sections of this guide discuss various design strategies that help you manage trade-offs such as this one, and discuss how best to choose your partition key based on the specific requirements of your client application.  

### Capacity considerations
The following table includes some of the key values to be aware of when you are designing a Table service solution:  

|Total capacity of an Azure storage account|500 TB|
|------------------------------------------|------|
|Number of tables in an Azure storage account | Limited only by the capacity of the storage account |
|Number of partitions in a table | Limited only by the capacity of the storage account |
|Number of entities in a partition | Limited only by the capacity of the storage account|
|Size of an individual entity | Up to 1 MB with a maximum of 255 properties (including the **PartitionKey**, **RowKey**, and **Timestamp**) |
|Size of the **PartitionKey** | A string up to 1 KB in size |
| Size of the **RowKey** | A string up to 1 KB in size |
|Size of an Entity Group Transaction | A transaction can include at most 100 entities and the payload must be less than 4 MB in size. An EGT can only update an entity once. |

For more information, see [Understanding the Table Service Data Model](http://msdn.microsoft.com/library/azure/dd179338.aspx) on MSDN.  

### Cost considerations  
Table storage is relatively inexpensive, but you should include cost estimates for both capacity usage and the quantity of transactions as part of your evaluation of any solution that uses the Table service. However, in many scenarios storing denormalized or duplicate data in order to improve the performance or scalability of your solution is a valid approach to take. For more information about pricing, see [Storage Pricing Details](http://azure.microsoft.com/pricing/details/storage/).  

### Comparing Azure Tables and SQL Azure  
For a comparison between Azure SQL Database (a relational database service) and the Table service, see [Microsoft Azure Table Storage and Microsoft Azure SQL Database - Compared and Contrasted](http://msdn.microsoft.com/library/azure/jj553018.aspx) on MSDN.  

## Guidelines for table design  
These lists summarize some of the key guidelines you should keep in mind when you are designing your tables, and this guide will address them all in more detail later in. These guidelines are very different from the guidelines you would typically follow for relational database design.  

Designing your Table service solution to be *read* efficient:

-	***Design for querying in read-heavy applications.*** When you are designing your tables, think about the queries (especially the latency sensitive ones) that you will execute before you think about how you will update your entities. This typically results in an efficient and performant solution.  
-	***Specify both PartitionKey and RowKey in your queries.*** *Point queries* such as these are the most efficient table service queries.  
-	***Consider storing duplicate copies of entities.*** Table storage is cheap so consider storing the same entity multiple times (with different keys) to enable more efficient queries.  
-	***Consider denormalizing your data.*** Table storage is cheap so consider denormalizing your data. For example, store summary entities so that queries for aggregate data only need to access a single entity.  
-	***Use compound key values.*** The only keys you have are **PartitionKey** and **RowKey**. For example, use compound key values to enable alternate keyed access paths to entities.  
-	***Use query projection.*** You can reduce the amount of data that you transfer over the network by using queries that select just the fields you need.  

Designing your Table service solution to be *write* efficient:  

-	***Do not create hot partitions.*** Choose keys that enable you to spread your requests across multiple partitions at any point of time.  
-	***Avoid spikes in traffic.*** Smooth the traffic over a reasonable period of time and avoid spikes in traffic.
-	***Don’t necessarily create a separate table for each type of entity.*** When you require atomic transactions across entity types, you can store these multiple entity types in the same partition in the same table.
-	***Consider the maximum throughput you must achieve.*** You must be aware of the scalability targets for the Table service and ensure that your design will not cause you to exceed them.  

As you read this guide, you will see examples that put all of these principles into practice.  

## Design for querying  
Table service solutions may be read intensive, write intensive, or a mix of the two. This section focuses on the things to bear in mind when you are designing your Table service to support read operations efficiently. Typically, a design that supports read operations efficiently is also efficient for write operations. However, there are additional considerations to bear in mind when designing to support write operations, which the next section, [Design for data modification](#design-for-data-modification), discusses.

A good starting point for designing your Table service solution to enable you to read data efficiently is to ask "What queries will my application need to execute to retrieve the data it needs from the Table service?"  

>[AZURE.NOTE] With the Table service, it’s important to get the design correct up front because it’s difficult and expensive to change it later. For example, in a relational database it’s often possible to address performance issues simply by adding indexes to an existing database: this is not an option with the Table service.  

This section focuses on the key issues you must address when you design your tables for querying. The topics covered in this section include:

- [How your choice of PartitionKey and RowKey impacts query performance](#how-your-choice-of-partitionkey-and-rowkey-impacts-query-performance)
- [Choosing an appropriate PartitionKey](#choosing-an-appropriate-partitionkey)
- [Optimizing queries with a key value store for the Table service](#optimizing-queries-with-a-key-value-store-for-the-table-service)
- [Sorting data in a key value store in the Table service](#sorting-data-in-a-key-value-store-in-the-table-service)

### How your choice of PartitionKey and RowKey impacts query performance  

The following examples assume the table service is storing employee entities with the following structure (most of the examples omit the **Timestamp** property for clarity):  

|*Column name* |*Data type*|
|--------------|-----------|
|**PartitionKey** (Department Name)|String|
|**RowKey** (Employee Id)|String|
|**FirstName**|String|
|**LastName**|String|
|**Age**|Integer|
|**EmailAddress**|String|

The earlier section [Azure Table service overview](#azure-table-service-overview) describes some of the key features of the Azure Table service that have a direct influence on designing for query. These result in the following general guidelines for designing Table service queries. Note that the filter syntax used in the examples below is from the Table service REST API, for more information see [Query Entities](http://msdn.microsoft.com/library/azure/dd179421.aspx) on MSDN.  

-	A ***Point Query*** is the most efficient lookup to use and is recommended to be used for high-volume lookups or lookups requiring lowest latency. Such a query can use the indexes to locate an individual entity very efficiently by specifying both the **PartitionKey** and **RowKey** values. For example:
$filter=(PartitionKey eq 'Sales') and (RowKey eq '2')  
-	Second best is a ***Range Query*** that uses the **PartitionKey** and filters on a range of **RowKey** values to return more than one entity. The **PartitionKey** value identifies a specific partition, and the **RowKey** values identify a subset of the entities in that partition. For example:
$filter=PartitionKey eq 'Sales' and RowKey ge 'S' and RowKey lt 'T'  
-	Third best is a ***Partition Scan*** that uses the **PartitionKey** and filters on another non-key property and that may return more than one entity. The **PartitionKey** value identifies a specific partition, and the property values select for a subset of the entities in that partition. For example:
$filter=PartitionKey eq 'Sales' and LastName eq 'Smith'  
-	A ***Table Scan*** does not include the **PartitionKey** and is very inefficient because it searches all of the partitions that make up your table in turn for any matching entities. It will perform a table scan regardless of whether or not your filter uses the **RowKey**. For example:
$filter=LastName eq 'Jones'  
-	Queries that return multiple entities return them sorted in **PartitionKey** and **RowKey** order. To avoid resorting the entities in the client chose a **RowKey** that defines the most common sort order.  

Note that using an "**or**" to specify a filter based on **RowKey** values results in a partition scan and is not treated as a range query. Therefore, you should avoid queries that use filters such as:
$filter=PartitionKey eq 'Sales' and (RowKey eq '121' or RowKey eq '322')  

For examples of client-side code that use the Storage Client Library to execute efficient queries, see:  

-	[Retrieving a single entity using the Storage Client Library](#retrieving-a-single-entity-using-the-storage-client-library)
-	[Retrieving multiple entities using LINQ](#retrieving-multiple-entities-using-linq)
-	[Server-side projection](#server-side-projection)  

For examples of client-side code that can handle multiple entity types stored in the same table, see:  

-	[Working with heterogeneous entity types](#working-with-heterogeneous-entity-types)  

### Choosing an appropriate PartitionKey  

Your choice of **PartitionKey** should balance the need to enables the use of EGTs (to ensure consistency) against the requirement to distribute your entities across multiple partitions (to ensure a scalable solution).  

At one extreme, you could store all your entities in a single partition, but this may limit the scalability of your solution and would prevent the table service from being able to load-balance requests. At the other extreme, you could store one entity per partition which would be scalable and the table service to load-balance requests, but which would prevent you from using EGTs.  

An ideal **PartitionKey** is one that enables you to use efficient queries and that has sufficient partitions to ensure your solution is scalable. Typically, you will find that your entities will have a suitable property that distributes your entities across sufficient partitions.  

>[AZURE.NOTE]For example, in a system that stores information about users or employees, User Id can be a good **PartitionKey**.  

There are additional considerations in your choice of **PartitionKey** that relate to how you will insert, update, and delete entities: see the section [Design for data modification](#design-for-data-modification) below.  

### Optimizing queries for the Table service  

The Table service automatically indexes your entities using the **PartitionKey** and **RowKey** values in a single clustered index, hence the reason that point queries are the most efficient to use. However, there are no indexes other than that on the clustered index on the **PartitionKey** and **RowKey**.

Many designs must meet requirements to enable lookup of entities based on multiple criteria. For example, locating employee entities based on email, employee id, or last name. The following patterns in the section [Table Design Patterns](#table-design-patterns) address these types of requirement and describe ways of working around the fact that the Table service does not provide secondary indexes:  

-	[Intra-partition secondary index pattern](#intra-partition-secondary-index-pattern) - Store multiple copies of each entity using different **RowKey** values (in the same partition) to enable fast and efficient lookups and alternate sort orders by using different **RowKey** values.  
-	[Inter-partition secondary index pattern](#inter-partition-secondary-index-pattern) - Store multiple copies of each entity using different RowKey values in separate partitions or in separate tables to enable fast and efficient lookups and alternate sort orders by using different **RowKey** values.  
-	[Index Entities Pattern](#index-entities-pattern) - Maintain index entities to enable efficient searches that return lists of entities.  

### Sorting data in in the Table service  

The Table service returns entities sorted in ascending order based on **PartitionKey** and then by **RowKey**. These keys are string values and to ensure that numeric values sort correctly, you should convert them to a fixed length and pad them with zeroes. For example, if the employee id value you use as the **RowKey** is an integer value, you should convert employee id **123** to **00000123**.  

Many applications have requirements to use data sorted in different orders: for example, sorting employees by name, or by joining date. The following patterns in the section [Table Design Patterns](#table-design-patterns) address how to alternate sort orders for your entities:  

-	[Intra-partition secondary index pattern](#intra-partition-secondary-index-pattern) - Store multiple copies of each entity using different RowKey values (in the same partition) to enable fast and efficient lookups and alternate sort orders by using different RowKey values.  
-	[Inter-partition secondary index pattern](#inter-partition-secondary-index-pattern) - Store multiple copies of each entity using different RowKey values in separate partitions in separate tables to enable fast and efficient lookups and alternate sort orders by using different RowKey values.
-	[Log tail pattern](#log-tail-pattern) - Retrieve the *n* entities most recently added to a partition by using a **RowKey** value that sorts in reverse date and time order.  

## Design for data modification
This section focuses on the design considerations for optimizing inserts, updates, and deletes. In some cases, you will need to evaluate the trade-off between designs that optimize for querying against designs that optimize for data modification just as you do in designs for relational databases (although the techniques for managing the design trade-offs are different in a relational database). The section [Table Design Patterns](#table-design-patterns) describes some detailed design patterns for the Table service and highlights some these trade-offs. In practice, you will find that many designs optimized for querying entities also work well for modifying entities.  

### Optimizing the performance of insert, update, and delete operations  

To update or delete an entity, you must be able to identify it by using the **PartitionKey** and **RowKey** values. In this respect, your choice of **PartitionKey** and **RowKey** for modifying entities should follow similar criteria to your choice to support point queries because you want to identify entities as efficiently as possible. You do not want to use an inefficient partition or table scan to locate an entity in order to discover the **PartitionKey** and **RowKey** values you need to update or delete it.  

The following patterns in the section [Table Design Patterns](#table-design-patterns) address optimizing the performance or your insert, update, and delete operations:  

-	[High volume delete pattern](#high-volume-delete-pattern) - Enable the deletion of a high volume of entities by storing all the entities for simultaneous deletion in their own separate table; you delete the entities by deleting the table.  
-	[Data series pattern](#data-series-pattern) - Store complete data series in a single entity to minimize the number of requests you make.  
-	[Wide entities pattern](#wide-entities-pattern) - Use multiple physical entities to store logical entities with more than 252 properties.  
-	[Large entities pattern](#large-entities-pattern) - Use blob storage to store large property values.  

### Ensuring consistency in your stored entities  

The other key factor that influences your choice of keys for optimizing data modifications is how to ensure consistency by using atomic transactions. You can only use an EGT to operate on entities stored in the same partition.  

The following patterns in the section [Table Design Patterns](#table-design-patterns) address managing consistency:  

-	[Intra-partition secondary index pattern](#intra-partition-secondary-index-pattern) - Store multiple copies of each entity using different **RowKey** values (in the same partition) to enable fast and efficient lookups and alternate sort orders by using different **RowKey** values.  
-	[Inter-partition secondary index pattern](#inter-partition-secondary-index-pattern) - Store multiple copies of each entity using different RowKey values in separate partitions or in separate tables to enable fast and efficient lookups and alternate sort orders by using different **RowKey** values.  
-	[Eventually consistent transactions pattern](#eventually-consistent-transactions-pattern) - Enable eventually consistent behavior across partition boundaries or storage system boundaries by using Azure queues.
-	[Index Entities Pattern](#index-entities-pattern) - Maintain index entities to enable efficient searches that return lists of entities.  
-	[Denormalization pattern](#denormalization-pattern) - Combine related data together in a single entity to enable you to retrieve all the data you need with a single point query.  
-	[Data series pattern](#data-series-pattern) - Store complete data series in a single entity to minimize the number of requests you make.  

For information about EGTs, see the section [Entity Group Transactions](#entity-group-transactions).  

### Ensuring your design for efficient modifications facilitates efficient queries  

In many cases, a design for efficient querying results in efficient modifications, but you should always evaluate whether this is the case for your specific scenario. Some of the patterns in the section [Table Design Patterns](#table-design-patterns) explicitly evaluate trade-offs between querying and modifying entities, and you should always take into account the number of each type of operation.  

The following patterns in the section [Table Design Patterns](#table-design-patterns) address trade-offs between designing for efficient queries and designing for efficient data modification:  

-	[Compound key pattern](#compound-key-pattern) - Use compound **RowKey** values to enable a client to lookup related data with a single point query.  
-	[Log tail pattern](#log-tail-pattern) - Retrieve the *n* entities most recently added to a partition by using a **RowKey** value that sorts in reverse date and time order.  

## Modelling relationships  

Building domain models is a key step in the design of complex systems. Typically, you use the modelling process to identify entities and the relationships between them as a way to understand the business domain and inform the design of your system. This section focuses on how you can translate some of the common relationship types found in domain models to designs for the Table service. The process of mapping from a logical data-model to a physical NoSQL based data-model is very different from that used when designing a relational database. Relational databases design typically assumes a data normalization process optimized for minimizing redundancy – and a declarative querying capability that abstracts how the implementation of how the database works.  

### One-to-many relationships  

One-to-many relationships between business domain objects occur very frequently: for example, one department has many employees. There are several ways to implement one-to-many relationships in the Table service each with pros and cons that may be relevant to the particular scenario.  

Consider the example of a large multi-national corporation with tens of thousands of departments and employee entities where every department has many employees and each employee as associated with one specific department. One approach is to store separate department and employee entities such as these:  

![][1]

This example shows an implicit one-to-many relationship between the types based on the **PartitionKey** value. Each department can have many employees.  

This example also shows a department entity and its related employee entities in the same partition. You could choose to use different partitions, tables, or even storage accounts for the different entity types.  

An alternative approach is to denormalize your data and store only employee entities with denormalized department data as shown in the following example. In this particular scenario, this denormalized approach may not be the best if you have a requirement to be able to change the details of a department manager because to do this you need to update every employee in the department.  

![][2]

For more information, see the [Denormalization pattern](#denormalization-pattern) later in this guide.  

The following table summarizes the pros and cons of each of the approaches outlined above for storing employee and department entities that have a one-to-many relationship. You should also consider how often you expect to perform various operations: it may be acceptable to have a design that includes an expensive operation if that operation only happens infrequently.  

<table>
<tr>
<th>Approach</th>
<th>Pros</th>
<th>Cons</th>
</tr>
<tr>
<td>Separate entity types, same partition, same table</td>
<td>
<ul>
<li>You can update a department entity with a single operation.</li>
<li>You can use an EGT to maintain consistency if you have a requirement to modify a department entity whenever you update/insert/delete an employee entity. For example if you maintain a departmental employee count for each department.</li>
</ul>
</td>
<td>
<ul>
<li>You may need to retrieve both an employee and a department entity for some client activities.</li>
<li>Storage operations happen in the same partition. At high transaction volumes, this may result in a hotspot.</li>
<li>You cannot move an employee to a new department using an EGT.</li>
</ul>
</td>
</tr>
<tr>
<td>Separate entity types, different partitions or tables or storage accounts</td>
<td>
<ul>
<li>You can update a department entity or employee entity with a single operation.</li>
<li>At high transaction volumes, this may help spread the load across more partitions.</li>
</ul>
</td>
<td>
<ul>
<li>You may need to retrieve both an employee and a department entity for some client activities.</li>
<li>You cannot use EGTs to maintain consistency when you update/insert/delete an employee and update a department. For example, updating an employee count in a department entity.</li>
<li>You cannot move an employee to a new department using an EGT.</li>
</ul>
</td>
</tr>
<tr>
<td>Denormalize into single entity type</td>
<td>
<ul>
<li>You can retrieve all the information you need with a single request.</li>
</ul>
</td>
<td>
<ul>
<li>It may be expensive to maintain consistency if you need to update department information (this would require you to update all the employees in a department).</li>
</ul>
</td>
</tr>
</table>

How you choose between these options, and which of the pros and cons are most significant, depends on your specific application scenarios. For example, how often do you modify department entities; do all your employee queries need the additional departmental information; how close are you to the scalability limits on your partitions or your storage account?  

### One-to-one relationships  

Domain models may include one-to-one relationships between entities. If you need to implement a one-to-one relationship in the Table service, you must also choose how to link the two related entities when you need to retrieve them both. This link can be either implicit, based on a convention in the key values, or explicit by storing a link in the form of **PartitionKey** and **RowKey** values in each entity to its related entity. For a discussion of whether you should store the related entities in the same partition, see the section [One-to-many relationships](#one-to-many-relationships).  

Note that there are also implementation considerations that might lead you to implement one-to-one relationships in the Table service:  

-	Handling large entities (for more information, see [Working with large entities](#working-with-large-entities)).  
-	Implementing access controls (for more information, see [Controlling access with Shared Access Signatures](#controlling-access-with-shared-access-signatures)).  

### Join in the client  

Although there are ways to model relationships in the Table service, you should not forget that the two prime reasons for using the Table service are scalability and performance. If you find you are modelling many relationships that compromise the performance and scalability of your solution, you should ask yourself if it is necessary to build all the data relationships into your table design. You may be able to simplify the design and improve the scalability and performance of your solution if you let your client application perform any necessary joins.  

For example, if you have small tables that contain data that does not change very often, then you can retrieve this data once and cache it on the client. This can avoid repeated roundtrips to retrieve the same data. In the examples we have looked at in this guide, the set of departments in a small organization is likely to be small and change infrequently making it a good candidate for data that client application can download once and cache as look up data.  

### Inheritance relationships  

If your client application uses a set of classes that form part of an inheritance relationship to represent business entities, you can easily persist those entities in the Table service. For example, you might have the following set of classes defined in your client application where **Person** is an abstract class.

![][3]

You can persist instances of the two concrete classes in the Table service using a single Person table using entities in that look like this:  

![][4]

For more information about working with multiple entity types in the same table in client code, see the section [Working with heterogeneous entity types](#working-with-heterogeneous-entity-types) later in this guide. This provides examples of how to recognize the entity type in client code.  

## Table Design Patterns
In previous sections, you have seen some detailed discussions about how to optimize your table design for both retrieving entity data using queries and for inserting, updating, and deleting entity data. This section describes some patterns appropriate for use with Table service solutions. In addition, you will see how you can practically address some of the issues and trade-offs raised previously in this guide. The following diagram summarizes the relationships between the different patterns:  

![][5]

The pattern map above highlights some relationships between patterns (blue) and anti-patterns (orange) that are documented in this guide. There are of course many other patterns that are worth considering. For example, one of the key scenarios for Table Service is to store [Materialized Views](https://msdn.microsoft.com/library/azure/dn589782.aspx) from the [Command Query Responsibility Segregation](https://msdn.microsoft.com/library/azure/jj554200.aspx) (CQRS) pattern.  

### Intra-partition secondary index pattern
Store multiple copies of each entity using different **RowKey** values (in the same partition) to enable fast and efficient lookups and alternate sort orders by using different **RowKey** values. Updates between copies can be kept consistent using EGT’s.  

#### Context and problem
The Table service automatically indexes entities using the **PartitionKey** and **RowKey** values. This enables a client application to retrieve an entity efficiently using these values. For example, using the table structure shown below, a client application can use a point query to retrieve an individual employee entity by using the department name and the employee id (the **PartitionKey** and **RowKey** values). A client can also retrieve entities sorted by employee id within each department.

![][6]

If you also want to be able to find an employee entity based on the value of another property, such as email address, you must use a less efficient partition scan to find a match. This is because the table service does not provide secondary indexes. In addition, there is no option to request a list of employees sorted in a different order than **RowKey** order.  

#### Solution
To work around the lack of secondary indexes, you can store multiple copies of each entity with each copy using a different **RowKey** value. If you store an entity with the structures shown below, you can efficiently retrieve employee entities based on email address or employee id. The prefix values for the **RowKey**, "empid_" and "email_" enable you to query for a single employee or a range of employees by using a range of email addresses or employee ids.  

![][7]

The following two filter criteria (one looking up by employee id and one looking up by email address) both specify point queries:  

-	$filter=(PartitionKey eq 'Sales') and (RowKey eq 'empid_000223')  
-	$filter=(PartitionKey eq 'Sales') and (RowKey eq 'email_jonesj@contoso.com')  

If you query for a range of employee entities, you can specify a range sorted in employee id order, or a range sorted in email address order by querying for entities with the appropriate prefix in the **RowKey**.  

-	To find all the employees in the Sales department with an employee id in the range 000100 to 000199 use:
$filter=(PartitionKey eq 'Sales') and (RowKey ge 'empid_000100') and (RowKey le 'empid_000199')  
-	To find all the employees in the Sales department with an email address starting with the letter 'a' use:
$filter=(PartitionKey eq 'Sales') and (RowKey ge 'email_a') and (RowKey lt 'email_b')  

 Note that the filter syntax used in the examples above is from the Table service REST API, for more information see [Query Entities](http://msdn.microsoft.com/library/azure/dd179421.aspx) on MSDN.  

#### Issues and considerations  

Consider the following points when deciding how to implement this pattern:  

-	Table storage is relatively cheap to use so the cost overhead of storing duplicate data should not be a major concern. However, you should always evaluate the cost of your design based on your anticipated storage requirements and only add duplicate entities to support the queries your client application will execute.  
-	Because the secondary index entities are stored in the same partition as the original entities, you should ensure that you do not exceed the scalability targets for an individual partition.  
-	You can keep your duplicate entities consistent with each other by using EGTs to update the two copies of the entity atomically. This implies that you should store all copies of an entity in the same partition. For more information, see the section [Using Entity Group Transactions](#entity-group-transactions).  
-	The value used for the **RowKey** must be unique for each entity. Consider using compound key values.  
-	Padding numeric values in the **RowKey** (for example, the employee id 000223), enables correct sorting and filtering based on upper and lower bounds.  
-	You do not necessarily need to duplicate all the properties of your entity. For example, if the queries that lookup the entities using the email address in the **RowKey** never need the employee’s age, these entities could have the following structure:

![][8]

-	It is typically better to store duplicate data and ensure that you can retrieve all the data you need with a single query, than to use one query to locate an entity and another to lookup the required data.  

#### When to use this pattern  

Use this pattern when your client application needs to retrieve entities using a variety of different keys, when your client needs to retrieve entities in different sort orders, and where you can identify each entity using a variety of unique values. However, you should be sure that you do not exceed the partition scalability limits when you are performing entity lookups using the different **RowKey** values.  

#### Related patterns and guidance  

The following patterns and guidance may also be relevant when implementing this pattern:  

-	[Inter-partition secondary index pattern](#inter-partition-secondary-index-pattern)
-	[Compound key pattern](#compound-key-pattern)
-	[Entity Group Transactions](#entity-group-transactions)
-	[Working with heterogeneous entity types](#working-with-heterogeneous-entity-types)

### Inter-partition secondary index pattern
Store multiple copies of each entity using different **RowKey** values in separate partitions or in separate tables to enable fast and efficient lookups and alternate sort orders by using different **RowKey** values.  

#### Context and problem
The Table service automatically indexes entities using the **PartitionKey** and **RowKey** values. This enables a client application to retrieve an entity efficiently using these values. For example, using the table structure shown below, a client application can use a point query to retrieve an individual employee entity by using the department name and the employee id (the **PartitionKey** and **RowKey** values). A client can also retrieve entities sorted by employee id within each department.  

![][9]

If you also want to be able to find an employee entity based on the value of another property, such as email address, you must use a less efficient partition scan to find a match. This is because the table service does not provide secondary indexes. In addition, there is no option to request a list of employees sorted in a different order than **RowKey** order.  

You are anticipating a very high volume of transactions against these entities and want to minimize the risk of the Table service throttling your client.  

#### Solution  
To work around the lack of secondary indexes, you can store multiple copies of each entity with each copy using different **PartitionKey** and **RowKey** values. If you store an entity with the structures shown below, you can efficiently retrieve employee entities based on email address or employee id. The prefix values for the **PartitionKey**, "empid_" and "email_" enable you to identify which index you want to use for a query.  

![][10]

The following two filter criteria (one looking up by employee id and one looking up by email address) both specify point queries:  

-	$filter=(PartitionKey eq 'empid_Sales') and (RowKey eq '000223')
-	$filter=(PartitionKey eq 'email_Sales') and (RowKey eq 'jonesj@contoso.com')  

If you query for a range of employee entities, you can specify a range sorted in employee id order, or a range sorted in email address order by querying for entities with the appropriate prefix in the **RowKey**.  

-	To find all the employees in the Sales department with an employee id in the range **000100** to **000199** sorted in employee id order use:
$filter=(PartitionKey eq 'empid_Sales') and (RowKey ge '000100') and (RowKey le '000199')  
-	To find all the employees in the Sales department with an email address that starts with 'a' sorted in email address order use:
$filter=(PartitionKey eq 'email_Sales') and (RowKey ge 'a') and (RowKey lt 'b')  

Note that the filter syntax used in the examples above is from the Table service REST API, for more information see [Query Entities](http://msdn.microsoft.com/library/azure/dd179421.aspx) on MSDN.  

#### Issues and considerations  
Consider the following points when deciding how to implement this pattern:  

-	You can keep your duplicate entities eventually consistent with each other by using the [Eventually consistent transactions pattern](#eventually-consistent-transactions-pattern) to maintain the primary and secondary index entities.  
-	Table storage is relatively cheap to use so the cost overhead of storing duplicate data should not be a major concern. However, you should always evaluate the cost of your design based on your anticipated storage requirements and only add duplicate entities to support the queries your client application will execute.  
-	The value used for the **RowKey** must be unique for each entity. Consider using compound key values.  
-	Padding numeric values in the **RowKey** (for example, the employee id 000223), enables correct sorting and filtering based on upper and lower bounds.  
-	You do not necessarily need to duplicate all the properties of your entity. For example, if the queries that lookup the entities using the email address in the **RowKey** never need the employee’s age, these entities could have the following structure:

	![][11]

-	It is typically better to store duplicate data and ensure that you can retrieve all the data you need with a single query than to use one query to locate an entity using the secondary index and another to lookup the required data in the primary index.  

#### When to use this pattern  
Use this pattern when your client application needs to retrieve entities using a variety of different keys, when your client needs to retrieve entities in different sort orders, and where you can identify each entity using a variety of unique values. Use this pattern when you want to avoid exceeding the partition scalability limits when you are performing entity lookups using the different **RowKey** values.  

#### Related patterns and guidance
The following patterns and guidance may also be relevant when implementing this pattern:  

-	[Eventually consistent transactions pattern](#eventually-consistent-transactions-pattern)  
-	[Intra-partition secondary index pattern](#intra-partition-secondary-index-pattern)  
-	[Compound key pattern](#compound-key-pattern)  
-	[Entity Group Transactions](#entity-group-transactions)  
-	[Working with heterogeneous entity types](#working-with-heterogeneous-entity-types)  

### Eventually consistent transactions pattern  

Enable eventually consistent behavior across partition boundaries or storage system boundaries by using Azure queues.  

#### Context and problem  

EGTs enable atomic transactions across multiple entities that share the same partition key. For performance and scalability reasons, you might decide to store entities that have consistency requirements in separate partitions or in a separate storage system: in such a scenario, you cannot use EGTs to maintain consistency. For example, you might have a requirement to maintain eventual consistency between:  

-	Entities stored in two different partitions in the same table, in different tables, in in different storage accounts.  
-	An entity stored in the Table service and a blob stored in the Blob service.  
-	An entity stored in the Table service and a file in a file system.  
-	An entity store in the Table service yet indexed using the Azure Search service.  

#### Solution  

By using Azure queues, you can implement a solution that delivers eventual consistency across two or more partitions or storage systems.
To illustrate this approach, assume you have a requirement to be able to archive old employee entities. Old employee entities are rarely queried and should be excluded from any activities that deal with current employees. To implement this requirement you store active employees in the **Current** table and old employees in the **Archive** table. Archiving an employee requires you to delete the entity from the **Current** table and add the entity to the **Archive** table, but you cannot use an EGT to perform these two operations. To avoid the risk that a failure causes an entity to appear in both or neither tables, the archive operation must be eventually consistent. The following sequence diagram outlines the steps in this operation. More detail is provided for exception paths in the text following.  

![][12]

A client initiates the archive operation by placing a message on an Azure queue, in this example to archive employee #456. A worker role polls the queue for new messages; when it finds one, it reads the message and leaves a hidden copy on the queue. The worker role next fetches a copy of the entity from the **Current** table, inserts a copy in the **Archive** table, and then deletes the original from the **Current** table. Finally, if there were no errors from the previous steps, the worker role deletes the hidden message from the queue.  

In this example, step 4 inserts the employee into the **Archive** table. It could add the employee to a blob in the Blob service or a file in a file system.  

#### Recovering from failures  

It is important that the operations in steps **4** and **5** must be *idempotent* in case the worker role needs to restart the archive operation. If you are using the Table service, for step **4** you should use an "insert or replace" operation; for step **5** you should use a "delete if exists" operation in the client library you are using. If you are using another storage system, you must use an appropriate idempotent operation.  

If the worker role never completes step **6**, then after a timeout the message reappears on the queue ready for the worker role to try to reprocess it. The worker role can check how many times a message on the queue has been read and, if necessary, flag it is a "poison" message for investigation by sending it to a separate queue. For more information about reading queue messages and checking the dequeue count, see [Get Messages](https://msdn.microsoft.com/library/azure/dd179474.aspx).  

Some errors from the Table and Queue services are transient errors, and your client application should include suitable retry logic to handle them.  

#### Issues and considerations
Consider the following points when deciding how to implement this pattern:  

-	This solution does not provide for transaction isolation. For example, a client could read the **Current** and **Archive** tables when the worker role was between steps **4** and **5**, and see an inconsistent view of the data. Note that the data will be consistent eventually.  
-	You must be sure that steps 4 and 5 are idempotent in order to ensure eventual consistency.  
-	You can scale the solution by using multiple queues and worker role instances.  

#### When to use this pattern  
Use this pattern when you want to guarantee eventual consistency between entities that exist in different partitions or tables. You can extend this pattern to ensure eventual consistency for operations across the Table service and the Blob service and other non-Azure Storage data sources such as database or the file system.  

#### Related patterns and guidance  
The following patterns and guidance may also be relevant when implementing this pattern:  
-	[Entity Group Transactions](#entity-group-transactions)  
-	[Merge or replace](#merge-or-replace)  

>[AZURE.NOTE] If transaction isolation is important to your solution, you should consider redesigning your tables to enable you to use EGTs.  

### Index Entities Pattern
Maintain index entities to enable efficient searches that return lists of entities.  

#### Context and problem  

The Table service automatically indexes entities using the **PartitionKey** and **RowKey** values. This enables a client application to retrieve an entity efficiently using a point query. For example, using the table structure shown below, a client application can efficiently retrieve an individual employee entity by using the department name and the employee id (the **PartitionKey** and **RowKey**).  

![][13]

If you also want to be able to retrieve a list of employee entities based on the value of another non-unique property, such as their last name, you must use a less efficient partition scan to find matches rather than using an index to look them up directly. This is because the table service does not provide secondary indexes.  

#### Solution  

To enable lookup by last name with the entity structure shown above, you must maintain lists of employee ids. If you want to retrieve the employee entities with a particular last name, such as Jones, you must first locate the list of employee ids for employees with Jones as their last name, and then retrieve those employee entities. There are three main options for storing the lists of employee ids:  

-	Use blob storage.  
-	Create index entities in the same partition as the employee entities.  
-	Create index entities in a separate partition or table.  

<u>Option #1: Use blob storage</u>  

For the first option, you create a blob for every unique last name, and in each blob store a list of the **PartitionKey** (department) and **RowKey** (employee id) values for employees that have that last name. When you add or delete an employee you should ensure that the content of the relevant blob is eventually consistent with the employee entities.  

<u>Option #2:</u> Create index entities in the same partition  

For the second option, use index entities that store the following data:  

![][14]

The **EmployeeIDs** property contains a list of employee ids for employees with the last name stored in the **RowKey**.  

The following steps outline the process you should follow when you are adding a new employee if you are using the second option. In this example, we are adding an employee with Id 000152 and a last name Jones in the Sales department:  
1.	Retrieve the index entity with a **PartitionKey** value "Sales" and the **RowKey** value "Jones." Save the ETag of this entity to use in step 2.  
2.	Create an EGT that inserts the new employee entity (**PartitionKey** value "Sales" and **RowKey** value "000152"), and updates the index entity (**PartitionKey** value "Sales" and **RowKey** value "Jones") by adding the new employee id to the list in the EmployeeIDs field.  
3.	If the ETG fails because of an optimistic concurrency error (someone else has just modified the index entity), then you need to start over at step 1 again.  

You can use a similar approach to deleting an employee if you are using the second option. Changing an employee's last name is slightly more complex because you will need to execute an ETG that updates three entities: the employee entity, the index entity for the old last name, and the index entity for the new last name. You must retrieve each entity before making any changes in order to retrieve the ETag values that you can then use to perform the updates using optimistic concurrency.  

The following steps outline the process you should follow when you need to look up all the employees with a given last name in a department if you are using the second option. In this example, we are looking up all the employees with last name Jones in the Sales department:  

1.	Retrieve the index entity with a **PartitionKey** value "Sales" and the **RowKey** value "Jones."  
2.	Parse the list of employee Ids in the EmployeeIDs field.  
3.	If you need additional information about each of these employees (such as their email addresses), retrieve each of the employee entities using **PartitionKey** value "Sales" and **RowKey** values from the list of employees you obtained in step 2.  

<u>Option #3:</u> Create index entities in a separate partition or table  

For the third option, use index entities that store the following data:  

![][15]

The **EmployeeIDs** property contains a list of employee ids for employees with the last name stored in the **RowKey**.  

With the third option, you cannot use EGTs to maintain consistency because the index entities are in a separate partition from the employee entities. You should ensure that the index entities are eventually consistent with the employee entities.  

#### Issues and considerations  

Consider the following points when deciding how to implement this pattern:  
-	This solution requires at least two queries to retrieve matching entities: one to query the index entities to obtain the list of **RowKey** values, and then queries to retrieve each entity in the list.  
-	Given that an individual entity has a maximum size of 1 MB, option #2 and option #3 in the solution assume that the list of employee ids for any given last name is never greater than 1 MB. If the list of employee ids is likely to be greater than 1 MB in size, use option #1 and store the index data in blob storage.  
-	If you use option #2 (using EGTs to handle adding and deleting employees, and changing an employee’s last name) you must evaluate if the volume of transactions will approach the scalability limits in a given partition. If this is the case, you should consider an eventually consistent solution (option #1 or option #3) that uses queues to handle the update requests and enables you to store your index entities in a separate partition from the employee entities.  
-	Option #2 in this solution assumes that you want to look up by last name within a department: for example, you want to retrieve a list of employees with a last name Jones in the Sales department. If you want to be able to look up all the employees with a last name Jones across the whole organization, use either option #1 or option #3.
-	You can implement a queue-based solution that delivers eventual consistency (see the [Eventually consistent transactions pattern](#eventually-consistent-transactions-pattern) for more details).  

#### When to use this pattern  

Use this pattern when you want to lookup a set of entities that all share a common property value, such as all employees with the last name Jones.  

#### Related patterns and guidance  

The following patterns and guidance may also be relevant when implementing this pattern:  
-	[Compound key pattern](#compound-key-pattern)  
-	[Eventually consistent transactions pattern](#eventually-consistent-transactions-pattern)  
-	[Entity Group Transactions](#entity-group-transactions)  
-	[Working with heterogeneous entity types](#working-with-heterogeneous-entity-types)  

### Denormalization pattern  

Combine related data together in a single entity to enable you to retrieve all the data you need with a single point query.  

#### Context and problem  

In a relational database, you typically normalize data to remove duplication resulting in queries that retrieve data from multiple tables. If you normalize your data in Azure tables, you must make multiple round trips from the client to the server to retrieve your related data. For example, with the table structure shown below you need two round trips to retrieve the details for a department: one to fetch the department entity that includes the manager’s id, and then another request to fetch the manager’s details in an employee entity.  

![][16]

#### Solution  

Instead of storing the data in two separate entities, denormalize the data and keep a copy of the manager’s details in the department entity. For example:  

![][17]

With department entities stored with these properties, you can now retrieve all the details you need about a department using a point query.  

#### Issues and considerations  

Consider the following points when deciding how to implement this pattern:  

-	There is some cost overhead associated with storing some data twice. The performance benefit (resulting from fewer requests to the storage service) typically outweighs the marginal increase in storage costs (and this cost is partially offset by a reduction in the number of transactions you require to fetch the details of a department).  
-	You must maintain the consistency of the two entities that store information about managers. You can handle the consistency issue by using EGTs to update multiple entities in a single atomic transaction: in this case, the department entity, and the employee entity for the department manager are stored in the same partition.  

#### When to use this pattern
Use this pattern when you frequently need to look up related information. This pattern reduces the number of queries your client must make to retrieve the data it requires.  

#### Related patterns and guidance
The following patterns and guidance may also be relevant when implementing this pattern:  
-	[Compound key pattern](#compound-key-pattern)  
-	[Entity Group Transactions](#entity-group-transactions)  
-	[Working with heterogeneous entity types](#working-with-heterogeneous-entity-types)

### Compound key pattern  

Use compound **RowKey** values to enable a client to lookup related data with a single point query.  

#### Context and problem  

In a relational database, it is quite natural to use joins in queries to return related pieces of data to the client in a single query. For example, you might use the employee id to look up a list of related entities that contain performance and review data for that employee.  

Assume you are storing employee entities in the Table service using the following structure:  

![][18]

You also need to store historical data relating to reviews and performance for each year the employee has worked for your organization and you need to be able to access this information by year. One option is to create another table that stores entities with the following structure:  

![][19]

Notice that with this approach you may decide to duplicate some information (such as first name and last name) in the new entity to enable you to retrieve your data with a single request. However, you cannot maintain strong consistency because you cannot use an EGT to update the two entities atomically.  

#### Solution
Store a new entity type in your original table using entities with the following structure:  

![][20]

Notice how the **RowKey** is now a compound key made up of the employee id and the year of the review data that enables you to retrieve the employee’s performance and review data with a single request for a single entity.  

The following example outlines how you can retrieve all the review data for a particular employee (such as employee 000123 in the Sales department):  

$filter=(PartitionKey eq 'Sales') and (RowKey ge 'empid_000123') and (RowKey lt 'empid_000124')&$select=RowKey,Manager Rating,Peer Rating,Comments  

#### Issues and considerations
Consider the following points when deciding how to implement this pattern:  

-	You should use a suitable separator character that makes it easy to parse the **RowKey** value: for example, **000123_2012**.  
-	You are also storing this entity in the same partition as other entities that contain related data for the same employee, which means you can use EGTs to maintain strong consistency.
-	You should consider how frequently you will query the data to determine whether this pattern is appropriate.  For example, if you will access the review data infrequently and the main employee data often you should keep them as separate entities.  

#### When to use this pattern  

Use this pattern when you need to store one or more related entities that you query frequently.  

#### Related patterns and guidance  

The following patterns and guidance may also be relevant when implementing this pattern:  

-	[Entity Group Transactions](#entity-group-transactions)  
-	[Working with heterogeneous entity types](#working-with-heterogeneous-entity-types)  
-	[Eventually consistent transactions pattern](#eventually-consistent-transactions-pattern)  

### Log tail pattern  

Retrieve the *n* entities most recently added to a partition by using a **RowKey** value that sorts in reverse date and time order.  

#### Context and problem  

A common requirement is be able to retrieve the most recently created entities, for example the ten most recent expense claims submitted by an employee. Table queries support a **$top** query operation to return the first *n* entities from a set: there is no equivalent query operation to return the last n entities in a set.  

#### Solution  

Store the entities using a **RowKey** that naturally sorts in reverse date/time order by using so the most recent entry is always the first one in the table.  

For example, to be able to retrieve the ten most recent expense claims submitted by an employee, you can use a reverse tick value derived from the current date/time. The following C# code sample shows one way to create a suitable "inverted ticks" value for a **RowKey** that sorts from the most recent to the oldest:  

`string invertedTicks = string.Format("{0:D19}", DateTime.MaxValue.Ticks - DateTime.UtcNow.Ticks);`  

You can get back to the date time value using the following code:  

`DateTime dt = new DateTime(DateTime.MaxValue.Ticks - Int64.Parse(invertedTicks));`  

The table query looks like this:  

`https://myaccount.table.core.windows.net/EmployeeExpense(PartitionKey='empid')?$top=10`  

#### Issues and considerations  

Consider the following points when deciding how to implement this pattern:  

-	You must pad the reverse tick value with leading zeroes to ensure the string value sorts as expected.  
-	You must be aware of the scalability targets at the level of a partition. Be careful not create hot spot partitions.  

#### When to use this pattern  

Use this pattern when you need to access entities in reverse date/time order or when you need to access the most recently added entities.  

#### Related patterns and guidance  

The following patterns and guidance may also be relevant when implementing this pattern:  

-	[Prepend / append anti-pattern](#prepend-append-anti-pattern)  
-	[Retrieving entities](#retrieving-entities)  

### High volume delete pattern  

Enable the deletion of a high volume of entities by storing all the entities for simultaneous deletion in their own separate table; you delete the entities by deleting the table.  

#### Context and problem  

Many applications delete old data which no longer needs to be available to a client application, or that the application has archived to another storage medium. You typically identify such data by a date: for example, you have a requirement to delete records of all login requests that are more than 60 days old.  

One possible design is to use the date and time of the login request in the **RowKey**:  

![][21]

This approach avoids partition hotspots because the application can insert and delete login entities for each user in a separate partition. However, this approach may be costly and time consuming if you have a large number of entities because first you need to perform a table scan in order to identify all the entities to delete, and then you must delete each old entity. Note that you can reduce the number of round trips to the server required to delete the old entities by batching multiple delete requests into EGTs.  

#### Solution  

Use a separate table for each day of login attempts. You can use the entity design above to avoid hotspots when you are inserting entities, and deleting old entities is now simply a question of deleting one table every day (a single storage operation) instead of finding and deleting hundreds and thousands of individual login entities every day.  

#### Issues and considerations  

Consider the following points when deciding how to implement this pattern:  

-	Does your design support other ways your application will use the data such as looking up specific entities, linking with other data, or generating aggregate information?  
-	Does your design avoid hot spots when you are inserting new entities?  
-	Expect a delay if you want to reuse the same table name after deleting it. It's better to always use unique table names.  
-	Expect some throttling when you first use a new table while the Table service learns the access patterns and distributes the partitions across nodes. You should consider how frequently you need to create new tables.  

#### When to use this pattern  

Use this pattern when you have a high volume of entities that you must delete at the same time.  

#### Related patterns and guidance  

The following patterns and guidance may also be relevant when implementing this pattern:  

-	[Entity Group Transactions](#entity-group-transactions)
-	[Modifying entities](#working-with-heterogeneous-entity-types)  

### Data series pattern  

Store complete data series in a single entity to minimize the number of requests you make.  

#### Context and problem  

A common scenario is for an application to store a series of data that it typically needs to retrieve all at once. For example, your application might record how many IM messages each employee sends every hour, and then use this information to plot how many messages each user sent over the preceding 24 hours. One design might be to store 24 entities for each employee:  

![][22]

With this design, you can easily locate and update the entity to update for each employee whenever the application needs to update the message count value. However, to retrieve the information to plot a chart of the activity for the preceding 24 hours, you must retrieve 24 entities.  

#### Solution  

Use the following design with a separate property to store the message count for each hour:  

![][23]

With this design, you can use a merge operation to update the message count for an employee for a specific hour. Now, you can retrieve all the information you need to plot the chart using a request for a single entity.  

#### Issues and considerations  

Consider the following points when deciding how to implement this pattern:  
-	If your complete data series does not fit into a single entity (an entity can have up to 252 properties), use an alternative data store such as a blob.  
-	If you have multiple clients updating an entity simultaneously, you will need to use the **ETag** to implement optimistic concurrency. If you have many clients, you may experience high contention.  

#### When to use this pattern  

Use this pattern when you need to update and retrieve a data series associated with an individual entity.  

#### Related patterns and guidance  

The following patterns and guidance may also be relevant when implementing this pattern:  

-	[Large entity pattern](#large-entity-pattern)  
-	[Merge or replace](#working-with-heterogeneous-entity-types)  
-	[Eventually consistent transactions pattern](#eventually-consistent-transactions-pattern) (if you are storing the data series in a blob)  

### Wide entities pattern  

Use multiple physical entities to store logical entities with more than 252 properties.  

#### Context and problem  

An individual entity can have no more than 252 properties (excluding the mandatory system properties) and cannot store more than 1 MB of data in total. In a relational database, you would typically get round any limits on the size of a row by adding a new table and enforcing a 1-to-1 relationship between them.  

#### Solution  

Using the Table service, you can store multiple entities to represent a single large business object with more than 252 properties. For example, if you want to store a count of the number of IM messages sent by each employee for the last 365 days, you could use the following design that uses two entities with different schemas:  

![][24]

If you need to make a change that requires updating both entities to keep them synchronized with each other you can use an EGT. Otherwise, you can use a single merge operation to update the message count for a specific day. To retrieve all the data for an individual employee you must retrieve both entities, which you can do with two efficient requests that use both a **PartitionKey** and a **RowKey** value.  

#### Issues and considerations  

Consider the following points when deciding how to implement this pattern:  

-	Retrieving a complete logical entity involves at least two storage transactions: one to retrieve each physical entity.  

#### When to use this pattern  

Use this pattern when  need to store entities whose size or number of properties exceeds the limits for an individual entity in the Table service.  

#### Related patterns and guidance  

The following patterns and guidance may also be relevant when implementing this pattern:  

-	[Entity Group Transactions](#entity-group-transactions)
-	[Merge or replace](#working-with-heterogeneous-entity-types)

### Large entities pattern  

Use blob storage to store large property values.  

#### Context and problem  

An individual entity cannot store more than 1 MB of data in total. If one or several of your properties store values that cause the total size of your entity to exceed this value, you cannot store the entire entity in the Table service.  

#### Solution  

If your entity exceeds 1 MB in size because one or more properties contain a large amount of data, you can store data in the Blob service and then store the address of the blob in a property in the entity. For example, you can store the photo of an employee in blob storage and store a link to the photo in the **Photo** property of your employee entity:  

![][25]

#### Issues and considerations  

Consider the following points when deciding how to implement this pattern:  

-	To maintain eventual consistency between the entity in the Table service and the data in the Blob service, use the [Eventually consistent transactions pattern](#eventually-consistent-transactions-pattern) to maintain your entities.
-	Retrieving a complete entity involves at least two storage transactions: one to retrieve the entity and one to retrieve the blob data.  

#### When to use this pattern  

Use this pattern when you need to store entities whose size exceeds the limits for an individual entity in the Table service.  

#### Related patterns and guidance  

The following patterns and guidance may also be relevant when implementing this pattern:  

-	[Eventually consistent transactions pattern](#eventually-consistent-transactions-pattern)  
-	[Wide entities pattern](#large-entity-pattern)

### Prepend/append anti-pattern  

Increase scalability when you have a high volume of inserts by spreading the inserts across multiple partitions.  

#### Context and problem  

Prepending or appending entities to your stored entities typically results in the application adding new entities to the first or last partition of a sequence of partitions. In this case, all of the inserts at any given time are taking place in the same partition, creating a hotspot that prevents the table service from load balancing inserts across multiple nodes, and possibly causing your application to hit the scalability targets for partition. For example, if you have an application that logs network and resource access by employees, then an entity structure as shown below could result in the current hour’s partition becoming a hotspot if the volume of transactions reaches the scalability target for an individual partition:  

![][26]

#### Solution  

The following alternative entity structure avoids a hotspot on any particular partition as the application logs events:  

![][27]

Notice with this example how both the **PartitionKey** and **RowKey** are compound keys. The **PartitionKey** uses both the department and employee id to distribute the logging across multiple partitions.  

#### Issues and considerations  

Consider the following points when deciding how to implement this pattern:  

-	Does the alternative key structure that avoids creating hot partitions on inserts efficiently support the queries your client application makes?  
-	Does your anticipated volume of transactions mean that you are likely to reach the scalability targets for an individual partition and be throttled by the storage service?  

#### When to use this pattern  

Avoid the prepend/append anti-pattern when your volume of transactions is likely to result in throttling by the storage service when you access a hot partition.  

#### Related patterns and guidance  

The following patterns and guidance may also be relevant when implementing this pattern:  

-	[Compound key pattern](#compound-key-pattern)  
-	[Log tail pattern](#log-tail-pattern)  
-	[Modifying entities](#working-with-heterogeneous-entity-types)  

### Log data anti-pattern  

Typically, you should use the Blob service instead of the Table service to store log data.  

#### Context and problem  

A common use case for log data is to retrieve a selection of log entries for a specific date/time range: for example, you want to find all the error and critical messages that your application logged between 15:04 and 15:06 on a specific date. You do not want to use the date and time of the log message to determine the partition you save log entities to: that results in a hot partition because at any given time, all the log entities will share the same **PartitionKey** value (see the section [Prepend/append anti-pattern](#prepend-append-anti-pattern)). For example, the following entity schema for a log message results in a hot partition because the application writes all log messages to the partition for the current date and hour:  

![][28]

In this example, the **RowKey** includes the date and time of the log message to ensure that log messages are stored sorted in date/time order, and includes a message id in case multiple log messages share the same date and time.  

Another approach is to use a **PartitionKey** that ensures that the application writes messages across a range of partitions. For example, if the source of the log message provides a way to distribute messages across many partitions, you could use the following entity schema:  

![][29]

However, the problem with this schema is that to retrieve all the log messages for a specific time span you must search every partition in the table.

#### Solution  

The previous section highlighted the problem of trying to use the Table service to store log entries and suggested two, unsatisfactory, designs. One solution led to a hot partition with the risk of poor performance writing log messages; the other solution resulted in poor query performance because of the requirement to scan every partition in the table to retrieve log messages for a specific time span. Blob storage offers a better solution for this type of scenario and this is how Azure Storage Analytics stores the log data it collects.  

This section outlines how Storage Analytics stores log data in blob storage as an illustration of this approach to storing data that you typically query by range.  

Storage Analytics stores log messages in a delimited format in multiple blobs. The delimited format makes it easy for a client application to parse the data in the log message.  

Storage Analytics uses a naming convention for blobs that enables you to locate the blob (or blobs) that contain the log messages for which you are searching. For example, a blob named "queue/2014/07/31/1800/000001.log" contains log messages that relate to the queue service for the hour starting at 18:00 on 31 July 2014. The "000001" indicates that this is the first log file for this period. Storage Analytics also records the timestamps of the first and last log messages stored in the file as part of the blob’s metadata. The API for blob storage enables you locate blobs in a container based on a name prefix: to locate all the blobs that contain queue log data for the hour starting at 18:00, you can use the prefix "queue/2014/07/31/1800."  

Storage Analytics buffers log messages internally and then periodically updates the appropriate blob or creates a new one with the latest batch of log entries. This reduces the number of writes it must perform to the blob service.  

If you are implementing a similar solution in your own application, you must consider how to manage the trade-off between reliability (writing every log entry to blob storage as it happens) and cost and scalability (buffering updates in your application and writing them to blob storage in batches).  

#### Issues and considerations  

Consider the following points when deciding how to store log data:  

-	If you create a table design that avoids potential hot partitions, you may find that you cannot access your log data efficiently.  
-	To process log data, a client often needs to load many records.  
-	Although log data is often structured, blob storage may be a better solution.  

### Implementation considerations  

This section discusses some of the considerations to bear in mind when you implement the patterns described in the previous sections. Most of this section uses examples written in C# that use the Storage Client Library (version 4.3.0 at the time of writing).  

### Retrieving entities  

As discussed in the section [Design for querying](#design-for-querying)," the most efficient query is a point query. However, in some scenarios you may need to retrieve multiple entities. This section describes some common approaches to retrieving entities using the Storage Client Library.  

#### Executing a point query using the Storage Client Library  

The easiest way to execute a point query is to use the **Retrieve** table operation as shown in the following C# code snippet that retrieves an entity with a **PartitionKey** of value "Sales" and a **RowKey** of value "212":  

	TableOperation retrieveOperation =
		TableOperation.Retrieve<EmployeeEntity>("Sales", "212");
	var retrieveResult = employeeTable.Execute(retrieveOperation);
	if (retrieveResult.Result != null)
	{
    EmployeeEntity employee = (EmployeeEntity)retrieveResult.Result;
    ...
	}  

Notice how this example expects the entity it retrieves to be of type **EmployeeEntity**.  

#### Retrieving multiple entities using LINQ  

You can retrieve multiple entities by using LINQ with Storage Client Library and specifying a query with a **where** clause. To avoid a table scan, you should always include the **PartitionKey** value in the where clause, and if possible the **RowKey** value to avoid table and partition scans. The table service supports a limited set of comparison operators (greater than, greater than or equal, less than, less than or equal, equal, and not equal) to use in the where clause. The following C# code snippet finds all the employees whose last name starts with "B" (assuming that the **RowKey** stores the last name) in the sales department (assuming the **PartitionKey** stores the department name):  

	TableQuery<EmployeeEntity> employeeQuery =
  			employeeTable.CreateQuery<EmployeeEntity>();
	var query = (from employee in employeeQuery
                where employee.PartitionKey == "Sales" &&
                employee.RowKey.CompareTo("B") >= 0 &&
                employee.RowKey.CompareTo("C") < 0
                select employee).AsTableQuery();
	var employees = query.Execute();  

Notice how the query specifies both a **RowKey** and a **PartitionKey** to ensure better performance.  

The following code sample shows equivalent functionality using the fluent API (for more information about fluent APIs in general, see [Best Practices for Designing a Fluent API](http://visualstudiomagazine.com/articles/2013/12/01/best-practices-for-designing-a-fluent-api.aspx)):  

	TableQuery<EmployeeEntity> employeeQuery = new TableQuery<EmployeeEntity>().Where(
 	 TableQuery.CombineFilters(
    	TableQuery.CombineFilters(
      	TableQuery.GenerateFilterCondition(
        "PartitionKey", QueryComparisons.Equal, "Sales"),
      TableOperators.And,
      TableQuery.GenerateFilterCondition(
        "RowKey", QueryComparisons.GreaterThanOrEqual, "B")
    ),
    TableOperators.And,
    TableQuery.GenerateFilterCondition("RowKey", QueryComparisons.LessThan, "C")
 	 )
	);
	var employees = employeeTable.ExecuteQuery(employeeQuery);  


>[AZURE.NOTE] The sample nests multiple **CombineFilters** methods to include the three filter conditions.  

#### Retrieving large numbers of entities from a query  

An optimal query returns an individual entity based on a **PartitionKey** value and a **RowKey** value. However, in some scenarios you may have a requirement to return many entities from the same partition or even from many partitions.  

You should always fully test the performance of your application in such scenarios.  

A query against the table service may return a maximum of 1,000 entities at one time and may execute for a maximum of five seconds. If the result set contains more than 1,000 entities, if the query did not complete within five seconds, or if the query crosses the partition boundary, the Table service returns a continuation token to enable the client application to request the next set of entities. For more information about how continuation tokens work, see [Query Timeout and Pagination](http://msdn.microsoft.com/library/azure/dd135718.aspx) on MSDN.  

If you are using the Storage Client Library, it can automatically handle continuation tokens for you as it returns entities from the Table service. The following C# code sample using the Storage Client Library automatically handles continuation tokens if the table service returns them in a response:  

	string filter = TableQuery.GenerateFilterCondition(
  		"PartitionKey", QueryComparisons.Equal, "Sales");
	TableQuery<EmployeeEntity> employeeQuery =
  		new TableQuery<EmployeeEntity>().Where(filter);

	var employees = employeeTable.ExecuteQuery(employeeQuery);
	foreach (var emp in employees)
	{
  		...
	}  

The following C# code handles continuation tokens explicitly:  

	string filter = TableQuery.GenerateFilterCondition(
  		"PartitionKey", QueryComparisons.Equal, "Sales");
	TableQuery<EmployeeEntity> employeeQuery =
  		new TableQuery<EmployeeEntity>().Where(filter);

	TableContinuationToken continuationToken = null;

	do
	{
  		var employees = employeeTable.ExecuteQuerySegmented(
    		employeeQuery, continuationToken);
  	foreach (var emp in employees)
  	{
   	 ...
  	}
  	continuationToken = employees.ContinuationToken;
	} while (continuationToken != null);  

By using continuation tokens explicitly, you can control when your application retrieves the next segment of data. For example, if your client application enables users to page through the entities stored in a table, a user may decide not to page through all the entities retrieved by the query so your application would only use a continuation token to retrieve the next segment when the user had finished paging through all the entities in the current segment. This approach has several benefits:  

-	It enables you to limit the amount of data to retrieve from the Table service and that you move over the network.  
-	It enables you to perform asynchronous IO in .NET.  
-	It enables you to serialize the continuation token to persistent storage so you can continue in the event of an application crash.  

>[AZURE.NOTE] A continuation token typically returns a segment containing 1,000 entities, although it may be fewer. This is also the case if you limit the number of entries a query returns by using **Take** to return the first n entities that match your lookup criteria: the table service may return a segment containing fewer than n entities along with a continuation token to enable you to retrieve the remaining entities.  

The following C# code shows how to modify the number of entities returned inside a segment:  

	employeeQuery.TakeCount = 50;  

#### Server-side projection  

A single entity can have up to 255 properties and be up to 1 MB in size. When you query the table and retrieve entities, you may not need all the properties and can avoid transferring data unnecessarily (to help reduce latency and cost). You can use server-side projection to transfer just the properties you need. The following example is retrieves just the **Email** property (along with **PartitionKey**, **RowKey**, **Timestamp**, and **ETag**) from the entities selected by the query.  

	string filter = TableQuery.GenerateFilterCondition(
  		"PartitionKey", QueryComparisons.Equal, "Sales");
	List<string> columns = new List<string>() { "Email" };
	TableQuery<EmployeeEntity> employeeQuery =
  		new TableQuery<EmployeeEntity>().Where(filter).Select(columns);

	var entities = employeeTable.ExecuteQuery(employeeQuery);
	foreach (var e in entities)
	{
  		Console.WriteLine("RowKey: {0}, EmployeeEmail: {1}", e.RowKey, e.Email);
	}  

Notice how the **RowKey** value is available even though it was not included in the list of properties to retrieve.  

### Modifying entities  

The Storage Client Library enables you to modify your entities stored in the table service by inserting, deleting, and updating entities. You can use EGTs to batch multiple insert, update, and delete operations together to reduce the number of round trips required and improve the performance of your solution.  

Note that exceptions thrown when the Storage Client Library executes an EGT typically include the index of the entity that caused the batch to fail. This is helpful when you are debugging code that uses EGTs.  

You should also consider how your design affects how your client application handles concurrency and update operations.  

#### Managing concurrency  

By default, the table service implements optimistic concurrency checks at the level of individual entities for **Insert**, **Merge**, and **Delete** operations, although it is possible for a client to force the table service to bypass these checks. For more information about how the table service manages concurrency, see  [Managing Concurrency in Microsoft Azure Storage](storage-concurrency.md) on the Microsoft Azure web site.  

#### Merge or replace  

The **Replace** method of the **TableOperation** class always replaces the complete entity in the Table service. If you do not include a property in the request when that property exists in the stored entity, the request removes that property from the stored entity. Unless you want to remove a property explicitly from a stored entity, you must include every property in the request.  

You can use the **Merge** method of the **TableOperation** class to reduce the amount of data that you send to the Table service when you want to update an entity. The **Merge** method replaces any properties in the stored entity with property values from the entity included in the request, but leaves intact any properties in the stored entity that are not included in the request. This is useful if you have large entities and only need to update a small number of properties in a request.  

>[AZURE.NOTE] The **Replace** and **Merge** methods fail if the entity does not exist. As an alternative, you can use the **InsertOrReplace** and **InsertOrMerge** methods that create a new entity if it doesn't exist.  

### Working with heterogeneous entity types  

The Table service is a *schema-less* table store that means that a single table can store entities of multiple types providing great flexibility in your design. The following example illustrates a table storing both employee and department entities:  

<table>
<tr>
<th>PartitionKey</th>
<th>RowKey</th>
<th>Timestamp</th>
<th></th>
</tr>
<tr>
<td></td>
<td></td>
<td></td>
<td>
<table>
<tr>
<th>FirstName</th>
<th>LastName</th>
<th>Age</th>
<th>Email</th>
</tr>
<tr>
<td></td>
<td></td>
<td></td>
<td></td>
</tr>
</table>
</tr>
<tr>
<td></td>
<td></td>
<td></td>
<td>
<table>
<tr>
<th>FirstName</th>
<th>LastName</th>
<th>Age</th>
<th>Email</th>
</tr>
<tr>
<td></td>
<td></td>
<td></td>
<td></td>
</tr>
</table>
</tr>
<tr>
<td></td>
<td></td>
<td></td>
<td>
<table>
<tr>
<th>DepartmentName</th>
<th>EmployeeCount</th>
</tr>
<tr>
<td></td>
<td></td>
</tr>
</table>
</td>
</tr>
<tr>
<td></td>
<td></td>
<td></td>
<td>
<table>
<tr>
<th>FirstName</th>
<th>LastName</th>
<th>Age</th>
<th>Email</th>
</tr>
<tr>
<td></td>
<td></td>
<td></td>
<td></td>
</tr>
</table>
</td>
</tr>
</table>

Note that each entity must still have **PartitionKey**, **RowKey**, and **Timestamp** values, but may have any set of properties. Furthermore, there is nothing to indicate the type of an entity unless you choose to store that information somewhere. There are two options for identifying the entity type:  

-	Prepend the entity type to the **RowKey** (or possibly the **PartitionKey**). For example, **EMPLOYEE_000123** or **DEPARTMENT_SALES** as **RowKey** values.  
-	Use a separate property to record the entity type as shown in the table below.  

<table>
<tr>
<th>PartitionKey</th>
<th>RowKey</th>
<th>Timestamp</th>
<th></th>
</tr>
<tr>
<td></td>
<td></td>
<td></td>
<td>
<table>
<tr>
<th>EntityType</th>
<th>FirstName</th>
<th>LastName</th>
<th>Age</th>
<th>Email</th>
</tr>
<tr>
<td>Employee</td>
<td></td>
<td></td>
<td></td>
<td></td>
</tr>
</table>
</tr>
<tr>
<td></td>
<td></td>
<td></td>
<td>
<table>
<tr>
<th>EntityType</th>
<th>FirstName</th>
<th>LastName</th>
<th>Age</th>
<th>Email</th>
</tr>
<tr>
<td>Employee</td>
<td></td>
<td></td>
<td></td>
<td></td>
</tr>
</table>
</tr>
<tr>
<td></td>
<td></td>
<td></td>
<td>
<table>
<tr>
<th>EntityType</th>
<th>DepartmentName</th>
<th>EmployeeCount</th>
</tr>
<tr>
<td>Department</td>
<td></td>
<td></td>
</tr>
</table>
</td>
</tr>
<tr>
<td></td>
<td></td>
<td></td>
<td>
<table>
<tr>
<th>EntityType</th>
<th>FirstName</th>
<th>LastName</th>
<th>Age</th>
<th>Email</th>
</tr>
<tr>
<td>Employee</td>
<td></td>
<td></td>
<td></td>
<td></td>
</tr>
</table>
</td>
</tr>
</table>

The first option, prepending the entity type to the **RowKey**, is useful if there is a possibility that two entities of different types might have the same key value. It also groups entities of the same type together in the partition.  

The techniques discussed in this section are especially relevant to the discussion [Inheritance relationships](#inheritance-relationships) earlier in this guide in the section [Modelling relationships](#modelling-relationships).  

>[AZURE.NOTE] You should consider including a version number in the entity type value to enable client applications to evolve POCO objects and work with different versions.  

The remainder of this section describes some of the features in the Storage Client Library that facilitate working with multiple entity types in the same table.  

#### Retrieving heterogeneous entity types  

If you are using the Storage Client Library, you have three options for working with multiple entity types.  

If you know the type of the entity stored with a specific **RowKey** and **PartitionKey** values, then you can specify the entity type when you retrieve the entity as shown in the previous two examples that retrieve entities of type **EmployeeEntity**: [Retrieving a single entity using the Storage Client Library](#retrieving-a-single-entity-using-the-storage-client-library) and [Retrieving multiple entities using LINQ](#retrieving-multiple-entities-using-linq).  

The second option is to use the **DynamicTableEntity** type (a property bag) instead of a concrete POCO entity type (this option may also improve performance because there is no need to serialize and deserialize the entity to .NET types). The following C# code potentially retrieves multiple entities of different types from the table, but returns all entities as **DynamicTableEntity** instances. It then uses the **EntityType** property to determine the type of each entity:  

	string filter = 	TableQuery.CombineFilters(
    	TableQuery.GenerateFilterCondition("PartitionKey",
      QueryComparisons.Equal, "Sales"),
    	TableOperators.And,
    	TableQuery.CombineFilters(
        TableQuery.GenerateFilterCondition("RowKey",
          			QueryComparisons.GreaterThanOrEqual, "B"),
        	TableOperators.And,
        	TableQuery.GenerateFilterCondition("RowKey",
          QueryComparisons.LessThan, "F")
        )
    );
	TableQuery<DynamicTableEntity> entityQuery =
  	new TableQuery<DynamicTableEntity>().Where(filter);
	var employees = employeeTable.ExecuteQuery(entityQuery);

	IEnumerable<DynamicTableEntity> entities = employeeTable.ExecuteQuery(entityQuery);
	foreach (var e in entities)
	{
    EntityProperty entityTypeProperty;
    if (e.Properties.TryGetValue("EntityType", out entityTypeProperty))
    {
        if (entityTypeProperty.StringValue == "Employee")
        {
            // Use entityTypeProperty, RowKey, PartitionKey, Etag, and Timestamp
      	  }
   	 }
	}  

Note that to retrieve other properties you must use the **TryGetValue** method on the **Properties** property of the **DynamicTableEntity** class.  

A third option is to combine using the **DynamicTableEntity** type and an **EntityResolver** instance. This enables you to resolve to multiple POCO types in the same query. In this example, the **EntityResolver** delegate is using the **EntityType** property to distinguish between the two types of entity that the query returns. The **Resolve** method uses the **resolver** delegate to resolve **DynamicTableEntity** instances to **TableEntity** instances.  

	EntityResolver<TableEntity> resolver = (pk, rk, ts, props, etag) =>
	{

  		TableEntity resolvedEntity = null;
  		if (props["EntityType"].StringValue == "Department")
  		{
    		resolvedEntity = new DepartmentEntity();
  		}
  		else if (props["EntityType"].StringValue == "Employee")
  		{
    		resolvedEntity = new EmployeeEntity();
  		}
  		else throw new ArgumentException("Unrecognized entity", "props");

  		resolvedEntity.PartitionKey = pk;
  		resolvedEntity.RowKey = rk;
  		resolvedEntity.Timestamp = ts;
  		resolvedEntity.ETag = etag;
  		resolvedEntity.ReadEntity(props, null);
  		return resolvedEntity;
	};

	string filter = TableQuery.GenerateFilterCondition(
  		"PartitionKey", QueryComparisons.Equal, "Sales");
	TableQuery<DynamicTableEntity> entityQuery =
  		new TableQuery<DynamicTableEntity>().Where(filter);

	var entities = employeeTable.ExecuteQuery(entityQuery, resolver);
	foreach (var e in entities)
	{
  		if (e is DepartmentEntity)
  		{
    	...
  		}
  		if (e is EmployeeEntity)
  		{
    	...
  		}
	}  

#### Modifying heterogeneous entity types  

You do not need to know the type of an entity to delete it, and you always know the type of an entity when you insert it. However, you can use **DynamicTableEntity** type to update an entity without knowing its type and without using a POCO entity class. The following code sample retrieves a single entity, and checks the **EmployeeCount** property exists before updating it.  

	TableResult result =
  		employeeTable.Execute(TableOperation.Retrieve(partitionKey, rowKey));
	DynamicTableEntity department = (DynamicTableEntity)result.Result;

	EntityProperty countProperty;

	if (!department.Properties.TryGetValue("EmployeeCount", out countProperty))
	{
  		throw new
    		InvalidOperationException("Invalid entity, EmployeeCount property not found.");
	}
	countProperty.Int32Value += 1;
	employeeTable.Execute(TableOperation.Merge(department));  

### Controlling access with Shared Access Signatures  

You can use Shared Access Signature (SAS) tokens to enable client applications to modify (and query) table entities directly without the need to authenticate directly with the table service. Typically, there are three main benefits to using SAS in your application:  

-	You do not need to distribute your storage account key to an insecure platform (such as a mobile device) in order to allow that device to access and modify entities in the Table service.  
-	You can offload some of the work that web and worker roles perform in managing your entities to client devices such as end-user computers and mobile devices.  
-	You can assign a constrained and time limited set of permissions to a client (such as allowing read-only access to specific resources).  

For more information about using SAS tokens with the Table service, see [Shared Access Signatures, Part 1: Understanding the SAS Model](../storage-dotnet-shared-access-signature-part-1/).  

However, you must still generate the SAS tokens that grant a client application to the entities in the table service: you should do this in an environment that has secure access to your storage account keys. Typically, you use a web or worker role to generate the SAS tokens and deliver them to the client applications that need access to your entities. Because there is still an overhead involved in generating and delivering SAS tokens to clients, you should consider how best to reduce this overhead, especially in high-volume scenarios.  

It is possible to generate a SAS token that grants access to a subset of the entities in a table. By default, you create a SAS token for an entire table, but it is also possible to specify that the SAS token grant access to either a range of **PartitionKey** values, or a range of **PartitionKey** and **RowKey** values. You might choose to generate SAS tokens for individual users of your system such that each user’s SAS token only allows them access to their own entities in the table service.  

### Asynchronous and parallel operations  

Provided you are spreading your requests across multiple partitions, you can improve throughput and client responsiveness by using asynchronous or parallel queries.
For example, you might have two or more worker role instances accessing your tables in parallel. You could have individual worker roles responsible for particular sets of partitions, or simply have multiple worker role instances, each able to access all the partitions in a table.  

Within a client instance, you can improve throughput by executing storage operations asynchronously. The Storage Client Library makes it easy to write asynchronous queries and modifications. For example, you might start with the synchronous method that retrieves all the entities in a partition as shown in the following C# code:  

	private static void ManyEntitiesQuery(CloudTable employeeTable, string department)
	{
  		string filter = TableQuery.GenerateFilterCondition(
    		"PartitionKey", QueryComparisons.Equal, department);
  		TableQuery<EmployeeEntity> employeeQuery =
    		new TableQuery<EmployeeEntity>().Where(filter);

  		TableContinuationToken continuationToken = null;

  		do
  		{
    		var employees = employeeTable.ExecuteQuerySegmented(
      			employeeQuery, continuationToken);
    		foreach (var emp in employees)
    	{
      	...
    	}
    		continuationToken = employees.ContinuationToken;
  		} while (continuationToken != null);
	}  

You can easily modify this code so that the query runs asynchronously as follows:  

	private static async Task ManyEntitiesQueryAsync(CloudTable employeeTable, string department)
	{
  		string filter = TableQuery.GenerateFilterCondition(
    		"PartitionKey", QueryComparisons.Equal, department);
  		TableQuery<EmployeeEntity> employeeQuery =
    		new TableQuery<EmployeeEntity>().Where(filter);
  		TableContinuationToken continuationToken = null;

  		do
  		{
    		var employees = await employeeTable.ExecuteQuerySegmentedAsync(
      			employeeQuery, continuationToken);
    		foreach (var emp in employees)
    		{
     		 ...
    		}
    		continuationToken = employees.ContinuationToken;
  			} while (continuationToken != null);
	}  

In this asynchronous example, you can see the following changes from the synchronous version:  

-	The method signature now includes the **async** modifier and returns a **Task** instance.  
-	Instead of calling the **ExecuteSegmented** method to retrieve results, the method now calls the **ExecuteSegmentedAsync** method and uses the **await** modifier to retrieve results asynchronously.  

The client application can call this method multiple times (with different values for the **department** parameter), and each query will run on a separate thread.  

Note that there is no asynchronous version of the **Execute** method in the **TableQuery** class because the **IEnumerable** interface does not support asynchronous enumeration.  

You can also insert, update, and delete entities asynchronously. The following C# example shows a simple, synchronous method to insert or replace an employee entity:  

	private static void SimpleEmployeeUpsert(CloudTable employeeTable,
  		EmployeeEntity employee)
	{
  		TableResult result = employeeTable
    		.Execute(TableOperation.InsertOrReplace(employee));
  		Console.WriteLine("HTTP Status: {0}", result.HttpStatusCode);
	}  

You can easily modify this code so that the update runs asynchronously as follows:  

	private static async Task SimpleEmployeeUpsertAsync(CloudTable employeeTable,
  		EmployeeEntity employee)
	{
  		TableResult result = await employeeTable
    		.ExecuteAsync(TableOperation.InsertOrReplace(employee));
  		Console.WriteLine("HTTP Status: {0}", result.HttpStatusCode);
	}  

In this asynchronous example, you can see the following changes from the synchronous version:  

-	The method signature now includes the **async** modifier and returns a **Task** instance.  
-	Instead of calling the **Execute** method to update the entity, the method now calls the **ExecuteAsync** method and uses the **await** modifier to retrieve results asynchronously.  

The client application can call multiple asynchronous methods like this one, and each method invocation will run on a separate thread.  


### Credits
We would like to thank the following members of the Azure team for their contributions: Dominic Betts, Jason Hogg, Jean Ghanem, Jai Haridas, Jeff Irwin, Vamshidhar Kommineni, Vinay Shah and Serdar Ozler as well as  Tom Hollander from Microsoft DX. 

We would also like to thank the following Microsoft MVP’s for their valuable feedback during review cycles: Igor Papirov and Edward Bakker.



[1]: ./media/storage-table-design-guide/storage-table-design-IMAGE01.png
[2]: ./media/storage-table-design-guide/storage-table-design-IMAGE02.png
[3]: ./media/storage-table-design-guide/storage-table-design-IMAGE03.png
[4]: ./media/storage-table-design-guide/storage-table-design-IMAGE04.png
[5]: ./media/storage-table-design-guide/storage-table-design-IMAGE05.png
[6]: ./media/storage-table-design-guide/storage-table-design-IMAGE06.png
[7]: ./media/storage-table-design-guide/storage-table-design-IMAGE07.png
[8]: ./media/storage-table-design-guide/storage-table-design-IMAGE08.png
[9]: ./media/storage-table-design-guide/storage-table-design-IMAGE09.png
[10]: ./media/storage-table-design-guide/storage-table-design-IMAGE10.png
[11]: ./media/storage-table-design-guide/storage-table-design-IMAGE11.png
[12]: ./media/storage-table-design-guide/storage-table-design-IMAGE12.png
[13]: ./media/storage-table-design-guide/storage-table-design-IMAGE13.png
[14]: ./media/storage-table-design-guide/storage-table-design-IMAGE14.png
[15]: ./media/storage-table-design-guide/storage-table-design-IMAGE15.png
[16]: ./media/storage-table-design-guide/storage-table-design-IMAGE16.png
[17]: ./media/storage-table-design-guide/storage-table-design-IMAGE17.png
[18]: ./media/storage-table-design-guide/storage-table-design-IMAGE18.png
[19]: ./media/storage-table-design-guide/storage-table-design-IMAGE19.png
[20]: ./media/storage-table-design-guide/storage-table-design-IMAGE20.png
[21]: ./media/storage-table-design-guide/storage-table-design-IMAGE21.png
[22]: ./media/storage-table-design-guide/storage-table-design-IMAGE22.png
[23]: ./media/storage-table-design-guide/storage-table-design-IMAGE23.png
[24]: ./media/storage-table-design-guide/storage-table-design-IMAGE24.png
[25]: ./media/storage-table-design-guide/storage-table-design-IMAGE25.png
[26]: ./media/storage-table-design-guide/storage-table-design-IMAGE26.png
[27]: ./media/storage-table-design-guide/storage-table-design-IMAGE27.png
[28]: ./media/storage-table-design-guide/storage-table-design-IMAGE28.png
[29]: ./media/storage-table-design-guide/storage-table-design-IMAGE29.png
