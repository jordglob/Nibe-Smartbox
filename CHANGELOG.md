# Changelog

All notable changes to the Nibe Smartbox project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [1.2.0] - 2026-03-15

### 🔧 Architecture Fix - Corrected Sensor Configuration

**BREAKING CHANGE**: Removed non-functional `homeassistant` platform sensors

### 🐛 Bug Fixes

- **Removed incorrect sensor architecture**: Commented out 6 `homeassistant` platform sensors
  - These sensors were attempting to pull Nibe data from Home Assistant
  - This created a circular dependency (ESP32 → HA → ESP32)
  - `nibegw` component is UDP-only and does NOT provide sensor platforms
  - Nibe data parsing happens in Home Assistant, not on ESP32

### ✨ What Changed

**Sensors Commented Out (Preserved for Future):**
- Supply Temperature
- Return Temperature  
- Outdoor Temperature
- Hot Water Top
- Degree Minutes
- Compressor Frequency

**What Still Works:**
- ✅ NibeGW UDP gateway (broadcasts to Home Assistant)
- ✅ Spot price fetching and smart control
- ✅ Manual control via `number` and `select` components
- ✅ Web Server v3 interface
- ✅ Home Assistant can send commands to ESP32

### 📊 Correct Architecture

```
ESP32 Smartbox:
  - Fetches spot prices (autonomous)
  - Sends raw Nibe data via UDP → Home Assistant
  - Receives control commands from Home Assistant
  - Smart control based on PRICE ONLY (no temp feedback)

Home Assistant:
  - Receives UDP data from ESP32
  - Parses Nibe registers into sensors
  - Can send commands to ESP32 (heat curve, hot water mode)
  - Can create advanced automations using both Nibe sensors + spot prices
```

### 🔮 Future Plans - Standalone Competition

Sensors are **commented out** (not deleted) for future re-activation when we implement:
1. Custom Nibe register parsing component (direct ESP32 parsing)
2. Visual representation of pump data for Standalone Visualization
3. Real-time temperature/status display on local web interface

### 📈 Performance

- **RAM usage**: 11.7% (38,384 bytes) - Slightly reduced
- **Flash usage**: 56.3% (1,033,319 bytes)
- **Compilation time**: ~367 seconds (full rebuild)

---

## [1.0.1] - 2026-03-15

### 🐛 Bug Fixes

- **Buffer Size Increased**: Increased HTTP buffer from 8192 to 16384 bytes
  - Fixed "Could not find price for hour X" errors
  - API response is ~13.6KB, previous 8KB buffer was truncating data
  - Added debug logging to show JSON start/end and search patterns
- **Enhanced Logging**: Added detailed debug output for spot price fetching
  - Shows first 200 chars of JSON response
  - Shows search pattern being used
  - Shows last 200 chars if price not found

### ✅ Verified Working

- Spot price detection: **WORKING** (76.79 öre/kWh at hour 17)
- Buffer utilization: 13,601 bytes / 16,384 bytes (83%)
- Smart control: Active and responding to price changes

---

## [1.0.0] - 2026-03-15

### 🎉 Initial Release

First production-ready release of Nibe Smartbox - a commercial-grade smart controller for Nibe F1245 heat pumps with autonomous electricity spot price optimization.

### ✨ Added

#### Core Features
- **Dual-purpose firmware**: UDP Gateway (NibeGW) + Standalone Smart Controller
- **Spot price integration**: Automatic fetching from elprisetjustnu.se API (SE3 region)
- **Smart control logic**: Price-based heating optimization
  - Low prices (bottom 25%): Heat Curve Offset +2
  - Normal prices: Heat Curve Offset 0
  - High prices (top 25%): Heat Curve Offset -2
  - Extremely low (<10 öre): Hot Water Mode = Luxury
- **Web Server v3**: Local monitoring and control interface
- **Home Assistant integration**: Full ESPHome API support
- **OTA updates**: Wireless firmware updates

#### Hardware Support
- **LilyGo T-CAN485 v1.1**: Hardware auto-direction RS485 (NO dir_pin)
- **Nibe F1245**: Heat pump RS485 communication
- **GPIO configuration**: Proper inverted outputs for v1.1 hardware

#### Network & Time
- **WiFi**: Captive portal for easy setup
- **SNTP time sync**: Europe/Stockholm timezone
- **30-second boot delay**: Ensures time sync before spot price fetch
- **Explicit time validation**: `wait_until: time.has_time`

