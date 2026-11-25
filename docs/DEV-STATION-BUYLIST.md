# Dev station buy list (approved: dev-kit gate)

Decision D11-dev (2026-11-25): dev station is a CM5 8 GB Lite (no-wireless) that later becomes the field core. One purchase serves both bench development and the field node.

| # | Item | What to search | Est. price (₽) | Notes |
|---|------|----------------|------------------|-------|
| 1 | Raspberry Pi CM5 8GB Lite, no wireless (CM5108000) | «Raspberry Pi CM5 8GB Lite» (AliExpress.ru) | ~17800 | Must be Lite (no eMMC) and no-wireless; has its own antenna option — we don't want it |
| 2 | Waveshare CM5-IO-BASE-A mini carrier | «Waveshare CM5-IO-BASE-A» | ~3000 | 40-pin GPIO header, HDMI, USB, Ethernet, microSD slot for Lite |
| 3 | PSU 5V/5A 27W USB-C | «блок питания Raspberry Pi 5 27W 5V 5A» | ~1500 | Same PSU family serves Pi-class boards |
| 4 | CM5 active cooler | «кулер Raspberry Pi CM5» | ~400 | Included in some Waveshare BOX kits — check before buying separately |
| 5 | microSD 64GB A1 | «microSD 64GB A1 Samsung SanDisk» | ~700 | Lite has no eMMC — system lives on SD |
| 6 | LoRa bench: SX1276/RFM95W 868 MHz module + USB-UART CH340 | «RFM95W 868», «SX1276 868», «USB UART CH340» | ~1000 | Region fixed by D13: 868 MHz; wires to GPIO header UART |
| 7 | Dupont wires + breadboard | «набор dupont макетная плата» | ~500 | For bench wiring to GPIO header |

**Total estimate: ~25–26k ₽.**

After the field carrier passes Gate 3, the CM5 moves from the Waveshare dock into our board — nothing from this list is discarded.

Purchase gate note: this is the only purchase allowed right now (dev-kit gate). Field purchases (carrier components, final power parts) stay locked behind Gates 1–4.
