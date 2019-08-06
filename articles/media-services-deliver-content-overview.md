<properties 
	pageTitle="Delivering Content to Customers Overview" 
	description="This topic gives an overview of what is involved in delivering your content with Azure Media Services." 
	services="media-services" 
	documentationCenter="" 
	authors="Juliako" 
	manager="dwrede" 
	editor=""/>

<tags 
	ms.service="media-services" 
	ms.workload="media" 
	ms.tgt_pltfrm="na" 
	ms.devlang="na" 
	ms.topic="article" 
	ms.date="04/08/2015" 
	ms.author="juliako"/>


#Delivering Content to Customers Overview

##Overview

When delivering your content to customers (streaming live events or video-on-demand) your goal is to deliver a high quality video to various devices under different network conditions. 

To achieve this goal:

- encode your stream to multi-bitrate (adaptive bitrate) video stream (this will take care of quality and network conditions) and 
- use Media Services [Dynamic Packaging](media-services-dynamic-packaging-overview.md) to dynamically re-package your stream into different protocols (this will take care of streaming on different devices). Media Services supports delivery of the following adaptive bitrate streaming technologies: HTTP Live Streaming (HLS), Smooth Streaming, MPEG DASH, and HDS (for Adobe PrimeTime/Access licensees only).

This topic gives an overview of  [content delivery concepts](media-services-deliver-content-overview.md#concepts) and links to topics that show how to perform content delivery [tasks](media-services-deliver-content-overview.md#tasks).

##<a id="concepts"></a>Concepts

The following list describes useful terminology and concepts when delivering media.

###Dynamic packaging

It is recommended to use dynamic packaging to deliver your content. For more information see [Dynamic Packaging](media-services-dynamic-packaging-overview.md).  

To take advantage of dynamic packaging, you must first get at least one On-demand streaming unit for the streaming endpoint from which you plan to delivery your content. For more information, see [How to Scale Media Services](media-services-manage-origins.md#scale_streaming_endpoints).

###Locators

To provide your user with a URL that can be used to stream or download your content, you first need to "publish" your asset by creating a locator.  Locators provide an entry point to access the files contained in an asset. Media Services supports two types of locators: 

- **OnDemandOrigin** locators, used to stream media (for example, MPEG DASH, HLS, or Smooth Streaming) or progressively download files.
-  **SAS** (access signature) URL locators, used to download media files to your local computer. 

An **Access Policy** is used to define the permissions (such as read, write, and list) and duration that a client has access to a given asset. Note, that the list permission (AccessPermissions.List) should not be used when creating an OrDemandOrigin locator.

Locators have an expiration date. When using Portal to publish your assets, locators with a 100 years expiration date are created. 

>[AZURE.NOTE] If you used Portal to create locators before March 2015, locators with a two years expiration date were created.  

To update expiration date on a locator, use [REST](http://msdn.microsoft.com/library/azure/hh974308.aspx#update_a_locator ) or [.NET](http://go.microsoft.com/fwlink/?LinkID=533259) APIs. Note that when you update the expiration date of a SAS locator, the URL changes. 
 
Locators are not designed to manage per-user access control. To give different access rights to individual users, use Digital Rights Management (DRM) solutions. For more information, see [Securing Media](http://msdn.microsoft.com/library/azure/dn282272.aspx).

Note that when you create a locator, there may be a 30-second delay due to required storage and propagation processes in Azure Storage.


###Adaptive streaming 

Adaptive bitrate technologies allow video player applications to determine network conditions and select from among several bitrates. When network communication degrades, the client can select a lower bitrate allowing the player to continue to play the video at a lower video quality. As network conditions improve the client can switch to a higher bitrate with improved video quality. Azure Media Services supports the following adaptive bitrate technologies: HTTP Live Streaming (HLS), Smooth Streaming, MPEG DASH, and HDS.

To provide users with streaming URLs, you first must create an OnDemandOrigin locator. Creating the locator, gives you the base Path to the asset that contains the content you want to stream. However, to be able to stream this content you need to modify this path further. To construct a full URL to the streaming manifest file, you must concatenate the locator’s Path value and the manifest (filename.ism) file name. Then, append /Manifest and an appropriate format (if needed) to the locator path. 

You can also stream your content over an SSL connection. To do this, make sure your streaming URLs start with HTTPS. 

Note that you can only stream over SSL if the streaming endpoint from which you deliver your content was created after September 10th, 2014. If your streaming URLs are based on the streaming endpoints created after September 10th, the URL contains “streaming.mediaservices.windows.net” (the new format). Streaming URLs that contain “origin.mediaservices.windows.net” (the old format) do not support SSL. If your URL is in the old format and you want to be able to stream over SSL, create a new streaming endpoint. Use URLs created based on the new streaming endpoint to stream your content over SSL. 


####Streaming URL formats:

**MPEG DASH format**

{streaming endpoint name-media services account name}.streaming.mediaservices.windows.net/{locator ID}/{filename}.ism/Manifest(format=mpd-time-csf) 

Example

	http://testendpoint-testaccount.streaming.mediaservices.windows.net/fecebb23-46f6-490d-8b70-203e86b0df58/BigBuckBunny.ism/Manifest(format=mpd-time-csf)

**Apple HTTP Live Streaming (HLS) V4 format**

{streaming endpoint name-media services account name}.streaming.mediaservices.windows.net/{locator ID}/{filename}.ism/Manifest(format=m3u8-aapl)

	http://testendpoint-testaccount.streaming.mediaservices.windows.net/fecebb23-46f6-490d-8b70-203e86b0df58/BigBuckBunny.ism/Manifest(format=m3u8-aapl)

**Apple HTTP Live Streaming (HLS) V3 format**

{streaming endpoint name-media services account name}.streaming.mediaservices.windows.net/{locator ID}/{filename}.ism/Manifest(format=m3u8-aapl-v3)
	
	http://testendpoint-testaccount.streaming.mediaservices.windows.net/fecebb23-46f6-490d-8b70-203e86b0df58/BigBuckBunny.ism/Manifest(format=m3u8-aapl-v3)

**Smooth Streaming format**

{streaming endpoint name-media services account name}.streaming.mediaservices.windows.net/{locator ID}/{filename}.ism/Manifest

Example:

	http://testendpoint-testaccount.streaming.mediaservices.windows.net/fecebb23-46f6-490d-8b70-203e86b0df58/BigBuckBunny.ism/Manifest

**Smooth Streaming 2.0 manifest (legacy manifest)**

By default Smooth Streaming manifest format contains the repeat tag (r-tag). However, some players do not support the r-tag. Such clients can use format that disables the r-tag:

{streaming endpoint name-media services account name}.streaming.mediaservices.windows.net/{locator ID}/{filename}.ism/Manifest(format=fmp4-v20)

	http://testendpoint-testaccount.streaming.mediaservices.windows.net/fecebb23-46f6-490d-8b70-203e86b0df58/BigBuckBunny.ism/Manifest(format=fmp4-v20)

**HDS (for Adobe PrimeTime/Access licensees only)**

{streaming endpoint name-media services account name}.streaming.mediaservices.windows.net/{locator ID}/{filename}.ism/Manifest(format=f4m-f4f)

	http://testendpoint-testaccount.streaming.mediaservices.windows.net/fecebb23-46f6-490d-8b70-203e86b0df58/BigBuckBunny.ism/Manifest(format=f4m-f4f)


###Dynamic packaging

Media Services provides dynamic packaging which allows you to deliver your adaptive bitrate MP4 or Smooth Streaming encoded content in streaming formats supported by Media Services (MPEG DASH, HLS, Smooth Streaming, HDS) without you having to re-package into these streaming formats. 

To take advantage of dynamic packaging, you need to do the following:

- Encode your mezzanine (source) file into a set of adaptive bitrate MP4 files or adaptive bitrate Smooth Streaming files.
- Get at least one On-Demand streaming unit for the streaming endpoint from which you plan to delivery your content. For more information, see [How to Scale On-Demand Streaming Reserved Units](media-services-manage-origins.md#scale_streaming_endpoints/).

With dynamic packaging you only need to store and pay for the files in single storage format and Media Services will build and serve the appropriate response based on requests from a client. 

Note that in addition to being able to use the dynamic packaging capabilities, On-Demand Streaming reserved units provide you with dedicated egress capacity that can be purchased in increments of 200 Mbps. By default, on-demand streaming is configured in a shared-instance model for which server resources (for example, compute, egress capacity, etc.) are shared with all other users. To improve an on-demand streaming throughput, it is recommended to purchase On-Demand Streaming reserved units.

###Progressive download 

Progressive download allows you to start playing media before the entire file has been downloaded. You cannot progressively download .ism* (ismv, isma, ismt, ismc) files. 

To progressively download content, use the OnDemandOrigin type of locator. The following example shows the URL that is based on the OnDemandOrigin type of locator:

	http://amstest1.streaming.mediaservices.windows.net/3c5fe676-199c-4620-9b03-ba014900f214/BigBuckBunny_H264_650kbps_AAC_und_ch2_96kbps.mp4

The following consideration applies:

- You must decrypt any storage encrypted assets that you wish to stream from the origin service for progressive download.


###Download

To download your content onto a client device, you must create a SAS Locator. The SAS locator gives you access the Azure Storage container where your file is located. To build the download URL, you have to embed the file name between the host and SAS signature. 

The following example shows the URL that is based on the SAS locator:

	https://test001.blob.core.windows.net/asset-ca7a4c3f-9eb5-4fd8-a898-459cb17761bd/BigBuckBunny.mp4?sv=2012-02-12&se=2014-05-03T01%3A23%3A50Z&sr=c&si=7c093e7c-7dab-45b4-beb4-2bfdff764bb5&sig=msEHP90c6JHXEOtTyIWqD7xio91GtVg0UIzjdpFscHk%3D

The following considerations apply:

- You must decrypt any storage encrypted assets that you wish to stream from the origin service for progressive download.
- A download that has not completed within 12 hours will fail.



###Streaming Endpoints

A **Streaming Endpoint** represents a streaming service that can deliver content directly to a client player application, or to a Content Delivery Network (CDN) for further distribution. The outbound stream from a streaming endpoint service can be a live stream, or a video on demand asset in your Media Services account. In addition, you can control the capacity of the Streaming Endpoint service to handle growing bandwidth needs by adjusting streaming reserved units. You should allocate at least one reserved unit for applications in a production environment. For more information, see [How to Scale a Media Service](media-services-manage-origins.md#scale_streaming_endpoints).

##<a id="tasks"></a>Tasks related to delivering assets


###Configuring streaming endpoints

For an overview about streaming endpoints and information on how to manage them, see [How to Manage Streaming Endpoints in a Media Services Account](media-services-manage-origins.md).

###Uploading media 

Upload your files using **Azure Management Portal**, **.NET** or **REST API**.

[AZURE.INCLUDE [media-services-selector-upload-files](../includes/media-services-selector-upload-files.md)]

###Encoding assets

Encode with **Azure Media Encoder** using **Azure Management Portal**, **.NET**, or **REST API**.
 
[AZURE.INCLUDE [media-services-selector-encode](../includes/media-services-selector-encode.md)]

###Configuring asset delivery policy

Configure asset delivery policy using **.NET** or **REST API**.

[AZURE.INCLUDE [media-services-selector-asset-delivery-policy](../includes/media-services-selector-asset-delivery-policy.md)]

###Publishing assets

Publish assets (by creating Locators) using **Azure Management Portal** or **.NET**.

[AZURE.INCLUDE [media-services-selector-publish](../includes/media-services-selector-publish.md)]


##Related topics

[Update Media Services locators after rolling storage keys](media-services-roll-storage-access-keys.md)
