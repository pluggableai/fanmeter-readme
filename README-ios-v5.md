# Pluggable's Fanmeter SDK for iOS apps

The Fanmeter SDK is the one responsible for enabling Fanmeter in any mobile application. It is a simple plug-&-play SDK that makes available a set of methods that can be used to allow users to participate in activations such as the FAN OF THE MATCH or the SUPER FAN.

## Table of Contents

+ [Overview](#overview)
+ [Requirements](#requirements)
  + [Minimum versions](#minimum-versions)
  + [Info.plist and capabilities](#infoplist-and-capabilities)
+ [Getting Started](#getting-started)
  + [1. Install the SDK](#1-install-the-sdk)
  + [2. Set up push notifications](#2-set-up-push-notifications)
  + [3. Initialize the SDK](#3-initialize-the-sdk)
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

This SDK requires a minimum deployment target of **iOS 13.0**. For full compatibility, be sure to use the recommended versions: **XCODE=15**, **SWIFT=5.9**, and **COCOAPODS=1.14.2**.

### Info.plist and capabilities

Add the **Background Modes** capability and enable **Location Updates**. Additionally, open your `Info.plist` file and add the following code at the bottom of the file:

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

Location permission is necessary for fans to participate in geo-restricted events, but you do not need to request it yourself: whenever a service is started, the SDK asks the user for the location permission and, if only the approximate location was granted, for the precise one (using the `PreciseLocationRequired` key above). Push notification permission, on the other hand, must be requested by your app (see [step 2](#2-set-up-push-notifications)).

## Getting Started

### 1. Install the SDK

There are two approaches to include the SDK in your app:

1. Using [CocoaPods](https://cocoapods.org/pods/fanmeter_sdk_ios), by adding the SDK to your `Podfile` (check which version is the latest) and running `pod install`:

    ```ruby
    pod 'fanmeter_sdk_ios', '~> 5.0.1'
    ```

2. Using the XCFramework file containing the Fanmeter SDK (the link is made available by your account manager). In Xcode, go to your project main page, select the desired target, and in the **General** tab scroll down to **Frameworks, Libraries, and Embedded Content**. There, add the framework file, selecting the option **Embed & Sign**.

**NOTE**: If the build fails with a sandbox error, set the **ENABLE_USER_SCRIPT_SANDBOXING** build setting of your project to **No**.

You are now able to properly use the SDK:

```swift
import fanmeter_sdk_ios
```

Objective-C apps can use `EntryPointBridge`, which exposes the same methods as `EntryPoint`.

### 2. Set up push notifications

Fanmeter notifications reach your app through [Firebase Cloud Messaging (FCM)](https://firebase.google.com/docs/cloud-messaging/ios/get-started) - if not done already, integrate it and add the **Push Notifications** capability to your target. Then, as soon as the app finishes launching, set the `UNUserNotificationCenter` delegate (through which the notification taps reach your app), ask for the push notification permission, and register the app for remote notifications (`AppDelegate.swift`):

```swift
import UIKit
import FirebaseCore
import FirebaseMessaging
import UserNotifications
import fanmeter_sdk_ios

class AppDelegate: UIResponder, UIApplicationDelegate {

    func application(_ application: UIApplication, didFinishLaunchingWithOptions launchOptions: [UIApplication.LaunchOptionsKey : Any]? = nil) -> Bool {
        FirebaseApp.configure()
        Messaging.messaging().delegate = self
        // Set before launching ends, so a tap that launched the app is also delivered.
        UNUserNotificationCenter.current().delegate = self
        // Ask for the push notification permission.
        UNUserNotificationCenter.current().requestAuthorization(options: [.alert, .badge, .sound]) { _, _ in }
        application.registerForRemoteNotifications()
        // Initialize the SDK as early as possible (step 3).
        // EntryPoint.initialize(...)
        return true
    }
}
```

If you're using **SwiftUI**, attach this app delegate to your `App` struct via `@UIApplicationDelegateAdaptor(AppDelegate.self)`.

You are also **required to subscribe the user to a FCM topic** so that the user can receive Fanmeter notifications. This can be done in the FCM delegate, which also receives the user's FCM token (to pass as `fcmToken` in step 3) - it fires at each app startup and whenever a new token is generated:

```swift
extension AppDelegate: MessagingDelegate {

    func messaging(_ messaging: Messaging, didReceiveRegistrationToken fcmToken: String?) {
        NSLog("TOKEN is: \(String(describing: fcmToken))")
        // Subscribe to a topic.
        Messaging.messaging().subscribe(toTopic: "football_senior") { _ in
            NSLog("Subscribed to topic: football_senior!")
        }
    }
}
```

Fanmeter notifications may carry an image. To display it, add a **Notification Service Extension**, as described in [Firebase's documentation](https://firebase.google.com/docs/cloud-messaging/ios/send-image).

**NOTE**: FCM via APNs **does not work on iOS Simulators**. To receive messages & notifications a real device is required.

### 3. Initialize the SDK

The `initialize` method is responsible for initializing the SDK and must be invoked before using any other method (a possible approach is to invoke it as soon as the app finishes launching). It returns, in its callback, an auth token on success, or nil otherwise.

```swift
import fanmeter_sdk_ios

let YOUR_COMPANY_NAME: String = "companyName"
let YOUR_COMPANY_KEY: String = "AAAA-BBBB-CCCC-DDDD"
let USER_ID: String = "externalUserId"
let TOKEN_ID: String = "externalTokenId"
let USERNAME: String = "username"
let USER_EMAIL: String = "externalUserEmail"
let FCM_TOKEN: String = "fcm_token"
let FANMETER_LOG: Bool = false

let initializeDataMap: [String: String] = [
    "companyName": YOUR_COMPANY_NAME,
    "companyKey": YOUR_COMPANY_KEY,
    "externalUserId": USER_ID,
    "externalTokenId": TOKEN_ID,
    // Optional:
    "username": USERNAME,
    "externalUserEmail": USER_EMAIL,
    "fcmToken": FCM_TOKEN
]

// When the Fanmeter SDK is initialized, it initializes all company and user data.
EntryPoint.initialize(
    initializeDataMap: initializeDataMap,
    log: FANMETER_LOG
) { token in
    NSLog("EntryPoint.initialize: \(String(describing: token))")
    // Plug-&-Play UI: keep the auth token where the Fanzone view can reach it, since the
    // Fanzone (step 5) needs it to load its page.
    Fanzone.shared.authToken = token
}
```

Where - required keys:
  * companyName, the name of your company in Fanmeter;
  * companyKey, your company's license key;
  * externalUserId, the user identifier in your db (can be the username, the uuid, ...);
  * externalTokenId, the individual smartphone identifier (allows for the same account in different devices).

Optional keys (the map does not take nil values, so leave out the ones you do not have):
  * username, the user's display name;
  * externalUserEmail, the user's email;
  * fcmToken, the FCM token id of the user.

Additional parameter:
  * FANMETER_LOG, enables additional logging.

### 4. Handle Fanmeter notifications

The `execute` method is called when a Fanmeter notification is tapped - whether the app was in the foreground, in the background, or terminated. It does not open any view itself - its callback returns a `(code, eventId)` pair and, when `eventId` is not nil, your app is responsible for opening the screen of that event: the Fanzone, in the Plug-&-Play UI, or your own event screen, in the Build your own UI. Also, since iOS does not display notifications that arrive while the app is in the foreground, implement `willPresent` so the user still sees (and can tap) them:

```swift
extension AppDelegate: UNUserNotificationCenterDelegate {

    // Called when the user taps a notification.
    func userNotificationCenter(_ center: UNUserNotificationCenter, didReceive response: UNNotificationResponse, withCompletionHandler completionHandler: @escaping () -> Void) {
        let userInfo = response.notification.request.content.userInfo
        EntryPoint.execute(notificationData: userInfo) { code, eventId in
            NSLog("EntryPoint.execute: \(String(describing: code)), \(String(describing: eventId))")
            // `execute` does not open any view - when an eventId is returned, your app
            // is responsible for opening the screen of that event.
            if let eventId = eventId {
                // Plug-&-Play UI (step 5). In the Build your own UI, open your own event screen instead.
                Fanzone.shared.openEvent(eventId)
            }
        }
        completionHandler()
    }

    // Called when a notification arrives while the app is in the foreground.
    func userNotificationCenter(_ center: UNUserNotificationCenter, willPresent notification: UNNotification, withCompletionHandler completionHandler: @escaping (UNNotificationPresentationOptions) -> Void) {
        if #available(iOS 14.0, *) {
            completionHandler([.banner, .list, .sound])
        } else {
            completionHandler([.alert, .sound])
        }
    }
}
```

Where:
  * notificationData (`userInfo`), is the remote data received with the notification.

The possible `code` values are listed in the [Return codes](#return-codes).

## Plug-&-Play UI

### 5. Add the Fanzone screen

Fanzone is Pluggable's web-based activation interface - it is the view you present to your fans. Show it in a `WKWebView` with the Fanmeter JavaScript bridge installed (`FanmeterWebInterface`), loading it with the auth token returned by `initialize` (as an `Authorization: Bearer` header). There are two URLs to open, depending on how the user got there:

  * `https://data.fanofthematch.ai/fanzone/` - the **Fanzone home**, opened from your app itself (a button, a banner, a navbar item);
  * `https://data.fanofthematch.ai/fanzone/fotm?eventId=EVENT_ID` - the **Fan of the Match activation for one event**, opened from a tapped Fanmeter notification, using the event id that `execute` returned.

```swift
import SwiftUI
import WebKit
import fanmeter_sdk_ios

struct FanzoneView: UIViewRepresentable {
    let url: URL
    let authToken: String
    let onClose: () -> Void

    func makeUIView(context: Context) -> WKWebView {
        let webView = WKWebView(frame: .zero, configuration: WKWebViewConfiguration())
        // 1. Install the Fanmeter bridge, exposed to the page as window.FanmeterBridge.
        FanmeterWebInterface.attach(to: webView, onClose: onClose)
        // 2. Load the Fanzone URL, passing the token returned by initialize.
        var request = URLRequest(url: url)
        request.setValue("Bearer \(authToken)", forHTTPHeaderField: "Authorization")
        webView.load(request)
        return webView
    }

    func updateUIView(_ webView: WKWebView, context: Context) {}
}
```

Then, keep what the Fanzone shows in an object that both your views and the `AppDelegate` can reach (it also prevents multiple openings of the Fanzone view):

```swift
final class Fanzone: ObservableObject {

    static let shared = Fanzone()
    static let FANZONE_URL = "https://data.fanofthematch.ai/fanzone/"

    // The token returned by initialize.
    var authToken: String?
    @Published var isPresented = false
    private(set) var url = URL(string: Fanzone.FANZONE_URL)!

    // From a button/banner/navbar: the Fanzone home.
    func openHome() {
        open(Fanzone.FANZONE_URL)
    }

    // From a tapped notification: the FOTM activation of the event execute returned.
    func openEvent(_ eventId: Int) {
        open("\(Fanzone.FANZONE_URL)fotm?eventId=\(eventId)")
    }

    private func open(_ address: String) {
        DispatchQueue.main.async {
            // Prevent multiple openings: only present it when no previous one is on screen.
            if(self.isPresented) {
                return
            }
            if(self.authToken == nil) {
                NSLog("Cannot open the Fanzone without an auth token - initialize first.")
                return
            }
            self.url = URL(string: address)!
            self.isPresented = true
        }
    }
}
```

And present it from your root view:

```swift
struct RootView: View {
    @ObservedObject private var fanzone = Fanzone.shared

    var body: some View {
        ContentView()
            .fullScreenCover(isPresented: $fanzone.isPresented) {
                FanzoneView(url: fanzone.url, authToken: fanzone.authToken ?? "") {
                    fanzone.isPresented = false
                }
            }
    }
}
```

Two rules to keep in mind:

  * Call `attach(to:onClose:)` **before** loading the page, otherwise the page is left without the bridge;
  * **Do not render a native header or close button around the WebView**: the Fanzone has its own, which calls `onClose` (in SwiftUI, always pass it, wired to the state that presents the Fanzone, as above).

If your app uses UIKit, `FanmeterWebInterface.makeWebView()` returns a `WKWebView` with the bridge already installed - load the Fanzone request on it as above. Presented modally, it closes itself without an `onClose`.

### 6. Open the Fanzone from your app

From a button, a banner or a navbar item, open the Fanzone home:

```swift
Fanzone.shared.openHome()
```

## Build your own UI

### 5. Get the event

The `getEventData` method is used to obtain the full data of a particular Fanmeter event, in a dictionary, including the defined rewards and leaderboard classifications, if existing. If nil, or not provided, this returns the closest event to date:

```swift
let EVENT_NAME: String = "Round 1 2025-2026"

// After initialized, you will be able to use the SDK methods.
EntryPoint.getEventData(eventTitle: EVENT_NAME) { eventData in
    let eventId = eventData["eventId"] as? Int ?? -99
    NSLog("EntryPoint.getEventData(\(EVENT_NAME)): \(eventId)")
}

// OR.

// After initialized, you will be able to use the SDK methods.
EntryPoint.getEventData() { eventData in
    let eventId = eventData["eventId"] as? Int ?? -99
    NSLog("EntryPoint.getEventData(): \(eventId)")
}
```

### 6. Build your event screen

In this integration, your app is responsible for the screen where the user participates in the event - it is the one you open from your app, and from a notification tap (step 4). It should:

  * Show the event, with the data obtained in the previous step;
  * Offer a button for the user to join and leave the event, calling `startService` and `stopService`;
  * Show the current state of that button, with `isServiceRunning`;
  * Tell the user what went wrong whenever `startService` returns an error code (see the [Return codes](#return-codes)).

The `startService` method is used to start the Fanmeter service that enables Fanmeter for your client's device during a particular event. This method accepts the eventId obtained in the previous step and should be called as follows (associated, for example, to a particular button). When needed, the SDK asks the user for the location permission and for the precise location itself:

```swift
// e.g. on a button click.
EntryPoint.startService(eventId: EVENT_ID) { result in
    NSLog("EntryPoint.startService: \(result)")
}
```

The `stopService` method is used to stop the Fanmeter service (can be toggled with the previous method). Even if the user does not explicitly stop the service, it will automatically stop as soon as the event finishes. This returns 1, if success, otherwise an error code.

```swift
EntryPoint.stopService() { result in
    NSLog("EntryPoint.stopService: \(result)")
}
```

Finally, the `isServiceRunning` method is used to check the current status of the Fanmeter service. This returns `isRunning`, 1 if the service is running (0 otherwise), and `eventId`, the id of the event being tracked, which is only set while it is running.

```swift
EntryPoint.isServiceRunning() { isRunning, eventId in
    NSLog("EntryPoint.isServiceRunning: \(isRunning), \(String(describing: eventId))")
}
```

### 7. Listen to the user's participation (optional)

You can subscribe to a listener that will notify you whenever there is a change in the user's participation status. For example, if the user leaves the venue of a geo-restricted event, the listener will alert you to this change. This can be particularly useful for updating the status of banners or buttons within your app.

Two methods are available: `subscribeUserParticipationListener` and `unsubscribeUserParticipationListener`. The first method subscribes the listener to a specific event id, and you can subscribe to multiple events. The second method unsubscribes the listener (if no event id is provided, it will unsubscribe all active listeners) and always returns 1 (Success), as it forces the stop of any or all running listeners.

```swift
// Subscribing a listener. The EVENT_ID (an Int) can be obtained with getEventData.
EntryPoint.subscribeUserParticipationListener(eventId: EVENT_ID) { state in
    NSLog("Event \(EVENT_ID) | Participation state changed: \(state)")
}

// OR.

// Unsubscribing a listener. If no event id is set, it cancels all listeners.
EntryPoint.unsubscribeUserParticipationListener() { result in
    NSLog("Unsubscribed with result: \(result)")
}
```

The possible `state` values are listed in the [Participation states](#participation-states).

## Reference

### Methods

| Method | What it does | Returns |
| --- | --- | --- |
| `EntryPoint.initialize` | Initializes the SDK. Required before any other method. | The auth token, or nil. |
| `EntryPoint.execute` | Processes a tapped Fanmeter notification. | A `(code, eventId)` pair. |
| `EntryPoint.getEventData` | Gets an event's data (the closest event, if no title is given). | A dictionary. |
| `EntryPoint.startService` | Starts the Fanmeter service for an event. | A return code. |
| `EntryPoint.stopService` | Stops the Fanmeter service. | 1, if success, otherwise an error code. |
| `EntryPoint.isServiceRunning` | Checks the current status of the Fanmeter service. | An `(isRunning, eventId)` pair. |
| `EntryPoint.subscribeUserParticipationListener` | Notifies every change in the user's participation in an event. | A participation state. |
| `EntryPoint.unsubscribeUserParticipationListener` | Cancels the listener of an event (or all of them). | Always 1. |
| `FanmeterWebInterface.attach(to:onClose:)` | Installs the Fanmeter bridge on your `WKWebView`. | The bridge (safe to ignore). |
| `FanmeterWebInterface.makeWebView()` | Creates a `WKWebView` with the Fanmeter bridge installed. | The `WKWebView`. |

### Return codes

The `execute` and `startService` methods return the following values:
  * 1, success;
  * -80, no GPS/PUSH permissions;
  * -81, GPS disabled;
  * -82, invalid event coordinates;
  * -85, restricted location permission (`startService` only);
  * -86, no precise location permission (`startService` only);
  * -87, unknown location permission status (`startService` only);
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
