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

The first module standard should be defined around:

- card-style modules;
- keyed insertion;
- front-panel connectors;
- replaceable RF panel;
- 3D-printable enclosure support;
- serviceability without full disassembly.

The exact connector part number is not selected yet. Connector choice is gated by:

- current rating;
- pin count;
- impedance/RF separation needs;
- availability;
- cost;
- assembly risk.

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
A full system power budget must exist before buying power parts.

Gate 2 — module interface freeze  
Backplane pinout, module connector family, and module descriptor format must be frozen before buying connectors.

Gate 3 — core carrier reference  
The first carrier should be validated as schematic/mock-up before buying expensive compute modules.

Gate 4 — RF plan  
A region-specific RF plan must exist before buying antennas, filters, DMR, LTE or SDR modules.

## Current open questions

- final MCU companion choice;
- battery chemistry and power path;
- Android ↔ node control protocol;
- module descriptor schema version;
- enclosure dimensions;
- region-specific RF rules.

## Next engineering actions

1. Freeze module descriptor JSON schema.
2. Draft backplane power tree.
3. Draft MVP carrier block diagram.
4. Compare 2–3 module connector families.
5. Produce a no-purchase bill of materials for review.
