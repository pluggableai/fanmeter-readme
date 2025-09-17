# Pluggable's Fanmeter SDK for Android

The Fanmeter SDK enables fan engagement features in any Android application. It is a plug-&-play plugin that provides methods for participating in activations such as FAN OF THE MATCH or SUPER FAN.

## Plug-&-Play vs Build your own UI

There are two possible integration types:

1. **Plug-&-Play UI**: everything is automated, including the default Fanmeter UI and push notifications. Use the methods _initialize_, _getEventData_, _subscribeUserParticipationListener_, _unsubscribeUserParticipationListener_, _launchFanmeterView_, _isServiceRunning_.
2. **Build your own UI**: full control over the interface and logic, using _startService_, _stopService_, and _isServiceRunning_.

## Prerequisites

### Permissions and Meta-Data

- Push notification permission is required to inform the user about the foreground service.
- Location permission is required for users to participate in geo-restricted events.

On Android, add to your `AndroidManifest.xml`:

```xml
<key>UIBackgroundModes</key>
<array>
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

**Note:** Since Android API 34, when publishing an app that uses foreground data sync services, you must justify the use of `FOREGROUND_SERVICE_DATA_SYNC`. If prompted, select **OTHER** and provide the video and description as per the SDK documentation:

- Video: https://youtu.be/w4d7Pgksok0
- Description: "Fanmeter is an Android SDK, incorporated in this app, that uses a foreground data sync service to communicate non-intrusive sensor data with Pluggable servers, which are used to quantify user engagement in real time. The service must start as soon as the user opts to participate in the event and remain active until the user decides to end their participation."

### Push Notifications

For a full plug-&-play experience, integrate [Firebase Cloud Messaging](https://firebase.google.com/docs/cloud-messaging):

1. Install [Firebase](https://firebase.google.com/docs/android/setup).
2. Add the dependencies to your `build.gradle`:
   ```gradle
   implementation 'com.google.android.gms:play-services-location:21.3.0'
   implementation 'com.android.volley:volley:1.2.1'
   implementation 'androidx.work:work-runtime-ktx:2.8.0'
   implementation 'androidx.constraintlayout:constraintlayout:2.1.4'
   implementation 'androidx.navigation:navigation-fragment-ktx:2.5.3'
   implementation 'androidx.databinding:viewbinding:7.1.2'
   implementation 'androidx.swiperefreshlayout:swiperefreshlayout:1.1.0'
   implementation 'io.github.pluggableai:fanmeter:4.1.12'
   ```
3. Configure Firebase as per the [official tutorial](https://firebase.google.com/docs/cloud-messaging/android/client).

## SDK Installation

Add the dependency to your `build.gradle`:

```gradle
implementation 'io.github.pluggableai:fanmeter:4.1.12'
```

## Main Methods: initialize and getEventData

The `initialize` method is mandatory for both scenarios. It initializes the SDK and must be called before any other method.

```kotlin
private const val YOUR_COMPANY_NAME: String = "companyName"
private const val YOUR_COMPANY_KEY: String = "AAAA-BBBB-CCCC-DDDD"
private const val USER_ID: String = "externalUserId"
private const val TOKEN_ID: String = "externalTokenId"
private const val USER_EMAIL: String = "externalUserEmail"
private const val FCM_TOKEN: String = "fcm_token"
private const val TICKET_NUMBER: String = "ticket_number"
private const val TICKET_STAND: String = "ticket_stand"
private const val REGULATION_URL: String = "https://link.to.the.regulation"
private const val FANMETER_LOG: Boolean = false

val res = EntryPoint.initialize(
    companyName = YOUR_COMPANY_NAME,
    companyKey = YOUR_COMPANY_KEY,
    userId = USER_ID,
    tokenId = TOKEN_ID,
    userEmail = USER_EMAIL,
    fcmToken = FCM_TOKEN,
    ticketNumber = TICKET_NUMBER,
    ticketStand = TICKET_STAND,
    regulationUrl = REGULATION_URL,
    log = FANMETER_LOG
)
```

The `getEventData` method returns the full data of a Fanmeter event:

```kotlin
private const val EVENT_NAME: String = "Round 1 2025-2026"

EntryPoint.getEventData(
    eventName = EVENT_NAME
) { eventData ->
    Log.d("A_LOG_TAG", "EntryPoint.getEventData(): $eventData")
}

// Or, for the closest event:
EntryPoint.getEventData(
    applicationContext
) { eventData ->
    Log.d("A_LOG_TAG", "EntryPoint.getEventData(): $eventData")
}
```

## User Participation Listeners

You can subscribe a listener to be notified about changes in the user's participation status:

```kotlin
private const val EVENT_ID: String = "-99"

EntryPoint.subscribeUserParticipationListener(
    eventId = EVENT_ID
) { state ->
    // Update your button/banner status.
}

EntryPoint.unsubscribeUserParticipationListener() {
    Log.d("A_LOG_TAG", "EntryPoint.unsubscribeUserParticipationListener: $it")
}
```

## Plug-&-Play UI

After initializing the SDK, you can launch the default Fanmeter UI:

```kotlin
Button(onClick = {
    EntryPoint.launchFanmeterView(EVENT_ID)
})
```

## Check Service Status

```kotlin
EntryPoint.isServiceRunning(
    applicationContext
) {
    Log.d("EntryPoint.isServiceRunning", "Service status: $it")
}
```

## Notes

- To customize the notification icon, add your desired icon to the Android `drawable` folder and name it `ic_push_app_icon`.
- Refer to the official documentation for details on Firebase integration and permissions.
- For questions or support, visit https://pluggableai.xyz/ or email info@pluggableai.xyz.
