# Pluggable's Fanmeter Plugin for Cordova/Capacitor/Outsystems apps

The Fanmeter plugin is the one responsible for enabling Fanmeter in any mobile application. It is a simple plug-&-play plugin that makes available a set of methods that can be used to allow users to participate in activations such as the FAN OF THE MATCH or the SUPER FAN.

## Plug-&-Play vs Build your own UI

There are two possible types of integration. One is plug-&-play and uses a default native UI that is launched to your users for them to participate in Fanmeter events - it is also able to launch and deliver push notifications as soon as the event starts and finishes, if you configure it. If, on the other hand, you wish to create your own Fanmeter UI, you will follow a "build your own UI" approach.

  1. **Plug-&-Play UI**: you want everything automated (including a default Fanmeter view). You should also integrate with FCM to handle received notifications that start the event (in summary, you'll need to use the *execute*, *launchFanmeterNotification* and the *launchFanmeterView* methods);

  2. **Build your own UI**: you want to handle the conditions yourself and develop your own UI (just call the *startService*, *stopService*, and *isServiceRunning* methods).

Note that in both scenarios you can integrate with Firebase to handle received notifications from Fanmeter (in summary, you'll need to use the *execute* and the *launchFanmeterNotification* methods).

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

If you want a **fully plug-&-play experience** and wish to receive push notifications (if not receiving yet) from Fanmeter in your app, you should integrate `Firebase Cloud Messaging`. A complete, step-by-step tutorial is available in [Firebase's wiki](https://firebase.google.com/docs/cloud-messaging).

Unfortunately, there is no official support from Google to Cordova's Firebase. One possible plugin, [Cloud Messaging](https://firebase.google.com/docs/cloud-messaging), can be used even though it has a set of issues that require specific attention when implementing it. For capacitor/Outsystems, look for the official firebase plugin. Nonetheless, the integration of Firebase in Cordova can be done as follows:

1. Create a new [Firebase project](https://console.firebase.google.com/);

2. **Android Setup** - a configuration file must be downloaded and added to the project:
  * On the Firebase console, add a new Android application and enter the projects details. The "Android package name" must match the local package name which can be found inside your app's _config.xml_ file;
  * Download the `google-services.json` file and place it into the root directory of your Cordova project;
  * Add a new tag for the Android platform inside your app's _config.xml_ file:
    ```
    ...
    <platform name="android">
        <resource-file src="google-services.json" target="app/google-services.json" />
    </platform>
    ...
    ```

3. **iOS Setup** - to allow the iOS app to securely connect to the Firebase project, a configuration file must be downloaded and added to the project:
  * On the Firebase console, add a new iOS application and enter your projects details. The "Apple bundle ID" must match your local project bundle ID. The bundle ID can be found within the "General" tab when opening the project with Xcode or in your app's _config.xml_ file;
  * Download the `GoogleService-Info.plist` file and place it into the root directory of your Cordova project;
  * Add a new tag for the iOS platform inside your app's _config.xml_ file:
    ```
    ...
    <platform name="ios">
        <resource-file src="GoogleService-Info.plist" />
    </platform>
    ...
    ```

4. Additional **iOS Setup** - iOS requires further configuration before you can start receiving and sending messages through Firebase. For instance:
  * You must upload your APNs authentication key to Firebase. If you don't already have an APNs authentication key, make sure to create one in the [Apple Developer Member Center](https://developer.apple.com/membercenter/index.action);
  * Inside your project in the Firebase console, select the gear icon, select Project Settings, and then select the Cloud Messaging tab;
  * In APNs authentication key under iOS app configuration, click the Upload button.
  * Browse to the location where you saved your key, select it, and click Open. Add the key ID for the key (available in the [Apple Developer Member Center](https://developer.apple.com/membercenter/index.action);) and click Upload.
  * For further notes, you can check [React-Native iOS with Firebase Cloud Messaging](https://rnfirebase.io/messaging/usage/ios-setup), which is quite illustrative.

**NOTE**: FCM via APNs **does not work on iOS Simulators**. To receive messages & notifications a real device is required. The same is recommended for Android.

## Set up the Fanmeter Plugin in your Cordova App

Few steps are required to be ready to use this SDK.

### Install the Plugin

You are now ready to use this plugin to properly engage with your fans! To install the [Fanmeter Package](https://www.npmjs.com/package/cordova-plugin-fanmeter) just run in the root of the project:

```
cordova plugin add cordova-plugin-fanmeter
```

For **Android**, to customize the used notification icon, just add the desired icon in the Android's drawable folder and name it **ic_push_app_icon**. Otherwise, a default icon, not related to your app, will be used.

### Methods initialize and getEventData

The `initialize` method is mandatory for both plug-&-play and build your own UI scenarios. It is responsible for initializing the SDK and must be invoked before using any other method (a possible approach is to invoke it in the app's main activity). It returns success or an error code.

```
let YOUR_COMPANY_NAME = 'companyName';
let YOUR_COMPANY_KEY = 'AAAA-BBBB-CCCC-DDDD';
let USER_ID = 'externalUserId';
let TOKEN_ID = 'externalTokenId';
let USER_EMAIL = 'externalUserEmail';
let FCM_TOKEN = 'fcm_token';
let TICKET_NUMBER = 'ticket_number';
let TICKET_STAND = 'ticket_stand';
let REGULATION_URL = 'https://link.to.the.regulation';
let FANMETER_LOG = false;

// When the Fanmeter SDK is initialized, it initializes all company and user data.
cordova.plugins.FanmeterPlugin.initialize(
    YOUR_COMPANY_NAME,
    YOUR_COMPANY_KEY,
    USER_ID, 
    TOKEN_ID, 
    USER_EMAIL, 
    FCM_TOKEN, 
    TICKET_NUMBER, 
    TICKET_STAND, 
    REGULATION_URL, 
    FANMETER_LOG,
    function(response){
        console.log('[FANMETER INITIALIZE] Success : ' + JSON.stringify(response));
    }, function(error){
        console.log('[FANMETER INITIALIZE] Error: ' + JSON.stringify(error));
    }
);
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

The `getEventData` method is used to obtain the full data of a particular Fanmeter event, in a dictionary, including the defined rewards and leaderboard classifications, if existing. If null, or not provided, this returns the closest event to date:

```
let EVENT_NAME = 'Round 1 2025-2026'
let EVENT_ID = '-99'

// After initialized, you will be able to use the SDK methods.
cordova.plugins.FanmeterPlugin.getEventData(
    EVENT_NAME,
    function(response){
        console.log('[GETEVENTDATA] Success: ', response);
        // Setting the eventId.
        EVENT_ID = response['eventId'];
    }, 
    function(error) {
        console.log('[GETEVENTDATA] Error: ', error);
    }
);

// OR.

// After initialized, you will be able to use the SDK methods.
cordova.plugins.FanmeterPlugin.getEventData(
    null,
    function(response){
        console.log('[GETEVENTDATA] Success: ', response);
        // Setting the eventId.
        EVENT_ID = response['eventId'];
    }, 
    function(error) {
        console.log('[GETEVENTDATA] Error: ', error);
    }
);
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
cordova.plugins.FanmeterPlugin.subscribeUserParticipationListener(
    EVENT_ID,
    function(response){
        console.log('[SUBSCRIBEUSERPARTICIPATION] ' + EVENT_ID + ' | Participation Status | ' + JSON.stringify(response, null, 2));
        // Update the status of your button/banner.
    }, 
    function(error){
        console.log('[SUBSCRIBEUSERPARTICIPATION] ' + EVENT_ID + ' | Failed to subscribe | ' + error);
    }
);

// OR.

// Unsubscribing a listener. If no event id is set, it cancels all listeners.
cordova.plugins.FanmeterPlugin.unsubscribeUserParticipationListener(
    EVENT_ID,
    function(response){
        console.log('[UNSUBSCRIBEUSERPARTICIPATION] ' + EVENT_ID + ' | Unsubscribed | ' + response);
    }, function(error){
        console.log('[UNSUBSCRIBEUSERPARTICIPATION] ' + EVENT_ID + ' | Failed to unsubscribe | ' + error);
    }
);
```

### Plug-&-Play UI

After initializing the Plugin, you are now ready to start calling Fanmeter. In particular, if you want to automate the entire process, this library exposes three methods, that must be called as demonstrated below. In particular:
  * `execute`, launches the SDK Fanmeter default's view as soon as a Fanmeter notification is tapped by the user;
  * `launchFanmeterNotification`, launches a local Fanmeter notification to the user, which is required by Android when the app is in the foreground;
  * `launchFanmeterView`, launches the SDK Fanmeter default's view. It can be associated with a button or banner in your app, redirecting the user to the Fanmeter native view, allowing users without notification permissions to participate in the event. It will open the Fanmeter view with the event with the closest date to the current date.

The `execute` method is used for backgrounded processes and will open the default Fanmeter view to the user. On the other hand, the `launchFanmeterNotification` method launches a local notification when the Android app is in a foreground state. An example is as follows, used in your *.js* files as demonstrated in the next lines. 

```
// Register the background handler and notification tap.
// Using, as example, the firebase.messaging plugin.
cordova.plugins.firebase.messaging.onBackgroundMessage(function(notificationData) {
    cordova.plugins.FanmeterPlugin.execute(
        notificationData,
        null,
        function(response){
            console.log('[FANMETER EXECUTE] Success: ' + JSON.stringify(response));
        }, function(error){
            console.log('[FANMETER EXECUTE] Error: ' + JSON.stringify(error));
        }
    );
});

// Register the foreground handler.
// Using, as example, the firebase.messaging plugin.
cordova.plugins.firebase.messaging.onMessage(function(notificationData) {
    cordova.plugins.FanmeterPlugin.launchFanmeterNotification(
        notificationData,
        null,
        function(response){
            console.log('[LAUNCHFANMETERNOTIFICATION] Success: ' + JSON.stringify(response));
        }, function(error){
            console.log('[LAUNCHFANMETERNOTIFICATION] Error: ' + JSON.stringify(error));
        }
    );
});
```

Where:
  * notificationData, is the remote data received with the notification;
  * NOTIFICATION_CLASS_RESPONSE, the name of the class that is being instantiated when the user clicks the notification - example: "com.company.activities.SearchActivity" (null opens the app's default view).

Finally, the `launchFanmeterView` method should be called on a button click, redirecting users to Fanmeter's default UI. This method accepts an eventId, that should be obtained using the `getEventData` method. If a null eventId is passed, this method will open the closest available event to the current date.

```
// Launch view!
cordova.plugins.FanmeterPlugin.launchFanmeterView(
    EVENT_ID,
    function(response){
        console.log('[LAUNCHFANMETERVIEW] Success: ' + response);
    }, function(error){
        console.log('[LAUNCHFANMETERVIEW] Error: ' + error);
    }
);

// OR.

// Launch view!
cordova.plugins.FanmeterPlugin.launchFanmeterView(
    null,
    function(response){
        console.log('[LAUNCHFANMETERVIEW] Success: ' + response);
    }, function(error){
        console.log('[LAUNCHFANMETERVIEW] Error: ' + error);
    }
);

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
// Using, as example, the firebase.messaging plugin.
cordova.plugins.firebase.messaging.subscribe("football_senior");
```

### Build your own UI

If you want full control and implement your own UI, this library exposes three methods, that must be called as demonstrated below. In particular:
  * `startService`, starts the Fanmeter service that enables Fanmeter for your client's device during a particular event;
  * `stopService`, stops the Fanmeter service. The service will, still, terminate automatically as soon as the event ends. This returns 1, if success, otherwise an error code;
  * `isServiceRunning`, used to check the current status of the Fanmeter service. Returns 1, if the service is running, otherwise 0.

The `startService` method is used to start the Fanmeter service. This method accepts an eventId, that should be obtained using the `getEventData` method and should be called as follows (associated, for example, to a particular button):

```
// Start the Fanmeter Service using a specific button!
cordova.plugins.FanmeterPlugin.startService(
    EVENT_ID, 
    NOTIFICATION_CLASS_RESPONSE, 
    function(response){
        console.log('[STARTSERVICE] Success: ' + JSON.stringify(response));
    }, function(error){
        console.log('[STARTSERVICE] Error: ' + JSON.stringify(error));
  });
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
// Stop the Fanmeter Service using a specific button!
cordova.plugins.FanmeterPlugin.stopService(
    function(response){
        console.log('[STOPSERVICE] Success: '  + JSON.stringify(response));
    }, function(error){
        console.log('[STOPSERVICE] Error: ' + JSON.stringify(error));
    }
);
```

Finally, the `isServiceRunning` method is used to check the current status of the Fanmeter service. This returns 1, if the service is running, otherwise 0.

```
// Check the Fanmeter Service status.
cordova.plugins.FanmeterPlugin.isServiceRunning(
    function(response){
        console.log("'[ISSERVICERUNNING] Success: ' + JSON.stringify(response));
    }, function(error){
        console.log('[ISSERVICERUNNING] Error: ' + JSON.stringify(error));
    }
);
```

### Additional info

Other important methods are to **request user permission** to be able to **send notifications** and **request GPS permission**. Also, **get the user token and listen for changes** to get the user FCM_TOKEN and update it when it changes.

``` 
// Request push permission.
cordova.plugins.firebase.messaging.requestPermission({forceShow: true}).then(function() {
    console.log("Push messaging is allowed");
});

// Get user's FCM token.
// Using, as example, the firebase.messaging plugin.
cordova.plugins.firebase.messaging.getToken().then(function(token) {
    // Get the token each time the application loads.
    FCM_TOKEN = token
});
``` 

In addition, **prevent multiple calls to the launchFanmeterView** method so that it is only triggered when no previous call is in progress.

If your platform app (iOS and/or Android) is unable to get the new plugin, then, between steps 1. and 2., you'll probably need to relink the platform data:
```
# Android apps
cordova platform remove android
cordova platform add android

# iOS apps
cordova platform remove ios
cordova platform add ios
```

For full compatibility, attention to the used versions of XCODE, SWIFT and COCOAPODS. Recommended versions are **XCODE=15**, **SWIFT=5.9**, and **COCOAPODS=1.14.2**. For more info visit https://pluggableai.xyz/ or give us feedback to info@pluggableai.xyz.
