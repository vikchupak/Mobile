# Flutter

### key.properties

`android/key.properties` file points to Java Key Store

```text
storeFile=/Users/admin/Development/keystore.jks // Location of .jks
storePassword=XXXX // Password for .jks
keyAlias=upload // key name inside .jks
keyPassword=XXXX // Password for the key name
```

### SHA-1 and SHA-256

```bash
keytool -list -v \
  -keystore /Users/admin/DevelopInstall/hh-keystore.jks \
  -alias upload
```

- It will prompt for the keystore password. Enter the keystore password.
- It prints detailed information about the key stored under the alias upload inside your keystore file.
  - Including SHA-1 and SHA-256 fingerprints.

### Firebase

1. **Copy the SHA1 and SHA-256** from the output.
2. Go to:
   * [Firebase Console](https://console.firebase.google.com/) → Project Settings → General → Your Android app
3. **Paste the SHA-1 and SHA-256 fingerprints**
4. If using Firebase: **Download the updated `google-services.json`** and replace it in `android/app/` OR `android/app/src/stage`
5. Then:

```bash
flutter clean
flutter pub get
flutter build apk --release --flavor stage --dart-define=ENVIRONMENT=Dev
```
