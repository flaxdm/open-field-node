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
- region-specific RF rules;
- exact Class L connector keying part number;
- per-bay limiter: IC eFuse vs discrete;
- PD trigger chip choice (CH224K / IP2721 / similar);
- PD source controller for charging output (IP2723T / CH235 / similar);
- input OR-ing: Schottky diodes vs ideal-diode MOSFET.

## Progress log

- Power tree and rails: drafted.
- Dual power input (D8: XT30 priority + USB-C PD 12 V): decided.
- Charging outputs (D9: USB-C PD source + USB-A 5 V): decided.
- Module identification (D10): manual + I2C hints for ready-made modules, Pi HAT EEPROM for own modules — decided. Module descriptor schema freeze reduces to adopting the Pi HAT EEPROM format.
- Module connector classes: drafted.
- MVP carrier block diagram: drafted.

## Next engineering actions

1. Freeze module descriptor format — adopt Pi HAT EEPROM format (open standard, existing tools).
2. Draft backplane power tree schematic from Power architecture v0.2.
3. Draft MVP carrier schematic (CM5 + backplane connector pinout).
4. Confirm Class L IDC keying part numbers and current rating.
5. Produce a no-purchase bill of materials for review.
