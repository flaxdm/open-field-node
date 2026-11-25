# SPEC v0.2 — Architecture baseline before purchases

Status: draft for review

Goal: define the platform well enough that no unnecessary hardware is bought.

## Main rule

No purchase is approved until a decision gate in this document is passed.

The project is still in architecture definition. v0.2 is not a shopping list.

## Decisions

| ID | Decision | Status |
|----|----------|--------|
| D1 | Reference compute target is Raspberry Pi CM5-class, preferably 8 GB Lite/no-wireless for the first field node | accepted |
| D2 | Backplane must provide 12 V main bus, controlled rails, USB 3.0, Gigabit Ethernet, I2C, UART, GPIO and module ID bus | accepted |
| D3 | Backplane should reserve optional PCIe lane routing for future network/AI/storage modules | accepted |
| D4 | Module format is a card-style replaceable unit with keyed mechanical installation and front-panel RF/connectors | accepted |
| D5 | MVP purchase scope is limited to: core carrier, power, status, one network module, one LoRa module | accepted |
| D6 | DMR, LTE/SIM, second Wi-Fi, dual Ethernet and SDR host are post-MVP expansion modules | accepted |
| D7 | Low-speed modules use a keyed IDC 2×8 connector; high-speed uses one PCIe x1 edge slot and one USB3 slot | accepted |
| D8 | Dual power input: field XT30 (8–18 V, priority) + USB-C with PD trigger at 12 V, OR-ed at the input | accepted |
| D9 | Two charging outputs: USB-C PD source (9 V/2 A or 12 V/1.5 A) for fast phone charging + simple USB-A 5 V/2.4 A | accepted |
| D10 | Module identification: ready-made third-party modules (e.g. mikroBUS Click) — manual slot configuration + I2C address hints; own/community modules — ID EEPROM in Raspberry Pi HAT format (AT24C32-class on I2C 0x50) | accepted |
| D11 | Dev station is a CM5 8 GB Lite (no-wireless, CM5108000) on a Waveshare CM5-IO-BASE-A dock, bought under the dev-kit gate; the same CM5 later becomes the field core — one purchase, no discarded hardware | accepted |
| D12 | Control link phone ↔ node: Wi-Fi AP hosted on the MVP network module (option A); USB-C tethering reserved as fallback | accepted |
| D13 | RF region: 868 MHz band (LoRa RU864 profile); 433 MHz kept as secondary option | accepted |

## Core selection rationale

Primary reference: Raspberry Pi CM5-class module.

Why:
- strong ecosystem and documentation;
- Linux support;
- USB 3.0 and Gigabit Ethernet;
- good peripheral support;
- long production horizon for CM5 family;
- compatible with a broad carrier ecosystem;
- good upgrade path for future cores.

Practical note: 16 GB variants are not the MVP target because of cost and current memory price pressure.

## Backplane v0.2 requirements

The backplane must provide:

- 12 V main field input;
- reverse polarity protection;
- controlled module rails;
- local 5 V / 3.3 V rails;
- USB 3.0;
- Gigabit Ethernet;
- I2C for module ID and telemetry;
- UART for service;
- GPIO for button/status;
- optional PCIe lane footprint.

The first carrier may leave PCIe unpopulated if it increases cost or risk.

## Module connector / mechanical baseline

Two connector classes:

- Class L: keyed shrouded IDC 2×8 box header for low-speed bays (LoRa, GPS, status, DMR control). 16 pins: 5 V ×3, GND ×3, I2C, UART, SPI ×3, CS/ID, EN/PG, 3.3 V.
- Class H: one PCIe x1 edge slot (H1) for SDR/NVMe/2nd GbE and one USB3 slot (H2) for USB3 SDR/LTE/storage.

See [Module connector comparison v0.2](MODULE-CONNECTOR-v0.2.md).

## Power baseline

See [Power architecture v0.2](POWER-ARCHITECTURE-v0.2.md). Summary:

- 12 V field input backbone, 8–18 V, from XT30 (priority) or USB-C PD trigger at 12 V;
- buck 5 V / 6 A;
- buck 3.3 V / 2 A;
- per-bay switched 5 V, limited to 2 A, with measurement;
- INA219 monitoring on input and 5 V backbone;
- two device-charging outputs (D9): USB-C PD source + USB-A 5 V/2.4 A;
- battery postponed to a separate gate.

## MVP purchase scope

Allowed only after gates pass:

1. compute core carrier;
2. power module;
3. status/service module;
4. one network module;
5. one LoRa module.

Not in MVP purchase scope:

- DMR hotspot module;
- LTE modem;
- SIM hardware beyond connector planning;
- second Wi-Fi module;
- dual Ethernet beyond one initial port unless already planned on the same module;
- SDR host module;
- 5G hardware;
- final enclosure production.

## Dev station (separate from MVP field scope)

- CM5 8 GB Lite (no-wireless) + Waveshare CM5-IO-BASE-A + 27 W PSU + cooler + microSD + LoRa bench set. Full list: [Dev station buy list](DEV-STATION-BUYLIST.md) (~25–26k ₽).
- Purpose: software bring-up (Android ↔ node protocol, slot logic, LoRa software via GPIO UART) before the custom carrier exists.
- The CM5 moves into the field carrier after Gate 3 — this purchase doubles as the field core.
- Purchase rule: dev-kit gate — allowed to buy now, does not unlock any other field purchase.

## Purchase gates

Gate 1 — power budget

Drafted in Power architecture v0.2. Needs review.

Gate 2 — module interface freeze

Connector classes drafted in Module connector comparison v0.2. Backplane pinout and module descriptor still open.

Gate 3 — core carrier reference

The first carrier should be validated as schematic/mock-up before buying expensive compute modules.

Gate 4 — RF plan

A region-specific RF plan must exist before buying antennas, filters, DMR, LTE or SDR modules.

## Current open questions

- final MCU companion choice;
- battery chemistry and power path;
- Android ↔ node control protocol;
- module descriptor schema version (resolved by D10: Pi HAT EEPROM format);
- Class L pinout compatibility with mikroBUS (same signal set, different power pin count — decide at schematic stage);
- enclosure dimensions;
- region-specific RF rules (closed by D13: 868 MHz; legal TX power check remains in RF plan);
- exact Class L connector part number (family fixed: keyed shrouded IDC 2×8 2.54 mm; brand/part at PCB entry);
- per-bay limiter: IC eFuse vs discrete;
- PD trigger chip choice (CH224K / IP2721 / similar);
- PD source controller for charging output (IP2723T / CH235 / similar);
- input OR-ing: Schottky diodes vs ideal-diode MOSFET.

## Progress log

- Power tree and rails: drafted.
- Dual power input (D8: XT30 priority + USB-C PD 12 V): decided.
- Charging outputs (D9: USB-C PD source + USB-A 5 V): decided.
- Module identification (D10): manual + I2C hints for ready-made modules, Pi HAT EEPROM for own modules — decided. Module descriptor schema freeze reduces to adopting the Pi HAT EEPROM format.
- Backplane power schematic stage design: drafted (see BACKPLANE-SCHEMATIC-v0.2.md).
- MVP carrier schematic stage design: drafted (see CARRIER-SCHEMATIC-v0.2.md).
- Class L connector family confirmed: generic keyed 2.54 mm shrouded 2×8 box header (≈1 USD; mikroBUS signal-set compatible, socket compat open).
- No-purchase BOM: drafted (see BOM-v0.2.md), board total ≈30–38 USD excl. core and LoRa radio.
- Gate 1 power budget: passed with changes — 12 V input 5 A, OR-ing SS54, bay cap 3 A, USB-A shed rule (see POWER-BUDGET-v0.2.md).
- Module descriptor: Pi HAT EEPROM format adopted with OFN extension fields (see MODULE-DESCRIPTOR-v0.2.md).
- Software architecture: services, control API, Android companion, survival mode drafted (see SOFTWARE-ARCHITECTURE-v0.2.md). Control link decided: Wi-Fi AP (D12).
- RF plan draft for Gate 4 (868 MHz, see RF-PLAN-v0.2.md).
- Ready module market check: bench bring-up of the full power tree possible from ready modules (XL4015, PDC004, INA219, MCP23017, 65 W PD source); custom work reduces to per-bay limiter + input protection (see READY-MODULES-v0.2.md).
- Module connector classes: drafted.
- MVP carrier block diagram: drafted.

## Next engineering actions

1. KiCad entry of backplane power schematic + carrier schematic (after review of design pack).
2. Envelope/mechanical draft: board size, bay spacing, front panel (blocks enclosure).
3. Wi-Fi AP module selection for the MVP network slot (ESP32-class vs USB-WiFi on H2).
4. First code milestone on dev station: ofn-powerd + ofn-slotd skeleton over Waveshare dock (GPIO header).
