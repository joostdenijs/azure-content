<properties
	pageTitle="Manage Azure Storage using Azure Automation"
	description="Learn about how the Azure Automation service can be used to manage Azure Storage at scale."
	services="storage, automation"
	documentationCenter=""
	authors="jodoglevy"
	manager="eamono"
	editor=""/>

<tags
	ms.service="storage"
	ms.workload="storage"
	ms.tgt_pltfrm="na"
	ms.devlang="na"
	ms.topic="article"
	ms.date="02/20/2015"
	ms.author="jolevy"/>



#Managing Azure Storage using Azure Automation

This guide will introduce you to the Azure Automation service, and how it can be used to simplify management of your Azure Storage blobs, tables, and queues.


## What is Azure Automation?

[Azure Automation](http://azure.microsoft.com/services/automation/) is an Azure service for simplifying cloud management through process automation. Using Azure Automation, long-running, manual, error-prone, and frequently repeated tasks can be automated to increase reliability, efficiency, and time to value for your organization.

Azure Automation provides a highly-reliable and highly-available workflow execution engine that scales to meet your needs as your organization grows. In Azure Automation, processes can be kicked off manually, by 3rd-party systems, or at scheduled intervals so that tasks happen exactly when needed.

Lower operational overhead and free up IT / DevOps staff to focus on work that adds business value by moving your cloud management tasks to be run automatically by Azure Automation.


## How can Azure Automation help manage Azure Storage?

Azure Storage can be managed in Azure Automation by using the PowerShell cmdlets that are available in the [Azure PowerShell tools](https://msdn.microsoft.com/library/azure/jj156055.aspx). Azure Automation has these Storage PowerShell cmdlets available out of the box, so that you can perform all of your blob, table, and queue management tasks within the service. You can also pair these cmdlets in Azure Automation with the cmdlets for other Azure services, to automate complex tasks across Azure services and 3rd party systems.

The [Azure Automation runbook gallery](http://azure.microsoft.com/blog/2014/10/07/introducing-the-azure-automation-runbook-gallery/) contains a variety of product team and community runbooks to get started automating management of Azure Storage, other Azure services, and 3rd party systems. Gallery runbooks include:

 * [Remove Azure Storage blobs older than X days old](https://gallery.technet.microsoft.com/scriptcenter/Remove-Storage-Blobs-that-aae4b761)
 * [Download a blob from Azure Storage to Azure Automation](https://gallery.technet.microsoft.com/scriptcenter/a-Blob-from-Azure-Storage-6bc13745)
 * [Create copies of Azure VM data disks in an Azure Cloud Service](https://gallery.technet.microsoft.com/scriptcenter/Make-copies-of-Azure-VM-065a6394)


## Next Steps

Now that you've learned the basics of Azure Automation and how it can be used to manage Azure Storage blobs, tables, and queues, follow these links to learn more about Azure Automation.

See the Azure Automation [Getting Started Tutorial](automation-create-runbook-from-samples.md)
