# OFN P0 Acceptance Criteria v0.2

P0 ends not when "everything works" but when specific numbers are recorded. These criteria are written **before** assembly so results can't be retro-fitted. Each item: measurement → threshold → instrument.

## 1. Power chain (Gate 1 confirmation)

| # | Check | Threshold | Instrument |
|---|---|---|---|
| 1.1 | XL4015 output, no load | 5.05 ±0.05 V | multimeter |
| 1.2 | 5 V sag under CM5 + 2 bays + USB-A load | <0.15 V from 5.05 | INA219 #2 (0x41) |
| 1.3 | 12 V input sag at full load (PDC004) | <0.4 V | INA219 #1 (0x40) |
| 1.4 | Whole-node current idle / under stress | record (expect 0.5–2.5 A at 12 V) | INA219 #1 |
| 1.5 | USB-A shedding when bays exceed 2 A total | USB-A drops, node stays up | test resistor/load |
| 1.6 | XL4015 / SY6280 temperature after 30 min full load | <60 °C | pyrometer/thermocouple |

## 2. Compute and monitoring

| # | Check | Threshold |
|---|---|---|
| 2.1 | `vcgencmd get_throttled` after 30 min stress+radio | `0x0` |
| 2.2 | SoC temperature under load | <80 °C, no throttling |
| 2.3 | All I2C addresses visible: 0x20, 0x40, 0x41 | stable in `i2cdetect -y 1` |
| 2.4 | MCP23017 register write/read (toggle an output) | works, bus doesn't hang |

## 3. Radio

| # | Check | Threshold |
|---|---|---|
| 3.1 | LoRa ping RFM95W↔RFM95W at 868 MHz | 10/10 packets on the bench, RSSI recorded |
| 3.2 | Indoor/outdoor range | record RSSI vs distance (1/5/15 m) for Gate 4 |
| 3.3 | LoRa TX current | record peak (expect <350 mA at 25 mW) |

## 4. Control link

| # | Check | Threshold |
|---|---|---|
| 4.1 | Phone associates with AP, holds 30 min | 0 drops |
| 4.2 | API round-trip (ping 10.0.0.1) | <30 ms average |
| 4.3 | LoRa ping while AP is active | no more packet loss than without AP (+0–5%) |

## 5. Survival power

| # | Check | Threshold |
|---|---|---|
| 5.1 | Whole load from powerbank via PDC004 (12 V) | stable 30 min, throttled=0 |
| 5.2 | Consumption in survival mode (AP off, LoRa sleep) | record (target <2 W) |

## 6. Recording protocol

Everything goes into `p0-results.md` in the repo (`tests/` folder): numbers, conditions, deviations. Any failed criterion becomes a schematic v0.3 change item **before** KiCad. Skipping a criterion without a recorded reason is not allowed.

## 7. P0 exit

P0 is passed when: all criteria 1.x–4.x are green, criteria 5.x are at least measured and recorded. Only then: P1 purchases and KiCad entry are unlocked. No board orders before that.
