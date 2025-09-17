# Pluggable's Fanmeter SDK for Android

The Fanmeter SDK is the plugin responsible for enabling Fanmeter in any mobile application. It is a simple plug-&-play SDK that makes available a set of methods that can be used to allow users to participate in activations such as the **FAN OF THE MATCH**. The SDK is available in multiple formats, i.e., as a native SDK or as a framework plugin, such as for Flutter or React Native, among others.

## Plug-&-Play vs Build your own UI

There are two possible types of integration. One is plug-&-play and uses a default native UI that is launched to your users for them to participate in Fanmeter events - it is also able to launch and deliver push notifications as soon as the event starts and finishes, if you configure it. If, on the other hand, you wish to create your own Fanmeter UI, you will follow a "build your own UI" approach.

  1. **Plug-&-Play UI**: you want everything automated (including a default Fanmeter view). You should also integrate with FCM to handle received notifications that start the event (in summary, you'll need to use the *execute*, *launchFanmeterNotification* and the *launchFanmeterView* methods);

  2. **Build your own UI**: you want to handle the conditions yourself and develop your own UI (just call the *startService*, *stopService*, and *isServiceRunning* methods).

## Pre-Conditions

### Meta-Data

On Android, push notification permission is required to inform the user that a foreground service is running. Additionally, Location permission is necessary for fans to participate in geo-restricted events. Make sure to request these permissions.

Note that, for Android only, Fanmeter is required to use a **Data Sync foreground service** to communicate non-intrusive sensor data with Pluggable's server. Since API 34 of Android, when publishing an app that uses such services, you are required to fill a **Policy Declaration** where you must justify the need for such services. In Fanmeter's case, when filling such policy, you will be asked the question "*What tasks require your app to use the FOREGROUND_SERVICE_DATA_SYNC permission?*". There, you should:

1. Select the option **OTHER** in **Other tasks**;

2. Provide the following video link: 
    ```
    https://youtu.be/w4d7Pgksok0
    ```

3. Provide the following description: 
    ```
    Fanmeter is an Android SDK, incorporated in this app, that uses a Data Sync foreground service to communicate non-intrusive sensor data with Pluggable servers, which are used to quantify the engagement of the user in real-time. The foreground service must start as soon as the user opts to participate in the event (so that it can collect and communicate data) and must keep running until the user himself decides to terminate his/her participation.
    ```

### Push Notifications

If you want a **fully plug-&-play experience** and wish to receive push notifications (if not receiving yet) from Fanmeter in your app, you should integrate Firebase Cloud Messaging. A complete, step-by-step tutorial is available in [Firebase's wiki](https://firebase.google.com/docs/cloud-messaging).

## Set up the Fanmeter SDK in your Android App

Few steps are required to be ready to use this SDK.

### Import the SDK

1. In your project's Gradle file, declare the dependencies for the Fanmeter SDK:

    i. Check the latest version of the SDK and, in the dependencies tag, import the SDK:
    ```gradle
    implementation 'io.github.pluggableai:fanmeter:4.3.0'
    ```

    ii. The SDK may require additional default libraries. Hence, in the same tag and build.gradle file, you may need to add:
    ```gradle
    implementation 'com.android.volley:volley:1.2.1'
    implementation 'androidx.work:work-runtime-ktx:2.8.0'
    implementation 'androidx.constraintlayout:constraintlayout:2.1.4'
    implementation 'androidx.navigation:navigation-fragment-ktx:2.5.3'
    implementation 'androidx.databinding:viewbinding:7.1.2'
    implementation 'androidx.swiperefreshlayout:swiperefreshlayout:1.1.0'
    implementation 'com.google.android.gms:play-services-location:21.3.0'
    ```

2. Sync your app to ensure that all dependencies have the necessary versions.

### Push Notifications

If your app is not yet receiving push notifications and you aim to, you must, in your app's manifest, inside the application tag, add a service that extends `FirebaseMessagingService`. For better engagement with the Plug-&-Play UI, you should handle push notifications. This is required if you want to do any message handling beyond receiving notifications on apps in the background. To receive notifications in foregrounded apps, to receive data payload, to send upstream messages, and so on, you must extend this service.

```xml
<service
   android:name=".notifications.MyFirebaseMessagingService"
   android:foregroundServiceType="remoteMessaging"
   android:exported="false">
   <intent-filter>
       <action android:name="com.google.firebase.MESSAGING_EVENT" />
   </intent-filter>
</service>
```

Then, optionally, add metadata elements to set a default notification icon and colour. Android uses these values whenever incoming messages do not explicitly set these features.

```xml
<meta-data
   android:name="com.google.firebase.messaging.default_notification_icon"
   android:resource="@drawable/notification_icon" />
<meta-data
   android:name="com.google.firebase.messaging.default_notification_color"
   android:resource="@color/notification_color" />
<meta-data
   android:name="com.google.firebase.messaging.default_notification_channel_id"
   android:value="@string/default_notification_channel_id" />
```

For Android, to customize the used notification icon, just add the desired icon in the Android's drawable folder and name it ic_push_app_icon. Otherwise, a default icon, not related to your app, will be used.

### Methods initialize and getEventData

The `EntryPoint.initialize()` method is mandatory for both plug-&-play and build your own UI scenarios. It is responsible for initializing the SDK and must be invoked before using any other method (a possible approach is to invoke it in the app's main activity). It returns success or an error code.

```kotlin
private const val YOUR_COMPANY_NAME: String = "companyName"
private const val YOUR_COMPANY_KEY: String = "AAAA-BBBB-CCCC-DDDD"
private const val USER_ID: String = "externalUserId"
private const val TOKEN_ID: String = "externalTokenId"
private const val USER_EMAIL: String = "externalUserEmail"
private const val FCM_TOKEN: String = "fcm_token"
private const val TICKET_NUMBER: String = "ticket_number"
private const val TICKET_STAND: String = "ticket_stand"
private const val REGULATION_URL: String = "https://link.to.the.regulation"
private const val FANMETER_LOG: Boolean = false

// When the Fanmeter SDK is initialized, it initializes all company and user data.
val res = EntryPoint.initialize(
    this,
    YOUR_COMPANY_NAME,
    YOUR_COMPANY_KEY,
    USER_ID,
    TOKEN_ID,
    USER_EMAIL,
    FCM_TOKEN,
    TICKET_NUMBER,
    TICKET_STAND,
    REGULATION_URL,
    FANMETER_LOG
)
```

Where:
  * *this*, is the application context;
  * *YOUR_COMPANY_NAME*, the name of your company in Fanmeter;
  * *YOUR_COMPANY_KEY*, your company's license key;
  * *USER_ID*, the user identifier in your db (can be the username, the uuid, ...);
  * *TOKEN_ID*, the individual smartphone identifier (allows for the same account in different devices);
  * *USER_EMAIL*, the user's email. Nullable;
  * *FCM_TOKEN*, the FCM token id of the user. Nullable;
  * *TICKET_NUMBER*, the user's ticket number - used for additional analytics. Nullable;
  * *TICKET_STAND*, the stand where the user is - used for additional analytics. Nullable;
  * *REGULATION_URL*, the URL to the regulation for your events. Nullable;
  * *FANMETER_LOG*, enables additional logging.

The `EntryPoint.getEventData()` method is used to obtain the full data of a particular Fanmeter event, in a dictionary, including the defined rewards and leaderboard classifications, if existing. If null, or not provided, this returns the closest event to date:

```kotlin
private const val EVENT_NAME: String = "Round 1 2025-2026"
private const val EVENT_ID: String = "-99"

// After initialized, you will be able to use the SDK methods.
EntryPoint.getEventData(
    applicationContext,
    EVENT_NAME
) { eventData ->
    val eventId = eventData.optLong("eventId", -99)
    Log.d("A_LOG_TAG", "EntryPoint.getEventData($EVENT_NAME): $eventData")
}

// OR.

// After initialized, you will be able to use the SDK methods.
EntryPoint.getEventData(
    applicationContext
) { eventData ->
    val eventId = eventData.optLong("eventId", -99)
    Log.d("A_LOG_TAG", "EntryPoint.getEventData(): $eventData")
}
```

### Listeners for User Participation

In both scenarios, you can subscribe to a listener that will notify you whenever there is a change in the user's participation status. For example, if the user leaves the venue of a geo-restricted event, the listener will alert you to this change. This can be particularly useful for updating the status of banners or buttons within your app.

Two methods are available: `EntryPoint.subscribeUserParticipationListener()` and `EntryPoint.unsubscribeUserParticipationListener()`. The first method subscribes the listener to a specific event id, and you can subscribe to multiple events. The second method unsubscribes the listener (if no event id is provided, it will unsubscribe all active listeners).

While the unsubscribeUserParticipationListener method always returns 1 (Success), as it forces the stop of any or all running listeners, the subscribeUserParticipationListener listener can return the following status codes:

  * 0: User is not participating; 
  * 1: Validating the user participation (validation may take up to 30 seconds);
  * 2: User is participating;
  * 3: User has GPS disabled; 
  * 4: User is out of venue; 
  * 5: Unknown GPS coordinates.

These methods can be invoked as shown below:

```kotlin
// Subscribing a listener.
EntryPoint.subscribeUserParticipationListener(
    applicationContext,
    eventId = EVENT_ID
) { state ->
    Log.d("A_LOG_TAG", "Event $eventId | EntryPoint.subscribeUserParticipationListener: $state")
}

// OR.

// Unsubscribing a listener. If no event id is set, it cancels all listeners.
EntryPoint.unsubscribeUserParticipationListener() {
    Log.d("A_LOG_TAG", "EntryPoint.unsubscribeUserParticipationListener: $it")
}
```

### Plug-&-Play UI

After initializing the SDK, you are now ready to start calling Fanmeter. In particular, if you want to automate the entire process, this library exposes three methods, that must be called as demonstrated below. In particular:

  * `execute`, launches the SDK Fanmeter default's view as soon as a Fanmeter notification is tapped by the user; 
  * `launchFanmeterNotification`, launches a local Fanmeter notification to the user, which is required by Android when the app is in the foreground; 
  * `launchFanmeterView`, launches the SDK Fanmeter default's view. It can be associated with a button or banner in your app, redirecting the user to the Fanmeter native view, allowing users without notification permissions to participate in the event. It will open the Fanmeter view with the event with the closest date to the current date.

The `execute` method is used for backgrounded processes and will open the default Fanmeter view to the user. It must be called in the app's Main Activity. An example is as follows:

```kotlin
// Loads the Fanmeter SDK.
import com.pluggableai.fanmeter.EntryPoint

class MainActivity: ComponentActivity() {

    //...

    override fun onCreate(savedInstanceState: Bundle?) {
        //...
        if(intent != null){
            launchFanmeter(intent)
        }
    }

    private fun launchFanmeter(intent: Intent) {
        Log.d(TAG, "Message data payload: ${intent.extras}")
        // Execute the Fanmeter SDK and launch the default Fanmeter view.
        EntryPoint.execute(
            this,
            intent.extras,
            null
        ){
            Log.d(TAG, "EntryPoint.execute: $it")
        }
    }
}
```

Where:
  * *this*, is the application context;
  * *extras*, the remote data received with the intent;
  * *notificationClassResponse*, the name of the class that is being instantiated when the user clicks the notification - example: "com.company.activities.SearchActivity" (null opens the app's default view).

The `launchFanmeterNotification` method launches a local notification when the app is in a foreground state. Since you have already added the service that extends `FirebaseMessagingService`, you are now required to implement it so that we can display notifications when the app is in the foreground (background notifications are displayed automatically). The service should override the `onMessageReceived` callback. It should handle any message within 20 seconds of receipt (10 seconds on Android Marshmallow) - the Fanmeter SDK respects these limits and executes in milliseconds. You must guarantee that you don't add any extra processing that consumes the available computation time. The next lines show an example of a class that extends `FirebaseMessagingService` and that calls the SDK's `EntryPoint.launchFanmeterNotification`.

```kotlin
import android.util.Log
import com.google.firebase.messaging.FirebaseMessagingService
import com.google.firebase.messaging.RemoteMessage
// Loads the Fanmeter SDK.
import com.pluggableai.fanmeter.EntryPoint

class MyFirebaseMessagingService: FirebaseMessagingService() {
    companion object {
        private const val TAG: String = "MyFirebaseMessagingService"
    }

    override fun onMessageReceived(remoteMessage: RemoteMessage) {
        Log.d(TAG, "Notification received from: ${remoteMessage.from}")
        // Check if message contains a data payload.
        if (remoteMessage.data.isNotEmpty()) {
            launchFanmeter(remoteMessage)
        }
    }

    private fun launchFanmeter(remoteMessage: RemoteMessage) {
        Log.d(TAG, "Message data payload: ${remoteMessage.data}")
        // Execute the Fanmeter SDK and show a local notification to the user.
        EntryPoint.launchFanmeterNotification(
            this,
            remoteMessage.data,
            null
        ){
            Log.d(TAG, "onMessageReceived | launchFanmeterNotification: $it")
        }
    }
}
```

Where:
  * *this*, is the application context;
  * *notificationData*, the remote data received with the notification;
  * *notificationClassResponse*, the name of the class that is being instantiated when the user clicks the notification - example: "com.company.activities.SearchActivity" (null opens the app's default view).

Finally, the `launchFanmeterView` method should be called on a button click, redirecting users to Fanmeter's default UI. This method accepts an eventId, that should be obtained using the `getEventData` method. If a null eventId is passed, this method will open the closest available event to the current date.

```kotlin
// Launch view button!
Button(onClick = {
    EntryPoint.launchFanmeterView(
        applicationContext
    ){
        Log.d(TAG, "EntryPoint.launchFanmeterView(): $it")
    }
})
```

Where:
  * *applicationContext*, is the application context.

These functions return the following values:

  * 1, success;
  * -80, no GPS/PUSH permissions;
  * -81, GPS disabled;
  * -82, invalid event coordinates;
  * -89, SDK not initialized;
  * -91, invalid notification data;
  * -92, invalid company license;
  * -93, invalid event;
  * -94, event not happening now;
  * -95, invalid external user data;
  * -96, failed to get event data;
  * -97, failed to start the Fanmeter service;
  * -98, another Fanmeter service is already running.

You are also **required to subscribe the user to a FCM topic** so that the user can receive Fanmeter notifications. This can be done, for example, on your Main Activity.

```kotlin
//...

class MainActivity: ComponentActivity() {
    //...

    override fun onCreate(savedInstanceState: Bundle?) {
        //...
        if(intent != null){
            launchFanmeter(intent)
        }
        //...
        Firebase.messaging.subscribeToTopic("football_senior").addOnCompleteListener { task ->
            Log.d("messaging.subscribeToTopic", task.isSuccessful)
        }
    }
}
```

**NOTE**: You can, and should, customize the used notification icon. For that, just add the desired icon in the Android's drawable folder and name it `ic_push_app_icon`.

### Build your own UI

If you want full control and implement your own UI, this library exposes three methods, that must be called as demonstrated below. In particular:

  * `startService`, starts the Fanmeter service that enables Fanmeter for your client's device during a particular event;
  * `stopService`, stops the Fanmeter service. The service will, still, terminate automatically as soon as the event ends. This returns 1, if success, otherwise an error code;
  * `isServiceRunning`, used to check the current status of the Fanmeter service. Returns 1, if the service is running, otherwise 0.

The `startService` method is used to start the Fanmeter service. This method accepts an eventId, that should be obtained using the `getEventData` method and should be called as follows (associated, for example, to a particular button):

```kotlin
// Start the Fanmeter Service using a specific button!
Button(onClick = {
    EntryPoint.startService(
        applicationContext,
        EVENT_ID,
        null
    ){
        Log.d("EntryPoint.startService", "Service started: $it")
    }
})
```

Where:
  * *applicationContext*, is the application context;
  * *EVENT_ID*, the id of the event the user will participate when the start service is called;
  * *notificationClassResponse*, the name of the class that is being instantiated when the user clicks the notification - example: "com.company.activities.SearchActivity" (null opens the app's default view).

This method returns the following values:

  * 1, success;
  * -80, no GPS/PUSH permissions;
  * -81, GPS disabled;
  * -82, invalid event coordinates;
  * -89, SDK not initialized;
  * -92, invalid company license;
  * -93, invalid event;
  * -94, event not happening now;
  * -95, invalid external user data;
  * -96, failed to get event data;
  * -97, failed to start the Fanmeter service;
  * -98, another Fanmeter service is already running.

The `stopService` method is used to stop the Fanmeter service (can be toggled with the previous method). Even if the user does not explicitly stop the service, it will automatically stop as soon as the event finishes. This returns 1, if success, otherwise an error code.

```kotlin
// Stop the Fanmeter Service using a specific button!
Button(onClick = {
    EntryPoint.stopService(
        applicationContext
    ){
        Log.d("EntryPoint.stopService", "Service stopped: $it")
    }
})
```

Where:
  * *applicationContext*, is the application context.

Finally, the `isServiceRunning` method is used to check the current status of the Fanmeter service. This returns 1, if the service is running, otherwise 0.

```kotlin
// Check the Fanmeter Service status.
EntryPoint.isServiceRunning(
    applicationContext
){
    Log.d("EntryPoint.isServiceRunning", "Service status: $it")
}
```

Where:
  * *applicationContext*, is the application context.

### Additional info

Other important methods are to **request user permission** to be able to **send notifications** and **request GPS permission**. Also, **get the user token and listen for changes** to get the user FCM_TOKEN and update it when it changes.

In addition, **prevent multiple calls to the launchFanmeterView** method so that it is only triggered when no previous call is in progress.

**NOTE**: You can, and should, customize the used notification icon. For that, just add the desired icon in the Android's drawable folder and name it `ic_push_app_icon`.

In addition, please note that FCM via APNs does not work on iOS Simulators. To receive messages and notifications a real device is required. The same is recommended for Android.