# OFN Software Update Mechanism v0.2

How the node's software updates in the field. Fixed before P1 so the architecture is updatable from day one — retrofitting costs more.

## 1. Principles

1. **One node, one source of truth.** Updates always flow top-down: phone/operator first, node second. The node never pulls from the internet on its own initiative.
2. **Atomicity and rollback.** An update must never brick the node: there is always a working previous slot.
3. **Signature.** Only packages signed with the project key (ed25519) are accepted. This is an open-source project, but updates to a live node don't come from an anonymous forum.
4. **Any channel.** Wi-Fi AP + phone, USB-C tethering, microSD by hand. A field node is not required to have internet.

## 2. Scheme: A/B dual-slot update

The SD card is partitioned into two root filesystems (rootfs A / rootfs B) plus a shared boot partition. The active slot boots; the inactive one is staging.

1. The update package (`ofn-update-<ver>.tar`, signature in a separate file) arrives via the app or USB.
2. `ofn-syncd` verifies signature and format, unpacks into the **inactive** slot.
3. Boot flag switched → reboot → new slot.
4. First-5-minutes watchdog: if services don't come up / core unreachable, automatic rollback to the previous slot, `update.rollback` flag in the log.
5. If all is well, the previous slot stays as reserve until the next update.

microSD 64 GB has plenty of room: each slot ~8–10 GB, the rest for logs and missions.

## 3. Update levels

| Level | What | How |
|---|---|---|
| Services (ofn-*) | most often | package inside rootfs, shipped with the rootfs update |
| Configs/missions | constantly | over control API, no reboot |
| Kernel/boot | rarely | together with rootfs, same A/B mechanism |
| Module descriptors | rarely | data package over control API |

Bootloader (CM5 EEPROM) — manually, via a service mode, documented but not "over the air".

## 4. Needed for P1 (KiCad and image build)

- SD image layout: `bootfs` (FAT, 512 MB) + `rootfs_a` + `rootfs_b` + `data` (logs/missions).
- Two boot flags in `data`: `active_slot`, `boot_attempts`.
- `ofn-syncd` — already in the architecture; add signature check and slot switching.
- Project public key baked into rootfs (read-only part); private key stays with the releaser (you). A "release process" doc — for release time, not now.

## 5. Rejected options

- **OTA straight from GitHub** — field nodes often have no internet; channel trust is harder. Rejected for MVP, remains an option later (same signed package, different delivery channel).
- **In-place update without A/B** — any mid-flash failure = brick; rejected immediately.
- **SD reflash on a computer only** — fine for P0 and early development, not for the field; kept as the emergency recovery path.
