# AirHealth — Air Quality & CO2 Monitor Device
## Project Plan

---

## Overview

AirHealth is a small consumer device that continuously monitors indoor air quality and CO2 concentration. It communicates air status at a glance using a single RGB indicator light: green (good), yellow (caution), red (poor). Target use cases: home offices, bedrooms, living rooms.

The device connects to home WiFi and logs readings to a cloud backend. The LED works independently of the cloud — if WiFi is unavailable the light still functions. No screen, no buzzer, no app required to read the light.

## Locked Decisions (2026-04-29)

| Decision | Choice |
|---|---|
| WiFi / cloud logging | Yes — device POSTs readings to cloud backend every 60s |
| OLED display | No |
| Buzzer | No |
| Target market | Consumer (home offices, bedrooms) |
| Prototype sourcing | Breakout boards on breadboard first |
| Prototype AQ sensor | PMS5003 (PM) + SGP30 (VOC) — swap to SEN55 for PCB version |
| Microcontroller | ESP32-C3 (WiFi built in, no extra cost) |
| CO2 sensor | Sensirion SCD40 |
| LED | WS2812B NeoPixel (single) |
| Power | USB-C bus power, v1 only |

---

## Stage 1 — Requirements & Thresholds

### What the Device Measures

| Measurement | Sensor Type | Why It Matters |
|---|---|---|
| CO2 (ppm) | NDIR (Non-Dispersive Infrared) | Primary indicator of ventilation quality and cognitive impact |
| VOCs (volatile organic compounds) | MOX gas sensor | Paints, cleaning products, off-gassing furniture |
| Particulate Matter (PM2.5 / PM10) | Optical particle counter | Dust, smoke, allergens |
| Temperature & Humidity | Capacitive / resistive | Context for sensor accuracy; bonus data |

### Indicator Logic

The LED color is determined by the worst of all active readings:

**CO2 Thresholds (ppm):**

| Color | Range | Meaning |
|---|---|---|
| Green | < 800 ppm | Good — well ventilated |
| Yellow | 800–1,200 ppm | Caution — open a window |
| Red | > 1,200 ppm | Poor — ventilate immediately |

**PM2.5 Thresholds (µg/m³, based on EPA AQI breakpoints):**

| Color | Range | AQI Equivalent |
|---|---|---|
| Green | 0–12 | Good (0–50) |
| Yellow | 12.1–35.4 | Moderate (51–100) |
| Red | > 35.4 | Unhealthy (101+) |

**VOC Thresholds (index score, Sensirion VOC Index):**

| Color | Range |
|---|---|
| Green | 0–100 |
| Yellow | 101–200 |
| Red | > 200 |

**Combined rule:** LED shows the worst (highest) status across all three measurements.

---

## Stage 2 — Hardware Selection

### Core Components

#### Microcontroller

| Option | Cost | Notes |
|---|---|---|
| ESP32-C3 mini | ~$3 | WiFi/BLE capable, small, 3.3V I2C/UART, excellent for future expansion |
| Raspberry Pi Pico W | ~$6 | WiFi, MicroPython friendly, easy to prototype |
| Arduino Nano | ~$4 | Simple, no WiFi, fine for LED-only v1 |

**Recommendation:** ESP32-C3 (or ESP32-S3 mini). WiFi is not needed for v1 but adds future value (optional data logging, app integration) at negligible cost difference.

---

#### CO2 Sensor

| Option | Cost | Interface | Notes |
|---|---|---|---|
| Sensirion SCD40 | ~$15 | I2C | True NDIR, compact, accurate, industry standard |
| Sensirion SCD41 | ~$20 | I2C | Same as SCD40 + on-chip altitude compensation |
| MH-Z19C | ~$12 | UART/PWM | Good budget option, slightly bulkier |
| Senseair S8 | ~$25 | UART | Industrial grade, used in commercial HVAC |

**Recommendation:** **SCD40** — best balance of size, accuracy, and price. The SCD41 is worth the $5 premium if altitude compensation matters.

---

#### Air Quality Sensor (VOC + PM)

**Option A — Two separate sensors:**
- SGP41 (VOC, NOx index) ~$8 + PMS5003 or SPS30 (PM2.5/PM10) ~$15–$25
- More accurate, more wiring

**Option B — All-in-one:**

