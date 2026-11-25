# Backplane power schematic v0.2 (design document)

Stage-by-stage design for the backplane power section. Every stage lists: function, reference parts (common/cheap class), key connections, and notes for layout. This is the drawing set for KiCad entry; it is not a PCB layout yet.

## Stage 0 — Inputs and OR-ing

| Ref | Function | Part (class) | Notes |
|-----|----------|--------------|-------|
| J1 | Field input XT30, 8–18 V | Amass XT30UPB or generic XT30 | 2 A continuous target, 15 A plug rating |
| J2 | USB-C input | USB-C 16-pin mid-mount SMD receptacle | CC1/CC2 to PD trigger |
| U1 | PD trigger, requests 12 V | CH224K or IP2721 | Rd on CC; output 12 V from a PD supply |
| D1/D2 | Input OR-ing | SS54 Schottky ×2 (or dual ideal-diode P-MOSFET) | Sized 5 A per Power budget v0.2; XT30 priority by topology; ideal-diode variant lowers loss |
| Q1 | Reverse polarity P-MOSFET, high-side | AO3401A-class, Vds ≥ 30 V | Body diode orients so correct polarity turns FET on |
| TVS1 | Surge clamp | SMBJ16A–SMBJ18A class | Final value set at layout after input range check |

Net: `VIN_12V` (after OR-ing, Q1, TVS1) feeds all downstream stages.

## Stage 1 — 5 V main buck (backbone)

| Ref | Function | Part | Notes |
|-----|----------|------|-------|
| U2 | Buck 12→5 V, 6 A | TPS54531 (sync, 3.5–28 V in) or XL4015 module for bench bring-up | Set 5.05 V to compensate distribution drop |
| L1 | 10 µH shielded inductor, ≥7 A Isat | common 10×10 mm drum | |
| Cin/Cout | 2×22 µF ceramic + 100 µF electrolytic | | |

Net: `5V_MAIN`.

## Stage 2 — Per-bay limiters (×6), Class L slots

| Ref | Function | Part | Notes |
|-----|----------|------|-------|
| U3..U8 | Current-limited high-side switch, 2 A | TPS2553D (adjustable ILIM) | ILIM resistor sets 2 A; EN pin per bay; /FAULT open-drain output |
| Alternative | Discrete: P-MOSFET + 25 mΩ sense + 2N7000 + ref | | Cheaper, more parts, no /FAULT |

Per bay: `5V_MAIN → U3 → BAYn_5V → slot pins 1–3` (three paralleled 5 V pins).
EN driven from MCP23017; /FAULT routed to MCP23017 input.

Current measurement policy (cost decision, recorded here):
- One INA219 on `VIN_12V` input, one on `5V_MAIN` → total input and backbone power.
- Per-bay current: via TPS2553 /FAULT (over-limit flag) + bay rail voltage readback at the bay connector through a 100 k/100 k divider into an ADC channel of the MCU companion / CM5 ADC (GPIO26) via analog mux if needed.
- Optional per-bay INA199 (+shunt) variant reserved in BOM for diagnostics builds.

## Stage 3 — 3.3 V logic rail

| Ref | Function | Part | Notes |
|-----|----------|------|-------|
| U9 | Buck 5→3.3 V, 2 A | SY8120 or MP2236 | Feeds EEPROMs, sensors, MCP23017, ID logic |

Net: `3V3_LOGIC`.

## Stage 4 — 12 V aux passthrough

Fused `VIN_12V → 12V_AUX` (PTC/polyfuse 3 A) for future RF PA / LTE amp. Present as unpopulated footprint in MVP.

## Stage 5 — Charging outputs (D9)

| Ref | Function | Part | Notes |
|-----|----------|------|-------|
| U10 | PD source controller | CH235S or IP2723T class | Advertises 9 V/2 A and 12 V/1.5 A PPS-less profiles |
| U11 | Buck for PD output, 5–12 V adjustable | XL4015-class controller, FB driven by U10 | Input `VIN_12V`; output isolated from `5V_MAIN` to avoid coupling fast-charge ripple into bays |
| J3 | USB-C receptacle (output) | same family as J2 | |
| U12 | USB-A switch, 2.4 A | SY6280 / MIC2005-class load switch | From `5V_MAIN`; /FAULT to MCP23017 |
| J4 | USB-A receptacle | | D+/D– shorted (≤200 Ω) = dedicated-charger signature |

## Stage 6 — Monitoring and control

| Ref | Function | Part | I2C addr | Notes |
|-----|----------|------|----------|-------|
| U13 | Rail monitoring | INA219 | 0x40 | on `VIN_12V` |
| U14 | Rail monitoring | INA219 | 0x41 | on `5V_MAIN` |
| U15 | GPIO expander: 6×EN out, 6×/FAULT in, status LEDs | MCP23017 | 0x20 | Interrupt line to CM5 GPIO |
| U16.. | Module ID EEPROMs (one per own module) | AT24C32-class | 0x50–0x57 | Pi HAT format per D10; ready-made modules lack EEPROM → manual config |

## Protection summary

- Reverse polarity: Q1 (whole board).
- Input surge: TVS1.
- Per bay: TPS2553 limit 2 A + /FAULT.
- USB-A out: U12 current-limited switch.
- PD out: controller current limit + input fuse.
- Board input: 5 A PTC polyfuse before OR-ing (both inputs each own PTC) — raised from 3 A per Power budget v0.2.

## Layout notes

- OR-ing diodes and Q1 near input connectors; TVS at board edge.
- 5 V backbone: star from U2 to limiters, wide pours.
- Keep PD-output buck away from LoRa/GPS bay routing (switching noise).
- INA219 shunts in-line on power path, Kelvin sense.

## Power policy (from Power budget v0.2)

- Total bays capped at 3 A (15 W); USB-A output sheds to 1 A when bays > 2 A; shedding order USB-A → bay6..bay1; core never shed.
- On USB-C PD input: platform runs in low-power mode (PD out off, bays limited, app notified).

## Open points for schematic entry

- Final TVS voltage (after input range freeze).
- TPS2553 vs discrete limiter (BOM cost compare — see BOM v0.2).
- PD source: fixed 9 V profile only vs 9/12 V dual profile (simpler = fixed 9 V/2.2 A).
- Ideal-diode OR-ing IC vs Schottky (efficiency vs cost).
