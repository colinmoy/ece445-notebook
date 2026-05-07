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
## Date: 2026-03-18

### Objective
Configure ESP-IDF I2C drivers and LEDC PWM timers in preparation for physical hardware integration.

### Record
* Initialized the I2C master driver on GPIO 8 (SCL) and GPIO 9 (SDA) operating at 100 kHz to prepare for the ToF sensor.
* Configured the ESP32 `ledc` peripheral for the progressive feedback system. Assigned Timer 0 to the vibration motor and LED at 1000 Hz, and Timer 1 to the piezoelectric buzzer at 2000 Hz, both utilizing 13-bit resolution.
* Collaborated with Zhiyuan on the software architecture to connect my sensor logic to his Wi-Fi HTTP server. We established a shared `newSessionReady` boolean flag and a `lastDistanceTime` variable so his HTTP handlers can asynchronously read my state machine's events without blocking the main loop.

---

## Date: 2026-04-02

### Objective
Integrate the physical VL53L0X Time-of-Flight sensor on the breadboard and develop the custom I2C read sequences.

### Record
* The physical VL53L0X breakout board arrived. Wired it to the ESP32 and utilized the XSHUT pin (GPIO 2) to manually reset the sensor's boot state.
* Wrote custom `read_reg16()` and `write_reg()` I2C wrapper functions. 
* Developed `vl53l0x_simple_init()` to verify the device ID (0xEE) and extract the required NVM stop variable from register 0x91. 
* **Debugging Note:** The sensor occasionally returned erratic 0mm readings due to complete IR absorption. Implemented software clamping: if `distance_mm > 2000` or `== 0`, it is forcefully set to a safe default of `1200` to prevent false posture alarms.

---

## Date: 2026-04-07

### Objective
Finalize the dynamic PWM state machine and present the Progress Demo.

### Record
* Completed the non-blocking state machine using `esp_timer_get_time()`. If `distance_mm < 300`, the system successfully logs `distanceStartTime` and scales the 13-bit PWM duty cycle linearly up to a 50% cap.
* Verified that if the 3-second window expires, the system triggers the HTTP JSON payload via the `newSessionReady` flag.
* **Demo Results & Debugging Note:** The Progress Demo was unsuccessful. During the live presentation, the VL53L0X sensor locked up and continuously output a single, frozen distance value, completely halting the state machine's ability to track movement. 
* **Hypothesis & Next Steps:** I suspect the breadboard wiring caused a transient voltage drop that crashed the sensor's internal state, or the I2C bus locked up. Moving forward, I need to implement an automatic hardware reset in the main loop. I plan to use the sensor's XSHUT pin to physically power-cycle the VL53L0X if it becomes unresponsive.

---
