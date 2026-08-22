# Pluggable's Fanmeter Plugin for Flutter apps

The Fanmeter plugin is the one responsible for enabling Fanmeter in any mobile application. It is a simple plug-&-play plugin that makes available a set of methods that can be used to allow users to participate in activations such as the FAN OF THE MATCH (FOTM) or the SUPER FAN.

## Table of Contents

+ [Plug-&-Play vs Build your own UI](#plug--play-vs-build-your-own-ui)
+ [Pre-Conditions](#pre-conditions)
  + [Compatibility](#compatibility)
  + [Meta-Data](#meta-data)
  + [Google Play Policy Declarations](#google-play-policy-declarations)
  + [Push Notifications](#push-notifications)
+ [Set up the Fanmeter Plugin in your Flutter App](#set-up-the-fanmeter-plugin-in-your-flutter-app)
  + [Install the Plugin](#install-the-plugin)
  + [Methods initialize and getEventData](#methods-initialize-and-geteventdata)
  + [Listeners for User Participation](#listeners-for-user-participation)
+ [Fanzone Web View](#fanzone-web-view)
+ [Plug-&-Play UI](#plug--play-ui)
+ [Build your own UI](#build-your-own-ui)
+ [Additional info](#additional-info)

## Plug-&-Play vs Build your own UI

There are two possible types of integration. One is plug-&-play and uses a default native UI that is launched to your users for them to participate in FOTM events - it is also able to launch and deliver push notifications as soon as the event starts and finishes, if you configure it. If, on the other hand, you wish to create your own FOTM UI, you will follow a "build your own UI" approach.

  1. **Plug-&-Play UI**: you want everything automated (including a default Fanmeter view). You should also integrate with FCM to handle received notifications that start the event (in summary, you'll need to use the *execute* and *launchFanmeterNotification* methods together with the embedded FOTM web view);

  2. **Build your own UI**: you want to handle the conditions yourself and develop your own UI (just call the *startService*, *stopService*, and *isServiceRunning* methods).

## Pre-Conditions

### Compatibility

For full compatibility, be sure to use the recommended versions: **XCODE=15**, **SWIFT=5.9**, and **COCOAPODS=1.14.2**.

### Meta-Data

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

### Google Play Policy Declarations

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

### Push Notifications

If you want a fully **plug-&-play experience** and wish to receive push notifications (if not receiving yet) from Fanmeter in your app, you should integrate `Firebase Cloud Messaging`. A complete, step-by-step tutorial is available in [Firebase's official documentation](https://firebase.google.com/docs/cloud-messaging).

**NOTE**: FCM via APNs **does not work on iOS Simulators**. To receive messages & notifications a real device is required. The same is recommended for Android.

## Set up the Fanmeter Plugin in your Flutter App

Few steps are required to be ready to use this SDK.

### Install the Plugin

After configuring your app to integrate with FCM, you are ready to use this plugin to properly engage with your fans! To install the plugin just run, in the root of the project:

```bash
flutter pub add fanmeter
```

Or run `flutter pub upgrade` to update to the latest compatible versions of all the dependencies listed in the **pubspec.yaml** file.

For **Android**, to customize the used notification icon, just add the desired icon in the Android's drawable folder and name it **ic_push_app_icon**. Otherwise, a default icon, not related to your app, will be used.

### Methods initialize and getEventData

The `initialize` method is mandatory for both plug-&-play and build your own UI scenarios. It is responsible for initializing the SDK and must be invoked before using any other method (a possible approach is to invoke it before launching the FOTM). It returns an auth token on success, `Invalid parameters` if the provided map is invalid, or `null` otherwise.

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

/// Keep the auth token at a scope both your init flow and the Fanzone helper
/// can reach (e.g. a top-level variable or a field in your app state/service),
/// since `openFanzoneView` (see below) needs it to load the Fanzone page.
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

### Listeners for User Participation

In both scenarios, you can subscribe to a listener that will notify you whenever there is a change in the user's participation status. For example, if the user leaves the venue of a geo-restricted event, the listener will alert you to this change. This can be particularly useful for updating the status of banners or buttons within your app.

Two methods are available: `subscribeUserParticipationListener` and `unsubscribeUserParticipationListener`. The first method subscribes the listener to a specific event id, and you can subscribe to multiple events. The second method unsubscribes the listener (if no event id is provided, it will unsubscribe all active listeners).

While the `unsubscribeUserParticipationListener` method always returns 1 (Success), as it forces the stop of any or all running listeners, the `subscribeUserParticipationListener` listener can return the following status codes:

+ 0: User is not participating;
+ 1: Validating the user participation (validation may take up to 30 seconds);
+ 2: User is participating;
+ 3: User has GPS disabled;
+ 4: User is out of venue;
+ 5: Unknown GPS coordinates.

These methods can be invoked as shown below:

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

## Fanzone Web View

Fanzone is Pluggable's web-based activation interface — it is the view you present to your fans (for example, the one you open after `execute` returns an event). The Fanmeter plugin ships a built-in JavaScript bridge so the Fanzone web app can call native SDK methods directly, without any platform-specific logic on the JavaScript side. The app retains full control over the WebView — the plugin only configures the bridge.

Before loading the Fanzone, the SDK must be initialized using the `initialize` method in order to obtain a valid authentication token. This token must then be included in the request used to load the Fanzone page (as an `Authorization: Bearer` header).

To use it, add `webview_flutter` to your `pubspec.yaml`:

```yaml
dependencies:
  webview_flutter: ^4.0.0
```

Then import `FanmeterWebInterface` and call `makeController()`, which returns a pre-configured `WebViewController` with the Fanmeter bridge already set up. A convenient way is to wrap this in an `openFanzoneView` helper that the plug-&-play flow (below) calls right after `execute` returns an event.

Your app keeps full ownership of the WebView — it loads the URL and places the `WebViewWidget` in its own tree — and the SDK shows its own localized GPS permission dialogs when they are needed, so you do not have to build them yourself.

Two things to keep in mind:

+ `makeController()` is asynchronous — `await` it **before** calling `loadRequest`, otherwise the bridge is not yet available to the page being loaded.
+ Do not call `enableZoom` on the returned controller: it resets the WebView's user scripts, which would remove the bridge.

Since the app owns the WebView, it's also responsible for the system back button/gesture: by default it closes the whole screen instead of stepping back through the Fanzone SPA's own navigation. Wrap it with `FanmeterBackHandler`, which steps back through the SPA's own navigation first (Android hardware/predictive back and iOS edge-swipe-to-pop are both covered). Likewise, the web layer can ask to close the screen itself (e.g. a back button in the Fanzone appbar, via `window.FanmeterBridge.closeWebView()`); `makeController` exposes this as the `onCloseRequested` callback, which you should wire to pop the same route:

```dart
import 'package:fanmeter/fanmeter_web_interface.dart';
import 'package:webview_flutter/webview_flutter.dart';

// ...

// Global navigator key so the Fanzone view can be pushed from anywhere
// (e.g. from a push notification handler). Wire it into your MaterialApp:
// MaterialApp(navigatorKey: navigatorKey, ...).
final navigatorKey = GlobalKey<NavigatorState>();

/// Opens the Fanzone view for a given event, passing the auth token obtained
/// from `initialize` as an `Authorization: Bearer` header. `authToken` is the
/// shared variable set by the `initialize` call above.
Future<void> openFanzoneView(int eventId) async {
    // 1. Create a pre-configured WebViewController with the Fanmeter bridge,
    //    closing the pushed route when the web layer requests it.
    //    Awaited so the native bridge is installed before the page loads.
    final controller = await FanmeterWebInterface.makeController(
        onCloseRequested: () => navigatorKey.currentState?.pop(),
    );
    // 2. Load the Fanzone URL for the event.
    controller.loadRequest(
        Uri.parse('https://data.fanofthematch.ai/fanzone/fotm?eventId=$eventId'),
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
```

## Plug-&-Play UI

After initializing the Plugin, you are now ready to start calling Fanmeter. In particular, if you want to automate the entire process through push notifications, this library exposes two methods, that must be called as demonstrated below. In particular:

+ `execute`, processes a tapped Fanmeter notification and returns the event the user should be directed to (your app then opens the Fanzone view for that event);
+ `launchFanmeterNotification`, launches a local Fanmeter notification to the user, which is required by Android when the app is in the foreground.

The `execute` method is called when a Fanmeter notification is tapped. It does not open any view itself — it returns a `(int? code, int? eventId)` record and, when `eventId` is not null, your app is responsible for opening the Fanzone view for that event (see the [Fanzone Web View](#fanzone-web-view) section). On the other hand, the `launchFanmeterNotification` method launches a local notification when the Android app is in a foreground state. An example is as follows, used in your *.dart* files as demonstrated in the next lines.

```dart
import 'package:fanmeter/fanmeter.dart';

// ...

// `openFanzoneView` is the helper defined in the "Fanzone Web View" section above.

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
    // is responsible for opening the Fanzone view for that event.
    if(eventId != null) {
        openFanzoneView(eventId);
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
/// NOTE: Fanmeter pushes are data-only, so the payload arrives in `message.data`.
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
        openFanzoneView(eventId);
    }
}
```

Where:

+ notificationData, is the remote data received with the notification;
+ NOTIFICATION_CLASS_RESPONSE, the name of the class that is being instantiated when the user clicks the notification - example: "com.company.activities.SearchActivity" (null opens the FOTM's default web view).

The `execute` method returns a `(int? code, int? eventId)` record — `code` is the status of the operation and `eventId` identifies the event whose Fanzone view your app should open (if any). These functions return the following `code` values:

+ 1, success;
+ -80, no GPS/PUSH permissions;
+ -81, GPS disabled;
+ -82, invalid event coordinates;
+ -89, SDK not initialized;
+ -91, invalid notification data;
+ -92, invalid company license;
+ -93, invalid event;
+ -94, event not happening now;
+ -95, invalid external user data;
+ -96, failed to get event data;
+ -97, failed to start the Fanmeter service;
+ -98, another Fanmeter service is already running.

You are also required to subscribe the user to a FCM topic so that the user can receive Fanmeter notifications. This can be done, for example, as follows:

```dart
// Subscribe to specific topic.
await FirebaseMessaging.instance.subscribeToTopic('football_senior');
```

## Build your own UI

If you want full control and implement your own UI, this library exposes three methods, that must be called as demonstrated below. In particular:

+ `startService`, starts the Fanmeter service that enables Fanmeter for your client's device during a particular event;
+ `stopService`, stops the Fanmeter service. The service will, still, terminate automatically as soon as the event ends. This returns 1, if success, otherwise an error code;
+ `isServiceRunning`, used to check the current status of the Fanmeter service. Returns a `(bool isRunning, int? eventId)` record, where `eventId` identifies the event the service is currently running for.

The `startService` method is used to start the Fanmeter service. This method accepts an eventId, that should be obtained using the `getEventData` method and should be called as follows (associated, for example, to a particular button):

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

This method returns the following values:

+ 1, success;
+ -80, no GPS/PUSH permissions;
+ -81, GPS disabled;
+ -82, invalid event coordinates;
+ -89, SDK not initialized;
+ -92, invalid company license;
+ -93, invalid event;
+ -94, event not happening now;
+ -95, invalid external user data;
+ -96, failed to get event data;
+ -97, failed to start the Fanmeter service;
+ -98, another Fanmeter service is already running.

The `stopService` method is used to stop the Fanmeter service (can be toggled with the previous method). Even if the user does not explicitly stop the service, it will automatically stop as soon as the event finishes. This returns 1, if success, otherwise an error code.

```dart
final result = await _fanmeterPlugin.stopService();
print("[FANMETER - STOPSERVICE]: $result");
```

Finally, the `isServiceRunning` method is used to check the current status of the Fanmeter service. This returns a `(bool isRunning, int? eventId)` record.

```dart
final (isRunning, eventId) = await _fanmeterPlugin.isServiceRunning();
print("[FANMETER - ISSERVICERUNNING]: isRunning=$isRunning, eventId=$eventId");
```

## Additional info

Other important methods are to **request user permission** to be able to **send notifications** and **request GPS permission**. Also, **get the user token and listen for changes** to get the user FCM_TOKEN and update it when it changes.

For more info visit [our website](https://fanofthematch.ai/) or give us feedback to <info@pluggableai.xyz>.
