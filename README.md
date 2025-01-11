# A-Better-End-Print-Macro

This was loosly based off of the idea from jontek2's "A better Print Start Macro" : https://github.com/jontek2/A-better-print_start-macro

The idea behind this is to have the printer detect is maximum boundries and apply a percentage threashold for the Z axis to raise to if the Z axis gets with 20% of the maximum heigh threshold. 

The End Print macro should work for all styles of printers (cartesian, corexz, corexy, delta, ect). 

Make sure to update your slicer end gcode accordingly to use `END_PRINT`.

<b>Please note:</b> there are additional macros embedded within the End Print macro that call for LED status (https://github.com/julianschill/klipper-led_effect/blob/master/docs/LED_Effect.md).
The LED macros have been commented out by default. 

In addition to this, I had included delayed gcodes for a nevermore to remain running (if installed) for 5 mins to help scrub the air. As well as included a delayed gcode for LED status to update after 5 min to reflect the printer part status. 
These have been commented out by default to avoid any errors for someone that does not have these installed. If you do have a nevermore, or LEDs, installed please feel free to uncomment these options. 

```
[gcode_macro END_PRINT]
gcode:
  #Get Boundaries
  {% set max_x = printer.configfile.config["stepper_x"]["position_max"]|float %}
  {% set max_y = printer.configfile.config["stepper_y"]["position_max"]|float %}
  {% set max_z = printer.configfile.config["stepper_z"]["position_max"]|float %}
  {% set min_x = printer.configfile.config["stepper_x"]["position_endstop"]|float %}

  #Check end position to determine safe directions to move
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

  SET_FILAMENT_SENSOR SENSOR=filament_sensor ENABLE=0
  #Commence END_PRINT
#  STATUS_COOLING
  M400 ; wait for buffer to clear
  G92 E0 ; zero the extruder
  G1 E-4.0 F3600 ; retract
  G91 ; relative positioning
  G0 Z{z_safe} F3600 ; move nozzle up
  M104 S0 ; turn off hotend
  M140 S0 ; turn off bed
  M106 S0 ; turn off fan
  M107 ; turn off part cooling fan
  G90 ; absolute positioning
  G1 X{min_x} Y{max_y} F2000 ; move nozzle and present
#  SET_PIN PIN=nevermore VALUE=0                      # Turns off the nevermore
  #UPDATE_DELAYED_GCODE ID=turn_off_nevermore DURATION=300
  SET_DISPLAY_TEXT MSG="Print finished!!"            # Displays info
#  STATUS_PART_READY
#  UPDATE_DELAYED_GCODE ID=set_ready_status DURATION=60
```
```
[delayed_gcode set_ready_status]
gcode:
  STATUS_READY
```
```
[delayed_gcode turn_off_nevermore]
gcode:
  SET_PIN PIN=nevermore VALUE=0                      # Turns off the nevermore
```
