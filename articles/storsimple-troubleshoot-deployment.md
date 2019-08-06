<properties 
   pageTitle="Troubleshoot StorSimple device deployment issues"
   description="Describes how to diagnose and fix errors that occur when you first deploy StorSimple."
   services="storsimple"
   documentationCenter="NA"
   authors="SharS"
   manager="adinah"
   editor="tysonn" />
<tags 
   ms.service="storsimple"
   ms.devlang="NA"
   ms.topic="article"
   ms.tgt_pltfrm="NA"
   ms.workload="TBD"
   ms.date="04/06/2015"
   ms.author="v-sharos" />

# Troubleshoot StorSimple device deployment issues

## Overview

This article provides helpful troubleshooting guidance for your Microsoft Azure StorSimple deployment. It describes common issues, possible causes, and recommended steps to help you resolve problems that you might experience when you configure StorSimple. This information applies to both the StorSimple on-premises physical device and the StorSimple virtual device.

> [AZURE.NOTE] Device configuration-related issues that you may face can occur when you deploy the device for the first time, or they can occur later, when the device is operational. This article focuses on troubleshooting first-time deployment issues. To troubleshoot an operational device, go to [Troubleshoot operational device issues](storsimple-troubleshoot-operational-device.md).

This article also describes the tools for troubleshooting StorSimple deployments and provides a step-by-step troubleshooting example.

## First-time deployment issues

If you run into an issue when deploying your device for the first time, consider the following:

