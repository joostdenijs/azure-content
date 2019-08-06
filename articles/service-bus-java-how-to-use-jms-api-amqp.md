<properties 
	pageTitle="How to use AMQP 1.0 with the Java Service Bus API - Azure" 
	description="Learn how to use the Java Message Service (JMS) with Azure Service Bus and Advanced Message Queuing Protodol (AMQP) 1.0." 
	services="service-bus" 
	documentationCenter="java" 
	authors="sethmanheim" 
	writer="sethm" 
	manager="timlt" 
	editor="mattshel"/>

<tags 
	ms.service="service-bus" 
	ms.workload="tbd" 
	ms.tgt_pltfrm="na" 
	ms.devlang="Java" 
	ms.topic="article" 
	ms.date="03/26/2015" 
	ms.author="sethm"/>




# How to use the Java Message Service (JMS) API with Service Bus and AMQP 1.0

##Introduction

The Advanced Message Queuing Protocol (AMQP) 1.0 is an efficient, reliable, wire-level messaging protocol that you can use to build robust, cross-platform messaging applications.

Support for AMQP 1.0 in Service Bus means that you can use the queuing and publish/subscribe brokered messaging features from a range of platforms using an efficient binary protocol. Furthermore, you can build applications comprised of components built using a mix of languages, frameworks, and operating systems.

This how-to guide explains how to use the Service Bus brokered messaging features (queues and publish/subscribe topics) from Java applications using the popular Java Message Service (JMS) API standard. There is a companion How-to guide that explains how to do the same using the Service Bus .NET API. You can use these two guides together to learn about cross-platform messaging using AMQP 1.0.

##Get started with Service Bus

