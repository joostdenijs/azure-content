<properties 
   pageTitle="Configure Performance Load Balancing"
   description="THis article will help you configure performance load balancing in Traffic Manager"
   services="traffic-manager"
   documentationCenter=""
   authors="cherylmc"
   manager="adinah"
   editor="tysonn" />
<tags 
   ms.service="traffic-manager"
   ms.devlang="na"
   ms.topic="article"
   ms.tgt_pltfrm="na"
   ms.workload="infrastructure-services"
   ms.date="02/27/2015"
   ms.author="cherylmc" />

# Configure Performance Load Balancing

In order to load balance cloud services and websites (endpoints) that are located in different datacenters across the globe (also known as regions), you can direct incoming traffic to the endpoint with the lowest latency from the requesting client. Typically, the datacenter with the lowest latency corresponds to the closest in geographic distance. The Performance load balancing method will allow you to distribute based on lowest latency, but cannot take into account real-time changes in network configuration or load. For more information on the different load balancing methods that Azure Traffic Manager provides, ses [About Traffic Manager Load Balancing Methods](traffic-manager-load-balancing-methods.md).

## Load balance traffic based on lowest latency across a set of endpoints:

1. In the Management Portal, in the left pane, click the **Traffic Manager** icon to open the Traffic Manager pane. If you have not yet created your Traffic Manager profile, see [Manage Traffic Manager Profiles](traffic-manager-manage-profiles.md) for the steps to create a basic Traffic Manager profile.
2. In the Management Portal, on the Traffic Manager pane, locate the Traffic Manager profile that contains the settings that you want to modify, and then click the arrow to the right of the profile name. This will open the settings page for the profile.
3. On the page for your profile, click **Endpoints** at the top of the page and verify that the service endpoints that you want to include in your configuration are present. For steps to add or remove endpoints from your profile, see [Manage Endpoints in Traffic Manager](traffic-manager-endpoints.md).
4. On the page for your profile, click **Configure** at the top to open the configuration page.
5. For **Load Balancing Method Settings**, verify that the load balancing method is **Performance*. If it’s not, click **Performance** in the dropdown list.
6. Verify that the **Monitoring Settings** are configured appropriately. Monitoring ensures that endpoints that are offline are not sent traffic. In order to monitor endpoints, you must specify a path and filename. Note that a forward slash “/“ is a valid entry for the relative path and implies that the file is in the root directory (default). For more information about monitoring, see [About Traffic Manager Monitoring](traffic-manager-monitoring.md).
7. After you complete your configuration changes, click **Save** at the bottom of the page.
8. Test the changes in your configuration. For more information, see [Testing Traffic Manager Settings](traffic-manager-testing-settings.md).
9. Once your Traffic Manager profile is setup and working, edit the DNS record on your authoritative DNS server to point your company domain name to the Traffic Manager domain name. For more information about how to do this, see [Point a Company Internet Domain to a Traffic Manager Domain](traffic-manager-point-internet-domain.md).

## See Also

[About Traffic Manager Load Balancing Methods](traffic-manager-load-balancing-methods.md)

[Traffic Manager Configuration Tasks](https://msdn.microsoft.com/library/azure/hh744830.aspx)

[Traffic Manager Overview](traffic-manmager-overview.md)

[Cloud Services](http://go.microsoft.com/fwlink/?LinkId=314074)

[Websites](http://go.microsoft.com/fwlink/p/?LinkId=393327)

[Operations on Traffic Manager (REST API Reference)](http://go.microsoft.com/fwlink/?LinkId=313584)

[Azure Traffic Manager Cmdlets](http://go.microsoft.com/fwlink/p/?LinkId=400769)