| Sensor | Cost | Measures | Interface |
|---|---|---|---|
| Sensirion SEN55 | ~$35 | PM1/2.5/4/10, VOC, NOx, Temp, Humidity | I2C |
| Sensirion SEN54 | ~$30 | PM1/2.5/4/10, VOC, Temp, Humidity (no NOx) | I2C |
| Plantower PMS5003 + SGP30 | ~$20 total | PM + VOC separately | UART + I2C |

**Recommendation:** **SEN55** for production — single sensor, single I2C connection, covers everything, Sensirion ecosystem matches SCD40. Use PMS5003 + SGP30 for initial prototyping (cheaper, more available).

---

#### LED Indicator

| Option | Cost | Notes |
|---|---|---|
| WS2812B NeoPixel (single) | ~$0.50 | Addressable RGB, single data pin, easy brightness control |
| Common-cathode RGB LED | ~$0.10 | Simple, needs 3 PWM pins + resistors |
| Adafruit NeoPixel Jewel | ~$6 | 7-LED cluster, very visible |

**Recommendation:** **WS2812B single NeoPixel** for v1 — one data pin, software brightness control, no resistor math. Use a diffused lens or frosted acrylic over it to soften the light.

---

#### Power

| Option | Notes |
|---|---|
| USB-C (bus powered) | Simplest — plug into wall adapter or USB port. No battery management needed. |
| USB-C + 500mAh LiPo | Portable, 6–8h battery life, adds $5–8 in BOM (LiPo + TP4056 charger IC) |

**Recommendation:** **USB-C bus power only for v1.** Battery is a v2 feature.

---

#### Enclosure

| Option | Cost | Notes |
|---|---|---|
| 3D printed (PLA/PETG) | ~$1–3 material | Full custom, venting slots for sensors, frosted window for LED |
| Hammond 1551 series ABS box | ~$4–6 | Off-the-shelf, drill for sensor vents and LED |
| Custom injection mold | $3,000–8,000 tooling | Only viable at 1,000+ unit production volume |

**Recommendation:** **3D printed for prototype and small batch (<100 units). Injection mold feasibility study at 500+ units.**

---

### Full BOM (Bill of Materials) — Per Unit

| Component | Part | Unit Cost (qty 1) | Unit Cost (qty 100) |
|---|---|---|---|
| Microcontroller | ESP32-C3 mini module | $3.50 | $2.20 |
| CO2 sensor | Sensirion SCD40 | $15.00 | $11.00 |
| AQ sensor | Sensirion SEN55 | $35.00 | $26.00 |
| LED | WS2812B NeoPixel | $0.50 | $0.25 |
| Power connector | USB-C breakout or direct PCB connector | $0.80 | $0.40 |
| Capacitors, resistors, passives | — | $0.50 | $0.20 |
| PCB (custom, 2-layer) | JLCPCB / PCBWay | $3.00 | $1.20 |
| Enclosure (3D print) | PLA/PETG material | $2.00 | $1.50 |
| USB-C cable (included) | Generic | $1.50 | $0.90 |
| Packaging | Cardboard box, foam insert | $1.00 | $0.60 |
| **Total BOM** | | **~$62.80** | **~$44.25** |

> Note: qty-1 cost is prototyping/hobby sourcing (Mouser, Adafruit, Pimoroni). Qty-100 assumes Mouser/Digi-Key + JLCPCB assembly. Qty-1000+ would reduce further, especially SEN55 and SCD40.

---

## Stage 3 — Firmware Development

**Language:** C/C++ via Arduino framework (PlatformIO).

**Libraries:**
- `Sensirion Arduino Core` — SCD40 driver
- `Adafruit PM25 AQI Sensor` — PMS5003 driver (prototype)
- `Adafruit SGP30` — VOC sensor driver (prototype)
- `Adafruit NeoPixel` — LED control
- `ArduinoJson` — serialize readings for cloud POST
- `WiFiManager` — captive portal for WiFi credential setup (user connects to device hotspot on first boot, enters home WiFi details — no hardcoded credentials)
- `HTTPClient` (ESP32 built-in) — POST readings to cloud backend

### Firmware Architecture

```
boot:
  1. Start WiFiManager — if no saved credentials, open hotspot "AirHealth-Setup"
  2. User connects to hotspot, enters WiFi via captive portal
  3. Device saves credentials, connects to home WiFi
  4. LED pulses white (warm-up mode) for 30s (SCD40 stabilization)

main loop (1Hz):
  1. Read SCD40   → co2_ppm, temp_c, humidity_rh
  2. Read PMS5003 → pm25_ug, pm10_ug
  3. Read SGP30   → voc_ppb, eco2_ppb (use SCD40 temp/humidity to compensate SGP30)
  4. Evaluate thresholds → status = worst(co2_status, pm25_status, voc_status)
  5. Set LED color (green / yellow / red)

cloud loop (every 60s):
  6. POST JSON payload to cloud API
  7. On failure: retry once, then continue — LED never depends on cloud
```