- If you are troubleshooting a physical device, make sure that the hardware has been installed and configured as described in [Hardware installation of your device](https://msdn.microsoft.com/library/azure/dn772375.aspx).
- Check prerequisites for deployment. Make sure that you have all the information described in the [deployment checklist](storsimple-deployment-walkthrough.md#pre-installation-checklist).
- Review the StorSimple Release Notes to see if the problem is described. The release notes include workarounds for known installation problems. 

During device deployment, the most common issues that users face occur when they run the setup wizard and when they register the device via Windows PowerShell for StorSimple. (You use Windows PowerShell for StorSimple to register and configure your StorSimple device. For more information on device registration, see [Register your device](https://msdn.microsoft.com/library/azure/dn757742.aspx).)

The following sections can help you resolve issues that you encounter when you configure the StorSimple device for the first time.

## First-time setup wizard process

The following steps summarize the setup wizard process. For detailed setup information, see [Deploy your on-premises StorSimple device](storsimple-deployment-walkthrough.md).

1. Run the [Invoke-HcsSetupWizard](https://technet.microsoft.com/library/dn688135.aspx) cmdlet to start the setup wizard that will guide you through the remaining steps. 
2. Configure the network: the setup wizard lets you configure network settings for the DATA 0 network interface on your StorSimple device. These settings include the following:
  - Virtual IP (VIP), subnet mask, and gateway – The [Set-HcsNetInterface](https://technet.microsoft.com/library/dn688161.aspx) cmdlet is executed in the background. It configures the IP address, subnet mask, and gateway for the DATA 0 network interface on your StorSimple device.
  - Primary DNS server – The [Set-HcsDnsClientServerAddress](https://technet.microsoft.com/library/dn688172.aspx) cmdlet is executed in the background. It configures the DNS settings for your StorSimple solution.
  - NTP server – The [Set-HcsNtpClientServerAddress](https://technet.microsoft.com/library/dn688138.aspx) cmdlet is executed in the background. It configures the NTP server settings for your StorSimple solution.
  - Optional web proxy – The [Set-HcsWebProxy](https://technet.microsoft.com/library/dn688154.aspx) cmdlet is executed in the background. It sets and enables the web proxy configuration for your StorSimple solution.
3. Set up the passwords: the next step is to set up device administrator and StorSimple Snapshot Manager passwords.
  - The device administrator password is used to log on to your device. The default device password is **Password1**.
  - The StorSimple Snapshot Manager password is required when you configure a device to use StorSimple Snapshot Manager. You need to first set the password in the setup wizard, and then you can set and change it from the StorSimple Manager service. This password authenticates the device with StorSimple Snapshot Manager.
 
    > [AZURE.IMPORTANT] Passwords are collected before registration, but applied only after you successfully register the device. If there is a failure to apply a password, you will be prompted to supply the password again until the required passwords (that meet the complexity requirements) are collected.

4. Register the device: the final step is to register the device with the StorSimple Manager service running in Microsoft Azure. The registration requires you to [get the service registration key.](https://msdn.microsoft.com/library/azure/cd4dee49-6ae8-4ff0-b79b-74b2027cb694#sec03) from the Azure Management Portal, and provide it in the setup wizard. After the device is successfully registered, a service data encryption key is provided to you. Be sure to keep this encryption key in a safe location because it will be required to register all subsequent devices with the service.

## Common errors during device deployment

The following tables list the common errors that you might encounter when you:

- Configure the required network settings.
- Configure the optional web proxy settings.
- Set up the device administrator and StorSimple Snapshot Manager passwords. 
- Register the device. 

### Errors that occur when you configure the required network settings

| No.| Error message | Possible causes | Recommended action |
| ---| ------------- | --------------- | ------------------ |
| 1  | Invoke-HcsSetupWizard: This command can only be run on the active controller. | Configuration was being performed on the passive controller.| Run this command from the active controller. For more information, see [Identify an active controller on your device](https://msdn.microsoft.com/library/azure/dn790262.aspx).|
| 2 | Invoke-HcsSetupWizard: Device not ready. | There are issues with the network connectivity on DATA 0.| Check the physical network connectivity on DATA 0.|
| 3 | Invoke-HcsSetupWizard: There is an IP address conflict with another system on the network (Exception from HRESULT: 0x80070263). | The IP supplied for DATA 0 was already in use by another system. | Provide a new IP that is not in use.|
| 4 | Invoke-HcsSetupWizard: A cluster resource failed. (Exception from HRESULT:0x800713AE). | Duplicate VIP. The supplied IP is already in use.| Provide a new IP that is not in use.|
| 5 | Invoke-HcsSetupWizard: Invalid IPv4 address. | The IP address is provided in an incorrect format.| Check the format and supply your IP address again. For more information, see [Ipv4 Addressing][1]. |
| 6 | Invoke-HcsSetupWizard: Invalid IPv6 address. | The IP address is provided in an incorrect format.| Check the format and supply your IP address again. For more information, see [Ipv6 Addressing][2].|
| 7 | Invoke-HcsSetupWizard: There are no more endpoints available from the endpoint mapper.(Exception from HRESULT: 0x800706D9) | The cluster functionality is not working. | [Contact Microsoft Support](https://msdn.microsoft.com/library/azure/dn757750.aspx) for next steps.

### Errors that occur when you configure the optional web proxy settings

| No.| Error message | Possible causes | Recommended action |
| ---| ------------- | --------------- | ------------------ |
| 1  | Invoke-HcsSetupWizard: Invalid parameter (Exception from HRESULT: 0x80070057) | One of the parameters provided for the proxy settings is not valid.| The URI is not provided in the correct format. Use the following format: http://*<IP address or FQDN of the web proxy server>*:*<TCP port number>* |
| 2 | Invoke-HcsSetupWizard: RPC server not available (Exception from HRESULT: 0x800706ba) | The root cause is one of the following:<ol><li>The cluster is not up.</li><li>The passive controller cannot communicate with the active controller, and the command is run from passive controller.</li></ol> | Depending on the root cause:<ol><li>[Contact Microsoft Support](https://msdn.microsoft.com/library/azure/dn757750.aspx) to make sure that the cluster is up.</li><li>Run the command from the active controller. If you want to run the command from the passive controller, you will need to ensure that the passive controller can communicate with the active controller. You will need to [contact Microsoft Support](https://msdn.microsoft.com/library/azure/dn757750.aspx) if this connectivity is broken.</li></ol> |
| 3 | Invoke-HcsSetupWizard: RPC call failed (Exception from HRESULT: 0x800706be) | Cluster is down. | [Contact Microsoft Support](https://msdn.microsoft.com/library/azure/dn757750.aspx) to make sure that the cluster is up.|
| 4 | Invoke-HcsSetupWizard: Cluster resource not found (Exception from HRESULT: 0x8007138f) | The cluster resource is not found. This can happen when the installation was not correct. | You may need to reset the device to the factory default settings. [Contact Microsoft Support](https://msdn.microsoft.com/library/azure/dn757750.aspx) to create a cluster resource.|
| 5 | Invoke-HcsSetupWizard: Cluster resource not online (Exception from HRESULT: 0x8007138c)| Cluster resources are not online. | [Contact Microsoft Support](https://msdn.microsoft.com/library/azure/dn757750.aspx) for next steps.|

### Errors that occur when you set up the device administrator and StorSimple Snapshot Manager passwords

The default device administrator password is **Password1**. This password expires after the first log on; therefore, you will need to use the setup wizard to change it. You must provide a new device administrator password when you register the device for the first time. 

If you use the StorSimple Snapshot Manager software running on the Windows Server host to manage the device, then you must also provide a StorSimple Snapshot Manager password during first-time registration. 

Make sure that your passwords meet the following requirements:

- Your device administrator password should be between 8 and 15 characters long.
- Your StorSimple Snapshot Manager password should be 14 or 15 characters long.
- Passwords should contain 3 of the following 4 character types: lowercase, uppercase, numeric, and special. 
- Your password cannot be the same as the last 24 passwords.

In addition, keep in mind that passwords expire every year, and can be changed only after you successfully register the device. If the registration fails for any reason, the passwords will not be changed. 
For more information on device administrator and StorSimple Snapshot Manager passwords, go to [Passwords](https://msdn.microsoft.com/library/azure/00fb2b46-e29e-4c4d-90a8-1dbb7db3672e#sec_02).

You may encounter one or more of the following errors when setting up the device administrator and StorSimple Snapshot Manager passwords.

| No.| Error message | Recommended action |
| ---| ------------- | ------------------ | 
| 1  | The password exceeds the maximum length. |Use a password that meets these requirements:<ul><li>Your device administrator password must be between 8 and 15 characters long.</li><li>Your StorSimple Snapshot Manager password must be 14 or 15 characters long.</li></ul> | 
| 2 | The password does not meet the required length. | Use a password that meets these requirements:<ul><li>Your device administrator password must be between 8 and 15 characters long.</li><li>Your StorSimple Snapshot Manager password must be 14 or 15 characters long.</lu></ul> |
| 3 | The password must contain lowercase characters. | Passwords must contain 3 of the following 4 character types: lowercase, uppercase, numeric, and special. Make sure that your password meets these requirements. |
| 4 | The password must contain numeric characters. | Passwords must contain 3  of the following 4 character types: lowercase, uppercase, numeric, and special. Make sure that your password meets these requirements. |
| 5 | The password must contain special characters. | Passwords must contain 3 of the following 4 character types: lowercase, uppercase, numeric, and special. Make sure that your password meets these requirements. |
| 6 | The password must contain 3 of the following 4 character types: uppercase, lowercase, numeric, and special. | Your password does not contain the required types of characters. Make sure that your password meets these requirements. |
| 7 | Parameter does not match confirmation. | Make sure that your password meets all requirements and that you entered it correctly. |
| 8 | Your password cannot match the default. | The default password is  *Password1*. You need to change this password after you log on for the first time. |
| 9 | The password you have entered does not match the device password. Please retype the password. | Check the password and type it again. |

Passwords are collected before the device is registered, but are applied only after successful registration. The password recovery workflow requires the device to be registered. 

> [AZURE.IMPORTANT] In general, if an attempt to apply a password fails, then the software repeatedly attempts to collect the password until it is successful. In rare instances, the password cannot be applied. In this situation, you can register the device and proceed, however the passwords will not be changed. You will receive no indication as to which password was not changed — the device administrator password or the StorSimple Snapshot Manager password. If this situation occurs, we recommend that you change both passwords.

You can reset the passwords from the Management Portal of your StorSimple Manager service. For more information, see: 

- [Configure the device administrator password](https://msdn.microsoft.com/library/azure/02f1412f-e196-4a88-8eda-2113247ea47c#sec09)
- [Configure the StorSimple Snapshot Manager password](https://msdn.microsoft.com/library/azure/02f1412f-e196-4a88-8eda-2113247ea47c#sec08)

### Errors that occur when you register the device

You use the StorSimple Manager service running in Microsoft Azure to register the device. You could encounter one or more of the following issues during device registration.

| No.| Error message | Possible causes | Recommended action |
| ---| ------------- | --------------- | ------------------ |
| 1  | Error 350027: Failed to register the device with the StorSimple Manager. |  | Wait for a few minutes and then try the operation again. If the issue persists, [contact Microsoft Support](https://msdn.microsoft.com/library/azure/dn757750.aspx).|
| 2  | Error 350013: An error has occurred in registering the device. This could be due to incorrect service registration key. | | Please register the device again with the correct service registration key. For more information, see [Get the service registration key.](https://msdn.microsoft.com/library/azure/cd4dee49-6ae8-4ff0-b79b-74b2027cb694#sec03) |
| 3 | Error 350063: Authentication to StorSimple Manager service passed but registration failed. Please retry the operation after some time. | This error indicates that authentication with ACS has passed but the register call made to the service has failed. This could be a result of a sporadic network glitch. | If the issue persists, please [contact Microsoft Support](https://msdn.microsoft.com/library/azure/dn757750.aspx). |
| 4 | Error 350049: The service could not be reached during registration. | When the call is made to the service, a web exception is received. In some cases, this may get fixed with retrying the operation later. | Please check your IP address and DNS name and then retry the operation. If the problem persists, [contact Microsoft Support.](https://msdn.microsoft.com/library/azure/dn757750.aspx) | 
| 5 | Error 350031: The device has already been registered. | | No action necessary. |
| 6 | Error 350016: Device Registration failed. | |Please make sure the registration key is correct. |
| 7 | Invoke-HcsSetupWizard: An error has occurred while registering your device; this could be due to incorrect IP address or DNS name. Please check your network settings and try again. If the problem persists, [contact Microsoft Support](https://msdn.microsoft.com/library/azure/dn757750.aspx).(Error 350050) | Ensure that your device can ping the outside network. If you do not have connectivity to outside network, the registration may fail with this error. This error may be a combination of one or more of the following:<ul><li>Incorrect IP</li><li>Incorrect subnet</li><li>Incorrect gateway</li><li>Incorrect DNS settings</li></ul> | See the steps in the [Step-by-step troubleshooting example](#step-by-step-storsimple-troubleshooting-example). |
| 8 | Invoke-HcsSetupWizard: The current operation failed due to an internal service error [0x1FBE2]. Please retry the operation after sometime. If the issue persists, please contact Microsoft Support. | This is a generic error thrown for all user invisible errors from service or agent. The most common reason may be that the ACS authentication has failed. A possible cause for the failure is that there are issues with the NTP server configuration and time on the device is not set correctly. | Correct the time (if there are issues) and then retry the registration operation. If this issue persists, [contact Microsoft Support](https://msdn.microsoft.com/library/azure/dn757750.aspx) for next steps. |
| 9 | Warning: Could not activate the device. Your device administrator and StorSimple Snapshot Manager passwords have not been changed. | If the registration fails, the device administrator and StorSimple Snapshot Manager passwords are not changed. |

## Tools for troubleshooting StorSimple deployments

StorSimple includes several tools that you can use to troubleshoot your StorSimple solution. These include:

- Support packages and device logs 
- Cmdlets specifically designed for troubleshooting 

## Support packages and device logs available for troubleshooting

A support package contains all the relevant logs that can assist the Microsoft Support team with troubleshooting device issues. You can use Windows PowerShell for StorSimple to generate an encrypted support package that you can then share with support personnel.

### To view the logs or the contents of the support package

1. Use Windows PowerShell for StorSimple to generate a support package as described in [Generate a support package](https://msdn.microsoft.com/library/azure/dn772348.aspx).

2. Download the [decryption script](https://gallery.technet.microsoft.com/scriptcenter/Script-to-decrypt-a-a8d1ed65) locally on your client computer.

3. Use this [step-by-step procedure](https://msdn.microsoft.com/library/azure/dn762117.aspx) to open and decrypt the support package.

4. The decrypted support package logs are in etw/etvx format. You can perform the following steps to view these files in Windows Event Viewer:
  1. Run the **eventvwr** command on your Windows client. This will start the Event Viewer.
  2. In the **Actions** pane, click **Open Saved Log** and point to the log files in etvx/etw format (the support package). You can now view the file. After you open the file, you can right-click and save the the file as text.
   
    > [AZURE.IMPORTANT] You can also use the **Get-WinEvent** cmdlet to open these file in Windows PowerShell. For more information, see [Get-WinEvent](https://technet.microsoft.com/library/hh849682.aspx) in the Windows PowerShell cmdlet reference documentation.

5. When the logs open in Event Viewer, look for the following logs that contain issues related to the device configuration:

  - hcs_pfconfig/Operational Log
  - hcs_pfconfig/Config

6. In the log files, search for strings related to the cmdlets called by the setup wizard. See [First-time setup wizard process](#first-time-setup-wizard-process) for a list of these cmdlets. 

7. If you are not able to figure out the cause of the problem, you can [contact Microsoft Support](https://msdn.microsoft.com/library/azure/dn757750.aspx) for next steps. Use the steps in [Start a support session](https://msdn.microsoft.com/library/azure/dn757804.aspx) when you contact Microsoft Support for assistance.

## Cmdlets available for troubleshooting

Use the following Windows PowerShell cmdlets to detect connectivity errors.

- Get-NetAdapter: Use this cmdlet to detect the health of network interfaces. 

- Test-Connection: Use this cmdlet to check the network connectivity inside and outside of the network.

- Test-HcsmConnection: Use this cmdlet to check the connectivity of a successfully registered device.

## Troubleshoot with the Get-NetAdapter cmdlet

When you configure network interfaces for a first-time device deployment, the hardware status is not available in the StorSimple Manager service UI because the device is not yet registered with the service. Additionally, the Hardware Status page may not always correctly reflect the state of the device, especially if there are issues that affect service synchronization. In these situations, you can use the Get-NetAdapter cmdlet to determine the health and status of your network interfaces.

### To see a list of all the network adapters on your device

1. Start Windows PowerShell for StorSimple, and then type **Get-NetAdapter**. 

2. Use the output of the Get-NetAdapter cmdlet and the following guidelines to understand the status of your network interface.
  - If the interface is healthy and enabled, the **ifIndex** status is shown as **Up**.
  - If the interface is healthy but is not physically connected (by a network cable), the **ifIndex** is shown as **Disabled**.
  - If the interface is healthy but not enabled, the **ifIndex** status is shown as **NotPresent**.
  - If the interface does not exist, it does not appear in this list. The StorSimple Manager service UI will still show this interface in a failed state.

For more information on how to use this cmdlet, go to [GetNetAdapter](https://technet.microsoft.com/library/jj130867.aspx) in the Windows PowerShell cmdlet reference. 

The following sections show samples of output from the Get-NetAdapter cmdlet. 

 In these samples, controller 0 was the passive controller, and was configured as follows:

- DATA 0, DATA 1, DATA 2, and DATA 3 network interfaces existed on the device.
- DATA 4 and DATA 5 network interface cards were not present; therefore, they are not listed in the output.
- DATA 0 was enabled.

Controller 1 was the active controller, and was configured as follows:

- DATA 0, DATA 1, DATA 2, DATA 3, DATA 4, and DATA 5 network interfaces existed on the device.
- DATA 0 was enabled.

**Sample output – controller 0**

The following is the output from controller 0 (the passive controller). DATA 1, DATA 2, and DATA 3 are not connected. DATA 4 and DATA 5 are not listed because they are not present on the device. 

     Controller0>Get-NetAdapter
     Name                 InterfaceDescription                        ifIndex  Status
     ------               --------------------                        -------  ----------
     DATA3                Mellanox ConnectX-3 Ethernet Adapter #2     17       NotPresent
     DATA2                Mellanox ConnectX-3 Ethernet Adapter        14       NotPresent
     Ethernet 2           HCS VNIC                                    13       Up
     DATA1                Intel(R) 82574L Gigabit Network Co...#2     16       NotPresent
     DATA0                Intel(R) 82574L Gigabit Network Conn...     15       Up


**Sample output – controller 1**

The following is the output from controller 1 (the active controller). Only the DATA 0 network interface on the device is configured and working.

     Controller1>Get-NetAdapter
     Name                 InterfaceDescription                        ifIndex  Status
     ------               --------------------                        -------  ----------
     DATA3                Mellanox ConnectX-3 Ethernet Adapter        18       NotPresent
     DATA2                Mellanox ConnectX-3 Ethernet Adapter #2     19       NotPresent
     DATA1                Intel(R) 82574L Gigabit Network Co...#2     16       NotPresent
     DATA0                Intel(R) 82574L Gigabit Network Conn...     15       Up
     Ethernet 2           HCS VNIC                                    13       Up
     DATA5                Intel(R) Gigabit ET Dual Port Server...     14       NotPresent
     DATA4                Intel(R) Gigabit ET Dual Port Serv...#2     17       NotPresent

 
## Troubleshoot with the Test-Connection cmdlet

You can use the Test-Connection cmdlet to determine whether your StorSimple device can connect to the outside network. If all the networking parameters, including the DNS, are configured correctly in the setup wizard, you can use the Test-Connection cmdlet to ping a known address outside of the network, such as outlook.com. 

See the following samples of output from the Test-Connection cmdlet. 

> [AZURE.NOTE] In the first sample, the device is configured with an incorrect DNS. In the second sample, the DNS is correct.
 
**Sample output – incorrect DNS**

In the following sample, there is no output for the IPV4 and IPV6 addresses, which indicates that the DNS is not resolved. This means that there is no connectivity to the outside network and a correct DNS needs to be supplied. 

     Source        Destination     IPV4Address      IPV6Address
     ------        -----------     -----------      -----------
     HCSNODE0      outlook.com
     HCSNODE0      outlook.com
     HCSNODE0      outlook.com
     HCSNODE0      outlook.com

**Sample output – correct DNS**

In the following sample, the DNS returns the IPV4 address, indicating that the DNS is configured correctly. This confirms that there is connectivity to the outside network. 

     Source        Destination     IPV4Address      IPV6Address
     ------        -----------     -----------      -----------
     HCSNODE0      outlook.com     132.245.92.194
     HCSNODE0      outlook.com     132.245.92.194
     HCSNODE0      outlook.com     132.245.92.194
     HCSNODE0      outlook.com     132.245.92.194

## Troubleshoot with the Test-HcsmConnection cmdlet

Use the Test-HcsmConnection cmdlet for a device that is already connected to and registered with your StorSimple Manager service. This cmdlet helps you verify the connectivity between a registered device and the corresponding StorSimple Manager service. You can run this command on Windows PowerShell for StorSimple. 

### To run the Test-HcsmConnection cmdlet

1. Make sure that the device is registered.

2. Check the device status. If the device is deactivated, in maintenance mode, or offline, you might see the following errors: 

   - ErrorCode.CiSDeviceDecommissioned – this indicates that the device is deactivated.
   - ErrorCode.DeviceNotReady – this indicates that the device is in maintenance mode.
   - ErrorCode.DeviceNotReady – this indicates that the device is not online.

3. Verify that the StorSimple Manager service is running (use the [Get-ClusterResource](https://technet.microsoft.com/library/ee461004.aspx) cmdlet). If the service is not running, you might see the following errors:

   - ErrorCode.CiSApplianceAgentNotOnline
   - ErrorCode.CisPowershellScriptHcsError – this indicates that there was an exception when you ran Get-ClusterResource.

4. Check the Access Control Service (ACS) token. If it throws a web exception, it might be the result of a gateway problem, a missing proxy authentication, an incorrect DNS, or an authentication failure. You might see the following errors:

   - ErrorCode.CiSApplianceGateway – this indicates an HttpStatusCode.BadGateway exception: the name resolver service could not resolve the host name. 
   - ErrorCode.CiSApplianceProxy – this indicates an HttpStatusCode.ProxyAuthenticationRequired exception (HTTP status code 407): the client could not authenticate with the proxy server. 
   - ErrorCode.CiSApplianceDNSError – this indicates a WebExceptionStatus.NameResolutionFailure exception: the name resolver service could not resolve the host name. .
   - ErrorCode.CiSApplianceACSError – this indicates that the service returned an authentication error, but there is connectivity.
   
    If it does not throw a web exception, check for ErrorCode.CiSApplianceFailure. This indicates that the appliance failed.

5. Check the cloud service connectivity. If the service throws a web exception, you might see the following errors:

  - ErrorCode.CiSApplianceGateway – this indicates an HttpStatusCode.BadGateway exception: an intermediate proxy server received a bad request from another proxy or from the original server.
  - ErrorCode.CiSApplianceProxy – this indicates an HttpStatusCode.ProxyAuthenticationRequired exception (HTTP status code 407): the client could not authenticate with the proxy server. 
  - ErrorCode.CiSApplianceDNSError – this indicates a WebExceptionStatus.NameResolutionFailure exception: the name resolver service could not resolve the host name. .
  - ErrorCode.CiSApplianceACSError – this indicates that the service returned an authentication error, but there is connectivity.
  
    If it does not throw a web exception, check for ErrorCode.CiSApplianceSaasServiceError. This indicates a problem with the StorSimple Manager service.
 
6. Check Azure Service Bus connectivity. ErrorCode.CiSApplianceServiceBusError indicates that the device cannot connect to the Service Bus.
 
The log files CiSCommandletLog0Curr.errlog and CiSAgentsvc0Curr.errlog will have more information, such as exception details. 

For more information on how to use the cmdlet, go to [Test-HcsmConnection](https://technet.microsoft.com/library/dn715782.aspx) in the Windows PowerShell reference documentation.

> [AZURE.IMPORTANT] You can run this cmdlet for both the active and the passive controller. 
 
See the following samples of output from the Test-HcsmConnection cmdlet. 

**Sample output – successfully registered device**

The first sample is from a device that is successfully registered with the StorSimple Manager service and has no connectivity issues. 

     Controller1>Test-HcsmConnection -verbose
     Checking device state  ... Success.
     Device registered successfully
     Checking device authentication.  ... This operation will take few minutes to complete....
     Checking device authentication.  ... Success.
     Checking connectivity from device to StorSimple Manager service.  ... This operation will take few minutes to complete....
     Checking connectivity from device to StorSimple Manager service.  ... Success.
     Checking connectivity from StorSimple Manager service to StorSimple device. .... Success.
     Controller1>

**Sample output – offline device**

This sample is from a device that has a status of **Offline** in the Management Portal.

     Checking device state: Success 
     Device is registered successfully 
     Checking connectivity from device to SaaS.. Failure

The device could not connect using the current web proxy configuration. This could be an issue with the web proxy configuration or a network connectivity problem. In this case, you should make sure that your web proxy settings are correct and your web proxy servers are online and reachable. 

## Step-by-step StorSimple troubleshooting example

The following example shows step-by-step troubleshooting of a StorSimple deployment. In the example scenario, device registration fails with an error message indicating that the network settings or the DNS name is incorrect.

The error message returned is:

     Invoke-HcsSetupWizard: An error has occurred while registering the device. This could be due to incorrect IP address or DNS name. Please check your network settings and try again. If the problems persist, contact Microsoft Support.
     +CategoryInfo: Not specified
     +FullyQualifiedErrorID: CiSClientCommunicationErros, Microsoft.HCS.Management.PowerShell.Cmdlets.InvokeHcsSetupWizardCommand

The error could be caused by any of the following:

- Incorrect hardware installation
- Faulty network interface(s)
- Incorrect IP address, subnet mask, gateway, primary DNS server, or web proxy
- Incorrect registration key
- Incorrect firewall settings

### To locate and fix the device registration problem

1. Check your device configuration: on the active controller, run **Invoke-HcsSetupWizard**.

     > [AZURE.NOTE] The setup wizard must run on the active controller. To verify that you are connected to the active controller, look at the banner presented in the serial console. The banner indicates whether you are connected to controller 0 or controller 1, and whether the controller is active or passive. For more information, go to [Identify an active controller on your device](https://msdn.microsoft.com/library/azure/dn790262.aspx).
 
2. Make sure that the device is cabled correctly: check the network cabling on the device back plane. The cabling is specific to the device model. For more information, go to [Cable your 8100 device](https://msdn.microsoft.com/library/azure/dn757738.aspx) or [Cable your 8600 device](https://msdn.microsoft.com/library/azure/dn757762.aspx).

     > [AZURE.NOTE] If you are using 10 GbE network ports, you will need to use the provided QSFP-SFP adapters and SFP cables. For more information, see the [list of cables, switches, and transceivers recommended by the OEM supplier for Mellanox ports](http://www.mellanox.com/page/cables?mtag=cable_overview).
 
3. Verify the health of the network interface:

   - Use the Get-NetAdapter cmdlet to detect the health of the network interfaces for DATA 0. 
   - If the link isn't functioning, the **ifindex** status will indicate that the interface is down. You will then need to check the network connection of the port to the appliance and to the switch. You will also need to rule out bad cables. 
   - If you suspect that the DATA 0 port on the active controller has failed, you can confirm this by connecting to the DATA 0 port on controller 1. To confirm this, disconnect the network cable from the back of the device from controller 0, connect the cable to controller 1, and then run the Get-NetAdapter cmdlet again. 
   If the DATA 0 port on a controller fails, [contact Microsoft Support](https://msdn.microsoft.com/library/azure/dn757750.aspx) for next steps. You might need to replace the controller on your system.
 
4. Verify the connectivity to the switch:
   - Make sure that DATA 0 network interfaces on controller 0 and controller 1 in your primary enclosure are on the same subnet. 
   - Check the hub or router. Typically, you should connect both controllers to the same hub or router. 
   - Make sure that the switches you use for the connection have DATA 0 for both controllers in the same vLAN.
   
5. Eliminate any user errors:

  - Run the setup wizard again (run **Invoke-HcsSetupWizard**), and enter the values again to make sure that there are no errors. 
  - Verify the registration key used. The same registration key can be used to connect multiple devices to a StorSimple Manager service. Use the procedure in [Get the service registration key](https://msdn.microsoft.com/library/azure/cd4dee49-6ae8-4ff0-b79b-74b2027cb694#sec03) to ensure that you are using the correct registration key.

    > [AZURE.IMPORTANT] If you have multiple services running, you will need to ensure that the registration key for the appropriate service is used to register the device. If you have registered a device with the wrong StorSimple Manager service, you will need to [contact Microsoft Support](https://msdn.microsoft.com/library/azure/dn757750.aspx) for next steps. You may have to perform a factory reset of the device (which could result in data loss) to then connect it to the intended service.

6. Use the Test-Connection cmdlet to verify that you have connectivity to the outside network. For more information, go to [Troubleshoot with the Test-Connection cmdlet](#troubleshoot-with-the-test-connection-cmdlet).

7. Check for firewall interference. If you have verified that the virtual IP (VIP), subnet, gateway, and DNS settings are all correct, and you still see connectivity issues, then it is possible that your firewall is blocking communication between your device and the outside network. You need to ensure that ports 80 and 443 are available on your StorSimple device for outbound communication. For more information, see [Networking requirements for StorSimple device](https://msdn.microsoft.com/library/azure/dn772371.aspx).

8. Look at the logs. Go to [Support packages and device logs available for troubleshooting](#support-packages-and-device-logs-available-for-troubleshooting).

9. If the preceding steps do not resolve the problem, [contact Microsoft Support](https://msdn.microsoft.com/library/azure/dn757750.aspx) for assistance.

## Next steps
[Troubleshoot an operational device](storsimple-troubleshoot-operational-device.md)

<!--Link references-->

[1]: https://technet.microsoft.com/library/dd379547(v=ws.10).aspx
[2]: https://technet.microsoft.com/library/dd392266(v=ws.10).aspx