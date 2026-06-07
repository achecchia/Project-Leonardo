# Current Config Notes

This document records important Klipper/Mainsail/Moonraker configuration changes and calibration values that were discovered after the major hotend/cooling/bed upgrade phase.

It is intended as a human-readable summary of the live config state. The actual active configuration files remain under `printer_data/config/`.

---

## Current Baseline Summary

The current repository config reflects Leonardo E3 after the major hardware commissioning work:

- CHC hotend installed and configured
- Mellow all-metal heatbreak installed
- DLC nozzle installed
- Dual part-cooling fans installed
- Orion 24V hotend fan installed
- Silicone bed mounts installed
- Extrusion path alignment issue fixed
- Z-offset calibrated
- Bed mesh regenerated
- Extruder rotation distance calibration completed
- Pressure advance updated from the earlier conservative baseline

---

## `printer.cfg` Important Notes

### Config Structure / Ownership

`printer.cfg` is now treated as the hardware and machine-definition file.

Current includes:

```ini
[include mainsail.cfg]
[include mainsail_overrides.cfg]
[include shell_command.cfg]
[include macros.cfg]
#[include adxlmcu.cfg]
```

Important ownership notes:

- `mainsail.cfg` owns Mainsail client helpers like `[display_status]`, `PAUSE`, `RESUME`, and `CANCEL_PRINT`.
- `mainsail_overrides.cfg` owns Project Leonardo-specific `_CLIENT_VARIABLE` settings.
- `shell_command.cfg` owns Git/Mainsail shell helper macros.
- `macros.cfg` owns print start/end, mesh helpers, prime line, PID helpers, and parking/testing macros.
- `printer.cfg` owns hardware, motion, heaters, probe, bed mesh definition, display pins, and `[exclude_object]`.
- `adxlmcu.cfg` remains commented out for normal printing and should only be enabled while using the accelerometer.

---

### Headless Printer Note

Leonardo E3 is normally operated as a headless printer.

The stock Ender 3 screen has been removed from normal operation. However, the current `printer.cfg` still contains `[display]`, `[output_pin beeper]`, and EXP1 board pin aliases.

This is not currently a problem as long as Klipper loads correctly, but it is worth remembering:

- The printer is controlled through Mainsail/Moonraker.
- Stock-screen workflows should not be assumed.
- The screen can be temporarily reinstalled for troubleshooting if it ever becomes useful.
- Future config cleanup could remove or comment LCD/display sections if they become noisy or unnecessary.

---

### Motion Limits

Current `[printer]` motion limits:

```ini
max_velocity: 300
max_accel: 3000
max_z_velocity: 5
max_z_accel: 100
```

Current interpretation:

- These are conservative/reasonable for a tuned Ender 3 bedslinger baseline.
- Do not raise them until extrusion, retraction, pressure advance, and print-quality behavior are stable.
- Future speed tuning should be deliberate and tested.

---

### Axis Travel / Homing

Current axis limits:

```ini
[stepper_x]
position_max: 235

[stepper_y]
position_max: 235

[stepper_z]
position_max: 250
position_min: -6
```

Current safe Z home:

```ini
[safe_z_home]
home_xy_position: 117.5, 145.5
z_hop: 10
```

Notes:

- Safe Z home accounts for the BLTouch Y offset.
- Final usable Z should still be checked later because the top-mounted filament runout sensor may affect tall-print clearance.

---

### Extruder / Hotend

Current `[extruder]` important values:

```ini
rotation_distance: 7.77
sensor_type: Generic 3950
min_temp: 0
max_temp: 300
pressure_advance: 0.08 #0.04 pre calibration PLA
max_extrude_cross_section: 64
max_power: 0.5
```

Important notes:

- `sensor_type: Generic 3950` matches the Trianglelab CHC Pro kit's NTC 100K B3950 thermistor.
- `max_temp: 300` is currently used for the CHC hotend setup.
- `max_power: 0.5` remains in place as a conservative limit for the 115W CHC heater.
- `rotation_distance: 7.77` reflects the current calibrated extruder value after the hotend upgrade phase.
- `pressure_advance` is now `0.08`, replacing the older conservative `0.04` baseline for PLA.
- `max_extrude_cross_section: 64` supports purge/prime behavior but should not be interpreted as permission to run unvalidated high-flow print settings.

Saved hotend PID values are currently in `SAVE_CONFIG`:

```ini
pid_kp = 15.051
pid_ki = 1.320
pid_kd = 42.896
```

Notes:

- These values are tied to the installed CHC hotend/heater/thermistor configuration.
- Re-run hotend PID if heater hardware, thermistor, fan ducting, or `max_power` changes.

---

### Bed Heater / Bed Mesh

Current bed heater config:

```ini
[heater_bed]
sensor_type: EPCOS 100K B57560G104F
min_temp: 0
max_temp: 130
```

