<properties
    pageTitle="Publish an app in RemoteApp"
    description="Learn how to publish applications and resources in RemoteApp."
    services="remoteapp"
    solutions="" documentationCenter=""
    authors="lizap"
    manager="mbaldwin" />

<tags
    ms.service="remoteapp"
    ms.workload="tbd"
    ms.tgt_pltfrm="na"
    ms.devlang="na"
    ms.topic="article"
    ms.date="05/13/2015" 
    ms.author="elizapo" />


# How to publish an app in RemoteApp

After you create your RemoteApp collection, you need to publish the apps or resources that you want to make available for your users. The template images provided with your subscription only have a few apps published by default - to share the other apps, you need to publish them.

On the **Publishing** tab in the portal, click **Publish**. You can either add an app from your template image's **Start** menu or provide the path to where the app is installed on the template image. If you choose to add from the Start menu, choose the app to publish from the list. If you choose to provide the path to the app, enter a name for the app and the path to the app. Use variables in the path - for example, "%systemdrive%" instead of "c:\".

**Note:** If you want to add your app from the Start menu, you need to have *added that app to the Start menu on your template image.* Otherwise, RemoteApp will only see what *is* on the Start menu, and you will be confused. If you forgot to add the app to the **Start** menu when you created the template, choose to add the path to the app. (Or recreate your template image, but that's quite a bit more work.)
