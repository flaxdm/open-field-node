# Черновик Capability API

Модули описывают не сырые железки, а свои возможности.

Примеры:

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

## Пример module descriptor

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

## Модель миссий

Миссии — декларативные задачи, исполняемые узлом:

- по расписанию;
- по событию;
- по команде телефона;
- автономно с записью результата.

Примеры:

- записать прохождение спутника;
- сканировать Wi‑Fi каждые N минут;
- вести GPS/Wi‑Fi wardriving log;
- отправлять SMS alert при потере uplink;
- собрать отчёт по тестовой VLAN.
