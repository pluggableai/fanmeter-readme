# Pluggable's Fanmeter SDK for iOS apps

The Fanmeter SDK is the one responsible for enabling Fanmeter in any mobile application. It is a simple plug-&-play SDK that makes available a set of methods that can be used to allow users to participate in activations such as the FAN OF THE MATCH or the SUPER FAN.

## Table of Contents

+ [Plug-&-Play vs Build your own UI](#plug--play-vs-build-your-own-ui)
+ [Pre-Conditions](#pre-conditions)
  + [Compatibility](#compatibility)
  + [Meta-Data](#meta-data)
  + [Push Notifications](#push-notifications)
+ [Set up the Fanmeter SDK in your iOS App](#set-up-the-fanmeter-sdk-in-your-ios-app)
  + [Import the SDK](#import-the-sdk)
  + [Methods initialize and getEventData](#methods-initialize-and-geteventdata)
  + [Listeners for User Participation](#listeners-for-user-participation)
+ [Fanzone Web View](#fanzone-web-view)
+ [Plug-&-Play UI](#plug--play-ui)
+ [Build your own UI](#build-your-own-ui)
+ [Additional info](#additional-info)

## Plug-&-Play vs Build your own UI

There are two possible types of integration. One is plug-&-play and uses the Fanzone, Pluggable's web-based activation interface, that you present to your users for them to participate in Fanmeter events - it is also able to launch and deliver push notifications as soon as the event starts and finishes, if you configure it. If, on the other hand, you wish to create your own Fanmeter UI, you will follow a "build your own UI" approach.

  1. **Plug-&-Play UI**: you want everything automated (including the Fanzone view). You should also integrate with FCM to handle received notifications that start the event (in summary, you'll need to use the *execute* method, plus the [Fanzone Web View](#fanzone-web-view));

  2. **Build your own UI**: you want to handle the conditions yourself and develop your own UI (just call the *startService*, *stopService*, and *isServiceRunning* methods).

## Pre-Conditions

### Compatibility

This SDK requires a minimum deployment target of **iOS 13.0**. For full compatibility, be sure to use the recommended versions: **XCODE=15**, **SWIFT=5.9**, and **COCOAPODS=1.14.2**.

### Meta-Data

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

Location permission is necessary for fans to participate in geo-restricted events, but you do not need to request it yourself: whenever a service is started, the SDK asks the user for the location permission and, if only the approximate location was granted, for the precise one (using the `PreciseLocationRequired` key above). Push notification permission, on the other hand, must be requested by your app (see [Plug-&-Play UI](#plug--play-ui)).

### Push Notifications

If you want a **fully plug-&-play experience** and wish to receive push notifications (if not receiving yet) from Fanmeter in your app, you should integrate [Firebase Cloud Messaging (FCM)](https://firebase.google.com/docs/cloud-messaging/ios/get-started). A complete, step-by-step tutorial is available in [Firebase's official documentation](https://firebase.google.com/docs/cloud-messaging).

In Xcode, go to your target's **Signing & Capabilities** tab and add the **Push Notifications** capability (the **Remote notifications** background mode is already enabled by the `Info.plist` code above).

Fanmeter notifications may carry an image. To display it, add a **Notification Service Extension** to your project, following Apple's guide on [modifying content in newly delivered notifications](https://developer.apple.com/documentation/usernotifications/modifying_content_in_newly_delivered_notifications), and let FCM populate the notification content:

```swift
import UserNotifications
import FirebaseMessaging

class NotificationService: UNNotificationServiceExtension {

    var contentHandler: ((UNNotificationContent) -> Void)?
    var bestAttemptContent: UNMutableNotificationContent?

    override func didReceive(_ request: UNNotificationRequest, withContentHandler contentHandler: @escaping (UNNotificationContent) -> Void) {
        self.contentHandler = contentHandler
        bestAttemptContent = (request.content.mutableCopy() as? UNMutableNotificationContent)
        if let bestAttemptContent = bestAttemptContent {
            Messaging.serviceExtension().populateNotificationContent(bestAttemptContent, withContentHandler: contentHandler)
        }
    }

    // ...
}
```

**NOTE**: FCM via APNs **does not work on iOS Simulators**. To receive messages & notifications a real device is required.

## Set up the Fanmeter SDK in your iOS App

Few steps are required to be ready to use this SDK.

### Import the SDK

After configuring your app to integrate with FCM, you are ready to use this SDK to properly engage with your fans! There are two approaches to include it in your app:

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

### Methods initialize and getEventData

The `initialize` method is mandatory for both plug-&-play and build your own UI scenarios. It is responsible for initializing the SDK and must be invoked before using any other method (a possible approach is to invoke it as soon as the app finishes launching). It returns, in its callback, an auth token on success, or nil otherwise.

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
    // Keep the auth token where the Fanzone view can reach it, since the Fanzone (see below)
    // needs it to load its page.
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

## Fanzone Web View

Fanzone is Pluggable's web-based activation interface - it is the view you present to your fans (for example, the one you open after `execute` returns an event). The Fanmeter SDK ships a built-in JavaScript bridge, `FanmeterWebInterface`, so the Fanzone web app can call native SDK methods directly, without any platform-specific logic on the JavaScript side. The app retains full control over the WebView - the SDK only installs the bridge.

Before loading the Fanzone, the SDK must be initialized using the `initialize` method in order to obtain a valid authentication token. This token must then be included in the request used to load the Fanzone page (as an `Authorization: Bearer` header).

There are two URLs to open, depending on how the user got there:

  * `https://data.fanofthematch.ai/fanzone/` - the **Fanzone home**, listing every activation. This is what you open when the user reaches the Fanzone from your app itself: a button, a banner, a navbar item. No event id is involved;
  * `https://data.fanofthematch.ai/fanzone/fotm?eventId=EVENT_ID` - the **Fan of the Match activation for one event**. This is where a tapped Fanmeter notification leads, using the event id that `execute` returned.

To use it, create a `WKWebView` and call `FanmeterWebInterface.attach(to:onClose:)` on it before loading the Fanzone URL. Your app keeps full ownership of the WebView - it creates it, loads the URL and decides how the screen is presented - and the SDK shows its own localized GPS permission dialogs when they are needed, so you do not have to build them yourself.

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

Then, keep what the Fanzone shows in an object that both your views and the `AppDelegate` can reach, which the plug-&-play flow (below) also calls right after `execute` returns an event:

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

A few things to keep in mind:

  * Call `attach(to:onClose:)` **before** loading the page: the bridge is injected at document start and only applies to page loads started after it, so loading the URL first would leave the page without `window.FanmeterBridge`;
  * The WebView keeps the bridge alive, so you do not need to hold a reference to it;
  * If your app uses UIKit, `FanmeterWebInterface.makeWebView()` returns a `WKWebView` with the bridge already installed - load the Fanzone request on it as above.

The Fanzone ships its own close control, so **you should not render a native header or close button around the WebView** - the web layer already has one, and it closes the screen by calling `window.FanmeterBridge.closeWebView()`. By default, that dismisses the topmost presented view controller, which is enough for a UIKit screen presented modally. If you pass an `onClose` callback, it is invoked instead - in SwiftUI always pass it, wired to the state that presents the Fanzone (as above), since dismissing the view controller behind SwiftUI's back would leave that state out of sync.

## Plug-&-Play UI

After initializing the SDK, you are now ready to start calling Fanmeter. In particular, if you want to automate the entire process, this library exposes one method, that must be called as demonstrated below:
  * `execute`, processes a tapped Fanmeter notification and returns the event the user should be directed to (your app then opens the Fanzone view for that event).

FCM delivers all messages targeting Apple apps through APNs, so the tap reaches your app through the `UNUserNotificationCenter` delegate (see Apple's documentation on [Handling Notifications and Notification-Related Actions](https://developer.apple.com/documentation/usernotifications/handling_notifications_and_notification-related_actions)). Set it, together with the FCM delegate, as soon as the app finishes launching, and register the app for remote notifications (`AppDelegate.swift`):

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
        // Initialize the SDK as early as possible, since the Fanzone needs its auth token.
        // EntryPoint.initialize(...)
        return true
    }
}
```

If you're using **SwiftUI**, attach this app delegate to your `App` struct via `UIApplicationDelegateAdaptor`:

```swift
@main
struct YourApp: App {
    // Register the app delegate for Firebase and notifications setup.
    @UIApplicationDelegateAdaptor(AppDelegate.self) var delegate

    var body: some Scene {
        WindowGroup {
            RootView()
        }
    }
}
```

The `execute` method is called when a Fanmeter notification is tapped - whether the app was in the foreground, in the background, or terminated. It does not open any view itself - its callback returns a `(code, eventId)` pair and, when `eventId` is not nil, your app is responsible for opening the Fanzone view for that event (see the [Fanzone Web View](#fanzone-web-view) section). Also, since iOS does not display notifications that arrive while the app is in the foreground, implement `willPresent` so the user still sees (and can tap) them:

```swift
extension AppDelegate: UNUserNotificationCenterDelegate {

    // Called when the user taps a notification.
    func userNotificationCenter(_ center: UNUserNotificationCenter, didReceive response: UNNotificationResponse, withCompletionHandler completionHandler: @escaping () -> Void) {
        let userInfo = response.notification.request.content.userInfo
        EntryPoint.execute(notificationData: userInfo) { code, eventId in
            NSLog("EntryPoint.execute: \(String(describing: code)), \(String(describing: eventId))")
            // `execute` does not open any view - when an eventId is returned, your app
            // is responsible for opening the Fanzone view for that event.
            if let eventId = eventId {
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

The `execute` method returns a `(code, eventId)` pair, where the code is one of the following values:
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

You are also **required to subscribe the user to a FCM topic** so that the user can receive Fanmeter notifications. This can be done, for example, as follows:

```swift
extension AppDelegate: MessagingDelegate {

    func messaging(_ messaging: Messaging, didReceiveRegistrationToken fcmToken: String?) {
        // Subscribe to a topic.
        Messaging.messaging().subscribe(toTopic: "football_senior") { _ in
            NSLog("Subscribed to topic: football_senior!")
        }
    }
}
```

## Build your own UI

If you want full control and implement your own UI, this library exposes three methods, that must be called as demonstrated below. In particular:
  * `startService`, starts the Fanmeter service that enables Fanmeter for your client's device during a particular event;
  * `stopService`, stops the Fanmeter service. The service will, still, terminate automatically as soon as the event ends. This returns 1, if success, otherwise an error code;
  * `isServiceRunning`, used to check the current status of the Fanmeter service. Returns whether it is running and, if so, the event being tracked.

The `startService` method is used to start the Fanmeter service. This method accepts an eventId, that should be obtained using the `getEventData` method and should be called as follows (associated, for example, to a particular button). When needed, the SDK asks the user for the location permission and for the precise location itself:

```swift
// e.g. on a button click.
EntryPoint.startService(eventId: EVENT_ID) { result in
    NSLog("EntryPoint.startService: \(result)")
}
```

Where:
  * EVENT_ID, the id of the event the user will participate when the start service is called.

This method returns the following values:
  * 1, success;
  * -80, no GPS/PUSH permissions;
  * -81, GPS disabled;
  * -82, invalid event coordinates;
  * -85, restricted location permission;
  * -86, no precise location permission;
  * -87, unknown location permission status;
  * -89, SDK not initialized;
  * -92, invalid company license;
  * -93, invalid event;
  * -94, event not happening now;
  * -95, invalid external user data;
  * -96, failed to get event data;
  * -97, failed to start the Fanmeter service;
  * -98, another Fanmeter service is already running.

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

## Additional info

Other important methods are to **request user permission** to be able to **send notifications** (shown in the [Plug-&-Play UI](#plug--play-ui) section). Also, **get the user token and listen for changes** to get the user FCM_TOKEN and update it when it changes.

In particular, if you need to know the FCM token ID associated with each user device, the `MessagingDelegate` above receives it - this callback fires at each app startup and whenever a new token is generated:

```swift
func messaging(_ messaging: Messaging, didReceiveRegistrationToken fcmToken: String?) {
    NSLog("TOKEN is: \(String(describing: fcmToken))")
    // ...
}
```

In addition, **prevent multiple openings of the Fanzone view** so that it is only presented when no previous one is on screen. The `Fanzone` object from the [Fanzone Web View](#fanzone-web-view) section already does it, ignoring any request to open it while it is presented.

For more info visit [our website](https://pluggableai.xyz/) or give us feedback to <info@pluggableai.xyz>.
