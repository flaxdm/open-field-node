# Power budget v0.2 (Gate 1 review document)

Verifies the power tree against real loads. Result: Gate 1 passes with two part-class changes recorded below.

## 1. Source budget

| Input | Nominal | Max continuous | Available power |
|-------|---------|----------------|-----------------|
| XT30 field 8–18 V | 12 V | 5 A (raised from 3 A, see findings) | 60 W |
| USB-C PD 12 V | 12 V | 3 A (PD profile) | 36 W |

Worst supported case: XT30 input at 60 W; USB-C input derates the platform (charging outputs limited).

## 2. Load budget (12 V domain)

| Consumer | Rail | Typical | Worst | Notes |
|----------|------|---------|-------|-------|
| 5 V backbone buck | VIN→5 V | 22 W | 30 W (6 A) | ~90% eff |
| PD charging out | VIN→9/12 V | 0 | 27 W | own buck |
| **Total 12 V** | | 22 W | **57 W** | fits 60 W XT30; NOT fits 36 W PD-input → platform derate mode on USB-C input |

## 3. Load budget (5 V rail, 6 A = 30 W cap)

| Consumer | Typical | Worst | Policy |
|----------|---------|-------|--------|
| CM5 core (5V_CORE, 3 A limit) | 7 W | 12–15 W | always on, highest priority after safety |
| 3.3 V rail (from 5 V, 2 A) | 2 W | 6.6 W | |
| Bay 1..6 (2 A each max) | 1.5 W (2 loaded bays) | up to 60 W theoretical | **capped: total bays ≤ 3 A (15 W)** |
| USB-A out (2.4 A) | 0 | 12 W | sheds first: limited to 1 A when bays > 2 A total |
| Misc (LED, MCP23017, INA219) | 1 W | 2 W | |

Policy: `bays_total + USB-A + core_typical ≤ 5.5 A`; hard cap 6 A enforced by buck foldback + bay shedding (power daemon, see Software architecture).

Typical field config: core 7 W + 2 loaded bays (LoRa ~2.5 W, GPS ~1 W) + logic 2 W ≈ 12.5 W — huge headroom. Worst legal config: core 15 W + bays 15 W + USB-A 5 W (shed) + logic 6.6 W ≈ 41 W — over 30 W rail cap → shed rule drops USB-A and bays above cap in priority order: USB-A → bay6..bay1 → (core never shed).

## 4. Findings → design changes (paper fixes)

1. **Input current 3 A → 5 A.** 57 W worst case over 36 W. Change: PTC 3 A → 5 A; OR-ing diodes SS34 (3 A) → SS54 (5 A) or ideal-diode MOSFET; XT30 fine (15 A plug).
2. **5 V rail policy needed** (above). No hardware change.
3. PD-input mode: when powered from USB-C PD input, charging outputs and bay count are limited (platform announces "low power mode" to the phone app).

## 5. Bay current reference (planning numbers)

| Module class | 5 V draw typ | 5 V draw max | Notes |
|--------------|--------------|--------------|-------|
| LoRa (RA-02-class + MCU) | 0.25 A | 0.5 A | TX bursts |
| GPS | 0.05 A | 0.15 A | |
| Status/service | 0.1 A | 0.3 A | LEDs, buzzer |
| DMR control (post-MVP) | 0.3 A | 0.8 A | |
| Future SDR host (H1, 12V/3.3V) | — | 2.5 A from 12V aux | separate path |

## 6. Gate 1 verdict

**Pass with changes:** 12 V input sized 5 A, OR-ing SS54-class, bay total cap 3 A, USB-A shed rule. Update BACKPLANE-SCHEMATIC and BOM accordingly.
