<properties
	pageTitle="Service-side Authorization of Users in Mobile Services with .NET Backend | Mobile Dev Center"
	description="Learn how to authorize users in .NET backend of Azure Mobile Services"
	services="mobile-services"
	documentationCenter="windows"
	authors="krisragh"
	manager="dwrede"
	editor=""/>

<tags
	ms.service="mobile-services"
	ms.workload="mobile"
	ms.tgt_pltfrm="mobile-multiple"
	ms.topic="article"
	ms.devlang="dotnet"
	ms.date="05/10/2015"
	ms.author="krisragh"/>

# Service-side Authorization of Users in Mobile Services

> [AZURE.SELECTOR-LIST (Platform | Backend)]
- [(Any | .NET)](mobile-services-dotnet-backend-service-side-authorization.md)
- [(Any | Javascript)](mobile-services-javascript-backend-service-side-authorization.md)

This topic shows you how to use server-side logic to authorize users.  In this tutorial, you modify table controllers, filter queries based on user IDs, and give users access to only their own data. Filtering a user's query results by the user ID is the most basic form of authorization. Depending on your specific scenario, you might also want to create Users or Roles tables to track more detailed user authorization information, such as which endpoints a given user is permitted to access.

This tutorial is based on the Mobile Services Quick Start and builds on the [Add Authentication to Existing Mobile Services App] tutorial. Please complete [Add Authentication to Existing Mobile Services App] first.

## <a name="register-scripts"></a>Modify data access methods

1. In Visual Studio, open your mobile project, expand the DataObjects folder, and open **TodoItem.cs**. The **TodoItem** class defines the data object, and you need to add a **UserId** property to use for filtering. Add the following new UserId property to the **TodoItem** class:

		public string UserId { get; set; }

	>[AZURE.NOTE] To make this data model change and maintain existing data in the database, you must use [Code First Migrations](mobile-services-dotnet-backend-how-to-use-code-first-migrations.md).

2. In Visual Studio, expand the Controllers folder and open **TodoItemController.cs**. Locate the **PostTodoItem** method and add the following code at the beginning of the method. This code adds the user ID of the authenticated user to the item, before it is inserted into the TodoItem table.

			// Get the logged in user
			var currentUser = User as ServiceUser;

			// Set the user ID on the item
			item.UserId = currentUser.Id;

3. Locate the **GetAllTodoItems** method and replace the existing **return** statement with the following line of code. This query filters the returned TodoItem objects so that each user only receives the items that they inserted.

				// Get the logged in user
				var currentUser = User as ServiceUser;

				return Query().Where(todo => todo.UserId == currentUser.Id);

4. Republish the mobile service project to Azure.


## <a name="test-app"></a>Test the app

1. Notice that when you now run your client-side app, although there are items already in the database from previous tutorials, no items are returned. This happens because previous items were inserted without the user ID column and now have null values.

2. If you have additional login accounts, verify that users can only see their own data by closing and deleting the app and running it again. When the login credentials dialog is displayed, enter a different login and verify that the items entered under the previous login are not displayed.



<!-- Anchors. -->
[Register server scripts]: #register-scripts
[Next Steps]:#next-steps

<!-- Images. -->

[3]: ./media/mobile-services-dotnet-backend-ios-authorize-users-in-scripts/mobile-quickstart-startup-ios.png

<!-- URLs. -->
[Add Authentication to Existing Mobile Services App]: mobile-services-dotnet-backend-ios-get-started-users.md
