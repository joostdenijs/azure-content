<properties 
	pageTitle="How To Control Inbound Traffic to an App Service Environment" 
	description="Learn about how to configure network security rules to control inbound traffic to an App Service Environment." 
	services="app-service\web" 
	documentationCenter="" 
	authors="ccompy" 
	manager="wpickett" 
	editor=""/>

<tags 
	ms.service="app-service-web" 
	ms.workload="web" 
	ms.tgt_pltfrm="na" 
	ms.devlang="na" 
	ms.topic="article" 
	ms.date="04/14/2015" 
	ms.author="stefsh"/>	

# How To Control Inbound Traffic to an App Service Environment

## Overview ##
An App Service Environment is always created in a subnet of a regional [virtual network][virtualnetwork].  A new regional virtual network and new subnet can be defined at the time an App Service Environment is created.  Alternatively, an App Service Environment can be created in a pre-existing regional virtual network and pre-existing subnet.  For more details on creating an App Service Environment see [How To Create an App Service Environment][HowToCreateAnAppServiceEnvironment].

An App Service Environment must always be created within a subnet because a subnet provides a network boundary which can be used to lock down inbound traffic behind upstream devices and services such that HTTP and HTTPS traffic is only accepted from specific upstream IP addresses.

Inbound and outbound network traffic on a subnet is controlled using a [network security group][NetworkSecurityGroups].  Controlling inbound traffic requires creating network security rules in a network security group, and then assigning the network security group the subnet containing the App Service Environment.

Once a network security group is assigned to a subnet, inbound traffic to apps in the App Service Environment is allowed/blocked based on the allow and deny rules defined in the network security group.

## Network Ports Used in an App Service Environment ##
Before locking down inbound network traffic with a network security group, it is important to know the set of required and optional network ports used by an App Service Environment.  Accidentally closing off traffic to some ports can result in loss of functionality in an App Service Environment.

The following is a list of ports used by an App Service Environment:

- 454:  **Required port** used by Azure infrastructure for managing and maintaining App Service Environments.  Do not block traffic to this port.
- 455:  **Required port** used by Azure infrastructure for managing and maintaining App Service Environments.  Do not block traffic to this port.
- 80:  Default port for inbound HTTP traffic to apps running in App Service Plans in an App Service Environment
- 443: Default port for inbound SSL traffic to apps running in App Service Plans in an App Service Environment
- 21:  Control channel for FTP.  This port can be safely blocked if FTP is not being used.
- 10001-10020: Data channels for FTP.  As with the control channel, these ports can be safely blocked if FTP is not being used   (**Note:** the FTP data channels may change during preview.)
- 4016: Used for remote debugging with Visual Studio 2012.  This port can be safely blocked if the feature is not being used.
- 4018: Used for remote debugging with Visual Studio 2013.  This port can be safely blocked if the feature is not being used.
- 4020: Used for remote debugging with Visual Studio 2015.  This port can be safely blocked if the feature is not being used.

## Creating a Network Security Group ##
For full details on how network security groups work see the following [information][NetworkSecurityGroups].  The details below touch on highlights of network security groups, with a focus on configuring and applying a network security group to a subnet that contains an App Service Environment.

Network security groups are first created as a standalone entity associated with a subscription. Since network security groups are created in an Azure region, ensure that the network security group is created in the same region as the App Service Environment.

The following demonstrates creating a network security group:

    New-AzureNetworkSecurityGroup -Name "testNSGexample" -Location "South Central US" -Label "Example network security group for an app service environment"

Once a network security group is created, one or more network security rules are added to it.  Since the set of rules may change over time, it is recommended to space out the numbering scheme used for rule priorities to make it easy to insert additional rules over time.

The example below shows a rule that explicitly grants access to the management ports needed by the Azure infrastructure to manage and maintain an App Service Environment.  Note that all management traffic flows over SSL and is secured by client certificates, so even though the ports are opened they are inaccessible by any entity other than Azure management infrastructure.


    Get-AzureNetworkSecurityGroup -Name "testNSGexample" | Set-AzureNetworkSecurityRule -Name "ALLOW AzureMngmt" -Type Inbound -Priority 100 -Action Allow -SourceAddressPrefix 'INTERNET'  -SourcePortRange '*' -DestinationAddressPrefix '*' -DestinationPortRange '454-455' -Protocol TCP
    

When locking down access to port 80 and 443 to "hide" an App Service Environment behind upstream devices or services, you will need to know the upstream IP address.  For example, if you are using a web application firewall (WAF), the WAF will have its own IP address (or addresses) which it uses when proxying traffic to a downstream App Service Environment.  You will need to use this IP address in the *SourceAddressPrefix* parameter of a network security rule.

In the example below, inbound traffic from a specific upstream IP address is explicitly allowed.  The address *1.2.3.4* is used as a placeholder for the IP address of an upstream WAF.  Change the value to match the address used by your upstream device or service.

    Get-AzureNetworkSecurityGroup -Name "testNSGexample" | Set-AzureNetworkSecurityRule -Name "RESTRICT HTTP" -Type Inbound -Priority 200 -Action Allow -SourceAddressPrefix '1.2.3.4/32'  -SourcePortRange '*' -DestinationAddressPrefix '*' -DestinationPortRange '80' -Protocol TCP
    Get-AzureNetworkSecurityGroup -Name "testNSGexample" | Set-AzureNetworkSecurityRule -Name "RESTRICT HTTPS" -Type Inbound -Priority 300 -Action Allow -SourceAddressPrefix '1.2.3.4/32'  -SourcePortRange '*' -DestinationAddressPrefix '*' -DestinationPortRange '443' -Protocol TCP
    
