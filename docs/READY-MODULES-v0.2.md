# Ready-made modules mapping v0.2

Market check (2026-11): which stages of our power/control design exist as cheap ready modules. Use modules for bench bring-up and field prototypes; the final board still uses bare ICs (size, cost, reliability).

| Our stage | Ready module | Price | Status |
|-----------|--------------|-------|--------|
| 5 V/6 A buck (backbone) | XL4015 5A buck module (CC/CV) | $2.5–5 | exists, ubiquitous — bench bring-up OK; note ripple/EMI, no 6 A continuous without heatsink |
| USB-C input PD trigger 12 V | PDC004 (IP2721 hardware decoy) | $1–2.5 | exists; fixed 12 V, ≤5 A; supports PD2.0/3.0 |
| Input OR-ing | none found | — | build from SS54 diodes (trivial) |
| Reverse polarity + TVS | none found | — | discrete parts, 3 components |
| Per-bay limiter 2 A | none found as module | — | TPS2553 bare IC or discrete MOSFET+sense on protoboard |
| 3.3 V rail | MP1584/SY8120 mini modules | $1–2 | exists; fine for bench |
| PD charging output (D9) | 65 W DC-input PD source modules (8–30 V in, USB-C PD 5–20 V out, PD3.1/QC) | ~$4–9 per board | exists; e.g. iHaospace-class boards — good fast-charge bench solution |
| USB-A 5 V/2.4 A out | any 5 V buck/USB charger module + PTC | $1–2 | exists; SY6280-class load switches exist bare |
| Rail monitoring | INA219 I2C module | $1.5–2.5 | exists, many vendors |
| GPIO expander EN/FAULT | MCP23017 I2C module | $1–4 | exists |

## What this changes

1. **Bench bring-up needs zero custom boards**: XL4015 + PDC004 + INA219 + MCP23017 + 65 W PD module covers the whole power tree on a breadboard with the Waveshare CM5 dock.
2. **Prototyping path**: power section can be validated module-by-module before KiCad entry — de-risks Gate 3.
3. **Only truly custom power work**: per-bay limiter (TPS2553 or discrete) and OR-ing/protection — both are simple discrete builds.
4. BOM strategy unchanged: final board uses bare ICs; modules are the dev-station substitute for a lab bench PSU.

## Verified examples (search terms)

- "XL4015 5A buck module" — CC/CV, 5–30 V out adjustable, heatsink recommended >3 A
- "PDC004 IP2721 PD decoy 12V" — fixed 12 V output
- "65W PD fast charging module DC 8-30V input" — USB-C PD3.1 source
- "CJMCU-219 INA219" / "MCU-219" — I2C power monitor
- "MCP23017 16Bit I2C IO expander"
