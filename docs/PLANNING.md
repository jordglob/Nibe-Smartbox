# Planning & Feature Brainstorming

**Internal development notes — moved from README.md**

---

## V1.3 Planning Notes (Original Swedish)

Planering för V1.3
här är vi kvar på .yaml nivå, ingen ändring av NibeGW orginalkod

1. Kan NibeGW HELA projekt inklusive cloes and open issues laddas ner lokalt på hårddisken, verkar som vissa AI inte kan nå hemsidan...
2. Kan en EXTREMT enkel Indikator LED läggas till i yaml koden.
3. Hur får jag tillgång till best practice för parsing nibes modbus protokoll. 
4. behöver jag ange Nibe-modell eller är modbus- protokollet fungerande generellt?
5. hur vet jag vad som styr just nu, säg att HA är inkopplat och programerad för styrning. samtidigt får liligo in elpriset via nätet (eller via ett anrop till server.relvolt-energy.org som jag har planer på att lägga en mini AI för styrning baserat på väderprognos effkekttariff feeback från tibber API mm mm) kanske en spak som jag själv väljer.
6. nu finns det hårdkodade parametrar i koden, kan man ha en "kongiurator" man går igenom där man bland annat väljer vilket SE-område eller land. frågan är generell lista alla parametrar med rekomendation
7. Kan websidan få en nörd-flik-knapp? där tänker jag att senast tilldelad ipnummer, Mackadress builddatum version mm mm presenteras 
sists. föreslå om man ska dela upp detta olika byggen? hur komplex kan man göra deta innan det blir ostablit? när HA är anslutet 

---

## V1.3 Status — IMPLEMENTED ✅

| # | Feature Request | Status | Implementation |
|---|---|---|---|
| 1 | Download elupus project locally | ✅ Done | Analyzed `nibe_issues.json` (25+ issues) |
| 2 | Simple LED indicator | ✅ Done | GPIO4 RGB via `esp32_rmt_led_strip`, internal, boot-only |
| 3 | Modbus parsing best practice | ✅ Done | Rely on `elupus/esphome-nibe` (UDP gateway) |
| 4 | Nibe model specification | ✅ Done | F1245 via `acknowledge: MODBUS40` |
| 5 | Control mode selector | ✅ Done | "The Brain" — 3 modes: HA Manual / Local Spot / Cloud |
| 6 | Configurator | ✅ Partial | Max Price Threshold saved to flash. Region ready but not wired |
| 7 | Nerd tab | ✅ Done | IP, MAC, ESPHome Version, WiFi Signal, Uptime |

---

## V1.4 Roadmap Ideas

- [ ] **Tomorrow's prices** — Fetch day-ahead prices after 13:00 CET
- [ ] **Region selector wired to URL** — SE1/SE2/SE3/SE4 dropdown changes API endpoint
- [ ] **Cloud mode (Revolt Energy)** — API endpoint at `server.revolt-energy.org`
- [ ] **Weather forecast integration** — Adjust heating based on outdoor temp prediction
- [ ] **Effekttariff awareness** — Peak power tariff avoidance
- [ ] **Tibber API** — Alternative price source with push notifications
- [ ] **Standalone Visualization** — Custom Nibe register parsing, visual pump display
- [ ] **Multi-language web UI** — Swedish/English toggle
- [ ] **Historical price logging** — Store 7 days of price data on flash
- [ ] **Mobile-friendly web UI** — Responsive CSS for phone access

---

**Last Updated**: 2026-03-17