### Cloud Payload (JSON)

```json
{
  "device_id": "aa:bb:cc:dd:ee:ff",
  "ts": 1714400000,
  "co2_ppm": 847,
  "pm25_ug": 8.2,
  "pm10_ug": 11.4,
  "voc_ppb": 120,
  "temp_c": 21.4,
  "humidity_rh": 48.2,
  "status": "yellow"
}
```

### Sensor Warm-Up

- SCD40: ~30s for first stable reading. LED pulses white during warm-up.
- PMS5003: ~10s fan spin-up.
- SGP30: requires 15s init, then uses SCD40 humidity/temp for compensation. First reliable VOC readings after ~30s.
- All warm-up runs in parallel. LED goes live after the longest (30s).

### SGP30 Humidity Compensation

SGP30 accuracy improves significantly when fed real humidity and temperature. Feed it the SCD40 readings every loop iteration before reading VOC.

### Calibration

- SCD40 Automatic Self-Calibration (ASC) enabled by default — assumes device is in a ventilated space at least once a week.
- Forced calibration via serial command for production QA.

### Firmware Stages

- **v0.1** — SCD40 + NeoPixel only. CO2 → LED. WiFi disabled. Validates sensor wiring and threshold logic.
- **v0.2** — Add PMS5003 + SGP30. Combined threshold logic. All three sensors driving LED.
- **v0.3** — Add WiFiManager setup portal and 60s cloud POST. LED still works offline.
- **v0.4** — Warm-up animation, LED breathing effect on yellow, solid red on red.
- **v1.0** — Watchdog timer, OTA update support, serial calibration command, device ID from MAC address.

### Deliverables

- [ ] PlatformIO project scaffold under `air_health/firmware/`
- [ ] v0.1: SCD40 + NeoPixel, CO2 threshold → LED
- [ ] v0.2: PMS5003 + SGP30 integration, combined status
- [ ] v0.3: WiFiManager + cloud POST
- [ ] v0.4: Animations and LED polish
- [ ] v1.0: Watchdog, OTA stub, serial calibration

---

## Stage 4 — PCB Design

For prototype: use breakout boards on a breadboard or Adafruit FeatherWing system.
For production: design a custom 2-layer PCB.

**PCB Scope:**
- ESP32-C3 module footprint (SMD)
- JST-SH connectors for SCD40 and SEN55 (I2C, 4-pin Qwiic/STEMMA QT compatible)
- USB-C port with ESD protection (USBLC6-2SC6)
- 3.3V LDO regulator (AMS1117-3.3 or AP2112K)
- WS2812B footprint with decoupling cap
- Optional: UART test pads for serial calibration
- Optional: BOOT and RESET buttons (recessed, accessible via pin hole in enclosure)

**PCB Fabrication:**
- Prototype: JLCPCB 5-board minimum run ~$5 + shipping
- Production: JLCPCB or PCBWay with SMT assembly service — send BOM + CPL file, boards arrive mostly assembled

**Deliverables:**
- [ ] Schematic (KiCad)
- [ ] PCB layout (KiCad)
- [ ] Gerber files for fab
- [ ] BOM + CPL for JLCPCB assembly
- [ ] 5-unit prototype order

---

## Stage 5 — Enclosure Design

**Tool:** Fusion 360 or FreeCAD (open source).

**Design Requirements:**
- Sensor venting: slots or holes on sides/bottom for passive airflow to sensors (CO2 and PM sensors need ambient air, not trapped air)
- LED window: frosted or diffused panel on front face — NeoPixel light should be visible across a room
- USB-C cutout on bottom or back
- Snap-fit or 2-screw assembly (no glue)
- Flat base (sits on desk) or keyhole slot on back (wall mount)
- Max dimensions target: 60mm × 60mm × 30mm

**Deliverables:**
- [ ] Fusion 360 / FreeCAD model
- [ ] STL files for FDM printing
- [ ] Print settings (0.2mm layer, 20% infill, PETG for temperature resistance)
- [ ] Test fit with real PCB and sensors

---

## Stage 6 — Prototype Build & QA

**Prototype batch: 5 units**

