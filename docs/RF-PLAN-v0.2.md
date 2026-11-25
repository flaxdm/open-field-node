# RF plan v0.2 (draft, Gate 4)

Region decision D13: 868 MHz primary. All RF work in the project follows the security rule: own devices / own network / isolated lab only.

## Bands and intended use

| Function | Band | Profile | Status |
|----------|------|---------|--------|
| LoRa / Meshtastic | 868 MHz | LoRaWAN RU864 channel plan (864.1–865.9 MHz range class) | primary |
| LoRa fallback | 433 MHz | RU433 profile | optional module variant |
| Wi-Fi control AP | 2.4 GHz | 802.11n AP on network module (D12) | local link only, low TX power |
| SDR listening | wide RX | RX only in MVP; TX gated separately | lab use |
| DMR hotspot | UHF 400–470 MHz (regional) | post-MVP, needs legal check | not planned for purchase |

## TX power policy (before any TX module purchase)

- 868 MHz ISM: keep TX ≤ 25 mW ERP with duty-cycle discipline per regional rules; final numbers must be verified against current RU/EU regulations before enabling TX in firmware.
- Wi-Fi AP: minimum necessary power, fixed channel, no internet routing through it.
- All TX-capable modules carry the RF region bitmap in their descriptor; firmware refuses TX if descriptor region ≠ configured region.

## Antenna and coexistence plan (board-level)

- LoRa 868 and Wi-Fi 2.4 antennas: keep ≥ 5 cm separation at the front panel; use different polarizations where possible.
- GPS (L1 1575 MHz) bay: farthest bay from Wi-Fi AP module; GPS front-end sees Wi-Fi 3rd harmonic — budget a SAW filter footprint on the GPS module.
- SDR H1 slot: 12 V aux reserved; antenna routed via front-panel SMA.
- No onboard antennas: all RF modules use front-panel SMA/IPEX pigtails.

## Module RF requirements (for module spec)

- LoRa module: SX1276-class, 868 MHz, low-TX variant (≤100 mW PA, firmware-limited), IPEX/SMA.
- Wi-Fi module: 2.4 GHz AP-capable (ESP32-class or USB-WiFi on H2), external antenna via IPEX if shielded enclosure.
- All modules: harmonic/spurious pass per module spec test plan (lab, own equipment).

## Gate 4 checklist (before RF hardware purchase)

1. Region regulations re-verified (868 MHz TX power, duty cycle, SRD rules).
2. LoRa module variant chosen (868, antenna connector, TX limit).
3. Wi-Fi module chosen (AP capability confirmed).
4. Antenna isolation sketch on the mechanical draft (envelope doc).
5. RF module test plan (lab bench, own equipment).
