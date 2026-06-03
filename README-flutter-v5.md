# Pluggable's Fanmeter Plugin for Flutter apps

The Fanmeter plugin is the one responsible for enabling Fanmeter in any mobile application. It is a simple plug-&-play plugin that makes available a set of methods that can be used to allow users to participate in activations such as the FAN OF THE MATCH or the SUPER FAN.

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

Note that, for Android only, Fanmeter is required to use a **Data Sync foreground service** to communicate non-intrusive sensor data with Pluggable's server. Since API 34 of Android, when publishing an app that uses such services, you are required to fill a **Policy Declaration** where you must justify the need for such services. In Fanmeter's case, when filling such policy, you will be asked the question "*What tasks require your app to use the FOREGROUND_SERVICE_DATA_SYNC permission?*". There, you should:

1. Select the option **OTHER** in **Other tasks**;

2. Provide the following video link:

    ```txt
    https://youtu.be/w4d7Pgksok0
    ```

3. Provide the following description:

    ```txt
    Fanmeter is an Android SDK, incorporated in this app, that uses a Data Sync foreground service to communicate non-intrusive sensor data with Pluggable servers, which are used to quantify the engagement of the user in real-time. The foreground service must start as soon as the user opts to participate in the event (so that it can collect and communicate data) and must keep running until the user himself decides to terminate his/her participation.
    
### Push Notifications

If you want a fully **plug-&-play experience** and wish to receive push notifications (if not receiving yet) from Fanmeter in your app, you should integrate `Firebase Cloud Messaging`. A complete, step-by-step tutorial is available in [Firebase's official documentation](https://firebase.google.com/docs/cloud-messaging). The steps are as follows:

1. First install [FlutterFire](https://firebase.flutter.dev/docs/overview/), a tool which automatically configures your app to use firebase;

2. Then, install [firebase_messaging](https://firebase.flutter.dev/docs/messaging/overview/), a cross-platform messaging solution;

3. Then, to start using the Cloud Messaging package within your project follow these [steps](https://firebase.flutter.dev/docs/messaging/usage);

4. Integrating the Cloud Messaging plugin on iOS requires additional setup before your devices receive messages. The full steps are available in the [official docs](https://firebase.google.com/docs/cloud-messaging/flutter/get-started#ios-setup-requirements), but the following prerequisites are required to be able to enable messaging:
    + You must have an active Apple Developer Account;
    + You must have a physical iOS device to receive messages.

5. Also for iOS:
    + If you are using Swift, in your `AppDelegate.swift` make sure you have added this code:

    ```swift
    import Firebase
    
    FirebaseApp.configure() //add this before the code below
    GeneratedPluginRegistrant.register(with: self)
    ```

    + If you're using flutter with Objective-C, add this code to your `appdelegate.m` file.

    ```objective-c
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

```bash
flutter pub add fanmeter
```

Or run `flutter pub upgrade` to update to the latest compatible versions of all the dependencies listed in the **pubspec.yaml** file.

For **Android**, to customize the used notification icon, just add the desired icon in the Android's drawable folder and name it **ic_push_app_icon**. Otherwise, a default icon, not related to your app, will be used.

### Fanzone Web View
Fanzone is Pluggable's web-based activation interface. The Fanmeter plugin ships a built-in JavaScript bridge so the Fanzone web app can call native SDK methods directly, without any platform-specific logic on the JavaScript side. The app retains full control over the WebView — the plugin only configures the bridge.
To use it, add `webview_flutter` to your `pubspec.yaml`:
```yaml
dependencies:
  webview_flutter: ^4.0.0
```

### Initialize Method

The `initialize` method is mandatory for both plug-&-play. It is responsible for initializing the SDK and must be invoked before using any other method (a possible approach is to invoke it in the app's main activity). It returns an auth token on success, `"Invalid parameters"` if the provided map is invalid, or `null` otherwise.

```dart
import 'package:fanmeter/fanmeter.dart';

// ...

/// The name of your company in Fanmeter
String YOUR_COMPANY_NAME = 'companyName'
/// Your company's license key
String YOUR_COMPANY_KEY = 'AAAA-BBBB-CCCC-DDDD'
/// The user identifier in your db (can be the username, the uuid, ...)
String USER_ID = 'externalUserId'
/// The individual smartphone identifier (allows for the same account in different devices)
String TOKEN_ID = 'externalTokenId'
/// The username of the user. Nullable
String? USERNAME = 'username'
/// The user's email. Nullable
String? USER_EMAIL = 'externalUserEmail'
/// The FCM token id of the user. Nullable
String? FCM_TOKEN = 'fcm_token'
/// Enables additional logging
bool? FANMETER_LOG = false;

final _fanmeterPlugin = Fanmeter();

// When the Fanmeter SDK is initialized, it initializes all company and user data.
final authToken = await _fanmeterPlugin.initialize(
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
    log: false,
);
print("[FANMETER - INITIALIZE]: $authToken");
```

### FanZone WebView 

The FanZone endpoint is responsible for displaying the FanZone web experience inside your application. Before loading the FanZone, the SDK must be initialized using the `initialize` method in order to obtain a valid authentication token. This token must then be included in the request used to load the FanZone page.
The FanZone can be embedded in your application using a WebView. Fanmeter provides a pre-configured `WebViewController` through `FanmeterWebInterface.makeController()`, which automatically sets up the communication bridge required between the web content and the native application.

```dart
import 'package:fanmeter/fanmeter_web_interface.dart';
import 'package:webview_flutter/webview_flutter.dart';
// ...
// 1. Initialize Fanmeter and retain the auth token.
final authToken = await _fanmeterPlugin.initialize(
    initializeDataMap: { /* ... */ },
);

// 2. Build the Fanzone URL.
//    NOTE: the auth token obtained from initialize must be passed to the
//    Fanzone URL. The exact mechanism (query parameter or HTTP header) is
//    being defined — check with Pluggable for the latest guidance.
final auth_request_headers = {
    'Authorization': 'Bearer $authToken'
};
final fanzoneUri = Uri.parse('https://...');

// 3. Create a pre-configured WebViewController with the Fanmeter bridge.
final controller = FanmeterWebInterface.makeController();

// 4. Load the Fanzone URL.
controller.loadRequest(fanzoneUri, headers: auth_request_headers);

// 5. Present the WebView in your widget tree.
WebViewWidget(controller: controller)
```

## Additional info

Other important methods are to **request user permission** to be able to **send notifications** and **request GPS permission**. Also, **get the user token and listen for changes** to get the user FCM_TOKEN and update it when it changes.

```dart
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