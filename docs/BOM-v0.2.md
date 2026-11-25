# BOM v0.2 (no-purchase bill of materials)

Review list only — nothing here is approved for purchase. Prices are qty-1 street estimates (AliExpress/LCSC class) to size the build. See purchase gates in [SPEC v0.2](SPEC-v0.2.md).

## Power section (backplane)

| # | Item | Part / class | Qty | Unit est. | Sum |
|---|------|--------------|-----|-----------|-----|
| 1 | XT30 plug board-side | XT30UPB-F or generic | 1 | 0.40 | 0.40 |
| 2 | USB-C receptacle 16P | mid-mount SMD | 2 (in + out) | 0.30 | 0.60 |
| 3 | PD trigger (input) | CH224K / IP2721 | 1 | 0.60 | 0.60 |
| 4 | OR-ing Schottky | SS34 | 2 | 0.05 | 0.10 |
| 5 | Reverse-polarity P-MOSFET | AO3401A-class | 1 | 0.10 | 0.10 |
| 6 | TVS | SMBJ16A-class | 1 | 0.15 | 0.15 |
| 7 | Input PTC polyfuse 3 A | common | 2 | 0.10 | 0.20 |
| 8 | Buck 5 V/6 A | TPS54531 | 1 | 1.20 | 1.20 |
| 9 | Inductor 10 µH 7 A + caps | | 1 | 1.00 | 1.00 |
| 10 | Per-bay limiter | TPS2553D | 7 (6 bays + core) | 0.80 | 5.60 |
| 11 | Buck 3.3 V/2 A | SY8120 | 1 | 0.40 | 0.40 |
| 12 | Rail monitor | INA219 module/bare | 2 | 1.50 | 3.00 |
| 13 | GPIO expander | MCP23017 | 1 | 1.20 | 1.20 |
| 14 | PD source controller | CH235S / IP2723T | 1 | 1.50 | 1.50 |
| 15 | Buck for PD out 9 V/2 A+ | XL4015-class + inductor | 1 | 2.00 | 2.00 |
| 16 | USB-A load switch 2.4 A | SY6280-class | 1 | 0.30 | 0.30 |
| 17 | USB-A receptacle | | 1 | 0.20 | 0.20 |

Subtotal power ≈ 18.5 USD (with margin: 18–22)

## Carrier section

| # | Item | Part / class | Qty | Unit est. | Sum |
|---|------|--------------|-----|-----------|-----|
| 18 | SODIMM 200P socket | CM4/CM5 standard | 1 | 2.50 | 2.50 |
| 19 | RJ45 magjack | HanRun HR911105A-class | 1 | 1.20 | 1.20 |
| 20 | PCIe x1 edge socket | standard SMD | 1 | 1.50 | 1.50 |
| 21 | USB3 19-pin header | | 1 | 0.30 | 0.30 |
| 22 | IDC 2×8 shrouded box header (bays) | generic 2.54 mm | 6 | 0.15 | 0.90 |
| 23 | IDC 2×8 cable plugs + ribbon (modules) | FC-16-class | 6 | 0.40 | 2.40 |
| 24 | USB2/UART pin headers | | 2 | 0.10 | 0.20 |
| 25 | Buttons, LED, passives, PCB misc | | lot | — | 5.00 |

Subtotal carrier ≈ 14 USD

## Own-module electronics (first LoRa/status module prototype)

| # | Item | Part / class | Qty | Unit est. | Sum |
|---|------|--------------|-----|-----------|-----|
| 26 | ID EEPROM | AT24C32 | 1 | 0.30 | 0.30 |
| 27 | LDO 3.3 V (module side) | ME6211/HT7833-class | 1 | 0.10 | 0.10 |

## Totals

| Block | Estimate |
|-------|----------|
| Power | 18–22 |
| Carrier | 12–16 |
| Modules (first two) | ~1 + LoRa radio module (see Gate 4 RF plan) |
| **Board total (no core, no LoRa radio)** | **30–38 USD** |

Excludes: CM5 core (≈45–60 for 8 GB Lite), LoRa radio module (≈5–15 for RA-02/RFM95W-class), enclosure, antennas.

## Dev station (dev-kit gate, separate from field MVP)

| # | Item | Part / class | Qty | Unit est. | Sum |
|---|------|--------------|-----|-----------|-----|
| D1 | Dev board | Raspberry Pi 5 8 GB | 1 | 80 | 80 |
| D2 | PSU 5 V/5 A USB-C (official or PD 5A class) | | 1 | 15 | 15 |
| D3 | Case + microSD 64 GB | | 1 | 12 | 12 |
| D4 | LoRa USB dongle for bench (RA-02/RFM95W on USB or similar) | | 1 | 12 | 12 |

Dev station subtotal ≈ 120 USD. Allowed now under dev-kit gate; does not unlock any field purchase (per SPEC D11).

## Cost-reduction options (decision points)

- TPS2553 ×7 → discrete P-MOSFET limiter ×7: saves ≈4 USD, adds 12 parts, loses /FAULT.
- Schottky OR-ing → keep (ideal-diode +2 USD, not needed at 3 A).
- PD source 9/12 V dual profile → fixed 9 V only: saves ≈1 USD, halves negotiation.
- INA219 pair → INA219+discrete ADC shunt: saves ≈1 USD, less accuracy.
