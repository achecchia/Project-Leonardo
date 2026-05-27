# Ender 3 Klipper Upgrade v2

This repository documents and backs up my long-term Ender 3 Klipper upgrade project.

The goal of this machine is not to turn an Ender 3 into a completely different printer. The goal is to maintain and evolve it like a long-owned project car: reliable, well-understood, easier to service, faster than stock, and upgraded in deliberate stages.

This printer is no longer my primary machine, but it is being treated as a long-term custom platform for learning, testing, maintenance, and reliable everyday use.

---

## Project Goals

Primary goals:

- Keep the printer reliable and maintainable
- Improve performance beyond stock Ender 3 behavior
- Avoid unnecessary complexity
- Use Klipper intentionally, not randomly
- Preserve known-good configuration history
- Tune only after hardware changes are complete
- Create a dedicated OrcaSlicer profile after final hardware commissioning

Target behavior:

- Balanced speed and reliability
- Faster than stock Ender 3
- Ideally in the range of modern bedslinger performance, similar to or slightly beyond an Elegoo Neptune 4 class machine
- Stable enough to use without constant re-tuning

---

## Current Printer Platform

Base printer:

- Creality Ender 3
- Creality 4.2.2 mainboard
- Klipper firmware
- Raspberry Pi host connected to the printer mainboard over USB
- Mainsail web interface
- Moonraker backend
- GitHub-backed Klipper config history

Mainboard / stepper behavior:

- Creality 4.2.2 board
- Stepper current is adjusted physically with Vref potentiometers on the board
- No active Klipper UART/TMC driver control is currently configured
- No `[tmc2209]` or equivalent driver sections are currently active in the live config

---

## Raspberry Pi Power Setup

The Raspberry Pi is powered from the printer power system:

```text
Printer 24V PSU
    ↓
Buck converter stepped down to approximately 5.1V
    ↓
USB cable
    ↓
Raspberry Pi micro USB power input
