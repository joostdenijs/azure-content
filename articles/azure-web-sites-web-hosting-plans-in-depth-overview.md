<properties 
	pageTitle="Azure App Service plans in-depth overview" 
	description="Learn how App Service plans for Azure App Service work, and how they benefit your management experience." 
	services="app-service" 
	documentationCenter="" 
	authors="cephalin" 
	manager="wpickett" 
	editor=""/>

<tags 
	ms.service="app-service" 
	ms.workload="web" 
	ms.tgt_pltfrm="na" 
	ms.devlang="na" 
	ms.topic="article" 
	ms.date="03/24/2015" 
	ms.author="byvinyal"/>

#Azure App Service plans in-depth overview#

An **App Service plan** represents a set of features and capacity that you can share across multiple apps in [Azure App Service](http://go.microsoft.com/fwlink/?LinkId=529714), including Web Apps, Mobile Apps, Logic Apps or API Apps. These plans support 5 pricing tiers (**Free**, **Shared**, **Basic**, **Standard** and **Premium**) where each tier has its own capabilities and capacity. Apps in the same subscription and geographic location can share a plan. All the apps sharing a plan can leverage all the capabilities and features defined by the plan's tier. All apps associated with a given plan run on the resources defined by the plan. For example, if your plan is configured to use two "small" instances in the standard service tier, all apps associated with that plan will run on both instances and will have access to the standard service tier functionality. Plan instances on which apps are running on are fully managed and highly available.

In this article we'll explore the key characteristics such as tier and scale of an App Service plan and how they come into play while managing your apps.

##Apps, and App Service plans

An app in App Service can be associated with only one App Service plan at any given time. 

Both apps and plans are contained in a resource group. A resource group serves as the life-cycle boundary for every resource contained within it. Resource groups enable you to manage all the pieces of an application together.

The ability to have multiple App Service plans in a single resource group allows you to allocate different apps to different physical resources. For example, this allows separation of resources between dev, test and production environments. A scenario for this is when you might want to allocate one plan with its own dedicated set of resources for your production apps, and a second plan for your dev and test environments. In this way, load testing against a new version of your apps will not compete for the same resources as your production apps, which are serving real customers.

Having multiple plans in a single resource group also enables you to define an application that spans across geographical regions. For example, a highly available app running in two regions will include two plans, one for each region, and one app associated with each plan. In such a situation, all the copies of the app will be associated with a single resource group. Having a single view of a resource group with multiple plans and multiple apps makes it easy to manage, control and view the health of the application.

## Create a new App Service plan vs. use an existing plan

When creating a new app, you should consider creating a new resource group when the app you are about to create represents a brand new project. In this case, creating a new resource group, plan, and app is the right choice.

If the app you are about to create is a component for a larger application, then this web app should be created within the resource group allocated for that larger application.

Regardless of the new app being a altogether new application or part of a larger one, you can choose to leverage an existing App Service plan to host it or create a new one. This is more a question of capacity and expected load. If this new app is going to be resource intensive and have different scaling factors than the other apps hosted in an existing plan, it is recommended to isolate it into its own plan.

Creating a new plan allows you to allocate a new set of resource for your web app, and provides you with greater control over resource allocation, as each plan gets its own set of instances.
 
Having the capacity to move apps across plans also allows you to change the way resources are allocated across the bigger application.
 
Finally, if you want to create a new app in a different region, and that region doesn't have an existing plan, you will have to create a new plan in that region to be able to host your app there.

## Create an App Service plan

You can't create an empty App Service plan. However, you can explicitly create a new plan during app creation.

To do this in the [Azure Portal](http://go.microsoft.com/fwlink/?LinkId=529715), click **NEW**, then select **Web + mobile**, then select **Web Apps**, **Mobile Apps**, **Logic Apps** or **API Apps**. You can then select or create the App Service plan for the new app.
 
![App Service Plan F.A.Q.](./media/azure-web-sites-web-hosting-plans-in-depth-overview/azure-web-sites-web-hosting-plans-in-depth-overview01.png)

##Assign an app to an App Service plan

Apps can be assigned to an existing plan during the creation process.

To do this in the [Azure Portal](http://portal.azure.com), click **NEW**, then select **Web + mobile**, then select **Web Apps**, **Mobile Apps**, **Logic Apps** or **API Apps**. You can then select or create the App Service plan for the new app. Clicking **Or select existing** will give you the list of existing plan you can choose from.

![App Service Plan F.A.Q.](./media/azure-web-sites-web-hosting-plans-in-depth-overview/azure-web-sites-web-hosting-plans-in-depth-overview02.png)
 
## Move an app to a different App Service plan

You can move an app to a different app service plan in the [Azure Portal](http://portal.azure.com). Apps can be moved between plans in the same geographical region.

To move an app to another plan, navigate to the app you want to move, then click **Change App Service Plan**.
 
This will open the App Service Plan blade. At this point, you can either pick an existing plan, or create a new one. Plans in a different geographical location are grayed out and cannot be selected.

![App Service Plan F.A.Q.](./media/azure-web-sites-web-hosting-plans-in-depth-overview/azure-web-sites-web-hosting-plans-in-depth-overview03.png)

Note that each plan has its own pricing tier. When you move a site from a **Free** tier to a **Standard** tier, your app will be able to leverage all the features and resources of the **Standard** tier.

## Scale an App Service plan

There are three ways to scale a plan:

- Change the plan’s **pricing tier**. For example, a plan in the **Basic** tier can be converted into a **Standard** or **Premium** tier and all apps associated with that plan will be able to leverage the features offered in the new service tier.
- Change the plan’s **instance size**, as an example a plan in the **Basic** tier using **small** instances can be changed to use **large** instances. All apps associated with that plan will be able to leverage the additional memory and CPU resources offered by the larger instance size.
- Change the plan’s **instance count**. For example, a **Standard** plan scaled out to 3 instances can be scaled to 10 instances, and a **Premium** (preview) plan can be scaled out to 20 instances (under some restrictions). All apps associated with that plan will be able to leverage the additional memory and CPU resources offered by the larger instance count.

In the image below you can see the **App Service Plan** blade as well as the **Pricing Tier** blade. Clicking on the **Pricing Tier** part in the **App Service Plan** blade expands the **Pricing Tier** blade, where you can change the pricing tier and instance size for the plan.
 
![App Service Plan F.A.Q.](./media/azure-web-sites-web-hosting-plans-in-depth-overview/azure-web-sites-web-hosting-plans-in-depth-overview04.png)

##Summary

App Service plans represent a set of features and capacity that you can share across your apps. App Service plans give you the flexibility to allocate specific apps to a given set of resources and further optimize you Azure resource utilization. This way, if you want to save money on your testing environment you can share a plan across multiple apps. You can also maximize throughput for your production environment by scaling it across multiple regions and plans.

## What's changed

* For a guide to the change from Websites to App Service see: [Azure App Service and Its Impact on Existing Azure Services](http://go.microsoft.com/fwlink/?LinkId=529714)
* For a guide to the change of the old portal to the new portal see: [Reference for navigating the preview portal](http://go.microsoft.com/fwlink/?LinkId=529715)