Saved bed PID values are currently in `SAVE_CONFIG`:

```ini
pid_kp = 69.817
pid_ki = 1.178
pid_kd = 1034.169
```

Current bed mesh config:

```ini
[bed_mesh]
speed: 150
horizontal_move_z: 7
mesh_min: 10, 10
mesh_max: 220, 200
probe_count: 5,5
mesh_pps: 2,2
fade_start: 1
fade_end: 10
fade_target: 0
```

Current mesh policy:

- Print starts load the saved mesh profile named `default` if it exists.
- Print starts do not automatically run `BED_MESH_CALIBRATE`.
- Mesh creation is manual through `CREATE_BED_MESH` or `G29`.

Saved default bed mesh currently exists in `SAVE_CONFIG`.

---

### BLTouch / Z Offset

Current BLTouch config:

```ini
[bltouch]
sensor_pin: ^PA7
control_pin: PB0
x_offset: 0
y_offset: -28
stow_on_each_sample: True
probe_with_touch_mode: False
pin_up_touch_mode_reports_triggered: False
```

Current saved Z offset:

```ini
z_offset = 2.315
```

Notes:

- This Z offset belongs to the current CHC hotend / heatbreak / nozzle / duct / BLTouch arrangement.
- Recalibrate Z offset if nozzle, hotend mount, probe mount, duct, or bed surface changes.

---

### Screws Tilt Adjust

Current `screws_tilt_adjust` is configured for four bed screws:

```ini
screw_thread: CW-M4
```

Known commissioning result after silicone bed mount install:

```text
All screws tilt results were approximately 0:01.
```

This is excellent and should be treated as the current mechanical bed baseline.

---

### Part Cooling Fan

Current part cooling config:

```ini
[fan]
pin: PA0
```

Notes:

- Dual 24V part-cooling fans are installed for the Ductinator setup.
- Both fans are controlled together by the standard Klipper `[fan]` output.
- Fan behavior should be verified after any duct, wiring, or connector work.

---

### Input Shaper

Current saved input shaper values remain in `SAVE_CONFIG`:

```ini
shaper_type_y = mzv
shaper_freq_y = 33.4
shaper_type_x = mzv
shaper_freq_x = 74.0
```

Notes:

- These values are believable for the current Ender 3 bedslinger setup.
- They may predate the final CHC/duct/fan/toolhead configuration.
- Revalidate only if ringing/ghosting appears, the toolhead mass changed significantly, or speed/acceleration tuning becomes a priority.

---

## `macros.cfg` Important Notes

### Print Start / End Strategy

Current `PRINT_START` strategy:

- Absolute positioning
- Start bed heating
- Warm nozzle to 150°C before homing to reduce ooze while staying safer for probing
- Wait for bed temperature
- Home all axes
- Load saved bed mesh profile `default` if available
- Heat nozzle to final print temperature
- Run `PRIME_LINE`
- Lift Z

Important policy:

```text
No automatic bed mesh during PRINT_START.
Saved mesh is loaded instead.
```

This is intentional and keeps print starts faster and more predictable.

Current `PRINT_END` strategy:

- Finish queued moves
- Reset extruder position
- Relative retract
- Move away from the print
- Park center rear
- Turn part-cooling fan off
- Turn heaters off
- Disable X/Y/E motors while keeping Z enabled

---

### Bed Mesh Macros

Current manual mesh tools:

```text
CREATE_BED_MESH
_MESH_SAVE_AND_RESTART
_MESH_CANCEL_SAVE
G29
```

`CREATE_BED_MESH` supports heat soak and a Mainsail save/cancel prompt.

Recommended use:

```gcode
CREATE_BED_MESH TEMP=60 SOAK=300
```

Then use the prompt to save and restart if the mesh is accepted.

---

### Utility / Test Macros

Useful current macros include:

```text
HOME_IF_NEEDED
PARK_CENTER
PARK_FRONT
PARK_REAR
PARK_CENTER_REAR
HEAT_SOAK_BED
PID_HOTEND
PID_BED
TEST_PROBE
TEST_SPEED_SAFE
TEST_SPEED
```

Notes:

- `TEST_SPEED_SAFE` is a conservative movement sanity check.
- `TEST_SPEED` is a more aggressive speed/acceleration test and should be used deliberately.
- `PID_HOTEND` and `PID_BED` are convenience wrappers and still require `SAVE_CONFIG` after successful PID calibration.

---

## `mainsail_overrides.cfg` Important Notes

This file exists to override Mainsail client variables without editing the managed/read-only `mainsail.cfg`.

Current `_CLIENT_VARIABLE` behavior:

