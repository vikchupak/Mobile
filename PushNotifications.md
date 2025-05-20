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

userId, token, createAt, updateAt

### FCM

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
