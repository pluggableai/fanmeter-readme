# Pluggable's Fanmeter Plugin for React Native apps

The Fanmeter plugin is the one responsible for enabling Fanmeter in any mobile application. It is a simple plug-&-play plugin that makes available a set of methods that can be used to allow users to participate in activations such as the FAN OF THE MATCH or the SUPER FAN.

## Table of Contents

+ [Overview](#overview)
+ [Requirements](#requirements)
  + [Minimum versions](#minimum-versions)
  + [Permissions and Info.plist](#permissions-and-infoplist)
  + [Google Play policy declarations](#google-play-policy-declarations)
+ [Getting Started](#getting-started)
  + [1. Install the plugin](#1-install-the-plugin)
  + [2. Set up push notifications](#2-set-up-push-notifications)
  + [3. Initialize the plugin](#3-initialize-the-plugin)
  + [4. Handle Fanmeter notifications](#4-handle-fanmeter-notifications)
+ [Plug-&-Play UI](#plug--play-ui)
  + [5. Add the Fanzone screen](#5-add-the-fanzone-screen)
  + [6. Open the Fanzone from your app](#6-open-the-fanzone-from-your-app)
+ [Build your own UI](#build-your-own-ui)
  + [5. Get the event](#5-get-the-event)
  + [6. Build your event screen](#6-build-your-event-screen)
  + [7. Listen to the user's participation (optional)](#7-listen-to-the-users-participation-optional)
+ [Reference](#reference)
  + [Methods](#methods)
  + [Return codes](#return-codes)
  + [Participation states](#participation-states)

## Overview

There are two possible types of integration. One is plug-&-play and uses the Fanzone, Pluggable's web-based activation interface, that you present to your users for them to participate in Fanmeter events. If, on the other hand, you wish to create your own Fanmeter UI, you will follow a "build your own UI" approach.

  1. **[Plug-&-Play UI](#plug--play-ui)** (recommended): you want everything automated (including the Fanzone view);

  2. **[Build your own UI](#build-your-own-ui)**: you want to handle the conditions yourself and develop your own UI (calling the *startService*, *stopService*, and *isServiceRunning* methods).

Both integrations start by meeting the [Requirements](#requirements) and following the [Getting Started](#getting-started) steps (1 to 4). Then, follow the steps of the integration you chose. Every step is required, unless marked as optional.

## Requirements

### Minimum versions

For full compatibility, be sure to use the recommended versions: **XCODE=15**, **SWIFT=5.9**, and **COCOAPODS=1.14.2**.

For **Android**, this plugin requires a minimum **compileSdk** version of **34**. Make sure your project's `android/build.gradle` file has `compileSdk 34` or higher configured.

### Permissions and Info.plist

On Android, push notification permission is required to inform the user that a foreground service is running. Additionally, Location permission is necessary for fans to participate in geo-restricted events. Make sure to request these permissions.

For iOS, add the **Background Modes** capability and enable **Location Updates**. Additionally, open your _Info.plist_ file and add the following code at the bottom of the file:

```xml
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

### Google Play policy declarations

For **Android only**, when publishing to the Google Play Store an app that integrates Fanmeter, you are required to fill a **Policy Declaration** justifying two sensitive permissions Fanmeter relies on: the **Data Sync foreground service** and **background location**.

#### Foreground Service (Data Sync)

Fanmeter is required to use a **Data Sync foreground service** to communicate non-intrusive sensor data with Pluggable's server. Since API 34 of Android, when publishing an app that uses such services, you must justify the need for it. You will be asked the question "*What tasks require your app to use the FOREGROUND_SERVICE_DATA_SYNC permission?*". There, you should:

1. Select the option **OTHER** in **Other tasks**;

2. Provide the following video link:

    ```text
    https://youtu.be/w4d7Pgksok0
    ```

3. Provide the following description:

    ```text
    Fanmeter is an Android SDK, incorporated in this app, that uses a Data Sync foreground service to communicate non-intrusive sensor data with Pluggable servers, which are used to quantify the engagement of the user in real-time. The foreground service must start as soon as the user opts to participate in the event (so that it can collect and communicate data) and must keep running until the user himself decides to terminate his/her participation.
    ```

#### Access Background Location

Fanmeter relies on **background location** to guarantee that only fans physically present at the venue can participate, and to keep validating that presence throughout the whole match without forcing the fan to keep the app open. When publishing an app that requests the `ACCESS_BACKGROUND_LOCATION` permission, Google Play requires a **Background Location Policy Declaration**, where you must justify the feature with the app purpose, the reason background access is needed, and a demonstration video:

1. Provide the **app purpose**:

    ```text
    This app powers the FAN OF THE MATCH, a digital activation that identifies and quantifies the fans who show the most support during a match by measuring emotional engagement through non-intrusive smartphone sensors. To ensure that only fans physically present at the venue can participate, continuous location validation is essential. This verification must be maintained throughout the entire match, guaranteeing that participation is exclusive to attendees at the venue.
    ```

2. Justify **why background location is required**:

    ```text
    Since events can last hours, it is essential that fans can participate even when they are not actively using the app. Without background location, they would be forced to continuously interact with the app, which goes against the essence of the FAN OF THE MATCH - allow fans to engage naturally and immerse without distractions. Background location access is fundamental, ensuring a seamless and fair experience by enabling participation without interrupting the enjoyment of the match.
    ```

3. Provide a demonstration video link (either of the following):

    ```text
    https://youtu.be/w4d7Pgksok0
    https://www.youtube.com/watch?v=6trJOoCGfGU
    ```

## Getting Started

### 1. Install the plugin

To install the [Fanmeter Package](https://www.npmjs.com/package/fanmeter-sdk-reactnative) just run in the root of the project:

```bash
npm install fanmeter-sdk-reactnative
```

Once installed, the library must be linked to your project and your application needs to be rebuilt. Users on React Native 0.60+ automatically have access to "autolinking", requiring no further manual installation steps. To automatically link the package, rebuild your project:

```bash
# Android apps
npm run android

# iOS apps
cd ios/
pod install --repo-update
```

### 2. Set up push notifications

Fanmeter notifications reach your app through [React Native Firebase](https://rnfirebase.io/) and [Firebase Cloud Messaging (FCM)](https://rnfirebase.io/messaging/usage) - if not done already, integrate them.

Then, **request user permission** to be able to **send notifications**, and **get the user token and listen for changes** to get the user FCM_TOKEN (to pass as `fcmToken` in step 3) and update it when it changes:

```tsx
// Get user's FCM token. 
messaging().getToken().then(token => {
    console.log('TOKEN is:', token);
    FCM_TOKEN = token;
});
```

```tsx
// Listen to whether the token changes.
messaging().onTokenRefresh(token => {
    console.log('TOKEN refreshed to:', token);
    FCM_TOKEN = token;
});
```

You are also **required to subscribe the user to a FCM topic** so that the user can receive Fanmeter notifications. This can be done, for example, as follows:

```tsx
// Subscribe to a topic.
messaging().subscribeToTopic('football_senior').then(() => console.log('Subscribed to topic: football_senior!'));
```

For **Android**, to customize the used notification icon, just add the desired icon in the Android's drawable folder and name it **ic_push_app_icon**. Otherwise, a default icon, not related to your app, will be used.

**NOTE**: FCM via APNs **does not work on iOS Simulators**. To receive messages & notifications a real device is required. The same is recommended for Android.

**NOTE 2**: In iOS, foreground notifications are not displayed. If needed, you can use a package for local notifications.

### 3. Initialize the plugin

The `initialize` method is responsible for initializing the SDK and must be invoked before using any other method (a possible approach is to invoke it in the app's main activity). It returns an auth token on success, or null otherwise.

```tsx
import { 
    initialize,
    ...
} from 'fanmeter-sdk-reactnative';

const YOUR_COMPANY_NAME: string = 'companyName';
const YOUR_COMPANY_KEY: string = 'AAAA-BBBB-CCCC-DDDD';
const USER_ID: string = 'externalUserId';
let TOKEN_ID: string = 'externalTokenId';
const USERNAME: string | null = 'username';
const USER_EMAIL: string | null = 'externalUserEmail';
const FCM_TOKEN: string | null = 'fcm_token';
const FANMETER_LOG: boolean | null = false;

// ...
// Plug-&-Play UI: keep the auth token at a scope both your init flow and the Fanzone view can
// reach, since the Fanzone (step 5) needs it to load its page.
const [authToken, setAuthToken] = useState<string | null>(null);

// When the Fanmeter SDK is initialized, it initializes all company and user data.
const res = await initialize(
    {
        companyName: YOUR_COMPANY_NAME,
        companyKey: YOUR_COMPANY_KEY,
        externalUserId: USER_ID,
        externalTokenId: TOKEN_ID,
        // Optional:
        username: USERNAME,
        externalUserEmail: USER_EMAIL,
        fcmToken: FCM_TOKEN,
    },
    FANMETER_LOG
);
setAuthToken(res);
```

Where - required keys:
  * companyName, the name of your company in Fanmeter;
  * companyKey, your company's license key;
  * externalUserId, the user identifier in your db (can be the username, the uuid, ...);
  * externalTokenId, the individual smartphone identifier (allows for the same account in different devices).

Optional keys:
  * username, the user's display name;
  * externalUserEmail, the user's email;
  * fcmToken, the FCM token id of the user.

Additional parameter:
  * FANMETER_LOG, enables additional logging.

### 4. Handle Fanmeter notifications

This library exposes two methods to handle Fanmeter notifications, that must be called as demonstrated below:
  * `execute`, processes a tapped Fanmeter notification and returns the event the user should be directed to;
  * `launchFanmeterNotification`, launches a local Fanmeter notification to the user, which is required by Android when the app is in the foreground.

The `execute` method is called when a Fanmeter notification is tapped. It does not open any view itself - it returns a `{ code, eventId }` object and, when `eventId` is not null, your app is responsible for opening the screen of that event: the Fanzone, in the Plug-&-Play UI, or your own event screen, in the Build your own UI. On the other hand, the `launchFanmeterNotification` method launches a local notification when the Android app is in a foreground state. An example is as follows, used in your *.tsx* files as demonstrated in the next lines. 

```tsx
import { 
    ...
    execute,
    launchFanmeterNotification
} from 'fanmeter-sdk-reactnative';

// `openFanzoneForEvent` is the Plug-&-Play UI helper defined in step 5.
// In the Build your own UI, open your own event screen instead.

// Register background handler. This runs with no UI mounted, so the returned eventId is not used
// to open anything here - the Fanzone is only opened when the notification is actually tapped.
messaging().setBackgroundMessageHandler(async remoteMessage => {
    const notificationData = remoteMessage.data as { [key: string]: any };
    execute(
        notificationData, 
        null
    ).then(({ code, eventId }) => {
        console.log("[setBackgroundMessageHandler] Result: ", code, eventId);
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
        ).then(({ code, eventId }) => {
            if(eventId != null) openFanzoneForEvent(eventId);
        });
    }
});

// When the application is running, but in the background.
messaging().onNotificationOpenedApp(remoteMessage => {
    const notificationData = remoteMessage.data as {[key: string]: any};
    execute(
        notificationData,
        null
    ).then(({ code, eventId }) => {
        if(eventId != null) openFanzoneForEvent(eventId);
    });
});
```

Where:
  * notificationData, is the remote data received with the notification;
  * NOTIFICATION_CLASS_RESPONSE, the name of the class that is being instantiated when the user clicks the notification - example: "com.company.activities.SearchActivity" (null opens the app's default view).

On **Android**, a tap on the local notification launched by `launchFanmeterNotification` also has to reach your app so it can open the event's screen. Two methods are available for that: `getInitialNotificationClick`, for when the tap launched the app from a terminated state, and `onNotificationClick`, for when the app was already running. Both are Android-only - on iOS the SDK handles the notification tap natively, so `getInitialNotificationClick` always resolves null and `onNotificationClick` never fires. Neither of them reacts to FCM's own notifications, which are already handled above.

```tsx
import { 
    ...
    getInitialNotificationClick,
    onNotificationClick
} from 'fanmeter-sdk-reactnative';

useEffect(() => {
    function handleNotificationClick(extras) {
        execute(extras, null).then(({ code, eventId }) => {
            if(eventId != null) openFanzoneForEvent(eventId);
        });
    }

    // The app was launched from a terminated state by the notification.
    getInitialNotificationClick().then((extras) => {
        if(extras) handleNotificationClick(extras);
    });
    // The notification was tapped while the app was already running.
    const subscription = onNotificationClick(handleNotificationClick);
    return () => subscription.remove();
}, []);
```

The possible `code` values are listed in the [Return codes](#return-codes).

## Plug-&-Play UI

### 5. Add the Fanzone screen

Fanzone is Pluggable's web-based activation interface - it is the view you present to your fans. The Fanmeter plugin ships a built-in JavaScript bridge so the Fanzone web app can call native SDK methods directly, loaded with the auth token returned by `initialize` (as an `Authorization: Bearer` header). There are two URLs to open, depending on how the user got there:

  * `https://data.fanofthematch.ai/fanzone/` - the **Fanzone home**, opened from your app itself (a button, a banner, a navbar item);
  * `https://data.fanofthematch.ai/fanzone/fotm?eventId=EVENT_ID` - the **Fan of the Match activation for one event**, opened from a tapped Fanmeter notification, using the event id that `execute` returned.

To use it, add [react-native-webview](https://github.com/react-native-webview/react-native-webview) to your app:

```bash
npm install react-native-webview

# iOS apps
cd ios/ && pod install
```

Then use the `useFanmeterWebView` hook, which returns the props to spread onto your own `<WebView>`:

```tsx
import { View, SafeAreaView, StyleSheet } from 'react-native';
import WebView from 'react-native-webview';
import { useFanmeterWebView } from 'fanmeter-sdk-reactnative';

const FANZONE_URL = 'https://data.fanofthematch.ai/fanzone/';

// Opened with FANZONE_URL from a button/banner/navbar, or with
// `${FANZONE_URL}fotm?eventId=${eventId}` after a notification tap.
function FanzoneScreen({ url, authToken, onClose }) {
    const fanzone = useFanmeterWebView({
        url,
        authToken,               // the token returned by initialize
        onCloseRequested: onClose,
    });

    return (
        <SafeAreaView style={styles.container}>
            <View {...fanzone.containerProps} style={styles.container}>
                <WebView {...fanzone.webViewProps} style={styles.container} />
            </View>
        </SafeAreaView>
    );
}

const styles = StyleSheet.create({ container: { flex: 1 } });
```

Then, open it with small helpers, used both from your app (step 6) and from a tapped notification (step 4). They also prevent multiple openings of the Fanzone view, so that it is only presented when no previous one is on screen:

```tsx
// Holding the URL to open so both entry points share the same screen:
const [fanzoneTarget, setFanzoneTarget] = useState<string | null>(null);

const openFanzone = (url: string) => {
    if(fanzoneTarget != null) return;
    if(authToken == null){
        console.warn('Cannot open the Fanzone without an auth token - initialize first.');
        return;
    }
    setFanzoneTarget(url);
}

// From a button/banner/navbar: the Fanzone home.
const openFanzoneHome = () => openFanzone(FANZONE_URL);
// From a tapped notification: the FOTM activation of the event execute returned.
const openFanzoneForEvent = (eventId: number) => openFanzone(`${FANZONE_URL}fotm?eventId=${eventId}`);

if(fanzoneTarget != null){
    return (
        <FanzoneScreen
            url={fanzoneTarget}
            authToken={authToken}
            onClose={() => setFanzoneTarget(null)}
        />
    );
}
```

Rules to keep in mind:

  * The `containerProps` must go on a plain `<View>` wrapping the `<WebView>` - that wrapper is how the plugin reaches the native WebView;
  * Do not override `source`, `onNavigationStateChange` or `javaScriptEnabled` on the `<WebView>` - they are what the bridge needs to work. The WebView stays blank for a moment before the Fanzone loads, until the bridge is installed - that is deliberate;
  * **Do not render a native header or close button around the WebView**: the Fanzone has its own, which calls `onCloseRequested` - wire it to whatever dismisses the screen. On Android, the hardware back calls it too, once there is nothing left to go back to in the Fanzone, so if it is a no-op the app quits.

If you would rather wire your own WebView by hand, `attachWebInterface(viewOrTag, { onCloseRequested })` is the lower-level call the hook is built on, and must be awaited before the Fanzone URL is loaded.

### 6. Open the Fanzone from your app

From a button, a banner or a navbar item, open the Fanzone home:

```tsx
openFanzoneHome();
```

## Build your own UI

### 5. Get the event

The `getEventData` method is used to obtain the full data of a particular Fanmeter event, in a dictionary, including the defined rewards and leaderboard classifications, if existing. If null, or not provided, this returns the closest event to date:

```tsx
import { 
    getEventData,
    ...
} from 'fanmeter-sdk-reactnative';

const EVENT_NAME: string | null = 'Round 1 2025-2026';


// After initialized, you will be able to use the SDK methods.
getEventData(
    EVENT_NAME
).then((res) => {
    console.log('Result ' + EVENT_NAME + ': ', res.eventId);
});

// OR.

// After initialized, you will be able to use the SDK methods.
getEventData().then((res) => {
    console.log("Result: ", res.eventId);
});
```

### 6. Build your event screen

In this integration, your app is responsible for the screen where the user participates in the event - it is the one you open from your app, and from a tapped notification (step 4). It should:

  * Show the event, with the data obtained in the previous step;
  * Offer a button for the user to join and leave the event, calling `startService` and `stopService`;
  * Show the current state of that button, with `isServiceRunning`;
  * Tell the user what went wrong whenever `startService` returns an error code (see the [Return codes](#return-codes)).

The `startService` method is used to start the Fanmeter service that enables Fanmeter for your client's device during a particular event. This method accepts the eventId obtained in the previous step and should be called as follows (associated, for example, to a particular button):

```tsx
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

The `stopService` method is used to stop the Fanmeter service (can be toggled with the previous method). Even if the user does not explicitly stop the service, it will automatically stop as soon as the event finishes. This returns 1, if success, otherwise an error code.

```tsx
stopService().then((res) => {
    console.log("Result: ", res);
});
```

Finally, the `isServiceRunning` method is used to check the current status of the Fanmeter service. This returns `isRunning`, true if the service is running, and `eventId`, the id of the event being tracked, which is only set while it is running.

```tsx
isServiceRunning().then(({ isRunning, eventId }) => {
    console.log("Result: ", isRunning, eventId);
});
```

### 7. Listen to the user's participation (optional)

You can subscribe to a listener that will notify you whenever there is a change in the user's participation status. For example, if the user leaves the venue of a geo-restricted event, the listener will alert you to this change. This can be particularly useful for updating the status of banners or buttons within your app.

Two methods are available: `subscribeUserParticipationListener` and `unsubscribeUserParticipationListener`. The first method subscribes the listener to a specific event id, and you can subscribe to multiple events. The second method unsubscribes the listener (if no event id is provided, it will unsubscribe all active listeners) and always returns 1 (Success), as it forces the stop of any or all running listeners.

```tsx
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
unsubscribeUserParticipationListener().then((res) => {
    console.log('Unsubscribed with result: ', res )
});
```

The possible `state` values are listed in the [Participation states](#participation-states).

## Reference

### Methods

| Method | What it does | Returns |
| --- | --- | --- |
| `initialize` | Initializes the SDK. Required before any other method. | The auth token, or null. |
| `execute` | Processes a tapped Fanmeter notification. | A `{ code, eventId }` object. |
| `launchFanmeterNotification` | Launches a local Fanmeter notification, when the Android app is in the foreground. | A return code. |
| `getInitialNotificationClick` / `onNotificationClick` | Detect a tap on a Fanmeter local notification (Android only). | The notification data. |
| `getEventData` | Gets an event's data (the closest event, if no title is given). | A dictionary. |
| `startService` | Starts the Fanmeter service for an event. | A return code. |
| `stopService` | Stops the Fanmeter service. | 1, if success, otherwise an error code. |
| `isServiceRunning` | Checks the current status of the Fanmeter service. | An `{ isRunning, eventId }` object. |
| `subscribeUserParticipationListener` | Notifies every change in the user's participation in an event. | A participation state. |
| `unsubscribeUserParticipationListener` | Cancels the listener of an event (or all of them). | Always 1. |
| `useFanmeterWebView` | Returns the props that install the Fanmeter bridge on your `<WebView>`. | `containerProps` and `webViewProps`. |
| `attachWebInterface` | Installs the Fanmeter bridge on a WebView wired by hand. | - |

### Return codes

The `execute` and `startService` methods return the following `code` values:
  * 1, success;
  * -80, no GPS/PUSH permissions;
  * -81, GPS disabled;
  * -82, invalid event coordinates;
  * -89, SDK not initialized;
  * -91, invalid notification data (`execute` only);
  * -92, invalid company license;
  * -93, invalid event;
  * -94, event not happening now;
  * -95, invalid external user data;
  * -96, failed to get event data;
  * -97, failed to start the Fanmeter service;
  * -98, another Fanmeter service is already running.

### Participation states

The `subscribeUserParticipationListener` listener can return the following status codes:
  * 0: User is not participating;
  * 1: Validating the user participation (validation may take up to 30 seconds);
  * 2: User is participating;
  * 3: User has GPS disabled;
  * 4: User is out of venue;
  * 5: Unknown GPS coordinates.

For more info visit [our website](https://pluggableai.xyz/) or give us feedback to <info@pluggableai.xyz>.
