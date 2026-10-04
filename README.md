# HeyChat Flutter Mobile App

HeyChat is a cross-platform mobile messaging application built with Flutter, supporting real-time chat via WebSockets, user authentication, profile management, and cloud backup.

---

## 🚀 Quick Start

### Prerequisites
- [Flutter SDK](https://docs.flutter.dev/get-started/install) (v3.0.0 or higher)
- Android Studio / VS Code with Flutter extension
- Android Emulator or physical device
- Backend API running locally (e.g. Spring Boot running on port `8081` or `8080`)

### 1. Environment Configuration (`.env`)
Create or edit the `.env` file in the project root:

```env
# For Android Emulator with ADB reverse (Recommended)
BACKEND_URL=127.0.0.1:8081

# OR for Android Emulator directly via loopback alias
# BACKEND_URL=10.0.2.2:8081

# OR for Production / Cloud Backend
# BACKEND_URL=heychat-latest.onrender.com
```

### 2. Run the App
Always include `--dart-define-from-file=.env` when launching the app so Flutter loads the environment variables:

```bash
flutter run --dart-define-from-file=.env
```

---

## 🛠️ Local Backend Connection & Fixes

When testing the app with a local backend server on your development machine (e.g., `http://localhost:8081`), you may encounter:
> `ClientException with SocketException: Connection refused`  
> `TimeoutException: Could not reach host`

This section details how to solve connection issues.

---

### 🔍 Why Connection Fails

1. **`localhost` inside Android Emulator points to the emulator itself**:  
   Inside an Android Virtual Device (AVD), `localhost` or `127.0.0.1` resolves to the virtual phone itself—**not your host computer**. Therefore, requests never hit your backend and fail immediately with `Connection refused`.

2. **App Caches the Server Host in `SharedPreferences`**:  
   HeyChat saves the server host in device storage on initial launch (`AppConfig.init()`). If you updated `.env` after running the app once, **the app will keep using the previously saved host** until updated in-app or app data is cleared.

---

### 🔧 Fix 1: ADB Reverse Port Forwarding (Recommended)

`adb reverse` creates a direct bridge between your host computer and the Android emulator or connected device.

1. Ensure your backend is running on port `8081` on your host PC.
2. Run in your terminal:
   ```bash
   adb reverse tcp:8081 tcp:8081
   ```
3. In `.env` or in-app Server Settings, set the host to:
   ```env
   BACKEND_URL=127.0.0.1:8081
   ```
4. Now, any request to `127.0.0.1:8081` inside the mobile app is transparently forwarded to port `8081` on your PC!

---

### 🔧 Fix 2: Android Emulator Host IP (`10.0.2.2`)

Android Emulator provides `10.0.2.2` as a special alias to access `localhost` on the host PC.

1. Set `.env`:
   ```env
   BACKEND_URL=10.0.2.2:8081
   ```
2. If `10.0.2.2` times out, ensure your backend server (e.g. Spring Boot) is bound to `0.0.0.0` rather than `127.0.0.1` strictly:
   ```properties
   # application.properties (Spring Boot)
   server.address=0.0.0.0
   server.port=8081
   ```

---

### 🔧 Fix 3: Update Host via In-App Server Settings

If `SharedPreferences` cached an old URL (e.g., `localhost:8080` or production URL):

1. Open the app to the **Login Screen** (or OTP / Signup screen).
2. Tap the **WiFi Settings icon** at the top right of the app bar.
3. Enter your target backend host (e.g., `10.0.2.2:8081` or `127.0.0.1:8081`).
4. Tap **TEST PING** to verify connectivity to `/actuator/health`.
5. Tap **SAVE** to update the stored server host.

*(Alternatively, uninstall the app from the emulator to reset saved preferences).*

---

### 🔧 Fix 4: Physical Device over Wi-Fi LAN

If testing on a real phone over Wi-Fi:

1. Find your host computer's local LAN IP address:
   - **Linux / macOS**: `ip a` or `ifconfig` (e.g. `192.168.1.15`)
   - **Windows**: `ipconfig`
2. Connect both phone and PC to the **same Wi-Fi network**.
3. In the app's Server Settings, set host to `192.168.1.15:8081`.
4. Ensure your host machine's firewall (`firewalld`, `ufw`, or Windows Firewall) allows incoming traffic on port `8081`:
   ```bash
   # Fedora / RHEL firewalld example
   sudo firewall-cmd --add-port=8081/tcp --permanent
   sudo firewall-cmd --reload
   ```

---

## 🏗️ Architecture & Features

- **Dynamic Server Config** (`lib/config/app_config.dart`): Automatically switches between HTTP/HTTPS and WS/WSS based on host address.
- **Authentication** (`lib/services/auth_service.dart`): Mobile OTP verification, JWT session management, profile setup.
- **Real-time Messaging** (`lib/services/websocket_service.dart`): Stomp/WebSocket integration for instant messaging.
- **Local Storage** (`lib/services/db_service.dart`): SQLite local message persistence.
- **Cloud Backup** (`lib/services/google_drive_service.dart`): Chat backup and restore capability.
