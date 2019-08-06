<properties 
 pageTitle="Plans and Billing in Azure Scheduler" 
 description="" 
 services="scheduler" 
 documentationCenter=".NET" 
 authors="krisragh" 
 manager="dwrede" 
 editor=""/>
<tags 
 ms.service="scheduler" 
 ms.workload="infrastructure-services" 
 ms.tgt_pltfrm="na" 
 ms.devlang="dotnet" 
 ms.topic="article" 
 ms.date="05/12/2015" 
 ms.author="krisragh"/>
 
# Plans and Billing in Azure Scheduler

## Job Collection Plans

Job collections are the billable entity in Azure Scheduler. Job collections contain a number of jobs and come in three plans – Free, Standard, and Premium – that are described below.

|**Job Collection Plan**|**Max # of Jobs per Job Collection**|**Max Recurrence**|**Max Job Collections per Subscription**|**Limits**|
|:---|:---|:---|:---|:---|
|**Free**|5 jobs per job collection|Once per hour. Cannot execute jobs more often than once an hour|A subscription is allowed up to 1 free job collection|Cannot use [HTTP outbound authorization object](scheduler-outbound-authentication.md)
|**Standard**|50 jobs per job collection|Once per minute. Cannot execute jobs more often than once a minute|A subscription is allowed up to 100 standard job collections|Access to full feature set of Scheduler|
|**Premium**|50 jobs per job collection|Once per minute. Cannot execute jobs more often than once a minute|No limit on number of premium job collections in a subscription|Access to full feature set of Scheduler|

## Upgrades and Downgrades of Job Collection Plans

You may upgrade or downgrade a job collection plan anytime among the Free, Standard, and Premium plans. However, when downgrading to a free job collection, the downgrade may fail for one of the following reasons:

- A free job collection already exists in the subscription
- A job in the job collection has a higher recurrence than allowed for jobs in free job collections. The maximum recurrence allowed in a free job collection is once per hour
- There are more than 5 jobs in the job collection
- A job in the job collection has an HTTP or HTTPS action that uses an [HTTP outbound authorization object](scheduler-outbound-authentication.md)

## Billing and Azure Plans

Subscriptions are not charged for free job collections. If you have more than 100 standard job collections (10 standard billing units), then it's a better deal to have all job collections in the premium plan.

If you have one standard job collection and one premium job collection, you are billed one standard billing unit _and_ one premium billing unit. The Scheduler service bills based on the number of active job collections that are set to either standard or premium; this is explained further in the next two sections.

## Standard Billable Units

A standard billable unit can include up to 10 standard job collections. Since a standard job collection can have up to 50 jobs per job collection, one standard billing unit allows a subscription to have up to 500 jobs – up to almost 22 million job executions per month.

If you have between 1 and 10 standard job collections, you'll be billed for 1 standard billing unit. If you have between 11 and 20 standard job collections, you'll be billed for 2 standard billing units. If you have between 21 and 30 standard job collections, you'll be billed for 3 standard billing units, and so on.

## Premium Billable Units

A premium billable unit can include up to 10,000 premium job collections. Since a premium job collection can have up to 50 jobs per job collection, one premium billing unit allows a subscription to have up to 500,000 jobs – up to almost 22 billion job executions per month.

If you have between 1 and 10,000 premium job collections, you'll be billed for 1 premium billing unit. If you have between 10,001 and 20,000 premium job collections, you'll be billed for 2 premium billing units, and so on.

Thus, premium job collections have the same functionality as the standard job collections but provide a price break in case your application requires a lot of job collections. 

## Billing and Active Status

Job collections are always active unless your entire subscription has gone into some temporary disabled state due to billing issues. The only way to ensure that a job collection is not billed is to either set it to the _Free_ plan or to delete the job collection.

Although you may disable all jobs within a job collection in a single operation, it does not change the billing status of the job collection – the job collection will _still_ be billed. Similarly, empty job collections are considered active and will be billed. 

## Pricing

For pricing details, please see [Scheduler Pricing](http://azure.microsoft.com/pricing/details/scheduler/).

## See Also
 
 [What is Scheduler?](scheduler-intro.md)
 
 [Scheduler Concepts, Terminology, and Entity Hierarchy](scheduler-concepts-terms.md)

 [Get Started Using Scheduler in the Management Portal](scheduler-get-started-portal.md)

 [How to Build Complex Schedules and Advanced Recurrence with Azure Scheduler](scheduler-advanced-complexity.md)

 [Scheduler REST API Reference](https://msdn.microsoft.com/library/dn528946)   

 [Scheduler PowerShell Cmdlets Reference](scheduler-powershell-reference.md)
 
 [Scheduler High-Availability and Reliability](scheduler-high-availability-reliability.md)

 [Scheduler Limits, Defaults, and Error Codes](scheduler-limits-defaults-errors.md)

 [Scheduler Outbound Authentication](scheduler-outbound-authentication.md)
 