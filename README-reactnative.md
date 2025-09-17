# Pluggable's Fanmeter Plugin for React Native apps

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

If you want a **fully plug-&-play experience**and wish to receive push notifications (if not receiving yet) from Fanmeter in your app, you should integrate [React Native Firebase](https://rnfirebase.io/) and then with [Firebase Cloud Messaging (FCM)](https://rnfirebase.io/messaging/usage). The steps are as follows:

1. Create a new [Firebase project](https://console.firebase.google.com/). Note that current versions of firebase-ios-sdk have a minimum Xcode requirement of 14.1, which implies a minimum macOS version of 12.5 (macOS Monterey);

2. Install the React Native Firebase "app" module to the root of the React Native project with NPM: `npm install --save @react-native-firebase/app`. The `@react-native-firebase/app` module must be installed before using any other Firebase service;

3. **Android Setup** - a configuration file must be downloaded and added to the project:
    * On the Firebase console, add a new Android application and enter the projects details. The "Android package name" must match the local projects package name which can be found inside of the manifest tag within the _/android/app/src/main/AndroidManifest.xml_ file within the project;
    * Download the `google-services.json` file and place it inside of the project at the following location: _/android/app/google-services.json_;
    * To allow Firebase on Android to use the credentials, the google-services plugin must be enabled on the project. This requires modification to two files in the Android directory. First, add the google-services plugin as a dependency inside of the _/android/build.gradle_ file:
    ```
    buildscript {
        dependencies {
            // ... other dependencies
            classpath 'com.google.gms:google-services:4.3.15'
        }
    }
    ```
    * Lastly, execute the plugin by adding the following to the _/android/app/build.gradle_ file:
    ```
    apply plugin: 'com.android.application'
    apply plugin: 'com.google.gms.google-services' // <- Add this line 
    ```

4. **iOS Setup** - to allow the iOS app to securely connect to the Firebase project, a configuration file must be downloaded and added to the project, and you must enable frameworks in CocoaPods:
    * On the Firebase console, add a new iOS application and enter your projects details. The "Apple bundle ID" must match your local project bundle ID. The bundle ID can be found within the "General" tab when opening the project with Xcode;
    * Download the `GoogleService-Info.plist` file;
    * Using Xcode, open the projects _/ios/{projectName}.xcodeproj_ file (or _/ios/{projectName}.xcworkspace_ if using Pods) and right click on the project name and "Add files" to the project. Select the downloaded `GoogleService-Info.plist` file from your computer, and ensure the **Copy items if needed** checkbox is enabled;
    * To allow Firebase on iOS to use the credentials, the Firebase iOS SDK must be configured during the bootstrap phase of your application. To do this, open your _/ios/{projectName}/AppDelegate.mm_ file (or _AppDelegate.m_ if on older react-native), and add the following:
    ```
    #import "AppDelegate.h"
    #import <Firebase.h> // <- Add this line
    // ...
    - (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions {
        // Add me --- \/
        [FIRApp configure]; // <- Add this line
        // Add me --- /\
        // ...
    }
    // ...
    ```
    * Beginning with **firebase-ios-sdk v9+** (react-native-firebase v15+) you must tell CocoaPods to use frameworks. Open the file _./ios/Podfile_ and add the following line inside your targets (right after the line calling the react native Podfile function to get the native modules config):
    ```
    target 'reactnativeapp' do
    config = use_native_modules!
    use_frameworks! :linkage => :static # <- Add this line
    $RNFirebaseAsStaticFramework = true # <- Add this line (see next point)
    ...
    # Note that if you have use_frameworks! enabled, Flipper will not work and
    # you should disable the next line.
    #:flipper_configuration => flipper_config, # <- COMMENT this line
    ...
    ```
    * To use Static Frameworks on iOS, you also need to manually enable this for the project with the following global to your _./ios/Podfile_ file:
    ```
    # right after `use_frameworks! :linkage => :static`
    $RNFirebaseAsStaticFramework = true
    ```
    * iOS requires further configuration before you can start receiving and sending messages through Firebase. Read the documentation and follow the steps on how to setup [iOS with Firebase Cloud Messaging](https://rnfirebase.io/messaging/usage/ios-setup).

5. Once the above steps have been completed, the React Native Firebase library must be linked to the project and the application needs to be rebuilt. Users on React Native 0.60+ automatically have access to "autolinking", requiring no further manual installation steps. To automatically link the package, rebuild your project (for manual linking, if you're using an older version of React Native without autolinking support, or wish to integrate into an existing project, you can follow the manual installation steps for [iOS](https://rnfirebase.io/install-ios) and [Android](https://rnfirebase.io/install-android)):
    ```
    # Android apps
    npm run android

    # iOS apps
    cd ios/
    pod install --repo-update
    cd ..
    npm run ios
    ```

6. You can now install the messaging module by running:
    ```
    # Install the messaging module
    npm install @react-native-firebase/messaging

    # If you're developing your app using iOS, run this command
    cd ios/ && pod install

    cd ..
    npm run ios
    ```

**NOTE**: FCM via APNs **does not work on iOS Simulators**. To receive messages & notifications a real device is required. The same is recommended for Android. Also, for iOS, React-Native-Firebase uses `use_frameworks`, which has compatibility issues with Flipper, Hermes, and Fabric.

**NOTE 2**: In iOS foreground notification are not displayed. If it is needed you can use a package for local notifications.

## Set up the Fanmeter Plugin in your React Native App

Few steps are required to be ready to use this SDK.

### Install the Plugin

After configuring your app to integrate with FCM, you are ready to use this plugin to properly engage with your fans! To install the [Fanmeter Package](https://www.npmjs.com/package/fanmeter-sdk-reactnative) just run in the root of the project:
    ```
    npm install fanmeter-sdk-reactnative
    ```

Once the above steps have been completed, the React Native Firebase library must be linked to your project and your application needs to be rebuilt. Users on React Native 0.60+ automatically have access to "autolinking", requiring no further manual installation steps. To automatically link the package, rebuild your project:
    ```
    # Android apps
    npm run android

    # iOS apps
    cd ios/
    pod install --repo-update
    ```
    
**OPTIONAL**: For **iOS**, if you want to display push notifications while your app is in the foreground, you must allow that specific scenario:

* In your app's _AppDelegate.h_ file:
    ```
    // Import the native iOS UserNotifications package
    #import <UserNotifications/UserNotifications.h> // <- Add this line

    // Extend the UNUserNotificationCenterDelegate class
    @interface AppDelegate : RCTAppDelegate<UNUserNotificationCenterDelegate> // <- Update this line
    ```

* In your app's _AppDelegate.mm_ file:
    ```
    // Import the native iOS UserNotifications package
    #import <UserNotifications/UserNotifications.h> // <- Add this line

    // ...

    - (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions
    {
        // ...
        [UNUserNotificationCenter currentNotificationCenter].delegate = self; // <- Add this line
        // ...
    }
    
    // ...

    // Function to implement to display foreground push notifications
    - (void)userNotificationCenter:(UNUserNotificationCenter *)center willPresentNotification:(UNNotification *)notification withCompletionHandler:(void (^)(UNNotificationPresentationOptions options))completionHandler {
        completionHandler(UNNotificationPresentationOptionAlert|UNNotificationPresentationOptionBadge|UNNotificationPresentationOptionSound);
    }
    ```

For **Android**, to customize the used notification icon, just add the desired icon in the Android's drawable folder and name it **ic_push_app_icon**. Otherwise, a default icon, not related to your app, will be used.

### Methods initialize and getEventData

The `initialize` method is mandatory for both plug-&-play and build your own UI scenarios. It is responsible for initializing the SDK and must be invoked before using any other method (a possible approach is to invoke it in the app's main activity). It returns success or an error code.

```
import { 
    initialize,
    ...
} from 'fanmeter-sdk-reactnative';

const YOUR_COMPANY_NAME: string = 'companyName';
const YOUR_COMPANY_KEY: string = 'AAAA-BBBB-CCCC-DDDD';
const USER_ID: string = 'externalUserId';
let TOKEN_ID: string = 'externalTokenId';
const USER_EMAIL: string | null = 'externalUserEmail';
const FCM_TOKEN: string | null = 'fcm_token';
const TICKET_NUMBER: string | null = 'ticket_number';
const TICKET_STAND: string | null = 'ticket_stand';
const REGULATION_URL: string | null = 'https://link.to.the.regulation';
const FANMETER_LOG: boolean | null = false;

// ...
const [resultInitialize, setResultInitialize] = useState<string>('');

// When the Fanmeter SDK is initialized, it initializes all company and user data.
const res = await initialize(
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
);
setResultInitialize(`Success: ${res}`);
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
import { 
    getEventData,
    ...
} from 'fanmeter-sdk-reactnative';

const EVENT_NAME: string | null = 'Round 1 2025-2026';


// After initialized, you will be able to use the SDK methods.
getEventData(
    EVENT_NAME
).then((res) => {
    console.log('Result ' + eventTitle + ': ', res.eventId);
});

// OR.

// After initialized, you will be able to use the SDK methods.
getEventData().then((res) => {
    console.log("Result: ", res.eventId);
});
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
import {
    ...
    subscribeUserParticipationListener,
    unsubscribeUserParticipationListener
} from 'fanmeter-sdk-reactnative';

// Subscribing a listener.
subscribeUserParticipationListener(EVENT_ID, (state, eventId) => {
    console.log(`Event ${JSON.stringify(eventId)} | Participation state changed: ${JSON.stringify(state)}`);
});

// OR.

// Unsubscribing a listener. If no event id is set, it cancels all listeners.
unsubscribeUserParticipationListener().then((res) = > {
    console.log('Unsubscribed with result: ', res )
});
```

### Plug-&-Play UI

After initializing the Plugin, you are now ready to start calling Fanmeter. In particular, if you want to automate the entire process, this library exposes three methods, that must be called as demonstrated below. In particular:
  * `execute`, launches the SDK Fanmeter default's view as soon as a Fanmeter notification is tapped by the user;
  * `launchFanmeterNotification`, launches a local Fanmeter notification to the user, which is required by Android when the app is in the foreground;
  * `launchFanmeterView`, launches the SDK Fanmeter default's view. It can be associated with a button or banner in your app, redirecting the user to the Fanmeter native view, allowing users without notification permissions to participate in the event. It will open the Fanmeter view with the event with the closest date to the current date.

The `execute` method is used for backgrounded processes and will open the default Fanmeter view to the user. On the other hand, the `launchFanmeterNotification` method launches a local notification when the Android app is in a foreground state. An example is as follows, used in your *.tsx* files as demonstrated in the next lines. 

```
import { 
    ...
    execute,
    launchFanmeterNotification,
    launchFanmeterView
} from 'fanmeter-sdk-reactnative';

// Register background handler.
messaging().setBackgroundMessageHandler(async remoteMessage => {
    const notificationData = remoteMessage.data as { [key: string]: any };
    execute(
        notificationData, 
        null
    ).then((res) => {
        console.log("[setBackgroundMessageHandler] Result: ", res);
    });
});

// To listen to messages in the foreground, call the onMessage method inside of your application code. 
// Code executed via this handler has access to React context.

messaging().onMessage(async remoteMessage => {
    const notificationData = remoteMessage.data as {[key: string]: any};
    launchFanmeterNotification(
        notificationData,
        null
    ).then((result) => {
        console.log("Result: ", result);
    });
});

// When the application is opened from a quit state, check whether an initial notification is available.
messaging().getInitialNotification().then(remoteMessage => {
    if(remoteMessage){
        const notificationData = remoteMessage.data as {[key: string]: any};
        execute(
            notificationData,
            null
        ).then((result) => {
            console.log("Result: ", result);
        });
    }
});

// When the application is running, but in the background.
messaging().onNotificationOpenedApp(remoteMessage => {
    const notificationData = remoteMessage.data as {[key: string]: any};
    execute(
        notificationData,
        null
    ).then((result) => {
        console.log("Result: ", result);
    });
});
```

Where:
  * notificationData, is the remote data received with the notification;
  * NOTIFICATION_CLASS_RESPONSE, the name of the class that is being instantiated when the user clicks the notification - example: "com.company.activities.SearchActivity" (null opens the app's default view).

Finally, the `launchFanmeterView` method should be called on a button click, redirecting users to Fanmeter's default UI. This method accepts an eventId, that should be obtained using the `getEventData` method. If a null eventId is passed, this method will open the closest available event to the current date.

```
// Launch view!
launchFanmeterView().then((res) => {
    console.log("Result: ", res);
});

// OR.

// Launch view!
launchFanmeterView(EVENT_ID).then((res) => {
    console.log("Result: ", res);
});
```

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
// Subscribe to a topic.
messaging().subscribeToTopic('football_senior').then(() => console.log('Subscribed to topic: football_senior!'));
```

### Build your own UI

If you want full control and implement your own UI, this library exposes three methods, that must be called as demonstrated below. In particular:
  * `startService`, starts the Fanmeter service that enables Fanmeter for your client's device during a particular event;
  * `stopService`, stops the Fanmeter service. The service will, still, terminate automatically as soon as the event ends. This returns 1, if success, otherwise an error code;
  * `isServiceRunning`, used to check the current status of the Fanmeter service. Returns 1, if the service is running, otherwise 0.

The `startService` method is used to start the Fanmeter service. This method accepts an eventId, that should be obtained using the `getEventData` method and should be called as follows (associated, for example, to a particular button):

```
import { 
    ...
    startService,
    stopService,
    isServiceRunning
} from 'fanmeter-sdk-reactnative';

// ...

// You should manage the events to get their name programmatically instead of an hardcoded string.
startService(
    EVENT_ID,
    null
).then((res) => {
    console.log("Result: ", res);
});
```

Where:
  * eventId, the id of the event the user will participate when the start service is called;
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
stopService().then((res) => {
    console.log("Result: ", res);
});
```

Finally, the `isServiceRunning` method is used to check the current status of the Fanmeter service. This returns true, if the service is running, otherwise false.

```
isServiceRunning().then((res) => {
    console.log("Result: ", res);
});
```

### Additional info

Other important methods are to **request user permission** to be able to **send notifications** and **request GPS permission**. Also, **get the user token and listen for changes** to get the user FCM_TOKEN and update it when it changes.

In particular, if you need to know the FCM token ID associated with each user device:
```
// Get user's FCM token. 
messaging().getToken().then(token => {
    console.log('TOKEN is:', token);
    FCM_TOKEN = token;
});
```

```
// Listen to whether the token changes.
messaging().onTokenRefresh(token => {
    console.log('TOKEN refreshed to:', token);
    FCM_TOKEN = token;
});
```

In addition, **prevent multiple calls to the launchFanmeterView** method so that it is only triggered when no previous call is in progress.

```
// Prevent multiple view openings.
// A quick example:
const [isLoading, setIsLoading] = useState(false);

const launchView = () => {
    if(isLoading) return;
    console.log('Preparing to launch Fanmeter Native View!');
    setIsLoading(true);
    try {
        FanmeterSdkReactnative.launchFanmeterView(
            [], // Open the closest available event.
            (result) => {
                console.log('Launch Fanmeter Native View result:', result);
                setIsLoading(false);
            }
        );
    } catch (error) {
        console.error('Error launching Fanmeter Native View:', error);
        setIsLoading(false);
    }
}
return (
    <View style={styles.container}>
        <View style={styles.buttonContainer}>
            <Button title="Launch View For Closest Event!" onPress={launchView} disabled={isLoading} />
        </View>
        {isLoading && <ActivityIndicator size="small" style={styles.spinner} />}
    </View>
);


const styles = StyleSheet.create({container: {padding: 16, alignItems: 'center'}, buttonContainer: {marginBottom: 10}, spinner: {marginTop: 10}});
```

For full compatibility, attention to the used versions of XCODE, SWIFT and COCOAPODS. Recommended versions are **XCODE=15**, **SWIFT=5.9**, and **COCOAPODS=1.14.2**. For more info visit https://pluggableai.xyz/ or send your feedback to info@pluggableai.xyz.