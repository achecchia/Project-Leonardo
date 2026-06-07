# Config Audit / Calibration Notes

This page captures important configuration and calibration changes observed in the current backed-up Klipper files after the recent calibration work.

Reviewed files:

```text
printer_data/config/printer.cfg
printer_data/config/macros.cfg
printer_data/config/shell_command.cfg
printer_data/config/mainsail_overrides.cfg
printer_data/config/adxlmcu.cfg
```

---

## Current Good State Summary

```text
CHC hotend installed and PID tuned
Generic 3950 thermistor active
Extruder rotation distance calibrated to 7.77
Z offset saved at 2.315
Silicone bed mounts installed
Screw tilt reported around 0:01
Fresh 5x5 bed mesh saved as default
PRINT_START loads saved mesh instead of auto-meshing
Part cooling is on PA0
ADXL helper config exists but is disabled for normal printing
Git helper macros are present for backup/pull workflows
```

---

## Important Printer.cfg Values

### Motion

```ini
[printer]
max_velocity: 300
max_accel: 3000
max_z_velocity: 5
max_z_accel: 100
```

Notes:

- `max_accel: 3000` is a reasonable conservative current value.
- Leave speed/accel alone until print-quality tuning or input-shaper revalidation supports changing it.

### Z Axis

```ini
[stepper_z]
position_max: 250
position_min: -6
```

Notes:

- `position_min: -6` supports probe/Z-offset calibration movement below nominal zero.
- `position_max: 250` still needs real-world clearance verification because of the top-mounted runout sensor and filament path.

### Extruder / Hotend

```ini
[extruder]
rotation_distance: 7.77
nozzle_diameter: 0.400
filament_diameter: 1.750
heater_pin: PA1
sensor_type: Generic 3950
sensor_pin: PC5
min_temp: 0
max_temp: 300
pressure_advance: 0.04
max_extrude_cross_section: 64
max_power: 0.5
```

Notes:

- `rotation_distance` is now `7.77`, replacing the older BMG-style baseline of `7.71`.
- `sensor_type: Generic 3950` matches the Trianglelab CHC Pro NTC 100K B3950 thermistor information.
- `max_power: 0.5` is still active as a conservative CHC heater limit.
- If `max_power` is raised later, PID behavior should be verified again.
- `pressure_advance: 0.04` is the current active value. Keep only if the recent tuning confirmed it; otherwise treat it as a working baseline.

Saved extruder PID:

```ini
control = pid
pid_kp = 15.051
pid_ki = 1.320
pid_kd = 42.896
```

### BLTouch / Z Offset

Current BLTouch values:

```ini
[bltouch]
sensor_pin: ^PA7
control_pin: PB0
x_offset: 0
y_offset: -28
```

Saved Z offset:

```ini
z_offset = 2.315
```

Notes:

- Re-run Z offset if the nozzle, hotend, probe mount, build plate, or bed support changes.

### Bed Mesh

Current bed mesh definition:

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

A saved `default` mesh exists in `SAVE_CONFIG`.

Saved mesh range is approximately `-0.0075` to `0.2875`, about `0.295 mm` total variation.

Notes:

- This is the current working mesh after silicone bed mounts.
- `PRINT_START` loads the saved `default` mesh instead of running a new mesh every print.

### Screws Tilt

```ini
[screws_tilt_adjust]
screw1: 31,62
screw2: 201,62
screw3: 201,235
screw4: 31,235
screw_thread: CW-M4
```

Notes:

- The current screw tilt result was reported around `0:01`, which is excellent.

### Cooling

```ini
[fan]
pin: PA0
```

Notes:

- This is the part-cooling output for the dual Ductinator fan setup.

---

## Macro Stack Notes

### PRINT_START

Current behavior:

```text
Heat bed
Preheat nozzle to 150C
Wait for bed
Home all axes
Load saved mesh profile named default if present
Heat nozzle to final temp
Run PRIME_LINE
Lift Z
```

Important policy:

```text
No automatic bed mesh during print start.
No adaptive mesh during print start.
Normal prints load saved mesh profile default.
```

### CREATE_BED_MESH

`CREATE_BED_MESH` is the manual mesh routine. It heats the bed, optionally heat-soaks, homes, runs `BED_MESH_CALIBRATE`, and shows a Mainsail save/cancel prompt.

Use this when the bed, nozzle, probe, hotend, or bed supports change.

### PRINT_END

Current end behavior:

```text
Retract 3mm
Move away
Park center rear
Turn off fan/heaters
Disable X/Y/E motors while keeping Z enabled
```

### ADXL / Input Shaper

`adxlmcu.cfg` is present but disabled in `printer.cfg`:

```ini
#[include adxlmcu.cfg]
```

Saved input-shaper values currently remain:

```ini
shaper_type_y = mzv
shaper_freq_y = 33.4
shaper_type_x = mzv
shaper_freq_x = 74.0
```

Keep ADXL disabled for normal printing and only enable it while the accelerometer MCU is connected.

---

## Git / Backup Helper Notes

`shell_command.cfg` now has:

```text
UPDATE_GIT
BACKUP_CONFIGS
PULL_FROM_GIT
CHECK_GIT_HELPERS
```

Workflow:

```text
If GitHub docs/repo files changed first:
    Press PULL_FROM_GIT first
    Then press UPDATE_GIT or BACKUP_CONFIGS if backing up config

If only local printer config changed:
    Press UPDATE_GIT or BACKUP_CONFIGS
```

---

## Open Items / Watch List

1. Runout sensor firmware validation
   - The top-mounted runout sensor is physically installed, but no active `[filament_switch_sensor]` section is currently present in `printer.cfg`.
   - `PA4` remains the likely candidate pin, but it should be verified before enabling.

2. Safe Z max validation
   - `position_max: 250` is still active.
   - Confirm real safe Z height with the top-mounted runout sensor and filament path.

3. Pressure advance
   - Current active value is `0.04`.
   - Keep it if recent tuning confirmed it, otherwise treat it as a baseline until the next PA tune.

4. Retraction and slicer profile
   - Build the dedicated OrcaSlicer profile around the final CHC hotend, Mellow heatbreak, DLC nozzle, and dual-fan behavior.

5. Max flow and speed
   - Do not increase speed/accel or raise the CHC power limit until extrusion quality and thermal stability are characterized.

6. Display section while headless
   - The stock screen is removed, but `[display]` remains configured.
   - Leave it alone unless EXP pins are needed for something else.

---

## Rule Going Forward

Future config changes should be made one at a time, verified, then backed up.
