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
flutter build apk --release --flavor stage --dart-define=ENVIRONMENT=Stage
```

Output `build/app/outputs/flutter-apk/app-stage-release.apk`

### Prod vs stage vs dev

- Using the same signing key for both your production and test builds is **common and recommended practice**.

### iOS build command

```bash
flutter build ipa --release --flavor stage --dart-define=ENVIRONMENT=Stage
```

---

For Android, we need:
 - Actual `android/app/prod/google-services.json` and `android/app/stage/google-services.json` files.
 - Correct `googleWebServerClientId` in files
    - `assets/enviroment_values/production.json`
    - `assets/enviroment_values/dev.json`
    - `assets/enviroment_values/stage.json`
  
For iOS, we need:
- Actual `ios/Runner/Firebase/prod/GoogleService-Info.plist` and `ios/Runner/Firebase/stage/GoogleService-Info.plist` files.
- Correct `googleClientId` in files
    - `assets/enviroment_values/production.json`
    - `assets/enviroment_values/dev.json`
    - `assets/enviroment_values/stage.json`
 - Correct REVERSED `googleClientId` in `ios/Runner/Info.plist` file.
