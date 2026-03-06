# 🏠 Smart Home IoT Sensor Dashboard

A full-stack IoT system for real-time home monitoring. Sensor data is collected by an Arduino, transmitted wirelessly via an ESP32 module, stored in Firebase Realtime Database, and displayed on a live web dashboard with authentication.

---

## 📋 Table of Contents

- [Overview](#overview)
- [System Architecture](#system-architecture)
- [Hardware Components](#hardware-components)
- [Wiring & Pin Configuration](#wiring--pin-configuration)
- [Software Components](#software-components)
- [Prerequisites](#prerequisites)
- [Setup Instructions](#setup-instructions)
  - [1. Firebase Setup](#1-firebase-setup)
  - [2. Arduino Setup](#2-arduino-setup)
  - [3. ESP32 Setup](#3-esp32-setup)
  - [4. Website Setup](#4-website-setup)
- [Firebase Database Structure](#firebase-database-structure)
- [Dashboard Features](#dashboard-features)
- [Security Notes](#security-notes)
- [License](#license)

---

## Overview

This project implements a smart home sensor hub that:

- **Reads** data from 10 environmental sensors every 5 seconds via an Arduino microcontroller.
- **Transmits** the sensor data wirelessly to Firebase through an ESP32 module.
- **Stores** all readings in Firebase Realtime Database.
- **Displays** the live data on a responsive web dashboard with login-protected access and emergency alerts.

---

## System Architecture

```
┌─────────────────────────────────┐
│  10+ Sensors (Digital/Analog)   │
└─────────────┬───────────────────┘
              │ Digital/Analog reads
              ▼
┌─────────────────────────────────┐
│  Arduino (Sensor Hub)           │
│  - Reads all sensors            │
│  - Sends comma-separated data   │
│    via SoftwareSerial (9600)    │
└─────────────┬───────────────────┘
              │ Serial (TX → RX2 on ESP32)
              ▼
┌─────────────────────────────────┐
│  ESP32 (Wi-Fi Gateway)          │
│  - Parses incoming serial data  │
│  - Authenticates with Firebase  │
│  - Uploads data to RTDB         │
└─────────────┬───────────────────┘
              │ HTTPS (Wi-Fi)
              ▼
┌─────────────────────────────────┐
│  Firebase Realtime Database     │
│  - Stores sensor values         │
│  - Provides real-time sync      │
└─────────────┬───────────────────┘
              │ Firebase SDK (WebSocket)
              ▼
┌─────────────────────────────────┐
│  Web Dashboard (Browser)        │
│  - Login via Firebase Auth      │
│  - Live sensor cards & charts   │
│  - Emergency alerts             │
│  - Dark / Light theme toggle    │
└─────────────────────────────────┘
```

**Update cycle:** approximately every 5 seconds (set by `delay(5000)` in the Arduino sketch).

---

## Hardware Components

| Component | Model / Type | Purpose |
|-----------|-------------|---------|
| Microcontroller | Arduino (ATmega328P) | Sensor hub & serial gateway |
| Wi-Fi SoC | ESP32 | Cloud connectivity |
| Ultrasonic sensor | HC-SR04 | Water level measurement |
| PIR motion sensor | Generic PIR | Motion detection |
| Flame sensor | IR-based | Fire detection |
| Gas sensor | MQ-2 | LPG / smoke leak detection |
| IR sensor | TCRT5000 / Generic | Door open/closed detection |
| Tilt switch | Ball tilt switch | Tilt / vibration events |
| Vibration sensor | SW-420 | Structural vibration |
| Light sensor | LDR (photoresistor) | Ambient light level |
| Rain sensor | Resistive rain module | Rainfall level |
| Push button | Tactile button | Manual trigger |

---

## Wiring & Pin Configuration

### Arduino Pin Map

| Sensor | Arduino Pin | Mode |
|--------|-------------|------|
| Push Button | D2 | INPUT |
| Flame Sensor | D3 | INPUT |
| IR Sensor | D4 | INPUT |
| Vibration Sensor | D5 | INPUT |
| PIR Motion Sensor | D6 | INPUT |
| HC-SR04 TRIG | D7 | OUTPUT |
| HC-SR04 ECHO | D8 | INPUT |
| Tilt Switch | D9 | INPUT |
| SoftwareSerial RX | D0 | — |
| SoftwareSerial TX | D12 | → ESP32 RX2 (GPIO 16) |
| LDR | A1 | Analog INPUT |
| MQ-2 Gas Sensor | A2 | Analog INPUT |
| Rain Sensor | A3 | Analog INPUT |

### ESP32 Pin Map

| Function | ESP32 GPIO |
|----------|-----------|
| Serial2 RX (← Arduino TX) | GPIO 16 |
| Serial2 TX (→ Arduino RX) | GPIO 17 |

> **Note:** Connect the Arduino TX (D12) to the ESP32 RX2 (GPIO 16). A logic-level shifter is recommended since Arduino runs at 5 V and ESP32 at 3.3 V.

---

## Software Components

```
FIREBASE-IOT/
├── arduinoFirebase/
│   └── arduinoFirebase.ino              # Arduino sketch – sensor reading & serial output
├── ESP32 CODE/
│   ├── esp32_firebase_with_authentication/
│   │   └── esp32_firebase_with_authentication.ino   # ESP32 with email/password auth
│   └── esp32_without_authentication_no/
│       └── esp32_without_authentication_no.ino      # ESP32 with anonymous sign-up
└── WEBSITE/
    ├── firebase.json                    # Firebase Hosting & Database config
    ├── database.rules.json              # Firebase Realtime Database security rules
    ├── .firebaserc                      # Firebase project alias
    └── public/
        ├── index.html                   # Dashboard UI (HTML + CSS + inline scripts)
        ├── 404.html                     # Custom 404 error page
        └── scripts/
            ├── index.js                 # UI state management (login/logout toggles)
            └── auth.js                  # Firebase Authentication logic
```

### Key libraries

| Layer | Library / SDK | Version |
|-------|--------------|---------|
| Arduino | SoftwareSerial | Built-in |
| ESP32 | Firebase_ESP_Client | Latest |
| ESP32 | WiFi | Built-in (ESP32 core) |
| Frontend | Firebase JS SDK | 8.8.1 |
| Frontend | Chart.js | 3.9.1 |
| Frontend | Font Awesome | 6.0.0 |

---

## Prerequisites

### Hardware
- Arduino board (Uno / Nano or compatible ATmega328P)
- ESP32 development board
- All sensors listed in [Hardware Components](#hardware-components)
- USB cables for programming
- Jumper wires and breadboard (or PCB)
- Optional: 5 V → 3.3 V logic-level shifter

### Software
- [Arduino IDE](https://www.arduino.cc/en/software) 1.8+ (or Arduino IDE 2.x)
- ESP32 board support installed in Arduino IDE ([installation guide](https://docs.espressif.com/projects/arduino-esp32/en/latest/installing.html))
- [Firebase_ESP_Client](https://github.com/mobizt/Firebase-ESP-Client) library (install via Arduino Library Manager)
- [Firebase CLI](https://firebase.google.com/docs/cli) (for hosting deployment)
- A Google account with a Firebase project

---

## Setup Instructions

### 1. Firebase Setup

1. Go to the [Firebase Console](https://console.firebase.google.com/) and create a new project.
2. **Enable Realtime Database:**
   - Navigate to *Build → Realtime Database → Create database*.
   - Choose a region (e.g., `asia-southeast1`).
   - Start in **test mode** initially, then apply proper security rules before going live.
3. **Enable Authentication:**
   - Navigate to *Build → Authentication → Sign-in method*.
   - Enable **Email/Password** provider.
   - Add a user under the *Users* tab (e.g., `your@email.com` / `yourpassword`).
4. **Get your project credentials:**
   - Go to *Project Settings → General → Your apps → Web app*.
   - Copy the `apiKey` and the Realtime Database URL.
5. **Update security rules** in `WEBSITE/database.rules.json` to match your requirements before deploying.

---

### 2. Arduino Setup

1. Open `arduinoFirebase/arduinoFirebase.ino` in the Arduino IDE.
2. Verify the pin definitions at the top of the file match your physical wiring.
3. Adjust sensor thresholds if needed:
   ```cpp
   #define LPG_THRESHOLD 130        // Lower = more sensitive
   const int RAIN_THRESHOLD = 750;  // Light rain threshold
   const int HEAVY_RAIN_THRESHOLD = 500; // Heavy rain threshold
   ```
4. Select your Arduino board (*Tools → Board*) and port (*Tools → Port*).
5. Upload the sketch.
6. Open the Serial Monitor at **9600 baud** to verify sensor readings are printed every 5 seconds.

---

### 3. ESP32 Setup

Two firmware variants are provided. Choose the one that matches your Firebase configuration:

#### Option A – With Email/Password Authentication (recommended)

1. Open `ESP32 CODE/esp32_firebase_with_authentication/esp32_firebase_with_authentication.ino`.
2. Fill in your credentials:
   ```cpp
   #define WIFI_SSID      "Your_WiFi_SSID"
   #define WIFI_PASSWORD  "Your_WiFi_Password"
   #define API_KEY        "Your_Firebase_API_Key"
   #define DATABASE_URL   "https://your-project-default-rtdb.region.firebasedatabase.app/"
   #define USER_EMAIL     "your@email.com"
   #define USER_PASSWORD  "yourpassword"
   ```
3. Replace the UID check in `setup()` with your own Firebase user UID:
   ```cpp
   if (uid == "YOUR_FIREBASE_USER_UID") {
   ```
4. In the Arduino IDE, select your ESP32 board (*Tools → Board → ESP32 Dev Module*) and port.
5. Upload the sketch.
6. Open the Serial Monitor at **9600 baud** to confirm Wi-Fi connection and successful Firebase uploads.

#### Option B – Without Authentication (anonymous sign-up)

1. Open `ESP32 CODE/esp32_without_authentication_no/esp32_without_authentication_no.ino`.
2. Fill in your credentials:
   ```cpp
   #define WIFI_SSID    "Your_WiFi_SSID"
   #define WIFI_PASSWORD "Your_WiFi_Password"
   #define API_KEY      "Your_Firebase_API_Key"
   #define DATABASE_URL "https://your-project-default-rtdb.region.firebasedatabase.app/"
   ```
3. Enable **Anonymous** sign-in in the Firebase Console (*Authentication → Sign-in method → Anonymous*).
4. Upload the sketch.

> **Note:** This variant was built for an earlier 9-sensor configuration that included a temperature sensor. It may require updating to match the current 10-sensor data format from the Arduino sketch.

---

### 4. Website Setup

#### Development (local preview)

1. Install the Firebase CLI:
   ```bash
   npm install -g firebase-tools
   ```
2. Log in:
   ```bash
   firebase login
   ```
3. From the `WEBSITE/` directory, run:
   ```bash
   firebase serve
   ```
4. Open `http://localhost:5000` in your browser.

#### Production deployment

1. Update `WEBSITE/.firebaserc` with your Firebase project ID:
   ```json
   {
     "projects": {
       "default": "your-firebase-project-id"
     }
   }
   ```
2. Update the Firebase configuration object inside `WEBSITE/public/index.html` with your project's credentials:
   ```js
   const firebaseConfig = {
     apiKey: "YOUR_API_KEY",
     authDomain: "your-project.firebaseapp.com",
     databaseURL: "https://your-project-default-rtdb.region.firebasedatabase.app/",
     projectId: "your-project-id",
     storageBucket: "your-project.appspot.com",
     messagingSenderId: "YOUR_SENDER_ID",
     appId: "YOUR_APP_ID"
   };
   ```
3. Deploy:
   ```bash
   firebase deploy
   ```

---

## Firebase Database Structure

All sensor values are written under the `sensors/` path:

```
sensors/
├── water_level    (Integer, 0–100 %)
├── motion         (String: "Motion_Detected" | "No_Motion")
├── fire           (String: "Fire_Detected" | "No_Fire")
├── lpg            (String: "LPG_Leak_Detected" | "LPG_No_Leakage")
├── ir_sensor      (String: "Door_is_Closed" | "Door_is_Opened")
├── tilt           (String: "Tilt_Detected" | "No_Tilt")
├── vibration      (String: "Vibration_Detected" | "No_Vibration")
├── light          (Integer, 0–100 %)
├── rain           (String: "No_Rain" | "Light_Rain" | "Heavy_Rain")
├── button         (String: "Button_ON" | "Button_OFF")
└── timestamp      (Integer, seconds since ESP32 boot)
```

---

## Dashboard Features

| Feature | Description |
|---------|-------------|
| **Real-time sensor cards** | Each sensor has a dedicated card that updates live via Firebase WebSocket sync |
| **Animated progress bars** | Water level and light intensity displayed as animated percentage bars |
| **Fire alert popup** | A modal alert appears automatically when `fire == "Fire_Detected"` |
| **Rain animation** | An animated rain overlay is shown when `rain == "Heavy_Rain"` |
| **Dark / Light theme** | Toggle button switches between dark and light color schemes |
| **Firebase Authentication** | Email/password login gate — dashboard content is hidden until the user signs in |
| **Logout** | Signed-in users can log out from the navigation bar |
| **Last-updated timestamp** | Header shows the time of the most recent data refresh |
| **Responsive layout** | CSS Grid layout adapts from single-column (mobile) to multi-column (desktop) |
| **404 page** | Custom error page for unmatched routes on Firebase Hosting |

---

## Security Notes

- **Credentials in source code:** The ESP32 sketches currently contain Wi-Fi passwords, Firebase API keys, and user credentials as plain `#define` constants. Before sharing or open-sourcing this code, move sensitive values to a separate `secrets.h` file that is excluded from version control (add it to `.gitignore`).
- **Database rules:** The `database.rules.json` file uses timestamp-based open rules intended for development. Replace them with proper UID-scoped rules for any production deployment:
  ```json
  {
    "rules": {
      "sensors": {
        ".read": "auth != null",
        ".write": "auth != null && auth.uid == 'YOUR_DEVICE_UID'"
      }
    }
  }
  ```
- **Anonymous sign-up (Option B):** Allows any client with the API key to write to the database. Use the authenticated variant for any internet-facing deployment.

---
