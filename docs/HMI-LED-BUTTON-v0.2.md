# OFN Human Interface: LED & Button v0.2

No screen on the device. The whole interface is the Android app over Wi-Fi plus **one RGB LED and one button** on the top edge of the panel (position fixed in MECHANICAL-v0.2). This doc fixes their behaviour so software and schematic agree.

## 1. Hardware

- LED: one common-cathode RGB on 3 hardware PWM channels of the CM5 (GPIO12 red, GPIO13 green, GPIO19 blue; <20 mA per channel, resistors on board). WS2812B-mini rejected — it needs precise bit timing that an expander GPIO can't guarantee; plain PWM is boring and robust.
- Button: tactile on the top edge, pull-up to 3V3 + RC debounce (100n + 10k), GPIO with interrupt.

## 2. LED behaviour

"Breathing" = smooth 1 Hz pulse.

| Node state | Colour / pattern |
|---|---|
| Off / no power | dark |
| Booting | blue breathing |
| Normal operation | green, single blink every 5 s ("alive") |
| AP active, no phone connected | green breathing |
| Phone connected via API | green blink every 1 s |
| Warning (power sag, slot rebooted) | yellow, 3 flashes, back to state |
| Survival mode | yellow breathing |
| Error (power out of range, overheat, core won't start) | red, 4 Hz flashes |
| Shutting down (graceful) | blue 3 s fade to dark |
| Charging external device (USB-C/A out active) | white pulse every 3 s (additive, lowest priority) |

Rules: error pattern wins over everything; white charging indicator only during normal operation.

## 3. Button behaviour

| Action | Result |
|---|---|
| Short press (<0.5 s) | `button.short_press` event to control API; app decides (default: show status) |
| Hold 2 s | Toggle AP on/off — quick "radio silence" |
| Hold 5 s | Graceful shutdown of services and core power (soft power off) |
| Hold 10 s | Hard power-cut of the whole tree (battery-pull equivalent); "unclean shutdown" flag logged |

Hardware debounce + 30 ms software confirmation. All hold events are logged by `ofn-logd` (audit: who switched the node off and when).

## 4. "Dark" state

The node can run fully dark — LED off, button live. Toggled from the app or by 4 quick presses. Even in dark mode, errors still light up — safety beats camouflage.

## 5. For the carrier schematic (KiCad)

- GPIO12 (PWM0) → R → red anode; GPIO13 (PWM1) → green; GPIO19 (PWM1) → blue; common cathode → GND. Bit-banging is acceptable if PWM channels are needed elsewhere.
- Button: GPIO24, edge interrupt, RC 100n/10k on board.
- Both organs: 2.54 header to a top-edge sub-panel for P1; integrated into the v0.3 case panel later.
