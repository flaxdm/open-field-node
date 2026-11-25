# Open Field Node — Full project documentation (v0.2, 2026-11-25)

Consolidated project documentation in plain language. Authoritative details live in the specialized docs linked per section. Russian version: PROJECT-DOCUMENTATION-ru.md.

## 1. What it is

Open Field Node — an open modular headless field platform: no screen, no keyboard, controlled from an Android smartphone. Replaceable compute core (Raspberry Pi CM5), module slots for LoRa/Meshtastic, GPS, SDR radio monitoring, networking; DMR/LTE/Wi-Fi post-MVP. Runs autonomously, stores logs, syncs with the phone later. Field-extras: phone charging, power-from-anything (battery, car, solar), per-module power accounting.

Design intent: a cheap, open platform built from common parts; the community can build compatible modules.

## 2. Principles

1. **No unnecessary purchases** — every buy passes a gate (§10).
2. **Common standards** — IDC 2.54 mm, Pi HAT EEPROM format, mikroBUS-compatible signal set, USB-C PD. Nothing exotic.
3. **Security** — own networks/devices or isolated lab only; never third-party targets.
4. **Plan first** — 2–3 options discussed before any execution, on user's command only.
5. **Openness** — docs, schematics, code; module format specified so others can build compatible modules.

## 3. Architecture

Device = compute core + carrier + power backplane + modules.

- **Core (CM5 8GB Lite, no wireless)** — self-contained Linux computer; works standalone in a dock (D1). In the field it lives inside the node.
- **Carrier** — hosts the core: GbE, H-slots, bus fan-out to bays. Design: `CARRIER-SCHEMATIC-v0.2`.
- **Backplane** — power: inputs, converters, six 2 A bay channels, monitoring, charging outputs. Design: `BACKPLANE-SCHEMATIC-v0.2`.
- **Modules** — 22×60 mm cards (`MECHANICAL-v0.2`). Slot classes: L (low-speed, IDC 2×8) and H (one PCIe x1, one USB3). Decision D7.

See `SPEC-v0.2`, `ARCHITECTURE.md`, `MODULE-STANDARD.md`.

## 4. Power

### Inputs (D8)

OR-ed, field input has priority:

1. **XT30** — 8–18 V field input (battery/car/solar), up to 5 A.
2. **USB-C** — PD trigger (CH224K/IP2721) requests 12 V from any PD charger.

Protection: SS54 OR-ing diodes, reverse-polarity P-MOSFET, TVS, PTC fuses.

### Power tree

```
12 V (XT30 priority / USB-C PD) → protection → buck 5 V/6 A (backbone)
   ├── 6 bays × TPS2553 (2 A each, EN + fault flag)
   ├── 3.3 V/2 A (logic, EEPROM)
   └── USB-A 5 V/2.4 A (charging)
12 V branch → PD source 9 V/2 A or 12 V/1.5 A (fast charge USB-C)
12 V passthrough → reserved for future RF PA
```

### Power policy (Gate 1, passed)

- Bays capped at 3 A total; shedding order USB-A → bays 6..1 → core never.
- On USB-C input: low-power mode (charging limited, app notified).
- CM5 on a dedicated 3 A channel.

Docs: `POWER-ARCHITECTURE-v0.2`, `POWER-BUDGET-v0.2`, `READY-MODULES-v0.2`.

## 5. Modules and slots

- **Class L slot (IDC 2×8, 16-pin)**: 5 V on 3 pins (2 A), GND, I2C, UART, SPI, EN/PG, 3.3 V. mikroBUS-compatible signal set; full socket compatibility open until layout.
- **Class H**: H1 = PCIe x1 edge (SDR/NVMe/2nd GbE, D3 reserve); H2 = USB3 19-pin (Wi-Fi dongle for control, USB-SDR, storage). Rule: max two high-bandwidth USB devices; control modules never on USB.
- **Identification (D10)**: ready-made modules → manual config + I2C hints; own modules → EEPROM in Raspberry Pi HAT format with OFN fields (class, current limits, RF regions, name, UUID). Node applies limits before enabling; TX region enforced in firmware.

Docs: `MODULE-CONNECTOR-v0.2`, `MODULE-DESCRIPTOR-v0.2`, `MODULE-STANDARD-v0.2`.

## 6. Software

- **Control link (D12)**: Wi-Fi AP on the node (USB-WiFi MT7921AU/RTL8811AU on H2), hostapd; USB-C fallback. Zero custom firmware; same dongle works on the dev dock.
- **Services (systemd)**: ofn-powerd (INA219, shedding policy), ofn-slotd (bays, EEPROM), ofn-modmgr (registry), ofn-mission (tasks, LoRa sessions), ofn-logd, ofn-syncd (store-and-forward), ofn-ctl-api (WebSocket+JSON, TLS, pairing code).
- **Android app**: Kotlin, offline-first; dashboard, modules, missions, logs, settings. The app is the display the device lacks.
- **Survival mode**: autonomous mission execution and logging while the phone is away; battery profile post-MVP.

Doc: `SOFTWARE-ARCHITECTURE-v0.2`.

## 7. RF

