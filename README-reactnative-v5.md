# Pluggable's Fanmeter Plugin for React Native apps

The Fanmeter plugin is the one responsible for enabling Fanmeter in any mobile application. It is a simple plug-&-play plugin that makes available a set of methods that can be used to allow users to participate in activations such as the FAN OF THE MATCH or the SUPER FAN.

## Table of Contents

+ [Plug-&-Play vs Build your own UI](#plug--play-vs-build-your-own-ui)
+ [Pre-Conditions](#pre-conditions)
  + [Compatibility](#compatibility)
  + [Meta-Data](#meta-data)
  + [Push Notifications](#push-notifications)
+ [Set up the Fanmeter Plugin in your React Native App](#set-up-the-fanmeter-plugin-in-your-react-native-app)
  + [Install the Plugin](#install-the-plugin)
  + [Methods initialize and getEventData](#methods-initialize-and-geteventdata)
  + [Listeners for User Participation](#listeners-for-user-participation)
+ [Fanzone Web View](#fanzone-web-view)
+ [Plug-&-Play UI](#plug--play-ui)
+ [Build your own UI](#build-your-own-ui)
+ [Additional info](#additional-info)

## Plug-&-Play vs Build your own UI

There are two possible types of integration. One is plug-&-play and uses the Fanzone, Pluggable's web-based activation interface, that you present to your users for them to participate in Fanmeter events - it is also able to launch and deliver push notifications as soon as the event starts and finishes, if you configure it. If, on the other hand, you wish to create your own Fanmeter UI, you will follow a "build your own UI" approach.

  1. **Plug-&-Play UI**: you want everything automated (including the Fanzone view). You should also integrate with FCM to handle received notifications that start the event (in summary, you'll need to use the *execute* and *launchFanmeterNotification* methods, plus the [Fanzone Web View](#fanzone-web-view));

  2. **Build your own UI**: you want to handle the conditions yourself and develop your own UI (just call the *startService*, *stopService*, and *isServiceRunning* methods).

## Pre-Conditions

### Compatibility

For full compatibility, be sure to use the recommended versions: **XCODE=15**, **SWIFT=5.9**, and **COCOAPODS=1.14.2**.

For **Android**, this plugin requires a minimum **compileSdk** version of **34**. Make sure your project's `android/build.gradle` file has `compileSdk 34` or higher configured.

### Meta-Data

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

Note that, for Android only, Fanmeter is required to use a **Data Sync foreground service** to communicate non-intrusive sensor data with Pluggable's server. Since API 34 of Android, when publishing an app that uses such services, you are required to fill a **Policy Declaration** where you must justify the need for such services. In Fanmeter's case, when filling such policy, you will be asked the question "*What tasks require your app to use the FOREGROUND_SERVICE_DATA_SYNC permission?*". There, you should:

1. Select the option **OTHER** in **Other tasks**;

2. Provide the following video link:

    ```text
    https://youtu.be/w4d7Pgksok0
    ```

3. Provide the following description:

    ```text
    Fanmeter is an Android SDK, incorporated in this app, that uses a Data Sync foreground service to communicate non-intrusive sensor data with Pluggable servers, which are used to quantify the engagement of the user in real-time. The foreground service must start as soon as the user opts to participate in the event (so that it can collect and communicate data) and must keep running until the user himself decides to terminate his/her participation.
    ```

### Push Notifications

If you want a **fully plug-&-play experience** and wish to receive push notifications (if not receiving yet) from Fanmeter in your app, you should integrate [React Native Firebase](https://rnfirebase.io/) and then [Firebase Cloud Messaging (FCM)](https://rnfirebase.io/messaging/usage).

**NOTE**: FCM via APNs **does not work on iOS Simulators**. To receive messages & notifications a real device is required. The same is recommended for Android.

**NOTE 2**: In iOS, foreground notifications are not displayed. If needed, you can use a package for local notifications.

## Set up the Fanmeter Plugin in your React Native App

Few steps are required to be ready to use this SDK.

### Install the Plugin

After configuring your app to integrate with FCM, you are ready to use this plugin to properly engage with your fans! To install the [Fanmeter Package](https://www.npmjs.com/package/fanmeter-sdk-reactnative) just run in the root of the project:

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

For **Android**, to customize the used notification icon, just add the desired icon in the Android's drawable folder and name it **ic_push_app_icon**. Otherwise, a default icon, not related to your app, will be used.

### Methods initialize and getEventData

The `initialize` method is mandatory for both plug-&-play and build your own UI scenarios. It is responsible for initializing the SDK and must be invoked before using any other method (a possible approach is to invoke it in the app's main activity). It returns an auth token on success, or null otherwise.

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
// Keep the auth token at a scope both your init flow and the Fanzone view can reach, since the
// Fanzone (see below) needs it to load its page.
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

## Fanzone Web View

Fanzone is Pluggable's web-based activation interface - it is the view you present to your fans (for example, the one you open after `execute` returns an event). The Fanmeter plugin ships a built-in JavaScript bridge so the Fanzone web app can call native SDK methods directly, without any platform-specific logic on the JavaScript side. The app retains full control over the WebView - the plugin only configures the bridge.

Before loading the Fanzone, the SDK must be initialized using the `initialize` method in order to obtain a valid authentication token. This token must then be included in the request used to load the Fanzone page (as an `Authorization: Bearer` header).

There are two URLs to open, depending on how the user got there:

  * `https://data.fanofthematch.ai/fanzone/` - the **Fanzone home**, listing every activation. This is what you open when the user reaches the Fanzone from your app itself: a button, a banner, a navbar item. No event id is involved;
  * `https://data.fanofthematch.ai/fanzone/fotm?eventId=EVENT_ID` - the **Fan of the Match activation for one event**. This is where a tapped Fanmeter notification leads, using the event id that `execute` returned.

To use it, add [react-native-webview](https://github.com/react-native-webview/react-native-webview) to your app:

```bash
npm install react-native-webview

# iOS apps
cd ios/ && pod install
```

Then use the `useFanmeterWebView` hook, which returns the props to spread onto your own `<WebView>`. Your app keeps full ownership of the WebView - it renders it in its own tree and decides how the screen is presented - and the SDK shows its own localized GPS permission dialogs when they are needed, so you do not have to build them yourself.

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

A few things to keep in mind:

  * The `containerProps` must go on a plain `<View>` wrapping the `<WebView>`. That wrapper is how the plugin reaches the native WebView, since react-native-webview's own ref is an imperative handle rather than a view;
  * The hook only hands `webViewProps.source` over once the bridge is installed, so the WebView stays blank for a moment before the Fanzone loads. That is deliberate: the bridge is injected at document start and only applies to page loads started after it, so loading the URL yourself would leave the page without `window.FanmeterBridge`;
  * Do not override `source`, `onNavigationStateChange` or `javaScriptEnabled` on the `<WebView>` - they are what the bridge needs to work. Every other prop is yours;
  * Switching targets while the screen is up - a notification tapped while the Fanzone is already open - is handled for you: the new URL loads into the same WebView, and the plugin makes that page the root of the WebView's history, so the Android back leaves the screen instead of walking into the Fanzone the user came from.

The Fanzone ships its own close control, so **you should not render a native header or close button around the WebView** - the web layer already has one, and it closes the screen by calling `window.FanmeterBridge.closeWebView()`. The hook surfaces that call as the `onCloseRequested` callback, which you wire to whatever dismisses the screen - `() => navigation.goBack()`, or whatever your navigation looks like.

On Android, the hardware back button walks the Fanzone's own navigation first and, once there is nothing left to go back to, calls that same `onCloseRequested` - so both the Fanzone's close button and the system back leave the screen the same way. Wire it to something that actually dismisses the screen: if it is a no-op, the press falls through to React Native's default handler, which quits the app.

If you would rather wire your own WebView by hand, `attachWebInterface(viewOrTag, { onCloseRequested })` is the lower-level call the hook is built on. It takes a react tag, a ref, or a component pointing at the WebView or any of its ancestors, and must be awaited before the Fanzone URL is loaded.

## Plug-&-Play UI

After initializing the Plugin, you are now ready to start calling Fanmeter. In particular, if you want to automate the entire process, this library exposes two methods, that must be called as demonstrated below. In particular:
  * `execute`, processes a tapped Fanmeter notification and returns the event the user should be directed to (your app then opens the Fanzone view for that event);
  * `launchFanmeterNotification`, launches a local Fanmeter notification to the user, which is required by Android when the app is in the foreground.

The `execute` method is called when a Fanmeter notification is tapped. It does not open any view itself - it returns a `{ code, eventId }` object and, when `eventId` is not null, your app is responsible for opening the Fanzone view for that event (see the [Fanzone Web View](#fanzone-web-view) section). On the other hand, the `launchFanmeterNotification` method launches a local notification when the Android app is in a foreground state. An example is as follows, used in your *.tsx* files as demonstrated in the next lines. 

```tsx
import { 
    ...
    execute,
    launchFanmeterNotification
} from 'fanmeter-sdk-reactnative';

// `openFanzoneForEvent` is your own helper, opening FanzoneScreen with the `fotm?eventId=` URL
// shown in the "Fanzone Web View" section.

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

On **Android**, a tap on the local notification launched by `launchFanmeterNotification` also has to reach your app so it can open the Fanzone. Two methods are available for that: `getInitialNotificationClick`, for when the tap launched the app from a terminated state, and `onNotificationClick`, for when the app was already running. Both are Android-only - on iOS the SDK handles the notification tap natively, so `getInitialNotificationClick` always resolves null and `onNotificationClick` never fires. Neither of them reacts to FCM's own notifications, which are already handled above.

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

The `execute` method returns a `{ code, eventId }` object, where the code is one of the following values:
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

```tsx
// Subscribe to a topic.
messaging().subscribeToTopic('football_senior').then(() => console.log('Subscribed to topic: football_senior!'));
```

## Build your own UI

If you want full control and implement your own UI, this library exposes three methods, that must be called as demonstrated below. In particular:
  * `startService`, starts the Fanmeter service that enables Fanmeter for your client's device during a particular event;
  * `stopService`, stops the Fanmeter service. The service will, still, terminate automatically as soon as the event ends. This returns 1, if success, otherwise an error code;
  * `isServiceRunning`, used to check the current status of the Fanmeter service. Returns an object with whether it is running and, if so, the event being tracked.

The `startService` method is used to start the Fanmeter service. This method accepts an eventId, that should be obtained using the `getEventData` method and should be called as follows (associated, for example, to a particular button):

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

## Additional info

Other important methods are to **request user permission** to be able to **send notifications** and **request GPS permission**. Also, **get the user token and listen for changes** to get the user FCM_TOKEN and update it when it changes.

In particular, if you need to know the FCM token ID associated with each user device:
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

In addition, **prevent multiple openings of the Fanzone view** so that it is only presented when no previous one is on screen. Since the Fanzone is a screen in your own tree, that comes down to guarding whichever state drives it:

```tsx
// Prevent multiple view openings.
// A quick example, holding the URL to open so both entry points share the same screen:
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

For more info visit [our website](https://pluggableai.xyz/) or give us feedback to <info@pluggableai.xyz>.