If FTP support is desired, the following rules can be used as a template to grant access to the FTP control port and data channel ports.  Since FTP is a stateful protocol, you may not be able to route FTP traffic through a traditional HTTP/HTTPS firewall or proxy device.  In this case you will need to set the *SourceAddressPrefix* to a different value - for example the IP address range of developer or deployment machines on which FTP clients are running. 

    Get-AzureNetworkSecurityGroup -Name "testNSGexample" | Set-AzureNetworkSecurityRule -Name "RESTRICT FTPCtrl" -Type Inbound -Priority 400 -Action Allow -SourceAddressPrefix '1.2.3.4/32'  -SourcePortRange '*' -DestinationAddressPrefix '*' -DestinationPortRange '21' -Protocol TCP
    Get-AzureNetworkSecurityGroup -Name "testNSGexample" | Set-AzureNetworkSecurityRule -Name "RESTRICT FTPDataRange" -Type Inbound -Priority 500 -Action Allow -SourceAddressPrefix '1.2.3.4/32'  -SourcePortRange '*' -DestinationAddressPrefix '*' -DestinationPortRange '10001-10020' -Protocol TCP

(**Note:**  the data channel port range may change during the preview period.)

If remote debugging with Visual Studio is used, the following rules demonstrate how to grant access.  There is a separate rule for each supported version of Visual Studio since each version uses a different port for remote debugging.  As with FTP access, remote debugging traffic may not flow properly through a traditional WAF or proxy device.  The *SourceAddressPrefix* can instead be set to the IP address range of developer machines running Visual Studio.

    Get-AzureNetworkSecurityGroup -Name "testNSGexample" | Set-AzureNetworkSecurityRule -Name "RESTRICT RemoteDebuggingVS2012" -Type Inbound -Priority 600 -Action Allow -SourceAddressPrefix '1.2.3.4/32'  -SourcePortRange '*' -DestinationAddressPrefix '*' -DestinationPortRange '4016' -Protocol TCP
    Get-AzureNetworkSecurityGroup -Name "testNSGexample" | Set-AzureNetworkSecurityRule -Name "RESTRICT RemoteDebuggingVS2013" -Type Inbound -Priority 700 -Action Allow -SourceAddressPrefix '1.2.3.4/32'  -SourcePortRange '*' -DestinationAddressPrefix '*' -DestinationPortRange '4018' -Protocol TCP
    Get-AzureNetworkSecurityGroup -Name "testNSGexample" | Set-AzureNetworkSecurityRule -Name "RESTRICT RemoteDebuggingVS2015" -Type Inbound -Priority 800 -Action Allow -SourceAddressPrefix '1.2.3.4/32'  -SourcePortRange '*' -DestinationAddressPrefix '*' -DestinationPortRange '4020' -Protocol TCP

## Assigning a Network Security Group to a Subnet ##
A network security group has a default security rule which denies access to all external traffic.  The result from combining the network security rules described above, and the default security rule blocking inbound traffic, is that only traffic from source address ranges associated with an *Allow* action will be able to send traffic to apps running in an App Service Environment.

After a network security group is populated with security rules, it needs to be assigned to the subnet containing the App Service Environment.  The assignment command references both the name of the virtual network where the App Service Environment resides, as well as the name of the subnet where the App Service Environment was created.  

The example below shows a network security group being assigned to a subnet and virtual network:


    Get-AzureNetworkSecurityGroup -Name "testNSGexample" | Set-AzureNetworkSecurityGroupToSubnet -VirtualNetworkName 'testVNet' -SubnetName 'Subnet-test'

Once the network security group assignment succeeds (the assignment is a long-running operations and can take a few minutes to complete), only inbound traffic matching *Allow* rules will successfully reach apps in the App Service Environment.

For completeness the following example shows how to remove and thus dis-associate the network security group from the subnet:


    Get-AzureNetworkSecurityGroup -Name "testNSGexample" | Remove-AzureNetworkSecurityGroupFromSubnet -VirtualNetworkName 'testVNet' -SubnetName 'Subnet-test'

## Special Considerations for Explicit IP-SSL ##
If an app is configured with an explicit IP address, instead of the default IP address of the App Service Environment, both HTTP and HTTPS traffic flows into the subnet over a different set of ports than ports 80 and 443.

During the initial preview of App Service Environments it is not possible to determine the specific ports used by IP-SSL.  However once this information is exposed through the portal, command-line tools and REST APIs, developers will be able to  configure network security groups to also control traffic over these ports.

## Getting started

To get started with App Service Environments, see [Introduction to App Service Environment][IntroToAppServiceEnvironment]

For details around apps securely connecting to backend resource from an App Service Environment, see [Securely connecting to Backend resources from an App Service Environment][SecurelyConnecttoBackend]

For more information about the Azure App Service platform, see [Azure App Service][AzureAppService].

[AZURE.INCLUDE [app-service-web-whats-changed](../includes/app-service-web-whats-changed.md)]

[AZURE.INCLUDE [app-service-web-try-app-service](../includes/app-service-web-try-app-service.md)]

<!-- LINKS -->
[virtualnetwork]: https://msdn.microsoft.com/library/azure/dn133803.aspx
[HowToCreateAnAppServiceEnvironment]: http://azure.microsoft.com/documentation/articles/app-service-web-how-to-create-an-app-service-environment/
[NetworkSecurityGroups]: https://msdn.microsoft.com/library/azure/dn848316.aspx
[AzureAppService]: http://azure.microsoft.com/documentation/articles/app-service-value-prop-what-is/
[IntroToAppServiceEnvironment]:  http://azure.microsoft.com/documentation/articles/app-service-app-service-environment-intro/
[SecurelyConnecttoBackend]:  http://azure.microsoft.com/documentation/articles/app-service-app-service-environment-securely-connecting-to-backend-resources/ 

<!-- IMAGES -->
