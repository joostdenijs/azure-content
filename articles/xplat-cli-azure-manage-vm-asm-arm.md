<properties
	pageTitle="Equivalent Resource Manager and Service Management Commands for VM Operations with the Azure CLI for Mac, Linux, and Windows"
	description="Shows equivalent Azure CLI commands to create and manage Azure VMs in Resource Manager and Service Management modes"
	services="virtual-machines"
	documentationCenter=""
	authors="dlepow"
	manager="timlt"
	editor=""/>

<tags
	ms.service="virtual-machines"
	ms.devlang="na"
	ms.topic="article"
	ms.tgt_pltfrm="command-line-interface"
	ms.workload="infrastructure-services"
	ms.date="04/28/2015"
	ms.author="danlep"/>


# Equivalent Resource Manager and Service Management Commands for VM Operations with the Azure CLI for Mac, Linux, and Windows
This article shows equivalent Azure command-line interface (Azure CLI) commands to create and manage Azure VMs in Service Management (asm) mode and Resource Manager (arm) mode. Use this as a handy guide to migrate scripts from one command mode to the other.

* If you haven't already installed the Azure CLI and connected to your subscription, see [Install the Azure CLI for Mac, Linux, and Windows](xplat-cli-install.md) and [Connect to an Azure subscription from the Azure CLI for Mac, Linux, and Windows](xplat-cli-connect.md). Be sure to connect with the login method when you use the arm mode commands.

* To get started with arm mode in the Azure CLI and switch command modes, see [Using the Azure Command-Line Interface with the Resource Manager](xplat-cli-azure-resource-manager.md).

* For online command help and options, type `azure <command> <subcommand> --help` or `azure help <command> <subcommand>`.

## Scenarios
Following are common VM operations you can perform with Azure CLI commands in asm and arm mode. With many arm mode commands you need to pass the name of an existing resource group.

> [AZURE.NOTE] These examples don't include template-based operations in arm mode. For information see [Using the Azure Command-Line Interface with the Resource Manager](xplat-cli-azure-resource-manager.md).

Scenario | asm mode | arm mode
-------------- | ----------- | -------------------------
Create the most basic VM | `azure vm create [options] <dns-name> <image> [userName] [password]` | `azure vm quick-create [options] <resource-group> <name> <location> <os-type> <image-urn> <admin-username> <admin-password>`
Create a Linux VM | `azure vm create [options] <dns-name> <image> [userName] [password]` | `azure  vm create [options] <resource-group> <name> <location> -y "Linux"`
Create a Windows VM | `azure vm create [options] <dns-name> <image> [userName] [password]` | `azure  vm create [options] <resource-group> <name> <location> -y "Windows"`
List VMs | `azure  vm list [options]` | `azure  vm list [options] <resource_group>`
Get information about a VM | `azure  vm show [options] <vm_name>` | `azure  vm show [options] <resource_group> <name>`
Start a VM | `azure vm start [options] <name>` | `azure vm start [options] <resource_group> <name>`
Stop a VM | `azure vm shutdown [options] <name>` | `azure vm stop [options] <resource_group> <name>`
Restart a VM | `azure vm restart [options] <vname>` | `azure vm restart [options] <resource_group> <name>`
Delete a VM | `azure vm delete [options] <name>` | `azure vm delete [options] <resource_group> <name>`
Capture a VM | `azure vm capture [options] <name>` | `azure vm capture [options] <resource_group> <name>`
Create a VM from a user image | `azure  vm create [options] <dns-name> <image> [userName] [password]` | `azure  vm create [options] –q <image-name> <resource-group> <name> <location> <os-type>`
Create a VM from a specialized disk | `azure  vm create [options]-d <custom-data-file> <dns-name> [userName] [password]` | `azue  vm create [options] –d <os-disk-vhd> <resource-group> <name> <location> <os-type>`
Add a data disk to a VM | `azure  vm disk attach [options] <vm-name> <disk-image-name>` -OR- <br/>  `vm disk attach-new [options] <vm-name> <size-in-gb> [blob-url]` | `azure  vm disk attach-new [options] <resource-group> <vm-name> <size-in-gb> [vhd-name]`
Remove a data disk from a VM | `azure  vm disk detach [options] <vm-name> <lun>` | `azure  vm disk detach [options] <resource-group> <vm-name> <lun>`
Add a generic extension to a VM | `azure  vm extension set [options] <vm-name> <extension-name> <publisher-name> <version>` | `azure  vm extension set [options] <resource-group> <vm-name> <name> <publisher-name> <version>`
Add VM Access extension to a VM | not available | `azure vm reset-access [options] <resource-group> <name>`
Add Docker extension to a VM | `azure  vm docker create [options] <dns-name> <image> <user-name> [password]` | `azure  vm docker create [options] <resource-group> <name> <location> <os-type>`
Add Chef extension to a VM | `azure  vm extension get-chef [options] <vm-name>` | not available
Disable a VM extension | `azure  vm extension set [options] –b <vm-name> <extension-name> <publisher-name> <version>` | not available
Remove a VM extension | `azure  vm extension set [options] –u <vm-name> <extension-name> <publisher-name> <version>` | `azure  vm extension set [options] –u <resource-group> <vm-name> <name> <publisher-name> <version>`
List VM extensions | `azure vm extension list [options]` | `azure  vm extension get [options] <resource-group> <vm-name>`
List VM images | `azure vm image list [options]` | `azure vm image list [options] <location> <publisher> [offer] [sku]` -OR- <br/> `azure vm image list-publishers [options] <location>` -OR- <br/> `azure vm image list-offers [options] <location>` -OR- <br/> `azure vm image list-skus [options] <location>`
Show a VM image | `azure vm image show [options]` | not available


## Next steps

* For more about using the Azure CLI to work with resources in arm mode, see [Using the Azure Command-Line Interface with the Resource Manager](xplat-cli-azure-resource-manager.md) and [Managing Role-Based Access Control with the Azure Command-Line Interface](role-based-access-control-xplat-cli.md).
* For additional examples of the CLI commands, see [Using the Azure Command-Line Interface for Mac, Linux, and Windows](virtual-machines-command-line-tools.md) and
[Using the Azure CLI for Mac, Linux, and Windows with Azure Resource Manager](azure-cli-arm-commands.md).
