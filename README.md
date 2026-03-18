# Revolt Nibe Smartbox

**Commercial-grade smart controller firmware for Nibe heat pumps with autonomous electricity spot price optimization**

[![ESPHome](https://img.shields.io/badge/ESPHome-2026.2.2-blue.svg)](https://esphome.io)
[![License](https://img.shields.io/badge/License-MIT-green.svg)](LICENSE)
[![Hardware](https://img.shields.io/badge/Hardware-LilyGo%20T--CAN485%20v1.1-orange.svg)](https://www.lilygo.cc/)

---

## 🎯 Overview

Nibe Smartbox transforms your Nibe F1245 heat pump into an intelligent, cost-optimizing system that automatically adjusts heating based on real-time electricity spot prices. Built on ESPHome for the LilyGo T-CAN485 v1.1 ESP32 board, it serves dual purposes:

1. **UDP Gateway (NibeGW)** - Full Home Assistant integration via RS485
2. **Standalone Smart Controller** - Autonomous price-based optimization

All features, instructions, and compiled firmware are kept completely self-contained in version-specific folders. This means you can easily browse, download, and read the exact documentation for the specific version you want to use.

---

## 📦 Choose Your Version

Click on a version below to access its specific `README` instructions, YAML source code, and pre-compiled `.bin` files ready for flashing:

| Version | Key Changes | Link to Version Folder |
|---------|-------------|-------------------------|
| **v1.3.2** ⭐ | Fixed 15-min sparkline resolution | 📁 **[Go to v1.3.2](versions/v1.3.2/)** (Recommended) |
| v1.3.1 | Dynamic emergency threshold (% of daily max), DHW bugfix | 📁 [Go to v1.3.1](versions/v1.3.1/) |
| v1.3.0 | 15-min price slots, tomorrow prices, sparkline | 📁 [Go to v1.3.0](versions/v1.3/) |
| v1.2.0 | Basic spot price optimization, 60-min updates | 📁 [Go to v1.2.0](versions/v1.2/) |

> 💡 **Tip:** Each folder contains a `firmware.factory.bin` for first-time USB flashing, and a `firmware.ota.bin` for wireless web updates. The root `smartbox.yaml` is always identical to the latest recommended version.

---

## 📖 Global Documentation

General documents that apply across all versions of the project:

| Document | Description |
|----------|-------------|
| **📚 [HEAT_PUMP_OVERVIEW.md](docs/HEAT_PUMP_OVERVIEW.md)** | **Complete Nibe F1245 system guide** — sensors, circuits, pumps, degree minutes |
| **[BUILD_GUIDE.md](docs/BUILD_GUIDE.md)** | Hardware build & deployment guide (LilyGo T-CAN485) |
| **[PLANNING.md](docs/PLANNING.md)** | Feature brainstorming & roadmap |
| **[CHANGELOG.md](CHANGELOG.md)** | Detailed version history & bug fixes |
| **[GITHUB_UPLOAD_GUIDE.md](docs/GITHUB_UPLOAD_GUIDE.md)** | How to publish to GitHub |

---

## 🤝 Contributing & Support

Contributions welcome! Areas of interest:
- Alternative spot price APIs
- Enhanced smart control algorithms
- Multi-language support

For issues, please use the [GitHub Issues](https://github.com/jordglob/nibe-smartbox/issues) tab.

---

**Made with ❤️ for the Nibe heat pump community**

*AI Authorship: Initial architecture by Claude 3.7 Sonnet. Directory design & v1.3.2 fixes by Gemini 3.1 Pro.*
