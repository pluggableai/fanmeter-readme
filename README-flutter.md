# Pluggable's Fanmeter Plugin for Flutter apps

The Fanmeter plugin is the one responsible for enabling Fanmeter in any mobile application. It is a simple plug-&-play plugin that makes available a set of methods that can be used to allow users to participate in activations such as the FAN OF THE MATCH or the SUPER FAN.

## Plug-&-Play vs Build your own UI

There are two possible types of integration. One is plug-&-play and uses a default native UI that is launched to your users for them to participate in Fanmeter events - it is also able to launch and deliver push notifications as soon as the event starts and finishes, if you configure it. If, on the other hand, you wish to create your own Fanmeter UI, you will follow a "build your own UI" approach.

  1. **Plug-&-Play UI**: you want everything automated (including a default Fanmeter view). You should also integrate with FCM to handle received notifications that start the event (in summary, you'll need to use the *execute*, *launchFanmeterNotification* and the *launchFanmeterView* methods);

  2. **Build your own UI**: you want to handle the conditions yourself and develop your own UI (just call the *startService*, *stopService*, and *isServiceRunning* methods).

## Pre-Conditions

### Meta-Data

On Android, push notification permission is required to inform the user that a foreground service is running. Additionally, Location permission is necessary for fans to participate in geo-restricted events. Make sure to request these permissions.

For iOS, add the **Background Modes** capability and enable **Location Updates**. Additionally, open your _Info.plist_ file and add the following code at the bottom of the file:

```
<key>UIBackgroundModes</key>
<array>
    <string>fetch</string>
    <string>location</string>
    <string>remote-notification</string>
</array>
<key>NSLocationTemporaryUsageDescriptionDictionary</key>
<dict>
   <key>PreciseLocationRequired</key>
   <string>Access to precise location is required during Fanmeter events!</string>
</dict>
<key>NSLocationAlwaysUsageDescription</key>
<string>Location access is required to participate in Fanmeter events!</string>
<key>NSLocationWhenInUseUsageDescription</key>
<string>Location access is required to participate in Fanmeter events!</string>
```

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

If you want a fully **plug-&-play experience** and wish to receive push notifications (if not receiving yet) from Fanmeter in your app, you should integrate `Firebase Cloud Messaging`. A complete, step-by-step tutorial is available in [Firebase's wiki](https://firebase.google.com/docs/cloud-messaging). The steps are as follows:

1. First install [FlutterFire](https://firebase.flutter.dev/docs/overview/), a tool which automatically configures your app to use firebase;

2. Then, install [firebase_messaging](https://firebase.flutter.dev/docs/messaging/overview/), a cross-platform messaging solution;

3. Then, to start using the Cloud Messaging package within your project follow these [steps](https://firebase.flutter.dev/docs/messaging/usage);

4. Integrating the Cloud Messaging plugin on iOS requires additional setup before your devices receive messages. The full steps are available [here](https://firebase.flutter.dev/docs/messaging/apple-integration), but the following prerequisites are required to be able to enable messaging:
    + You must have an active Apple Developer Account;
    + You must have a physical iOS device to receive messages.

5. Also for iOS:
    + If you are using swift, in your `AppDelegate.swift` make sure you have added the first code;
    + If you're using flutter with objective-c, add the second code to your `appdelegate.m` file.

    ```
    import Firebase

    FirebaseApp.configure() //add this before the code below
    GeneratedPluginRegistrant.register(with: self)
    ```

    ```
    didFinishLaunchingWithOptions:(NSDictionary *)launchOptions {
        [FIRApp configure]; //add this at the top
        // ...
    }
    ```

**NOTE**: FCM via APNs **does not work on iOS Simulators**. To receive messages & notifications a real device is required. The same is recommended for Android.

## Set up the Fanmeter Plugin in your Flutter App

Few steps are required to be ready to use this SDK.

### Install the Plugin

After configuring your app to integrate with FCM, you are ready to use this plugin to properly engage with your fans! To install the plugin just run, in the root of the project:

```
flutter pub add fanmeter
```

Or run `flutter pub upgrade` to update to the latest compatible versions of all the dependencies listed in the **pubspec.yaml** file.

For **Android**, to customize the used notification icon, just add the desired icon in the Android's drawable folder and name it **ic_push_app_icon**. Otherwise, a default icon, not related to your app, will be used.

### Methods initialize and getEventData

The `initialize` method is mandatory for both plug-&-play and build your own UI scenarios. It is responsible for initializing the SDK and must be invoked before using any other method (a possible approach is to invoke it in the app's main activity). It returns success or an error code.

```
import 'package:fanmeter/fanmeter.dart';

// ...

String YOUR_COMPANY_NAME = 'companyName'
String YOUR_COMPANY_KEY = 'AAAA-BBBB-CCCC-DDDD'
String USER_ID = 'externalUserId'
String TOKEN_ID = 'externalTokenId'
String? USER_EMAIL = 'externalUserEmail'
String? FCM_TOKEN = 'fcm_token'
String? TICKET_NUMBER = 'ticket_number'
String? TICKET_STAND = 'ticket_stand'
String? REGULATION_URL = 'https://link.to.the.regulation'
bool? FANMETER_LOG = false;

final _fanmeterPlugin = Fanmeter();

// When the Fanmeter SDK is initialized, it initializes all company and user data.
final result = await _fanmeterPlugin.initialize(
    companyName: YOUR_COMPANY_NAME,
    companyKey: YOUR_COMPANY_KEY,
    externalUserId: USER_ID,
    externalTokenId: TOKEN_ID,
    externalUserEmail: USER_EMAIL,
    fcmToken: FCM_TOKEN,
    ticketNumber: TICKET_NUMBER,
    ticketStand: TICKET_STAND,
    urlRegulation: REGULATION_URL,
    log: FANMETER_LOG
);
print("[FANMETER - INITIALIZE]: $result");
```

Where:
  * YOUR_COMPANY_NAME, the name of your company in Fanmeter;
  * YOUR_COMPANY_KEY, your company's license key;
  * USER_ID, the user identifier in your db (can be the username, the uuid, ...);
  * TOKEN_ID, the individual smartphone identifier (allows for the same account in different devices);
  * USER_EMAIL, the user's email. Nullable;
  * FCM_TOKEN, the FCM token id of the user. Nullable;
  * TICKET_NUMBER, the user's ticket number - used for additional analytics. Nullable;
  * TICKET_STAND, the stand where the user is - used for additional analytics. Nullable;
  * REGULATION_URL, the URL to the regulation for your events. Nullable;
  * FANMETER_LOG, enables additional logging.


The `getEventData`()` method is used to obtain the full data of a particular Fanmeter event, in a dictionary, including the defined rewards and leaderboard classifications, if existing. If null, or not provided, this returns the closest event to date:

```
String? EVENT_NAME = 'Round 1 2025-2026'

// After initialized, you will be able to use the SDK methods.
final result = await _fanmeterPlugin.getEventData(EVENT_NAME);
print("[FANMETER - GETEVENTDATA - $EVENT_NAME]: $result");

// OR.

// After initialized, you will be able to use the SDK methods.
final result = await _fanmeterPlugin.getEventData();
print("[FANMETER - GETEVENTDATA]: $result");
```

### Listeners for User Participation

In both scenarios, you can subscribe to a listener that will notify you whenever there is a change in the user's participation status. For example, if the user leaves the venue of a geo-restricted event, the listener will alert you to this change. This can be particularly useful for updating the status of banners or buttons within your app.

Two methods are available: `subscribeUserParticipationListener` and `unsubscribeUserParticipationListener`. The first method subscribes the listener to a specific event id, and you can subscribe to multiple events. The second method unsubscribes the listener (if no event id is provided, it will unsubscribe all active listeners).

While the `unsubscribeUserParticipationListener` method always returns 1 (Success), as it forces the stop of any or all running listeners, the `subscribeUserParticipationListener` listener can return the following status codes:
  * 0: User is not participating;
  * 1: Validating the user participation (validation may take up to 30 seconds);
  * 2: User is participating;
  * 3: User has GPS disabled;
  * 4: User is out of venue;
  * 5: Unknown GPS coordinates.

These methods can be invoked as shown below:

```
// Subscribing a listener.
await _fanmeterPlugin.subscribeUserParticipationListener(EVENT_ID, ((value) {
    print("[FANMETER - SUBSCRIBEUSERPARTICIPATION] $EVENT_ID: $value");
    // Update the status of your button/banner.
}));
```

```
// Unsubscribing a listener. If no event id is set, it cancels all listeners.
final result = await _fanmeterPlugin.unsubscribeUserParticipationListener();
print("[FANMETER - UNSUBSCRIBEUSERPARTICIPATION]: $result");
```

### Plug-&-Play UI

After initializing the Plugin, you are now ready to start calling Fanmeter. In particular, if you want to automate the entire process, this library exposes three methods, that must be called as demonstrated below. In particular:
  * `execute`, launches the SDK Fanmeter default's view as soon as a Fanmeter notification is tapped by the user;
  * `launchFanmeterNotification`, launches a local Fanmeter notification to the user, which is required by Android when the app is in the foreground;
  * `launchFanmeterView`, launches the SDK Fanmeter default's view. It can be associated with a button or banner in your app, redirecting the user to the Fanmeter native view, allowing users without notification permissions to participate in the event. It will open the Fanmeter view with the event with the closest date to the current date.

The `execute` method is used for backgrounded processes and will open the default Fanmeter view to the user. On the other hand, the `launchFanmeterNotification` method launches a local notification when the Android app is in a foreground state. An example is as follows, used in your *.dart* files as demonstrated in the next lines. 

```
import 'package:fanmeter/fanmeter.dart';

// ...

/// Receive push while in the background.
@pragma('vm:entry-point')
Future<void> _firebaseMessagingBackgroundHandler(RemoteMessage message) async {
    // If you're going to use other Firebase services in the background, such as Firestore,
    // make sure you call `initializeApp` before using other Firebase services.
    await Firebase.initializeApp(
        options: DefaultFirebaseOptions.currentPlatform,
	);
    
    Map<String, String> notificationData = message.data.map((key, value) => MapEntry(key, value.toString()));

    final execute = await _fanmeterPlugin.execute(
        notificationData, 
        null
    );
    print("[Execute]: $execute");
}

/// Handle push tap by the user.
void _handlePushTap(RemoteMessage message) async {
    Map<String, String> notificationData = message.data.map((key, value) => MapEntry(key, value.toString()));
    final result = await _fanmeterPlugin.execute(
        notificationData, 
        null
    );
    print("[FANMETER - EXECUTE]: $result");
}

Future<void> setupInteractedMessage() async {
    // Get any messages which caused the application to open from a terminated state.
    RemoteMessage? initialMessage = await FirebaseMessaging.instance.getInitialMessage();
    if(initialMessage != null) {
        _handlePushTap(initialMessage);
    }
    // Also handle any interaction when the app is in the background.
    FirebaseMessaging.onMessageOpenedApp.listen(_handlePushTap);
}

/// Receive pushes while in the foreground.
void _firebaseMessagingForegroundHandler(RemoteMessage message) async {
    if(message.notification != null) {
        Map<String, String> notificationData = message.data.map((key, value) => MapEntry(key, value.toString()));
        final result = await _fanmeterPlugin.launchFanmeterNotification(
            notificationData,
            null
        );
        print("[FANMETER - LAUNCHFANMETERNOTIFICATION]: $result");
    }
}


@override
void initState() {
    // ...

    // Listen for foreground pushes.
    FirebaseMessaging.onMessage.listen(_firebaseMessagingForegroundHandler);

    // Handle notification tap.
    setupInteractedMessage();

    // ...
}
```

Where:
  * notificationData, is the remote data received with the notification;
  * NOTIFICATION_CLASS_RESPONSE, the name of the class that is being instantiated when the user clicks the notification - example: "com.company.activities.SearchActivity" (null opens the app's default view).

Finally, the `launchFanmeterView` method should be called on a button click, redirecting users to Fanmeter's default UI. This method accepts an eventId, that should be obtained using the `getEventData` method. If a null eventId is passed, this method will open the closest available event to the current date.

```
// Launch view!
final result = await _fanmeterPlugin.launchFanmeterView(EVENT_ID);
print("[FANMETER - LAUNCHFANMETERVIEW - EVENTID]: $result");

// OR.

// Launch view!
final result = await _fanmeterPlugin.launchFanmeterView();
print("[FANMETER - LAUNCHFANMETERVIEW]: $result");
```

Where:
  * EVENT_ID, is the event id to show (nullable).

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

You are also required to subscribe the user to a FCM topic so that the user can receive Fanmeter notifications. This can be done, for example, as follows:

```
// Subscribe to specific topic.
await FirebaseMessaging.instance.subscribeToTopic('football_senior');
```

### Build your own UI

If you want full control and implement your own UI, this library exposes three methods, that must be called as demonstrated below. In particular:
  * `startService`, starts the Fanmeter service that enables Fanmeter for your client's device during a particular event;
  * `stopService`, stops the Fanmeter service. The service will, still, terminate automatically as soon as the event ends. This returns 1, if success, otherwise an error code;
  * `isServiceRunning`, used to check the current status of the Fanmeter service. Returns 1, if the service is running, otherwise 0.

The `startService` method is used to start the Fanmeter service. This method accepts an eventId, that should be obtained using the `getEventData` method and should be called as follows (associated, for example, to a particular button):

```
// You can obtain the eventId of an event by calling the getEventData method.
final result = await _fanmeterPlugin.startService(
    EVENT_ID, 
    null
);
print("[FANMETER - STARTSERVICE]: $result");
```

Where:
  * EVENT_ID, the id of the event the user will participate when the start service is called;
  * NOTIFICATION_CLASS_RESPONSE, the name of the class that is being instantiated when the user clicks the notification - example: "com.company.activities.SearchActivity" (null opens the app's default view).

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

```
final result = await _fanmeterPlugin.stopService();
print("[FANMETER - STOPSERVICE]: $result");
```

Finally, the `isServiceRunning` method is used to check the current status of the Fanmeter service. This returns 1, if the service is running, otherwise 0.

```
final result = await _fanmeterPlugin.isServiceRunning();
print("[FANMETER - ISSERVICERUNNING]: $result");
```

### Additional info

Other important methods are to **request user permission** to be able to **send notifications** and **request GPS permission**. Also, **get the user token and listen for changes** to get the user FCM_TOKEN and update it when it changes.

```
Future requestPermission() async {
    FirebaseMessaging messaging = FirebaseMessaging.instance;
    // Ask for push permissions.
    NotificationSettings settings = await messaging.requestPermission(
        alert: true,
        announcement: false,
        badge: true,
        carPlay: false,
        criticalAlert: false,
        provisional: true,
        sound: true,
    );
    print('User granted permission: ${settings.authorizationStatus}');
    // Presentation for foreground push.
    await messaging.setForegroundNotificationPresentationOptions(
        alert: true,
        badge: true,
        sound: true,
    );
}

Future<void> setupToken() async {
    // Get the token each time the application loads.
    String? token = await FirebaseMessaging.instance.getToken();
    print('Device token is as follows: $token');
    fcmToken = token;
}
```

In addition, **prevent multiple calls to the launchFanmeterView** method so that it is only triggered when no previous call is in progress.

For full compatibility, attention to the used versions of XCODE, SWIFT and COCOAPODS. Recommended versions are **XCODE=15**, **SWIFT=5.9**, and **COCOAPODS=1.14.2**. For more info visit https://pluggableai.xyz/ or give us feedback to info@pluggableai.xyz.
