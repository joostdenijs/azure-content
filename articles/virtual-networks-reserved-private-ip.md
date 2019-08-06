<properties 
   pageTitle="How to set a static internal private IP"
   description="Understanding static internal IPs (DIPs) and how to manage them"
   services="virtual-network"
   documentationCenter="na"
   authors="telmosampaio"
   manager="adinah"
   editor="tysonn" />
<tags 
   ms.service="virtual-network"
   ms.devlang="na"
   ms.topic="article"
   ms.tgt_pltfrm="na"
   ms.workload="infrastructure-services"
   ms.date="04/22/2015"
   ms.author="telmos" />

# How to Set a Static Internal Private IP
In most cases, you won’t need to specify a static internal IP address for your virtual machine. VMs in a virtual network will automatically receive an internal IP address from a range that you specify. But in certain cases, specifying a static IP address for a particular VM makes sense. For example, if your VM is going to run DNS or will be a domain controller. 

>[AZURE.NOTE] A static internal IP address stays with the VM even through a stop/deprovision state. 

## How to verify if a specific IP address is available
To verify if the IP address *10.0.0.7* is available in a vnet named *TestVnet*, run the following PowerShell command and verify the value for *IsAvailable*:

	Test-AzureStaticVNetIP –VNetName TestVNet –IPAddress 10.0.0.7 

	IsAvailable          : True
	AvailableAddresses   : {}
	OperationDescription : Test-AzureStaticVNetIP
	OperationId          : fd3097e1-5f4b-9cac-8afa-bba1e3492609
	OperationStatus      : Succeeded

>[AZURE.NOTE] If you want to test the command above in a safe environment follow the guidelines in [Create a Virtual Network](https://msdn.microsoft.com/library/azure/dn631643.aspx) to create a vnet named *TestVnet* and ensure it uses the *10.0.0.0/8* address space.

## How to specify a static internal IP when creating a VM
The PowerShell script below creates a new cloud service named *TestService*, then retrieves an image from Azure, then creates a VM named *TestVM* in the new cloud service using the retrieved image, sets the VM to be in a subnet named *Subnet-1*, and sets *10.0.0.7* as a static internal IP for the VM:

	New-AzureService -ServiceName TestService -Location "Central US"
	$image = Get-AzureVMImage|?{$_.ImageName -like "*RightImage-Windows-2012R2-x64*"}
	New-AzureVMConfig -Name TestVM -InstanceSize Small -ImageName $image.ImageName `
	| Add-AzureProvisioningConfig -Windows -AdminUsername adminuser -Password MyP@ssw0rd!! `
	| Set-AzureSubnet –SubnetNames Subnet-1 `
	| Set-AzureStaticVNetIP -IPAddress 10.0.0.7 `
	| New-AzureVM -ServiceName "TestService" –VNetName TestVnet

## How to retrieve static internal IP information for a VM
To view the static internal IP information for the VM created with the script above, run the following PowerShell command and observe the values for *IpAddress*:

	Get-AzureVM -Name TestVM -ServiceName TestService

	DeploymentName              : TestService
	Name                        : TestVM
	Label                       : 
	VM                          : Microsoft.WindowsAzure.Commands.ServiceManagement.Model.PersistentVM
	InstanceStatus              : Provisioning
	IpAddress                   : 10.0.0.7
	InstanceStateDetails        : Windows is preparing your computer for first use...
	PowerState                  : Started
	InstanceErrorCode           : 
	InstanceFaultDomain         : 0
	InstanceName                : TestVM
	InstanceUpgradeDomain       : 0
	InstanceSize                : Small
	HostName                    : rsR2-797
	AvailabilitySetName         : 
	DNSName                     : http://testservice000.cloudapp.net/
	Status                      : Provisioning
	GuestAgentStatus            : Microsoft.WindowsAzure.Commands.ServiceManagement.Model.GuestAgentStatus
	ResourceExtensionStatusList : {Microsoft.Compute.BGInfo}
	PublicIPAddress             : 
	PublicIPName                : 
	NetworkInterfaces           : {}
	ServiceName                 : TestService
	OperationDescription        : Get-AzureVM
	OperationId                 : 34c1560a62f0901ab75cde4fed8e8bd1
	OperationStatus             : OK

## How to remove a static internal IP from a VM
To remove the static internal IP added to the VM in the script above, run the following PowerShell command:
	
	Get-AzureVM -ServiceName TestService -Name TestVM `
	| Remove-AzureStaticVNetIP `
	| Update-AzureVM

## How to add a static internal IP to an existing VM
To add a static internal IP to the VM created using the script above, runt he following command:

	Get-AzureVM -ServiceName TestService000 -Name TestVM `
	| Set-AzureStaticVNetIP -IPAddress 10.10.0.7 `
	| Update-AzureVM

## See Also

[Reserved Public IP](../virtual-networks-reserved-public-ip)

[Instance-Level Public IP (ILPIP)](../virtual-networks-instance-level-public-ip)

[Virtual Network Overview](https://msdn.microsoft.com/library/azure/jj156007.aspx)

[Reserved IP REST APIs](https://msdn.microsoft.com/library/azure/dn722420.aspx)
