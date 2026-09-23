# Secure Support User (Phase 1)

Android app for the Secure Support system. It is a separate project from `secure-support-admin`;
both talk to the same Firebase project.

| | |
|---|---|
| App name | Secure Support User |
| applicationId | `com.secure.user` (must match google-services.json exactly) |
| Kotlin namespace | `com.secure.user` |
| Firebase project | `support-e7981` |
| Realtime Database | https://support-e7981-default-rtdb.firebaseio.com |

## Status

**Phase 1 only.** Nothing from later phases is implemented yet.

| Feature | Status |
|---|---|
| Project structure, Gradle, Compose, Navigation | Done (not yet compiled, see below) |
| Theme (dark / red), Splash, Home, History, Settings | Done |
| Device ID generation (`SSU-XXXX-XXXX-XXXX`, SecureRandom, stored locally) | Done |
| Online / Offline indicator (device network state) | Done |
| Firebase initialised from google-services.json | Done (logs project id at startup) |
| Approval status | **Placeholder**: always PENDING until Phase 2 |
| Device registration in Firestore, Security Rules, FCM | Phase 2 |
| Mirror request screen, MediaProjection consent, foreground service | Phase 3 |
| WebRTC signaling and live stream | Phase 4 |
| History data, activity logs, error handling polish | Phase 5 |
| Live Screen Mirror screen | Phase 3 (needs the real session) |

The "Live Support" button on Home currently only opens an explanatory dialog. Sessions are started by
the administrator's request, never by this button alone.

## google-services.json

`app/google-services.json` is the file you uploaded, unchanged. It contains **both** clients
(`com.secure.user` and `com.secure.admin`), and the two files you uploaded are identical. The
Google Services Gradle plugin picks the client that matches `applicationId`, so the same file is used in
both projects. The `applicationId` in `app/build.gradle.kts` must stay exactly `com.secure.user`.

## Open and build

1. Install Android Studio or Android Code Studio with JDK 17 and Android SDK 33.
2. **File > Open** and choose this folder (the one containing `settings.gradle.kts`).
3. Android Studio reads `gradle/wrapper/gradle-wrapper.properties` and uses Gradle 7.4.2.
   `gradlew` and `gradlew.bat` are included. `gradle/wrapper/gradle-wrapper.jar` (a binary) is not — see
   "Gradle Wrapper: one important limitation" below for the three ways to get a working one.
4. Sync, then **Build > Build APK(s)** or run `./gradlew assembleDebug`.
   Output: `app/build/outputs/apk/debug/app-debug.apk`.

Versions (chosen for Gradle 7.4.x, as used by Android Code Studio): AGP 7.3.1, Gradle 7.4.2, Kotlin 1.8.22,
Compose compiler 1.4.8, Compose BOM 2023.06.01, Firebase BoM 32.3.1, google-services plugin 4.3.15,
minSdk 26, compileSdk/targetSdk 33, Java 17.

## Testing Phase 1

- First launch: splash animation, then Home with a generated Device ID. Close and reopen the app: the ID
  must stay the same.
- Turn on airplane mode: Connection Status changes to Offline (gray).
- Settings > Reset Device Registration: a new Device ID is generated.
- Logcat, tag `SecureSupport`: `Firebase ready, project = support-e7981`.

## Security notes

- `android:allowBackup="false"` so a backup restore cannot copy a Device ID to another phone.
- No Firebase Admin SDK credentials or service-account keys are, or will ever be, in this app.
- The API key in google-services.json is not a secret; access control comes from Firebase Auth and
  Firestore Security Rules (Phase 2). Restrict the key to these two apps in Google Cloud Console.

## Building in the cloud (GitHub Actions)

If the project cannot be built on the phone, push it to a GitHub repository. The workflow in
`.github/workflows/build-debug-apk.yml` builds the debug APK on every push to `main`/`master`, or on
demand (Actions tab > Build Debug APK > Run workflow). Download the APK from the run's **Artifacts**
section (`secure-support-user-debug-apk`).

- The repository root must be this folder (`settings.gradle.kts` at the top level).
- Use a **private** repository: google-services.json contains your Firebase API key.
- If the build fails, open the failed step in the Actions log and send the error text.

## Gradle Wrapper: one important limitation

`gradlew` and `gradlew.bat` (plain text scripts) are included and are complete. `gradle/wrapper/gradle-wrapper.jar`
is a compiled binary. Generating a **genuine** one requires either a Java compiler or a network
connection to Gradle's servers — this assistant's build environment has neither, so that exact file is
not included in this zip. A fabricated placeholder jar would just fail every build silently, which is
worse than being upfront about it. Three real fixes, pick whichever suits you:

1. **Easiest — let GitHub Actions do it.** `.github/workflows/build.yml` now installs the official
   Gradle 7.4.2 distribution on GitHub's server and runs `gradle wrapper --gradle-version 7.4.2` before
   building, which writes a genuine `gradle-wrapper.jar` as part of that run. `./gradlew assembleDebug`
   then runs normally. You do not need to do anything extra for this — just push the project, and also
   grab the **gradle-wrapper-jar** artifact from that same Actions run if you want a copy for step 3.
2. **Android Studio (PC/Mac) will offer to fix it for you.** Opening this project with `gradle-wrapper.jar`
   missing triggers a "Gradle wrapper is missing" prompt with a one-click **OK/Generate** action, or use
   **File > Sync Project with Gradle Files**.
3. **Command line, if you have any Gradle install (any version) or Android Studio's bundled one:**
   run `gradle wrapper --gradle-version 7.4.2 --distribution-type bin` once inside this folder, then
   commit the resulting `gradle/wrapper/gradle-wrapper.jar` — after that, `./gradlew` is fully standalone
   and this whole section becomes unnecessary.

This affects only local/offline command-line builds. GitHub Actions (option 1) already works today
without you doing anything.

