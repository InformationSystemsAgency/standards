Appendix: Mobile Universal Links & App Links for Hartak.am
===========================================================

**Integration Document – Mobile Appendix** \
April 2025 v1.0

## Table of Contents

- [1. Scope](#1-scope)
- [2. Service Availability Scenarios](#2-service-availability-scenarios)
- [3. Common Contract for Mobile Apps](#3-common-contract-for-mobile-apps)
- [4. iOS – Universal Links](#4-ios---universal-links)
  - [4.1. Requirements](#41-requirements)
  - [4.2. AASA file on the service provider host](#42-aasa-file-on-the-service-provider-host)
  - [4.3. iOS app configuration (Xcode)](#43-ios-app-configuration-xcode)
  - [4.4. Handling Universal Links in the app](#44-handling-universal-links-in-the-app)
- [5. Android – App Links](#5-android---app-links)
  - [5.1. Requirements](#51-requirements)
  - [5.2. Digital Asset Links file on the service provider host](#52-digital-asset-links-file-on-the-service-provider-host)
  - [5.3. AndroidManifest – HTTP(S) deep links (App Links)](#53-androidmanifest---https-deep-links-app-links)
  - [5.4. Handling the link in the Activity](#54-handling-the-link-in-the-activity)
- [6. Authentication Handoff to Mobile Apps](#6-authentication-handoff-to-mobile-apps)
  - [6.1. Can Hartak/YesEm authentication be passed to a mobile app directly?](#61-can-hartakyesem-authentication-be-passed-to-a-mobile-app-directly)
  - [6.2. Recommended secure approaches](#62-recommended-secure-approaches)
- [7. Testing Checklist](#7-testing-checklist)

## 1. Scope

This appendix describes **how mobile apps can consume links originating from Hartak.am** using modern HTTPS links:

- **Canonical HTTPS unique service links** (as defined in the main document).

The goal is that when a user taps one of these links, and the official mobile app is installed, the app can **open directly to the right screen**; otherwise, the link should gracefully fall back to the browser and follow the normal web flow through YesEm.

Throughout the examples we will assume:

- **Service provider base URL**: `https://serviceportal.am`
- **Canonical unique service link pattern**: `https://serviceportal.am/services/{serviceId}`

Where:

- `serviceId` is the identifier from the National Service Catalog (see [main document](./en.md)).

Adapt these domains, schemes, and IDs to your real deployment.

Important ownership rule: domain verification files must be hosted on **service provider domains** that serve the links, not on `hartak.am`.

Platform split:

- **iOS** verification uses only `apple-app-site-association` (AASA).
- **Android** verification uses only `/.well-known/assetlinks.json` (Digital Asset Links).

## 2. Service Availability Scenarios

Service providers can support one of the following models:

1. **Web only**
   - The canonical HTTPS link opens the browser flow only.
   - User continues through YesEm and then to the provider web service.
   - No app-link setup is required.

2. **Mobile app only** (iOS, Android, or both)
   - Keep the same canonical HTTPS link format from Hartak (`/services/{serviceId}`).
   - If app is installed and domain verification is configured, the app opens directly.
   - iOS requires AASA; Android requires `assetlinks.json`.
   - If app is not installed, open a provider-controlled web fallback page that explains:
     - install app, and/or
     - continue in browser if a web flow exists later.

3. **Web and mobile**
   - Keep one canonical HTTPS link (`/services/{serviceId}`) as the public entry point.
   - Installed app users are routed directly into native screens via Universal Links/App Links.
   - Non-installed users continue in browser through YesEm and the provider web flow.

## 3. Common Contract for Mobile Apps

To keep a consistent contract between web and mobile:

- **Single link policy**: Hartak stores and publishes only one canonical link format: `/services/{serviceId}`.
- Mobile apps should:
  - Accept canonical HTTPS links under `/services/{serviceId}`.
  - They may parse `serviceId`, or send the full URL to backend and let backend resolve routing/ownership.
  - Optionally use **extra query parameters** (e.g. `?source=hartak`) for analytics only.
  - If an unknown or unsupported link is received, open a generic screen or gracefully fall back to the browser.

## 4. iOS – Universal Links

### 4.1. Requirements

- iOS 9+.
- The service provider domain (e.g. `serviceportal.am`) must host an **`apple-app-site-association` (AASA)** file over HTTPS.
- The iOS app must enable the **Associated Domains** capability with the same domain.
- Android `assetlinks.json` is not used by iOS.

### 4.2. AASA file on the service provider host

Place `apple-app-site-association` on the service provider host:

`https://serviceportal.am/apple-app-site-association`
  - No file extension.
  - Content type `application/json`.
  - Must be served by the service provider domain that appears in the link.

Example content:

```json
{
  "applinks": {
    "apps": [],
    "details": [
      {
        "appID": "ABCDE12345.com.example.taxapp",
        "paths": [
          "/services/*"
        ]
      }
    ]
  }
}
```

- **`appID`** = `<TEAM_ID>.<BUNDLE_ID>`, e.g. `ABCDE12345.com.example.taxapp`.
- **`paths`** controls which URLs open in the app; here we target canonical links under `/services/`.

### 4.3. iOS app configuration (Xcode)

1. Open the iOS app in Xcode.
2. Select the app **target → Signing & Capabilities → + Capability → Associated Domains**.
3. Add:

   - `applinks:serviceportal.am`

4. Rebuild and install the app on a device.

### 4.4. Handling Universal Links in the app

Use official Apple documentation for implementation details:

- Supporting Universal Links in your app:  
  <a href="https://developer.apple.com/documentation/xcode/supporting-universal-links-in-your-app" target="_blank" rel="noopener noreferrer">https://developer.apple.com/documentation/xcode/supporting-universal-links-in-your-app</a>
- `UISceneDelegate` continuation API (`scene(_:continue:)`):  
  <a href="https://developer.apple.com/documentation/uikit/uiscenedelegate/scene(_:continue:)" target="_blank" rel="noopener noreferrer">https://developer.apple.com/documentation/uikit/uiscenedelegate/scene(_:continue:)</a>

## 5. Android – App Links

### 5.1. Requirements

- Android 6.0+ (for best App Links behavior).
- The service provider domain must be accessible over HTTPS.
- For verified App Links, host **Digital Asset Links** at `https://serviceportal.am/.well-known/assetlinks.json`.
- iOS AASA is not used by Android.

### 5.2. Digital Asset Links file on the service provider host

Place `assetlinks.json` at: `https://serviceportal.am/.well-known/assetlinks.json`
Must be served by the same service provider domain used in links.

Example content:

```json
[
  {
    "relation": ["delegate_permission/common.handle_all_urls"],
    "target": {
      "namespace": "android_app",
      "package_name": "com.example.taxapp",
      "sha256_cert_fingerprints": [
        "12:34:56:78:90:AB:CD:EF:12:34:56:78:90:AB:CD:EF:12:34:56:78:90:AB:CD:EF:12:34:56:78:90:AB:CD:EF"
      ]
    }
  }
]
```
### 5.3. AndroidManifest – HTTP(S) deep links (App Links)

In your app’s `AndroidManifest.xml`, declare an `Activity` that can handle the unique service links:

```xml
<!-- AndroidManifest.xml -->
<manifest package="com.example.serviceapp"
    xmlns:android="http://schemas.android.com/apk/res/android">

    <application
        android:label="Service App"
        android:icon="@mipmap/ic_launcher">

        <activity
            android:name=".ServiceEntryActivity"
            android:exported="true">

            <!-- App Link for unique service links -->
            <intent-filter android:autoVerify="true">
                <action android:name="android.intent.action.VIEW" />

                <category android:name="android.intent.category.DEFAULT" />
                <category android:name="android.intent.category.BROWSABLE" />

                <data
                    android:scheme="https"
                    android:host="serviceportal.am"
                    android:pathPrefix="/services" />
            </intent-filter>

        </activity>

    </application>
</manifest>
```

This enables Android to offer your app as a handler for canonical URLs like:

- `https://serviceportal.am/services/12345`

If App Links verification is correctly configured, Android can open the app directly without showing a chooser dialog.

### 5.4. Handling the link in the Activity

Use official Android documentation for implementation details:

- Android App Links overview and implementation guide:  
  <a href="https://developer.android.com/training/app-links" target="_blank" rel="noopener noreferrer">https://developer.android.com/training/app-links</a>
- Add intent filters and configure web domain association:  
  <a href="https://developer.android.com/training/app-links/add-applinks" target="_blank" rel="noopener noreferrer">https://developer.android.com/training/app-links/add-applinks</a>
- Verify Android App Links and test behavior:  
  <a href="https://developer.android.com/training/app-links/verify-android-applinks" target="_blank" rel="noopener noreferrer">https://developer.android.com/training/app-links/verify-android-applinks</a>

## 6. Authentication Handoff to Mobile Apps

### 6.1. Can Hartak/YesEm authentication be passed to a mobile app directly?

Not as a raw web session in a safe/portable way. Browser cookies and app sessions are different contexts, so the app should not receive or reuse browser session cookies directly.

### 6.2. Recommended secure approaches

1. **Authenticate directly via YesEm inside the mobile app**
   - User opens the app from the canonical link.
   - App starts YesEm login.
   - YesEm returns authorization code to the app callback.

2. **Continue from web to app after YesEm login**
   - User starts from the canonical HTTPS service link.
   - Redirect path reaches YesEm in browser.
   - After successful login, authorization result is returned to the mobile app (Universal Link/App Link or callback URI).
   - App exchanges the authorization code and creates its own app session.

## 7. Testing Checklist

Before going live:

- **Domain configuration**
  - iOS: AASA file deployed and reachable at `https://serviceportal.am/apple-app-site-association` on the service provider host.
  - Android: Digital Asset Links file deployed and reachable at `https://serviceportal.am/.well-known/assetlinks.json` on the same service provider host.
  - No iOS/Android verification file is hosted on `hartak.am` unless links are actually opened from `hartak.am`.

- **iOS**
  - Install the app on a real device (not only the simulator).
  - Tap canonical links from:
    - Notes, email, or browser.
  - Confirm that:
    - Links under `/services/{serviceId}` open the app.
    - Links outside `/services/*` open in the browser only.

- **Android**
  - Install the app from a signed build.
  - Tap canonical links from:
    - Email, messaging apps, browser.
  - Verify:
    - App is offered as a handler (or opens directly if verified).
    - Correct screen opens based on backend-resolved service context.

- **Fallback**
  - Uninstall the app and tap the same links.
  - Confirm that the user is correctly redirected through YesEm and into the Service Portal via the browser.

