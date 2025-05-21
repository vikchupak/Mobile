### Criteria
- On the same device/app, there should be possibility for different users to login. Each user notifications should be isolated from other users on that device/app.
- The same user can login on different devices/apps. The user should receive notification on all devices/apps he/she is logged in.

Pseudo code that guarantee the above criteria are meet
```dart
String? oldToken = await FirebaseMessaging.instance.getToken(); // get current device-app token
await FirebaseMessaging.instance.deleteToken(); // terminale current device-app token
String? newToken = await FirebaseMessaging.instance.getToken(); // Request new device-app token
await saveTokenOnBackend(userId, oldToken, newToken); // replace oldToken(if exists) with newToken or insert newToken(if old doesn't exist)
```

```dart
onUserSignup(userId) // generate and save newToken
onUserLogin(userId) // terminate oldToken, generate new token. Replace oldToken with newToken or insert newToken(if old doesn't exist)
onUserLogout(userId) // terminate and remove token
onUserDelete(userId) // terminate and remove all user tokens
```

### Token refresh

`FirebaseMessaging.onTokenRefresh` does not return the old token — it only gives you the new FCM token.

So to the handle the refresh, we need to know prev token.

- On login, we need to store "old" token to local starage, and subscribe to `onTokenRefresh`.
- On refresh, we just replace oldToken from local storage with newToken(in backend and local storage).

### Table schema

userId, token, updatedAt

# About FCM

**FCM token is unique per *device + app installation*** pair.

## ✅ FCM Token Uniqueness

| Factor                    | Included in Token Identity |
| ------------------------- | -------------------------- |
| Device hardware           | ✅ Yes                      |
| App install (Instance ID) | ✅ Yes                      |
| User account              | ❌ No (not tied directly)   |

---

### 🔄 What causes the FCM token to change?

| Action                      | Does Token Change? | Why?                                                |
| --------------------------- | ------------------ | --------------------------------------------------- |
| App reinstalled             | ✅ Yes              | New instance ID                                     |
| App cleared data            | ✅ Yes              | Token is regenerated                                |
| User logs out/logs in       | ❌ No               | Token is user-agnostic                              |
| OS-level permission changed | ❌/⚠️ Sometimes     | May change on some Android/iOS versions             |
| Token refresh               | ✅ Yes              | Randomized by Firebase for various internal reasons |
| OS update or device change  | ✅ Yes              | Token depends on device fingerprint                 |

---

## 🧠 Key Implications

* **Same user on multiple devices?** → Each device has a unique token.
* **Same device, multiple user accounts in app?** → All users share the same token (unless you separate them).
* **Same app, but reinstalled?** → New token.

# OS level permissions & in-app user notification settings

## 🔐 1. **OS-Level Permissions**

> Controlled by the device (iOS/Android)

### ✅ What you get from FCM:

```dart
NotificationSettings settings = await FirebaseMessaging.instance.requestPermission();
```

### Use this to determine:

* Whether the user **allowed notifications at system level**
* Whether FCM will actually **deliver notifications**

| Status                   | Meaning             | Can store token? | Can send notification? |
| ------------------------ | ------------------- | ---------------- | ---------------------- |
| `authorized`             | Allowed by OS       | ✅ Yes            | ✅ Yes                  |
| `denied`                 | User denied prompt  | ❌ No             | ❌ No                   |
| `notDetermined`          | Not asked yet       | ⚠️ Not safe      | ❌ No                   |
| `provisional` (iOS only) | Temporarily allowed | ✅ Yes            | ⚠️ Limited delivery    |

---

## 📱 2. **In-App Notification Preferences**

> Controlled by the user inside your app (e.g. toggle switch)

This is a **custom boolean flag** you track yourself, e.g.:

```dart
sendNotifications: true // user allows notifications from your app
```

| Scenario                            | OS Permission | App Toggle | Token Stored | Send Notification |
| ----------------------------------- | ------------- | ---------- | ------------ | ----------------- |
| ✅ Allowed by OS + App               | ✅             | ✅          | ✅            | ✅                 |
| ❌ Allowed by OS but disabled in app | ✅             | ❌          | ✅            | ❌                 |
| ❌ Denied by OS                      | ❌             | Any        | ❌            | ❌                 |

---

## 🛠 Recommended Handling Logic

### 🔁 On app launch or login:

```dart
final settings = await FirebaseMessaging.instance.requestPermission();

if (settings.authorizationStatus == AuthorizationStatus.authorized) {
  final token = await FirebaseMessaging.instance.getToken();
  if (token != null) {
    await saveTokenToBackend(
      userId: userId,
      token: token,
      sendNotifications: userPrefersNotifications, // your toggle
    );
  }
} else {
  // OS denied notifications — maybe update backend to reflect it
}
```

---

## 🧠 Summary

| Factor                | Store Token? | Send Notification?         |
| --------------------- | ------------ | -------------------------- |
| OS permission granted | ✅ Yes        | Depends on app toggle      |
| OS permission denied  | ❌ No         | ❌ No                       |
| App toggle = false    | ✅ Yes        | ❌ No (respect user choice) |

# Asking for permissions vs Checking existing permissions

Asking for permissions
```dart
import 'package:firebase_messaging/firebase_messaging.dart';

Future<void> requestNotificationPermission() async {
  NotificationSettings settings = await FirebaseMessaging.instance.requestPermission();

  if (settings.authorizationStatus == AuthorizationStatus.authorized) {
    print('User granted permission');
  } else if (settings.authorizationStatus == AuthorizationStatus.provisional) {
    print('User granted provisional permission');
  } else {
    print('User declined or has not accepted permission');
  }
}
```

Checking existing permissions
```dart
Future<void> checkNotificationPermission() async {
  NotificationSettings settings = await FirebaseMessaging.instance.getNotificationSettings();

  switch (settings.authorizationStatus) {
    case AuthorizationStatus.authorized:
      print('Permission granted');
      break;
    case AuthorizationStatus.provisional:
      print('Provisional permission');
      break;
    case AuthorizationStatus.denied:
      print('Permission denied');
      break;
    case AuthorizationStatus.notDetermined:
      print('Permission not determined');
      break;
    case AuthorizationStatus.ephemeral:
      print('Ephemeral permission (iOS only)');
      break;
  }
}
```
