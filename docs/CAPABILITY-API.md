# Capability API Draft

Modules expose capabilities instead of raw hardware details.

Examples:

- `net.route`
- `net.bridge`
- `net.monitor`
- `wifi.scan`
- `wifi.ap`
- `lte.modem`
- `gps.fix`
- `rf.lora`
- `rf.subghz`
- `dmr.hotspot`
- `sdr.rx`
- `sdr.iq_record`
- `power.battery`
- `power.solar`
- `storage.log`
- `sec.inventory`
- `sec.rogue_dhcp_lab`
- `sec.tls_mitm_lab`
- `sec.deauth_lab`

## Module descriptor example

```json
{
  "id": "net.dual_wifi",
  "type": "network",
  "hw_version": "0.1",
  "fw_version": "0.1.0",
  "capabilities": ["wifi.scan", "wifi.ap", "net.bridge", "net.monitor"],
  "power": {
    "input_rail": "12v",
    "max_w": 8
  },
  "rf": {
    "bands_mhz": [2400, 5000]
  }
}
```

## Mission model

Missions are declarative tasks executed by the node:

- scheduled;
- event-driven;
- phone-triggered;
- autonomous offline execution.

Examples:

- record satellite pass;
- scan Wi-Fi every N minutes;
- run GPS/Wi-Fi wardriving log;
- send SMS alert on uplink loss;
- create security report for test VLAN.
