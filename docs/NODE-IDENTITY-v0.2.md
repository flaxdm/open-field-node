# OFN Node Identity & RTC v0.2

Two small board features of the carrier that were missing from the docs: the node's own passport EEPROM and a real-time clock battery.

## 1. Node identity EEPROM

Modules have their own EEPROMs (MODULE-DESCRIPTOR-v0.2). The node itself needs a fixed identity for pairing, multi-node deployments and support.

- Chip: AT24C32-class, same as modules, **fixed I2C address 0x57** (module bay EEPROMs use 0x50–0x56 per bay; 0x57 is reserved for the node and never handed to a bay).
- Format (256 B, matching module descriptor layout where sensible):

| Offset | Size | Field |
|---|---|---|
| 0 | 4 | signature: 0x4F 0x46 0x4E 0x31 ("OFN1") |
| 4 | 16 | node UUID (random, written once at factory/first boot) |
| 20 | 4 | hardware revision (PCB rev, BCD) |
| 24 | 8 | manufacture date (Unix time) |
| 32 | 2 | board configuration bitmap (which bays populated at factory, H1/H2 fitted) |
| 34 | 20 | ASCII node name, default "ofn-node" |
| 54 | 2 | CRC16 of bytes 0–53 |
| 56..255 | — | reserved (0xFF) |

- The SSID suffix is derived from UUID: `OFN-<first 4 hex of UUID>`, e.g. `OFN-9f3a`. Factory default before provisioning: `OFN-0000`.
- Writing UUID happens once (first boot of a fresh board writes it if the field reads all-0xFF). After that the region is treated as read-only by software; updates only via signed maintenance command.
- `ofn-ctl-api` exposes identity read-only over the control API; the Android app shows it in "About node".

## 2. RTC battery

Problem: CM5 has an RTC but no battery supply on our carrier; a node in the field without GPS starts at 1970 and wrong log timestamps until first time sync. Logs without trustworthy time are useless for audits.

Decision: **CR2032 holder + series diode (BAT54) + 1kΩ to CM5 RTC pads** on the carrier.

- Cheap (~0.5 USD), 5–10 years life (RTC draws ~2 µA).
- Diode prevents back-feeding the rail from the battery; 1kΩ limits charge current if the holder is inserted backwards (cheap insurance).
- Time sync priority in software: GPS fix → NTP if internet → phone time over control API → RTC. After any better source, RTC is set.
- CM5 Lite RTC pinout per Raspberry Pi CM5 datasheet; verify on carrier schematic entry in KiCad (open check).

Alternatives considered: supercap (weeks of backup, costlier, bigger), no RTC (rejected — field logs need time). CR2032 wins.

## 3. Carrier schematic additions (to apply at KiCad entry)

1. U7 AT24C32, address pins to 0x57, on the same I2C bus as INA219/MCP23017.
2. BT1 CR2032 holder + D-batt BAT54 + R 1kΩ to RTC_VCC of CM5.
3. Both near the SODIMM area, keep-out respected; battery holder accessible without disassembly is a plus for v0.3 case, not blocking.
