Clean Klipper cfg set for Anthony's Ender 3 / Creality 4.2.2

Files included:
- printer.cfg       Cleaned main config based on your current working config.
- macros.cfg        Simplified macro file. Removes generated Voron/Klicky/QGL/adaptive macro complexity.
- mainsail.cfg      Your uploaded Mainsail config, preserved as-is.
- shell_command.cfg Preserved, but not included by default in printer.cfg.
- adxlmcu.cfg       Preserved, but not included by default in printer.cfg.

Before replacing files on the Pi:
1. Back up your current config folder.
2. Upload these files to ~/printer_data/config.
3. Restart Klipper.
4. If Klipper errors, restore your backup and copy the error text.

Important notes:
- TMC/UART sections are intentionally not added. Your Creality 4.2.2 board is treated as standalone Vref-adjusted drivers.
- CHC hotend upgrade: verify thermistor type before heating. If it is not EPCOS 100K B57560G104F, change sensor_type before first heat-up.
- After CHC/heatbreak/nozzle install, retune PID, pressure advance, retraction, extrusion, and max flow.

Recommended slicer start gcode:
M117
M190 S0
M109 S0
PRINT_START EXTRUDER_TEMP=[first_layer_temperature] BED_TEMP=[first_layer_bed_temperature]

Recommended slicer end gcode:
PRINT_END

To force mesh calibration at the start of a print instead of loading saved mesh:
PRINT_START EXTRUDER_TEMP=200 BED_TEMP=60 LOAD_MESH=0
