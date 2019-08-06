<properties
	pageTitle="How to use the service management API (Python) - feature guide"
	description="Learn how to programmatically perform common service management tasks from Python."
	services="cloud-services"
	documentationCenter="python"
	authors="huguesv"
	manager="wpickett"
	editor=""/>

<tags
	ms.service="cloud-services"
	ms.workload="tbd"
	ms.tgt_pltfrm="na"
	ms.devlang="python"
	ms.topic="article"
	ms.date="03/11/2015"
	ms.author="huvalo"/>

# How to use Service Management from Python

This guide will show you how to programmatically perform common service management tasks from Python. The **ServiceManagementService** class in the [Azure SDK for Python][download-SDK-Python] supports programmatic access to much of the service management-related functionality that is available in the [management portal][management-portal] (such as **creating, updating, and deleting cloud services, deployments, data management services and virtual machines**). This functionality can be useful in building applications that need programmatic access to service management.

## <a name="WhatIs"> </a>What is Service Management
The Service Management API provides programmatic access to much of the service management functionality available through the [management portal][management-portal]. The Azure SDK for Python allows you to manage your cloud services and storage accounts.

To use the Service Management API, you will need to [create an Azure account](http://www.windowsazure.com/pricing/free-trial/).

## <a name="Concepts"> </a>Concepts
The Azure SDK for Python wraps the [Azure Service Management API][svc-mgmt-rest-api], which is a REST API. All API operations are performed over SSL and mutually authenticated using X.509 v3 certificates. The management service may be accessed from within a service running in Azure, or directly over the Internet from any application that can send an HTTPS request and receive an HTTPS response.

## <a name="Connect"> </a>How to: Connect to service management
To connect to the Service Management endpoint, you need your Azure subscription ID and a valid management certificate. You can obtain your subscription ID through the [management portal][management-portal].

> [AZURE.NOTE] Since Azure SDK for Python v0.8.0, it is now possible to use certificates created with OpenSSL when running on Windows.  This requires Python 2.7.4 or later. We recommend users to use OpenSSL instead of .pfx, since support for .pfx certificates will likely be removed in the future.

### Management certificates on Windows/Mac/Linux (OpenSSL)
You can use [OpenSSL](http://www.openssl.org/) to create your management certificate.  You actually need to create two certificates, one for the server (a `.cer` file) and one for the client (a `.pem` file). To create the `.pem` file, execute this:

	`openssl req -x509 -nodes -days 365 -newkey rsa:1024 -keyout mycert.pem -out mycert.pem`

To create the `.cer` certificate, execute this:

	`openssl x509 -inform pem -in mycert.pem -outform der -out mycert.cer`

For more information about Azure certificates, see [Managing Certificates in Azure](http://msdn.microsoft.com/en-us/library/windowsazure/gg981929.aspx). For a complete description of OpenSSL parameters, see the documentation at [http://www.openssl.org/docs/apps/openssl.html](http://www.openssl.org/docs/apps/openssl.html).

After you have created these files, you will need to upload the `.cer` file to Azure via the "Upload" action of the "Settings" tab of the [management portal][management-portal], and you will need to make note of where you saved the `.pem` file.

After you have obtained your subscription ID, created a certificate, and uploaded the `.cer` file to Azure, you can connect to the Azure management endpoint by passing the subscription id and the path to the `.pem` file to **ServiceManagementService**:

	from azure import *
	from azure.servicemanagement import *

	subscription_id = '<your_subscription_id>'
	certificate_path = '<path_to_.pem_certificate>'

	sms = ServiceManagementService(subscription_id, certificate_path)

In the example above, `sms` is a **ServiceManagementService** object. The **ServiceManagementService** class is the primary class used to manage Azure services.

### Management certificates on Windows (MakeCert)

You can create a self-signed management certificate on your machine using `makecert.exe`.  Open a **Visual Studio command prompt** as an **administrator** and use the following command, replacing *AzureCertificate* with the certificate name you would like to use.

    makecert -sky exchange -r -n "CN=AzureCertificate" -pe -a sha1 -len 2048 -ss My "AzureCertificate.cer"

The command will create the `.cer` file, and install it in the **Personal** certificate store. For more details, see [Create and Upload a Management Certificate for Azure](http://msdn.microsoft.com/en-us/library/windowsazure/gg551722.aspx).

After you have created the certificate, you will need to upload the `.cer` file to Azure via the "Upload" action of the "Settings" tab of the [management portal][management-portal].

After you have obtained your subscription ID, created a certificate, and uploaded the `.cer` file to Azure, you can connect to the Azure management endpoint by passing the subscription id and the location of the certificate in your **Personal** certificate store to **ServiceManagementService** (again, replace *AzureCertificate* with the name of your certificate):

	from azure import *
	from azure.servicemanagement import *

	subscription_id = '<your_subscription_id>'
	certificate_path = 'CURRENT_USER\\my\\AzureCertificate'

	sms = ServiceManagementService(subscription_id, certificate_path)

In the example above, `sms` is a **ServiceManagementService** object. The **ServiceManagementService** class is the primary class used to manage Azure services.

## <a name="ListAvailableLocations"> </a>How to: List available locations

To list the locations that are available for hosting services, use the **list\_locations** method:

	from azure import *
	from azure.servicemanagement import *

	sms = ServiceManagementService(subscription_id, certificate_path)

	result = sms.list_locations()
	for location in result:
		print(location.name)

When you create a cloud service or storage service you will need to provide a valid location. The **list\_locations** method will always return an up-to-date list of the currently available locations. As of this writing, the available locations are:

- West Europe
- North Europe
- Southeast Asia
- East Asia
- Central US
- North Central US
- South Central US
- West US
- East US
- Japan East
- Japan West
- Brazil South
- Australia East
- Australia Southeast

## <a name="CreateCloudService"> </a>How to: Create a cloud service

When you create an application and run it in Azure, the code and configuration together are called an Azure [cloud service] (known as a *hosted service* in earlier Azure releases). The **create\_hosted\_service** method allows you to create a new hosted service by providing a hosted service name (which must be unique in Azure), a label (automatically encoded to base64), a description and a location.

	from azure import *
	from azure.servicemanagement import *

	sms = ServiceManagementService(subscription_id, certificate_path)

	name = 'myhostedservice'
	label = 'myhostedservice'
	desc = 'my hosted service'
	location = 'West US'

	sms.create_hosted_service(name, label, desc, location)

You can list all the hosted services for your subscription with the **list\_hosted\_services** method:

	result = sms.list_hosted_services()

	for hosted_service in result:
		print('Service name: ' + hosted_service.service_name)
		print('Management URL: ' + hosted_service.url)
		print('Location: ' + hosted_service.hosted_service_properties.location)
		print('')

If you want to get information about a particular hosted service, you can do so by passing the hosted service name to the **get\_hosted\_service\_properties** method:

	hosted_service = sms.get_hosted_service_properties('myhostedservice')

	print('Service name: ' + hosted_service.service_name)
	print('Management URL: ' + hosted_service.url)
	print('Location: ' + hosted_service.hosted_service_properties.location)

After you have created a cloud service, you can deploy your code to the service with the **create\_deployment** method.

## <a name="DeleteCloudService"> </a>How to: Delete a cloud service

You can delete a cloud service by passing the service name to the **delete\_hosted\_service** method:

	sms.delete_hosted_service('myhostedservice')

Note that before you can delete a service, all deployments for the the service must first be deleted. (See [How to: Delete a deployment](#DeleteDeployment) for details.)

## <a name="DeleteDeployment"> </a>How to: Delete a deployment

To delete a deployment, use the **delete\_deployment** method. The following example shows how to delete a deployment named `v1`.

	from azure import *
	from azure.servicemanagement import *

	sms = ServiceManagementService(subscription_id, certificate_path)

	sms.delete_deployment('myhostedservice', 'v1')

## <a name="CreateStorageService"> </a>How to: Create a storage service

A [storage service] gives you access to Azure [Blobs][azure-blobs], [Tables][azure-tables], and [Queues][azure-queues]. To create a storage service, you need a name for the service (between 3 and 24 lowercase characters and unique within Azure), a description, a label (up to 100 characters, automatically encoded to base64), and a location. The following example shows how to create a storage service by specifying a location.

	from azure import *
	from azure.servicemanagement import *

	sms = ServiceManagementService(subscription_id, certificate_path)

	name = 'mystorageaccount'
	label = 'mystorageaccount'
	location = 'West US'
	desc = 'My storage account description.'

	result = sms.create_storage_account(name, desc, label, location=location)

	operation_result = sms.get_operation_status(result.request_id)
	print('Operation status: ' + operation_result.status)

Note in the example above that the status of the **create\_storage\_account** operation can be retrieved by passing the result returned by **create\_storage\_account** to the **get\_operation\_status** method.  

You can list your storage accounts and their properties with the **list\_storage\_accounts** method:

	from azure import *
	from azure.servicemanagement import *

	sms = ServiceManagementService(subscription_id, certificate_path)

	result = sms.list_storage_accounts()
	for account in result:
		print('Service name: ' + account.service_name)
		print('Location: ' + account.storage_service_properties.location)
		print('')

## <a name="DeleteStorageService"> </a>How to: Delete a storage service

You can delete a storage service by passing the storage service name to the **delete\_storage\_account** method. Deleting a storage service will delete all data stored in the service (blobs, tables and queues).

	from azure import *
	from azure.servicemanagement import *

	sms = ServiceManagementService(subscription_id, certificate_path)

	sms.delete_storage_account('mystorageaccount')

## <a name="ListOperatingSystems"> </a>How to: List available operating systems

To list the operating systems that are available for hosting services, use the **list\_operating\_systems** method:

	from azure import *
	from azure.servicemanagement import *

	sms = ServiceManagementService(subscription_id, certificate_path)

	result = sms.list_operating_systems()

	for os in result:
		print('OS: ' + os.label)
		print('Family: ' + os.family_label)
		print('Active: ' + str(os.is_active))

Alternatively, you can use the **list\_operating\_system\_families** method, which groups the operating systems by family:

	result = sms.list_operating_system_families()

	for family in result:
		print('Family: ' + family.label)
		for os in family.operating_systems:
			if os.is_active:
				print('OS: ' + os.label)
				print('Version: ' + os.version)
		print('')

## <a name="CreateVMImage"> </a>How to: Create an operating system image

To add an operating system image to the image repository, use the **add\_os\_image** method:

	from azure import *
	from azure.servicemanagement import *

	sms = ServiceManagementService(subscription_id, certificate_path)

	name = 'mycentos'
	label = 'mycentos'
	os = 'Linux' # Linux or Windows
	media_link = 'url_to_storage_blob_for_source_image_vhd'

	result = sms.add_os_image(label, media_link, name, os)

	operation_result = sms.get_operation_status(result.request_id)
	print('Operation status: ' + operation_result.status)

To list the operating system images that are available, use the **list\_os\_images** method. This includes all platform images and user images:

	result = sms.list_os_images()

	for image in result:
		print('Name: ' + image.name)
		print('Label: ' + image.label)
		print('OS: ' + image.os)
		print('Category: ' + image.category)
		print('Description: ' + image.description)
		print('Location: ' + image.location)
		print('Media link: ' + image.media_link)
		print('')

## <a name="DeleteVMImage"> </a>How to: Delete an operating system image

To delete a user image, use the **delete\_os\_image** method:

	from azure import *
	from azure.servicemanagement import *

	sms = ServiceManagementService(subscription_id, certificate_path)

	result = sms.delete_os_image('mycentos')

	operation_result = sms.get_operation_status(result.request_id)
	print('Operation status: ' + operation_result.status)

## <a name="CreateVM"> </a>How to: Create a virtual machine

To create a virtual machine, you first need to create a [cloud service](#CreateCloudService).  Then create the virtual machine deployment using the **create\_virtual\_machine\_deployment** method:

	from azure import *
	from azure.servicemanagement import *

	sms = ServiceManagementService(subscription_id, certificate_path)

	name = 'myvm'
	location = 'West US'

	#Set the location
	sms.create_hosted_service(service_name=name,
		label=name,
		location=location)

	# Name of an os image as returned by list_os_images
	image_name = 'OpenLogic__OpenLogic-CentOS-62-20120531-en-us-30GB.vhd'

	# Destination storage account container/blob where the VM disk
	# will be created
	media_link = 'url_to_target_storage_blob_for_vm_hd'

	# Linux VM configuration, you can use WindowsConfigurationSet
	# for a Windows VM instead
	linux_config = LinuxConfigurationSet('myhostname', 'myuser', 'mypassword', True)

	os_hd = OSVirtualHardDisk(image_name, media_link)

	sms.create_virtual_machine_deployment(service_name=name,
		deployment_name=name,
		deployment_slot='production',
		label=name,
		role_name=name,
		system_config=linux_config,
		os_virtual_hard_disk=os_hd,
		role_size='Small')

## <a name="DeleteVM"> </a>How to: Delete a virtual machine

To delete a virtual machine, you first delete the deployment using the **delete\_deployment** method:

	from azure import *
	from azure.servicemanagement import *

	sms = ServiceManagementService(subscription_id, certificate_path)

	sms.delete_deployment(service_name='myvm',
		deployment_name='myvm')

The cloud service can then be deleted using the **delete\_hosted\_service** method:

	sms.delete_hosted_service(service_name='myvm')

##How To: Create a Virtual Machine from a Captured Virtual Machine Image

To capture a VM image, you first call the **capture\_vm\_image** method:

	from azure import *
	from azure.servicemanagement import *

	sms = ServiceManagementService(subscription_id, certificate_path)

	# replace the below three parameters with actual values
	hosted_service_name = 'hs1'
	deployment_name = 'dep1'
	vm_name = 'vm1'

	image_name = vm_name + 'image'
	image = CaptureRoleAsVMImage	('Specialized',
		image_name,
		image_name + 'label',
		image_name + 'description',
		'english',
		'mygroup')

	result = sms.capture_vm_image(
			hosted_service_name,
			deployment_name,
			vm_name,
			image
		)

Next, to make sure that you have successfully captured the image, use the **list\_vm\_images** api and make sure your image is displayed in the results:

	images = sms.list_vm_images()

To finally create the virtual machine using the captured image, use the **create\_virtual\_machine\_deployment** method as before, but this time pass in the vm_image_name instead

	from azure import *
	from azure.servicemanagement import *

	sms = ServiceManagementService(subscription_id, certificate_path)

	name = 'myvm'
	location = 'West US'

	#Set the location
	sms.create_hosted_service(service_name=name,
		label=name,
		location=location)

	sms.create_virtual_machine_deployment(service_name=name,
		deployment_name=name,
		deployment_slot='production',
		label=name,
		role_name=name,
		system_config=linux_config,
		os_virtual_hard_disk=None,
		role_size='Small',
		vm_image_name = image_name)

To learn more about how to capture a Linux Virtual Machine, see [How to Capture a Linux Virtual Machine.](virtual-machines-linux-capture-image.md)

To learn more about how to capture a Windows Virtual Machine, see [How to Capture a Windows Virtual Machine.](virtual-machines-capture-image-windows-server.md)

## <a name="What's Next"> </a>Next Steps

Now that you've learned the basics of service management, you can access the [Complete API reference documentation for the Azure Python SDK](http://azure-sdk-for-python.readthedocs.org/en/documentation/index.html) and perform complex tasks easily to manage your python application.



[What is Service Management]: #WhatIs
[Concepts]: #Concepts
[How to: Connect to service management]: #Connect
[How to: List available locations]: #ListAvailableLocations
[How to: Create a cloud service]: #CreateCloudService
[How to: Delete a cloud service]: #DeleteCloudService
[How to: Create a deployment]: #CreateDeployment
[How to: Update a deployment]: #UpdateDeployment
[How to: Move deployments between staging and production]: #MoveDeployments
[How to: Delete a deployment]: #DeleteDeployment
[How to: Create a storage service]: #CreateStorageService
[How to: Delete a storage service]: #DeleteStorageService
[How to: List available operating systems]: #ListOperatingSystems
[How to: Create an operating system image]: #CreateVMImage
[How to: Delete an operating system image]: #DeleteVMImage
[How to: Create a virtual machine]: #CreateVM
[How to: Delete a virtual machine]: #DeleteVM
[Next Steps]: #NextSteps
[management-portal]: https://manage.windowsazure.com/
[svc-mgmt-rest-api]: http://msdn.microsoft.com/library/windowsazure/ee460799.aspx


[download-SDK-Python]: https://www.windowsazure.com/develop/python/common-tasks/install-python/
[cloud service]:http://windowsazure.com/documentation/articles/cloud-services-what-is
[service package]: http://msdn.microsoft.com/library/windowsazure/jj155995.aspx
[Azure PowerShell cmdlets]: https://www.windowsazure.com/develop/php/how-to-guides/powershell-cmdlets/
[cspack commandline tool]: http://msdn.microsoft.com/library/windowsazure/gg432988.aspx
[Deploying an Azure Service]: http://msdn.microsoft.com/library/windowsazure/gg433027.aspx
[storage service]: https://www.windowsazure.com/manage/services/storage/what-is-a-storage-account/
[azure-blobs]: https://www.windowsazure.com/develop/python/how-to-guides/blob-service/
[azure-tables]: https://www.windowsazure.com/develop/python/how-to-guides/table-service/
[azure-queues]: https://www.windowsazure.com/develop/python/how-to-guides/queue-service/
[Azure Service Configuration Schema (.cscfg)]: http://msdn.microsoft.com/library/windowsazure/ee758710.aspx
[Cloud Services]: http://msdn.microsoft.com/library/windowsazure/jj155995.aspx
[Virtual Machines]: http://msdn.microsoft.com/library/windowsazure/jj156003.aspx
