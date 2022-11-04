# Voxelab Aquila 3D printer initialization

This guide is how I got my Voxelab Aquila 3D printer working properly with a BLTouch auto bed leveller.

Thanks to [alexqzd](https://github.com/alexqzd/Marlin) over on Github, as the software and much of the documentation came from him.

There you can fetch alternate printer and display firmware images. The ones listed below are just the ones I use for my own printer.

## Flashing printer firmware

- On Windows, format the SDcard FAT32 4096 Byte
- Create a `firmware` directory in the root of the SD card
- Download the [image file](https://github.com/stevieb9/voxelab-aquila/raw/main/firmware/printer/BLTouch-4x4-G32.bin)
- Place the downloaded firmware binary into the `firmware` directory on the SD card
- Put the SD card in the printer, and reboot the printer
- The screen should go red indicating the flash was successful
- Remove the SD card and reboot the printer

## Flashing the UI display

- On Windows, format the SDcard FAT32 4096 Byte (the `firmware` directory must be removed)
- Download the [display firmware](https://github.com/stevieb9/voxelab-aquila/raw/main/firmware/display/DWIN_SET.zip)
zip file. Extract it, and place the entire `DWIN_SET` directory and contents onto the root of the SD card
- Remove the back of the display, insert the SD card, plug the display back in and reboot the printer
- The screen will go blue for a few seconds, then red, indicating the process is complete
- Remove the SD card, and reboot the printer

## Configuring OctoPrint (OctoPi)

- Flash an SD card using the Rasperry Pi Imager with the OctoPi firmware

- Configure the Pi

- Configure the default printer settings (all settings not shown are default)

    - General 
      - Name: Aquila
      - Model: Voxelab Aquila
  - Print bed & build volume:
    - Form factor: Rectangular
    - Origin: Lower left
    - Heated bed: True
    - Width(X): 200mm
    - Depth(Y): 200mm
    - Height(Z): 250mm
  - Add the following GCode scripts:

    - Before print jobs start:

          ;Jyers gcode
          M75 ;Start Print Job on Display
          M117 <F>{{ event.name }} ;Send Filename to Display

    - After print job completes

          ;Jyers gcode
          M77 ;Stop Print Job on Display

    - After print job is cancelled:

          ; disable motors
          M84

          ;disable all heaters
          {% snippet 'disable_hotends' %}
          {% snippet 'disable_bed' %}

          ;disable fan
          M106 S0

          ;Jyers code
          M77 ;Stop Print Job on Display

    - After print job is paused

          ;Jyers code
          M76 ;Pause Print Job on Display

    - Before print job is resumed

          ;Jyers code
          M75 ;Start Print Job on Display

## Setting Z-Axis for auto bed level

The following information was taken and slightly adapted from
[this website](https://www.webcarpenter.com/blog/162-3D-Print---How-to-calibrate-Z-offset-with-a-BLTouch-bed-leveling-probe-sensor).

Home the print head
 
Reset Z0-Offset
 
    M851 Z0

Store settings to EEPROM

    M500

Set active parameters

    M501

Display active parameters

    M503

Home the nozzle and show the Z-Axis

    G28

Move the nozzle to true 0 offset

    G1 F60 Z0

Disable soft end stops (so the print head can go below zero)

    M211 S0

Using the 'Control' feature in Octoprint, move the print head down to the table
until a piece of paper can barely move under it.

Take note of what the Z axis says on the display. Mine was `2.57`. Add a
fraction to add for the paper, so mine would then be `2.58`.

Set the Z-Axis (note the negative number)

    M851 Z -2.58

Enable soft end stops

    M211 51

Save settings to EEPROM

    M500

Set active parameters

    M501

Display current settings

    M503

Tell the printer to go home

    G28

Move the nozzle to true zero offset to see results

    G1 F60 Z0


