<properties 
   pageTitle="Reserved Public IP"
   description="Understand reserved IPs, VIP, ILPIP, and how to manage them"
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

# Reserved Public IP Overview
IP addresses in Azure fall into two categories: dynamic and reserved. Public IP addresses managed by Azure are dynamic by default. That means that the IP address used for a given cloud service (VIP) or to access a VM or role instance directly (ILPIP) can change from time to time, including when resources are restarted or allocated.

To prevent IP addresses from changing, you can reserve an IP address. Reserved IPs can be used as a VIP or ILPIP, ensuring that the IP address for the resource will be the same even as resources are restarted or allocated. Furthermore, you can convert existing dynamic IPs used as a VIP or ILPIP to a reserved IP address.

![Difference between ILPIP and VIP](./media/virtual-networks-reserved-public-ip/Figure1.png)


As shown in Figure 1, the cloud service is accessed using a VIP, while the individual VMs are normally accessed using VIP:&lt;port number&gt;. By assigning an ILPIP to a specific VM, that VM can be accessed directly using that ILPIP.

## When do I need a reserved IP?
- **You want to ensure that the IP is reserved in your subscription**. If you want to reserve an IP address that will not be released from your subscription under any circumstance, you should use a reserved public IP.  
- **You want your IP to stay with your cloud service even across stopped or deallocated state (VMs)**. If you want your service to be accessed by using an IP address that will not change even when VMs in the cloud service are stop or deallocated.
- **You want to ensure that outbound traffic from Azure uses a predictable IP address**. You may have your on-premises firewall configured to allow only traffic from specific IP addresses. By reserving an IP, you will know the source IP address and won’t have to update your firewall rules due to an IP change.

## FAQ
1. Can I use a reserved IP for all Azure services?  
  - Reserved IPs can only be used for VMs and cloud service instance roles.
1. How many reserved IPs can I have?  
  - At this time, all Azure subscriptions are authorized to use 20 reserved IPs. However, you can request additional reserved IPs. See the [Subscription and Service Limits](../azure-subscription-service-limits/) page for more information.
