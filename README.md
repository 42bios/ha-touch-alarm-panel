# ESPHome Alarm-Bedieneinheit

## Important Notice
- Spontaneous side/fun project - an idea that came up on a whim, not a
  serious product build. Notes here are ongoing considerations, not a
  build guide.
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

## Enclosure ideas / considerations
Not a build plan, just where the thinking is at:

- Idea: mount it behind a **JUNG LS 990** single-gang cover frame, in a
  standard flush-mount ("Kaiser") box, with a custom face plate replacing
  the switch insert — plate cut to shape, 12 touch symbols laser-engraved
  rather than cut through.
- Touch: thinking **MPR121** (I2C) for 12 capacitive channels instead of
  mechanical buttons — no holes needed for digits/symbols, just copper pads
  behind the face plate. Haven't worked out exact pad shapes/positions or
  the carrier PCB yet.
- Backlight idea: 2 RGB LEDs shining edge-on into the face plate instead of
  individual LEDs per zone. Laser-cut acrylic edges are polished enough for
  total internal reflection, and the engraved symbols would scatter that
  light out — so the numbers/ARM/DISARM could glow from inside while the
  rest of the plate stays dark. Color would carry the status:
  - white (brief) = key press feedback
  - blue (brief) = armed
  - green (brief) = disarmed
  - red (blinking) = alarm
  - all driven from Home Assistant via `light.turn_on` with `rgb_color`
- Open question to keep in mind: the LED-facing edge would need to stay
  clear of the LS990 frame/mounting bracket, or no light gets in.
- Open question to keep in mind: the ESP32-POE-ISO board itself (~65x51mm)
  plus its RJ45 jack probably won't fit inside a normal round flush-mount
  box together with the touch carrier PCB — might need a deeper/rectangular
  back box, or keep the ESP32 module elsewhere and only run touch/LED wiring
  through the cover plate.

## Open Questions / Roadmap
- **OLED**: still an open idea — not decided whether one even fits next to
  the touch grid in a single LS990 gang, and if so which size/model would
  actually fit without crowding the 12 touch zones.
- **Touch zones**: current 3x4 MPR121 layout (digits + ARM/DISARM) is just
  a starting point, not final — could still change once the enclosure and
  any OLED placement are settled.
- **Custom PCB**: combining MPR121, LED wiring and (if it fits) the OLED
  onto one small board matching the final panel layout. Natural next step
  once the above settles, but no rush — whenever there's time for it.
- **Home Assistant package**: the actual alarm logic (PIN handling,
  arm/disarm, door contacts, siren, lockout) — not started yet.

## Notes
- This setup is a robust baseline, but pin mapping is project-specific.
- On ESP32 Ethernet setups, RMII pins are reserved and cannot be reused freely.
- Strapping pins (for example GPIO0/GPIO2/GPIO15) may affect boot behavior.
- If boot/flash is unstable, review and remap these pins first.
