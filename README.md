# Revolt Nibe Smartbox v1.3.1

**Commercial-grade smart controller firmware for Nibe heat pumps with autonomous electricity spot price optimization**

> 📋 *Development planning notes moved to [docs/PLANNING.md](docs/PLANNING.md)*

[![ESPHome](https://img.shields.io/badge/ESPHome-2026.2.2-blue.svg)](https://esphome.io)
[![License](https://img.shields.io/badge/License-MIT-green.svg)](LICENSE)
[![Hardware](https://img.shields.io/badge/Hardware-LilyGo%20T--CAN485%20v1.1-orange.svg)](https://www.lilygo.cc/)

---

## 🎯 Overview

Nibe Smartbox transforms your Nibe F1245 heat pump into an intelligent, cost-optimizing system that automatically adjusts heating based on real-time electricity spot prices. Built on ESPHome for the LilyGo T-CAN485 v1.1 ESP32 board, it serves dual purposes:

1. **UDP Gateway (NibeGW)** - Full Home Assistant integration via RS485
2. **Standalone Smart Controller** - Autonomous price-based optimization

### ✨ Key Features

- 🔌 **Plug & Play RS485** - Hardware auto-direction (critical for T-CAN485 v1.1)
- 💰 **15-Min Spot Price Optimization** - Quarter-hour price matching (96 slots/day)
- 🧠 **Control Mode "The Brain"** - Switch between HA Manual, Local Spot Price, or Cloud
- 🚨 **Emergency Cutback** - Auto offset -3 when price exceeds user threshold
- 🌐 **Web Interface** - Local monitoring and control (Web Server v3)
- 🏠 **Home Assistant Ready** - Full integration via ESPHome API + UDP gateway
- 📡 **OTA Updates** - Wireless firmware updates
- 💡 **RGB Status LED** - Blue=booting, Green=connected (lightweight, boot-only)
- 🔧 **Nerd Tab** - IP, MAC, ESPHome Version, WiFi Signal, Uptime
- ⚡ **Timing-Safe Architecture** - Dual-priority boot, 60s stabilization guard

---

## 📊 How It Works

```
┌─────────────────┐
│  Spot Price API │ (elprisetjustnu.se)
└────────┬────────┘
         │ Every 15 min
         ▼
┌─────────────────┐
│  ESP32 Smartbox │
│  (LilyGo T-CAN) │
└────────┬────────┘
         │ RS485
         ▼
┌─────────────────┐      ┌──────────────────┐
│ Nibe F1245 HP   │◄────►│ Home Assistant   │
│ Heat Pump       │      │ (Optional)       │
└─────────────────┘      └──────────────────┘
```

### Smart Control Logic

| Price Level | Heat Curve Offset | Hot Water Mode |
|-------------|-------------------|----------------|
| **Low** (bottom 25%) | +2 (boost heating) | Normal |
| **Normal** (middle 50%) | 0 (standard) | Normal |
| **High** (top 25%) | -2 (reduce heating) | Economy |
| **Extremely Low** (<10 öre) | +2 | **Luxury** |
| **Above User Max** | -3 (emergency) | Economy |

---

## 📦 Choosing a Version

Each version includes both the **ESPHome YAML** (for compiling) and **pre-built firmware** (for flashing directly):

```
firmware/
├── v1.2/          ← Basic spot price, 60-min updates
│   ├── smartbox.yaml
│   ├── firmware.bin
│   ├── firmware.factory.bin
│   └── firmware.ota.bin
├── v1.3/          ← 15-min resolution, tomorrow prices, sparkline
│   ├── smartbox.yaml
│   ├── firmware.bin
│   ├── firmware.factory.bin
│   └── firmware.ota.bin
└── v1.3.1/        ← Dynamic emergency threshold, DHW bugfix (latest)
    ├── smartbox.yaml
    ├── firmware.bin
    ├── firmware.factory.bin
    └── firmware.ota.bin
```

| Version | Key Changes | Compile From |
|---------|-------------|--------------|
| **v1.3.1** ⭐ | Dynamic emergency threshold (% of daily max), DHW bugfix | `firmware/v1.3.1/smartbox.yaml` |
| v1.3.0 | 15-min price slots, tomorrow prices, sparkline | `firmware/v1.3/smartbox.yaml` |
| v1.2.0 | Basic spot price optimization, 60-min updates | `firmware/v1.2/smartbox.yaml` |

**To compile a specific version:**
```bash
# Copy the version's YAML to root, edit WiFi credentials, then compile
cp firmware/v1.3.1/smartbox.yaml smartbox.yaml
esphome compile smartbox.yaml
```

**To flash pre-built firmware (no compilation needed):**
```bash
esptool.py write_flash 0x0 firmware/v1.3.1/firmware.factory.bin
```

> 💡 The root `smartbox.yaml` always contains the **latest** version.

---

## 🚀 Quick Start

### Prerequisites

- **Hardware**: LilyGo T-CAN485 v1.1 + Nibe F1245 Heat Pump
- **Software**: ESPHome 2026.2.2+, Python 3.9+

### Installation

```bash
# 1. Clone repository
git clone https://github.com/jordglob/nibe-smartbox.git
cd nibe-smartbox

# 2. Install ESPHome
pip install esphome

# 3. Edit configuration
# Update WiFi credentials in smartbox.yaml

# 4. Compile & Upload
esphome run smartbox.yaml
```

### First Boot

After upload, monitor logs for successful spot price fetch:

```
[12:31:08][I][spot_price]: ✓ Current spot price: 55.19 öre/kWh
[12:31:08][I][spot_price]: Daily range: 27.26 - 76.88 öre/kWh
[12:31:08][I][smart_control]: NORMAL PRICE - Setting Heat Curve Offset to 0
```

---

## 📖 Documentation

| Document | Description |
|----------|-------------|
| **[docs/BUILD_GUIDE.md](docs/BUILD_GUIDE.md)** | Complete build & deployment guide |
| **[docs/PLANNING.md](docs/PLANNING.md)** | Feature brainstorming & V1.4 roadmap |
| **[docs/GITHUB_UPLOAD_GUIDE.md](docs/GITHUB_UPLOAD_GUIDE.md)** | How to publish to GitHub |
| **[docs/VSCODE_GIT_GUIDE.md](docs/VSCODE_GIT_GUIDE.md)** | VS Code Git workflow |
| **[CHANGELOG.md](CHANGELOG.md)** | Version history & bug fixes |

---

## ⚙️ Configuration

### Essential Settings

Edit `smartbox.yaml`:

```yaml
wifi:
  ssid: "YOUR_WIFI_SSID"
  password: "YOUR_WIFI_PASSWORD"

ota:
  password: "YOUR_OTA_PASSWORD"
```

### Hardware Pins (LilyGo T-CAN485 v1.1)

| Pin | Function | Notes |
|-----|----------|-------|
| GPIO19 | RS485 Enable | Inverted |
| GPIO17 | Auto-Direction | Inverted |
| GPIO16 | 5V Enable | Inverted |
| GPIO21 | UART RX | 9600 baud |
| GPIO22 | UART TX | 9600 baud |

⚠️ **CRITICAL**: Do NOT configure `dir_pin` - hardware handles direction automatically!

---

## 📈 Performance

| Version | RAM | Flash | Update Interval |
|---------|-----|-------|-----------------|
| **V1.3** | 12.0% (39,276 bytes) | 58.5% (1,073,939 bytes) | Every 15 min |
| V1.2 | 11.7% (38,456 bytes) | 56.5% (1,036,027 bytes) | Every 60 min |

- **HTTP Buffer**: 16,384 bytes (API response ~13.6KB)
- **Boot Stabilization**: 60s guard before spot price fetch
- **RS485 Communication**: Stable (continuous token passing, no alarm 251)

---

## 🔧 Troubleshooting

### Common Issues

**Problem**: "Time not synced yet"
- **Solution**: Increase boot delay or check SNTP servers

**Problem**: "Response may be truncated"
- **Solution**: Buffer is at 8192 bytes limit - API response is large

**Problem**: No RS485 communication
- **Solution**: Verify wiring and ensure NO `dir_pin` configured

See [docs/BUILD_GUIDE.md](docs/BUILD_GUIDE.md#troubleshooting) for detailed troubleshooting.

---

## 🛣️ Roadmap

- [ ] **Standalone Competition Integration** 🎨
  - Custom Nibe register parsing component
  - Visual pump representation (temperatures, status, flow)
  - Real-time display on local web interface
  - Direct ESP32 sensor access (no HA dependency)
- [ ] Multi-region support (SE1-SE4, NO, DK, FI)
- [ ] Alternative spot price APIs
- [ ] Weather forecast integration
- [ ] Historical data logging
- [ ] Mobile app

---

## 🤝 Contributing

Contributions welcome! Areas of interest:

- Alternative spot price APIs
- Enhanced smart control algorithms
- Better error handling
- Documentation improvements
- Multi-language support

### Development

```bash
# Test compilation
esphome compile smartbox.yaml

# Deploy to device
esphome run smartbox.yaml

# Monitor logs
esphome logs smartbox.yaml
```

---

## 📜 License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

---

## 🙏 Credits

- **ESPHome** - https://esphome.io
- **elupus/esphome-nibe** - https://github.com/elupus/esphome-nibe
- **elprisetjustnu.se** - Spot price API provider
- **LilyGo** - T-CAN485 hardware

---

## ⚠️ Disclaimer

This firmware is provided "as is" without warranty. Use at your own risk. Always monitor your heat pump operation and have backup heating available. The author is not responsible for any damage to equipment or property.

---

## 📞 Support

- **Issues**: [GitHub Issues](https://github.com/jordglob/nibe-smartbox/issues)
- **Documentation**: [docs/BUILD_GUIDE.md](docs/BUILD_GUIDE.md)
- **ESPHome**: [ESPHome Documentation](https://esphome.io)

---

**Made with ❤️ for the Nibe heat pump community**

**Version**: 1.3.1  
**Last Updated**: 2026-03-17  
**Tested With**: ESPHome 2026.2.2, LilyGo T-CAN485 v1.1, Nibe F1245
