# Pluggable's Fanmeter Plugin for Flutter apps

The Fanmeter plugin is the one responsible for enabling Fanmeter in any mobile application. It is a simple plug-&-play plugin that makes available a set of methods that can be used to allow users to participate in activations such as the FAN OF THE MATCH (FOTM) or the SUPER FAN.

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

For full compatibility, be sure to use the recommended versions: **XCODE=26** (or later, as required by the App Store), **SWIFT=6.2**, and **FLUTTER=3.24** (or later). On iOS, the plugin requires **iOS 15.0** or later and supports both Swift Package Manager (the default since Flutter 3.44) and CocoaPods (**COCOAPODS=1.16.2**, only if your app has not migrated to Swift Package Manager yet).

### Permissions and Info.plist

On Android, push notification permission is required to inform the user that a foreground service is running. Additionally, Location permission is necessary for fans to participate in geo-restricted events. Make sure to request these permissions.

For iOS, add the **Background Modes** capability and enable **Location Updates**. Additionally, open your *Info.plist* file and add the following code at the bottom of the file:

```plist
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

    ```txt
    https://youtu.be/w4d7Pgksok0
    ```

3. Provide the following description:

    ```txt
    Fanmeter is an Android SDK, incorporated in this app, that uses a Data Sync foreground service to communicate non-intrusive sensor data with Pluggable servers, which are used to quantify the engagement of the user in real-time. The foreground service must start as soon as the user opts to participate in the event (so that it can collect and communicate data) and must keep running until the user himself decides to terminate his/her participation.
    ```

#### Access Background Location

Fanmeter relies on **background location** to guarantee that only fans physically present at the venue can participate, and to keep validating that presence throughout the whole match without forcing the fan to keep the app open. When publishing an app that requests the `ACCESS_BACKGROUND_LOCATION` permission, Google Play requires a **Background Location Policy Declaration**, where you must justify the feature with the app purpose, the reason background access is needed, and a demonstration video:

1. Provide the **app purpose**:

    ```txt
    This app powers the FAN OF THE MATCH, a digital activation that identifies and quantifies the fans who show the most support during a match by measuring emotional engagement through non-intrusive smartphone sensors. To ensure that only fans physically present at the venue can participate, continuous location validation is essential. This verification must be maintained throughout the entire match, guaranteeing that participation is exclusive to attendees at the venue.
    ```

2. Justify **why background location is required**:

    ```txt
    Since events can last hours, it is essential that fans can participate even when they are not actively using the app. Without background location, they would be forced to continuously interact with the app, which goes against the essence of the FAN OF THE MATCH - allow fans to engage naturally and immerse without distractions. Background location access is fundamental, ensuring a seamless and fair experience by enabling participation without interrupting the enjoyment of the match.
    ```

3. Provide a demonstration video link (either of the following):

    ```txt
    https://youtu.be/w4d7Pgksok0
    https://www.youtube.com/watch?v=6trJOoCGfGU
    ```

## Getting Started

### 1. Install the plugin

To install the plugin just run, in the root of the project:

```bash
flutter pub add fanmeter
```

Or run `flutter pub upgrade` to update to the latest compatible versions of all the dependencies listed in the **pubspec.yaml** file.

### 2. Set up push notifications

Fanmeter notifications reach your app through `Firebase Cloud Messaging` - if not done already, integrate it. A complete, step-by-step tutorial is available in [Firebase's official documentation](https://firebase.google.com/docs/cloud-messaging).

Then, **request user permission** to be able to **send notifications**, and **get the user token and listen for changes** to get the user FCM_TOKEN (to pass as `fcmToken` in step 3) and update it when it changes.

You are also **required to subscribe the user to a FCM topic** so that the user can receive Fanmeter notifications. This can be done, for example, as follows:

```dart
// Subscribe to specific topic.
await FirebaseMessaging.instance.subscribeToTopic('football_senior');
```

For **Android**, to customize the used notification icon, just add the desired icon in the Android's drawable folder and name it **ic_push_app_icon**. Otherwise, a default icon, not related to your app, will be used.

**NOTE**: FCM via APNs **does not work on iOS Simulators**. To receive messages & notifications a real device is required. The same is recommended for Android.

### 3. Initialize the plugin

The `initialize` method is responsible for initializing the SDK and must be invoked before using any other method (a possible approach is to invoke it before launching the FOTM). It returns an auth token on success, or `null` otherwise.

```dart
import 'package:fanmeter/fanmeter.dart';

// ...

