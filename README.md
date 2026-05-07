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
## Date: 2026-03-04

### Objective
Set up the initial breadboard hardware to mock sensor inputs and feedback mechanisms for the upcoming Breadboard Demo.

### Record
* Because the I2C sensors (VL53L0X and ICM-42670-P) and feedback components (ERM motor, buzzer) have not yet arrived, I designed a mocked hardware environment to test our firmware logic.
* Wired tactile push buttons to the ESP32 GPIO pins to simulate the ToF and IMU threshold triggers (simulating a "bad posture" event).
* Wired standard LEDs with current-limiting resistors to serve as visual stand-ins for the progressive feedback system.
* The custom power subsystem is not yet built, so the entire breadboard assembly is being powered directly via a USB-C cable connected to the ESP32 development board.

---

## Date: 2026-03-10

### Objective
Program the non-blocking state machine logic for the 3-second delay and successfully present the Breadboard Demo.

### Record
* Wrote the core C++ firmware using `esp_timer_get_time()` to implement a non-blocking 3.0-second delay without using `vTaskDelay()`.
* Configured the logic so that pressing and holding a mock "sensor" button logs an anchor timestamp. If the button is released before 3 seconds (posture corrected), the timer instantly resets. If held for >= 3 seconds, the ESP32 successfully drives the "feedback" LED pin HIGH.
* Successfully presented the Breadboard Demo to the TAs. Proved that the core timing architecture functions reliably and can track sustained events without freezing the microcontroller's main loop.

---
