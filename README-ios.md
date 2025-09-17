# Pluggable's Fanmeter SDK for iOS

The Fanmeter SDK enables fan engagement features in any iOS application. It is a plug-&-play plugin that provides methods for participating in activations such as FAN OF THE MATCH or SUPER FAN.

## Plug-&-Play vs Build your own UI

There are two possible integration types:
1. **Plug-&-Play UI**: everything is automated, including the default Fanmeter UI and push notifications. Use the methods _initialize_, _getEventData_, _subscribeUserParticipationListener_, _unsubscribeUserParticipationListener_, _launchFanmeterView_, _isServiceRunning_.
2. **Build your own UI**: full control over the interface and logic, using _startService_, _stopService_, and _isServiceRunning_.

## Prerequisites

### Permissions and Meta-Data

- Push notification permission is required to inform the user about the foreground service.
- Location permission is required for users to participate in geo-restricted events.

On iOS, add the **Background Modes** capability and enable **Location Updates**. Also, open your _Info.plist_ file and add the following:

```xml
<key>UIBackgroundModes</key>
<array>
    <string>fetch</string>
    <string>remote-notification</string>
</array>
<key>NSLocationTemporaryUsageDescriptionDictionary</key>
<dict>
    <string>Access to precise location is required during Fanmeter events!</string>
</dict>
<key>NSLocationAlwaysUsageDescription</key>
<string>Location access is required to participate in Fanmeter events!</string>
<key>NSLocationWhenInUseUsageDescription</key>
<string>Location access is required to participate in Fanmeter events!</string>
```

### Push Notifications

For a full plug-&-play experience, integrate [Firebase Cloud Messaging](https://firebase.google.com/docs/cloud-messaging):

1. Install [Firebase](https://firebase.google.com/docs/ios/setup).
2. Add the dependencies to your project (FirebaseCore, FirebaseMessaging, UserNotifications).
3. Configure Firebase in your AppDelegate:
   ```swift
   import FirebaseCore
   import FirebaseMessaging
   import UserNotifications

   FirebaseApp.configure()
   Messaging.messaging().delegate = self
   UNUserNotificationCenter.current().delegate = self
   let authOptions: UNAuthorizationOptions = [.alert, .badge, .sound]
   UNUserNotificationCenter.current().requestAuthorization(options: authOptions, completionHandler: { _, _ in })
   UIApplication.shared.registerForRemoteNotifications()
   ```

## SDK Installation

Add the Fanmeter SDK to your project as instructed by your integration manager.

## Main Methods: initialize and getEventData

The `initialize` method is mandatory for both scenarios. It initializes the SDK and must be called before any other method.

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

let res = EntryPoint.initialize(
    companyName: YOUR_COMPANY_NAME,
    companyKey: YOUR_COMPANY_KEY,
    userId: USER_ID,
    tokenId: TOKEN_ID,
    userEmail: USER_EMAIL,
    fcmToken: FCM_TOKEN,
    ticketNumber: TICKET_NUMBER,
    ticketStand: TICKET_STAND,
    regulationUrl: REGULATION_URL,
    log: FANMETER_LOG
)
```

The `getEventData` method returns the full data of a Fanmeter event:

```swift
let EVENT_NAME: String = "Round 1 2025-2026"

EntryPoint.getEventData(eventTitle: EVENT_NAME) { eventData in
    // Handle eventData
}

// Or, for the closest event:
EntryPoint.getEventData() { eventData in
    // Handle eventData
}
```

## User Participation Listeners

You can subscribe a listener to be notified about changes in the user's participation status:

```swift
let EVENT_ID: String = "-99"

EntryPoint.subscribeUserParticipationListener(eventId: EVENT_ID) { state in
    // Update your button/banner status.
}

EntryPoint.unsubscribeUserParticipationListener() { state in
    NSLog("EntryPoint.unsubscribeUserParticipationListener(): \(state)")
}
```

## Plug-&-Play UI

After initializing the SDK, you can launch the default Fanmeter UI:

```swift
Button(action: {
    // Launch Fanmeter view for EVENT_ID
})
```

## Check Service Status

```swift
EntryPoint.isServiceRunning() { res in
    NSLog(String(res))
}
```

## Notes

- To customize the notification icon, add your desired icon to the iOS asset catalog.
- Refer to the official documentation for details on Firebase integration and permissions.
- For questions or support, visit https://pluggableai.xyz/ or email info@pluggableai.xyz.
