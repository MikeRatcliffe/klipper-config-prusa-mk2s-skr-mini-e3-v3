# klipper-config-prusa-mk2s-skr-mini-e3-v3

Klipper config files for the Prusa MK2 family running on a BigTreeTech SKR Mini e3 v3.0 with a Revo nozzle. The default profile now targets the MK2.5S upgrade, which replaces the original PINDA probe with the temperature-compensated PINDA 2 sensor and relies on standard bed mesh leveling instead of the older XYZ calibration routine.

## MK2.5S upgrade highlights

* **PINDA 2 thermistor support.** The `temperature_sensor pinda` section reads the built-in thermistor on analog pin PA1 so Klipper can account for probe temperature drift. The value is exposed on the status screen and can be queried from the console using standard temperature commands.
* **`PINDA_PREHEAT` macro.** Use `PINDA_PREHEAT` before a mesh or first-layer calibration to warm the bed and wait until the probe temperature is stable (default 35 °C). Optional parameters allow you to override the bed (`B`), nozzle (`S` or `NOZZLE`), target (`T`), and hysteresis (`H`) values.
* **Bed mesh leveling.** The included `G80`/`G81` macros provide the MK2.5S-style mesh leveling workflow that supersedes the retired XYZ calibration from the original MK2 firmware.
* **Live Adjust Z during a print.** The LCD `Live adjust Z` entry now proxies Prusa's firmware behaviour by driving a dedicated `LIVE_Z` macro. Adjustments immediately move the nozzle, are compatible with `M290` commands from OctoPrint or slicer start G-code, and persist until you either apply them to the probe with `Z_OFFSET_APPLY_PROBE` or reset them with `LIVE_Z RESET=1` (invoked automatically on `CANCEL_PRINT`).
* **12 V E3D Revo hotend ready.** The stock extruder configuration now targets the Semitec 104NT thermistor bundled with the Revo heater assembly, so temperatures and safety checks line up with the drop-in 0.4 mm Revo upgrade. Run a PID tune after installation to dial in your specific heater cartridge.

Please note that you could need to adjust the probe offsets slightly since the values from the Original Prusa Firmware were not well centered in my setup

Pressure advance values I found on my system (to be used in Filament-->Custom GCode--> Start GCODE):

0.4mm Nozzle:

  ASA (including Z_OFFSET for the PINDA PROBE respect to PLA):

    SET_GCODE_OFFSET Z_ADJUST=-0.09 MOVE=1
    SET_PRESSURE_ADVANCE ADVANCE=0.078

  ABS (including Z_OFFSET for the PINDA PROBE respect to PLA):

    SET_GCODE_OFFSET Z_ADJUST=-0.09 MOVE=1
    SET_PRESSURE_ADVANCE ADVANCE=0.105

  PETG (including Z_OFFSET for the PINDA PROBE respect to PLA):

    SET_GCODE_OFFSET Z_ADJUST=-0.09 MOVE=1
    SET_PRESSURE_ADVANCE ADVANCE=0.1

  PLA:

    SET_PRESSURE_ADVANCE ADVANCE=0.0775

Optional Features (added 09/09/2022):

to be enabled by uncommenting (deleting the #) in the file printer.cfg

`#[include config/custom/menu_autoload.cfg]` --> creates autoload/unload filament entries in the Preheat menu (macro to automatically heat to a certain temperature, load/unload the filament, then cooldown)

`#[include config/custom/macro_cold_pull.cfg]` --> creates automatical Coldpull entry in the Preheat Menu (preheat to selectable temperature [default 85°C for PLA], then automatically coldpull using the motor, no hand pull required, it works very well for PLA at 85 °C, beep at the end to alert the user [remember to pull the lever on the extruder to extract the filament after the automatic coldpull])

`#[include config/custom/filament_diameter.cfg]` --> enable the "correct filament diameter" feature in the Tune menu, to change the filament diameter during printing, use the

    M404 W{filament_diameter[0]};

command in the start GCODE to let Klipper know the filament diameter initially used in the slicer, then you can change the filament diameter on the flight while printing (for example if you use a different spool of filament)

`#[include config/accelerometer.cfg]` --> config for Raspberry Pi with adxl345 accelerometer for resonance testing (input shaper calibration)
