# Nibe Smartbox v1.2
Planering för V1.3
här är vi kvar på .yaml nivå, ingen ändring av NibeGW orginalkod
1. Kan NibeGW HELA projekt inklusive cloes and open issues laddas ner lokalt på hårddisken, verkar som vissa AI inte kan nå hemsidan...
2. Kan en EXTREMT enkel Indikator LED läggas till i yaml koden.
3. Hur får jag tillgång till best practice för parsing nibes modbus protokoll. 
4. behöver jag ange Nibe-modell eller är modbus- protokollet fungerande generellt?
5. hur vet jag vad som styr just nu, säg att HA är inkopplat och programerad för styrning. samtidigt får liligo in elpriset via nätet (eller via ett anrop till server.relvolt-energy.org som jag har planer på att lägga en mini AI för styrning baserat på väderprognos effkekttariff feeback från tibber API mm mm) kanske en spak som jag själv väljer.
6. nu finns det hårdkodade parametrar i koden, kan man ha en "kongiurator" man går igenom där man bland annat väljer vilket SE-område eller land. frågan är generell lista alla parametrar med rekomendation
7. Kan websidan få en nörd-flik-knapp? där tänker jag att senast tilldelad ipnummer, Mackadress builddatum version mm mm presenteras 
sists. föreslå om man ska dela upp detta olika byggen?hur komplex kan man göra deta innan det blir ostablit? när HA är anslutet 

**Commercial-grade smart controller firmware for Nibe heat pumps with autonomous electricity spot price optimization**

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
- 💰 **Smart Price Optimization** - Automatic heating adjustments based on spot prices
- 🌐 **Web Interface** - Local monitoring and control (Web Server v3)
- 🏠 **Home Assistant Ready** - Full integration via ESPHome API
- 📡 **OTA Updates** - Wireless firmware updates
- ⚡ **Real-time Control** - Dynamic heat curve and hot water mode adjustments

---

## 📊 How It Works

```
┌─────────────────┐
│  Spot Price API │ (elprisetjustnu.se)
└────────┬────────┘
         │ Hourly fetch
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

- **[BUILD_GUIDE.md](BUILD_GUIDE.md)** - Complete build & deployment guide
  - Architecture & design decisions
  - Compilation challenges & solutions
  - Troubleshooting guide
  - Configuration reference

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

- **RAM Usage**: 11.7% (38,456 bytes)
- **Flash Usage**: 56.5% (1,036,027 bytes)
- **Spot Price Fetch**: ~1.3 seconds
- **Update Frequency**: Every 60 minutes
- **RS485 Communication**: Stable (continuous token passing)

---

## 🔧 Troubleshooting

### Common Issues

**Problem**: "Time not synced yet"
- **Solution**: Increase boot delay or check SNTP servers

**Problem**: "Response may be truncated"
- **Solution**: Buffer is at 8192 bytes limit - API response is large

**Problem**: No RS485 communication
- **Solution**: Verify wiring and ensure NO `dir_pin` configured

See [BUILD_GUIDE.md](BUILD_GUIDE.md#troubleshooting) for detailed troubleshooting.

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
- **Documentation**: [BUILD_GUIDE.md](BUILD_GUIDE.md)
- **ESPHome**: [ESPHome Documentation](https://esphome.io)

---

**Made with ❤️ for the Nibe heat pump community**

**Version**: 1.2  
**Last Updated**: 2026-03-15  
**Tested With**: ESPHome 2026.2.2, LilyGo T-CAN485 v1.1, Nibe F1245
