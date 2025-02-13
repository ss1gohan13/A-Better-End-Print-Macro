# A-Better-End-Print-Macro

This was loosly based off of the idea from jontek2's [A better Print Start Macro](https://github.com/jontek2/A-better-print_start-macro)

<b>Please note:</b> [julianschill's Klipper LED effects](https://github.com/julianschill/klipper-led_effect) are also listed within this macro. They have been commented out by default. If you have LEDs installed, and the plugin installed, feel free to uncomment them. 

In addition to this, I had included delayed gcodes for a nevermore to remain running (if installed) for 5 mins to help scrub the air. As well as included a delayed gcode for LED status to update after 5 min to reflect the printer part status. 
These have been commented out by default to avoid any errors for someone that does not have these installed. If you do have a nevermore, or LEDs, installed please feel free to uncomment these options. 

The idea behind this end print macro:
- have the printer detect is maximum z boundries
- apply a percentage threashold for the Z axis to raise to if the Z axis gets with 20% of the maximum heigh threshold.
- if z reached the 20% boundry, limits the z lift at end of the print and also then lowers the toolhead 10mm so next unknown home doesn't slam into top of Z

The End Print macro should work for all styles of printers (cartesian, corexz, corexy, delta, ect). 

## :warning: Required changes in your slicer :warning:
You need to replace your "End G-code" in your slicer to be able to send data from slicer to this macro. Click on the slicer you use below and read the instructions.

<details>
<summary>SuperSlicer</summary>
In Superslicer go to "Printer settings" -> "Custom g-code" -> "End G-code" and update it to:

```
END_PRINT
```
</details>
<details>
<summary>OrcaSlicer</summary>
In OrcaSlicer go to "Printer settings" -> "Machine End G-code" and update it to:

```
END_PRINT
```
</details>
<details>
<summary>PrusaSlicer</summary>

In PrusaSlicer go to "Printer settings" -> "Custom g-code" -> "End G-code" and update it to:

```
END_PRINT
```
</details>
<details>
<summary>Cura</summary>

In Cura go to "Settings" -> "Printer" -> "Manage printers" -> "Machine settings" -> "End G-code" and update it to:

```
END_PRINT
```
</details>

# END_PRINT Macro

<details>

```
#####################################################################
#   A better end_print macro
#####################################################################

[gcode_macro END_PRINT]
gcode:
  # Get Boundaries
  {% set max_x = printer.configfile.config["stepper_x"]["position_max"]|float %}
  {% set max_y = printer.configfile.config["stepper_y"]["position_max"]|float %}
  {% set max_z = printer.configfile.config["stepper_z"]["position_max"]|float %}
  {% set min_x = printer.configfile.config["stepper_x"]["position_endstop"]|float %}

  # Check end position to determine safe directions to move
  {% if printer.toolhead.position.x < (max_x - 20) %}
      {% set x_safe = 20.0 %}
    {% else %}
      {% set x_safe = -20.0 %}
    {% endif %}

  {% if printer.toolhead.position.y < (max_y - 20) %}
      {% set y_safe = 20.0 %}
    {% else %}
      {% set y_safe = -20.0 %}
    {% endif %}

  {% if printer.toolhead.position.z < (max_z - 2) %}
      {% set z_safe = 2.0 %}
    {% else %}
      {% set z_safe = max_z - printer.toolhead.position.z %}
    {% endif %}

  # Commence END_PRINT
  # STATUS_COOLING
  M400                                                           # wait for buffer to clear
  G92 E0                                                         # zero the extruder
  G1 E-4.0 F3600                                                 # retract
  G91                                                            # relative positioning
  G0 Z{z_safe} F3600                                             # move nozzle up
  M104 S0                                                        # turn off hotend
  M140 S0                                                        # turn off bed
  M106 S0                                                        # turn off fan
  M107                                                           # turn off part cooling fan
  G90                                                            # absolute positioning
  G1 X{min_x} Y{max_y} F2000                                     # move nozzle and present

  # Safe Z-drop if near maximum height (after parking)
  {% if printer.toolhead.position.z > (max_z - 20) %}
    G91                                                          # relative positioning
    G1 Z-10 F600                                                 # drop 10mm if near the top
    G90                                                          # back to absolute
  {% endif %}

  # Conditional check for nevermore pin
  {% if 'nevermore' in printer.configfile.settings %}
    SET_PIN PIN=nevermore VALUE=1                                 # Keep the nevermore running
    UPDATE_DELAYED_GCODE ID=check_nevermore_status DURATION=300   # Schedule to check the nevermore status after 5 minutes
  {% endif %}

  M117 Print finished!!                                          # Displays info on LCD
  # STATUS_PART_READY
  UPDATE_DELAYED_GCODE ID=set_ready_status DURATION=60            # Schedule ready status
  # UPDATE_DELAYED_GCODE ID=reset_printer_status DURATION=65
  # M84                                                           # Disable motors (currently disabled to allow idle timeout)
```  
```
[delayed_gcode set_ready_status]
gcode:
  STATUS_READY                                       # # Sets SB-LEDs to ready-mode
```
```
[delayed_gcode check_nevermore_status]
gcode:
  {% if 'nevermore' in printer.configfile.settings and printer.print_stats.state == 'idle' %}
    SET_PIN PIN=nevermore VALUE=0                                 # Turns off the nevermore if the printer is idle
  {% else %}
    UPDATE_DELAYED_GCODE ID=check_nevermore_status DURATION=60    # Recheck after 1 minute if not idle
  {% endif %}
```

</details>

## Interested in more macros?

Hungry for more macromania? Make sure to check out these awesome links.

- [A Better Start Print Macro-SV08 Edition](https://github.com/ss1gohan13/A-better-print_start-macro-SV08)
- [More replacement SV08 Macros](https://github.com/ss1gohan13/SV08-Replacement-Macros)
- [Mjonuschat optimized bed leveling macros for](https://mjonuschat.github.io/voron-mods/docs/guides/optimized-bed-leveling-macros/)
- [Ellis Useful Macros](https://ellis3dp.com/Print-Tuning-Guide/articles/index_useful_macros.html)
- [Voron Klipper Macros](https://github.com/The-Conglomerate/Voron-Klipper-Common/)
- [KAMP](https://github.com/kyleisah/Klipper-Adaptive-Meshing-Purging)
- [Klipper Shake&Tune plugin](https://github.com/Frix-x/klippain-shaketune)


## Credits

A big thank you to the Voron Communuity for helping make this macro. 

## Feedback

If you have feedback please open an issue on github.
