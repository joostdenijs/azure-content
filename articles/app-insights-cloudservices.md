<properties
   pageTitle="Application Insights for Azure Cloud Services"
   description="Monitor your web and worker roles effectively with Application Insights"
   services="application-insights"
   documentationCenter=""
   authors="soubhagyadash"
   manager="victormu"
   editor="alancameronwills"/>

<tags
   ms.service="application-insights"
   ms.devlang="na"
   ms.tgt_pltfrm="ibiza"
   ms.topic="article"
   ms.workload="tbd"
   ms.date="05/12/2015"
   ms.author="sdash"/>

# Application Insights for Azure Cloud Services

**DRAFT -- TBD items**

* Add portal experience images
* Add code snippets
* Fix links to sample code


*Application Insights is in preview*

[Microsoft Azure Cloud service apps](http://azure.microsoft.com/services/cloud-services/) can be monitored by [Visual Studio Application Insights][start] for availability, performance, failures and usage.

For illustration purposes, we have added Application Insights to this [sample Azure cloud service](sample link). This code is available [here](git link), for you to follow along with the steps below.

## Create an Application Insights resource for each role

An Application Insights resource is where your telemetry data will be analyzed and displayed.  

1.  In the [Azure portal][portal], create a new Application Insights resource. For application type, choose ASP.NET app. 

    ![Click New, Application Insights](./media/app-insights-cloudservices/01-new.png)

2.  Take a copy of the Instrumentation Key. You'll need this shortly to configure the SDK.

    ![Click Properties, select the key, and press ctrl+C](./media/app-insights-cloudservices/02-props.png)


It's usually best to create a separate resource for the data from each web and worker role. 

As an alternative, you could send data from all the roles to just one resource, but set a [default property][apidefaults] so that you can filter or group the results from each role.

## <a name="sdk"></a>Install the SDK in each project


1. In Visual Studio, edit the NuGet packages of your desktop app project.
    ![Right-click the project and select Manage Nuget Packages](./media/app-insights-cloudservices/03-nuget.png)

2. Install the Application Insights SDK for Web Apps.

    ![Select **Online**, **Include prerelease**, and search for "Application Insights"](./media/app-insights-cloudservices/04-ai-nuget.png)

    (As an alternative, you could choose Application Insights SDK for Web Apps. This provides some built-in performance counter telemetry. )

3. Configure the SDK to send data to the Application Insights resource.

    Open `ApplicationInsights.config` and insert this line:

    `<InstrumentationKey>` *the instrumentation key you copied* `</InstrumentationKey>`

    Use the instrumentation key you copied from the Application Insights resource.

    As an alternative, you can [set the key at run time][apidynamicikey]. This lets you direct telemetry from different environments (development, test, production) to different resources. 

## Run your app

Run your project in debug mode on your development machine, or publish it to Azure. Use it for a few minutes to generate some telemetry data.

## <a name="monitor"></a> View your telemetry

Return to the [Azure portal][portal] and Browse to your Application Insights resources.


Look for data in the Overview charts. At first, you'll just see one or two points. For example:

![Click through to more data](./media/app-insights-asp-net/12-first-perf.png)

Click through any chart to see more detailed metrics. [Learn more about metrics.][perf]

Now publish your application and watch the data accumulate.


#### No data?

* Open the [Search][diagnostic] tile, to see individual events.
* Use the application, opening different pages so that it generates some telemetry.
* Wait a few seconds and click Refresh.
* See [Troubleshooting][qna].


## Complete your installation

To get the full 360-degree view of your application, there are some more things you should do:


* [Add the JavaScript SDK to your web pages][client] to get browser-based telemetry such as page view counts, page load times, script exceptions, and to let you write custom telemetry in your page scripts.
* Add dependency tracking to diagnose issues caused by databases or other components used by your app:
 * [in your Azure Web App or VM][azure]
 * [in your on-premises IIS server][redfield]
* [Capture log traces][netlogs] from your favorite logging framework
* [Track custom events and metrics][api] in client or server or both, to learn more about how your application is used.
* [Set up web tests][availability] to make sure your application stays live and responsive.

#### Track requests for worker roles

A request is in essence a unit of *named* server side work that can be *timed* and independently *succeed/fail*.

As such, Requests can be applied for worker roles, and provides a handy way to capture performance and success telemetry of the different operations a worker role may be doing.

Here's a typical run loop for a worker role:

```C#

    while (true)
    {
      Stopwatch s1 = Stopwatch.StartNew();
      var startTime = DateTimeOffset.UtcNow;
      try
      {
        // ... get and process messages ...

        s1.Stop();
        aiClient.TrackRequest("CheckItemsTable",
            startTime, s1.Elapsed, SUCCESS_CODE, true);
      }
      catch (Exception ex)
      {
        string err = ex.Message;
        if (ex.InnerException != null)
        {
           err += " Inner Exception: " + ex.InnerException.Message;
        }
        s1.Stop();
        aiClient.TrackRequest("CheckItemsTable", 
            startTime, s1.Elapsed, FAILURE_CODE, false);
        aiClient.TrackException(ex);

        // Don't flood if we have a bug in queue process loop.
        System.Threading.Thread.Sleep(60 * 1000);
      }
    }
```




[api]: app-insights-api-custom-events-metrics.md
[apidefaults]: app-insights-api-custom-events-metrics.md#default-properties
[apidynamicikey]: app-insights-api-custom-events-metrics.md#dynamic-ikey
[availability]: app-insights-monitor-web-app-availability.md
[azure]: app-insights-azure.md
[client]: app-insights-javascript.md
[diagnostic]: app-insights-diagnostic-search.md
[netlogs]: app-insights-asp-net-trace-logs.md
[perf]: app-insights-web-monitor-performance.md
[portal]: http://portal.azure.com/
[qna]: app-insights-troubleshoot-faq.md
[redfield]: app-insights-monitor-performance-live-website-now.md
[start]: app-insights-get-started.md