**QA Checklist per unit:**
- [ ] Powers on via USB-C
- [ ] Warm-up animation plays (LED white/pulsing for 30s)
- [ ] LED transitions to green in clean air
- [ ] CO2 reading verifiable: breathe into sensor opening, LED transitions yellow → red within 30s
- [ ] Sensor returns to green within 2–3 minutes in open air
- [ ] Serial calibration command accepted
- [ ] No resets or watchdog reboots over 24h continuous run
- [ ] Enclosure snap-fit secure, no wobble

**Deliverables:**
- [ ] 5 assembled prototypes
- [ ] QA log for each unit
- [ ] Any firmware or hardware revisions noted for v1.1

---

## Stage 7 — Production Planning

### Small Batch (25–100 units)

- PCB + SMT assembly: JLCPCB or PCBWay
- Sensors (SCD40, SEN55): Mouser or Digi-Key (stock-check lead times — SEN55 can have 8–12 week lead times)
- Enclosure: FDM print in-house or via a print farm (Craftcloud, Treatstock)
- Final assembly: hand-assembly (attach sensors, snap enclosure, flash firmware, QA)
- Estimated labor per unit at small batch: ~15–20 minutes

### Mid-Scale (500–2,000 units)

- Move to full PCBA (all components assembled by fab)
- Injection-molded enclosure (requires tooling investment: $4,000–$8,000)
- Firmware flashing jig (pogo-pin fixture for batch flashing)
- Formal QA process with test fixture

### Lead Times to Plan For

| Item | Lead Time |
|---|---|
| PCB fabrication (JLCPCB) | 5–7 business days |
| SMT assembly | +5–7 business days |
| Sensirion SCD40 (Mouser) | 2–4 weeks |
| Sensirion SEN55 (Mouser) | 8–12 weeks (check stock) |
| 3D printed enclosures | 3–5 days in-house |
| Injection mold tooling | 6–10 weeks |

---

## Stage 8 — Cost & Pricing Analysis

### Cost Summary (Per Unit)

| Volume | BOM Cost | Assembly Labor | Enclosure | Total COGS |
|---|---|---|---|---|
| 1 (prototype) | $62.80 | — | $2.00 | ~$65 |
| 25 (small batch) | $55.00 | $8.00 (20 min @ $24/hr) | $2.50 | ~$65.50 |
| 100 units | $44.25 | $6.00 (15 min @ $24/hr) | $2.00 | ~$52.25 |
| 500 units | ~$32.00 | $3.00 (PCBA + jig) | $5.00 (injection) | ~$40.00 |
| 1,000 units | ~$26.00 | $2.00 | $3.50 | ~$31.50 |

> Injection mold tooling cost (~$6,000) is a one-time capital expense, amortized above ~500 units.

### Suggested Retail Pricing

| Channel | Price | Margin at 100 units |
|---|---|---|
| Direct (own website) | $89 | ~41% |
| Amazon FBA | $89 (after ~$15 fees) | ~28% |
| Wholesale to retail | $50 (50% of $89 MSRP) | ~5% at 100 units; improves at scale |

**Target MSRP: $79–$99.** At $89 direct and 100-unit COGS of ~$52, gross margin is ~42%. This is acceptable for a hardware product; typical consumer hardware targets 40–60% gross margin.

### Break-Even Analysis

| Units Sold | Revenue (@ $89) | Total COGS | Gross Profit |
|---|---|---|---|
| 25 | $2,225 | $1,638 | $587 |
| 100 | $8,900 | $5,225 | $3,675 |
| 500 | $44,500 | $20,000 | $24,500 |

Non-COGS costs to recover (one-time): PCB design, firmware dev, enclosure design, mold tooling. Estimate $8,000–$15,000 in development and tooling before first sale. Break-even on total investment at approximately 200–250 units sold direct.

---

## Staged Execution Roadmap

| Stage | Focus | Est. Duration |
|---|---|---|
| 1 | Finalize requirements and thresholds | 1 week |
| 2 | Source prototype components, breadboard test | 2–3 weeks |
| 3 | Firmware v0.1 (CO2 + LED only) | 1–2 weeks |
| 3 | Firmware v0.2 (full sensors) | 1 week |
| 4 | PCB schematic + layout (KiCad) | 2–3 weeks |
| 4 | PCB prototype order and receipt | 2 weeks |
| 5 | Enclosure design + test prints | 1–2 weeks |
| 6 | 5-unit prototype build and QA | 1 week |
| 3 | Firmware v0.3–v1.0 (polish + production) | 1–2 weeks |
| 7 | 25-unit small batch production | 4–6 weeks (sensor lead time) |
| 8 | Pricing finalized, sales channel setup | Parallel with Stage 7 |

