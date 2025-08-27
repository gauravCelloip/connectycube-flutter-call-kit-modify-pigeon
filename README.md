# ConnectyCube Flutter Call Kit Plugin

[![Stand With Ukraine](https://raw.githubusercontent.com/vshymanskyy/StandWithUkraine/main/banner2-direct.svg)](https://stand-with-ukraine.pp.ua)

A Flutter plugin for displaying a call screen when the app is in the background or terminated. It provides a comprehensive solution for implementing background call features in your app, including token management and displaying the incoming call screen.

## Features

- **Device Token Access:** Get FCM token for Android and VoIP token for iOS.
- **Token Refresh Notifications:** Notifies the app about token updates via a callback.
- **Incoming Call Screen:** Displays a native-style incoming call screen, even when the app is terminated.
- **User Action Handling:** Notifies the app of user actions like accepting, rejecting, or muting a call.
- **Manual Call Management:** Provides methods to manually show and manage the incoming call screen.
- **Call State Information:** Retrieve data about the current call during a session.
- **Customization:** Customize the ringtone, app icon, and accent color (Android).
- **Android 14 Support:** Includes features for checking and requesting `USE_FULL_SCREEN_INTENT` permission.

## Supported Platforms

- Android
- iOS

## Screenshots

<p float="left">
  <kbd><img alt="Flutter P2P Calls code sample, incoming call in background Android" src="https://developers.connectycube.com/images/code_samples/flutter/background_call_android.png" height="440" /></kbd>
  <kbd><img alt="Flutter P2P Calls code sample, incoming call locked Android" src="https://developers.connectycube.com/images/code_samples/flutter/background_call_android_locked.png" height="440" /></kbd>
  <kbd><img alt="Flutter P2P Calls code sample, incoming call in background iOS" src="https://developers.connectycube.com/images/code_samples/flutter/background_call_ios.PNG" height="440" /></kbd>
  <kbd><img alt="Flutter P2P Calls code sample, incoming call locked iOS" src="https://developers.connectycube.com/images/code_samples/flutter/background_call_ios_locked.PNG" height="440" /></kbd>
</p>

## Installation

Add the following to your `pubspec.yaml` file:

```yaml
dependencies:
  connectycube_flutter_call_kit: ^2.8.0 # Use the latest version
```

Then, run `flutter pub get`.

## Platform Configuration

### Android

1.  **Google Services:**
    - Download your `google-services.json` file from your Firebase project.
    - Place it in the `android/app/` directory of your project.

2.  **build.gradle:**
    - Add the following line to the end of your `android/app/build.gradle` file:
      ```groovy
      apply plugin: 'com.google.gms.google-services'
      ```

3.  **Permissions:**
    - For apps targeting `targetSdkVersion 31` or higher, you may need to request the `SYSTEM_ALERT_WINDOW` permission to start the app from the background. You can use a plugin like `permission_handler` for this.
    - For apps targeting `targetSdkVersion 33` or higher, you must request the `POST_NOTIFICATIONS` permission.

### iOS

1.  **Info.plist:**
    - Add the following to your `ios/Runner/Info.plist` file:
      ```xml
      <key>UIBackgroundModes</key>
      <array>
        <string>remote-notification</string>
        <string>voip</string>
      </array>
      ```

## Usage

### Initialization

Initialize the plugin and set up your call event listeners.

```dart
ConnectycubeFlutterCallKit.instance.init(
  onCallAccepted: _onCallAccepted,
  onCallRejected: _onCallRejected,
  onCallIncoming: _onCallIncoming,
);

Future<void> _onCallAccepted(CallEvent callEvent) async {
  // Handle accepted call
}

Future<void> _onCallRejected(CallEvent callEvent) async {
  // Handle rejected call
}

Future<void> _onCallIncoming(CallEvent callEvent) async {
  // Handle incoming call
}
```

### Getting Tokens

- **Get the current token:**
  ```dart
  ConnectycubeFlutterCallKit.getToken().then((token) {
    // Use the token for push notification subscriptions
  });
  ```

- **Listen for token refreshes:**
  ```dart
  ConnectycubeFlutterCallKit.onTokenRefreshed = (token) {
    // Resubscribe with the new token
  };
  ```

### Showing the Incoming Call UI

You can show the call notification manually:

```dart
CallEvent callEvent = CallEvent(
  sessionId: 'your_session_id',
  callType: 1, // 1 for video, 0 for audio
  callerId: 123,
  callerName: 'John Doe',
  opponentsIds: {456, 789},
  userInfo: {'custom_param': 'value'},
);

ConnectycubeFlutterCallKit.showCallNotification(callEvent);
```

### Handling Background Call Actions (Android)

For calls in a terminated state on Android, define top-level functions:

```dart
@pragma('vm:entry-point')
Future<void> onCallAcceptedWhenTerminated(CallEvent callEvent) async {
  // Handle accepted call
}

@pragma('vm:entry-point')
Future<void> onCallRejectedWhenTerminated(CallEvent callEvent) async {
  // Handle rejected call
}

// Register these handlers
ConnectycubeFlutterCallKit.onCallAcceptedWhenTerminated = onCallAcceptedWhenTerminated;
ConnectycubeFlutterCallKit.onCallRejectedWhenTerminated = onCallRejectedWhenTerminated;
```

### Managing Call State

- **Report call accepted:**
  ```dart
  ConnectycubeFlutterCallKit.reportCallAccepted(sessionId: 'your_session_id');
  ```

- **Report call ended:**
  ```dart
  ConnectycubeFlutterCallKit.reportCallEnded(sessionId: 'your_session_id');
  ```

- **Get call data:**
  ```dart
  ConnectycubeFlutterCallKit.getCallData(sessionId: 'your_session_id').then((data) {
    // Use call data
  });
  ```

- **Clear call data:**
  ```dart
  ConnectycubeFlutterCallKit.clearCallData(sessionId: 'your_session_id');
  ```

### Customization

Customize the call notification appearance:

```dart
ConnectycubeFlutterCallKit.instance.updateConfig(
  ringtone: 'custom_ringtone',
  icon: 'app_icon',
  color: '#07711e',
);
```

### Platform-Specifics

#### Android

- **Lock Screen Visibility:**
  ```dart
  // Show app on lock screen
  ConnectycubeFlutterCallKit.setOnLockScreenVisibility(isVisible: true);

  // Hide app on lock screen
  ConnectycubeFlutterCallKit.setOnLockScreenVisibility(isVisible: false);
  ```

- **Android 14+ Full-Screen Intent:**
  ```dart
  bool canUse = await ConnectycubeFlutterCallKit.canUseFullScreenIntent();
  if (!canUse) {
    ConnectycubeFlutterCallKit.provideFullScreenIntentAccess();
  }
  ```

#### iOS

- **Mute/Unmute Call:**
  ```dart
  ConnectycubeFlutterCallKit.onCallMuted = (mute, uuid) {
    // Handle mute state change
  };

  // Report mute state from your app
  ConnectycubeFlutterCallKit.reportCallMuted(sessionId: 'your_session_id', muted: true);
  ```

## Push Notifications

To automatically display the incoming call screen via push notifications, send a payload with the following parameters:

```json
{
  "message": "Incoming Video call",
  "call_type": 1,
  "session_id": "your_session_id",
  "caller_id": 123,
  "caller_name": "John Doe",
  "call_opponents": "456,789",
  "signal_type": "startCall",
  "ios_voip": 1
}
```

To hide the notification, send a `signal_type` of `endCall` or `rejectCall`.

## Example

For a complete example, see the [P2P Calls code sample](https://github.com/ConnectyCube/connectycube-flutter-samples/tree/master/p2p_call_sample).

## Contributing

Contributions are welcome! If you have a feature request or bug report, please [open an issue](https://github.com/ConnectyCube/connectycube-flutter-call-kit/issues).

## License

This project is licensed under the [LICENSE](LICENSE) file.
