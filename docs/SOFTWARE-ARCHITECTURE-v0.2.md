# Software architecture v0.2 (paper design, pre-hardware)

Runs on the CM5 dev station first (D11), later on the field carrier. Linux (Raspberry Pi OS base), all services user-space, systemd-managed.

## Control link (open decision — needs user input)

The node has no wireless by design. Phone ↔ node link options:

| Option | Hardware | Pros | Cons |
|--------|----------|------|------|
| A. Wi-Fi AP on the one network module (ESP32/USB-WiFi on H2) | the MVP network module becomes Wi-Fi AP | no cables, phone stays on its own mobile data | Wi-Fi next to LoRa/GPS; module choice locks to Wi-Fi |
| B. USB-C cable phone↔node (tethering) | none extra | zero RF, powers/charges from node?, max reliability | cable only; USB-C roles/PD negotiation fiddly |
| C. BLE via ESP32 companion | ESP32 on a Class L bay | low power, modern pairing | BLE throughput low; extra module |

Recommendation: **A for MVP** (the network module we must buy anyway can be a Wi-Fi AP module; control traffic is tiny and stays on the node's own AP — phone does not lose internet if it has mobile data). B as fallback. Decide before Gate 2 freeze.

## Node services (systemd units)

| Service | Duty |
|---------|------|
| ofn-powerd | Reads INA219s, enforces power policy (shedding order USB-A → bays → never core), low-power mode on USB-C input |
| ofn-slotd | Bay enumeration: read ID EEPROM (Pi HAT format), apply current limits from descriptor, manual-config slots from /etc/ofn/slots.conf, I2C-scan hints for ready-made modules |
| ofn-modmgr | Module lifecycle: load drivers/bindings by module class, expose module registry over the control API |
| ofn-mission | Mission runner: tasks, schedules, LoRa/Meshtastic sessions, capture jobs |
| ofn-logd | Local logging + storage management + ring buffers |
| ofn-syncd | Store-and-forward: when phone connects, syncs logs/mission results |
| ofn-ctl-api | Control API endpoint (see below) — the only door for the Android app |

All state in /var/lib/ofn (plain files + sqlite). Config in /etc/ofn. No cloud dependency anywhere.

## Control API

- Transport: WebSocket over the control link (option A: on node AP, 10.0.0.1:8443, TLS with pinned cert shipped in the Android app).
- Protocol: JSON messages, request/response + events (module hotplug, power events, mission progress).
- Auth: pairing code shown by LED blink pattern / printed once; app stores node identity. No accounts, no external services.
- Scope: module list/control, mission start/stop/status, telemetry view (live INA219/bay data), log fetch, power mode, firmware/config updates.
- Security lab features run only against targets marked owned in mission config (project security rule).

## Android app (companion)

- Kotlin, connects over the control link, offline-first: plans missions, then syncs.
- Screens: Dashboard (power/telemetry), Modules, Missions, Logs, Settings.
- App is also the display the device lacks: full mission planning happens on the phone.

## Survival mode

- Trigger: power input lost from both sources? No — with no battery yet, survival mode means: phone disconnected + power OK → node runs scheduled missions autonomously, logs locally.
- With battery (post-MVP gate): input loss → ofn-powerd switches to battery profile, sheds all charging outputs, sheds bays by mission priority, core throttled.

## Repo layout (planned)

```
open-field-node/
  docs/            (this design set)
  ofn-node/        (services, Rust or Go — pick at first code milestone)
  ofn-android/     (companion app)
  ofn-fw/          (module firmware, if modules carry MCUs)
  ofn-eep/         (module descriptor templates + eepmake sources)
```

## Open questions for next paper iteration

1. Control link: A/B/C (recommend A).
2. Service language: Rust vs Go (recommend Go — faster bring-up, fine for this scale; power/safety-critical bits stay in kernel/drivers anyway).
3. Real-time telemetry rate needs (defines INA219 polling policy).
4. App distribution: F-Droid / APK sideload (no Google dependency) — confirm.
