# MVP carrier schematic v0.2 (design document)

The carrier hosts the compute core and fans out to the module bays. Companion to [Backplane power schematic v0.2](BACKPLANE-SCHEMATIC-v0.2.md).

## Top-level blocks

```
                    ┌────────────────────────── MVP CARRIER ──────────────────────────┐
                    │                                                                  │
  CM5 SODIMM 200P ──┤ U1 socket (standard CM4/CM5 SODIMM, 5.2 mm height)             │
                    │                                                                  │
                    ├─ 5V_CORE (dedicated TPS2553 channel, 3 A) ← from 5V_MAIN        │
                    ├─ GbE MDI ──► J1 RJ45 magjack (HanRun HR911105A-class)            │
                    ├─ USB3 SS+HS ──► H2 USB3 slot (USB3 header, 19-pin or Type-A)    │
                    ├─ USB2 ──► J2 4-pin header (debug / LTE modem, internal)          │
                    ├─ PCIe x1 ──► H1 edge slot (PCIe x1 SMD socket, e.g. 36-pin)       │
                    ├─ I2C0 ──► BAY I2C bus (EEPROM 0x50..0x57, INA219, MCP23017)       │
                    ├─ SPI0 ──► BAY SPI bus (shared, per-bay CS from MCP23017)        │
                    ├─ UART0 ──► BAY UART (per-bay mux or fixed bay assignment)         │
                    ├─ GPIO ──► MCP23017 INT, button inputs, status LED                 │
                    │                                                                  │
                    ├─ BAY1..BAY6 Class L IDC 2×8 (see pinout below)                    │
                    └─ Power-good + EN daisy-chain from backplane power stage          │
```

## Compute core

- U1: CM4/CM5-compatible SODIMM 200-pin socket (2×100, 0.6 mm pitch, 5.2 mm stack height). Same socket family across CM4/CM5 ecosystem — cheap and stocked everywhere.
- 5V_CORE: own TPS2553 limiter at 3 A from `5V_MAIN`, EN tied to board power-good. Core never shares a limiter with bays.
- Reset/power sequencing per CM5 dev kit reference: nPMIC_RESET, RUN_PG, GLOBAL_EN, RTC battery pads (unpopulated in MVP).

## Network

- GbE: CM5 on-module PHY MDI signals → J1 RJ45 with integrated magnetics (HanRun HR911105A-class; bob-smith termination 75 Ω + 1 nF to GND per pair).
- Second GbE possible later via H1 PCIe (D3 reserve).

## High-speed slots

- H1: PCIe x1 edge socket, standard SMD x1 connector, PERST# and REFCLK from CM5 PCIe pins; 12 V and 3.3 V available per slot policy (3.3 V from `3V3_LOGIC` limited 1 A; 12 V from `12V_AUX` fused 2 A).
- H2: USB3 slot: 19-pin header (Intel-style internal header) preferred over chassis Type-A — no case needed; SS pairs 90 Ω differential, AC caps on TX per spec; 5 V from `5V_MAIN` via 2 A limiter.

## Low-speed bays (Class L, IDC 2×8)

Per-bay pinout (IDC numbering, keying notch orientation per layout):

| Pin | Signal | Pin | Signal |
|-----|--------|-----|--------|
| 1 | BAYn_5V | 2 | BAYn_5V |
| 3 | BAYn_5V | 4 | GND |
| 5 | GND | 6 | GND |
| 7 | I2C_SDA | 8 | I2C_SCL |
| 9 | UART_TX | 10 | UART_RX |
| 11 | SPI_SCK | 12 | SPI_MISO |
| 13 | SPI_MOSI | 14 | SPI_CS (per-bay, from MCP23017) |
| 15 | EN/PG (enable out + power-good in, dual-function pin, protection series R) | 16 | 3V3_LOGIC |

Notes:
- 5 V on three pins = 2 A at ~0.7 A/pin, inside 1 A/pin rating of common headers.
- mikroBUS signal-set compatible (SPI/UART/I2C/EN/PG/INT share roles with AN/RST/PWM positions — full mikroBUS socket compatibility left open per SPEC; bay is IDC).
- I2C per-bay EEPROM: own modules use 0x50+n per bay; bus has 2.2 kΩ pullups on carrier.

## Control and service

- SW1 power button (GLOBAL_EN), SW2 user button → GPIO, D1 RGB or single status LED → GPIO.
- J3: 3-pin UART console header (3.3 V) for bring-up; optional USB-UART footprint.
- Buzzer footprint unpopulated (status module owns sound in MVP).

## Connectors summary

| Ref | Connector | Class |
|-----|-----------|-------|
| U1 | SODIMM 200P socket | compute core |
| J1 | RJ45 magjack HR911105A-class | network |
| J2 | USB2 4-pin header + J3 UART 3-pin | service |
| H1 | PCIe x1 edge socket | high-speed |
| H2 | USB3 19-pin header | high-speed |
| BAY1..6 | IDC 2×8 shrouded box header, 2.54 mm, key notch | Class L modules |
| J1/J2 backplane power | XT30 + USB-C | power (see power schematic) |

## Layout notes

- SODIMM near board center; bays along one edge for module fronts.
- Keep H1/H2 differential pairs short; no vias on PCIe REFCLK if possible.
- Bay IDC headers: polarizing notch faces front panel.
- Separate GND stitching under magnetics (RJ45) per datasheet.

## Open points

- UART per-bay policy: fixed assignment (Bay2 = GPS) vs analog mux — decide in firmware-first review.
- H2 19-pin vs Type-A: 19-pin chosen; Type-A variant = BOM alternative line.
- RTC coin cell: unpopulated pads in MVP.
