# Mechanical envelope v0.2 (draft)

First cut at board size, bay layout and front panel so RF plan and enclosure work can proceed. Numbers are planning targets, not final CAD.

## Board

| Parameter | Value | Driver |
|-----------|-------|--------|
| Board outline | 160 × 100 mm | 6 bays along one long edge |
| Layers | 4 (signal/GND/GND/signal) | PCIe/USB3 impedance + RF return paths |
| Thickness | 1.6 mm | standard, PCIe edge connector OK |
| Mounting | 4 × M3 corner holes, 5 mm inset | enclosure standoffs |

## Compute area

- CM5 SODIMM socket (U1) mid-board, CPU side up, 11.5 mm stack height variant to clear bay rail below/above — final at layout.
- Keep-out: 15 mm around SODIMM for heatsink/fan solution.
- RJ45 magjack on short edge (network module slot = fixed position, see carrier schematic).

## Bay rail (Class L, 6 positions)

- IDC 2×8 header footprint: 20.3 × 5.1 mm + keying shroud.
- Bay pitch: 25 mm (module card 22 mm wide, 3 mm gap for finger clearance).
- Rail length: 6 × 25 = 150 mm + end margins 5 mm each = 160 mm — defines board long side.
- Module cards: 22 × 60 mm PCB, thickness 1.0–1.6 mm, gold fingers or soldered IDC plug — mechanical choice at module standard doc.
- Bay 1 (GPS) placed farthest from Wi-Fi module position (RF plan isolation rule).

## Front panel (one long side)

Left to right: XT30 power → USB-C in → USB-C PD out → USB-A out → 6 × SMA/IPEX pigtails (per installed module) → RJ45.

Status LED + power button on panel top edge.

## Envelope (case planning)

- Internal: 170 × 110 × 35 mm target for the full assembly with CM5 heatsink.
- Material: aluminum extrusion or printed PETG for MVP prototype; metal case = shielding bonus (then Wi-Fi needs panel antenna).
- Cooling: passive fins in aluminum case, or 30 mm fan over CM5 (fan header on carrier).

## Open points

1. SODIMM socket stack height (5.2 vs 11.5 mm) — depends on heatsink choice, decide at layout.
2. Module card retention: friction + key slot vs ejector clip — choose in module standard update.
3. XT30 on panel vs side — panel preferred (power cable strain), confirm at enclosure sketch.
4. Fan: none / 30 mm / case-extrusion fins — decide with thermal estimate (CM5 Lite at 12–15 W is passive-capable with fins).
