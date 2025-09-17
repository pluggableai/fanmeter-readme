# Pluggable's Fanmeter Plugin for NativeScript apps

The Fanmeter plugin is the one responsible for enabling Fanmeter in any mobile application. It is a simple plug-&-play plugin that makes available a set of methods that can be used to allow users to participate in activations such as the FAN OF THE MATCH or the SUPER FAN.

## Plug-&-Play vs Build your own UI

There are two possible types of integration. One is plug-&-play and uses a default native UI that is launched to your users for them to participate in Fanmeter events - it is also able to launch and deliver push notifications as soon as the event starts and finishes, if you configure it. If, on the other hand, you wish to create your own Fanmeter UI, you will follow a "build your own UI" approach.

  1. **Plug-&-Play UI**: you want everything automated (including a default Fanmeter view). You should also integrate with FCM to handle received notifications that start the event (in summary, you'll need to use the *execute*, *launchFanmeterNotification* and the *launchFanmeterView* methods);

  2. **Build your own UI**: you want to handle the conditions yourself and develop your own UI (just call the *startService*, *stopService*, and *isServiceRunning* methods).

## Pre-Conditions

### Meta-Data

On Android, push notification permission is required to inform the user that a foreground service is running. Additionally, Location permission is necessary for fans to participate in geo-restricted events. Make sure to request these permissions.

For iOS, add the **Background Modes** capability and enable **Location Updates**. Additionally, open your _app/App_Resources/iOS/Info.plist_ file and add the following code at the bottom of the file:

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

### Push Notifications [Integrate with FCM]

Before using this plugin you should guarantee that your app already integrates with [NativeScript Firebase](https://v8.docs.nativescript.org/plugins/firebase-core.html) and then with [Firebase Cloud Messaging (FCM)](https://v8.docs.nativescript.org/plugins/firebase-messaging.html). The steps are quite easy to follow.

1. Install the `@nativescript/firebase-core` plugin (a plugin to initialize FirebaseApp in your app) by running the following command in the root directory of your project (this plugin must be installed before using any other Firebase service).
    ```
    npm install @nativescript/firebase-core
    ```

2. **Android Setup** - a configuration file must be downloaded and added to the project:
  * On the [Firebase console](https://console.firebase.google.com/), add a new Android application and enter the projects details. The "Android package name" must match the local package name which can be found inside your app's _nativescript.config.ts_ file;
  * Download the `google-services.json` file and place it inside the project at the following location: _/App_Resources/Android/src/google-services.json_.

3. **iOS Setup** - to allow the iOS app to securely connect to the Firebase project, a configuration file must be downloaded and added to the project:
  * On the [Firebase console](https://console.firebase.google.com/), add a new iOS application and enter your projects details. The "Apple bundle ID" must match your local project bundle ID. The bundle ID can be found within the "General" tab when opening the project with Xcode or in your app's _nativescript.config.ts_ file;
  * Download the `GoogleService-Info.plist` file and place it inside the project at the following location: _/App_Resources/iOS/GoogleService-Info.plist_;

4. Additional **iOS Setup** - iOS requires further configuration before you can start receiving and sending messages through Firebase. For instance:
  * You must upload your APNs authentication key to Firebase. If you don't already have an APNs authentication key, make sure to create one in the [Apple Developer Member Center](https://developer.apple.com/membercenter/index.action);
  * Inside your project in the Firebase console, select the gear icon, select Project Settings, and then select the **Cloud Messaging** tab;
  * In APNs authentication key under iOS app configuration, click the Upload button.
  * Browse to the location where you saved your key, select it, and click Open. Add the key ID for the key (available in the [Apple Developer Member Center](https://developer.apple.com/membercenter/index.action);) and click Upload.

**NOTE**: FCM via APNs **does not work on iOS Simulators**. To receive messages & notifications a real device is required. The same is recommended for Android.

5. Instantiate Firebase and initialize a default app (for example, in the _/app/app.ts_ file):
    ```
    import { firebase } from '@nativescript/firebase-core'
    const defaultApp = await firebase().initializeApp()
    ```

6. You can now install the messaging module by running:
    ```
    npm install @nativescript/firebase-messaging
    ```

7. Then, add the SDK by importing the `@nativescript/firebase-messaging` module. You should import this module once in your app, ideally in the main file (e.g. app.ts or main.ts).
    ```
    import '@nativescript/firebase-messaging'
    ```

8. You can then rebuild the project by running in the root's terminal:
    ```
    ns clean
    ns build android
    ns build ios
    ```

**NOTE**: Firebase Messaging in NativeScript **MUST be called as soon as possible in your App's initialization process**, otherwise it will no be able to detect and handle notifications and notification clicks. Please, make sure you implement all the required methods as soon as your app inits.

## Set up the Fanmeter Plugin in your NativeScript App

Few steps are required to be ready to use this Plugin.

### Install the Plugin

You are now ready to use this plugin to properly engage with your fans! To install the [Fanmeter Package](https://www.npmjs.com/package/@pluggableai/fanmeter-sdk) just run in the root of the project:
    ```
    npm install @pluggableai/fanmeter-sdk
    ```

For **Android**, to customize the used notification icon, just add the desired icon in the Android's drawable folder and name it **ic_push_app_icon**. Otherwise, a default icon, not related to your app, will be used.

### Methods initialize and getEventData

The `initialize` method is mandatory for plug-&-play UI. It is responsible for initializing the SDK and must be invoked before using any other method (a possible approach is to invoke it in the app's main activity). It returns success or an error code.

```
// ...
import { FanmeterSdk } from '@pluggableai/fanmeter-sdk';
// ...

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
let res = FanmeterSdk.initialize(
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
);
console.log('Fanmeter SDK initialized: ', result);
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
let EVENT_NAME = 'Round 1 2025-2026';

// After initialized, you will be able to use the SDK methods.
try {
    FanmeterSdk.getEventData(
        EVENT_NAME, 
        (data) => {
            console.log('Data for event "${EVENT_NAME}": ', data);
        }
    );
} catch (error) {
    console.log('Error fetching data for event "${EVENT_NAME}": ' + error);
}

// OR

// After initialized, you will be able to use the SDK methods.
try {
    FanmeterSdk.getEventData(
        null, 
        (data) => {
            console.log('Closest event data: ', data);
        }
    );
} catch (error) {
    console.log('Error getting event data for closest event: ' + error);
}
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
let EVENT_ID = '-99';

// Subscribing a listener.
FanmeterSdk.subscribeUserParticipationListener(
    EVENT_ID,
    (state: number) => {
        console.log(`Event ${EVENT_ID} | Participation state changed: ${state}`);
        // Update the status of your button/banner.
    }
);

// Unsubscribing a listener. If no event id is set, it cancels all listeners.
FanmeterSdk.unsubscribeUserParticipationListener(
    EVENT_ID,
    (res: number) => {
        console.log(`Event ${EVENT_ID} | Unsubscribed with result: ${res}`);
    }
);
```

### Plug-&-Play UI

After initializing the Plugin, you are now ready to start calling Fanmeter. In particular, if you want to automate the entire process, this library exposes three methods, that must be called as demonstrated below. In particular:
  * `execute`, launches the SDK Fanmeter default's view as soon as a Fanmeter notification is tapped by the user;
  * `launchFanmeterNotification`, launches a local Fanmeter notification to the user, which is required by Android when the app is in the foreground;
  * `launchFanmeterView`, launches the SDK Fanmeter default's view. It can be associated with a button or banner in your app, redirecting the user to the Fanmeter native view, allowing users without notification permissions to participate in the event. It will open the Fanmeter view with the event with the closest date to the current date.

The `execute` method is used for backgrounded processes and will open the default Fanmeter view to the user. On the other hand, the `launchFanmeterNotification` method launches a local notification when the Android app is in a foreground state. An example is as follows, used in your *.ts* files as demonstrated in the next lines.

```
// ...
import { FanmeterSdk } from '@pluggableai/fanmeter-sdk';
// ...

const defaultApp = async function () {
    console.log('defaultApp function started');
    await firebase().initializeApp();
    console.log('Firebase Initialized');
};
defaultApp();

// ...

const setFirebaseInits = async function () {
    // Allows you to always display notifications while the application is in the foreground.
    firebase().messaging().showNotificationsWhenInForeground = true;

    firebase().messaging().onNotificationTap((message) => {
        let notificationData = message.data;
        FanmeterSdk.execute(
            notificationData, 
            null, 
            (res: number) => {
                console.log('Fanmeter Notification Tap: ' + res);
            }
        );
    });

    firebase().messaging().onMessage(async (remoteMessage) => {
        let notificationData = remoteMessage.data;
        FanmeterSdk.launchFanmeterNotification(
            notificationData, 
            null, 
            (res: number) => {
                console.log('Fanmeter Message: ' + res);
            }
        );
    });
}
```

Where:
* notificationData, is the remote data received with the notification;
* NOTIFICATION_CLASS_RESPONSE, the name of the class that is being instantiated when the user clicks the notification - example: "com.company.activities.SearchActivity" (null opens the app's default view).

Finally, the `launchFanmeterView` method should be called on a button click, redirecting users to Fanmeter's default UI. This method accepts an eventId, that should be obtained using the `getEventData` method. If a null eventId is passed, this method will open the closest available event to the current date.

```
// Launch view!
try {
    FanmeterSdk.launchFanmeterView(
        EVENT_ID, 
        (res: number | null) => {
            if(res !== null) {
                console.log('Launch Fanmeter Native View for event id "${EVENT_ID}": ' + res);
            } else {
                console.log('No result returned');
            }
        }
    );
} catch (error) {
    console.log('Error launching Fanmeter Native View for event id "${EVENT_ID}": ' + error);
}

// OR.

try {
    FanmeterSdk.launchFanmeterView(
        null, 
        (res: number) => {
            console.log('Launch Fanmeter Native View: ' + res);
        }
    );
} catch (error) {
    console.log('Error launching Fanmeter Native View for closest event: ' + error);
}
```

Where:
* eventId, is the event id to show (nullable).

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
firebase().messaging().subscribeToTopic('football_senior').then(() => {
    console.log('Subscribed to topic: football_senior')
});
```

### Build your own UI

If you want full control and implement your own UI, this library exposes three methods, that must be called as demonstrated below. In particular:
  * `startService`, starts the Fanmeter service that enables Fanmeter for your client's device during a particular event;
  * `stopService`, stops the Fanmeter service. The service will, still, terminate automatically as soon as the event ends. This returns 1, if success, otherwise an error code;
  * `isServiceRunning`, used to check the current status of the Fanmeter service. Returns 1, if the service is running, otherwise 0.

The `startService` method is used to start the Fanmeter service. This method accepts an eventId, that should be obtained using the `getEventData` method and should be called as follows (associated, for example, to a particular button):

```
// Start the Fanmeter Service using a specific button!
try {
    FanmeterSdk.startService(
        EVENT_ID, 
        null, 
        (res: number) => {
            console.log(`Start service result: ${res}`);
        }
    );
} catch (error) {
    console.log(`Error starting service: ${error}`);
}
```

Where:
  * EVENT_ID, the id of the event the user will participate when the start service is called;
  * NOTIFICATION_CLASS_RESPONSE, the name of the class that is being instantiated when the user clicks the notification - example: "com.company.activities.SearchActivity" (null opens the app's default view).

The `stopService` method is used to stop the Fanmeter service (can be toggled with the previous method). Even if the user does not explicitly stop the service, it will automatically stop as soon as the event finishes. This returns 1, if success, otherwise an error code.

```
// Stop the Fanmeter Service using a specific button!
try {
    FanmeterSdk.stopService(
        (res: number) => {
            console.log(`Stop service result: ${res}`);
        }
    );
} catch (error) {
    console.log(`Error stopping service: ${error}`);
}
```

Finally, the `isServiceRunning` method is used to check the current status of the Fanmeter service. This returns 1, if the service is running, otherwise 0.

```
// Check the Fanmeter Service status.
try {
    FanmeterSdk.isServiceRunning(
        (res: number) => {
            console.log(`Service running status: ${res}`);
        }
    );
} catch (error) {
    console.log(`Error checking service status: ${error}`);
}
```

### Additional info

Other important methods are to **request user permission** to be able to **send notifications** and **request GPS permission**. Also, **get the user token and listen for changes** to get the user FCM_TOKEN and update it when it changes.

```
// Request user permission.
async function requestUserPermission() {
    const authStatus = await firebase().messaging().requestPermission({
        ios: {
            alert: true
        }
    })
    const enabled =
        authStatus === AuthorizationStatus.AUTHORIZED || 
        authStatus === AuthorizationStatus.PROVISIONAL
  
    console.log('Authorization status: ', authStatus)
    if(enabled) {
        const didRegister = await firebase().messaging().registerDeviceForRemoteMessages().then(() => {
            // Init Firebase and calling the below methods.
            setFirebaseInits();
        });
    }
}

// Get user's FCM token.
// Using, as example, the firebase.messaging plugin.
firebase().messaging().getToken().then((token) => {
    console.log('User token: ', token)
    FCM_TOKEN = token
    //saveTokenToDatabase(token)
});

// Listen to user token changes.
firebase().messaging().onToken((token) => {
    console.log('User token changed to: ', token)
    FCM_TOKEN = token
    //saveTokenToDatabase(token)
});
```

In addition, **prevent multiple calls to the launchFanmeterView** method so that it is only triggered when no previous call is in progress.

```
// Prevent multiple view openings.
// A quick example:
<!-- XML -->
<Button text="Launch View For Closest Event" tap="{{ launchView }}" class="btn btn-primary" isEnabled="{{ !isLoading }}"/>
<ActivityIndicator busy="{{ isLoading }}" visibility="{{ isLoading ? 'visible' : 'collapsed' }}"/>

// TypeScript (ViewModel)
isLoading: boolean = false;

launchView() {
    console.log('Preparing to launch Fanmeter Native View!');
    this.set('isLoading', true);
    try {
        FanmeterSdk.launchFanmeterView(
            null,  // Passing null to open the closest available event.
            (result: number | null) => {
                console.log('Launch Fanmeter Native View: ', result);
                this.set('isLoading', false);
            }
        );
    } catch (error) {
        console.log('Error launching Fanmeter Native View for closest event: ', error);
        this.set('isLoading', false);
    }
}
```

### Additional info

For full compatibility, attention to the used versions of XCODE, SWIFT and COCOAPODS. Recommended versions are **XCODE=15**, **SWIFT=5.9**, and **COCOAPODS=1.14.2**. For more info visit https://pluggableai.xyz/ or give us feedback to info@pluggableai.xyz.