1. Is there a charge for reserved IPs? 
  - See [Reserved IP Address Pricing Details](http://go.microsoft.com/fwlink/?LinkID=398482) for pricing information.
1. How many reserved IPs can I assign to a cloud service? 
  - Only one IP address is allowed is per cloud service, whether or not that IP address is reserved. Reserving an IP address does not allow you to add additional IP addresses to a cloud service.
1. How do I reserve an IP address? 
  - You can use PowerShell or the [Azure Management REST API](https://msdn.microsoft.com/library/azure/dn722420.aspx) to request a reserved IP from a particular region. Azure will reserve an IP address from that region and correlate it to your subscription. You can then use the reserved IP in that region. You cannot reserve an IP address by using the Management Portal.
1. Can I use this with affinity group based VNets? 
  - Reserved IPs are only supported in regional VNets. It is not supported for VNets that are associated with affinity groups. For more information about associating a VNet with a region or an affinity group, see [About Regional VNets and Affinity Groups](https://msdn.microsoft.com/library/azure/jj156085.aspx). 

## How to manage reserved VIPs

Before you can use reserved IPs, you must add it to your subscription. To create a reserved IP from the pool of public IP addresses available in the *Central US* location, run the following PowerShell command:

	New-AzureReservedIP –ReservedIPName MyReservedIP –Location “Central US”

Notice, however, that you cannot specify what IP is being reserved. To view what IP addresses are reserved in your subscription, run the following PowerShell command, and notice the values for *ReservedIPName* and *Address*:

	Get-AzureReservedIP

	ReservedIPName       : MyReservedIP
	Address              : 23.101.114.211
	Id                   : d73be9dd-db12-4b5e-98c8-bc62e7c42041
	Label                : 
	Location             : Central US
	State                : Created
	InUse                : False
	ServiceName          : 
	DeploymentName       : 
	OperationDescription : Get-AzureReservedIP
	OperationId          : 55e4f245-82e4-9c66-9bd8-273e815ce30a
	OperationStatus      : Succeeded

Once an IP is reserved, it remains associated to your subscription until you delete it. To delete the reserved IP shown above, run the following PowerShell command:

	Remove-AzureReservedIP -ReservedIPName "MyReservedIP"

## How to associate a reserved IP to a new cloud service
The script below creates a new reserved IP, then associates it to a new cloud service named *TestService*.

	New-AzureReservedIP –ReservedIPName MyReservedIP –Location “Central US”
	$image = Get-AzureVMImage|?{$_.ImageName -like "*RightImage-Windows-2012R2-x64*"}
	New-AzureVMConfig -Name TestVM -InstanceSize Small -ImageName $image.ImageName `
	| Add-AzureProvisioningConfig -Windows -AdminUsername adminuser -Password MyP@ssw0rd!! `
	| New-AzureVM -ServiceName TestService -ReservedIPName MyReservedIP -Location "Central US"

>[AZURE.Note] When you create a reserved IP to use with a cloud service, you’ll still need to refer to the VM by using *VIP:&lt;port number&gt;* for inbound communication. Reserving an IP does not mean you can connect to the VM directly. The reserved IP is assigned to the cloud service that the VM has been deployed to. If you want to connect to a VM by IP directly, you have to configure an instance-level public IP. An instance-level public IP is a type of public IP (called a ILPIP) that is assigned directly to your VM. See [Instance-level Public IP (ILPIP)](../virtual-networks-instance-level-public-ip) for more information.

## How to remove a reserved IP from a running deployment
To remove the reserved IP added to the new service created in the script above, run the following PowerShell command:

	Remove-AzureReservedIPAssociation -ReservedIPName MyReservedIP -ServiceName TestService

>[AZURE.NOTE] Removing a reserved IP from a running deployment does not remove the reservation from your subscription. It simply frees the IP to be used by another resource in your subscription.

## How to associate a reserved IP to a running deployment
The script below creates a new cloud service named *TestService2* with a new VM named *TestVM2*, and then associates the existing reserved IP named *MyReservedIP* to the cloud service.

	$image = Get-AzureVMImage|?{$_.ImageName -like "*RightImage-Windows-2012R2-x64*"}
	New-AzureVMConfig -Name TestVM2 -InstanceSize Small -ImageName $image.ImageName `
	| Add-AzureProvisioningConfig -Windows -AdminUsername adminuser -Password MyP@ssw0rd!! `
	| New-AzureVM -ServiceName TestService2 -Location "Central US"
	Set-AzureReservedIPAssociation -ReservedIPName MyReservedIP -ServiceName TestService2

## How to associate a reserved IP to a cloud service by using a service configuration file
You can also associate a reserved IP to a cloud service by sing a service configuration (CSCFG) file. The sample xml below shows how to configure a cloud service to use a reserved VIP named *MyReservedIP*: 
	
	<?xml version="1.0" encoding="utf-8"?>
	<ServiceConfiguration serviceName="ReservedIPSample" xmlns="http://schemas.microsoft.com/ServiceHosting/2008/10/ServiceConfiguration" osFamily="4" osVersion="*" schemaVersion="2014-01.2.3">
	  <Role name="WebRole1">
	    <Instances count="1" />
	    <ConfigurationSettings>
	      <Setting name="Microsoft.WindowsAzure.Plugins.Diagnostics.ConnectionString" value="UseDevelopmentStorage=true" />
	    </ConfigurationSettings>
	  </Role>
	  <NetworkConfiguration>
	    <AddressAssignments>
	      <ReservedIPs>
	       <ReservedIP name="MyReservedIP"/>
	      </ReservedIPs>
	    </AddressAssignments>
	  </NetworkConfiguration>
	</ServiceConfiguration>

## See Also

[Reserved Private IP](../virtual-networks-reserved-private-ip)

[Instance-Level Public IP (ILPIP)](../virtual-networks-instance-level-public-ip)

[Virtual Network Overview](https://msdn.microsoft.com/library/azure/jj156007.aspx)

[Reserved IP REST APIs](https://msdn.microsoft.com/library/azure/dn722420.aspx)
