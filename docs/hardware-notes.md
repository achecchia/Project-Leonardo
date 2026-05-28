# Hardware Notes

This file keeps verified printer-specific hardware notes. It intentionally avoids treating earlier AI suggestions as facts unless they were confirmed on this printer.

---

## Pancake Extruder Stepper Wiring

Status: completed and working.

Related hardware:

- BMG-style direct-drive conversion
- Pancake extruder stepper motor
- Creality 4.2.2 mainboard

### Verified result

The final working wire order for the pancake extruder stepper connector is:

```text
Black, Green, Red, Blue
```

The earlier order below did not work correctly:

```text
Black, Red, Blue, Green
```

Observed failed behavior:

- thump/noise
- no smooth extruder movement
- symptoms consistent with a phase mismatch

### Coil pairs

The motor coil pairs were identified as:

```text
Phase A: Black + Green
Phase B: Red + Blue
```

Final mapping:

```text
Pin 1: Black
Pin 2: Green
Pin 3: Red
Pin 4: Blue
```

### Future note

The Creality 4.2.2 board uses physical Vref potentiometers for stepper current. Do not add software current-control sections such as `[tmc2209 extruder]` unless the board is upgraded to hardware that supports that.

---

## BLTouch Wiring

Status: completed and working.

A previous Z-homing crash was resolved by correcting the BLTouch wiring orientation.

Known working note:

```text
White = signal
Black = ground
```

Do not change the BLTouch wiring unless a real fault is confirmed.

---

## Filament Runout Sensor / DET Port

Status: installed physically; final firmware behavior still needs validation.

Earlier AI-assisted notes claimed the Creality 4.2.2 `DET` port uses:

```ini
switch_pin: PA4
```

Treat `PA4` as a candidate pin, not as fully verified, until confirmed by:

- the live working config
- a trusted pinout for the exact board revision
- direct Klipper sensor testing

Candidate config for later validation:

```ini
[filament_switch_sensor runout_sensor]
pause_on_runout: True
switch_pin: PA4
```

If the logic is backwards after testing, the pin may need inversion:

```ini
switch_pin: !PA4
```

Because the runout sensor is mounted at the top of the printer, final Z-height clearance must be checked before the slicer profile is finalized.

---

## Mainboard Enclosure

Status: conservative documentation only.

A printed mainboard enclosure may be fine mechanically, but earlier AI-assisted grounding advice should not be treated as authoritative.

Project rule:

- Preserve the stock printer safety design unless there is a measured reason to change it.
- Do not add extra grounding or bonding wires based only on AI advice.
- If anything around the PSU, enclosure, or electronics bay is changed, verify the work before powering the printer.

---

## Fan Voltage Rule

Status: ongoing rule.

Do not assume fan voltage based on size, connector style, or AI recommendation.

Before using a fan, verify:

- voltage rating
- power source voltage
- current draw
- control method

Important lesson:

A 12V fan is not a correct direct replacement for a Raspberry Pi 5V-powered fan path.
