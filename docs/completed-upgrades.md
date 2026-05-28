# Completed / Installed Upgrades

This document tracks upgrades and supporting services that have already been installed or configured on the Ender 3 Klipper build.

Items listed here are considered part of the current machine baseline unless otherwise noted.

---

## Hardware Upgrades

### BMG-Style Direct Drive Extruder

Status: Installed

Resource link:
https://www.aliexpress.us/item/3256805805447850.html?spm=a2g0o.order_list.order_list_main.59.b0e318020SxXM1&gatewayAdapt=glo2usa

Notes:

- Replaced the stock Bowden-style extrusion setup with a BMG-style dual-gear direct drive setup.
- Current live config uses a BMG-style `rotation_distance` baseline of `7.71`.
- Final extrusion calibration should be revisited after the hotend/cooling upgrades are complete.

---

### Pancake Extruder Stepper Motor

Status: Installed

Resource link:
https://www.amazon.com/dp/B0FHHXHHRW?ref=ppx_yo2ov_dt_b_fed_asin_title

Notes:

- Installed as part of the direct-drive conversion.
- Motor current is controlled by the physical Vref potentiometer on the Creality 4.2.2 board.
- No Klipper UART/TMC driver current control is currently active.
- Motor temperature and torque should be evaluated after the final hotend is installed and extrusion testing begins.

---

### Ductinator Cooling Duct

Status: Installed / in use for the current toolhead plan

Resource link:
https://www.thingiverse.com/thing:6939003

Notes:

- Current BLTouch offset assumptions are based around the Ductinator-style duct placement.
- Current probe offsets in the live config are:

```ini
x_offset: 0
y_offset: -28
```

- These should be left alone unless the physical probe location changes.

---

### Magnetic PEI Build Plate

Status: Installed

Resource link:
Not provided.

Notes:

- Used as the current print surface.
- Future OrcaSlicer profile should assume a PEI build surface.

---

### Top-Mounted Filament Runout Sensor

Status: Installed

Mount resource link:
https://www.printables.com/model/368594-mount-for-the-filament-sensor-for-ender-3-v2

Notes:

- Filament sensor is mounted at the top of the printer.
- This may reduce safe usable Z height depending on filament path, toolhead clearance, and cable routing.
- User is okay with reducing Z height if needed.
- Final safe Z height must be matched in both `printer.cfg` and the dedicated OrcaSlicer profile.

---

### BLTouch

Status: Installed and working

Resource link:
Not provided.

Notes:

- BLTouch is configured as the Z virtual endstop.
- The previous Z-axis homing crash was resolved by correcting the BLTouch signal/ground wiring orientation.
- Current working BLTouch-related configuration includes:

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

---

## Power / Electronics Setup

### Raspberry Pi Printer-Powered Setup

Status: Installed

Resource link:
Not provided.

Power path:

```text
Printer 24V PSU
    ↓
Buck converter stepped down to approximately 5.1V
    ↓
USB cable
    ↓
Raspberry Pi micro USB power input
```

Notes:

- The Pi is powered from the printer's 24V PSU through a buck converter.
- Power is fed into the Pi through the normal micro USB power input so the Pi's built-in fuse/protection path is retained.
- Future mainboard power-switching plans should keep the Pi powered independently of the Creality mainboard.

---

### Creality 4.2.2 Mainboard / Klipper USB Connection

Status: Installed and working

Resource link:
Not provided.

Notes:

- The Raspberry Pi communicates with the Creality 4.2.2 board over USB.
- Stepper currents are adjusted physically on the board through Vref potentiometers.
- No active UART-controlled TMC configuration is currently used in Klipper.

---

## Firmware / Software / Remote Access

### Klipper + Mainsail

Status: Installed and working

Resource link:
Not provided.

Notes:

- Klipper is running on the Raspberry Pi.
- Mainsail is the local web interface.
- Moonraker is the backend/API layer.

---

### Klipper-Backup

Status: Installed and working

Resource link:
Not provided.

Notes:

- Klipper configuration is backed up to this GitHub repository.
- Manual backup is available through the Mainsail `update_git` button.
- Manual console backup command:

```gcode
update_git
```

- Manual console backup with message:

```gcode
update_git MESSAGE="backup message here"
```

---

### G-Code Shell Command Extension

Status: Installed and working

Resource link:
Not provided.

Notes:

- Required for the Mainsail `update_git` macro/button to trigger Klipper-Backup from the UI.
- `shell_command.cfg` is now included by the active `printer.cfg`.

---

### OctoEverywhere for Klipper

Status: Installed

Resource link:
Not provided.

Notes:

- Installed for cloud access, print notifications, webcam/failure monitoring, and spaghetti detection support.
- UniFi VPN remains the preferred path for higher-risk admin tasks like SSH, config edits, and Klipper/Moonraker updates.

---

### GitHub Documentation / Project Tracking

Status: Active

Resource link:
https://github.com/achecchia/Ender-3-Klipper-Upgrade-v2

Notes:

- README is now being used as the project overview.
- `docs/punch-list.md` tracks the staged upgrade process.
- `docs/future-upgrade-ideas.md` tracks optional future ideas.
- This file tracks completed/installed upgrades.

---

## Config / Tuning Milestones Already Completed

### Bed Mesh

Status: Completed baseline mesh exists

Notes:

- A saved bed mesh exists in the live `printer.cfg` `SAVE_CONFIG` block.
- Mesh should be regenerated after silicone bed mounts and hotend/toolhead hardware changes are complete.

---

### Input Shaping

Status: Completed baseline input shaping data exists

Notes:

Current saved values in the live config:

```ini
shaper_type_y = mzv
shaper_freq_y = 33.4
shaper_type_x = mzv
shaper_freq_x = 74.0
```

- These are believable for the current Ender 3 bedslinger setup.
- Revalidate later only if the final toolhead mass changes significantly.

---

## Related Links for Parts Not Yet Completed

These links were provided during the project discussion but are not listed as completed upgrades yet.

They should remain tracked in the punch list and/or future upgrade documents until installed and validated.

### Trianglelab 115W CHC Pro Hotend Kit

Status: Pending install

https://www.aliexpress.us/item/3256804038017822.html?spm=a2g0o.order_list.order_list_main.11.b0e318020SxXM1&gatewayAdapt=glo2usa

### Mellow All-Metal Bi-Metal CR10/Crazy Heatbreak

Status: Pending install

https://www.aliexpress.us/item/3256802721411891.html?spm=a2g0o.order_list.order_list_main.23.b0e318020SxXM1&gatewayAdapt=glo2usa

### Mellow Gold DLC Hardened Steel / Copper Bimetal GHC V6 Nozzle

Status: Pending install

https://www.aliexpress.us/item/3256809178438391.html?spm=a2g0o.order_list.order_list_main.29.b0e318020SxXM1&gatewayAdapt=glo2usa

### Dual Blower Fans for Duct Setup

Status: Pending install / planned with duct setup

https://www.aliexpress.us/item/3256808842298737.html?spm=a2g0o.order_list.order_list_main.35.3c1818020NUC69&gatewayAdapt=glo2usa

### Silicone Bed Mounts / Spring Replacements

Status: Pending install

https://www.aliexpress.com/item/3256808932604349.html?spm=a2g0o.order_list.order_list_main.17.3c1818020NUC69
