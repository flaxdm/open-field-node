# OFN Wi-Fi Bring-up Notes v0.2

How to turn the CM5 dev station into the node's control AP using a USB-WiFi dongle on the H2 (USB3 19-pin) slot. Decision D12.

## 1. Hardware

| Part | Status | Notes |
|---|---|---|
| Dongle MT7921AU (in-kernel driver) | preferred | works out of the box on kernel ≥5.18; CM5 kernel 6.6+ OK |
| Dongle RTL8811AU | fallback | needs out-of-tree driver `rtl8811au` (dkms); buy only if MT7921AU unavailable |
| Antenna | required | stock 2 dBi duck is fine for 10–20 m control link |

Buy one MT7921AU dongle. Avoid RTL8812AU (driver mess) and anything Realtek with no mainline driver.

## 2. Identify the interface

```bash
lsusb | grep -i -E "mediatek|ralink|realtek"
ip link                 # new wlan interface appears
iw dev                  # shows phy#, interface name, supported bands
```

Expected: `Interface wlan1` (or wlan0 if CM5 Lite has no wireless — our core has none, so it will be wlan0).

## 3. Baseline hostapd config

Save as `/etc/hostapd/ofn.conf`:

```ini
interface=wlan0
driver=nl80211
ssid=OFN-0001
hw_mode=g
channel=6
ieee80211n=1
wmm_enabled=1
auth_algs=1
wpa=2
wpa_passphrase=change-me-pairing
wpa_key_mgmt=WPA-PSK
rsn_pairwise=CCMP
country_code=RU
ignore_broadcast_ssid=0
```

Notes:
- Fixed channel 6 — no auto-channel on a field box (predictability beats throughput).
- Power: leave default; if interference matters, `txpower` via `iw` (≤14 dBm per RF plan).
- The SSID suffix is the device ID from pairing; `OFN-0001` is the factory default until paired.

## 4. Static IP + DHCP for phones only

`/etc/dnsmasq.d/ofn.conf`:

```ini
interface=wlan0
bind-interfaces
dhcp-range=10.0.0.10,10.0.0.50,255.255.255.0,24h
dhcp-option=3,10.0.0.1
dhcp-option=6,10.0.0.1
```

`nmcli`/`netplan` equivalent: assign `10.0.0.1/24` static on wlan0, no upstream route (control link is isolated by design; internet comes via Ethernet/USB-C later).

Start services:

```bash
systemctl unmask hostapd
systemctl enable --now hostapd dnsmasq
```

## 5. Verification checklist

- [ ] `iw dev wlan0 info` → AP mode, channel 6
- [ ] Phone sees `OFN-0001`, connects with passphrase
- [ ] Phone gets IP in 10.0.0.10–50
- [ ] From CM5: `ping 10.0.0.x` (phone IP) works
- [ ] No internet expected through the AP — that's correct for the isolated control net

## 6. Known pitfalls

1. **RTW88 chips** (RTL8822CE etc.) can claim the interface name first — check `iw dev` and fix the config interface line.
2. If `hostapd` fails with `nl80211: could not configure driver mode`: dongle is in station mode; run `ip link set wlan0 down`, then restart hostapd.
3. Power saving on USB dongles causes latency spikes: `iw dev wlan0 set power_save off`.
4. MT7921AU gets warm under AP load — normal, but keep it off the RF module bays.
5. Country code must be set (RU) or the driver restricts channels to 1–11 in a wrong way and hostapd can fail on channel 12/13.

## 7. When it becomes the real service

The working config moves into `ofn-ctl-api` (Wi-Fi AP managed by software, SSID = `OFN-<id>`, pairing code = WPA passphrase set during first pairing). Until then, the manual config above is the P0 baseline.
