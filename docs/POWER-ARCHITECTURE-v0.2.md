# Power architecture v0.2

Goal: a cheap, common-parts power tree for the backplane. No exotic ICs. Battery is postponed to a separate gate.

## Principles

1. 12 V field input is the backbone, fed from two OR-ed sources: field XT30 (8–18 V, priority) and USB-C with a PD trigger set to 12 V.
2. Backplane generates 5 V (≈6 A) and 3.3 V (≈2 A).
3. Every low-speed module bay gets its own switched, current-limited 5 V rail with measurement.
4. 3.3 V is a shared low-current rail for logic, module EEPROMs and sensors.
5. No module may draw power from signal pins. Power contacts are always grouped separately.
6. High-power RF amplifiers are out of MVP scope.

## Power tree

```
XT30 field input (8–18 V) ─┐
                            ├── OR-ing (Schottky / ideal diode) ── Reverse polarity P-MOSFET ── TVS ──┐
USB-C ── PD trigger 12 V ───┘                                                                        │
                                                                                                       │
   ├─── Buck 5 V / 6 A ─── 5 V backbone ──┼── Bay 1..6: eFuse/lim 2 A + INA measure
   │                                      │             └─── Buck/LDO 3.3 V
   │                                      └─── 3.3 V / 2 A shared logic rail
   │
   └─── 12 V passthrough (fused, reserved for future RF PA)
```

## Rails

| Rail | Source | Limit | Consumers |
|------|--------|-------|-----------|
| 12 V in | XT30 (priority) or USB-C PD trigger (12 V), OR-ed | 8–18 V, 3 A continuous | buck, passthrough |
| 5 V main | buck from 12 V | 6 A | module bays, 3.3 V buck, H-bays |
| 5 V bay-N | eFuse/limiter from 5 V | 2 A per bay | LoRa, GPS, status, DMR |
| 3.3 V | buck from 5 V | 2 A | EEPROM, sensors, MCU logic |
| 12 V aux | fused passthrough | reserved | future RF PA / LTE |

## Reference parts (cheap and common)

- 5 V buck: XL4015 module or TPS54531 (5 A class).
- 3.3 V buck: SY8120 or MP2236 (2 A class). AMS1117 only for tiny auxiliary loads.
- Per-bay current limiter: TPS2553 or discrete P-MOSFET + sense resistor.
- Rail monitoring: INA219 (I2C) on 12 V input and on 5 V backbone.
- Enable / power-good signals: MCP23017 I2C GPIO expander.
- Input protection: P-MOSFET reverse polarity + TVS (SMBJ series).
- PD trigger for USB-C input: CH224K or IP2721 (set to 12 V).
- Input OR-ing: Schottky diodes (cheap) or ideal-diode P-MOSFET (lower loss).
- Battery path: postponed. Candidate: LiFePO4 with charge controller; separate gate.

## Estimated power-section cost

Roughly 7–11 USD for input protection, both bucks, six bay limiters and monitoring, plus PD trigger and input OR-ing, excluding connectors.

## Open questions

- Input OR-ing method: Schottky diodes vs ideal-diode MOSFET (loss vs cost).
- PD trigger chip: CH224K / IP2721 / similar.
- Per-bay limiter: IC eFuse vs discrete (cost vs simplicity).
- Battery chemistry and charge controller (separate gate).
- Peak current of H-bay SDR / LTE modules.
