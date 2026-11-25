# Module connector comparison v0.2

Constraint: modules must use common, cheap parts and avoid stacking everything on USB.

## Connector options

| Option | Cost | Current | Data | Verdict |
|--------|------|---------|------|---------|
| 2.54 mm IDC box header (2×5 / 2×8) | very cheap | ~1–3 A via paralleled pins | UART, I2C, SPI, GPIO | best for low-speed bays |
| JST-XH / PH wired | cheap | 1–3 A | low-speed only | cables, not backplane backbone |
| M.2 socket (E-key / M-key) | moderate | 3.3 V logic only | PCIe, USB | good for H-bay, but custom pinout risk |
| PCIe x1 gold finger edge | cheap pair | power via pins OK | PCIe gen1/gen2 | best for one H-bay |
| USB-C as module connector | cheap | PD complexity | one USB lane | rejected: misleading and fragile |

## Recommended two-class system

### Class L — low-speed module bays (LoRa, GPS, status, DMR control)

Keying: shrouded 2.54 mm IDC box header, 2×8 (16 pin).

Proposed pinout:

| Pins | Signal |
|------|--------|
| 1–3 | +5 V (paralleled) |
| 4–6 | GND |
| 7 | I2C SDA |
| 8 | I2C SCL |
| 9 | UART TX |
| 10 | UART RX |
| 11 | SPI SCK |
| 12 | SPI MISO |
| 13 | SPI MOSI |
| 14 | CS / module ID |
| 15 | EN / PG |
| 16 | +3.3 V |

Rated 5 V / 2 A per bay via three paralleled pins.

### Class H — high-speed slots

- H1: PCIe x1 edge connector (gold fingers on module) for SDR host, NVMe or second GbE.
- H2: USB 3.0 via internal header or cable for one USB3 device (second SDR, LTE modem, storage).
- Native GbE from CM5 routed to the backplane as the primary network path.

## USB budget rule

- Maximum two high-bandwidth USB devices.
- SDR class: only on H-bays.
- LTE modem: USB 2.0 is enough.
- Control modules never use USB.

## MVP carrier block diagram

```
                    ┌────────────────────────────┐
 12 V in ──prot────►│ 5V/6A buck      3.3V/2A   │
                    └───┬─────────┬────────┬─────┘
                        │         │        │
   ┌──── CM5 core ──────┼─────────┼────────┼──────────────┐
   │ GbE ───────────────►│ primary RJ45               │
   │ PCIe x1 ───────────►│ H1 (SDR / NVMe / 2nd GbE)  │
   │ USB3 ──────────────►│ H2 (USB3 SDR / LTE / disk)   │
   │ USB2 ──────────────►│ LTE / debug                  │
   │ SPI0/1, UART, I2C ─►│ Bay1 LoRa  Bay2 GPS  Bay3 …  │
   │ GPIO (via MCP23017)─►│ bay enables / power-good     │
   └──────────────────────┴──────────────────────────────┘
```

## Conclusion

- Freeze Class L on keyed IDC 2×8 box header.
- Freeze Class H1 on PCIe x1 edge, H2 on USB3.
- Next: draft schematic of MVP carrier and backplane pinout.
