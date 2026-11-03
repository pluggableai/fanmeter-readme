# Pluggable's Fanmeter SDK for Android apps

The Fanmeter SDK is the one responsible for enabling Fanmeter in any mobile application. It is a simple plug-&-play SDK that makes available a set of methods that can be used to allow users to participate in activations such as the FAN OF THE MATCH or the SUPER FAN.

## Table of Contents

+ [Plug-&-Play vs Build your own UI](#plug--play-vs-build-your-own-ui)
+ [Pre-Conditions](#pre-conditions)
  + [Compatibility](#compatibility)
  + [Meta-Data](#meta-data)
  + [Google Play Policy Declarations](#google-play-policy-declarations)
  + [Push Notifications](#push-notifications)
+ [Set up the Fanmeter SDK in your Android App](#set-up-the-fanmeter-sdk-in-your-android-app)
  + [Import the SDK](#import-the-sdk)
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

This SDK requires a minimum **minSdk** version of **23** and a minimum **compileSdk** version of **34**. Make sure your app's `build.gradle` file has `minSdk 23` and `compileSdk 34`, or higher, configured.

### Meta-Data

The SDK already declares, in its own manifest, every permission it relies on (internet, the data sync foreground service, and the fine, coarse and background location), so they are merged into your app's manifest automatically.

Push notification permission is required to inform the user that a foreground service is running. Make sure to request it (see [Additional info](#additional-info)).

Location permission is necessary for fans to participate in geo-restricted events, but you do not need to request it yourself: whenever a service is started with an `Activity` (see [Build your own UI](#build-your-own-ui)) or from the Fanzone (see [Fanzone Web View](#fanzone-web-view)), the SDK asks the user for the missing location permissions and to turn on the GPS, showing its own localized dialogs.

### Google Play Policy Declarations

When publishing to the Google Play Store an app that integrates Fanmeter, you are required to fill a **Policy Declaration** justifying two sensitive permissions Fanmeter relies on: the **Data Sync foreground service** and **background location**.

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

### Push Notifications

If you want a **fully plug-&-play experience** and wish to receive push notifications (if not receiving yet) from Fanmeter in your app, you should integrate [Firebase Cloud Messaging (FCM)](https://firebase.google.com/docs/cloud-messaging/android/get-started). A complete, step-by-step tutorial is available in [Firebase's official documentation](https://firebase.google.com/docs/cloud-messaging).

**NOTE**: To receive messages & notifications a real device is recommended.

## Set up the Fanmeter SDK in your Android App

Few steps are required to be ready to use this SDK.

### Import the SDK

After configuring your app to integrate with FCM, you are ready to use this SDK to properly engage with your fans! The [Fanmeter SDK](https://central.sonatype.com/artifact/io.github.pluggableai/fanmeter) is published in Maven Central. To import it, check the latest version and add it to the dependencies of your app's `build.gradle` file:

```gradle
dependencies {
    // ...
    implementation 'io.github.pluggableai:fanmeter:5.0.4'
}
```

Then, sync your project with the Gradle files. The libraries the SDK depends on are resolved automatically.

To customize the used notification icon, just add the desired icon in the Android's drawable folder and name it **ic_push_app_icon**. Otherwise, a default icon, not related to your app, will be used.

### Methods initialize and getEventData

The `initialize` method is mandatory for both plug-&-play and build your own UI scenarios. It is responsible for initializing the SDK and must be invoked before using any other method (a possible approach is to invoke it in the app's main activity). It returns, in its callback, an auth token on success, or null otherwise.

```kotlin
import com.pluggableai.fanmeter.EntryPoint

private const val YOUR_COMPANY_NAME: String = "companyName"
private const val YOUR_COMPANY_KEY: String = "AAAA-BBBB-CCCC-DDDD"
private const val USER_ID: String = "externalUserId"
private const val TOKEN_ID: String = "externalTokenId"
private const val USERNAME: String = "username"
private const val USER_EMAIL: String = "externalUserEmail"
private const val FCM_TOKEN: String = "fcm_token"
private const val FANMETER_LOG: Boolean = false

// ...
// Keep the auth token at a scope both your init flow and the Fanzone view can reach, since the
// Fanzone (see below) needs it to load its page.
private var authToken: String? = null

val initializeDataMap = mapOf(
    "companyName" to YOUR_COMPANY_NAME,
    "companyKey" to YOUR_COMPANY_KEY,
    "externalUserId" to USER_ID,
    "externalTokenId" to TOKEN_ID,
    // Optional:
    "username" to USERNAME,
    "externalUserEmail" to USER_EMAIL,
    "fcmToken" to FCM_TOKEN
)

// When the Fanmeter SDK is initialized, it initializes all company and user data.
EntryPoint.initialize(
    applicationContext,
    initializeDataMap,
    FANMETER_LOG
) { token ->
    Log.d(TAG, "EntryPoint.initialize: $token")
    authToken = token
}
```

Where - required keys:
  * companyName, the name of your company in Fanmeter;
  * companyKey, your company's license key;
  * externalUserId, the user identifier in your db (can be the username, the uuid, ...);
  * externalTokenId, the individual smartphone identifier (allows for the same account in different devices).

Optional keys (the map does not take null values, so leave out the ones you do not have):
  * username, the user's display name;
  * externalUserEmail, the user's email;
  * fcmToken, the FCM token id of the user.

Additional parameters:
  * applicationContext, the application context;
  * FANMETER_LOG, enables additional logging.

The `getEventData` method is used to obtain the full data of a particular Fanmeter event, in a `JSONObject`, including the defined rewards and leaderboard classifications, if existing. If null, or not provided, this returns the closest event to date:

```kotlin
private const val EVENT_NAME: String = "Round 1 2025-2026"

// After initialized, you will be able to use the SDK methods.
EntryPoint.getEventData(
    applicationContext,
    EVENT_NAME
) { eventData ->
    val eventId = eventData.optLong("eventId", -99)
    Log.d(TAG, "EntryPoint.getEventData($EVENT_NAME): $eventId")
}

// OR.

// After initialized, you will be able to use the SDK methods.
EntryPoint.getEventData(
    applicationContext
) { eventData ->
    val eventId = eventData.optLong("eventId", -99)
    Log.d(TAG, "EntryPoint.getEventData(): $eventId")
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

```kotlin
// Subscribing a listener. The EVENT_ID (a Long) can be obtained with getEventData.
EntryPoint.subscribeUserParticipationListener(
    applicationContext,
    EVENT_ID
) { state ->
    Log.d(TAG, "Event $EVENT_ID | Participation state changed: $state")
}

// OR.

// Unsubscribing a listener. If no event id is set, it cancels all listeners.
EntryPoint.unsubscribeUserParticipationListener {
    Log.d(TAG, "Unsubscribed with result: $it")
}
```

## Fanzone Web View

Fanzone is Pluggable's web-based activation interface - it is the view you present to your fans (for example, the one you open after `execute` returns an event). The Fanmeter SDK ships a built-in JavaScript bridge, `FanmeterWebInterface`, so the Fanzone web app can call native SDK methods directly, without any platform-specific logic on the JavaScript side. The app retains full control over the WebView - the SDK only provides the bridge and the WebView client it must use.

Before loading the Fanzone, the SDK must be initialized using the `initialize` method in order to obtain a valid authentication token. This token must then be included in the request used to load the Fanzone page (as an `Authorization: Bearer` header).

There are two URLs to open, depending on how the user got there:

  * `https://data.fanofthematch.ai/fanzone/` - the **Fanzone home**, listing every activation. This is what you open when the user reaches the Fanzone from your app itself: a button, a banner, a navbar item. No event id is involved;
  * `https://data.fanofthematch.ai/fanzone/fotm?eventId=EVENT_ID` - the **Fan of the Match activation for one event**. This is where a tapped Fanmeter notification leads, using the event id that `execute` returned.

To use it, create an `Activity` that hosts the WebView, configured as shown below. Your app keeps full ownership of the WebView - it creates it, loads the URL and decides how the screen is presented - and the SDK shows its own localized GPS permission dialogs when they are needed, so you do not have to build them yourself.

```kotlin
import android.os.Bundle
import android.webkit.WebView
import androidx.activity.ComponentActivity
import com.pluggableai.fanmeter.FanmeterWebInterface
import com.pluggableai.fanmeter.FanmeterWebViewClient

class FanzoneActivity : ComponentActivity() {

    companion object {
        const val FANZONE_URL = "https://data.fanofthematch.ai/fanzone/"
        const val EXTRA_URL = "url"
        const val EXTRA_TOKEN = "token"
    }

    private var webView: WebView? = null

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        val webView = WebView(this)
        // 1. Use the Fanmeter client, closing the screen if the system reclaims the WebView's renderer.
        webView.webViewClient = FanmeterWebViewClient {
            this.webView = null
            finish()
        }
        webView.settings.javaScriptEnabled = true
        // 2. Install the Fanmeter bridge, exposed to the page as window.FanmeterBridge.
        webView.addJavascriptInterface(FanmeterWebInterface(this, webView), "FanmeterBridge")
        // 3. Load the Fanzone URL, passing the token returned by initialize.
        val headers = mapOf("Authorization" to "Bearer ${intent.getStringExtra(EXTRA_TOKEN) ?: ""}")
        webView.loadUrl(intent.getStringExtra(EXTRA_URL) ?: FANZONE_URL, headers)
        setContentView(webView)
        this.webView = webView
    }

    override fun onDestroy() {
        // 4. Tear the WebView down.
        webView?.let { FanmeterWebViewClient.destroy(it) }
        webView = null
        super.onDestroy()
    }
}
```

Do not forget to declare it in your app's manifest, inside the application tag:

```xml
<activity
    android:name=".FanzoneActivity"
    android:exported="false" />
```

Then, open it from anywhere in your app with a small helper, which the plug-&-play flow (below) also calls right after `execute` returns an event:

```kotlin
// Opens the Fanzone screen with a given URL and the token returned by initialize.
private fun openFanzone(url: String) {
    val intent = Intent(this, FanzoneActivity::class.java)
    intent.putExtra(FanzoneActivity.EXTRA_URL, url)
    intent.putExtra(FanzoneActivity.EXTRA_TOKEN, authToken)
    // Prevents a second Fanzone from being opened on top of one that is already on screen.
    intent.addFlags(Intent.FLAG_ACTIVITY_SINGLE_TOP)
    startActivity(intent)
}

// From a button/banner/navbar: the Fanzone home.
private fun openFanzoneHome() = openFanzone(FanzoneActivity.FANZONE_URL)
// From a tapped notification: the FOTM activation of the event execute returned.
private fun openFanzoneForEvent(eventId: Long) = openFanzone("${FanzoneActivity.FANZONE_URL}fotm?eventId=$eventId")
```

A few things to keep in mind:

  * The `FanmeterWebViewClient` is required. With the default `WebViewClient`, when the system reclaims the WebView's renderer (which some devices do within seconds of the app going to the background), Android kills the whole app process - and the running Fanmeter service with it, silently ending the user's participation. Its optional callback runs after the dead WebView was torn down, so you can close (or rebuild) the screen;
  * If you need a WebView client of your own (`shouldOverrideUrlLoading`, `onPageFinished`, error pages, ...), extend `FanmeterWebViewClient` instead of `WebViewClient`. If the WebView already has a client you cannot replace, call `FanmeterWebViewClient.protect(webView)` after that client is set;
  * Call `FanmeterWebViewClient.destroy(webView)` in the Activity's `onDestroy`, otherwise the WebView keeps counting as a live WebView even after the screen is gone;
  * Add the `FanmeterWebInterface` **before** calling `loadUrl`, and with the name `FanmeterBridge`: Android only exposes it to pages loaded after it is added, and the Fanzone looks for `window.FanmeterBridge`;
  * Pass the `Activity` itself to `FanmeterWebInterface`, not the application context, since the SDK uses it to ask for the location permissions and to turn on the GPS when the user joins an event from the Fanzone.

The Fanzone ships its own close control, so **you should not render a native header or close button around the WebView** - the web layer already has one, and it closes the screen by calling `window.FanmeterBridge.closeWebView()`. By default, that finishes the `Activity` passed to `FanmeterWebInterface`. If the WebView is not the whole Activity (e.g., it lives in a Fragment or a Compose screen), pass an `onClose` callback, which is invoked instead and should dismiss only that screen:

```kotlin
webView.addJavascriptInterface(
    FanmeterWebInterface(requireActivity(), webView) {
        // Dismiss only the Fanzone screen, e.g.:
        findNavController().popBackStack()
    },
    "FanmeterBridge"
)
```

When the host is a `ComponentActivity` (such as `AppCompatActivity`), the system back walks the Fanzone's own navigation first and, once there is nothing left to go back to, falls back to the Activity's normal back behavior - so both the Fanzone's close button and the system back leave the screen. Hosts on the plain `Activity` API do not get this and must handle the back themselves (with `webView.canGoBack()` and `webView.goBack()`).

## Plug-&-Play UI

After initializing the SDK, you are now ready to start calling Fanmeter. In particular, if you want to automate the entire process, this library exposes two methods, that must be called as demonstrated below. In particular:
  * `execute`, processes a tapped Fanmeter notification and returns the event the user should be directed to (your app then opens the Fanzone view for that event);
  * `launchFanmeterNotification`, launches a local Fanmeter notification to the user, which is required by Android when the app is in the foreground.

The `launchFanmeterNotification` method launches a local notification when the app is in a foreground state. For that, in your app's manifest, inside the application tag, add a service that extends `FirebaseMessagingService` (if you do not have one yet):

```xml
<service
    android:name=".notifications.MyFirebaseMessagingService"
    android:exported="false">
    <intent-filter>
        <action android:name="com.google.firebase.MESSAGING_EVENT" />
    </intent-filter>
</service>
```

The service should override the `onMessageReceived` callback and call `launchFanmeterNotification`. It must handle any message within 20 seconds of receipt (10 seconds on Android Marshmallow) - the Fanmeter SDK respects these limits and executes in milliseconds, so make sure you don't add any extra processing that consumes the available computation time.

```kotlin
import android.util.Log
import com.google.firebase.messaging.FirebaseMessagingService
import com.google.firebase.messaging.RemoteMessage
import com.pluggableai.fanmeter.EntryPoint

class MyFirebaseMessagingService : FirebaseMessagingService() {

    companion object {
        private const val TAG: String = "MyFirebaseMessagingService"
    }

    override fun onMessageReceived(remoteMessage: RemoteMessage) {
        // Fanmeter pushes carry their payload in the data map.
        if(remoteMessage.data.isEmpty()) return
        EntryPoint.launchFanmeterNotification(
            this,
            remoteMessage.data,
            null
        ) {
            Log.d(TAG, "EntryPoint.launchFanmeterNotification: $it")
        }
    }
}
```

Where:
  * notificationData (`remoteMessage.data`), is the remote data received with the notification;
  * NOTIFICATION_CLASS_RESPONSE (`null` above), the name of the activity that is opened when the user taps the notification - example: "com.company.activities.SearchActivity" (null opens the app's launcher activity).

The `execute` method is called when a Fanmeter notification is tapped. On Android, tapping either the push displayed by the system or the local notification launched by `launchFanmeterNotification` opens your app's launcher activity (unless a NOTIFICATION_CLASS_RESPONSE is set) with the notification data as the intent's extras, so a single handler covers both. It does not open any view itself - its callback returns a `(code, eventId)` pair and, when `eventId` is not null, your app is responsible for opening the Fanzone view for that event (see the [Fanzone Web View](#fanzone-web-view) section).

Call it both when the activity is created, for when the tap launched the app, and in `onNewIntent`, for when the activity was already running. Since the Fanzone needs the auth token, on creation call it once `initialize` has returned. When the activity was not opened by a Fanmeter notification, `execute` just returns an error code (e.g., -91) and nothing happens.

```kotlin
import com.pluggableai.fanmeter.EntryPoint

class MainActivity : ComponentActivity() {

    // ...

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        // ...
        EntryPoint.initialize(applicationContext, initializeDataMap, FANMETER_LOG) { token ->
            authToken = token
            // The app may have been launched by a tap on a Fanmeter notification.
            handleFanmeterNotification(intent)
        }
    }

    override fun onNewIntent(intent: Intent) {
        super.onNewIntent(intent)
        setIntent(intent)
        // A Fanmeter notification was tapped while the activity was already running.
        handleFanmeterNotification(intent)
    }

    private fun handleFanmeterNotification(intent: Intent?) {
        EntryPoint.execute(
            this,
            intent?.extras,
            null
        ) { code, eventId ->
            Log.d(TAG, "EntryPoint.execute: $code, $eventId")
            // `execute` does not open any view - when an eventId is returned, your app
            // is responsible for opening the Fanzone view for that event.
            if(eventId != null) {
                openFanzoneForEvent(eventId)
            }
        }
    }
}
```

Where:
  * this, is the activity context;
  * extras (`intent?.extras`), the data received with the tapped notification;
  * NOTIFICATION_CLASS_RESPONSE (`null` above), the name of the activity to open - example: "com.company.activities.SearchActivity". When set, `execute` opens that activity itself and returns a null `eventId`.

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

You are also required to subscribe the user to a FCM topic so that the user can receive Fanmeter notifications. This can be done, for example, as follows:

```kotlin
// Subscribe to a topic.
Firebase.messaging.subscribeToTopic("football_senior").addOnCompleteListener { task ->
    Log.d(TAG, "Subscribed to topic football_senior: ${task.isSuccessful}")
}
```

## Build your own UI

If you want full control and implement your own UI, this library exposes three methods, that must be called as demonstrated below. In particular:
  * `startService`, starts the Fanmeter service that enables Fanmeter for your client's device during a particular event;
  * `stopService`, stops the Fanmeter service. The service will, still, terminate automatically as soon as the event ends. This returns 1, if success, otherwise an error code;
  * `isServiceRunning`, used to check the current status of the Fanmeter service. Returns whether it is running and, if so, the event being tracked.

The `startService` method is used to start the Fanmeter service. This method accepts an eventId, that should be obtained using the `getEventData` method and should be called as follows (associated, for example, to a particular button). Pass the current `Activity` as context: the SDK uses it to ask the user for any missing location permission and to turn on the GPS. With the application context it cannot ask, and returns -80 or -81 instead.

```kotlin
// e.g. on a button click.
EntryPoint.startService(
    this@MainActivity,
    EVENT_ID,
    null
) {
    Log.d(TAG, "EntryPoint.startService: $it")
}
```

Where:
  * this@MainActivity, is the current activity;
  * EVENT_ID, the id of the event the user will participate when the start service is called;
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

```kotlin
EntryPoint.stopService(applicationContext) {
    Log.d(TAG, "EntryPoint.stopService: $it")
}
```

Finally, the `isServiceRunning` method is used to check the current status of the Fanmeter service. This returns `isRunning`, true if the service is running, and `eventId`, the id of the event being tracked, which is only set while it is running.

```kotlin
EntryPoint.isServiceRunning(applicationContext) { isRunning, eventId ->
    Log.d(TAG, "EntryPoint.isServiceRunning: $isRunning, $eventId")
}
```

## Additional info

Other important methods are to **request user permission** to be able to **send notifications**. Also, **get the user token and listen for changes** to get the user FCM_TOKEN and update it when it changes.

In particular, to request the push notification permission (only required from Android 13 onwards):

```kotlin
private val requestPermissionLauncher = registerForActivityResult(
    ActivityResultContracts.RequestPermission()
) { isGranted ->
    Log.d(TAG, "Push notification permission granted: $isGranted")
}

private fun askNotificationPermission() {
    if(Build.VERSION.SDK_INT < Build.VERSION_CODES.TIRAMISU) return
    if(ContextCompat.checkSelfPermission(this, Manifest.permission.POST_NOTIFICATIONS) != PackageManager.PERMISSION_GRANTED) {
        requestPermissionLauncher.launch(Manifest.permission.POST_NOTIFICATIONS)
    }
}
```

And, if you need to know the FCM token ID associated with each user device:

```kotlin
// Get user's FCM token.
FirebaseMessaging.getInstance().token.addOnCompleteListener { task ->
    if(task.isSuccessful) {
        Log.d(TAG, "TOKEN is: ${task.result}")
    }
}
```

```kotlin
// Listen to whether the token changes (in your FirebaseMessagingService).
override fun onNewToken(token: String) {
    Log.d(TAG, "TOKEN refreshed to: $token")
}
```

In addition, **prevent multiple openings of the Fanzone view** so that it is only presented when no previous one is on screen. The `openFanzone` helper from the [Fanzone Web View](#fanzone-web-view) section already does it, with the `FLAG_ACTIVITY_SINGLE_TOP` flag, which keeps the Fanzone that is already on top instead of opening a second one.

For more info visit [our website](https://pluggableai.xyz/) or give us feedback to <info@pluggableai.xyz>.