**Total to first sellable unit: ~14–20 weeks**

---

## Stage 2 — Prototype Shopping List

Order these to begin breadboard development. All available on Adafruit, Pimoroni, or Amazon.

| Component | Specific Part | Qty | Est. Cost |
|---|---|---|---|
| Microcontroller | Adafruit QT Py ESP32-C3 (or any ESP32-C3 dev board) | 2 | ~$20 |
| CO2 sensor | Adafruit SCD-40 breakout (STEMMA QT) | 1 | ~$15 |
| PM sensor | Adafruit PMS5003 Air Quality Sensor + breakout | 1 | ~$40 |
| VOC sensor | Adafruit SGP30 Air Quality Sensor breakout | 1 | ~$18 |
| LED | Adafruit NeoPixel Jewel — 7 x WS2812B (or single NeoPixel) | 1 | ~$6 |
| Wiring | STEMMA QT / Qwiic cables (50mm + 100mm) | 4 | ~$4 |
| Breadboard | Half-size breadboard | 1 | ~$5 |
| Jumper wires | M-M and M-F assortment | 1 pack | ~$5 |
| USB-C cable | Short (30cm) USB-C to USB-A | 1 | ~$3 |
| USB power adapter | 5V 1A USB-A wall adapter | 1 | ~$6 |
| **Total** | | | **~$122** |

> Buy 2 ESP32-C3 boards — useful to have a spare and for firmware flashing/testing in parallel.
> PMS5003 from Adafruit includes the breakout adapter; raw sensor from AliExpress is ~$12 but needs a JST-PH cable.

---

## Locked Decisions — Round 2 (2026-04-29)

| Decision | Choice |
|---|---|
| Smart home ecosystem integration | Matter protocol (Google Home, Apple Home, Alexa, SmartThings natively) |
| Cloud backend | Supabase free tier (PostgreSQL + auto REST API + Auth) |
| Consumer dashboard | Web page only, Phase 1 (current readings + 24h graph) |
| Cloud cost model | Free — absorb server cost in device margin |
| FCC path | SDoC using pre-certified ESP32-C3 module (see Stage 9) |

---

## Stage 9 — Ecosystem Integration (Matter Protocol)

### Why Matter

Matter is the universal smart home standard backed by Apple, Google, Amazon, and Samsung. A single Matter implementation makes the device appear natively in Google Home, Apple Home, Amazon Alexa, and Samsung SmartThings — no cloud account linking, no proprietary bridge, no per-platform API work. Users commission by scanning a QR code in whichever app they already use.

Matter 1.2 added device types that are a direct fit for AirHealth:
- **Air Quality Sensor** device type
- **Carbon Dioxide Concentration Measurement** cluster
- **PM2.5 Concentration Measurement** cluster
- **VOC Index Measurement** cluster
- **Temperature Measurement** and **Relative Humidity Measurement** clusters

### ESP32-C3 + esp-matter SDK

Espressif maintains `esp-matter`, an official Matter SDK for all ESP32 variants including the C3. The ESP32-C3 has enough RAM and flash to run Matter alongside the full sensor loop.

Matter runs **locally** — the device talks directly to the home hub (Google Nest Hub, Apple HomePod, Amazon Echo) over WiFi. No cloud required for home automation. The Supabase cloud POST runs independently and simultaneously.

```
ESP32-C3
├── Matter stack (local, via esp-matter SDK)
│   ├── Air Quality cluster → Google Home, Apple Home, Alexa, SmartThings
│   └── Commissioning QR code (on device label / packaging insert)
│
└── Cloud POST (every 60s, independent of Matter)
    └── Supabase REST API → web dashboard history
```

### Matter Firmware Staging

Matter is substantial firmware work — scope it after v1.0 is stable:

- **v1.0**: LED + all sensors + WiFi cloud POST. No Matter.
- **v1.5**: Integrate esp-matter SDK. Implement Air Quality device type. Test commissioning with Google Home and Apple Home.
- **v2.0**: Pursue CSA Matter certification (~$5–10K) to use the Matter logo and appear in official compatible device listings.

---

## Stage 10 — Cloud Backend & Web Dashboard

### Backend: Supabase Free Tier

Supabase is a hosted PostgreSQL database with an auto-generated REST API, built-in auth, and a generous free tier (500MB DB, 2GB bandwidth/month, 50K monthly active users — no credit card required).

