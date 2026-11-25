# OFN Module Standard v0.2

Standard for field-replaceable modules (Class L) for the Open Field Node carrier.

## 1. Mechanical

| Parameter | Value |
|---|---|
| Card size | 22 × 60 mm |
| Card thickness | 1.6 mm FR4 (prototype), 0.8–1.0 mm allowed for RF modules |
| Mounting pitch on carrier | 25 mm (6 bays) |
| Connector | IDC 2×8 (16-pin, 2.54 mm), keyed, shrouded header on carrier |
| Retention | Card slides into 1.5 mm guide slots in the case wall; IDC friction + slot = no screws on prototype |
| Front face | SMA connector (RF modules) or status LED window, flush with front panel |
| Labelling | Silklayer name + version on front edge; bay number written on carrier |

Rationale: 22×60 matches the 25 mm bay pitch with 1.5 mm walls between bays. IDC 2×8 is cheap, keyed against reverse insertion, and a shrouded header protects pins in the field. No gold fingers on prototype — IDC is hand-solderable and debuggable; a "gold finger" edge connector is a v0.3 option if contact wear becomes an issue.

## 2. Electrical (Class L pinout)

16-pin IDC, looking at the module card, pin 1 marked by key/triangle:

| Pin | Signal | Notes |
|---|---|---|
| 1–3 | BAYn_5V | 5 V from bay limiter, ≤2 A per bay |
| 4–6 | GND | Common ground |
| 7 | SDA | I2C, 3.3 V logic, shared bus (1kΩ pull-ups on carrier) |
| 8 | SCL | I2C, 3.3 V logic |
| 9 | TX | UART from module → CM5 (3.3 V) |
| 10 | RX | UART from CM5 → module (3.3 V) |
| 11 | SCK | SPI (shared, 3 peripherals max) |
| 12 | MISO | SPI |
| 13 | MOSI | SPI |
| 14 | CS | Chip select, one per bay from MCP23017 |
| 15 | EN/PG | Enable out / power-good in (module pulls up when ready) |
| 16 | 3V3 | 3.3 V from carrier rail, ≤200 mA per bay |

Rules:
- All logic is 3.3 V. Never drive 5 V into pins 7–15.
- If a module needs only UART (e.g. LoRa), it leaves SPI pins unconnected — no problem.
- CS and EN/PG are per-bay, routed via the MCP23017 expander so the slot manager knows what is inserted and can reset a hung module by toggling EN.
- I2C is a shared bus: 0x50–0x57 reserved for module EEPROMs (HAT format), other addresses must be listed in the module descriptor.

## 3. Module identity

Two ways, decided in D10:

1. **Own modules**: AT24C32-class EEPROM (256 B) on I2C 0x50–0x57, layout per `MODULE-DESCRIPTOR-v0.2.md` (UUID, class, current limits, RF region, ASCII name).
2. **Ready-made modules (mikroBUS Click etc.)**: no EEPROM. `ofn-modmgr` identifies by manual config (bay → module type) plus optional I2C address probing hints from the descriptor registry.

Class codes: 0x01 LoRa, 0x02 GPS, 0x03 status, 0x04 network, 0x05 DMR.

## 4. Module card checklist (before plugging into bay)

- [ ] Card fits 22×60 outline, no parts taller than 12 mm except SMA/LED
- [ ] Pin 1 marking visible, keying shroud compatible
- [ ] 5 V draw ≤ declared in descriptor; inrush < limiter trip (TPS2553, 2 A)
- [ ] EEPROM programmed and readable at chosen 0x5x address
- [ ] EN/PG pull-up present (10k to 3V3 on module)
- [ ] SMA nut torque-hand-tight; no sharp card edges near coax

## 5. RF module extras

- Antenna keep-out: ≥5 mm ground-free ring around SMA on the card side.
- For 868 MHz: declare region bitmap in descriptor; firmware must refuse TX outside declared region.
- GPS modules: SAW filter footprint mandatory, place GPS in the bay farthest from Wi-Fi.

## 6. Open points for v0.3

- Edge-connector ("gold fingers") variant vs IDC: decide after P0 wear test.
- Card retention: if IDC friction is not enough in the case, add a printed top rail.
- Class H modules (PCIe/USB3): separate standard, not covered here.
