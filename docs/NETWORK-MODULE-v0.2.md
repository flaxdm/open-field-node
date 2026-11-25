# Network module selection v0.2 (Wi-Fi AP, per D12)

The MVP network module provides: GbE (fixed, on carrier) + Wi-Fi AP for phone control. Options for the Wi-Fi part:

| Option | HW | Pros | Cons |
|--------|----|------|------|
| A. USB-WiFi on H2 | RTL8811AU-class or MT7921AU dongle, IPEX antenna to panel | zero custom firmware; standard Linux `hostapd`; works on dev station today | uses the single H2 slot; dongle sticking out unless panel-mounted |
| B. ESP32 module in Class L bay | ESP32-WROOM + custom module firmware | integrated: slot power control, EEPROM descriptor, no USB budget spent | we write/maintain a Wi-Fi+bridge firmware; more bring-up |
| C. ESP32 + Ethernet co-processor | ESP32 with wired link to CM5 | frees H2 | duplicate network stack, complexity |

Recommendation: **A for MVP** — the control link is critical-path software (app ↔ node protocol), so take the option with zero firmware work: MT7921AU (in-kernel driver, good AP support) or RTL8811AU dongle, panel IPEX antenna, `hostapd` on the CM5. Revisit B in v0.3 when the module ecosystem matures — an ESP32 Wi-Fi card is a nice community module either way.

Dev station note: same dongle works on the Waveshare dock now — control-API development starts before the carrier exists.

## Hostapd baseline config (for the docs)

- Mode: g/n, 2.4 GHz fixed channel 6 (or least-congested), 20 MHz width.
- SSID: OFN-<short-id>; WPA2/WPA3 transition, password from pairing flow.
- No DHCP to uplink; dnsmasq serves 10.0.0.x to phones only; node at 10.0.0.1.
- TX power: set to ≤ 14 dBm initially per RF plan power policy.
