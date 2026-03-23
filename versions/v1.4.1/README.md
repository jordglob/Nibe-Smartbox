# Revolt Nibe Smartbox v1.4.1

**Commercial-grade smart controller firmware for Nibe heat pumps with autonomous electricity spot price optimization**

> 💡 *Development planning notes moved to [docs/PLANNING.md](../../docs/PLANNING.md)*

[![ESPHome](https://img.shields.io/badge/ESPHome-2026.2.2-blue.svg)](https://esphome.io)
[![License](https://img.shields.io/badge/License-MIT-green.svg)](../../LICENSE)
[![Hardware](https://img.shields.io/badge/Hardware-LilyGo%20T--CAN485%20v1.1-orange.svg)](https://www.lilygo.cc/)

---

## 🎯 Overview

Nibe Smartbox transforms your Nibe F1245 heat pump into an intelligent, cost-optimizing system that automatically adjusts heating based on real-time electricity spot prices. Built on ESPHome for the LilyGo T-CAN485 v1.1 ESP32 board, it serves dual purposes:

1. **UDP Gateway (NibeGW)** - Full Home Assistant integration via RS485
2. **Standalone Smart Controller** - Autonomous price-based optimization

### 🍎 Key Features

- 🔌 **Plug & Play RS485** - Hardware auto-direction (critical for T-CAN485 v1.1)
- 💰 **15-Min Spot Price Optimization** - Quarter-hour price matching (96 slots/day)
- 🧠 **Control Mode "The Brain"** - Switch between HA Manual, Local Spot Price, or Cloud
- 🚨 **Emergency Cutback** - Auto offset -3 when price exceeds user threshold
- 🌐 **Web Interface** - Local monitoring and control (Web Server v3)
- 🏠 **Home Assistant Ready** - Full integration via ESPHome API + UDP gateway
- 📡 **OTA Updates** - Wireless firmware updates
- 💡 **RGB Status LED** - Blue=booting, Green=connected (lightweight, boot-only)
- 🔧 **Nerd Tab** - IP, MAC, ESPHome Version, WiFi Signal, Uptime
- ⏱️ **Timing-Safe Architecture** - Dual-priority boot, 60s stabilization guard
- 📈 **Independent Price Graphs** - Separate price graphs for today and tomorrow

---

## 📈 V1.4.1 Release Notes

- **HTTP Buffer Truncation Fixed**: The elprisetjustnu.se API now returns ~13.6KB of data for the 96 daily price slots. ESPHome's default `http_request` buffer truncates this to 1024 bytes (resulting in only 1-2 hours of data being read). This release fixes the bug by explicitly setting `max_response_buffer_size: 16384` in the HTTP actions, allowing the pump to properly fetch, parse, and graph all 96 prices for both today and tomorrow.
- **Smart Control Logic Restored**: Re-added the core offset and hot water optimization code that was inadvertently removed in 1.4.0. The controller is now fully autonomous again.
- **Hardware Grounding Warning**: Added critical documentation on RS485 ground bridging to prevent data corruption and Alarm 251 issues.

---

## 🚀 Quick Start

### Prerequisites

- **Hardware**: LilyGo T-CAN485 v1.1 + Nibe F1245 Heat Pump
- **Software**: ESPHome 2026.2.2+, Python 3.9+

### Flashing Pre-built Firmware (No Compilation Needed)
```bash
esptool.py write_flash 0x0 versions/v1.4.1/firmware.factory.bin
```

### Or Compiling from Source
```bash
# Copy the version's YAML to root, edit WiFi credentials, then compile
cp versions/v1.4.1/smartbox.yaml smartbox.yaml
esphome compile smartbox.yaml
esphome run smartbox.yaml
```

---

## 📖 Documentation

| Document | Description |
|----------|-------------|
| **📚 [HEAT_PUMP_OVERVIEW.md](../../docs/HEAT_PUMP_OVERVIEW.md)** | **Complete Nibe F1245 system guide** — sensors, circuits, pumps, degree minutes |
| **[BUILD_GUIDE.md](../../docs/BUILD_GUIDE.md)** | Complete build & deployment guide |
| **[PLANNING.md](../../docs/PLANNING.md)** | Feature brainstorming & V1.5 roadmap |
| **[GITHUB_UPLOAD_GUIDE.md](../../docs/GITHUB_UPLOAD_GUIDE.md)** | How to publish to GitHub |
| **[VSCODE_GIT_GUIDE.md](../../docs/VSCODE_GIT_GUIDE.md)** | VS Code Git workflow |
| **[CHANGELOG.md](../../CHANGELOG.md)** | Version history & bug fixes |

---

**Made with ❤️ for the Nibe heat pump community**

**Version**: 1.4.1  
**Last Updated**: 2026-03-23  
**Tested With**: ESPHome 2026.2.2, LilyGo T-CAN485 v1.1, Nibe F1245