/// The name of your company in Fanmeter
String YOUR_COMPANY_NAME = 'companyName';
/// Your company's license key
String YOUR_COMPANY_KEY = 'AAAA-BBBB-CCCC-DDDD';
/// The user identifier in your db (can be the username, the uuid, ...)
String USER_ID = 'externalUserId';
/// The individual smartphone identifier (allows for the same account in different devices)
String TOKEN_ID = 'externalTokenId';
/// The user's display name. Nullable
String? USERNAME = 'username';
/// The user's email. Nullable
String? USER_EMAIL = 'externalUserEmail';
/// The FCM token id of the user. Nullable
String? FCM_TOKEN = 'fcm_token';
/// Enables additional logging
bool FANMETER_LOG = false;

final _fanmeterPlugin = Fanmeter();

/// Plug-&-Play UI: keep the auth token at a scope both your init flow and the
/// Fanzone helper can reach (e.g. a top-level variable or a field in your app
/// state/service), since `openFanzone` (step 5) needs it to load the Fanzone page.
String? authToken;

// When the Fanmeter SDK is initialized, it initializes all company and user data.
authToken = await _fanmeterPlugin.initialize(
    initializeDataMap: {
        'companyName':       YOUR_COMPANY_NAME,
        'companyKey':        YOUR_COMPANY_KEY,
        'externalUserId':    USER_ID,
        'externalTokenId':   TOKEN_ID,
        // Optional:
        'username':          USERNAME,
        'externalUserEmail': USER_EMAIL,
        'fcmToken':          FCM_TOKEN,
    },
    log: FANMETER_LOG,
);
print("[FANMETER - INITIALIZE]: $authToken");
```

Where — required keys:

+ `companyName`, the exact company name in Fanmeter;
+ `companyKey`, your company's license key;
+ `externalUserId`, the user identifier in your db (can be the username, the uuid, ...);
+ `externalTokenId`, the individual smartphone identifier (allows for the same account in different devices).

Optional keys:

+ `username`, the user's display name;
+ `externalUserEmail`, the user's email;
+ `fcmToken`, the FCM token id of the user.

Additional parameter:

+ `log`, enables additional logging.

### 4. Handle Fanmeter notifications

This library exposes two methods to handle Fanmeter notifications, that must be called as demonstrated below:

+ `execute`, processes a tapped Fanmeter notification and returns the event the user should be directed to;
+ `launchFanmeterNotification`, launches a local Fanmeter notification to the user, which is required by Android when the app is in the foreground.

The `execute` method is called when a Fanmeter notification is tapped. It does not open any view itself — it returns a `(int? code, int? eventId)` record and, when `eventId` is not null, your app is responsible for opening the screen of that event: the Fanzone, in the Plug-&-Play UI, or your own event screen, in the Build your own UI. On the other hand, the `launchFanmeterNotification` method launches a local notification when the Android app is in a foreground state. An example is as follows, used in your *.dart* files as demonstrated in the next lines.

```dart
import 'package:fanmeter/fanmeter.dart';

// ...

// `openFanzoneForEvent` is the Plug-&-Play UI helper defined in step 5.
// In the Build your own UI, open your own event screen instead.

/// Receive push while in the background.
/// NOTE: this runs in a separate background isolate with no widget tree, so it
/// cannot open the Fanzone view here — the view is opened on notification tap
/// (see `_handlePushTap`), once the app is in the foreground.
@pragma('vm:entry-point')
Future<void> _firebaseMessagingBackgroundHandler(RemoteMessage message) async {
    await Firebase.initializeApp(
        options: DefaultFirebaseOptions.currentPlatform,
    );

    Map<String, String> notificationData = message.data.map((key, value) => MapEntry(key, value.toString()));

    final (code, eventId) = await _fanmeterPlugin.execute(
        notificationData,
        null
    );
    print("[FANMETER - EXECUTE]: code=$code, eventId=$eventId");
}

