# ESPHome Alarm-Bedieneinheit

## Important Notice
- This project was created with the help of AI-assisted development.
- No warranty is provided for correctness, safety, or production readiness.
- Use at your own risk.

## Architecture
This ESP32 device is a **pure input/output panel (HMI)** — it has **no alarm
logic on board**. All decision-making (PIN validation, arm/disarm state,
door contacts, entry delay, siren, lockout, users) lives in **Home
Assistant** as a package/automation (e.g. built on top of the
`alarm_control_panel` domain, or the Alarmo add-on).

The ESP32 only:
- reports raw touch events (digits 0-9, ARM, DISARM) as `binary_sensor`
  entities over the ESPHome native API
- exposes one RGB backlight (`light.rgb`) that Home Assistant drives to
  show status

Everything else — collecting digits into a PIN, comparing it to a code,
tracking failed attempts/lockout, arming/disarming, reacting to door
contacts elsewhere in the house, triggering a siren — is a Home Assistant
automation/package that listens to these 12 touch entities and drives the
backlight. This keeps the device dumb and swappable, and all "software"
logic (delays, users, PINs) is edited in HA instead of reflashed firmware.

## Files
- `alarmanlage.yaml`: ESPHome configuration for the panel (touch + backlight only)
- `secrets.example.yaml`: Example secrets file (currently barely needed —
  no PIN/NFC secrets live on the device anymore)

## Quick Start
1. Copy `secrets.example.yaml` to `secrets.yaml` if you want an API encryption key.
2. Verify pin mapping against your real wiring.
3. Validate: `esphome config alarmanlage.yaml`
4. Flash: `esphome run alarmanlage.yaml`
5. Add the device in Home Assistant and build the arm/disarm/PIN logic there
   (package/automation, not covered by this repo yet).

## Hardware / Enclosure
Designed to fit behind a **JUNG LS 990** single-gang cover frame, mounted
into a standard flush-mount ("Kaiser") box. The face plate is cut to shape
and its 12 touch symbols are **laser-engraved** (not cut through), replacing
the switch insert.

- **Touch input — 1 MPR121** (I2C, address `0x5A`) reads 12 capacitive touch
  channels: no mechanical buttons, no holes for digits/symbols. Each channel
  is wired to a small copper pad on a carrier PCB mounted directly behind
  the (non-conductive) face plate.
- **Status backlight — 2 RGB LEDs, edge-lit into the face plate** (wired in
  parallel, driven from 3 ESP32 PWM pins). Laser-cut acrylic edges come out
  polished enough for total internal reflection, and the laser-engraved
  symbols scatter that light preferentially — so the numbers/ARM/DISARM glow
  from inside while the rest of the plate stays dark. Color conveys state:
  - White (brief) = key press feedback
  - Blue (brief) = armed successfully
  - Green (brief) = disarmed successfully
  - Red (blinking) = alarm triggered
  - All driven from Home Assistant via `light.turn_on` with `rgb_color`.
- **Sealed face**: only the 12 touch symbols are engraved and the LED edge
  is exposed — no through-holes anywhere else, better sealed against dust/
  moisture. Touch sensitivity works fine through 2-3mm acrylic/PMMA (adjust
  the MPR121 touch threshold in the ESPHome config if it's too sensitive/numb).
- The LED-facing edge of the plate must stay uncovered by the LS990 frame/
  mounting bracket, otherwise no light gets injected — check clearance
  before finalizing which edge the LEDs sit on.
- No OLED and no NFC reader on this panel: a single LS990 gang (~50x50mm
  usable) is too small to fit a display or an RC522 antenna alongside 12
  touch zones. Status is shown on the Home Assistant dashboard/app instead;
  NFC (if wanted later) should be its own separate panel.
- ⚠️ The ESP32-POE-ISO board itself (~65x51mm) plus its RJ45 jack does
  **not** fit inside a standard round flush-mount box together with the
  touch carrier PCB. Plan for a deep/rectangular back box, or mount the
  ESP32 module separately (e.g. in a nearby junction box) with only the
  touch wiring and LEDs going through the cover plate.

## Notes
- This setup is a robust baseline, but pin mapping is project-specific.
- On ESP32 Ethernet setups, RMII pins are reserved and cannot be reused freely.
- Strapping pins (for example GPIO0/GPIO2/GPIO15) may affect boot behavior.
- If boot/flash is unstable, review and remap these pins first.
