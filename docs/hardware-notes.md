# Hardware Notes

This file captures hardware-specific details that are easy to forget but important for future troubleshooting.

---

## Pancake Extruder Stepper Wiring

Status: Completed and working.

Related upgrade:

- BMG-style direct-drive conversion
- Pancake extruder stepper motor
- Creality 4.2.2 mainboard

### Why this note matters

The pancake extruder motor did not work correctly with the first assumed wire order. During testing, the motor made a thump/noise but did not rotate correctly. That behavior pointed to the stepper phases being wired incorrectly for the Creality 4.2.2 E-stepper port.

The motor has two coil/phase pairs:

```text
Phase A: Black + Green
Phase B: Red + Blue
```

The Creality 4.2.2 E-stepper port expects the phases grouped together as an inline layout:

```text
Pin 1: A+
Pin 2: A-
Pin 3: B+
Pin 4: B-
```

### Final working wire order

Looking at the connector from left to right, the final working order is:

```text
Black, Green, Red, Blue
```

Mapped electrically:

```text
Pin 1: Black  -> Phase A
Pin 2: Green  -> Phase A
Pin 3: Red    -> Phase B
Pin 4: Blue   -> Phase B
```

### Failed / problematic behavior to remember

The wrong wiring produced symptoms like:

- A single thump
- Motor noise without useful movement
- No smooth extruder motion
- Phase mismatch behavior during `STEPPER_BUZZ STEPPER=extruder`

Do not blindly swap stepper wires while powered. Power down before moving pins.

### Validation command

Use Klipper's stepper test command after verifying the connector wiring:

```gcode
STEPPER_BUZZ STEPPER=extruder
```

Expected behavior:

- The motor should physically pulse/jiggle cleanly.
- A thump or hum without movement suggests a phase/wiring issue.

### Notes for future work

- The Creality 4.2.2 board uses physical Vref potentiometers for stepper current.
- Do not add `[tmc2209 extruder]` or software `run_current` settings unless the electronics are upgraded to a board with UART-controlled stepper drivers.
- If the extruder runs backward after wiring is correct, fix direction in Klipper with the `dir_pin` invert flag rather than rewiring the phases.
- After the final hotend is installed, re-check extrusion behavior under heat and load.

---

## BLTouch Wiring Note

Status: Completed and working.

A previous Z-homing crash was resolved by correcting BLTouch wiring polarity at the mainboard.

Important note:

```text
White = signal
Black = ground
```

The BLTouch is currently working and should not be rewired unless there is a confirmed fault.

---

## Safety Reminder

When working on stepper wiring, BLTouch wiring, heater wiring, or fan wiring:

1. Power the printer off.
2. Unplug AC power if working inside the electronics bay.
3. Confirm wiring before powering back on.
4. Test one subsystem at a time.
5. Back up config before and after any related firmware/config change.
