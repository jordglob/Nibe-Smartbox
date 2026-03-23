# Changelog

All notable changes to the Revolt Smartbox project will be documented in this file.

---

## [1.4.1] - 2026-03-23

### 🐛 Fixed
- **HTTP Buffer Truncation**: Fixed an issue where the spot price array was only capturing the first 1-2 hours of prices. The `http_request` buffer defaults to 1024 bytes, but the 96x 15-minute intervals payload from the API is ~13.6KB. Added `max_response_buffer_size: 16384` to ensure the full payload is processed.
- **Hardware Grounding Warning**: Documented critical grounding requirements (RS485 GND must be bridged to ESP32 GND) to prevent data corruption and Alarm 251 issues.

---

## [1.4.0] - 2026-03-19

### ✨ New
- **Tomorrow's Prices**: Fetches and displays day-ahead prices from elprisetjustnu.se.
- **Independent Graphs**: Separate price graphs for today and tomorrow with 30-minute resolution.
- **Visualisation**: Cheapest upcoming 1-hour period highlighted ("valley" in graph).
- **Architecture**: Asynchronous, non-blocking feeder mechanism for robust GUI performance.

---

## [1.3.2] - 2026-03-18

### 🐛 Fixed
- **15-Min Sparkline Resolution**: Fixed a logic flaw where the spot price parsing only matched on the current hour. Since the elprisetjustnu.se API now provides 15-minute price slots (e.g. `00:00`, `00:15`, `00:30`), the previous hour-only regex caused the sensor to publish the exact same `00:00` price four times per hour, creating a stair-stepped sparkline graph. The parsing regex now correctly includes both hour and minute (`cur_h`, `cur_m`), allowing the graph to perfectly reflect true 15-minute price variations.

---

## [1.3.1] - 2026-03-17

### 🔧 Emergency Threshold Rework + Documentation

Two bug fixes and a comprehensive heat pump overview document.

### ✨ New

- **Dynamic Emergency Threshold** — Replaced fixed öre/kWh threshold with relative "% of daily max" (default 90%). Only the ~top 10% most expensive hours trigger emergency mode. Configurable via web UI slider (50-100%).
- **Heat Pump Overview** — New `docs/HEAT_PUMP_OVERVIEW.md`: complete English-language guide covering all three circuits (brine/heating/DHW), all BT sensors, the refrigerant cycle, DHW charging, degree minutes, and how the Smartbox interacts with the F1245.

### 🐛 Bug Fixes

16. **Fixed emergency threshold too aggressive** — Old fixed 100 öre threshold triggered emergency at 100.07 öre (0.07 öre over!). New dynamic threshold adapts to each day's price range automatically.
17. **Fixed DHW not set during emergency** — Emergency mode used `return` before reaching hot water control logic, leaving DHW mode stuck on whatever it was previously. Now both heating offset AND hot water mode are always set regardless of price zone.

### 📝 Config Changes

- `max_price_threshold` (öre/kWh, fixed) → `emergency_threshold_percent` (%, relative)
- Default: 90% = emergency only when price > 90% of daily max

---

## [1.3.0] - 2026-03-16

### 🚀 Timing-Safe Spot Price Controller — 15 Bug Fixes

V1.3 is a complete rewrite focused on MODBUS timing safety and 15-minute price granularity. Every feature was validated against the nibegw 30ms response deadline.

### ✨ New Features

- **15-minute price slots** — Matches Sweden's quarter-hour settlement (96 periods/day)
- **Current Price Slot** — Text sensor showing active slot (e.g., "22:15-22:30")
- **Control Mode "The Brain"** — Select: HA Manual / LilyGo Local (Spot Price) / Revolt Cloud
- **Max Price Threshold** — User-configurable emergency cutback level (saved to flash)
- **Emergency Economy** — Auto offset -3 when price exceeds user threshold
- **Nerd Tab** — IP, MAC, ESPHome Version, WiFi Signal, Uptime
- **RGB Status LED** — Blue=booting, Green=connected (internal, lightweight)
- **Midnight Auto-Fetch** — Reliable `on_time` trigger at 00:00:30

### 🐛 15 Bug Fixes (vs failed V1.3-draft and V1.2)

1. **Loop blocking >30ms** — Removed all `homeassistant` platform sensors, sparkline, free_heap lambda
2. **RMT LED timing** — Set only on boot, marked `internal: true`
3. **rx_buffer_size: 512** — Reverted to default 256 (proven stable)
4. **std::vector heap alloc** — Removed dynamic allocation entirely
5. **HTTP blocks nibegw** — Added 100ms yield before fetch
6. **No HTTP error handling** — Added proper fallback with status publishing
7. **stof() crash risk** — Replaced with `atof()` (no exceptions on bad data)
8. **GPIO enable timing** — Moved to `on_boot priority: 800` (hardware init)
9. **Mode change blocking** — Changed to `script.execute` for deferred reset
10. **No stabilization guard** — 60s post-boot delay before any non-essential processing
11. **Hot Water restore_value** — Removed; pump state is authoritative on boot
12. **WiFi LED during MODBUS** — LED now boot-only, no WiFi event callbacks
13. **Spot logic runs in Manual mode** — Added mode guard; skips smart control if not Spot Price
14. **HA service call spacing** — Emergency mode returns early to avoid stacking calls
15. **Midnight reset missed** — Changed from interval check to `on_time` trigger

### 📊 Performance

- **RAM**: 12.0% (39,276 / 327,680 bytes)
- **Flash**: 58.5% (1,073,939 / 1,835,008 bytes)
- **HTTP buffer**: 16,384 bytes
- **Compile**: Clean first attempt (1 minor fix: removed deprecated `rmt_channel`)

### ⚠️ Known Limitations

- Fetches today's prices only (tomorrow's day-ahead not yet implemented)
- Cloud mode is a placeholder (mode 3)
- Home Assistant required for Nibe register read/write
- Region hardcoded to SE3 (configurator input ready but not yet wired to URL)

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
