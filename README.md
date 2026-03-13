# klipper-config-prusa-mk2s-skr-mini-e3-v3

Klipper config files for the Prusa MK2 family with an LCD interface similar to the Prusa Original Firmware. The default profile now targets the MK2.5 upgrade, which replaces the original PINDA probe with the temperature-compensated PINDA 2 sensor and relies on standard bed mesh leveling instead of the older XYZ calibration routine.

## PrusaSlicer profile

The `config/prusaslicer/mk2s_input_shaper_0.4.ini` profile is based on Prusa's MK4 input shaper printer preset and tuned for this Klipper configuration. It keeps the MK4 motion defaults while switching to Klipper-compatible start and end G-code that leverage the bundled `PINDA_PREHEAT` and `G80` macros. Import the profile into PrusaSlicer to create G-code that matches the new input shaper settings and printer macros. Advanced MK4-specific firmware integrations (for example, network upload hooks or segmented purge routines) are not part of this conversion, so expect to adjust or disable those features if you import additional presets from Prusa.

## MK2.5 upgrade highlights

* **PINDA 2 thermistor support.** The `temperature_sensor pinda` section reads the built-in thermistor on analog pin A1 so Klipper can account for probe temperature drift. The value is exposed on the LCD status screen and can be queried from the console using standard temperature commands.
* **`PINDA_PREHEAT` macro.** Use `PINDA_PREHEAT` before a mesh or first-layer calibration to warm the bed and wait until the probe temperature is stable (default 35 °C). Optional parameters allow you to override the bed (`B`), nozzle (`S` or `NOZZLE`), target (`T`), and hysteresis (`H`) values.
* **Bed mesh leveling.** The included `G80`/`G81` macros provide the MK2.5-style mesh leveling workflow that supersedes the retired XYZ calibration from the original MK2 firmware.
* **Live Adjust Z during a print.** The LCD `Live adjust Z` entry now proxies Prusa's firmware behaviour by driving a dedicated `LIVE_Z` macro. Adjustments immediately move the nozzle, are compatible with `M290` commands from OctoPrint or slicer start G-code, and persist until you either apply them to the probe with `Z_OFFSET_APPLY_PROBE` or reset them with `LIVE_Z RESET=1` (invoked automatically on `CANCEL_PRINT`).
* **12 V E3D Revo hotend ready.** The stock extruder configuration now targets the Semitec 104NT thermistor bundled with the Revo heater assembly, so temperatures and safety checks line up with the drop-in 0.4 mm Revo upgrade. Run a PID tune after installation to dial in your specific heater cartridge.

Please notice that you could need to adjust the probe offsets slightly in config/mk25s/probe.cfg since the values from the Original Prusa Firmware were not well centered in my setup

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

`#[include config/custom/accelerometer.cfg]` --> config for Raspberry Pi with adxl345 accelerometer for resonance testing (input shaper calibration)

## LCD SD card pinout

The Mini-Rambo routes the LCD SD slot over the board's hardware SPI bus. For convenience the configuration now exposes those
signals via the `board_pins lcd_sdcard` aliases in `config/mk25s/display.cfg`.  Newer Klipper builds provide an `input_pin
lcd_sd_detect` helper for macros or menu entries to check whether a card is inserted; the definition is commented out by default
so legacy installations that lack the `input_pin` module can still load the configuration.

| Signal      | Arduino pin                  | AVR port  | Notes                                                                          |
| ----------- | ---------------------------- | --------- | ------------------------------------------------------------------------------ |
| CS          | 53                           | PB0       | Shared between Mini-Rambo 1.0 and 1.3                                          |
| SCLK        | 52                           | PB1       | Shared between Mini-Rambo 1.0 and 1.3                                          |
| MOSI        | 51                           | PB2       | Shared between Mini-Rambo 1.0 and 1.3                                          |
| MISO        | 50                           | PB3       | Shared between Mini-Rambo 1.0 and 1.3                                          |
| Card detect | 15 (Mk2S/1.3) / 72 (Mk2/1.0) | PJ0 / PH0 | Mk2S Mini-Rambo 1.3 reroutes the switch to PJ0; Mini-Rambo 1.0 keeps it on PH0 |

The bundled LCD menu continues to rely on Klipper's `virtual_sdcard` object for file browsing and printing, but it now hides the
"Print from SD" entry unless a card is physically detected on the display cable. If you experiment with micro-controller backed
SD support in the future, the aliases above save you from looking up the raw pin numbers in Prusa's firmware sources.
