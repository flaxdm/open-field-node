# Module descriptor v0.2 (Pi HAT EEPROM format, extended)

Per D10: own/community modules carry an ID EEPROM; we adopt the open Raspberry Pi HAT EEPROM format (documented in Raspberry Pi HAT+ spec) instead of inventing a private one. Programming tools exist (`eepmake`/`eepdump` from the Raspberry Pi utilities).

## Hardware

- AT24C32-class I2C EEPROM, address 0x50..0x57 per bay slot.
- Write-protect pin: grounded on module after programming (WP tied to GND enables writes; final modules tie WP to 3V3 via pad option).
- Powered from 3V3_LOGIC (pin 16 of the bay).

## EEPROM content (256-byte atom set)

| Offset | Size | Field | Content |
|--------|------|-------|---------|
| 0 | 4 | atom1 header | signature 0x52 'R' 0x2D '-', ver 0x01, count 2, per Pi HAT spec |
| 5..24 | 20 | uuid | 128-bit unique module ID (first 6 bytes = OUI-style vendor prefix) | |
| 24 | 2 | pid | product ID (vendor-assigned) | |
| 26 | 2 | pver | product version (BCD) | |
| 28 | 1 | vsn | PCB version | |
| 29 | 1 | signals used | bitmap: bit0 SPI, bit1 I2C, bit2 UART, bit3 EN/PG used, bit4..7 reserved | |
| 30 | 2 | 5V max current | mA, e.g. 500 | |
| 32 | 2 | 5V inrush | mA | |
| 34 | 1 | module class | 0x01 LoRa, 0x02 GPS, 0x03 status, 0x04 network, 0x05 DMR, 0x10.. custom | |
| 35 | 1 | RF region bitmap | bit0 433, bit1 868, bit2 915 MHz (0 = none) | |
| 36..55 | 20 | ASCII name | e.g. "OFN-LORA-01" | |
| 56 | 1 | custom data length | | |
| 57.. | n | custom data | vendor payload (pins, calibration) | |

atom 2 = full custom block? No — keep single-atom HAT structure with our extension fields in the vendor atom payload (offset ≥ 28 is vendor area per HAT spec). Total 256 bytes.

## Read flow (node side)

1. Bay powered, EEPROM readable at 0x50+slot.
2. Validate signature + CRC; on failure → treat slot as manual-config module (D10 path 1).
3. Read class, current limits → power daemon applies limits before enabling the bay.
4. Read name/uuid → telemetry and module registry.
5. RF region bitmap → Gate 4 RF policy check before allowing TX.

## Programming flow (module production)

1. Module assembled, EEPROM blank; WP open.
2. On carrier (or Waveshare dock bench): `eepmake ofn-lora-01.txt ofn-lora-01.eep` then write via I2C at 0x50.
3. Verify with `eepdump`; close WP solder jumper.

## Compatibility notes

- Ready-made mikroBUS Click boards have no EEPROM → slot config manual, module class chosen from app library (D10 path 1). I2C scan can hint module type by known device addresses (e.g. GPS u-blox at 0x42).
- The descriptor never contains executable code — only metadata. Security lab modules get an extra signature field in custom data (reserved, defined with security module later).
