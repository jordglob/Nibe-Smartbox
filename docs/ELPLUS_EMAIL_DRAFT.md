# E-postutkast till Elplus

> Kopiera texten nedan och skicka via e-post. Anpassa mottagare efter behov.

---

**Ämne:** Open source spotprisoptimering för Nibe värmepumpar — samarbete?

**Till:** [Elplus kontakt / GitHub-ägare]

---

Hej!

Jag heter Jordan och driver ett open source-projekt som heter **Revolt Nibe Smartbox** — en autonom spotpris-controller för Nibe F1245 värmepumpar, byggd på ESPHome och LilyGo T-CAN485.

**Vad den gör:**
- Hämtar spotpriser var 15:e minut från elprisetjustnu.se (96 prisslots/dag)
- Justerar värmekurvan automatiskt: boost vid billig el, reducering vid dyr el
- Styr varmvattenläge (Economy/Normal/Luxury) baserat på pris
- Dynamisk nödtröskel (% av dagens maxpris) — undviker att frysa ut huset
- Fungerar helt autonomt utan Home Assistant (men stödjer det också)
- Komplett RS485/MODBUS-gateway (NibeGW) inbyggd

**Tekniskt:**
- ESP32-baserad (LilyGo T-CAN485 v1.1)
- ESPHome YAML — ingen C++-kompilering behövs
- Webbgränssnitt för lokal övervakning
- OTA-uppdateringar
- MIT-licens

Projektet är nu publikt på GitHub:
🔗 **https://github.com/jordglob/Nibe-Smartbox**

Jag har även skrivit en omfattande utbildningsdokumentation om hur Nibe F1245 fungerar internt (sensorer, kretsar, cirkulationspumpar, gradminuter) som kan vara intressant för er community:
📚 [Heat Pump Overview](https://github.com/jordglob/Nibe-Smartbox/blob/main/docs/HEAT_PUMP_OVERVIEW.md)

**Varför jag kontaktar er:**
Jag såg ert projekt på GitHub och tänkte att det kunde vara intressant att dela med sig av detta. Om ni vill kan ni gärna länka till projektet, forka det, eller bara ta en titt. Alla bidrag och feedback är välkomna!

Hör gärna av er om ni har frågor eller vill diskutera samarbete.

Med vänliga hälsningar,
Jordan

---

*Revolt Nibe Smartbox v1.3.1 — MIT License*
*Testat med: ESPHome 2026.2.2, LilyGo T-CAN485 v1.1, Nibe F1245*