**Database schema:**

```sql
CREATE TABLE readings (
  id          bigserial PRIMARY KEY,
  device_id   text NOT NULL,
  recorded_at timestamptz NOT NULL DEFAULT now(),
  co2_ppm     numeric,
  pm25_ug     numeric,
  pm10_ug     numeric,
  voc_ppb     numeric,
  temp_c      numeric,
  humidity_rh numeric,
  status      text   -- 'green' | 'yellow' | 'red'
);

CREATE INDEX ON readings (device_id, recorded_at DESC);
```

Device POSTs JSON to Supabase REST endpoint every 60s. Supabase anon key in firmware header (sufficient for prototype; add per-device token for production).

### Web Dashboard: Next.js on Vercel Free Tier

Single-page dashboard per device showing:
- Large colored circle matching current LED status
- Current readings: CO2 ppm, PM2.5 µg/m³, VOC index, temp, humidity
- 24-hour sparkline chart per measurement
- Last-updated timestamp

**Stack:** Next.js (App Router) + Tailwind CSS + Recharts + Supabase JS client. Deployed to Vercel free tier.

**URL:** `airhealth.app/d/[device_id]` — one page per device, shareable link. No login for Phase 1.

**Source:** `air_health/web/`

### Deliverables

- [ ] Supabase project created, schema migrated
- [ ] Firmware v0.3 updated to POST to Supabase endpoint
- [ ] Next.js dashboard scaffold under `air_health/web/`
- [ ] Current status display + 24h charts
- [ ] Deployed to Vercel

---

## Stage 11 — FCC Compliance

### The Short Version

Using a pre-certified ESP32-C3 module makes FCC compliance achievable for **$2,000–$4,500** (versus $8,000–$15,000 for a device with custom RF circuitry). You self-declare via an **SDoC (Supplier's Declaration of Conformity)** backed by third-party lab testing.

### How It Works

The ESP32-C3 module (e.g. Espressif ESP32-C3-MINI-1) holds its own FCC ID and has **full modular approval**. When you integrate it into a host product:

- The radio/WiFi is already approved — you don't re-certify it.
- You test the **host product** (your PCB + enclosure) for **Part 15B unintentional emissions** — the digital noise from your processor, I2C bus, power supply, etc.
- After passing, you write an SDoC document and self-declare. No FCC filing required — keep records and produce if audited.

### Step-by-Step Path

| Step | Action | Cost | Time |
|---|---|---|---|
| 1 | Confirm ESP32-C3 module FCC ID on Espressif's certificate page | $0 | 1 hr |
| 2 | Don't modify the antenna or RF layout — follow Espressif's reference design | $0 | Design stage |
| 3 | Send 3 finished units to an accredited Part 15B lab | $1,500–$3,500 | 4–8 weeks |
| 4 | Lab runs radiated + conducted emissions tests, issues test report | Included | — |
| 5 | Write SDoC document (FCC KDB template) | $0–$500 | 1 week |
| 6 | Label product: include module FCC ID + compliance statement | $0 | Label design |
| 7 | Keep test report + SDoC on file | $0 | Ongoing |

**Total estimated cost: $2,000–$4,500**

### Required Label Text

```
Contains FCC ID: [ESP32-C3 module FCC ID]
This device complies with Part 15 of the FCC Rules.
Operation is subject to the following two conditions:
(1) This device may not cause harmful interference, and
(2) this device must accept any interference received,
    including interference that may cause undesired operation.
```

### Critical Rules

- **Do not** use a custom antenna or relocate the antenna from the module's reference design — this voids modular approval and forces full certification.
- **Do not** list for retail sale in the US before SDoC is complete and device is labeled.
- **CE (Europe)** is a separate process. Similar logic (use module's CE marking, run RED compliance testing), similar cost. Budget separately when targeting Europe.

### Timing

Begin FCC process **after Stage 6 prototype QA**, before Stage 7 small batch production. Submit 3 units from the first PCB run to the lab.

---

## Open Questions / Remaining

1. **Matter certification timing** — CSA certification (~$5–10K) needed before officially using the Matter logo and appearing in Google/Apple compatible device listings. Plan after v1.5 firmware is validated.
2. **CE certification** — Required for European sales. Budget $2,000–$5,000 (with pre-certified module). Scope when Europe becomes a target market.
3. **Dashboard auth** — Phase 1 dashboard is public by device ID. Phase 2: add login so only the device owner can view their data.