/// Handle push tap by the user.
void _handlePushTap(RemoteMessage message) async {
    Map<String, String> notificationData = message.data.map((key, value) => MapEntry(key, value.toString()));
    final (code, eventId) = await _fanmeterPlugin.execute(
        notificationData,
        null
    );
    print("[FANMETER - EXECUTE]: code=$code, eventId=$eventId");
    // `execute` does not open any view — when an eventId is returned, your app
    // is responsible for opening the screen of that event.
    if(eventId != null) {
        openFanzoneForEvent(eventId);
    }
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
/// NOTE: Fanmeter pushes carry both a notification and a data payload - the SDK only needs `message.data`.
void _firebaseMessagingForegroundHandler(RemoteMessage message) async {
    Map<String, String> notificationData = message.data.map((key, value) => MapEntry(key, value.toString()));
    final result = await _fanmeterPlugin.launchFanmeterNotification(
        notificationData,
        null
    );
    print("[FANMETER - LAUNCHFANMETERNOTIFICATION]: $result");
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

On **Android**, the notification launched by `launchFanmeterNotification` above is a *local* notification — tapping it is a separate event from the `onMessageOpenedApp`/`getInitialMessage` handling above (which only fires for pushes delivered directly by FCM). Use `getInitialNotificationClick` and `onNotificationClick` to detect that tap and feed its extras into `execute`, the same way as a tapped push notification. This is not needed on iOS, where the SDK handles the tap natively:

```dart
Future<void> setupInteractedMessage() async {
    // ... FCM handling from above ...

    // Handle a tap on a Fanmeter *local* notification (Android only).
    final initialNotificationClick = await _fanmeterPlugin.getInitialNotificationClick();
    if(initialNotificationClick != null) {
        _handleNotificationClick(initialNotificationClick);
    }
    _fanmeterPlugin.onNotificationClick.listen(_handleNotificationClick);
}

void _handleNotificationClick(Map<String, String> notificationData) async {
    final (code, eventId) = await _fanmeterPlugin.execute(notificationData, null);
    print("[FANMETER - EXECUTE]: code=$code, eventId=$eventId");
    if(eventId != null) {
        openFanzoneForEvent(eventId);
    }
}
```

Where:

+ notificationData, is the remote data received with the notification;
+ NOTIFICATION_CLASS_RESPONSE, the name of the class that is being instantiated when the user clicks the notification - example: "com.company.activities.SearchActivity" (null opens the FOTM's default web view).

The possible `code` values are listed in the [Return codes](#return-codes).

## Plug-&-Play UI

### 5. Add the Fanzone screen

Fanzone is Pluggable's web-based activation interface — it is the view you present to your fans. The Fanmeter plugin ships a built-in JavaScript bridge so the Fanzone web app can call native SDK methods directly, loaded with the auth token returned by `initialize` (as an `Authorization: Bearer` header). There are two URLs to open, depending on how the user got there:

+ `https://data.fanofthematch.ai/fanzone/` - the **Fanzone home**, opened from your app itself (a button, a banner, a navbar item);
+ `https://data.fanofthematch.ai/fanzone/fotm?eventId=EVENT_ID` - the **Fan of the Match activation for one event**, opened from a tapped Fanmeter notification, using the event id that `execute` returned.

To use it, add `webview_flutter` to your `pubspec.yaml`:

```yaml
dependencies:
  webview_flutter: ^4.10.0
```

Then import `FanmeterWebInterface` and call `makeController()`, which returns a pre-configured `WebViewController` with the Fanmeter bridge already set up, in an `openFanzone` helper used both from your app (step 6) and from a tapped notification (step 4):

```dart
import 'package:fanmeter/fanmeter_web_interface.dart';
import 'package:webview_flutter/webview_flutter.dart';

// ...

// Global navigator key so the Fanzone view can be pushed from anywhere
// (e.g. from a push notification handler). Wire it into your MaterialApp:
// MaterialApp(navigatorKey: navigatorKey, ...).
final navigatorKey = GlobalKey<NavigatorState>();

const FANZONE_URL = 'https://data.fanofthematch.ai/fanzone/';

/// Opens the Fanzone view with a given URL, passing the auth token obtained
/// from `initialize` as an `Authorization: Bearer` header. `authToken` is the
/// shared variable set by the `initialize` call (step 3).
Future<void> openFanzone(String url) async {
    // 1. Create a pre-configured WebViewController with the Fanmeter bridge,
    //    closing the pushed route when the web layer requests it.
    //    Awaited so the native bridge is installed before the page loads.
    final controller = await FanmeterWebInterface.makeController(
        onCloseRequested: () => navigatorKey.currentState?.pop(),
    );
    // 2. Load the Fanzone URL.
    controller.loadRequest(
        Uri.parse(url),
        headers: {'Authorization': 'Bearer ${authToken ?? ''}'},
    );
    // 3. Present the WebView in your widget tree, wrapped with FanmeterBackHandler.
    navigatorKey.currentState?.push(
        MaterialPageRoute(
            builder: (_) => FanmeterBackHandler(
                controller: controller,
                child: Scaffold(
                    body: SafeArea(child: WebViewWidget(controller: controller)),
                ),
            ),
        ),
    );
}

// From a button/banner/navbar: the Fanzone home.
Future<void> openFanzoneHome() => openFanzone(FANZONE_URL);
// From a tapped notification: the FOTM activation of the event execute returned.
Future<void> openFanzoneForEvent(int eventId) => openFanzone('${FANZONE_URL}fotm?eventId=$eventId');
```

Rules to keep in mind:

+ `makeController()` is asynchronous — `await` it **before** calling `loadRequest`, otherwise the bridge is not yet available to the page being loaded;
+ Do not call `enableZoom` on the returned controller: it resets the WebView's user scripts, which would remove the bridge;
+ Do not call `setNavigationDelegate` on it either — on Android that replaces the native `WebViewClient`, and with it the protection that keeps your app process alive (and the Fanmeter service running) when the system reclaims the WebView's renderer. Pass the callbacks to `makeController` instead: it accepts `onPageStarted`, `onPageFinished`, `onProgress`, `onNavigationRequest`, `onUrlChange`, `onWebResourceError`, `onHttpError` and `onHttpAuthRequest`, which it forwards to the delegate for you;
+ **Do not render a native header or close button around the WebView**: the Fanzone has its own, which calls `onCloseRequested` - wire it to pop the same route (as above);
+ Wrap the WebView with `FanmeterBackHandler`, so the system back steps back through the Fanzone's own navigation first, instead of closing the whole screen (Android hardware/predictive back and iOS edge-swipe-to-pop are both covered).

#### Showing a loading indicator

Since `makeController()` is asynchronous, awaiting it *before* pushing the route leaves the user on the previous screen, with no feedback, until the WebView is ready. Push the route first and create the controller inside it, using the callbacks above to drive the indicator — `onProgress` for the load progress, `onPageFinished` to hide it, and `onWebResourceError` so a failed load does not leave it up forever:

```dart
Future<void> _openFanzone() async {
    final controller = await FanmeterWebInterface.makeController(
        onCloseRequested: () {
            if(mounted) Navigator.of(context).pop();
        },
        onPageStarted: (_) => _setLoading(true),
        onProgress: (progress) {
            if(mounted) setState(() => _progress = progress);
        },
        onPageFinished: (_) => _setLoading(false),
        onWebResourceError: (error) {
            if(error.isForMainFrame ?? false) _setLoading(false);
        },
    );
    if(!mounted) return;
    // Matches the screen's own background, so the WebView does not flash a
    // different colour before the page paints.
    await controller.setBackgroundColor(Theme.of(context).scaffoldBackgroundColor);
    controller.loadRequest(
        Uri.parse(url),
        headers: {'Authorization': 'Bearer ${authToken ?? ''}'},
    );
    if(!mounted) return;
    setState(() => _controller = controller);
}
```

The `WebViewWidget` is then placed in a `Stack`, only once the controller exists, with the indicator on top while `_loading` is true.

### 6. Open the Fanzone from your app

From a button, a banner or a navbar item, open the Fanzone home:

```dart
openFanzoneHome();
```

## Build your own UI

### 5. Get the event

The `getEventData()` method can be used to obtain the full data of a particular Fanmeter event, in a dictionary, including the defined rewards and leaderboard classifications, if existing. If null, or not provided, this returns the closest event to date:

```dart
/// Event name to show (nullable)
String? EVENT_NAME = 'Round 1 2026-2027';

// After initialized, you will be able to use the SDK methods.
final result = await _fanmeterPlugin.getEventData(EVENT_NAME);
print("[FANMETER - GETEVENTDATA - $EVENT_NAME]: $result");

// OR.

// After initialized, you will be able to use the SDK methods.
final result = await _fanmeterPlugin.getEventData();
print("[FANMETER - GETEVENTDATA]: $result");
```

### 6. Build your event screen

In this integration, your app is responsible for the screen where the user participates in the event - it is the one you open from your app, and from a tapped notification (step 4). It should:

+ Show the event, with the data obtained in the previous step;
+ Offer a button for the user to join and leave the event, calling `startService` and `stopService`;
+ Show the current state of that button, with `isServiceRunning`;
+ Tell the user what went wrong whenever `startService` returns an error code (see the [Return codes](#return-codes)).

The `startService` method is used to start the Fanmeter service that enables Fanmeter for your client's device during a particular event. This method accepts the eventId obtained in the previous step and should be called as follows (associated, for example, to a particular button):

```dart
// You can obtain the eventId of an event by calling the getEventData method.
final result = await _fanmeterPlugin.startService(
    EVENT_ID,
    null
);
print("[FANMETER - STARTSERVICE]: $result");
```

Where:

+ EVENT_ID, the id of the event the user will participate when the start service is called;
+ NOTIFICATION_CLASS_RESPONSE, the name of the class that is being instantiated when the user clicks the notification - example: "com.company.activities.SearchActivity" (null opens the app's default view).

The `stopService` method is used to stop the Fanmeter service (can be toggled with the previous method). Even if the user does not explicitly stop the service, it will automatically stop as soon as the event finishes. This returns 1, if success, otherwise an error code.

```dart
final result = await _fanmeterPlugin.stopService();
print("[FANMETER - STOPSERVICE]: $result");
```

Finally, the `isServiceRunning` method is used to check the current status of the Fanmeter service. This returns a `(bool isRunning, int? eventId)` record, where `eventId` identifies the event the service is currently running for.

```dart
final (isRunning, eventId) = await _fanmeterPlugin.isServiceRunning();
print("[FANMETER - ISSERVICERUNNING]: isRunning=$isRunning, eventId=$eventId");
```

### 7. Listen to the user's participation (optional)

You can subscribe to a listener that will notify you whenever there is a change in the user's participation status. For example, if the user leaves the venue of a geo-restricted event, the listener will alert you to this change. This can be particularly useful for updating the status of banners or buttons within your app.

Two methods are available: `subscribeUserParticipationListener` and `unsubscribeUserParticipationListener`. The first method subscribes the listener to a specific event id, and you can subscribe to multiple events. The second method unsubscribes the listener (if no event id is provided, it will unsubscribe all active listeners) and always returns 1 (Success), as it forces the stop of any or all running listeners.

```dart
// Subscribing a listener.
await _fanmeterPlugin.subscribeUserParticipationListener(EVENT_ID, ((value) {
    print("[FANMETER - SUBSCRIBEUSERPARTICIPATION] $EVENT_ID: $value");
    // Update the status of your button/banner.
}));
```

```dart
// Unsubscribing a listener. If no event id is set, it cancels all listeners.
final result = await _fanmeterPlugin.unsubscribeUserParticipationListener();
print("[FANMETER - UNSUBSCRIBEUSERPARTICIPATION]: $result");
```

The possible `value` states are listed in the [Participation states](#participation-states).

## Reference

### Methods

| Method | What it does | Returns |
| --- | --- | --- |
| `initialize` | Initializes the SDK. Required before any other method. | The auth token, or null. |
| `execute` | Processes a tapped Fanmeter notification. | A `(code, eventId)` record. |
| `launchFanmeterNotification` | Launches a local Fanmeter notification, when the Android app is in the foreground. | A return code. |
| `getInitialNotificationClick` / `onNotificationClick` | Detect a tap on a Fanmeter local notification (Android only). | The notification data. |
| `getEventData` | Gets an event's data (the closest event, if no title is given). | A dictionary. |
| `startService` | Starts the Fanmeter service for an event. | A return code. |
| `stopService` | Stops the Fanmeter service. | 1, if success, otherwise an error code. |
| `isServiceRunning` | Checks the current status of the Fanmeter service. | An `(isRunning, eventId)` record. |
| `subscribeUserParticipationListener` | Notifies every change in the user's participation in an event. | A participation state. |
| `unsubscribeUserParticipationListener` | Cancels the listener of an event (or all of them). | Always 1. |
| `FanmeterWebInterface.makeController` | Creates a `WebViewController` with the Fanmeter bridge installed. | The `WebViewController`. |
| `FanmeterBackHandler` | Steps the system back through the Fanzone's own navigation. | - |

### Return codes

The `execute` and `startService` methods return the following `code` values:

+ 1, success;
+ -80, no GPS/PUSH permissions;
+ -81, GPS disabled;
+ -82, invalid event coordinates;
+ -89, SDK not initialized;
+ -91, invalid notification data (`execute` only);
+ -92, invalid company license;
+ -93, invalid event;
+ -94, event not happening now;
+ -95, invalid external user data;
+ -96, failed to get event data;
+ -97, failed to start the Fanmeter service;
+ -98, another Fanmeter service is already running.

### Participation states

The `subscribeUserParticipationListener` listener can return the following status codes:

+ 0: User is not participating;
+ 1: Validating the user participation (validation may take up to 30 seconds);
+ 2: User is participating;
+ 3: User has GPS disabled;
+ 4: User is out of venue;
+ 5: Unknown GPS coordinates.

For more info visit [our website](https://fanofthematch.ai/) or give us feedback to <info@pluggableai.xyz>.
