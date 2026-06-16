<div align="center">

<img src="https://img.shields.io/badge/Platform-Android-3DDC84?style=for-the-badge&logo=android&logoColor=white"/>
<img src="https://img.shields.io/badge/Language-Kotlin-7F52FF?style=for-the-badge&logo=kotlin&logoColor=white"/>
<img src="https://img.shields.io/badge/UI-Jetpack%20Compose-4285F4?style=for-the-badge&logo=jetpackcompose&logoColor=white"/>
<img src="https://img.shields.io/badge/Backend-Firebase-FFCA28?style=for-the-badge&logo=firebase&logoColor=black"/>
<img src="https://img.shields.io/badge/Architecture-MVVM-E91E8C?style=for-the-badge"/>

<br/><br/>

# 🛡️ SafeHer — Women Safety Android Application

**Final Year Engineering Project | IET Lucknow | 2024–25**

*A real-time women safety application that combines gesture recognition, live GPS tracking, hotspot mapping, and multi-channel emergency alerts into a single, reliable safety companion.*

</div>

---

## 📌 Table of Contents

- [Overview](#-overview)
- [Key Features](#-key-features)
- [Tech Stack](#-tech-stack)
- [Architecture](#-architecture)
- [Alert System](#-alert-system-core-objective)
- [Screens](#-screens)
- [Hotspot Mapping](#-hotspot-mapping)
- [Project Structure](#-project-structure)
- [Setup Guide](#-setup-guide)
- [Database Schema](#-database-schema)
- [Team](#-team)

---

## 🔍 Overview

SafeHer is an Android application designed for women's personal safety, built as a Final Year Project at IET Lucknow. The app provides real-time emergency response through a one-tap SOS system that simultaneously triggers SMS alerts, push notifications, and cloud persistence — ensuring help reaches the user through multiple channels even under poor network conditions.

The core objective is **reliable, multi-channel alert delivery**: when a woman is in danger, a single press of the SOS button notifies her emergency contacts via SMS, alerts all nearby SafeHer users via push notification, and logs the incident to a cloud dashboard for authorities.

---

## ✨ Key Features

| Feature | Description |
|---|---|
| 🚨 **One-Tap SOS** | Large emergency button with pulse animation — triggers 3 alert channels simultaneously |
| 📍 **Live GPS Tracking** | Real-time location updated every 5 seconds via foreground service, even when app is in background |
| 🗺️ **Hotspot Map** | Google Maps with color-coded danger zones near IET Lucknow, based on incident data |
| 📲 **Multi-Channel Alerts** | SMS (device SmsManager) + Fast2SMS API + FCM push notifications |
| 👥 **Nearby User Alerts** | FCM topic broadcast — all SafeHer users within range receive an alarm notification |
| 📞 **Emergency Contacts** | Add/manage contacts stored in Firestore; auto-alerted on SOS |
| 🔐 **Secure Auth** | Firebase Authentication (email + password) |
| 👤 **User Profile** | Editable profile with safety preference toggles |
| 🤲 **Gesture Recognition** | Camera-based gesture detection for hands-free SOS trigger *(ML module)* |

---

## 🛠️ Tech Stack

**Language & UI**
- Kotlin (100%) — modern, null-safe Android development
- Jetpack Compose — declarative UI with animations and state management
- Material Design 3 — dark theme with safety-focused color palette

**Architecture**
- MVVM (Model-View-ViewModel) with StateFlow
- Repository pattern for clean data access separation
- Coroutines + Flow for async operations

**Backend & Cloud**
- Firebase Authentication — secure user login and registration
- Cloud Firestore — user profiles, emergency contacts, SOS alert logs
- Firebase Realtime Database — live location sync
- Firebase Cloud Messaging (FCM HTTP v1) — push notifications via OAuth2 service account

**Device & Platform APIs**
- Android SmsManager — direct SMS from device SIM
- Fast2SMS REST API — backup SMS gateway (India)
- Google Maps SDK + Fused Location Provider API
- Android Foreground Service — background GPS tracking
- Accompanist Permissions — runtime permission handling

---

## 🏗️ Architecture

```
┌─────────────────────────────────────────────────────┐
│                    UI Layer                          │
│  Jetpack Compose Screens + Navigation Component      │
│  HomeScreen │ MapScreen │ ContactsScreen │ Profile   │
└──────────────────────┬──────────────────────────────┘
                       │ observes StateFlow
┌──────────────────────▼──────────────────────────────┐
│                 ViewModel Layer                      │
│              MainViewModel (MVVM)                    │
│         StateFlow<MainUiState> — single source       │
└────────────┬─────────────────────┬──────────────────┘
             │                     │
┌────────────▼──────────┐ ┌───────▼──────────────────┐
│   Repository Layer    │ │      Service Layer        │
│  FirebaseRepository   │ │  LocationSosService       │
│  (Firestore / Auth)   │ │  (Foreground GPS)         │
└────────────┬──────────┘ │  SafeHerFCMService        │
             │            │  (Push Notifications)     │
┌────────────▼──────────┐ └───────────────────────────┘
│   Utility Layer       │
│  SosManager           │
│  (SMS + FCM + API)    │
└───────────────────────┘
```

---

## 🚨 Alert System (Core Objective)

When the user presses SOS, three channels fire **simultaneously** in parallel coroutines:

```
User Presses SOS Button
         │
         ├──► 1. Android SmsManager
         │       └─► Direct SMS from device SIM to all emergency contacts
         │           Message: Name + Live Location Google Maps link + Phone
         │
         ├──► 2. Fast2SMS REST API (India backup SMS)
         │       └─► HTTP POST to fast2sms.com — sends even if SIM has issues
         │
         ├──► 3. FCM HTTP v1 (OAuth2) Topic Broadcast
         │       └─► Push notification to /topics/sos_alerts
         │           Received by ALL installed SafeHer users
         │           Loud alarm sound + vibration pattern
         │
         └──► 4. Firestore Write
                 └─► SOS alert document with timestamp, location, user details
                     Visible on authorities/admin dashboard in real-time
```

**FCM v1 Implementation** — uses OAuth2 service account (not deprecated server key):
- Service account JSON downloaded from Firebase Console → Project Settings → Service Accounts
- Access token fetched programmatically using Google Auth Library
- POST to `https://fcm.googleapis.com/v1/projects/{project-id}/messages:send`
- Token auto-refreshes every 60 minutes — no manual key rotation needed

---

## 📱 Screens

<table>
<tr>
<td align="center" width="20%">

**Login / Register**
Firebase Auth
Email + Password

</td>
<td align="center" width="20%">

**Home (SOS)**
Pulsing SOS button
Live location card
Nearby active alerts

</td>
<td align="center" width="20%">

**Map**
Google Maps
Color-coded hotspots
Live SOS pins

</td>
<td align="center" width="20%">

**Contacts**
Add emergency contacts
Relation tags
Firestore synced

</td>
<td align="center" width="20%">

**Profile**
Edit info
Safety toggles
Sign out

</td>
</tr>
</table>

---

## 🗺️ Hotspot Mapping

Three danger zones hardcoded near IET Lucknow, visualized with colored radius circles on Google Maps:

| # | Location | Coordinates | Risk Level | Incidents |
|---|---|---|---|---|
| 1 | Sitapur Road Crossing | 26.9124°N, 80.9674°E | 🔴 HIGH | 14 |
| 2 | Kursi Road Railway Underpass | 26.9005°N, 80.9810°E | 🔴 HIGH | 9 |
| 3 | IET Back Gate Area | 26.9065°N, 80.9733°E | 🟡 MEDIUM | 6 |

Each hotspot displays:
- Colored translucent circle (200m radius) on the map
- Incident count and risk level in marker snippet
- Detail card with safety advisory on tap

---

## 📂 Project Structure

```
SafeHer/
├── app/
│   ├── google-services.json              ← Firebase config (not committed)
│   └── src/main/
│       ├── AndroidManifest.xml           Permissions + service declarations
│       └── java/com/safeher/app/
│           ├── MainActivity.kt           Entry point + bottom nav
│           ├── ui/
│           │   ├── Navigation.kt         Sealed class route definitions
│           │   ├── theme/Theme.kt        Dark theme + brand colors
│           │   └── screens/
│           │       ├── AuthScreens.kt    Login + Register UI
│           │       ├── HomeScreen.kt     SOS button + location
│           │       ├── MapScreen.kt      Hotspot map + live alerts
│           │       ├── ContactsScreen.kt Contact management
│           │       └── ProfileScreen.kt  User profile + settings
│           ├── data/
│           │   ├── models/Models.kt      Data classes + hardcoded hotspots
│           │   └── repository/
│           │       └── FirebaseRepository.kt   All Firebase operations
│           ├── services/
│           │   ├── SafeHerFCMService.kt  Receive + display push notifications
│           │   └── LocationSosService.kt Foreground GPS tracker
│           ├── utils/
│           │   ├── AppConfig.kt          Central credentials file
│           │   └── SosManager.kt         Multi-channel SOS alert logic
│           └── viewmodel/
│               └── MainViewModel.kt      MVVM state + business logic
```

---

## ⚙️ Setup Guide

### Prerequisites
- Android Studio Hedgehog (2023.1.1) or newer
- JDK 11
- Android device / emulator (API 26+)
- Firebase account (free)
- Google Cloud Console account (for Maps API)

### Step 1 — Firebase Project Setup

1. Open [Firebase Console](https://console.firebase.google.com) → **Create Project**
2. Add Android app → Package name: `com.safeher.app`
3. Download **`google-services.json`** → place in `app/` folder

**Enable these Firebase services:**

| Service | Steps |
|---|---|
| Authentication | Build → Authentication → Sign-in method → Email/Password → Enable |
| Firestore | Build → Firestore Database → Create database → **Start in test mode** → Select region (asia-south1 for India) → **Database ID: leave as `(default)`** |
| Realtime Database | Build → Realtime Database → Create database → copy the URL |
| Cloud Messaging | Enabled automatically — go to Project Settings → Service Accounts |

> **Firestore Database ID question:** When Firebase asks for a Database ID during setup, just leave it as **`(default)`**. This is the standard single-database setup. Only advanced multi-database setups need a custom ID.

### Step 2 — FCM v1 Service Account (replaces old Server Key)

This is the new OAuth2-based method — no server key needed:

1. Firebase Console → ⚙️ **Project Settings** → **Service Accounts** tab
2. Click **"Generate new private key"** → **Generate Key**
3. A JSON file downloads — keep it safe, **do not commit to Git**
4. In `AppConfig.kt`, set the path to this file:

```kotlin
const val FCM_SERVICE_ACCOUNT_JSON = "safeher-firebase-adminsdk-xxxx.json"
// Place the JSON file in app/src/main/assets/ folder
```

The app reads this file at runtime to get a short-lived OAuth2 access token, then calls:
```
POST https://fcm.googleapis.com/v1/projects/YOUR_PROJECT_ID/messages:send
Authorization: Bearer <oauth2_token>
```

### Step 3 — Fill AppConfig.kt

Open `app/src/main/java/com/safeher/app/utils/AppConfig.kt`:

```kotlin
const val FIREBASE_PROJECT_ID  = "your-project-id"          // Firebase Console → Project Settings → General
const val FIREBASE_DB_URL      = "https://your-project-default-rtdb.firebaseio.com"
const val FAST2SMS_API_KEY     = "your_fast2sms_key"         // fast2sms.com → Dashboard → Dev API
```

### Step 4 — Google Maps API Key

1. [Google Cloud Console](https://console.cloud.google.com) → APIs & Services → Credentials
2. Create API Key → restrict to **Maps SDK for Android**
3. Enable: **Maps SDK for Android** and **Places API**
4. Paste key in `AndroidManifest.xml`:

```xml
<meta-data
    android:name="com.google.android.geo.API_KEY"
    android:value="AIzaSy_YOUR_KEY_HERE" />
```

### Step 5 — Build & Run

```bash
# Sync Gradle (File → Sync Project with Gradle Files)
# Connect Android device with USB debugging ON
# Run → Select device → Build & Install
```

On first launch the app requests: Location, SMS, Phone, and Notification permissions.

---

## 🗄️ Database Schema

**Firestore Collections:**

```
users/                          (collection)
  {uid}/                        (document)
    name:        "Priya Sharma"
    email:       "priya@email.com"
    phone:       "+919876543210"
    fcmToken:    "fcm_device_token"
    latitude:    26.9065
    longitude:   80.9733
    lastSeen:    1718612345000

    contacts/                   (subcollection)
      {contactId}/
        id:       "uuid"
        name:     "Asha Sharma"
        phone:    "+919123456789"
        relation: "Mother"

sos_alerts/                     (collection)
  {alertId}/                    (document)
    userId:    "uid"
    userName:  "Priya Sharma"
    phone:     "+919876543210"
    latitude:  26.9065
    longitude: 80.9733
    timestamp: 1718612345000
    resolved:  false
    mapsLink:  "https://maps.google.com/?q=26.9065,80.9733"
```

---

## 🔒 Permissions Used

| Permission | Purpose |
|---|---|
| `ACCESS_FINE_LOCATION` | Precise GPS for SOS location |
| `ACCESS_BACKGROUND_LOCATION` | GPS tracking when app is backgrounded |
| `FOREGROUND_SERVICE` | Keep location service alive |
| `SEND_SMS` | Send emergency SMS directly from device |
| `CALL_PHONE` | Emergency call capability |
| `POST_NOTIFICATIONS` | Show FCM alert notifications |
| `CAMERA` | Gesture recognition module |
| `INTERNET` | Firebase + FCM + Fast2SMS |

---

## 👥 Team

**IET Lucknow — B.Tech Final Year Project (2024–25)**

> Built with focus on real-world reliability — ensuring emergency alerts reach the right people through multiple independent channels, so no single point of failure can prevent help from arriving.

---

<div align="center">

**Made with ❤️ for safety | IET Lucknow**

![Kotlin](https://img.shields.io/badge/Kotlin-7F52FF?style=flat&logo=kotlin&logoColor=white)
![Firebase](https://img.shields.io/badge/Firebase-FFCA28?style=flat&logo=firebase&logoColor=black)
![Android](https://img.shields.io/badge/Android-3DDC84?style=flat&logo=android&logoColor=white)
![Compose](https://img.shields.io/badge/Jetpack_Compose-4285F4?style=flat&logo=jetpackcompose&logoColor=white)

</div>
