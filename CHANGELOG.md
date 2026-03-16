# Changelog

All notable changes to the Revolt Smartbox project will be documented in this file.

---

## [1.3.0] - 2026-03-16

### 🚀 Spot Price Optimizer with 15-Minute Resolution

V1.3 transforms the Smartbox into a fully autonomous spot-price-aware heat pump controller.

### ✨ New Features

- **15-minute price matching** — Sweden's new quarter-hour settlement periods (96 periods/day)
- **Price Sparkline** — ASCII text sensor showing future price trend from current period onward
- **Cheapest Period** — Finds the cheapest 1-hour window in remaining day
- **Price Horizon** — Shows loaded data count (e.g., "96 periods (24 hours)")
- **Control Mode Selector** — 3 modes: HA Manual, Local Spot Price, Revolt Cloud
- **The Configurator** — Region (SE1-SE4), Max Price Threshold, Cloud API Key (all saved to flash)
- **Smart Control Logic** — Adjusts Heat Curve and Hot Water Mode based on price quartiles
- **Nibe Sensors** — 6 sensors imported from Home Assistant (BT1, BT2, BT3, BT7, DM, Compressor)
- **Nerd Tab** — IP, MAC, ESPHome Version, WiFi Signal, Uptime, Free Heap
- **RGB Status LED** — Blue=booting, Green=connected, Red=disconnected

### 🐛 Bug Fixes

- **Fixed**: UDP target reverted to broadcast `255.255.255.255` (V1.2 accidentally set to unicast)
- **Fixed**: Price lookup matches exact 15-min slot, not just hour
- **Fixed**: Counter correctly reports "96 periods" instead of "96 hours"
- **Fixed**: Manual "Update Spot Price" button works in any mode
- **Fixed**: 5-minute boot delay prevents nibegw UART conflicts during startup

### 📊 Performance

- **RAM**: 12.0% (39,452 / 327,680 bytes)
- **Flash**: 59.2% (1,085,975 / 1,835,008 bytes)
- **HTTP buffer**: 16,384 bytes (API response ~13.6KB)
- **Free heap at runtime**: ~215KB

### ⚠️ Known Limitations

- Fetches today's prices only (tomorrow's day-ahead not yet implemented)
- Cloud mode is a placeholder
- Home Assistant required for Nibe register read/write

---

## [1.2.0] - 2026-03-15

### 🔧 Architecture Fix

- Removed non-functional direct Nibe sensor parsing (nibegw is UDP-only)
- Established correct architecture: ESP32 forwards raw MODBUS via UDP → HA parses registers
- RAM: 11.7%, Flash: 56.3%

---

## [1.0.0] - 2026-03-15

### 🎉 Initial Release

- NibeGW UDP gateway for Nibe F1245 on LilyGo T-CAN485 v1.1
- Spot price fetching from elprisetjustnu.se
- Web Server v3, Home Assistant API, OTA updates
- Hardware: GPIO16/17/19 inverted outputs, hardware auto-direction RS485

---

### 🙏 Credits

- [ESPHome](https://esphome.io) | [elupus/esphome-nibe](https://github.com/elupus/esphome-nibe) | [elprisetjustnu.se](https://www.elprisetjustnu.se) | [LilyGo T-CAN485](https://www.lilygo.cc)
