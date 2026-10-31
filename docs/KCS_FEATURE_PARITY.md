# KCS-style KC868-A6 functionality — implementation map

This file distinguishes features that really work in ModESP v4 from requested KCS parity features.
Never claim KCS v2.2.21 equivalence before each backend feature is implemented and hardware-tested.

## Implemented / reused
- ESP32 Wi-Fi STA/AP configuration: existing ModESP network service and WebUI.
- MQTT configuration: existing ModESP MQTT service and WebUI.
- HTTP authentication and OTA: existing ModESP backend.
- WebSocket live state and REST /api/state: existing ModESP backend.
- KC868-A6 board definition: SDA GPIO4, SCL GPIO15, I2C 100 kHz, PCF8574 inputs 0x22, outputs 0x24.
- New KC868 I/O page: six inputs and relays, bindings, user-defined per-browser channel labels.
- New network/system diagnostic snapshot: filters keys actually returned by /api/state; doesn't fake any metrics.
- Existing lighting schedule and door-trigger automations, only when related roles are bound.

## Not yet implemented (requires tested backend changes)
1. Six unrestricted general-purpose relay channels with independently assigned modes and safety interlocks. Existing equipment roles are NOT general-purpose switches. A raw write would bypass refrigeration safety logic.
2. General KCS input-to-output rules: edges, pulse, toggle, delay, latch, schedules, interlocks; persisted rule engine and event audit.
3. All six digital inputs exported as raw states even if not assigned to equipment roles; PCF8574 0x22 input polling.
4. Relay manual command REST + MQTT with authentication, safe-state and validation; PCF8574 0x24 output ownership resolution.
5. SSD1306 128x64 I2C @ 0x3C driver and shared I2C bus locking, configuration in NVS, WebUI display/contrast/layout controls. Do NOT expose a working OLED control until device driver exists.
6. Multi-channel timers, scenes, per-channel power-up mode, web-based channel naming persisted on ESP32 (current labels live only in the browser).
7. Fully generic GPIO/ADC/DAC/RS232/RS485/OneWire configuration consistent with KCS.
8. KCS-specific MQTT topic schema / Home Assistant entities / API compatibility as verified against an original device.
9. Backup/restore, log viewer, user/role settings beyond existing ModESP settings where original KCS behavior has been verified.

## Hardware acceptance test
- Disconnect all potentially hazardous loads; ensure outputs remain OFF across reset, OTA, invalid I2C transaction.
- Verify six PCF8574 outputs active-low, inputs polarity, and shared-bus access with OLED.
- Verify 3.3V I2C and no 24V on ESP32 GPIO.
- Verify Ethernet/Wi-Fi behavior and MQTT reconnect without toggling relays.
- Validate ESP-IDF and Node build, bootlogs, WebGUI controls and persistence.

## Build WebUI
```bash
cd webui
npm ci
npm run build
npm run deploy
```
Backend image requires the repository ESP-IDF build and LittleFS deployment. This branch has not been compiled on physical KC868-A6.
