<properties
   pageTitle="Disable or Enable a Traffic Manager endpoint"
   description="This article will help disable or enable your Traffic Manager profile endpoints."
   services="traffic-manager"
   documentationCenter="na"
   authors="cherylmc"
   manager="adinah"
   editor="tysonn" />
<tags 
   ms.service="traffic-manager"
   ms.devlang="na"
   ms.topic="article"
   ms.tgt_pltfrm="na"
   ms.workload="infrastructure-services"
   ms.date="02/23/2015"
   ms.author="cherylmc" />

# Disable or Enable a Traffic Manager Endpoint

You can also disable individual endpoints that are part of a Traffic Manager profile. Endpoints include both cloud services and websites. Disabling an endpoint leaves it as part of the profile, but the profile acts as if the endpoint is not included in it. This action is very useful for temporarily removing an endpoint that is in maintenance mode or being redeployed. Once the endpoint is up and running again, it can be enabled

[AZURE.NOTE] **Disabling an endpoint has nothing to do with its deployment state in Azure. A healthy endpoint will remain up and able to receive traffic even when disabled in Traffic Manager. Additionally, disabling an endpoint in one profile does not affect its status in another profile.**

## To disable an endpoint

1. On the Traffic Manager pane in the Management Portal, locate the Traffic Manager profile that contains the endpoint settings that you want to modify, and then click the arrow to the right of the profile name. This will open the settings page for the profile.
1. At the top of the page, click **Endpoints** to view the endpoints that are included in your configuration. 
1. Click the endpoint that you want to disable, and then click **Disable** at the bottom of the page.
1. Traffic will stop flowing to the endpoint based on the DNS Time-to-Live (TTL) configured for the Traffic Manager domain name. You can change the TTL from the Configuration page of the Traffic Manager profile.

## To enable an endpoint


1. On the Traffic Manager pane in the Management Portal, locate the Traffic Manager profile that contains the endpoint settings that you want to modify, and then click the arrow to the right of the profile name. This will open the settings page for the profile.
1. At the top of the page, click **Endpoints** to view the endpoints that are included in your configuration.
1. Click the endpoint that you want to enable, and then click **Enable** at the bottom of the page.
1. Traffic will start flowing to the service again as dictated by the profile.

## See Also

[Traffic Manager Configuration Tasks](https://msdn.microsoft.com/library/azure/hh744830.aspx)

[Traffic Manager Overview](traffic-manager.md)

[Cloud Services](http://go.microsoft.com/fwlink/?LinkId=314074)

[Websites](http://go.microsoft.com/fwlink/p/?LinkId=393327)


[Operations on Traffic Manager (REST API Reference)](http://go.microsoft.com/fwlink/?LinkId=313584)