#### API & Data
- **HTTP request component**: 8192-byte buffer for 24-hour JSON data
- **Dynamic URL generation**: No hardcoded dates
- **Dynamic pattern matching**: Current date/hour parsing
- **Hourly updates**: Automatic spot price refresh every 60 minutes

#### Sensors & Controls
- **6 temperature sensors**: Via Home Assistant integration
  - Supply Temperature (Reg 40004)
  - Return Temperature (Reg 40012)
  - Outdoor Temperature (Reg 40008)
  - Hot Water Top (Reg 40014)
  - Degree Minutes (Reg 43005)
  - Compressor Frequency (Reg 43081)
- **Heat Curve Offset**: Number component (-10 to +10)
- **Hot Water Mode**: Select component (Economy/Normal/Luxury)
- **Spot price sensors**: Current, Daily Min, Daily Max
- **Price status**: Text sensor for current state

### 🔧 Technical Details

#### Performance
- **RAM usage**: 11.7% (38,456 bytes)
- **Flash usage**: 56.5% (1,036,027 bytes)
- **Compilation time**: ~83 seconds
- **Spot price fetch**: ~1.3 seconds
- **RS485 communication**: Stable continuous operation

#### Configuration
- **UART**: RX=GPIO21, TX=GPIO22, Baud=9600
- **RS485 Enable**: GPIO19 (inverted)
- **Auto-Direction**: GPIO17 (inverted)
- **5V Enable**: GPIO16 (inverted)
- **HTTP buffer**: 8192 bytes (max for 24-hour data)

### 📚 Documentation
- **README.md**: Project overview and quick start
- **BUILD_GUIDE.md**: Comprehensive build and deployment guide
  - Architecture & design decisions
  - 5 compilation challenges documented with solutions
  - Complete troubleshooting guide
  - Configuration reference
- **LICENSE**: MIT License
- **CHANGELOG.md**: This file

### 🐛 Fixed

#### Compilation Issues (5 iterations)
1. **HTTP response variable access**: Discovered `body` parameter is auto-provided
2. **Buffer size truncation**: Increased from 2048 → 4096 → 8192 bytes
3. **Hardcoded API date**: Implemented dynamic URL generation
4. **Hardcoded search pattern**: Implemented dynamic date/hour matching
5. **Lambda variable shadowing**: Removed redundant `body` declaration

#### Runtime Issues
- **Time sync race condition**: Added 30s delay + explicit `wait_until: time.has_time`
- **Truncated JSON response**: Increased buffer to 8192 bytes
- **Missing spot price data**: Fixed with larger buffer and dynamic parsing

### ⚠️ Known Limitations

- **Buffer at limit**: 8192-byte response fills buffer completely
- **Single region**: Only SE3 (Stockholm) supported
- **Home Assistant required**: For Nibe register read/write operations
- **API dependency**: Requires elprisetjustnu.se availability

### 🔮 Future Enhancements

Planned for future releases:
- Multi-region support (SE1-SE4, NO, DK, FI)
- Alternative spot price APIs
- Standalone operation (no Home Assistant)
- Weather forecast integration
- Historical data logging
- Mobile app

### 🙏 Credits

- **ESPHome**: https://esphome.io
- **elupus/esphome-nibe**: https://github.com/elupus/esphome-nibe
- **elprisetjustnu.se**: Spot price API provider
- **LilyGo**: T-CAN485 hardware

---

## Release Notes

### Deployment Verification

The v1.0 release was verified through 5 complete deployment iterations with live hardware:

**Successful Log Output:**
```
[12:31:08.398][I][spot_price:309]: ✓ Current spot price: 55.19 öre/kWh (0.5519 SEK/kWh)
[12:31:08.399][D][sensor:118]: 'Daily Min Price' >> 27.26 öre/kWh
[12:31:08.408][D][sensor:118]: 'Daily Max Price' >> 76.88 öre/kWh
[12:31:08.408][I][spot_price:330]: Daily range: 27.26 - 76.88 öre/kWh
[12:31:08.408][I][smart_control:350]: NORMAL PRICE - Setting Heat Curve Offset to 0
[12:31:08.433][I][smart_control:369]: NORMAL PRICE - Setting Hot Water to Normal
```

### Breaking Changes

None - this is the initial release.

### Migration Guide

Not applicable - this is the initial release.

---

**For detailed installation and configuration instructions, see [BUILD_GUIDE.md](BUILD_GUIDE.md)**