Region **868 MHz** (D13, LoRaWAN RU864); 433 MHz optional fallback.

- LoRa: SX1276-class (RFM95W 868), ≤25 mW, panel antenna.
- Control Wi-Fi: 2.4 GHz, minimal power, fixed channel, no internet routing.
- SDR: RX only in MVP; DMR post-MVP with legal check.
- Antenna isolation: Wi-Fi↔LoRa ≥5 cm; GPS bay farthest from Wi-Fi; SAW filter footprint on GPS module.
- TX-capable modules carry region bitmap; firmware blocks mismatched TX.

Doc: `RF-PLAN-v0.2`; Gate 4 checklist includes re-verifying RU/EU regulations before any TX hardware.

## 8. Mechanics

- Board 160×100 mm, 4 layers; six bays at 25 mm pitch; module cards 22×60 mm.
- Front panel: XT30 → USB-C in → USB-C charge → USB-A → 6 antenna ports → RJ45.
- Case target 170×110×35 mm: aluminum extrusion (shielding) or printed PETG for prototype.
- CM5 cooling: passive fins; optional 30 mm fan.

Doc: `MECHANICAL-v0.2`.

## 9. P0 prototype path (current stage)

Everything validated on standard boards first — no custom PCB:

- CM5 in Waveshare BASE-A dock (own PSU).
- Power tree on perfboard from ready modules: PDC004, SS54, PTC, MOSFET, TVS, XL4015, TPS2553 ×2, SY6280, 65 W PD module.
- Monitoring: INA219 ×2 + MCP23017 on I2C.
- Wi-Fi dongle → hostapd → phone control.
- LoRa RFM95W 868 on GPIO UART.

Add-on ~3–3.5k ₽. After P0: KiCad entry, then field board order.

Docs: `P0-PROTOTYPE-v0.2`, `DEV-STATION-BUYLIST` (first purchase ≈28–30k ₽ total).

## 10. Purchase gates

| Gate | Content | Status |
|------|---------|--------|
| dev-kit | Dev station + P0 | **allowed now — the only open purchase** |
| Gate 1 | Power budget | passed with fixes (5 A input, SS54, limits) |
| Gate 2 | Module interface freeze | effectively complete (D1–D13) |
| Gate 3 | Carrier validated before core buy | awaits P0 + KiCad |
| Gate 4 | RF plan before radio purchase | draft ready, legal check pending |

No-purchase BOM: `BOM-v0.2` (board ≈30–38 USD excl. core/LoRa).

## 11. Decisions D1–D13

| ID | Decision |
|----|----------|
| D1 | CM5-class core, 8GB Lite, no wireless |
| D2 | Backplane: 12 V, controlled rails, USB3, GbE, I2C/UART/GPIO, ID bus |
| D3 | PCIe lane routing reserved |
| D4 | Card module with keying and front-panel connectors |
| D5 | MVP scope: carrier, power, status, one network, one LoRa module |
| D6 | DMR, LTE, second Wi-Fi, dual Ethernet, SDR host = post-MVP |
| D7 | Class L = IDC 2×8; Class H = PCIe x1 + USB3 |
| D8 | Dual input: XT30 (priority) + USB-C PD 12 V, OR-ed |
| D9 | Charging outputs: USB-C PD fast charge + USB-A 5 V/2.4 A |
| D10 | Module ID: manual+I2C hints for ready modules; Pi HAT EEPROM for own |
| D11 | Dev station = CM5 8GB Lite on Waveshare dock; same CM5 becomes field core |
| D12 | Control link: Wi-Fi AP (USB-WiFi dongle); USB-C fallback |
| D13 | RF region: 868 MHz (RU864) |

## 12. Roadmap

1. Buy dev station + P0 parts (only allowed spend).
2. P0 bench build: staged power bring-up, hostapd, service skeletons, LoRa ping.
3. Software: Android app + control API on the live bench.
4. KiCad: carrier + power schematics → review → board order (Gate 3).
5. Field board: CM5 moves over; first enclosure build.
6. Post-MVP: battery gate, DMR/LTE/SDR modules, ESP32 Wi-Fi card as community module.

## Document registry

SPEC-v0.2 · POWER-ARCHITECTURE-v0.2 · POWER-BUDGET-v0.2 · BACKPLANE-SCHEMATIC-v0.2 · CARRIER-SCHEMATIC-v0.2 · MODULE-CONNECTOR-v0.2 · MODULE-DESCRIPTOR-v0.2 · MODULE-STANDARD-v0.2 · SOFTWARE-ARCHITECTURE-v0.2 · NETWORK-MODULE-v0.2 · WIFI-BRINGUP-v0.2 · RF-PLAN-v0.2 · MECHANICAL-v0.2 · BOM-v0.2 · READY-MODULES-v0.2 · P0-PROTOTYPE-v0.2 · P0-ASSEMBLY-CHECKLIST · DEV-STATION-BUYLIST · plus v0.1 docs (ROADMAP, ARCHITECTURE, MODULE-STANDARD, CAPABILITY-API).

Repo: github.com/flaxdm/open-field-node (EN in `docs/`, RU in `docs/ru/`).
