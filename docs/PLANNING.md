# Revolt Nibe Smartbox — Version Planning

## V1.3 (Current — Released 2026-03-16)
**Status:** ✅ Deployed, monitoring for stability

15 bug fixes over V1.2. Spot price optimizer with 15-min resolution, sparkline, cheapest-period detection. See CHANGELOG.md.

**Stability watch items:**
- Alarm 251 (communication error) — should not occur
- "Ignoring byte" in logs — indicates timing regression
- WiFi drops / reconnect behavior
- Midnight price fetch reliability

---

## V1.4 (Skipped unless needed)
Minor incremental improvements only if V1.3 shows issues:
- Wire electricity region selector to API URL
- Tomorrow's day-ahead prices after 13:00
- Better error recovery on HTTP failures

---

## V1.5 — "The Major Rebuild" 🏗️
**Goal:** Parse ~10 Nibe sensor values directly on the ESP32 and display them on the local web page, while maintaining full HA integration via nibegw UDP.

### Key Findings from nibe_issues.json Research

#### 🔴 The 30ms Rule (Non-Negotiable)
Any operation blocking the ESP32 main loop >30ms kills the Nibe token-passing handshake. Causes alarm 251, "Ignoring byte" cascades, corrupted communication. (Issues #108, #123)

#### 🔴 Proven Loop-Blockers
- HTTP requests (spot price fetch)
- Web server serving heavy pages
- UDP send to unreachable IP (issue #123 — 91ms+ blocks)
- Verbose logging (issue #108)
- homeassistant platform sensors
- Display/LED updates during MODBUS

#### 🟡 nibegw Architecture
nibegw is a UDP forwarder only. It does NOT parse registers. It receives raw Nibe telegrams via UART and forwards them via UDP to HA. The Nibe HA integration then decodes registers.

#### 🟡 Direct Parsing IS Possible
From issues #115 (DEH500) and #108 (nptr): People have successfully parsed raw Nibe telegrams directly on ESP32. The `NibeGw.cpp` state machine already decodes frames. The data is there — just forwarded to UDP instead of being parsed locally.

### Architecture

```
┌─────────────────────────────────────────────────┐
│                ESP32 (LilyGo T-CAN485)          │
│                                                   │
│  ┌──────────┐    ┌──────────────┐                │
│  │ nibegw   │───►│ UDP Forward  │──► HA (as before)
│  │ (UART)   │    └──────────────┘                │
│  │          │    ┌──────────────┐                │
│  │          │───►│ Local Parser │──► Template Sensors
│  └──────────┘    │ (tap, no     │    (web page)  │
│                  │  extra UART) │                │
│                  └──────────────┘                │
│                                                   │
│  ┌──────────┐    ┌──────────────┐                │
│  │ Spot     │───►│ Smart Control│──► HA service  │
│  │ Price    │    └──────────────┘    calls       │
│  └──────────┘                                    │
└─────────────────────────────────────────────────┘
```

**Key insight:** We don't add a second UART reader. We hook into nibegw's existing `callback_msg_received_type` callback. The parser runs in callback context — just memcpy from an already-parsed buffer. <1ms, zero loop blocking.

### Implementation: Custom ESPHome Component

Write `components/nibe_parser/` C++ component that:
1. Registers as a listener on nibegw's message callback
2. Parses MODBUS40 data messages (token 0x68) for known register addresses
3. Publishes values to ESPHome template sensors
4. **Zero additional UART load** — piggybacks on nibegw's existing parsing
5. **Zero loop blocking** — just copies bytes from callback buffer

### Target Sensors (~10)

| Register | Name | Unit | Description |
|----------|------|------|-------------|
| 40004 | BT1 Outdoor Temp | °C | Outside temperature |
| 40008 | BT2 Supply Temp | °C | Heat medium flow |
| 40012 | BT3 Return Temp | °C | Return temperature |
| 40013 | BT7 HW Top | °C | Hot water top |
| 40014 | BT6 HW Load | °C | Hot water charging |
| 43005 | Degree Minutes | DM | Compressor need indicator |
| 43009 | Calc Supply Temp | °C | Calculated flow temp |
| 43084 | Elec Addition Power | kW | Current add. heat power |
| 44270 | Current (BE1) | A | Phase 1 current |
| 10012 | Compressor Status | on/off | Compressor running |

### Build Stages

**Stage 1: Custom Parser Component**
- Write `components/nibe_parser/` C++ component
- Hook into nibegw callback
- Parse 10 registers → template sensors
- Compile & verify no timing regression

**Stage 2: Web Visualization**
- Display parsed sensors on web page
- Simple HTML table (no heavy JS frameworks)
- Keep it under 2KB of HTML

**Stage 3: Remove HA Dependency for Control**
- Use parsed Degree Minutes + Supply Temp for smarter spot price logic
- Direct register writes via nibegw UDP (instead of homeassistant.service)
- True standalone operation possible

### Risk Assessment

| Risk | Severity | Mitigation |
|------|----------|------------|
| Custom component blocks loop >30ms | 🔴 Critical | Parser is just memcpy — <1ms |
| Web server request blocks during MODBUS | 🟡 Medium | Web Server v3 is async; keep page tiny |
| nibegw callback API changes upstream | 🟡 Medium | Pin to specific git ref |
| RAM exhaustion with 10 extra sensors | 🟢 Low | Currently at 12% RAM |
| ESPHome 2026.3.0 socket changes | 🟡 Medium | Monitor upstream |

### Hardware Constraints (Unchanged)
- GPIO16, GPIO17, GPIO19: MUST be inverted outputs (chip enable)
- NO dir_pin in software — crashes hardware auto-direction
- GPIO4: RGB LED (WS2812, lightweight status only)
- GPIO21/22: UART to Nibe (9600 baud)
- Framework: Arduino (not esp-idf — nibegw compatibility)

---

## Build Prompt for V1.5 (Copy-Paste for Next Session)

```
You are an Expert Embedded Systems Architect and ESPHome Developer continuing
the Revolt Nibe Smartbox project. We are building V1.5.

CONTEXT:
- Read docs/PLANNING.md for the full V1.5 architectural plan
- Read smartbox.yaml for the current V1.3 firmware
- Read temp/nibe_issues.json for known bugs and timing constraints
- The LilyGo T-CAN485 v1.1 hardware constraints are documented in PLANNING.md

TASK — V1.5 Stage 1: Custom Nibe Parser Component
1. Create components/nibe_parser/ with a C++ ESPHome custom component
2. The component must hook into nibegw's message callback to intercept
   MODBUS40 data telegrams (token 0x68)
3. Parse the ~10 registers listed in PLANNING.md
4. Expose them as ESPHome template sensors with proper names and units
5. The parser MUST NOT block the loop for >30ms (this is critical)
6. Update smartbox.yaml to include the new component and sensors
7. Compile with: esphome erun smartbox.yaml
8. Fix any errors autonomously (up to 5 iterations)
9. Verify RAM usage stays under 50%

CONSTRAINTS:
- GPIO16/17/19 must be inverted outputs. NO dir_pin.
- Framework: Arduino (not esp-idf)
- nibegw UDP broadcast to 255.255.255.255:9999 must continue working
- Web Server v3 on port 80 must continue working
- All V1.3 features (spot price, control modes) must be preserved
```
