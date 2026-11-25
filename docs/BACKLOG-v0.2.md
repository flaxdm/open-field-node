# OFN Backlog v0.2 (high-level working-out)

This document expands the table in SPEC. Each item: what, why, how it fits the current architecture, effort, dependencies, and activation trigger. Statuses: `backlog` (untouched) / `ready for discussion` (can be detailed) / `decided` (needs a date).

General backlog rule: an item activates on the user's command after its trigger fires. No purchases or design work before activation.

## B1. USB-gadget audit — `ready for discussion`

**What.** The CM5 can act as a USB device (gadget mode): pretend to be a keyboard, mouse, NIC, flash drive. We use this to test our own PCs/tablets: "what happens if an unknown device is plugged in" — a controlled vector, legal because the target is ours.

**Why.** The platform's security-lab profile: auditing trusted peripherals, checking autorun/USB-guard policies on own machines, pentest drills in own network.

**How it fits.** Pure software: `configfs`/libcomposite on Linux, no hardware changes. The USB-C output on the panel already exists; role switching (host/gadget) is software, exposed in `ofn-ctl-api`. Scenarios packaged as "missions": HID keystroke injection, RNDIS network, CDC-ECM.

**Effort.** 2–3 weeks of software (the `ofn-gadgetd` service is new to the v0.2 architecture), zero hardware.

**Depends on.** Working `ofn-ctl-api` and the mission mechanism (after the first code milestone).

**Trigger.** After P1, if security-lab scenarios are actually used.

## B2. Bluetooth module — `backlog`

**What.** A backup / low-power channel to the phone over BLE: pairing, status, "wake the node".

**Why.** The Wi-Fi AP eats both airtime and battery; BLE allows a standby mode for long autonomous deployments and control without raising the AP.

**How it fits.** Two paths: (a) USB-BLE dongle in H2 — fast, hacky; (b) Class L card on ESP32 — proper, part of B7. No BLE needed in the CM5 core itself.

**Effort.** (a) 3–4 days; (b) on top of B7 + one week of software.

**Trigger.** When a real long-standby task appears (field trips on a powerbank where hours of runtime matter).

## B3. DMR hotspot — `backlog`

**What.** A Class L card with a DMR modem (MMDVM-like) + 12 V aux from the backplane for a PA. The node becomes an entry point into own/friendly DMR repeater networks.

**Why.** Amateur DMR radio is popular with this device's target audience; strengthens the "field communications" mission.

**How it fits.** 22×60 card (STM32 + modem chip), descriptor class 0x05 already reserved, 12 V aux output is in the power schematic. Software: MMDVMHost integration as a service.

**Effort.** 4–6 weeks hardware, 2–3 weeks software.

**Depends on.** P1 (stable backplane), regulatory cleanliness: amateur bands only, regional rules — check RF plan Gate 4.

**Trigger.** After MVP release + access to a friendly DMR network for legal testing.

## B4. LTE/modem — `backlog`

**What.** A card/module with an LTE modem and SIM slot (backplane footprint reserved post-MVP).

**Why.** Node outside Wi-Fi/LoRa coverage: telemetry home, mission uploads, backup control channel.

**How it fits.** Option 1: M.2 WWAN modem in H1 (PCIe) — proper but expensive and large for the enclosure. Option 2 (recommended): USB modem (Quectel EG25-G class) in H2 via a USB3 19-pin → M.2-key-B adapter. SIM holder on backplane or on the card; decide at activation.

**Effort.** 2–3 weeks hardware (adapter + SIM), 1–2 weeks software (ModemManager).

**Depends on.** Local network coverage/pricing; extra SMA antenna connectors on the panel.

**Trigger.** A real "node far from base" scenario.

## B5. SDR — `backlog`

**What.** Software-defined radio: wideband reception and spectrum analysis of own networks/airwaves.

**Why.** Extended security-lab: hunt interference, watch spectrum usage, radio education.

**How it fits.** Off-the-shelf USB SDR (RTL-SDR Blog V4, ~$30) in H2 — zero custom hardware. Software: `ofn-spectr` service (wrapper over rtl_power/SoapySDR), data in the API. Transmission needs licensing — MVP is receive-only.

**Effort.** One week software, zero hardware (buy the SDR dongle).

**Trigger.** After B1 or together with it, if the security profile lives.

## B6. LiFePO4 subsystem — `backlog`

**What.** Built-in 12 V battery (4S LiFePO4 ~12.8 V) + charging from XT30/USB-C + controller. Currently replaced by an external powerbank.

**Why.** Full autonomy without external boxes: the node charges from any source and manages depth of discharge itself.

**How it fits.** The XT30 input tree already accepts 8–18 V — 4S LiFePO4 (10–14.6 V) is compatible. Add: battery XT30 connector, BMS with UART (JBD/Xiaoxiang class) on an I2C/UART line, charge policy in `ofn-powerd`. Critical: LiFePO4 is safer than Li-ion but needs a proper BMS.

**Effort.** 3–4 weeks hardware, one week software, battery+BMS purchase ~4–8k ₽.

**Depends on.** P1 thermal tests (a battery adds degrees), capacity choice from real P0 autonomy measurements.

**Trigger.** First real "into the wild" trip off-grid.

## B7. ESP32 Wi-Fi community card — `ready for discussion`

**What.** The reference Class L card on ESP32: a module template anyone can replicate. Standby Wi-Fi, BLE presence, community GPIO experiments.

**Why.** Proves the module format is open; cheap entry for contributors; B2 builds on it.

**How it fits.** Blank card (IDC 5V→3V3 supply, connector, ESP32-WROOM footprint + antenna) + example firmware (ESPHome-like UART-WiFi bridge). Published as a reference design in the repo.

**Effort.** One week hardware, one week example software.

**Trigger.** First external interest in modules OR our own need for B2.

## B8. Sealing/climate — `backlog` (deliberately deferred)

**What.** Panel splash protection, temperature range, board coating.

**Why.** Field = moisture, dust, frost.

**How it fits.** Sealed SMA/USB-C on the panel, conformal coating, component temp rating. These are **v0.3 case-level decisions**, not schematic.

**Trigger.** Before designing the v0.3 case, autumn–winter 2026 after first outings.

## Summary table

| # | Item | Hardware | Software | Depends on | Trigger |
|---|---|---|---|---|---|
| B1 | USB-gadget audit | 0 | 2–3 wk | ofn-ctl-api + missions | security-lab alive |
| B2 | Bluetooth module | 0–1 wk | 1 wk | B7 (proper path) | long standby |
| B3 | DMR hotspot | 4–6 wk | 2–3 wk | P1, legal access | MVP + DMR network |
| B4 | LTE modem | 2–3 wk | 1–2 wk | "far away" scenario | first outing |
| B5 | SDR receive | 0 (buy) | 1 wk | H2 free | with B1 |
| B6 | LiFePO4 | 3–4 wk | 1 wk | P1 thermal tests | first off-grid trip |
| B7 | ESP32 card | 1 wk | 1 wk | — | external interest |
| B8 | Sealing | v0.3 case | 0 | cold/wet outings | autumn–winter 2026 |

Activating any item = user's command + detailing to plan level, same as main stages (options, decision, documents).
