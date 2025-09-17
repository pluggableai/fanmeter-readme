# Pluggable's Fanmeter SDK for iOS

The Fanmeter SDK is the plugin responsible for enabling Fanmeter in any mobile application. It is a simple plug-&-play SDK that makes available a set of methods that can be used to allow users to participate in activations such as the **FAN OF THE MATCH**. The SDK is available in multiple formats, i.e., as a native SDK or as a framework plugin, such as for Flutter or React Native, among others.

## Plug-&-Play vs Build your own UI

There are two possible types of integration. One is plug-&-play and uses a default native UI that is launched to your users for them to participate in Fanmeter events - it is also able to launch and deliver push notifications as soon as the event starts and finishes, if you configure it. If, on the other hand, you wish to create your own Fanmeter UI, you will follow a "build your own UI" approach.

  1. **Plug-&-Play UI**: you want everything automated (including a default Fanmeter view). You should also integrate with FCM to handle received notifications that start the event (in summary, you'll need to use the *execute* and the *launchFanmeterView* methods);

  2. **Build your own UI**: you want to handle the conditions yourself and develop your own UI (just call the *startService*, *stopService*, and *isServiceRunning* methods).

## Pre-Conditions

### Meta-Data

For iOS, add the **Background Modes** capability and enable **Location Updates**. Additionally, open your `Info.plist` file and add the following code at the bottom of the file:

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

### Push Notifications

If you want a **fully plug-&-play experience** and wish to receive push notifications (if not receiving yet) from Fanmeter in your app, you should integrate Firebase Cloud Messaging. A complete, step-by-step tutorial is available in [Firebase's wiki](https://firebase.google.com/docs/cloud-messaging).

## Set up the Fanmeter SDK in your iOS App

Few steps are required to be ready to use this SDK.

### Import the SDK

There are two approaches to include Fanmeter's iOS SDK in your app:

1. Using CocoaPods, by adding `pod 'fanmeter_sdk_ios', '~> 4.3.2'` to your podfile (check which version is the latest);

2. Using a XCFramework SDK (the recommended way).

By using the second approach, you just need to **download the latest version of the framework file** containing the Fanmeter SDK (the link is made available by your account manager). Just add the framework file as a framework to your Apple project. For that, in Xcode, you must go to your project main page, select the desired target, and in the "General" tab scroll down to "Frameworks, Libraries, and Embedded Content". There, just add the framework file, selecting the option **"Embed and Sign"**.

You are now able to properly use the SDK.

```swift
// ...
import fanmeter_sdk_ios
// ...
```

If not done already, you will need to add FirebaseMessaging to your project. To do that, you can use Swift Package Manager or CocoaPods, as described [here](https://firebase.google.com/docs/ios/setup).

### Push Notifications

If your app is not yet receiving push notifications and you aim to, you must enable Firebase products in your Apple app. For better engagement with the Plug-&-Play UI, you should handle push notifications. To enable Firebase products in your Apple app, use Swift Package Manager to install and manage Firebase dependencies. You can also visit [Firebase's installation guide](https://firebase.google.com/docs/ios/setup) to learn about the different ways you can add Firebase SDKs to your Apple project, including importing frameworks directly and using CocoaPods.

1. In Xcode, with your app project open, navigate to **File > Add Packages**;

2. When prompted, add the Firebase Apple platforms SDK repository:
    ```
    https://github.com/firebase/firebase-ios-sdk
    ```

3. Select the SDK version that you want to use (it is recommended to use the default (latest) SDK version, but you can choose an older version if needed);

4. Choose the Firebase libraries you want to use. If Google Analytics is enabled in your Firebase project, make sure to add FirebaseAnalytics. For Analytics without IDFA collection capability, add FirebaseAnalyticsWithoutAdId instead.

When finished, Xcode will automatically begin resolving and downloading your dependencies in the background. More info is available [here](https://firebase.google.com/docs/ios/setup).

You'll need to add Firebase initialization code to your application. Import the Firebase module and configure a shared instance. For that, import the `FirebaseCore` module in your `UIApplicationDelegate`, as well as any other Firebase modules your app delegate uses.

```swift
import FirebaseCore
import FirebaseMessaging
import UserNotifications
// ...
```

Configure a `FirebaseApp` shared instance in your app delegate's `application(_:didFinishLaunchingWithOptions:)` method:

```swift
// Use Firebase library to configure APIs
FirebaseApp.configure()
```

If you're using **SwiftUI**, you must create an application delegate and attach it to your `App` struct via `UIApplicationDelegateAdaptor` or `NSApplicationDelegateAdaptor`. You must also disable app delegate swizzling. For more information, see the [SwiftUI instructions](https://firebase.google.com/docs/ios/setup).

```swift
@main
struct YourApp: App {
    // register app delegate for Firebase setup
    @UIApplicationDelegateAdaptor(AppDelegate.self) var delegate
    
    var body: some Scene {
        WindowGroup {
            NavigationView {
                ContentView()
            }
        }
    }
}
```

To handle push notifications, you must add such capability to your Apple project. For that, in Xcode, you must go to your project main page, select the desired target, go to the "Signing and Capabilities" tab and click the **"+ Capability"** button. Then, add the **"Push Notifications"** capability. To handle background operations, you should also add the **"Background Modes"** capability and enable the **"Remote notifications"** and the **"Background processing"** modes.

Then, either at startup, or at the desired point in your application flow, register your app for remote notifications. Call `registerForRemoteNotifications` as shown (in `AppDelegate.swift`):

```swift
UNUserNotificationCenter.current().delegate = self
FirebaseApp.configure()
Messaging.messaging().delegate = self

let authOptions: UNAuthorizationOptions = [.alert, .badge, .sound]
UNUserNotificationCenter.current().requestAuthorization(options: authOptions, completionHandler: { _, _ in })
UIApplication.shared.registerForRemoteNotifications()
```

You must assign the UNUserNotificationCenter's delegate property and FIRMessaging's delegate property. For example, in an iOS app, assign it in the `applicationWillFinishLaunchingWithOptions:` or `applicationDidFinishLaunchingWithOptions:` method of the app delegate.

Also, to be able to receive and handle notification images in an Apple app, you must add a **Notification Service Extension**. To add a service extension, perform the required setup tasks for [modifying and presenting notifications in APNs](https://developer.apple.com/documentation/usernotifications/modifying_content_in_newly_delivered_notifications), and then add the following:

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
            // Modify the notification content here...
            Messaging.serviceExtension().populateNotificationContent(bestAttemptContent, withContentHandler: contentHandler)
        }
    }
    
    // ...
}
```

### Methods initialize and getEventData

The `EntryPoint.initialize()` method is mandatory for both plug-&-play and build your own UI scenarios. It is responsible for initializing the SDK and must be invoked before using any other method (a possible approach is to invoke it in the main app flow). It returns success or an error code.

```swift
let YOUR_COMPANY_NAME: String = "companyName"
let YOUR_COMPANY_KEY: String = "AAAA-BBBB-CCCC-DDDD"
let USER_ID: String = "externalUserId"
let TOKEN_ID: String = "externalTokenId"
let USER_EMAIL: String = "externalUserEmail"
let FCM_TOKEN: String = "fcm_token"
let TICKET_NUMBER: String = "ticket_number"
let TICKET_STAND: String = "ticket_stand"
let REGULATION_URL: String = "https://link.to.the.regulation"
let FANMETER_LOG: Bool = false

// When the Fanmeter SDK is initialized, it initializes all company and user data.
let res = EntryPoint.initialize(
    companyName: YOUR_COMPANY_NAME,
    companyKey: YOUR_COMPANY_KEY,
    externalUserId: USER_ID,
    externalTokenId: TOKEN_ID,
    externalUserEmail: USER_EMAIL,
    fcmToken: FCM_TOKEN,
    ticketNumber: TICKET_NUMBER,
    ticketStand: TICKET_STAND,
    urlRegulation: REGULATION_URL,
    log: FANMETER_LOG
)
```

Where:
  * *YOUR_COMPANY_NAME*, is the name of your company in Fanmeter;
  * *YOUR_COMPANY_KEY*, your company's license key;
  * *USER_ID*, the user identifier in your db (can be the username, the uuid, ...);
  * *TOKEN_ID*, the individual smartphone identifier (allows for the same account in different devices);
  * *USER_EMAIL*, the user's email. Nullable;
  * *FCM_TOKEN*, the FCM token id of the user. Nullable;
  * *TICKET_NUMBER*, the user's ticket number - used for additional analytics. Nullable;
  * *TICKET_STAND*, the stand where the user is - used for additional analytics. Nullable;
  * *REGULATION_URL*, the URL to the regulation for your events. Nullable;
  * *FANMETER_LOG*, enables additional logging.

The `EntryPoint.getEventData()` method is used to obtain the full data of a particular Fanmeter event, in a dictionary, including the defined rewards and leaderboard classifications, if existing. If null, or not provided, this returns the closest event to date:

```swift
private let EVENT_NAME: String = "Round 1 2025-2026"
private let EVENT_ID: String = "-99"

// After initialized, you will be able to use the SDK methods. The eventTitle can be null.
EntryPoint.getEventData(eventTitle: EVENT_NAME) { eventData in
    let eventId = eventData["eventId"] as? Int ?? -99
    NSLog("EntryPoint.getEventData(\(EVENT_NAME)): \(eventData)")
}

// OR.

// After initialized, you will be able to use the SDK methods.
EntryPoint.getEventData() { eventData in
    let eventId = eventData["eventId"] as? Int ?? -99
    NSLog("EntryPoint.getEventData(): \(eventData)")
}
```

### Listeners for User Participation

In both scenarios, you can subscribe to a listener that will notify you whenever there is a change in the user's participation status. For example, if the user leaves the venue of a geo-restricted event, the listener will alert you to this change. This can be particularly useful for updating the status of banners or buttons within your app.

Two methods are available: `EntryPoint.subscribeUserParticipationListener()` and `EntryPoint.unsubscribeUserParticipationListener()`. The first method subscribes the listener to a specific event id, and you can subscribe to multiple events. The second method unsubscribes the listener (if no event id is provided, it will unsubscribe all active listeners).

While the unsubscribeUserParticipationListener method always returns 1 (Success), as it forces the stop of any or all running listeners, the subscribeUserParticipationListener listener can return the following status codes:

  * 0: User is not participating;
  * 1: Validating the user participation (validation may take up to 30 seconds);
  * 2: User is participating;
  * 3: User has GPS disabled;
  * 4: User is out of venue;
  * 5: Unknown GPS coordinates.

These methods can be invoked as shown below:

```swift
// Subscribing a listener.
EntryPoint.subscribeUserParticipationListener(eventId: self.eventId) { state in
    NSLog("\(eventId) Participation state changed: \(state)")
}

// OR.

// Unsubscribing a listener. If no event id is set, it cancels all listeners.
EntryPoint.unsubscribeUserParticipationListener() { state in
    NSLog("EntryPoint.unsubscribeUserParticipationListener(): \(state)")
}
```

### Plug-&-Play UI

After initializing the SDK, you are now ready to start calling Fanmeter. In particular, if you want to automate the entire process, this library exposes two methods that must be called as demonstrated below. In particular:

  * `execute`, launches the SDK Fanmeter default's view as soon as a notification is tapped by the user.
  * `launchFanmeterView`, launches the SDK Fanmeter default's view. It can be associated with a button or banner in your app, redirecting the user to the Fanmeter native view, allowing users without notification permissions to participate in the event. It will open the Fanmeter view with the event with the closest date to the current date.

The `execute` method launches the SDK Fanmeter default's view as soon as a notification is tapped by the user. It is worth noting that FCM delivers all messages targeting Apple apps through APNs. To learn more about receiving APNs notifications via UNUserNotificationCenter, see Apple's documentation on [Handling Notifications and Notification-Related Actions](https://developer.apple.com/documentation/usernotifications/handling_notifications_and_notification-related_actions). You must set the `UNUserNotificationCenter` delegate and implement the appropriate delegate methods to receive display notifications from FCM (`AppDelegate.swift`).

```swift
extension AppDelegate: UNUserNotificationCenterDelegate {
    // ...
    
    // Receive displayed notifications for iOS 10 devices.
    // Called by the SDK to show the notification.
    func userNotificationCenter(_ center: UNUserNotificationCenter, didReceive response: UNNotificationResponse, withCompletionHandler completionHandler: @escaping () -> Swift.Void) {
        
        // notification EXTRAS.
        let userInfo = response.notification.request.content.userInfo
        
        EntryPoint.execute(notificationData: userInfo) { result in
            NSLog("EntryPoint.execute: \(String(describing: result))")
        }
        
        completionHandler()
    }
    
    // ...
}
```

Where:
  * *notificationData*, is the remote data received with the notification.

Finally, the `launchFanmeterView` method should be called on a button click, redirecting users to Fanmeter's default UI. This method accepts an eventId, that should be obtained using the `getEventData` method. If a null eventId is passed, this method will open the closest available event to the current date.

```swift
// Launch view button!
Button(action: {
    DispatchQueue.main.async {
        EntryPoint.launchFanmeterView(eventId: self.eventId) { res in
            NSLog(String(res))
        }
    }
})
```

These methods return the following values:

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

You are also **required to subscribe the user to a FCM topic** so that the user may receive Fanmeter notifications. This can be done, for example, on your main app flow.

```swift
extension AppDelegate: MessagingDelegate {
    
    func messaging(_ messaging: Messaging, didReceiveRegistrationToken fcmToken: String?) {
        // Subscribe the user to a specific topic.
        Messaging.messaging().subscribe(toTopic: "football_senior") { _ in
            print("[APPDELEGATE] Subscribed to football_senior topic.")
        }
    }
}
```

### Build your own UI

If you want full control and implement your own UI, this library exposes three methods, that must be called as demonstrated below. In particular:

  * `startService`, starts the Fanmeter service that enables Fanmeter for your client's device during a particular event.
  * `stopService`, stops the Fanmeter service. The service will, still, terminate automatically as soon as the event ends. This returns 1, if success, otherwise an error code.
  * `isServiceRunning`, used to check the current status of the Fanmeter service. Returns 1, if the service is running, otherwise 0.

The `startService` method is used to start the Fanmeter service. This method accepts an eventId, that should be obtained using the `getEventData` method and should be called as follows (associated, for example, to a particular button):

```swift
// Start the Fanmeter Service using a specific button!
Button(action: {
    DispatchQueue.main.async {
        EntryPoint.startService(eventId: self.eventId) { res in
            NSLog(String(res))
        }
    }
})
```

Where:
  * *eventId*, is the id of the event the user will participate in when the start service is called.

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

```swift
// Stop the Fanmeter Service using a specific button!
Button(action: {
    EntryPoint.stopService() { res in
        NSLog(String(res))
    }
})
```

Finally, the `isServiceRunning` method is used to check the current status of the Fanmeter service. This returns 1, if the service is running, otherwise 0.

```swift
// Check the Fanmeter Service status.
EntryPoint.isServiceRunning() { res in
    NSLog(String(res))
}
```

### Additional info

Other important methods are to request user permission to be able to send notifications and request GPS permission. Also, get the user token and listen for changes to get the user FCM_TOKEN and update it when it changes.

In addition, please note that FCM via APNs does not work on iOS Simulators. To receive messages and notifications a real device is required.