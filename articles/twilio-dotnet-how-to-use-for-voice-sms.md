<properties 
	pageTitle="How to Use Twilio for Voice and SMS (.NET) - Azure" 
	description="Learn how to make a phone call and send a SMS message with the Twilio API service on Azure. Code samples written in .NET." 
	services="" 
	documentationCenter=".net" 
	authors="devinrader" 
	manager="twilio" 
	editor=""/>

<tags 
	ms.service="multiple" 
	ms.workload="na" 
	ms.tgt_pltfrm="na" 
	ms.devlang="dotnet" 
	ms.topic="article" 
	ms.date="04/24/2015" 
	ms.author="devinrader"/>





# How to use Twilio for voice and SMS capabilities from Azure

This guide demonstrates how to perform common programming tasks with the Twilio API service on Azure. The scenarios covered include making a phone call and sending a Short Message Service (SMS) message. For more information on Twilio and using voice and SMS in your applications, see the [Next steps](#NextSteps) section.

## <a id="WhatIs"></a>What is Twilio?
Twilio is powering the future of business communications, enabling developers to embed voice, VoIP, and messaging into applications. They virtualize all infrastructure needed in a cloud-based, global environment, exposing it through the Twilio communications API platform. Applications are simple to build and scalable. Enjoy flexibility with pay-as-you go pricing, and benefit from cloud reliability.

**Twilio Voice** allows your applications to make and receive phone calls. **Twilio SMS** enables your applications to send and receive SMS messages. **Twilio Client** allows you to make VoIP calls from any phone, tablet, or browser and supports WebRTC.

## <a id="Pricing"></a>Twilio Pricing and Special Offers
Azure customers receive a [special offer](http://www.twilio.com/azure): complimentary $10 of Twilio Credit when you upgrade your Twilio Account. This Twilio Credit can be applied to any Twilio usage ($10 credit equivalent to sending as many as 1,000 SMS messages or receiving up to 1000 inbound Voice minutes, depending on the location of your phone number and message or call destination). Redeem this Twilio credit and get started at [ahoy.twilio.com/azure](http://ahoy.twilio.com/azure).

Twilio is a pay-as-you-go service. There are no set-up fees and you can close your account at any time. You can find more details at [Twilio Pricing](http://www.twilio.com/voice/pricing).  

## <a id="Concepts"></a>Concepts
The Twilio API is a RESTful API that provides voice and SMS functionality for applications. Client libraries are available in multiple languages; for a list, see [Twilio API Libraries] [twilio_libraries].

Key aspects of the Twilio API are Twilio verbs and Twilio Markup Language (TwiML).

### <a id="Verbs"></a>Twilio verbs
The API makes use of Twilio verbs; for example, the **&lt;Say&gt;** verb instructs Twilio to audibly deliver a message on a call. 

The following is a list of Twilio verbs.  Learn about the other verbs and capabilities via [Twilio Markup Language documentation](http://www.twilio.com/docs/api/twiml).

* **&lt;Dial&gt;**: Connects the caller to another phone.
* **&lt;Gather&gt;**: Collects numeric digits entered on the telephone keypad.
* **&lt;Hangup&gt;**: Ends a call.
* **&lt;Play&gt;**: Plays an audio file.
* **&lt;Pause&gt;**: Waits silently for a specified number of seconds.
* **&lt;Record&gt;**: Records the caller's voice and returns a URL of a file that contains the recording.
* **&lt;Redirect&gt;**: Transfers control of a call or SMS to the TwiML at a different URL.
* **&lt;Reject&gt;**: Rejects an incoming call to your Twilio number without billing you
* **&lt;Say&gt;**: Converts text to speech that is made on a call.
* **&lt;Sms&gt;**: Sends an SMS message.

### <a id="TwiML"></a>TwiML
TwiML is a set of XML-based instructions based on the Twilio verbs that inform Twilio of how to process a call or SMS.

As an example, the following TwiML would convert the text **Hello World** to speech.

    <?xml version="1.0" encoding="UTF-8" ?>
    <Response>
       <Say>Hello World</Say>
    </Response>

When your application calls the Twilio API, one of the API parameters is the URL that returns the TwiML response. For development purposes, you can use Twilio-provided URLs to provide the TwiML responses used by your applications. You could also host your own URLs to produce the TwiML responses, and another option is to use the **TwiMLResponse** object.

For more information about Twilio verbs, their attributes, and TwiML, see [TwiML] [twiml]. For additional information about the Twilio API, see [Twilio API] [twilio_api].

## <a id="CreateAccount"></a>Create a Twilio Account
When you're ready to get a Twilio account, sign up at [Try Twilio] [try_twilio]. You can start with a free account, and upgrade your account later.

When you sign up for a Twilio account, you'll receive an account ID and an authentication token. Both will be needed to make Twilio API calls. To prevent unauthorized access to your account, keep your authentication token secure. Your account ID and authentication token are viewable at the [Twilio account page] [twilio_account], in the fields labeled **ACCOUNT SID** and **AUTH TOKEN**, respectively.

## <a id="create_app"></a>Create an Azure Application
An Azure application that hosts a Twilio enabled application is no different from any other Azure application. You add the Twilio .NET library and configure the role to use the Twilio .NET libraries.
For information on creating an initial Azure project, see [Creating an Azure project with Visual Studio][vs_project].

## <a id="configure_app"></a>Configure Your Application to use Twilio Libraries
Twilio provides a set of .NET helper libraries that wrap various aspects of Twilio to provide simple and easy ways to interact with the Twilio REST API and Twilio Client to generate TwiML responses.

Twilio provides five libraries for .NET developers:
<table border="1">
    <tr>
        <th>Library</th>
        <th>Description</th>
    </tr>
    <tr>
        <td>Twilio.API</td>
        <td>The core Twilio library that wraps the Twilio REST API in a friendly .NET library. This library is available for .NET, Silverlight, and Windows Phone 7.</td>
    </tr>
    <tr>
        <td>Twilio.TwiML</td>
        <td>Provides a .NET friendly way to generate TwiML markup.</td>
    </tr>
    <tr>
        <td>Twilio.MVC</td>
        <td>For developers using ASP.NET MVC, this library includes a TwilioController, TwiML ActionResult and request validation attribute.</td>
    </tr>
    <tr>
        <td>Twilio.WebMatrix</td>
        <td>For developers using Microsoft's free WebMatrix development tool, this library contains Razor syntax helpers for various Twilio actions.</td>
    </tr>
    <tr>
        <td>Twilio.Client.Capability</td>
        <td>Contains the Capability token generator for use with the Twilio Client JavaScript SDK.</td>
    </tr>
</table>

Note that all libraries require .NET 3.5, Silverlight 4, or Windows Phone 7 or later.

The samples provided in this guide use the Twilio.API library.

The libraries can be [installed using the NuGet package manager extension](http://www.twilio.com/docs/csharp/install) available for Visual Studio 2010 and 2012.  The source code is hosted on [GitHub][twilio_github_repo], which includes a Wiki that contains complete documentation for using the libraries.

By default, Microsoft Visual Studio 2010 installs version 1.2 of NuGet. Installing the Twilio libraries requires version 1.6 of NuGet or higher. For information on installing or updating NuGet, see [http://nuget.org/][nuget].

> [AZURE.NOTE] To install the latest verison of NuGet, you must first uninstall the loaded version using the Visual Studio Extension Manager. To do so, you must run Visual Studio as administrator. Otherwise, the Uninstall button is disabled.

### <a id="use_nuget"></a>To add the Twilio libraries to your Visual Studio project:

1.  Open your solution in Visual Studio.
2.  Right-click **References**.
3.  Click **Manage NuGet Packages...**
4.  Click **Online**.
5.  In the search online box, type *twilio*.
6.  Click **Install** on the Twilio package.


## <a id="howto_make_call"></a>How to: Make an outgoing call
The following shows how to make an outgoing call using the **TwilioRestClient** class. This code also uses a Twilio-provided site to return the Twilio Markup Language (TwiML) response. Substitute your values for the **From** and **To** phone numbers, and ensure that you verify the **From** phone number for your Twilio account prior to running the code.

    // Use your account SID and authentication token instead
    // of the placeholders shown here.
    string accountSID = "your_twilio_account";
    string authToken = "your_twilio_authentication_token";

    // Create an instance of the Twilio client.
    TwilioRestClient client;
    client = new TwilioRestClient(accountSID, authToken);

    // Use the Twilio-provided site for the TwiML response.
    String Url="http://twimlets.com/message";
    Url = Url + "?Message%5B0%5D=Hello%20World";

    // Instantiate the call options that are passed
    // to the outbound call
    CallOptions options = new CallOptions();

    // Set the call From, To, and URL values to use for the call.
    // This sample uses the sandbox number provided by
    // Twilio to make the call.
    options.From = "+NNNNNNNNNN";
    options.To = "NNNNNNNNNN";
    options.Url = Url;

    // Make the call.
    var call = client.InitiateOutboundCall(options);

For more information about the parameters passed in to the **client.InitiateOutboundCall** method, see [http://www.twilio.com/docs/api/rest/making-calls][twilio_rest_making_calls].

As mentioned, this code uses a Twilio-provided site to return the TwiML response. You could instead use your own site to provide the TwiML response. For more information, see [How to: Provide TwiML responses from your own web site](#howto_provide_twiml_responses).

## <a id="howto_send_sms"></a>How to: Send an SMS message
The following screenshot shows how to send an SMS message using the **TwilioRestClient**  class. The **From** number is provided by Twilio for trial accounts to send SMS messages. The **To** number must be verified for your Twilio account before you run the code.

        // Use your account SID and authentication token instead
        // of the placeholders shown here.
        string accountSID = "your_twilio_account";
        string authToken = "your_twilio_authentication_token";

        // Create an instance of the Twilio client.
        TwilioRestClient client;
        client = new TwilioRestClient(accountSID, authToken);

        // Send an SMS message.
        Message result = client.SendMessage(
            "+14155992671", "+12069419717", "This is my SMS message.");

        if (result.RestException != null)
        {
            //an exception occurred making the REST call
            string message = result.RestException.Message;
        }

## <a id="howto_provide_twiml_responses"></a>How to: Provide TwiML Responses from your own website
When your application initiates a call to the Twilio API - for example, via the **client.InitiateOutboundCall** method - Twilio sends your request to a URL that is expected to return a TwiML response. The example in [How to: Make an outgoing call](#howto_make_call) uses the Twilio-provided URL [http://twimlets.com/message][twimlet_message_url] to return the response. 

> [AZURE.NOTE] While TwiML is designed for use by web services, you can view the TwiML in your browser. For example, click [http://twimlets.com/message](twimlet_message_url) to see an empty &lt;Response&gt; element; as another example, click [http://twimlets.com/message?Message%5B0%5D=Hello%20World](twimlet_message_url_hello_world) to see a &lt;Response&gt; element that contains a &lt;Say&gt; element.

Instead of relying on the Twilio-provided URL, you can create your own URL site that returns HTTP responses. You can create the site in any language that returns HTTP responses. This topic assumes you'll be hosting the URL from an ASP.NET generic handler.

The following ASP.NET Handler crafts a TwiML response that says **Hello World** on the call.

    using System;
    using System.Collections.Generic;
    using System.Linq;
    using System.Web;

    namespace WebRole1
    {
        /// <summary>
        /// Summary description for Handler1
        /// </summary>
        public class Handler1 : IHttpHandler
        {

            public void ProcessRequest(HttpContext context)
            {
                context.Response.Clear();
                context.Response.ContentType = "text/xml";
                context.Response.ContentEncoding = System.Text.Encoding.UTF8;
                string twiMLResponse = "<Response><Say>Hello World.</Say></Response>";
                context.Response.Write(twiMLResponse);
                context.Response.End();
            }

            public bool IsReusable
            {
                get
                {
                    return false;
                }
            }
        }
    }

As you can see from the example above, the TwiML response is simply an XML document. The Twilio.TwiML library contains classes that will generate TwiML for you. The example below produces the equivalent response as shown above, but uses the TwilioResponse class.

    using System;
    using System.Collections.Generic;
    using System.Linq;
    using System.Web;

    namespace WebRole1
    {
        /// <summary>
        /// Summary description for Handler1
        /// </summary>
        public class Handler1 : IHttpHandler
        {

            public void ProcessRequest(HttpContext context)
            {
                var twiml = new Twilio.TwiML.TwilioResponse();
                twiml.Say("Hello World.");

                context.Response.Clear();
                context.Response.ContentType = "text/xml";
                context.Response.Write(twiml.ToString());
                context.Response.End();
            }

            public bool IsReusable
            {
                get
                {
                    return false;
                }
            }
        }
    }

For more information about TwiML, see [https://www.twilio.com/docs/api/twiml](https://www.twilio.com/docs/api/twiml).

Once you have set up a way to provide TwiML responses, you can pass that URL into the **client.InitiateOutboundCall** method. For example, if you have a web application named MyTwiML deployed to an Azure cloud service, and the name of your ASP.NET Handler is mytwiml.ashx, the URL can be passed to **client.InitiateOutboundCall** as shown in the following code sample:

    // Place the call From, To, and URL values into a hash map.
    // This sample uses the sandbox number provided by Twilio to make the call.
    options.From = "NNNNNNNNNN";
    options.To = "NNNNNNNNNN";
    options.Url = "http://<your_hosted_service>.cloudapp.net/MyTwiML/mytwiml.ashx";

    // Place the call.
    var call = client.InitiateOutboundCall(options);


For additional information about using Twilio on Azure with ASP.NET, see [How to make a phone call using Twilio in a web role on Azure][howto_phonecall_dotnet].

[AZURE.INCLUDE [twilio_additional_services_and_next_steps](../includes/twilio_additional_services_and_next_steps.md)]





[howto_phonecall_dotnet]: partner-twilio-cloud-services-dotnet-phone-call-web-role.md



[twimlet_message_url]: http://twimlets.com/message

[twilio_rest_making_calls]: http://www.twilio.com/docs/api/rest/making-calls

[vs_project]:http://msdn.microsoft.com/library/windowsazure/ee405487.aspx
[nuget]:http://nuget.org/
[twilio_github_repo]:https://github.com/twilio/twilio-csharp



[twilio_libraries]: https://www.twilio.com/docs/libraries
[twiml]: http://www.twilio.com/docs/api/twiml
[twilio_api]: http://www.twilio.com/api
[try_twilio]: https://www.twilio.com/try-twilio
[twilio_account]:  https://www.twilio.com/user/account
[verify_phone]: https://www.twilio.com/user/account/phone-numbers/verified#