This guide assumes that you already have a Service Bus namespace containing a queue named "queue1." If you do not, then you can create the namespace and queue using the [Azure Management Portal](http://manage.windowsazure.com). For more information about how to create Service Bus namespaces and queues, see [How to Use Service Bus Queues](service-bus-dotnet-how-to-use-queues.md).

**Note**: You must create the queue with partitioning disabled as partitioned queues and topics do not yet have AMQP support. For additional information, see [Partitioning Messaging Entities](http://msdn.microsoft.com/library/azure/dn520246.aspx).

##Downloading the AMQP 1.0 JMS client library

For information about where to download the latest version of the Apache Qpid JMS AMQP 1.0 client library, visit [http://people.apache.org/~rgodfrey/qpid-java-amqp-1-0-client-jms.html](http://people.apache.org/~rgodfrey/qpid-java-amqp-1-0-client-jms.html).

You must add the following four JAR files from the Apache Qpid JMS AMQP 1.0 distribution archive to the Java CLASSPATH when building and running JMS applications with Service Bus:

*    geronimo-jms\_1.1\_spec-1.0.jar
*    qpid-amqp-1-0-client-[version].jar
*    qpid-amqp-1-0-client-jms-[version].jar
*    qpid-amqp-1-0-common-[version].jar

##Coding Java applications

### Java Naming and Directory Interface (JNDI)
JMS uses the Java Naming and Directory Interface (JNDI) to create a separation between logical names and physical names. Two types of JMS objects are resolved using JNDI: ConnectionFactory and Destination. JNDI uses a provider model into which you can plug different directory services to handle name resolution duties. The Apache Qpid JMS AMQP 1.0 library comes with a simple properties file-based JNDI Provider that is configured using a properties file of the following format:

	# servicebus.properties - sample JNDI configuration
	
	# Register a ConnectionFactory in JNDI using the form:
	# connectionfactory.[jndi_name] = [ConnectionURL]
	connectionfactory.SBCF = amqps://[username]:[password]@[namespace].servicebus.windows.net
	
	# Register some queues in JNDI using the form
	# queue.[jndi_name] = [physical_name]
	# topic.[jndi_name] = [physical_name]
	queue.QUEUE = queue1


<p><strong>Configuring the ConnectionFactory</strong></p>

The entry used to define a **ConnectionFactory** in the Qpid Properties File JNDI Provider is of the following format:

	connectionfactory.[jndi_name] = [ConnectionURL]

Where [jndi_name] and [ConnectionURL] have the following meanings:

<table>
  <tr>
    <td>[jndi_name]</td>
    <td>The logical name of the ConnectionFactory. This is the name that will be resolved in the Java application using the JNDI IntialContext.lookup() method.</td>
  </tr>
  <tr>
    <td>[ConnectionURL]</td>
    <td>A URL that provides the JMS library with the information required to the AMQP broker.</td>
  </tr>
</table>

The format of the **ConnectionURL** is as follows:

	amqps://[username]:[password]@[namespace].servicebus.windows.net

Where [namespace], [username] and [password] have the following meanings:

<table>
  <tr>
    <td>[namespace]</td>
    <td>The Service Bus namespace obtained from the Azure Management Portal.</td>
  </tr>
  <tr>
    <td>[username]</td>
    <td>The Service Bus issuer name obtained from the Azure Management Portal.</td>
  </tr>
  <tr>
    <td>[password]</td>
    <td>URL encoded form of the Service Bus issuer key obtained from the Azure Management Portal.</td>
  </tr>
</table>

**Note**: You must URL-encode the password manually. A useful URL-encoding utility is available at [http://www.w3schools.com/tags/ref_urlencode.asp](http://www.w3schools.com/tags/ref_urlencode.asp).

For example, if the information obtained from the Azure Management portal is as follows:

<table>
  <tr>
    <td>Namespace:</td>
    <td>foo.servicebus.windows.net</td>
  </tr>
  <tr>
    <td>Issuer name:</td>
    <td>owner</td>
  </tr>
  <tr>
    <td>Issuer key:</td>
    <td>j9VYv1q33Ea+cbahWsHFYnLkEzrF0yA5SAqcLNvU7KM=</td>
  </tr>
</table>

Then in order to define a **ConnectionFactory** named "SBCF", the configuration string appears as follows:

	connectionfactory.SBCF = amqps://owner:j9VYv1q33Ea%2BcbahWsHFYnLkEzrF0yA5SAqcLNvU7KM%3D@foo.servicebus.windows.net

<p><strong>Configuring Destinations</strong></p>

The entry used to define a destination in the Qpid Properties File JNDI Provider is of the following format:

	queue.[jndi_name] = [physical_name]
or

	topic.[jndi_name] = [physical_name]

Where [jndi\_name] and [physical\_name] have the following meanings:

<table>
  <tr>
    <td>[jndi_name]</td>
    <td>The logical name of the destination. This is the name that will be resolved in the Java application using the JNDI IntialContext.lookup() method.</td>
  </tr>
  <tr>
    <td>[physical_name]</td>
    <td>The name of the Service Bus entity to which the application sends or receives messages.</td>
  </tr>
</table>

**Note**: When receiving from a Service Bus topic subscription, the physical name specified in JNDI should be the name of the topic. The subscription name is provided when the durable subscription is created in the JMS application code. The [Service Bus AMQP 1.0 Developer's Guide](http://msdn.microsoft.com/library/jj841071.aspx) provides more details on working with Service Bus topic subscriptions from JMS.

### Writing the JMS application

There are no special APIs or options required when using JMS with Service Bus. However, there are a few restrictions that will be covered later. As with any JMS application, the first thing required is configuration of the JNDI environment, to be able to resolve a **ConnectionFactory** and destinations.

<p><strong>Configuring the JNDI InitialContext</strong></p>

The JNDI environment is configured by passing a hashtable of configuration information into the constructor of the javax.naming.InitialContext class. The two required elements in the hashtable are the class name of the Initial Context Factory and the Provider URL. The following code shows how to configure the JNDI environment to use the Qpid properties file based JNDI Provider with a properties file named **servicebus.properties**.

	Hashtable<String, String> env = new Hashtable<String, String>(); 
	env.put(Context.INITIAL_CONTEXT_FACTORY, "org.apache.qpid.amqp_1_0.jms.jndi.PropertiesFileInitialContextFactory"); 
	env.put(Context.PROVIDER_URL, "servicebus.properties"); 
	InitialContext context = new InitialContext(env); 

### A simple JMS application using a Service Bus queue

The following example program sends JMS TextMessages to a Service Bus queue with the JNDI logical name of QUEUE, and receives the messages back.

	// SimpleSenderReceiver.java
	
	import javax.jms.*;
	import javax.naming.Context;
	import javax.naming.InitialContext;
	import java.io.BufferedReader;
	import java.io.InputStreamReader;
	import java.util.Hashtable;
	import java.util.Random;
	
	public class SimpleSenderReceiver implements MessageListener {
	    private static boolean runReceiver = true;
	    private Connection connection;
	    private Session sendSession;
	    private Session receiveSession;
	    private MessageProducer sender;
	    private MessageConsumer receiver;
	    private static Random randomGenerator = new Random();
	
	    public SimpleSenderReceiver() throws Exception {
	        // Configure JNDI environment
	        Hashtable<String, String> env = new Hashtable<String, String>();
	        env.put(Context.INITIAL_CONTEXT_FACTORY, 
                    "org.apache.qpid.amqp_1_0.jms.jndi.PropertiesFileInitialContextFactory");
	        env.put(Context.PROVIDER_URL, "servicebus.properties");
	        Context context = new InitialContext(env);
	
	        // Lookup ConnectionFactory and Queue
	        ConnectionFactory cf = (ConnectionFactory) context.lookup("SBCF");
	        Destination queue = (Destination) context.lookup("QUEUE");
	
	        // Create Connection
	        connection = cf.createConnection();
	
	        // Create sender-side Session and MessageProducer
	        sendSession = connection.createSession(false, Session.AUTO_ACKNOWLEDGE);
	        sender = sendSession.createProducer(queue);
	
	        if (runReceiver) {
	            // Create receiver-side Session, MessageConsumer,and MessageListener
	            receiveSession = connection.createSession(false, Session.CLIENT_ACKNOWLEDGE);
	            receiver = receiveSession.createConsumer(queue);
	            receiver.setMessageListener(this);
	            connection.start();
	        }
	    }
	
	    public static void main(String[] args) {
	        try {
	
	            if ((args.length > 0) && args[0].equalsIgnoreCase("sendonly")) {
	                runReceiver = false;
	            }
	
	            SimpleSenderReceiver simpleSenderReceiver = new SimpleSenderReceiver();
	            System.out.println("Press [enter] to send a message. Type 'exit' + [enter] to quit.");
	            BufferedReader commandLine = new java.io.BufferedReader(new InputStreamReader(System.in));
	
	            while (true) {
	                String s = commandLine.readLine();
	                if (s.equalsIgnoreCase("exit")) {
	                    simpleSenderReceiver.close();
	                    System.exit(0);
	                } else {
	                    simpleSenderReceiver.sendMessage();
	                }
	            }
	        } catch (Exception e) {
	            e.printStackTrace();
	        }
	    }
	
	    private void sendMessage() throws JMSException {
	        TextMessage message = sendSession.createTextMessage();
	        message.setText("Test AMQP message from JMS");
	        long randomMessageID = randomGenerator.nextLong() >>>1;
	        message.setJMSMessageID("ID:" + randomMessageID);
	        sender.send(message);
	        System.out.println("Sent message with JMSMessageID = " + message.getJMSMessageID());
	    }
	
	    public void close() throws JMSException {
	        connection.close();
	    }
	
	    public void onMessage(Message message) {
	        try {
	            System.out.println("Received message with JMSMessageID = " + message.getJMSMessageID());
	            message.acknowledge();
	        } catch (Exception e) {
	            e.printStackTrace();
	        }
	    }
	}	

### Running the application

Running the application produces output of the form:

	> java SimpleSenderReceiver
	Press [enter] to send a message. Type 'exit' + [enter] to quit.
	
	Sent message with JMSMessageID = ID:2867600614942270318
	Received message with JMSMessageID = ID:2867600614942270318
	
	Sent message with JMSMessageID = ID:7578408152750301483
	Received message with JMSMessageID = ID:7578408152750301483
	
	Sent message with JMSMessageID = ID:956102171969368961
	Received message with JMSMessageID = ID:956102171969368961
	exit

##Cross-platform messaging between JMS and .NET

This guide showed how to send and receive messages to and from Service Bus using JMS. However, one of the key benefits of AMQP 1.0 is that it enables applications to be built from components written in different languages, with messages exchanged reliably and at full fidelity.

Using the sample JMS application described above and a similar .NET application taken from a companion guide, [How to use AMQP 1.0 with the .NET Service Bus .NET API](http://aka.ms/lym3vk), you can exchange messages between .NET and Java. 

For more information about the details of cross-platform messaging using Service Bus and AMQP 1.0, see the [Service Bus AMQP 1.0 Developer's Guide](http://msdn.microsoft.com/library/jj841071.aspx).

### JMS to .NET

To demonstrate JMS to .NET messaging:

* Start the .NET sample application without any command-line arguments.
* Start the Java sample application with the "sendonly" command-line argument. In this mode, the application will not receive messages from the queue, it will only send.
* Press **Enter** a few times in the Java application console, which will cause messages to be sent.
* These messages are received by the .NET application.

<p><strong>Output from JMS application</strong></p>

	> java SimpleSenderReceiver sendonly
	Press [enter] to send a message. Type 'exit' + [enter] to quit.
	Sent message with JMSMessageID = ID:4364096528752411591
	Sent message with JMSMessageID = ID:459252991689389983
	Sent message with JMSMessageID = ID:1565011046230456854
	exit

<p><strong>Output from .NET application</strong></p>

	> SimpleSenderReceiver.exe	
	Press [enter] to send a message. Type 'exit' + [enter] to quit.
	Received message with MessageID = 4364096528752411591
	Received message with MessageID = 459252991689389983
	Received message with MessageID = 1565011046230456854
	exit

### .NET to JMS

To demonstrate .NET to JMS messaging:

* Start the .NET sample application with the "sendonly" command-line argument. In this mode, the application will not receive messages from the queue, it will only send.
* Start the Java sample application without any command-line arguments.
* Press **Enter** a few times in the .NET application console, which will cause messages to be sent.
* These messages are received by the Java application.

<p><strong>Output from .NET application</strong></p>

	> SimpleSenderReceiver.exe sendonly
	Press [enter] to send a message. Type 'exit' + [enter] to quit.
	Sent message with MessageID = d64e681a310a48a1ae0ce7b017bf1cf3	
	Sent message with MessageID = 98a39664995b4f74b32e2a0ecccc46bb
	Sent message with MessageID = acbca67f03c346de9b7893026f97ddeb
	exit


<p><strong>Output from JMS application</strong></p>

	> java SimpleSenderReceiver	
	Press [enter] to send a message. Type 'exit' + [enter] to quit.
	Received message with JMSMessageID = ID:d64e681a310a48a1ae0ce7b017bf1cf3
	Received message with JMSMessageID = ID:98a39664995b4f74b32e2a0ecccc46bb
	Received message with JMSMessageID = ID:acbca67f03c346de9b7893026f97ddeb
	exit

##Unsupported features and restrictions

The following restrictions exist when using JMS over AMQP 1.0 with Service Bus, namely:

* Only one **MessageProducer** or **MessageConsumer** is allowed per **Session**. If you need to create multiple **MessageProducers** or **MessageConsumers** in an application, create a dedicated **Session** for each of them.
* Volatile topic subscriptions are not currently supported.
* **MessageSelectors** are not currently supported.
* Temporary destinations, i.e., **TemporaryQueue**, **TemporaryTopic** are not currently supported, along with the **QueueRequestor** and **TopicRequestor** APIs that use them.
* Transacted sessions and distributed transactions are not supported.

##Summary

This how-to guide showed how to use Service Bus brokered messaging features (queues and publish/subscribe topics) from Java using the popular JMS API and AMQP 1.0.

You can also use Service Bus AMQP 1.0 from other languages, including .NET, C, Python, and PHP. Components built using these different languages can exchange messages reliably and at full fidelity using the AMQP 1.0 support in Service Bus. For more information, see the [Service Bus AMQP 1.0 Developer's Guide](http://msdn.microsoft.com/library/jj841071.aspx).

##Further information

* [AMQP 1.0 support in Azure Service Bus](http://aka.ms/pgr3dp)
* [How to use AMQP 1.0 with the Service Bus .NET API](http://aka.ms/lym3vk)
* [Service Bus AMQP 1.0 Developer's Guide](http://msdn.microsoft.com/library/jj841071.aspx)
* [How to Use Service Bus Queues](http://azure.microsoft.com/develop/net/how-to-guides/service-bus-queues/)

