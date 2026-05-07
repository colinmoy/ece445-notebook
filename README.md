# ece445-notebook
## Date: 2026-02-25

### Objective
Research wireless communication protocols (BLE vs. Wi-Fi) for the ESP32 and initialize the ESP-IDF development environment.

### Record
* Installed the Espressif IoT Development Framework (ESP-IDF) and FreeRTOS toolchain on my local machine to support the C++ firmware development.
* Began researching Bluetooth Low Energy (BLE) GATT profiles for transmitting sensor telemetry to the companion app.
* **Debugging Note:** Discovered a major architectural hurdle with our App team's stack. Standard React Native (via Expo Go) does not support native BLE bridging libraries out-of-the-box without ejecting the app and doing custom iOS/Android builds. 
* **Design Decision:** Proposing a pivot to establishing a local Wi-Fi Access Point (AP) and an HTTP server on the ESP32 instead of BLE. This will allow the React Native app to request data using standard JSON `fetch()` calls, entirely bypassing the Expo native-code limitations.

---
