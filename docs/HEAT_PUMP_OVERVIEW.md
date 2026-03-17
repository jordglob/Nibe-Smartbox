# Nibe F1245 Heat Pump — Complete System Overview

> **Audience:** Smartbox users, installers, and developers who want to understand
> how the Nibe F1245 ground-source heat pump works, what each sensor measures,
> and how the Smartbox spot-price controller interacts with the system.

---

## Table of Contents

1. [What is the F1245?](#what-is-the-f1245)
2. [The Three Circuits](#the-three-circuits)
3. [Circuit Isolation — Why Tap Water Never Mixes with Radiator Water](#circuit-isolation)
4. [The Refrigerant Cycle (How Heat is Moved)](#the-refrigerant-cycle)
5. [All BT Sensors Explained](#all-bt-sensors-explained)
6. [The Heating Cycle (Space Heating)](#the-heating-cycle)
7. [The Domestic Hot Water (DHW) Cycle](#the-dhw-cycle)
8. [DHW Temperature Modes](#dhw-temperature-modes)
9. [The Immersion Heater](#the-immersion-heater)
10. [Degree Minutes — The Brain of the Compressor](#degree-minutes)
11. [How the Smartbox Controls the Heat Pump](#how-the-smartbox-controls-the-heat-pump)

---

## What is the F1245?

The Nibe F1245 is an **all-in-one ground-source heat pump** with an integrated
**180-liter stainless steel domestic hot water (DHW) tank**. It extracts heat
from the ground via a brine circuit and delivers it to both the heating system
(radiators/underfloor) and the hot water tank.

| Property | Value |
|----------|-------|
| **Type** | Ground-source (geothermal) heat pump |
| **DHW Tank Volume** | 180 liters (stainless steel) |
| **Dimensions** | 1800 × 600 × 620 mm |
| **Heat Source 1** | Compressor (via refrigerant cycle) |
| **Heat Source 2** | Immersion heater (electric backup, max 9 kW) |
| **Communication** | MODBUS via RS-485 (9600 baud) |
| **Available Sizes** | 6, 8, 10, 12 kW |

---

## The Three Circuits

The F1245 contains **three physically separate fluid circuits** that never mix.
Heat is transferred between them via **heat exchangers** (metal surfaces where
heat passes through but fluids do not).

```
┌─────────────────────────────────────────────────────────────────┐
│                        Nibe F1245                               │
│                                                                 │
│  CIRCUIT 1: BRINE          CIRCUIT 2: HEATING     CIRCUIT 3:   │
│  (Ground Loop)             (Radiators)            DHW (Tap)     │
│                                                                 │
│  Ground ──► BT10           BT2 ──► Radiators      Cold In      │
│    (Brine In)                (Supply/Flow)           │          │
│       │                         │                    ▼          │
│       ▼                         ▼               ┌────────┐     │
│  ┌─────────┐              ┌──────────┐          │ 180L   │     │
│  │Evaporator│──refrigerant─►│Condenser│─heat──►│ Tank   │     │
│  │(heat     │              │(heat     │         │        │     │
│  │ absorbed)│              │ released)│         │ BT6 ←──│     │
│  └─────────┘              └──────────┘          │(Load)  │     │
│       │                         │               │        │     │
│       ▼                         ▼               │ BT7 ←──│     │
│  Ground ◄── BT11           BT3 ◄── Radiators   │(Top)   │     │
│    (Brine Out)               (Return)           └───┬────┘     │
│                                                     │          │
│                                                  Hot Out       │
│                                                  (to taps)     │
└─────────────────────────────────────────────────────────────────┘
```

---

## Circuit Isolation

This is a critical concept: **the three fluids never mix**.

| Circuit | Fluid | Purpose | Isolated By |
|---------|-------|---------|-------------|
| **Brine** | Ethanol/glycol + water | Absorbs heat from ground | Evaporator heat exchanger |
| **Heating** | Plain water (+ inhibitor) | Carries heat to radiators | Condenser heat exchanger |
| **DHW** | Potable drinking water | Hot water to taps/showers | Stainless steel tank wall + internal coil |

### Why this matters:

- **Tap water (DHW)** is completely isolated in a **stainless steel tank**.
  The heating medium never touches it. Heat is transferred through a
  **coil/heat exchanger inside the tank** — the metal gets hot, the tap water
  absorbs that heat, but the fluids stay separate.

- **Radiator water** circulates in a closed loop. It picks up heat at the
  condenser and releases it through radiators. It never enters the DHW tank.

- **Brine** is a mix of ethanol and water that circulates underground. It
  absorbs geothermal heat and delivers it to the evaporator. It never touches
  either the heating water or the tap water.

```
  Brine (ground)          Radiator water           Tap water
  ─────────────           ──────────────           ──────────
       │                       │                       │
       │    ┌──────────┐       │                       │
       └───►│Evaporator│       │                       │
            │ (heat in)│       │                       │
            └────┬─────┘       │                       │
                 │ refrigerant │                       │
            ┌────▼─────┐       │                       │
            │Condenser │───────┘                       │
            │(heat out)│──────────────────────────────►│
            └──────────┘     (via internal coil        │
                              in DHW tank)             │
```

**Key takeaway:** If you drink the hot water from your tap, it has **never**
been in contact with the brine or the radiator water. The heat pump only
transfers *heat energy*, not fluid.

---

## The Refrigerant Cycle

Inside the heat pump, a **refrigerant** (closed loop, never leaves the unit)
carries heat from the cold side (brine/evaporator) to the hot side
(condenser/heating+DHW):

```
         ┌──────────────────────────────────────┐
         │         REFRIGERANT CYCLE             │
         │         (sealed, internal)            │
         │                                       │
    Low pressure,                          High pressure,
    low temp gas                           high temp gas
         │                                       │
    ┌────▼────┐                            ┌─────┴─────┐
    │EVAPORATOR│◄── Brine heat in          │COMPRESSOR │
    │ (cold)   │                           │ (the pump)│
    └────┬────┘                            └─────▲─────┘
         │                                       │
    Low pressure                           High pressure,
    liquid                                 hot gas
         │                                       │
    ┌────▼──────┐                          ┌─────┴─────┐
    │EXPANSION  │                          │CONDENSER  │
    │VALVE      │                          │ (hot)     │──► Heat to
    └───────────┘                          └───────────┘    radiators
                                                            + DHW tank
```

**Sensors on the refrigerant circuit:**
- **BT14 (Hot Gas)** — Temperature after compressor (highest point, ~80-100°C)
- **BT17 (Suction Gas)** — Temperature before compressor (lowest point)
- **BT12 (Condenser Out)** — Temperature leaving the condenser

---

## All BT Sensors Explained

The F1245 uses **BT-series temperature sensors** placed at strategic points.
Here is the complete list:

### Outdoor & Climate

| Sensor | Register | Name | Location | Measures |
|--------|----------|------|----------|----------|
| **BT1** | 40004 | Outdoor Temp | Outside wall (north side) | Ambient air temperature. The heat pump uses this + the heat curve to calculate how much heating is needed. |

### Heating Circuit (Radiators / Underfloor)

| Sensor | Register | Name | Location | Measures |
|--------|----------|------|----------|----------|
| **BT2** | 40008 | Supply Temp (Flow) | Pipe leaving heat pump → radiators | Temperature of water going OUT to the heating system. Should match the calculated supply temp. |
| **BT3** | 40012 | Return Temp | Pipe returning from radiators | Temperature of water coming BACK from radiators. Lower than BT2 (heat was released). |
| **BT25** | 40025 | External Supply Temp | External pipe (if installed) | Supply temp measured externally (optional, for multi-zone systems). |

### Domestic Hot Water (DHW) Tank

| Sensor | Register | Name | Location | Measures |
|--------|----------|------|----------|----------|
| **BT6** | 40014 | HW Load (Charging) | Middle of DHW tank, at the charging coil | Temperature at the point where heat enters the tank. Rises during charging, falls during standby. |
| **BT7** | 40013 | HW Top | Top of DHW tank | Temperature of the hottest water (what comes out of the tap first). **This is the control sensor** that decides when to start/stop charging. |

### Brine Circuit (Ground Loop)

| Sensor | Register | Name | Location | Measures |
|--------|----------|------|----------|----------|
| **BT10** | 40015 | Brine In | Pipe from ground → evaporator | Temperature of brine returning from the ground. Typically 0-8°C. Higher = more ground heat available. |
| **BT11** | 40016 | Brine Out | Pipe from evaporator → ground | Temperature of brine going back to the ground. Lower than BT10 (heat was extracted). Typically -3 to +5°C. |

### Refrigerant Circuit (Internal)

| Sensor | Register | Name | Location | Measures |
|--------|----------|------|----------|----------|
| **BT12** | 40018 | Condenser Out | Refrigerant leaving condenser | How much heat was delivered. Should be close to BT2. |
| **BT14** | 40017 | Hot Gas | Refrigerant after compressor | Highest temperature in the system (~80-100°C). Used for compressor protection. |
| **BT17** | 40019 | Suction Gas | Refrigerant before compressor | Lowest temperature in refrigerant circuit. Used for superheat calculation. |

### Sensor Map on the Tank

```
┌──────────────────────────────┐
│        F1245 UNIT            │
│                              │
│  ┌────────────────────┐      │
│  │    DHW TANK        │      │
│  │    (180 liters)    │      │
│  │                    │      │
│  │  ══╗  ← BT7       │      │  BT7 = HW Top (control sensor)
│  │    ║  (top)        │      │  Measures: 44-51°C normally
│  │    ║               │      │
│  │    ║ ← Immersion   │      │  Electric backup heater
│  │    ║   Heater      │      │  (9 kW, top of tank)
│  │    ║               │      │
│  │  ══╝               │      │
│  │                    │      │
│  │  ┌──┐ ← BT6       │      │  BT6 = HW Load (charging sensor)
│  │  │//│ (middle)     │      │  Measures: 30-55°C depending on
│  │  │//│ ← Charging   │      │  charging state
│  │  │//│   Coil       │      │
│  │  └──┘              │      │
│  │                    │      │
│  │  ← Cold water in   │      │  Fresh cold water enters at bottom
│  └────────────────────┘      │
│                              │
│  BT14 ← Hot Gas (compressor out)
│  BT12 ← Condenser Out       │
│  BT17 ← Suction Gas         │
│  BT2  → Supply to radiators │
│  BT3  ← Return from radiators│
│  BT10 ← Brine In (from ground)
│  BT11 → Brine Out (to ground)│
│  BT1  ← Outdoor (external)  │
└──────────────────────────────┘
```

---

## The Heating Cycle

When the house needs heat (determined by **Degree Minutes**, see below):

```
Step 1: BT1 (outdoor) is cold → heat curve calculates target supply temp
Step 2: Compressor starts
Step 3: Brine circulates: Ground → BT10 → Evaporator → BT11 → Ground
Step 4: Refrigerant absorbs heat in evaporator
Step 5: Compressor pressurizes refrigerant → BT14 (hot gas, ~80°C)
Step 6: Condenser transfers heat to heating water → BT12
Step 7: Heated water flows out → BT2 (supply temp, e.g. 35-45°C)
Step 8: Water circulates through radiators, releasing heat
Step 9: Cooled water returns → BT3 (return temp, e.g. 25-35°C)
Step 10: Cycle repeats until Degree Minutes reach 0
```

**The heat curve** is a mapping: colder outdoor temp (BT1) → higher supply
temp (BT2). The **Heat Curve Offset** shifts this entire curve up or down.
The Smartbox adjusts this offset based on electricity price.

```
  Supply Temp (BT2)
  50°C ┤                          ╱ Offset +2 (boost)
       │                        ╱
  45°C ┤                      ╱── Offset 0 (normal)
       │                    ╱
  40°C ┤                  ╱──── Offset -3 (emergency)
       │                ╱
  35°C ┤              ╱
       │            ╱
  30°C ┤──────────╱
       └──┬───┬───┬───┬───┬───┬──► Outdoor Temp (BT1)
         -20 -15 -10  -5   0   5°C
```

---

## The Domestic Hot Water (DHW) Cycle

DHW charging has **priority over heating**. When BT7 drops below the start
temperature, the heat pump switches from heating mode to DHW charging:

```
Step 1: BT7 (HW Top) drops below start temp (e.g. 44°C in Normal mode)
Step 2: 3-way valve switches from heating circuit → DHW charging coil
Step 3: Compressor runs, heating the charging coil inside the tank
Step 4: BT6 (HW Load) rises as heat enters the tank
Step 5: Heat rises naturally in the tank (convection)
Step 6: BT7 (HW Top) rises as the top of the tank warms up
Step 7: BT7 reaches stop temp (e.g. 48°C in Normal mode)
Step 8: 3-way valve switches back to heating circuit
Step 9: Tank slowly cools down through standby losses and tap usage
Step 10: When BT7 drops below start temp again → cycle repeats
```

**Important relationships:**
- During charging: **BT6 > BT7** (heat enters at BT6 level, rises to BT7)
- During standby: **BT7 > BT6** (top stays warmest, middle cools faster)
- During tapping: **BT7 drops first** (hot water drawn from top, cold water enters at bottom)

```
  Temperature
  55°C ┤
       │    ┌─── BT6 during charging
  50°C ┤    │  ╲
       │   ╱    ╲──── BT7 (follows BT6 with delay)
  45°C ┤──╱      ╲
       │          ╲         ┌── Next charging cycle
  40°C ┤           ╲       ╱
       │            ╲─────╱ ← Tapping event
  35°C ┤              (BT7 drops below start temp)
       │
  30°C ┤
       └──┬────┬────┬────┬────┬────┬──► Time
         0:00 2:00 4:00 6:00 8:00 10:00
```

---

## DHW Temperature Modes

The F1245 has three hot water comfort levels. The **start temperature** is
when charging begins (BT7 has cooled enough). The **stop temperature** is
when charging ends (BT7 is hot enough).

| Mode | BT7 Start Temp | BT7 Stop Temp | Description |
|------|---------------|--------------|-------------|
| **Economy** | 40°C | 44°C | Minimum comfort. Saves energy. Good for 1-2 person households or when electricity is expensive. |
| **Normal** | 44°C | 48°C | Standard comfort. Suitable for most households. Recommended default. |
| **Luxury** | 47°C | 51°C | Maximum comfort. May engage the immersion heater. Higher operating cost. |

**Note:** These are the temperatures at **BT7 (top of tank)**. The actual tap
water temperature may be slightly lower due to mixing and pipe losses. BT6
(middle of tank) will typically be 2-5°C lower than BT7 during standby.

### What your tap water feels like:

| BT7 Reading | Tap Experience |
|-------------|---------------|
| 51°C | Very hot — needs mixing with cold |
| 48°C | Comfortably hot shower |
| 44°C | Warm but may feel lukewarm at end of shower |
| 40°C | Barely warm — uncomfortable for showers |
| 35°C | Cool — not suitable for comfortable use |
| 31°C | **Your current reading** — essentially cold |

---

## The Immersion Heater

The F1245 has a built-in **electric immersion heater** (up to 9 kW, 3-phase)
located at the **top of the DHW tank**, near BT7.

### When does it activate?

| Situation | Immersion Heater |
|-----------|-----------------|
| Normal operation, compressor handles load | **OFF** |
| Luxury mode, compressor can't reach 51°C fast enough | **ON** (assists compressor) |
| Compressor fault / emergency mode | **ON** (takes over heating, but NOT DHW) |
| Periodic anti-legionella raise (every ~week) | **ON** (heats to ~60°C briefly) |
| Very cold outdoor temps, heating demand exceeds compressor capacity | **ON** (supplements heating) |

**Important:** The immersion heater is a **direct electric heater** — it
converts electricity to heat at nearly 100% efficiency, but electricity is
3-4× more expensive than heat pump operation. The heat pump has a COP
(Coefficient of Performance) of 3-4, meaning 1 kWh of electricity produces
3-4 kWh of heat.

---

## Degree Minutes — The Brain of the Compressor

**Degree Minutes (DM)** is the most important internal value in the heat pump.
It determines when the compressor starts and stops.

### How it works:

```
DM = Σ (Target Supply Temp - Actual Supply Temp) × Time

  - Target is calculated from heat curve + BT1 (outdoor temp)
  - Actual is BT2 (supply temp)
  - Accumulated every minute
```

| DM Value | Meaning | Compressor |
|----------|---------|------------|
| **0** | House is at target temperature | Stops |
| **-30** | Slightly below target (normal) | Running |
| **-60** (default start) | Enough deficit accumulated | **Starts** |
| **-120** | Significant heat deficit | Running at higher speed |
| **+0** | Target reached | **Stops** |

**Example:**
- Target supply temp = 40°C, Actual BT2 = 38°C → deficit = 2°C
- After 30 minutes: DM = -60 → compressor starts
- Compressor heats water, BT2 rises to 40°C → DM climbs back to 0 → stops

### Why this matters for the Smartbox:

When the Smartbox sets **Heat Curve Offset = -3**, it lowers the target supply
temperature by 3°C. This means:
- The deficit is smaller → DM accumulates slower
- The compressor runs less → less electricity used
- But the house cools down gradually

When offset = **+2**, the opposite: compressor runs more, house gets warmer.

---

## How the Smartbox Controls the Heat Pump

The Smartbox does **not** directly control the compressor. Instead, it adjusts
two parameters that influence the heat pump's own control logic:

### 1. Heat Curve Offset (affects heating)

```
Spot Price Analysis → Heat Curve Offset → Target Supply Temp → Degree Minutes → Compressor
```

| Price Zone | Offset | Effect |
|-----------|--------|--------|
| Low (bottom 25%) | +2 | Higher target → more heating → pre-heat the house |
| Normal (middle 50%) | 0 | No change — heat pump runs normally |
| High (top 25%) | -2 | Lower target → less heating → coast on stored heat |
| Emergency (above threshold) | -3 | Significant reduction — save maximum electricity |

### 2. Hot Water Mode (affects DHW)

```
Spot Price Analysis → Hot Water Mode → Start/Stop Temps → DHW Charging Cycle
```

| Price Zone | HW Mode | BT7 Target | Effect |
|-----------|---------|-----------|--------|
| Very low (< 10 öre) | Luxury | 47-51°C | Max hot water — charge while cheap |
| Normal | Normal | 44-48°C | Standard comfort |
| High (top 25%) | Economy | 40-44°C | Minimum comfort — save electricity |

### The Complete Control Flow

```
Every 15 minutes:
  │
  ├─ Fetch spot price from elprisetjustnu.se
  │
  ├─ Calculate daily price statistics (min, max, thresholds)
  │
  ├─ Determine price zone:
  │   │
  │   ├─ Above emergency threshold? → Offset -3, status "EMERGENCY"
  │   ├─ Top 25% of daily range?    → Offset -2, HW Economy
  │   ├─ Bottom 25% of daily range? → Offset +2, HW Normal/Luxury
  │   └─ Middle 50%?                → Offset  0, HW Normal
  │
  └─ Publish to Nibe via Home Assistant service calls
      │
      ├─ number.set_value: heat_curve_offset
      └─ select.select_option: hot_water_mode
```

### What Happened in Your BT6 Graph

Your graph showed BT6 dropping from 48°C to 31°C because:

```
Timeline:
  Night (cheap)  → Normal/Boost mode → BT6 maintained at ~48°C
  Morning        → Prices rising     → Still OK
  Afternoon      → Price hit 100.07 öre (above 100 öre threshold)
                 → EMERGENCY mode activated
                 → Heat Curve Offset set to -3
                 → Compressor effort reduced significantly
                 → DHW tank not actively maintained
                 → 180 liters loses ~1-2°C per hour passively
                 → After several hours: 48°C → 31°C
```

**The fix:** The emergency threshold should be dynamic (relative to the day's
price range) rather than a fixed value. See the Smartbox configuration for
the "Emergency Threshold %" setting.

---

## Quick Reference Card

```
┌─────────────────────────────────────────────────────┐
│              NIBE F1245 QUICK REFERENCE              │
├─────────────────────────────────────────────────────┤
│                                                     │
│  OUTDOOR          BT1  → Heat curve input           │
│  HEATING SUPPLY   BT2  → Water TO radiators         │
│  HEATING RETURN   BT3  → Water FROM radiators       │
│  DHW LOAD         BT6  → Tank charging point        │
│  DHW TOP          BT7  → Tank top (control sensor)  │
│  BRINE IN         BT10 → From ground (warm side)    │
│  BRINE OUT        BT11 → To ground (cold side)      │
│  CONDENSER OUT    BT12 → Refrigerant heat output    │
│  HOT GAS          BT14 → Compressor discharge       │
│  SUCTION GAS      BT17 → Compressor intake          │
│                                                     │
│  TANK: 180L stainless steel                         │
│  HEATER: 9 kW immersion (backup)                    │
│  COMPRESSOR: Variable speed                         │
│  CONTROL: Degree Minutes (DM)                       │
│                                                     │
│  DHW MODES:                                         │
│    Economy  40→44°C  (min comfort)                  │
│    Normal   44→48°C  (standard)                     │
│    Luxury   47→51°C  (max comfort, uses heater)     │
│                                                     │
│  THREE CIRCUITS (never mix):                        │
│    1. Brine    (ground loop, ethanol+water)          │
│    2. Heating  (radiators, closed loop water)        │
│    3. DHW      (tap water, stainless steel tank)     │
│                                                     │
└─────────────────────────────────────────────────────┘
```

---

*Document version: 1.0 — 2026-03-17*
*Part of the Revolt Nibe Smartbox project*