```ini
variable_use_custom_pos   : True
variable_custom_park_x    : 117.5
variable_custom_park_y    : 220.0
variable_custom_park_dz   : 5.0
variable_retract          : 1.0
variable_cancel_retract   : 3.0
variable_speed_retract    : 35.0
variable_unretract        : 1.0
variable_speed_unretract  : 35.0
variable_speed_hop        : 15.0
variable_speed_move       : 100.0
variable_park_at_cancel   : True
variable_park_at_cancel_x : 117.5
variable_park_at_cancel_y : 220.0
```

Important ownership rule:

- `mainsail.cfg` owns `PAUSE`, `RESUME`, `CANCEL_PRINT`, `[pause_resume]`, `[display_status]`, and `[respond]`.
- `mainsail_overrides.cfg` only defines `_CLIENT_VARIABLE`.
- `macros.cfg` should not redefine `PAUSE`, `RESUME`, `CANCEL_PRINT`, or `[pause_resume]`.

---

## `shell_command.cfg` Important Notes

Current Git helper macros:

```text
UPDATE_GIT
BACKUP_CONFIGS
PULL_FROM_GIT
CHECK_GIT_HELPERS
```

Important behavior:

- `UPDATE_GIT` pushes current Klipper configs to GitHub using Klipper-Backup.
- `BACKUP_CONFIGS` is a friendly alias for `UPDATE_GIT`.
- `PULL_FROM_GIT` pulls GitHub changes into the Pi's local backup repo without pushing.
- `CHECK_GIT_HELPERS` verifies expected backup paths on the Pi.

The shell commands now use safer `bash -lc` wrappers and check expected paths before running.

Current workflow:

```text
If GitHub/docs changed:
1. Press PULL_FROM_GIT
2. Press UPDATE_GIT / BACKUP_CONFIGS only after pull succeeds

If only local printer config changed:
1. Press UPDATE_GIT / BACKUP_CONFIGS
```

---

## `adxlmcu.cfg` Important Notes

`adxlmcu.cfg` is temporary accelerometer support for input-shaper testing.

Normal printing:

```ini
#[include adxlmcu.cfg]
```

Input-shaper testing:

```ini
[include adxlmcu.cfg]
```

Important notes:

- Keep it commented out during normal printing.
- If the ADXL MCU is unplugged while included, Klipper may fail to connect.
- Use it only while running accelerometer testing, then comment it out again.

Current ADXL MCU serial:

```ini
serial: /dev/serial/by-id/usb-Anchor_Rampon-if00
```

Current resonance test point:

```ini
117.5,117.5,20
```

---

## `moonraker.conf` Important Notes

Current Moonraker config includes:

- Server listening on `0.0.0.0:7125`
- File manager object processing enabled
- Trusted LAN/client networks
- OctoPrint compatibility enabled
- Mainsail and OctoEverywhere announcements
- OctoEverywhere system config included

Important current policy:

```text
Do not expose Moonraker directly to the public internet.
Use UniFi VPN for higher-risk admin tasks.
OctoEverywhere is for cloud access, notifications, and monitoring.
```

Power device / Tasmota printer power control remains intentionally disabled.

---

## Config Items Worth Watching Later

### LCD Sections on a Headless Printer

The current config still includes `[display]` and `[output_pin beeper]` even though the stock screen is removed.

Leave alone if Klipper is happy. Consider cleanup later only if:

- Klipper complains about display pins
- The screen will never be reinstalled
- The config is being simplified during a dedicated cleanup pass

---

### `max_power: 0.5` on CHC Hotend

The CHC heater is currently limited to 50% power.

This was intentionally conservative for the 115W heater.

Consider raising only if:

- heat-up speed is unacceptably slow
- PID behavior is stable
- wiring and power delivery are verified
- thermal behavior remains controlled

Do not raise casually.

---

### Pressure Advance

`pressure_advance` is currently `0.08` with an inline note showing `0.04` as the pre-calibration PLA baseline.

Treat `0.08` as the current calibrated value for the tested filament/profile conditions, not as a universal value for every material.

---

### Bed Mesh

A saved default mesh currently exists and is loaded at print start.

Recreate mesh after:

- changing bed surface
- moving/removing the BLTouch
- changing hotend/nozzle height
- disturbing the bed mounts
- major mechanical work

---

## Current Known-Good State

As of this config snapshot, the project should be considered past the first major commissioning stage:

```text
Hotend installed and PID tuned
Extrusion verified after alignment fix
Silicone bed mounts installed
Screws tilt leveled to approximately 0:01
Z offset saved
Fresh default bed mesh saved
First layer validated
Extruder rotation distance calibrated
Current pressure advance set to 0.08
```

Next likely tuning/documentation phases:

```text
Retraction validation
Temperature/material profile validation
Pressure advance confirmation across real prints
Max flow testing
Speed/acceleration tuning
Optional input-shaper revalidation
OrcaSlicer profile cleanup
```
