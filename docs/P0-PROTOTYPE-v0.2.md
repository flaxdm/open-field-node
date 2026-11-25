# P0 prototype assembly v0.2

Goal: a working platform on standard boards only — no custom PCB. Validates the whole design (power tree, bays, control link, LoRa) before any KiCad entry.

## Architecture of P0

```
PDC004 (USB-C PD 12V) ──► SS54 OR ──► PTC 5A ──► P-MOSFET rev-polarity ──► TVS ──► VIN_12V
                                                                          │
        ┌─────────────────────────────────────────────────────────────────┤
        │                                                                 │
   XL4015 (5.05V)                                                  65W PD module
        │  5V_MAIN                                                   (charge out)
        ├── TPS2553 ── Bay1 (LoRa RFM95W 868)
        ├── TPS2553 ── Bay2 (GPS, optional)
        └── USB-A via SY6280 switch

   CM5 sits in Waveshare dock with its own 27W PSU (dock powers the core in P0).
   Wi-Fi dongle (MT7921AU) in dock USB → hostapd → phone control link.
   INA219 ×2 + MCP23017 on dock I2C header (3.3V logic).
   LoRa module on GPIO UART (TX/RX + 3.3V from dock, 5V from our tree).
```

Note: in P0 the CM5 is powered by the dock's own supply — our power tree only feeds bays, outputs and logic. This matches the final board where 5V_CORE gets its own limiter channel.

## Wiring table

| From | To | Detail |
|------|----|--------|
| USB-C PSU → PDC004 | VIN | PDC004 set/fixed 12 V |
| PDC004 out + SS54 ×2 | VIN_12V rail | OR-ing: second SS54 reserved for future XT30 bench input |
| VIN_12V → PTC 5A → AO3401A → SMBJ18A | protected rail | check polarity with multimeter before XL4015 |
| VIN_12V → XL4015 | 5V_MAIN @ 5.05 V | set voltage unloaded, then under load |
| 5V_MAIN → TPS2553 #1 (ILIM=2A, EN) | Bay1 5V | bay 5V on IDC pins 1–3 |
| 5V_MAIN → TPS2553 #2 | Bay2 5V | optional GPS |
| TPS2553 /FAULT ×2 → MCP23017 GPA0..1 | fault inputs | |
| MCP23017 GPB0..1 → TPS2553 EN | bay enables | |
| INA219 #1 shunt | VIN_12V high side | addr 0x40 |
| INA219 #2 shunt | 5V_MAIN | addr 0x41 |
| I2C (SDA/SCL) from dock header | MCP23017 (0x20) + INA219s | 2.2k pullups to 3.3V |
| Dock UART TX/RX | RFM95W RX/TX | cross-wired; 3.3V from dock header |
| Bay1 5V/GND | RFM95W VCC/GND | 5V from our tree (TPS2553 #1) |
| 5V_MAIN → SY6280 → USB-A | charge out | D+/D– shorted |
| VIN_12V → 65W PD module | USB-C charge out | confirm module out profile 9V/2A |
| LoRa RESET/DIO0 | dock GPIO pins | per ofn-fw bring-up doc later |

## Assembly order

1. Power tree only: PDC004 → protection → XL4015. Verify 5.05 V at 5V_MAIN with a 1 A dummy load (USB load resistor or bulb).
2. Add SY6280 + USB-A, test current limit with meter.
3. Add TPS2553 #1 with ILIM resistor; verify 2 A foldback with load.
4. Add MCP23017 + INA219s on I2C; `i2cdetect` must show 0x20/0x40/0x41.
5. Add PD 65W module, verify 9 V output into a phone.
6. Add RFM95W to Bay1 rails + UART; bench LoRa ping (later, with ofn services).

Every stage: check with multimeter before connecting the next stage. First power-up of each stage through a current-limited bench supply if available, else PTC + TPS2553 already limit damage.

## Software bring-up (on dock)

1. Raspberry Pi OS Lite 64-bit on microSD, SSH over dock Ethernet.
2. hostapd per NETWORK-MODULE baseline → phone sees OFN AP; control API dev starts here.
3. Skeleton services: ofn-powerd reads INA219s; ofn-slotd drives MCP23017 EN lines and reads /FAULT; both log to /var/lib/ofn.
4. LoRa: UART test with RFM95W (minimodem-class or meshtasticd when ready).

## What P0 does NOT cover

- PCIe H1, USB3 H2 routing, SDR — custom board territory.
- Battery path.
- All six bays (only 2 limiters assembled — that's the point).
- Field ruggedness.

## Prototype BOM add-on (beyond dev station list)

| Item | Qty | Est. ₽ |
|------|-----|--------|
| XL4015 module | 1 | 300 |
| PDC004 IP2721 trigger | 1 | 250 |
| SS54 diodes, AO3401A, SMBJ18A, PTC 5A, TPS2553 ×2, SY6280 | lot | 400 |
| MCP23017 module | 1 | 300 |
| INA219 module ×2 | 2 | 500 |
| 65W PD source module | 1 | 800 |
| Perfboard + terminals + wire | lot | 500 |

P0 add-on ≈ 3–3.5k ₽ on top of the dev station list.
