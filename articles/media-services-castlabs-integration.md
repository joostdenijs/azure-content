<properties 
	pageTitle="Using castLabs to deliver DRM licenses to Azure Media Services" 
	description="This article describes how you can use Azure Media Services (AMS) to deliver a stream that is dynamically encrypted by AMS with both PlayReady and Widevine DRMs. The PlayReady license comes from Media Services PlayReady license server and Widevine license is delivered by castLabs license server." 
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
	ms.date="05/12/2015" 
	ms.author="juliako"/>


#Using castLabs to deliver DRM licenses to Azure Media Services

##Overview

This article describes how you can use Azure Media Services (AMS) to deliver a stream that is dynamically encrypted by AMS with both PlayReady and Widevine DRMs. The PlayReady license comes from Media Services PlayReady license server and Widevine license is delivered by **castLabs** license server.

The following diagram demonstrates a high-level Azure Media Services and castLabs integration architecture.

![Scale page](./media/media-services-with-castlabs/media-services-castlabs-integration.png)

##Typical system set up

- Media content is stored in AMS.
- Key IDs of content keys are stored in both castLabs and AMS.
- castLabs and AMS both have token authentication built in. The following sections discuss authentication tokens. 
- When a client requests to stream the video, the content is dynamically encrypted with **Common Encryption** (CENC) and dynamically packaged by AMS to any (or all) of the specified protocols: Smooth streaming, HLS or DASH. 
- PlayReady license is retrieved from AMS license server and Widevine license is retrieved from castLabs license server. 
- Media Player automatically decides which license to fetch based on the client platform capability. 

##Authentication token generation for getting a license

Both castLabs and AMS support JWT (JSON Web Token) token format used to authorize a license. 

###JWT token in AMS 

The following table describes JWT token in AMS. 

<table border="1">
<tr><td>Issuer</td><td>Issuer string from the chosen Secure Token Service (STS)</td></tr>
<tr><td>Audience</td><td>Audience string from the used STS</td></tr>
<tr><td>Claims</td><td>A set of claims</td></tr>
<tr><td>NotBefore</td><td>Start validity of the token</td></tr>
<tr><td>Expires</td><td>End validity of the token</td></tr>
<tr><td>SigningCredentials</td><td>The key that is shared among PlayReady License Server, castLabs License Server and STS, it could be either symmetric or asymmetric key.</td></tr>
</table>

###JWT token in castLabs

The following table describes JWT token in castLabs. 

<table border="1">
<tr><td>optData</td><td>A JSON string containing information about you. </td></tr>
<tr><td>crt</td><td>A JSON string containing information about the asset, its license info and playback rights.</td></tr>
<tr><td>iat</td><td>The current datetime in epoch.</td></tr>
<tr><td>jti</td><td>A unique identifier about this token (every token can only be used once in the castLabs system).</td></tr>
</table> 

##Sample solution set up 

The [sample solution](https://github.com/AzureMediaServicesSamples/CastlabsIntegration) consists of two projects:

-	A console app that can be used to set DRM restrictions on an already ingested asset, for both PlayReady and Widevine.
-	A Web Application that hands out tokens, which could be seen as a VERY SIMPLIFIED version of an STS.


To use the console application:

1.	Change the app.config to setup AMS credentials, castLabs credentials, STS configuration and shared key.
2.	Upload an Asset into AMS.
3.	Get the UUID from the uploaded Asset, and change Line 32 in the Program.cs file:

		 var objIAsset = _context.Assets.Where(x => x.Id == "nb:cid:UUID:dac53a5d-1500-80bd-b864-f1e4b62594cf").FirstOrDefault();

4.	Use an AssetId for naming the asset in the castLabs system (Line 44 in the Program.cs file).

	You must set AssetId for **castLabs**; it needs to be a unique alphanumeric string.

5.	Run the program.


To use the Web Application (STS):

1.	Change the web.config to setup castlabs merchant ID, the STS configuration and the shared key.
2.	Deploy to Azure Websites.
3.	Navigate to the website.

##Playing back a video

To playback a video encrypted with common encryption (PlayReady and Widevine), you can use the [Azure Media Player](http://amsplayer.azurewebsites.net/azuremediaplayer.html). When running the console app, the Content Key ID and the Manifest URL are echoed.

1.	Open a new tab and launch your STS: http://[yourStsName].azurewebsites.net/api/token/assetid/[yourCastLabsAssetId]/contentkeyid/[thecontentkeyid].
2.	Go to [Azure Media Player](http://amsplayer.azurewebsites.net/azuremediaplayer.html).
3.	Paste in the streaming URL.
4.	Click the **Advanced Options** checkbox.
5.	Select PlayReady in the **Protection** dropdown.
6.	Paste the token that you got from your STS in the Token textbox.
7.	Update the player.
8.	The video should be playing.

For playing back the protected video in HTML5 with Chrome with the castLabs player, please contact castLabs to get access to the player. When you have access, there are 2 things to be aware of:

1.	The castLabs player needs access to the MPEG-DASH manifest file, so append (format=mpd-time-csf) to your manifest file to get the MPEG-DASH manifest file, instead of the default Smooth Streaming one.

2.	The castLab license server does not need the “Bearer=” prefix in front of the token. So please remove that before submitting the token.

