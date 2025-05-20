Criteria:
- On the same device/app, there should be possibility for different users to login. Each user notifications should be isolated from other users on that device/app.
- The same user can login on different devices/apps. The user should receive notification on all devices/apps he/she is logged in.

Pseudo code that guarantee the above criteria are meet
```dart
String? oldToken = await FirebaseMessaging.instance.getToken(); // get current token on device-app
await FirebaseMessaging.instance.deleteToken(); // terminale current device-app token
String? newToken = await FirebaseMessaging.instance.getToken(); // Request new device-app token
await saveTokenOnBackend(userId, oldToken, newToken); // replace oldToken(if exists) with newToken or insert newToken(if oldToken not exists)
```

```dart
onUserSignup(userId) // generate and save newToken
onUserLogin(userId) // terminate oldToken, generate new token. Replace oldToken with newToken or insert newToken(if old doesn't exist)
onUserLogout(userId) // terminate and remove token
onUserDelete(userId) // terminate and remove all user tokens
```
