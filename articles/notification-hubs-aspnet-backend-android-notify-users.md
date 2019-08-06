<properties 
	pageTitle="Azure Notification Hubs Notify Users" 
	description="Learn how to send push notifications to users in Azure. Code samples written in Java for Android" 
	documentationCenter="android" 
	services="notification-hubs" 
	authors="wesmc7777" 
	manager="dwrede" 
	editor=""/>

<tags 
	ms.service="notification-hubs" 
	ms.workload="mobile" 
	ms.tgt_pltfrm="mobile-android" 
	ms.devlang="java" 
	ms.topic="article" 
	ms.date="04/14/2015" 
	ms.author="wesmc"/>

#Azure Notification Hubs Notify Users

<div class="dev-center-tutorial-selector sublanding"> 
    	<a href="/documentation/articles/notification-hubs-aspnet-backend-windows-dotnet-notify-users/" title="Windows Universal">Windows Universal</a><a href="/documentation/articles/notification-hubs-aspnet-backend-ios-notify-users/" title="iOS">iOS</a>
		<a href="/documentation/articles/notification-hubs-aspnet-backend-android-notify-users/" title="Android" class="current">Android</a>
</div>

Push notification support in Azure enables you to access an easy-to-use, multiplatform, and scaled-out push infrastructure, which greatly simplifies the implementation of push notifications for both consumer and enterprise applications for mobile platforms. This tutorial shows you how to use Azure Notification Hubs to send push notifications to a specific app user on a specific device. An ASP.NET WebAPI backend is used to authenticate clients and to generate notifications, as shown in the guidance topic [Registering from your app backend](http://msdn.microsoft.com/library/dn743807.aspx). This tutorial builds on the notification hub that you created in the [Getting Started with Notification Hubs (Android)](notification-hubs-android-get-started.md) tutorial.

> [AZURE.NOTE] This tutorial assumes that you have created and configured your notification hub as described in [Getting Started with Notification Hubs (Android)](notification-hubs-android-get-started.md). 
> If you are using Mobile Services as your backend service, see the [Mobile Services version](mobile-services-javascript-backend-android-push-notifications-app-users.md) of this tutorial.

[AZURE.INCLUDE [notification-hubs-aspnet-backend-notifyusers](../includes/notification-hubs-aspnet-backend-notifyusers.md)]

## Create the Android Project

The next step is to create the Android application.

1. Follow the [Getting Started with Notification Hubs (Android)](notification-hubs-android-get-started.md) tutorial to create and configure your app to receive push notifications from GCM.

2. Open your **res/layout/activity_main.xml** file, replace the with the following content definitions. 
 
    This adds new EditText controls for logging in as a user. Also a field is added for a username tag that will be part of notifications you send:
			
		<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
            xmlns:tools="http://schemas.android.com/tools" android:layout_width="match_parent"
            android:layout_height="match_parent" android:paddingLeft="@dimen/activity_horizontal_margin"
            android:paddingRight="@dimen/activity_horizontal_margin"
            android:paddingTop="@dimen/activity_vertical_margin"
            android:paddingBottom="@dimen/activity_vertical_margin" tools:context=".MainActivity">
        
        <EditText
            android:id="@+id/usernameText"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:ems="10"
            android:hint="@string/usernameHint"
            android:layout_above="@+id/passwordText"
            android:layout_alignParentEnd="true" />
        <EditText
            android:id="@+id/passwordText"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:ems="10"
            android:hint="@string/passwordHint"
            android:inputType="textPassword"
            android:layout_above="@+id/buttonLogin"
            android:layout_alignParentEnd="true" />
        <Button
            android:id="@+id/buttonLogin"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="@string/loginButton"
            android:onClick="login"
            android:layout_above="@+id/toggleButtonGCM"
            android:layout_centerHorizontal="true"
            android:layout_marginBottom="24dp" />
        <ToggleButton
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:textOn="WNS on"
            android:textOff="WNS off"
            android:id="@+id/toggleButtonWNS"
            android:layout_toLeftOf="@id/toggleButtonGCM"
            android:layout_centerVertical="true" />
        <ToggleButton
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:textOn="GCM on"
            android:textOff="GCM off"
            android:id="@+id/toggleButtonGCM"
            android:checked="true"
            android:layout_centerHorizontal="true"
            android:layout_centerVertical="true" />
        <ToggleButton
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:textOn="APNS on"
            android:textOff="APNS off"
            android:id="@+id/toggleButtonAPNS"
            android:layout_toRightOf="@id/toggleButtonGCM"
            android:layout_centerVertical="true" />
        <EditText
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:id="@+id/editTextNotificationMessageTag"
            android:layout_below="@id/toggleButtonGCM"
            android:layout_centerHorizontal="true"
            android:hint="@string/notification_message_tag_hint" />
        <EditText
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:id="@+id/editTextNotificationMessage"
            android:layout_below="@+id/editTextNotificationMessageTag"
            android:layout_centerHorizontal="true"
            android:hint="@string/notification_message_hint" />
        <Button
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="@string/send_button"
            android:id="@+id/sendbutton"
            android:onClick="sendNotificationButtonOnClick"
            android:layout_below="@+id/editTextNotificationMessage"
            android:layout_centerHorizontal="true" />
        </RelativeLayout>



3. Open your **res/values/strings.xml** file and replace the `send_button` definition with the following lines that redefine the string for the `send_button` and add strings for the other controls:

        <string name="usernameHint">Username</string>
        <string name="passwordHint">Password</string>
        <string name="loginButton">1. Log in</string>
        <string name="send_button">2. Send Notification</string>
        <string name="notification_message_tag_hint">
			Recipient username tag
		</string>

	Your main_activity.xml graphical layout should now look like this:

	![][A1]

4. Create a new class named **RegisterClient** in the same package as your `MainActivity` class. Use the code below for the new class file. 
 
		import java.io.IOException;
        import java.io.UnsupportedEncodingException;
        import java.util.Set;
        
        import org.apache.http.HttpResponse;
        import org.apache.http.HttpStatus;
        import org.apache.http.client.ClientProtocolException;
        import org.apache.http.client.HttpClient;
        import org.apache.http.client.methods.HttpPost;
        import org.apache.http.client.methods.HttpPut;
        import org.apache.http.client.methods.HttpUriRequest;
        import org.apache.http.entity.StringEntity;
        import org.apache.http.impl.client.DefaultHttpClient;
        import org.apache.http.util.EntityUtils;
        import org.json.JSONArray;
        import org.json.JSONException;
        import org.json.JSONObject;
        
        import android.content.Context;
        import android.content.SharedPreferences;
        import android.util.Log;
        
        public class RegisterClient {
            private static final String PREFS_NAME = "ANHSettings";
            private static final String REGID_SETTING_NAME = "ANHRegistrationId";
            private String Backend_Endpoint;
            SharedPreferences settings;
            protected HttpClient httpClient;
            private String authorizationHeader;
        
            public RegisterClient(Context context, String backendEnpoint) {
                super();
                this.settings = context.getSharedPreferences(PREFS_NAME, 0);
                httpClient =  new DefaultHttpClient();
                Backend_Endpoint = backendEnpoint + "/api/register";
            }
        
            public String getAuthorizationHeader() {
                return authorizationHeader;
            }
        
            public void setAuthorizationHeader(String authorizationHeader) {
                this.authorizationHeader = authorizationHeader;
            }
        
            public void register(String handle, Set<String> tags) throws ClientProtocolException, IOException, JSONException {
                String registrationId = retrieveRegistrationIdOrRequestNewOne(handle);
        
                JSONObject deviceInfo = new JSONObject();
                deviceInfo.put("Platform", "gcm");
                deviceInfo.put("Handle", handle);
                deviceInfo.put("Tags", new JSONArray(tags));
        
                int statusCode = upsertRegistration(registrationId, deviceInfo);
        
                if (statusCode == HttpStatus.SC_OK) {
                    return;
                } else if (statusCode == HttpStatus.SC_GONE){
                    settings.edit().remove(REGID_SETTING_NAME).commit();
                    registrationId = retrieveRegistrationIdOrRequestNewOne(handle);
                    statusCode = upsertRegistration(registrationId, deviceInfo);
                    if (statusCode != HttpStatus.SC_OK) {
                        Log.e("RegisterClient", "Error upserting registration: " + statusCode);
                        throw new RuntimeException("Error upserting registration");
                    }
                } else {
                    Log.e("RegisterClient", "Error upserting registration: " + statusCode);
                    throw new RuntimeException("Error upserting registration");
                }
            }
        
            private int upsertRegistration(String registrationId, JSONObject deviceInfo)
                    throws UnsupportedEncodingException, IOException,
                    ClientProtocolException {
                HttpPut request = new HttpPut(Backend_Endpoint+"/"+registrationId);
                request.setEntity(new StringEntity(deviceInfo.toString()));
                request.addHeader("Authorization", "Basic "+authorizationHeader);
                request.addHeader("Content-Type", "application/json");
                HttpResponse response = httpClient.execute(request);
                int statusCode = response.getStatusLine().getStatusCode();
                return statusCode;
            }
        
            private String retrieveRegistrationIdOrRequestNewOne(String handle) throws ClientProtocolException, IOException {
                if (settings.contains(REGID_SETTING_NAME))
                    return settings.getString(REGID_SETTING_NAME, null);
        
                HttpUriRequest request = new HttpPost(Backend_Endpoint+"?handle="+handle);
                request.addHeader("Authorization", "Basic "+authorizationHeader);
                HttpResponse response = httpClient.execute(request);
                if (response.getStatusLine().getStatusCode() != HttpStatus.SC_OK) {
                    Log.e("RegisterClient", "Error creating registrationId: " + response.getStatusLine().getStatusCode());
                    throw new RuntimeException("Error creating Notification Hubs registrationId");
                }
                String registrationId = EntityUtils.toString(response.getEntity());
                registrationId = registrationId.substring(1, registrationId.length()-1);
        
                settings.edit().putString(REGID_SETTING_NAME, registrationId).commit();
        
                return registrationId;
            }
        }

	This component implements the REST calls required to contact the app backend, in order to register for push notifications. It also locally stores the *registrationIds* created by the Notification Hub as detailed in [Registering from your app backend](http://msdn.microsoft.com/library/dn743807.aspx). Note that it uses an authorization token stored in local storage when you click the **Log in** button.

5. In your `MainActivity` class remove or comment out your private field for `NotificationHub`, and add a field for the `RegisterClient` class and a string for your ASP.NET backend's endpoint. Be sure to replace `<Enter Your Backend Endpoint>` with the your actual backend endpoint obtained previously. For example, `http://mybackend.azurewebsites.net`.


		//private NotificationHub hub;
		private RegisterClient registerClient;
	    private static final String BACKEND_ENDPOINT = "<Enter Your Backend Endpoint>";

 
6. In your `MainActivity` class, in the `onCreate` method, remove or comment out the initialization of the `hub` field and the call to the `registerWithNotificationHubs` method. Then add code to initialize an instance of the `RegisterClient` class. The method should contain the following lines:

        @Override
        protected void onCreate(Bundle savedInstanceState) {
            super.onCreate(savedInstanceState);
    
            MyHandler.mainActivity = this;
            NotificationsManager.handleNotifications(this, SENDER_ID, MyHandler.class);
            gcm = GoogleCloudMessaging.getInstance(this);
    
            //hub = new NotificationHub(HubName, HubListenConnectionString, this);
            //registerWithNotificationHubs();
    
	        registerClient = new RegisterClient(this, BACKEND_ENDPOINT);

            setContentView(R.layout.activity_main);
        }

7. In your `MainActivity` class, delete or comment out the entire `registerWithNotificationHubs` method. It will not be used in this tutorial. 

8. Add the following `import` statements to your **MainActivity.java** file.

		import android.widget.Button;
		import java.io.UnsupportedEncodingException;
		import android.content.Context;
		import java.util.HashSet;
		import android.widget.Toast;
		import org.apache.http.client.ClientProtocolException;
		import java.io.IOException;
		import org.apache.http.HttpStatus;


9. Then, add the following methods to handle the **Log in** button click event and sending push notifications.

	    @Override
	    protected void onStart() {
	    	super.onStart();
        	Button sendPush = (Button) findViewById(R.id.sendbutton);
	        sendPush.setEnabled(false);
	    }
	
		public void login(View view) throws UnsupportedEncodingException {
	    	this.registerClient.setAuthorizationHeader(getAuthorizationHeader());
	    	
	    	final Context context = this;
	    	new AsyncTask<Object, Object, Object>() {
				@Override
				protected Object doInBackground(Object... params) {
					try {
						String regid = gcm.register(SENDER_ID);
				        registerClient.register(regid, new HashSet<String>());
					} catch (Exception e) {
	                    DialogNotify("MainActivity - Failed to register", e.getMessage());
						return e;
					}
					return null;
				}
	
				protected void onPostExecute(Object result) {
                	Button sendPush = (Button) findViewById(R.id.sendbutton);
			        sendPush.setEnabled(true);
					Toast.makeText(context, "Logged in and registered.",
							Toast.LENGTH_LONG).show();
				}
			}.execute(null, null, null);
	    }
	    
		private String getAuthorizationHeader() throws UnsupportedEncodingException {
			EditText username = (EditText) findViewById(R.id.usernameText);
	    	EditText password = (EditText) findViewById(R.id.passwordText);
	    	String basicAuthHeader = username.getText().toString()+":"+password.getText().toString();
	    	basicAuthHeader = Base64.encodeToString(basicAuthHeader.getBytes("UTF-8"), Base64.NO_WRAP);
	    	return basicAuthHeader;
		}

        /**
         * This method calls the ASP.NET WebAPI backend to send the notification message
         * to the platform notification service based on the pns parameter.
         *
         * @param pns     The platform notification service to send the notification message to. Must
         *                be one of the following ("wns", "gcm", "apns").
         * @param userTag The tag for the user who will receive the notification message. This string
         *                must not contain spaces or special characters.
         * @param message The notification message string. This string must include the double quotes
         *                to be used as JSON content.
         */
        public void sendPush(final String pns, final String userTag, final String message)
                throws ClientProtocolException, IOException {
            new AsyncTask<Object, Object, Object>() {
                @Override
                protected Object doInBackground(Object... params) {
                    try {
    
                        String uri = BACKEND_ENDPOINT + "/api/notifications";
                        uri += "?pns=" + pns;
                        uri += "&to_tag=" + userTag;
    
                        HttpPost request = new HttpPost(uri);
                        request.addHeader("Authorization", "Basic "+ getAuthorizationHeader());
                        request.setEntity(new StringEntity(message));
                        request.addHeader("Content-Type", "application/json");
    
                        HttpResponse response = new DefaultHttpClient().execute(request);
    
                        if (response.getStatusLine().getStatusCode() != HttpStatus.SC_OK) {
                            DialogNotify("MainActivity - Error sending " + pns + " notification", 
								response.getStatusLine().toString());
                            throw new RuntimeException("Error sending notification");
                        }
                    } catch (Exception e) {
                        DialogNotify("MainActivity - Failed to send " + pns + " notification ", e.getMessage());
                        return e;
                    }
    
                    return null;
                }
            }.execute(null, null, null);
        }


	The `login` handler for the **Log in** button generates a basic authentication token using on the input username and password (note that this represents any token your authentication scheme uses), then it uses `RegisterClient` to call the backend for registration. 

	The `sendPush` method calls the backend to trigger a secure notification to the user based on the user tag. The platform notification service that `sendPush` targets depends on the `pns` string passed in. 

10. In your `MainActivity` class, update the `sendNotificationButtonOnClick` method to call the `sendPush` method with the user's selected platform notification services as follows.

        /**
         * Send Notification button click handler. This method sends the push notification
         * message to each platform selected.
         *
         * @param v The view
         */
        public void sendNotificationButtonOnClick(View v)
                throws ClientProtocolException, IOException {
    
            String nhMessageTag = ((EditText) findViewById(R.id.editTextNotificationMessageTag))
                    .getText().toString();
            String nhMessage = ((EditText) findViewById(R.id.editTextNotificationMessage))
                    .getText().toString();
    
            // JSON String
            nhMessage = "\"" + nhMessage + "\"";
    
            if (((ToggleButton)findViewById(R.id.toggleButtonWNS)).isChecked())
            {
                sendPush("wns", nhMessageTag, nhMessage);
            }
            if (((ToggleButton)findViewById(R.id.toggleButtonGCM)).isChecked())
            {
                sendPush("gcm", nhMessageTag, nhMessage);
            }
            if (((ToggleButton)findViewById(R.id.toggleButtonAPNS)).isChecked())
            {
                sendPush("apns", nhMessageTag, nhMessage);
            }
        }



## Run the Application


1. Run the application on a device or an emulator using Android Studio.

2. In the Android app, enter a username and password. They must both be the same string value and they must not contain spaces or special characters. 

3. In the Android app, click **Log in**. Wait for a toast message that states **Logged in and registered**. This will enable the **Send Notification** button. 

	![][A2]

4. Click the toggle buttons to enable all platforms where you have ran the app and registered a user.
5. Enter the user's name that will receive the notification message. That user must be registered for notifications on the target devices.

6. Enter a message for the user to receive as a push notification message.
7. Click **Send Notification**.  Each device that has a registration with the matching username tag will receive the push notification.


[A1]: ./media/notification-hubs-aspnet-backend-android-notify-users/android-notify-users.png
[A2]: ./media/notification-hubs-aspnet-backend-android-notify-users/android-notify-users-enter-password.png

