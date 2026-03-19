# Revolt Nibe Smartbox - Build & Deployment Guide

## 📋 Table of Contents
1. [Overview](#overview)
2. [Prerequisites](#prerequisites)
3. [Quick Start](#quick-start)
4. [Architecture & Design Decisions](#architecture--design-decisions)
5. [Compilation Challenges & Solutions](#compilation-challenges--solutions)
6. [Configuration Reference](#configuration-reference)
7. [Deployment & Testing](#deployment--testing)
8. [Troubleshooting](#troubleshooting)
9. [Future Improvements](#future-improvements)

---

## Overview

This is a commercial-grade "Smartbox" firmware for the **LilyGo T-CAN485 (v1.1)** ESP32 board that connects to a **Nibe F1245 Heat Pump** via RS485. It serves two purposes simultaneously:

1. **UDP Gateway (NibeGW)**: Broadcasts heat pump data for Home Assistant integration
2. **Standalone Smart Controller**: Autonomously fetches electricity spot prices and optimizes heat pump operation

### Key Features
- ✅ Hardware auto-direction RS485 (critical for LilyGo T-CAN485 v1.1)
- ✅ Dynamic spot price fetching from elprisetjustnu.se API
- ✅ Autonomous heating optimization based on electricity prices
- ✅ Web Server v3 for local monitoring
- ✅ Home Assistant integration via ESPHome API
- ✅ OTA updates support

---

## Prerequisites

### Hardware Required
- **LilyGo T-CAN485 v1.1** ESP32 board
- **Nibe F1245** Heat Pump (or compatible model)
- RS485 cable connection to heat pump
- USB cable for initial programming
- WiFi network access

### Software Required
- **ESPHome** 2026.2.2 or later
- **Python** 3.9+ (for ESPHome)
- **PlatformIO** (automatically installed by ESPHome)
- **Git** (for cloning external components)

### Installation
```bash
# Install ESPHome
pip install esphome

# Verify installation
esphome version
```

---

## Quick Start

### Step 1: Clone or Download
```bash
git clone <your-repo-url>
cd "Nibe Smartbox"
```

### Step 2: Customize Configuration
Edit `smartbox.yaml` and update:
```yaml
wifi:
  ssid: "YOUR_WIFI_SSID"        # Change this
  password: "YOUR_WIFI_PASSWORD" # Change this

ota:
  password: "YOUR_OTA_PASSWORD"  # Change this
```

### Step 3: Compile
```bash
esphome compile smartbox.yaml
```

Expected output:
```
INFO Successfully compiled program.
RAM:   [=         ]  11.7% (used 38440 bytes from 327680 bytes)
Flash: [======    ]  56.4% (used 1035767 bytes from 1835008 bytes)
```

### Step 4: Upload to Device
```bash
# First time upload via USB
esphome run smartbox.yaml

# Select option [1] COM3 (USB-Enhanced-SERIAL CH9102)
# Or your specific COM port
```

### Step 5: Monitor Logs
After upload, the logs will automatically display. Look for:
```
[spot_price:XXX]: ✓ Current spot price: XX.XX öre/kWh (X.XXXX SEK/kWh)
[smart_control:XXX]: LOW PRICE - Setting Heat Curve Offset to +2
```

---

## Architecture & Design Decisions

### Critical Hardware Configuration

**⚠️ IMPORTANT**: The LilyGo T-CAN485 v1.1 uses **hardware auto-direction** for RS485. This is critical:

```yaml
# CORRECT - Hardware auto-direction
output:
  - platform: gpio
    id: ENABLE_PIN
    pin: { number: GPIO19, inverted: true }
  - platform: gpio
    id: SE_PIN
    pin: { number: GPIO17, inverted: true }
  - platform: gpio
    id: ENABLE_5V_PIN
    pin: { number: GPIO16, inverted: true }

uart:
  id: modbus_uart
  rx_pin: GPIO21
  tx_pin: GPIO22
  baud_rate: 9600
  # NO dir_pin - hardware handles direction automatically!
```

**❌ WRONG**: Adding a `dir_pin` will crash the token-passing timing!

### NibeGW Component Architecture (IMPORTANT!)

**Critical Understanding**: The `nibegw` component from `elupus/esphome-nibe` is a **UDP gateway ONLY**. It does NOT provide ESPHome sensor platforms for reading Nibe registers.

**How It Actually Works**:
```
ESP32 (nibegw) → UDP broadcast → Home Assistant (Nibe integration) → Sensors
```

**What nibegw Does**:
- ✅ Receives raw Nibe telegrams via RS485
- ✅ Forwards them via UDP to Home Assistant
- ✅ Can send read/write requests via UDP
- ❌ Does NOT parse registers into ESPHome sensors
- ❌ Does NOT provide `platform: nibe` for sensors

**Correct Architecture (v1.2)**:
- ESP32 acts as UDP gateway + spot price controller
- Home Assistant parses Nibe data into sensors
- ESP32 smart control uses ONLY spot prices (no temp feedback)
- Home Assistant can send commands to ESP32 (via `homeassistant.service`)
- For advanced control, use Home Assistant automations

**Why Sensors Are Commented Out (v1.2)**:
The 6 Nibe sensors using `platform: homeassistant` were creating a circular dependency:
1. ESP32 sends raw data → Home Assistant
2. Home Assistant creates sensors
3. ESP32 tries to pull sensors back ← This doesn't work reliably

**Future Plans (Standalone Direct Parsing)**:
Sensors are preserved (commented out) for when we implement:
1. Custom Nibe register parsing component (direct ESP32 parsing)
2. Visual pump representation (temperatures, status, flow)
3. Real-time temperature display on local web interface

### Time Synchronization Strategy

**Challenge**: The spot price fetch was failing with "Time not synced" even though time appeared synchronized.

**Solution**: Added explicit wait conditions in boot sequence:
```yaml
esphome:
  on_boot:
    priority: -100
    then:
      - wait_until:
          wifi.connected:
      - delay: 30s  # Increased from 10s
      - wait_until:
          time.has_time:  # Explicit time check
      - script.execute: fetch_spot_price
```

This ensures:
1. WiFi is connected
2. 30-second settling time
3. SNTP time sync is complete
4. Only then fetch spot prices

---

## Compilation Challenges & Solutions

### Challenge 1: HTTP Response Variable Access
**Error**: `status_code` and `body` were not declared in lambda scope

**Iterations**:
1. ❌ Tried `status_code` directly → Not in scope
2. ❌ Tried `response->status_code` → Correct!
3. ❌ Tried `response->get_string()` → Method doesn't exist
4. ❌ Tried `response->body_buf` → Member doesn't exist
5. ✅ Used `body` parameter (already provided by `on_response`)

**Final Solution**:
```yaml
on_response:
  then:
    - lambda: |-
        // response->status_code is available
        // body parameter is automatically provided
        ESP_LOGD("spot_price", "HTTP Response Code: %d", response->status_code);
        ESP_LOGI("spot_price", "Response body length: %d bytes", body.length());
```

### Challenge 2: HTTP Buffer Size Truncation
**Problem**: Response body was exactly 2048 bytes (the default buffer size), indicating truncation.

**Iterations**:
1. ❌ Default 1000 bytes → Truncated
2. ❌ Increased to 2048 bytes → Still truncated (exactly 2048)
3. ❌ Increased to 4096 bytes → Still truncated (exactly 4096)
4. ✅ Increased to 8192 bytes → **SUCCESS!** (but still at limit)

**Final Solution**: Set buffer to 8192 bytes:
```yaml
http_request.get:
  max_response_buffer_size: 8192  # Large enough for 24-hour JSON
```

**Note**: The elprisetjustnu.se API returns approximately 8KB of JSON for 24 hours of price data. The buffer is at the limit but working. For production, consider:
- Using a more compact API
- Fetching only current hour data
- Implementing chunked reading

**Verification**: Added warning log:
```cpp
if (body.length() >= 4000) {
  ESP_LOGW("spot_price", "WARNING: Response may be truncated!");
}
```

### Challenge 3: Hardcoded API Date
**Problem**: URL was hardcoded to `2026-03-15`, would fail on other dates.

**Solution**: Dynamic URL generation using time component:
```yaml
url: !lambda |-
  auto time = id(sntp_time).now();
  char url[100];
  snprintf(url, sizeof(url), "https://www.elprisetjustnu.se/api/v1/prices/%04d/%02d-%02d_SE3.json",
           time.year, time.month, time.day_of_month);
  return std::string(url);
```

### Challenge 4: Date Pattern Matching
**Problem**: Search pattern was hardcoded to `2026-03-15`.

**Solution**: Dynamic pattern generation:
```cpp
char search_pattern[50];
snprintf(search_pattern, sizeof(search_pattern), "\"time_start\":\"%04d-%02d-%02dT%02d:",
         time.year, time.month, time.day_of_month, current_hour);
```

### Challenge 5: Lambda Variable Shadowing
**Error**: `declaration of 'std::string body' shadows a parameter`

**Solution**: Removed redundant variable declaration - `body` is already provided by `on_response` callback.

---

## Configuration Reference

### GPIO Pin Assignments (LilyGo T-CAN485 v1.1)
| Pin | Function | Configuration |
|-----|----------|---------------|
| GPIO19 | RS485 Enable | Output, Inverted |
| GPIO17 | Auto-Direction | Output, Inverted |
| GPIO16 | 5V Enable | Output, Inverted |
| GPIO21 | UART RX | Input |
| GPIO22 | UART TX | Output |

### Network Settings
```yaml
wifi:
  ssid: "YOUR_SSID"
  password: "YOUR_PASSWORD"
  power_save_mode: none  # Important for reliability
```

### Spot Price API Configuration
- **Provider**: elprisetjustnu.se
- **Region**: SE3 (Stockholm)
- **Format**: JSON with 24-hour prices
- **Update Frequency**: Every 60 minutes
- **Buffer Size**: 16384 bytes (16KB)
- **Actual Response Size**: ~13.6KB (13,601 bytes)

### Smart Control Thresholds
```yaml
# Price-based heating control
Low Price (bottom 25%):  Heat Curve Offset = +2
Normal Price:            Heat Curve Offset = 0
High Price (top 25%):    Heat Curve Offset = -2

# Hot water mode control
Extremely Low (< 10 öre): Luxury mode
High Price (top 25%):     Economy mode
Normal Price:             Normal mode
```

---

## Deployment & Testing

### Initial Upload (USB)
```bash
esphome run smartbox.yaml
```

Select your COM port when prompted:
```
Found multiple options, please choose one:
  [1] COM3 (USB-Enhanced-SERIAL CH9102)
  [2] Over The Air (nibe-smartbox.local)
(number): 1
```

### Monitoring Logs
After upload, logs will stream automatically. Key indicators of success:

#### 1. WiFi Connection
```
[12:05:18.678][D][wifi:XXX]: Synchronized time: 2026-03-15 12:05:18
```

#### 2. Time Sync
```
[12:05:18.678][D][sntp:XXX]: Synchronized time: 2026-03-15 12:05:18
```

#### 3. Spot Price Fetch
```
[12:05:02.522][I][main:421]: Fetching current spot price...
[12:05:03.621][D][spot_price:252]: HTTP Response Code: 200
[12:05:03.624][I][spot_price:261]: Response body length: 2048 bytes
[12:05:03.624][I][spot_price:266]: Current hour: 12
[12:05:03.624][I][spot_price:XXX]: ✓ Current spot price: 85.00 öre/kWh (0.8500 SEK/kWh)
```

#### 4. Smart Control Activation
```
[12:05:03.624][I][spot_price:XXX]: Daily range: 45.00 - 120.00 öre/kWh
[12:05:03.624][I][smart_control:XXX]: NORMAL PRICE - Setting Heat Curve Offset to 0
```

### OTA Updates (After Initial Upload)
```bash
# Subsequent updates can be done wirelessly
esphome run smartbox.yaml

# Select option [2] Over The Air (nibe-smartbox.local)
```

### Web Interface
Access the web interface at:
```
http://nibe-smartbox.local
# or
http://192.168.X.X  (device IP address)
```

---

## Troubleshooting

### Problem: "Time not synced yet"
**Symptoms**: Logs show "Time not synced yet" even after boot

**Solutions**:
1. Check WiFi connection is stable
2. Verify SNTP servers are reachable
3. Increase boot delay (currently 30s)
4. Check timezone setting: `Europe/Stockholm`

**Verification**:
```
[D][sntp:XXX]: Synchronized time: 2026-03-15 12:05:18
```

### Problem: "Response may be truncated"
**Symptoms**: Warning about truncated response, price parsing fails

**Solutions**:
1. Increase `max_response_buffer_size` beyond 4096
2. Check API response size (should be ~3KB for 24 hours)
3. Verify internet connection speed

**Verification**:
```
[I][spot_price:XXX]: Response body length: 3456 bytes  # Should be < 4000
```

### Problem: "Could not find price for hour X"
**Symptoms**: Price parsing fails for current hour

**Possible Causes**:
1. API date format changed
2. Wrong timezone (should be Europe/Stockholm)
3. API endpoint changed
4. Network issues

**Debug Steps**:
1. Check the generated URL in logs
2. Manually visit the URL in browser
3. Verify JSON format matches expected pattern
4. Check current hour matches API data

### Problem: NibeGW Not Receiving Data
**Symptoms**: No RS485 communication with heat pump

**Solutions**:
1. Verify RS485 wiring (A, B, GND)
2. Check baud rate is 9600
3. Ensure NO `dir_pin` is configured
4. Verify GPIO pins are correct for v1.1
5. Check heat pump RS485 settings

**Verification**:
```
[D][nibegw:XXX]: Response to address: 0x20 token: 0x69 bytes: 6
```

### Problem: Compilation Errors
**Common Issues**:

1. **Missing external component**:
   ```
   ERROR: Component nibe not found
   ```
   Solution: Check internet connection, ESPHome will auto-download

2. **Lambda syntax errors**:
   ```
   error: 'body' was not declared in this scope
   ```
   Solution: Use the exact lambda code from this guide

3. **YAML syntax errors**:
   ```
   YAML Error: Map keys must be unique
   ```
   Solution: Check for duplicate `esphome:` or other top-level keys

---

## V1.3 Architecture Changes (2026-03-17)

### Timing-Safe Dual-Priority Boot

V1.3 introduced a critical architectural change: **dual-priority boot sequence** to prevent nibegw MODBUS timing violations (alarm 251).

```yaml
on_boot:
  # STAGE 1 (priority 800): Hardware init FIRST
  - priority: 800
    then:
      - output.turn_on: ENABLE_PIN
      - output.turn_on: SE_PIN
      - output.turn_on: ENABLE_5V_PIN

  # STAGE 2 (priority -100): Non-critical features AFTER nibegw stabilizes
  - priority: -100
    then:
      - light.turn_on: { id: status_led, blue: 100% }
      - wait_until: wifi.connected
      - light.turn_on: { id: status_led, green: 100% }
      - delay: 60s  # Let nibegw establish token passing
      - wait_until: time.has_time
      - script.execute: fetch_spot_price
```

**Why 60s?** The Nibe heat pump needs ~30-45s to establish stable token passing. Any HTTP fetch or LED update during this window can cause the ESP32 main loop to exceed 30ms, triggering alarm 251.

### 15-Minute Price Granularity

Sweden switched to 15-minute settlement periods (96 slots/day). V1.3 matches this:
- Interval changed from 60min → 15min
- Price lookup matches exact quarter-hour slot
- "Current Price Slot" sensor shows e.g. `"22:15-22:30"`

### Control Mode "The Brain"

Three operating modes via web UI select:
1. **Home Assistant (Manual)** — ESP32 is passive, HA controls everything
2. **LilyGo Local (Spot Price)** — ESP32 autonomously adjusts heat curve/hot water
3. **Revolt Energy Cloud** — Placeholder for future cloud API

Smart control logic only executes when mode 2 is active.

### V1.3 Compilation Notes

| Issue | Fix |
|-------|-----|
| `rmt_channel` deprecated | Removed from `esp32_rmt_led_strip` config |
| `Select::state` deprecated | Changed to `.current_option()` |
| `std::stof()` crash risk | Replaced with `atof()` (no exceptions) |
| `ESP_ERR_HTTP_CONNECT` | Added `follow_redirects: true`, timeout 15s |

### V1.3 Performance

```
RAM:   [=         ]  12.0% (used 39276 bytes from 327680 bytes)
Flash: [======    ]  58.5% (used 1073939 bytes from 1835008 bytes)
```

---

## Future Improvements

### 1. Standalone Direct Parsing
**Current Limitation**: Nibe register read/write requires Home Assistant

**Proposed Solution**:
- Implement direct Modbus register parsing in ESPHome
- Create custom component for Nibe protocol
- Would enable true standalone operation

### 2. Alternative Spot Price APIs
**Current**: elprisetjustnu.se (SE3 region only)

**Alternatives to Consider**:
- spotprice.nu (multi-region)
- nordpoolgroup.com (official source)
- Custom API with caching

### 3. Advanced Price Strategies
**Current**: Simple 25% threshold logic

**Enhancements**:
- Predictive heating based on weather forecast
- Machine learning for optimal timing
- Integration with solar production data
- Multi-day price optimization

### 4. Enhanced Error Handling
- Retry logic for failed HTTP requests
- Fallback to previous price data
- Email/notification on persistent failures
- Automatic recovery mechanisms

### 5. Configuration UI
- Web-based configuration editor
- Real-time price chart visualization
- Manual override controls
- Historical data logging

### 6. Multi-Region Support
- Auto-detect region from IP
- Support for all Nordic regions (SE1-SE4, NO, DK, FI)
- Currency conversion
- Regional price comparison

---

## Appendix: Complete File Structure

```
Nibe Smartbox/
├── README.md              # Project landing page
├── CHANGELOG.md           # Version history
├── LICENSE                # MIT License
├── smartbox.yaml          # Main ESPHome configuration
├── .gitignore             # Git ignore rules
├── docs/
│   ├── BUILD_GUIDE.md     # This file
│   ├── PLANNING.md        # Feature brainstorming & roadmap
│   ├── GITHUB_UPLOAD_GUIDE.md
│   └── VSCODE_GIT_GUIDE.md
├── firmware/
│   ├── firmware.bin        # Latest build
│   ├── firmware.factory.bin
│   ├── firmware.ota.bin
│   ├── v1.2/              # V1.2 archive
│   └── v1.3/              # V1.3 archive
├── backups/
│   └── smartbox_v12.yaml  # V1.2 YAML backup
└── components/            # (Auto-created by ESPHome)
```

---

## Support & Contributing

### Getting Help
1. Check this BUILD_GUIDE.md first
2. Review ESPHome logs for error messages
3. Verify hardware connections
4. Test with minimal configuration

### Reporting Issues
When reporting issues, include:
- ESPHome version (`esphome version`)
- Complete error logs
- Hardware model (LilyGo T-CAN485 v1.1)
- Network configuration (WiFi, internet access)
- Steps to reproduce

### Contributing
Contributions welcome! Areas of interest:
- Alternative spot price APIs
- Enhanced smart control algorithms
- Better error handling
- Documentation improvements
- Multi-language support

---

## License & Credits

### Credits
- **ESPHome**: https://esphome.io
- **elupus/esphome-nibe**: https://github.com/elupus/esphome-nibe
- **elprisetjustnu.se**: Spot price API provider

### Disclaimer
This firmware is provided "as is" without warranty. Use at your own risk. Always monitor your heat pump operation and have backup heating available.

---

**Last Updated**: 2026-03-17  
**Version**: 1.3  
**Tested With**: ESPHome 2026.2.2, LilyGo T-CAN485 v1.1, Nibe F1245
