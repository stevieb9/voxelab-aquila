# Voxelab Aquila 3D printer initialization

This guide is how I got my Voxelab Aquila 3D printer working properly with a
BLTouch auto bed leveler and remote power on/off via Octoprint using a Solid
State Relay.

Installing the BLTouch bed leveler is an exercise for the reader. Configuring it
is however part of this guide.

Thanks to [alexqzd](https://github.com/alexqzd/Marlin), as the  software and
much of the documentation came from him.

There you can fetch alternate printer and display firmware images. The ones
listed below are just the ones I use for my own printer.

## Table of contents

- [Flashing the printer firmware](#flashing-printer-firmware)
- [Flashing the display firmware](#flashing-the-ui-display)
- [Configure OctoPrint](#configuring-octoprint-octopi)
- [Display progress while printing from Octoprint](#display-progress-from-octoprint)
- [Setting Z-Axis](#setting-z-axis-for-auto-bed-level)
- [Setting Z-Axis quick guide](#setting-z-axis-quick-guide)
- [Auto level the bed - Bed Visualizer](#auto-level-the-bed-bed-visualizer)
- [Auto level the bed - Display panel](#auto-level-the-bed-display-panel)
- [Setting up Cura slicer](#setting-up-cura-slicer)
- [Using the saved bed mesh](#using-the-saved-bed-mesh-auto-level)
- [Print vertical display bracket](#print-the-vertical-display-bracket)
- [Print accessories](#print-the-other-accessories)
- [Setting up UI power toggle](#setting-up-ui-power-toggle)
- [Disable the Pi from powering the printer](#disable-pi-powering-the-printer)
- [Configure iPhone app](#configure-iphone-app)
- [PETG Cura configuration](#petg-cura-configuration)
 
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
    - Width(X): 220mm
    - Depth(Y): 220mm
    - Height(Z): 250mm
  - Add the following GCode scripts:

    - Before print jobs start:

          ;Jyers gcode
          M75 ;Start Print Job on Display
          M117 <F>{{ event.name }} ;Send Filename to Display

    - After print job completes

          G91 ;Relative positioning
          G1 E-2 F2700 ;Retract filament a bit
          G1 E-2 Z0.2 F2400 ;Retract filament more and raise Z
          G1 X5 Y5 F3000 ;Wipe out
          G1 Z10 ;Raise Z more
          G90 ;Absolute positioning

          ;disable all heaters
          M104 S0 ; hotend
          M140 S0 ; bed

          ;disable fan
          M106 S0

          ; disable all steppers
          ;M84 X Y E ; (all except Z)
          M84        ; (all)

          ;Jyers gcode
          M77 ;Stop Print Job on Display

    - After print job is cancelled:

          G91 ;Relative positioning
          G1 E-2 F2700 ;Retract filament a bit
          G1 E-2 Z0.2 F2400 ;Retract filament more and raise Z
          G1 X5 Y5 F3000 ;Wipe out
          G1 Z10 ;Raise Z more
          G90 ;Absolute positioning

          ;disable all heaters
          M104 S0 ; hotend
          M140 S0 ; bed

          ;disable fan
          M106 S0

          ; disable all steppers
          ;M84 X Y E ; (all except Z)
          M84        ; (all)

          ;Jyers code
          M77 ;Stop Print Job on Display

    - After print job is paused

          ;Jyers code
          M76 ;Pause Print Job on Display

    - Before print job is resumed

          ;Jyers code
          M75 ;Start Print Job on Display

## Display progress from Octoprint

To have the UI display the progress when printing from Octoprint, within octoprint,
install the `M73 Progress` plugin.

## Setting Z-Axis for auto bed level

The following information was taken and slightly adapted from
[this website](https://www.webcarpenter.com/blog/162-3D-Print---How-to-calibrate-Z-offset-with-a-BLTouch-bed-leveling-probe-sensor).

Home the hot end

    G28

Reset Z0-Offset

    M851 Z0

Store settings to EEPROM

    M500

Set active parameters

    M501

Display active parameters

    M503

Home the hot end and show the Z-Axis

    G28

Move the nozzle to true 0 offset

    G1 F60 Z0

Disable soft end stops (so the hot end can go below zero)

    M211 S0

Using the 'Control' feature in Octoprint, move the print head down to the table
until a piece of paper can barely move under it.

Take note of what the Z axis says on the display. Mine was `2.57`. Add a
fraction to add for the paper, so mine would then be `2.58`.

Set the Z-Axis (note we've converted the number from the last step to be negative)

    M851 Z -2.58

Enable soft end stops

    M211 51

Save settings to EEPROM

    M500

Set active parameters

    M501

Display current settings

    M503

Home the hot end

    G28

Move the hot end to true zero offset to see results

    G1 F60 Z0

## Setting Z-Axis quick guide

First half script:

    G28
    M851 Z0
    M500
    M501
    M503
    G28
    G1 F60 Z0
    M211 S0

Using the 'Control' feature in Octoprint, move the print head down to the table
until a piece of paper can barely move under it.

Take note of what the Z axis says on the display. Mine was `2.57`. Add a
fraction to add for the paper, so mine would then be `2.58`.

Second half script:

    M851 Z -2.58
    M211 51
    M500
    M501
    M503
    G28
    G1 F60 Z0

## Auto level the bed (Display panel)

On the display, go to `Level`, `Create new mesh`. When complete, save it to
EEPROM.

Your ideal tolerance range when viewing the mesh is less than `0.1`.

Be certain to read the [using the saved bed mesh](#using-the-saved-bed-mesh-auto-level)
section a bit later in this document, or the saved mesh won't be used!

## Auto level the bed (Bed Visualizer)

Install the `Bed Visualizer` plugin.

Go to the plugin's settings. Each section below is for the various configuration
tabs.

Paste the following GCODE into 'Collection tab', 'GCODE Commands' window:

    M140 S60              ; starting by heating the bed for nominal mesh accuracy
    M117 Homing all axes  ; send message to printer display
    G28                   ; home all axes
    M420 S0               ; Turning off bed leveling while probing, if firmware is set

    ; to restore after G28

    M117 Heating the bed    ; send message to printer display
    M190 S60                ; waiting until the bed is fully warmed up
    M300 S1000 P500         ; chirp to indicate bed mesh levels is initializing
    M117 Creating the bed mesh levels ; send message to printer display
    M155 S30                ; reduce temperature reporting rate to reduce output pollution
    @BEDLEVELVISUALIZER	    ; tell the plugin to watch for reported mesh
    G29 T	                ; run bilinear probing
    M155 S3                 ; reset temperature reporting
    M140 S0                 ; cooling down the bed
    M500                    ; store mesh in EEPROM

Enable or disable 'Save Mesh'.

Enable or disable 'Webcam while processing'.

Leave all other settings default.

You can view your mesh data in the `Data` tab.

## Setting up Cura slicer

This will allow you to use Cura to slice, and send jobs directly to the printer
through Octoprint.

Download and install [Ultimaker Cura](https://ultimaker.com/software/ultimaker-cura)

Install the OctoPrint Connection plug-in from the Cura Marketplace.

Install the Auto-Orientation plugin.

Configure the printer:

- Name: Voxelab Aquila
- X (Width): 220mm
- Y (Depth): 220mm
- Z (Height): 250mm
- Heated bed: Y
- G-code flavor: Marlin
- Leave everything else default

Then simply click 'Print with OctoPrint' when you have a job loaded.

## Using the saved bed mesh (auto-level)

In the Manage Printer section of Cura settings, add the following line directly
beneath the `G28` line in your printer's "Start G-Code":

    M420 S1 ; Use saved bed mesh

## Print the vertical display bracket

The vertical display bracket files can be found [here](https://github.com/stevieb9/voxelab-aquila/raw/main/files/vertical_display_mount.zip).

That project was downloaded from [here](https://www.thingiverse.com/thing:4764038).

## Print the other accessories

All accessory STL files can be found [here](https://github.com/stevieb9/voxelab-aquila/tree/main/files).

- Pi3 under chassis mount
- Y-Axis camera mount
- Filament guide
- Solid State Relay rail mount

## Setting up UI power toggle

Connect a relay to the Ground connection of the Pi, and GPIO (BCM) pin 21.

Install the `PSU Control` plugin into OctoPi, and configure as follows:

GPIO Device

- `/dev/gpiochip0`

Switching

- Switching method: `GPIO`
- On/off GPIO pin: `21` (use invert depending on your relay)
- Enable switching with GCode commands: `Yes`
- On G-Code command: `M80`
- Off G-Code command: `M81`
- Turn off on unrecoverable error: `Yes`

Sensing:

Sensing method: `Internal`

## Disable Pi powering the printer

Cut a small piece of electrical tape, and place it over the 5v pin of the USB
cable that connects to the Pi.

## Configure iPhone app

In OctoPrint, install the `OctoPod` notifications.

In OctoPrint, install the `Obico for OctoPrint` plugin.

Proceed through the Obico configuration.

Install the `OctoPod` plugin from the App Store.

Go to `Settings`.

Click `Printers`.

Click the `+` icon to add a new printer.

Use the `The Spaghetti Detective` option.

Proceed through the steps.

Once done, you will be asked to authorize using a QR Code. In OctoPrint settings,
go to `Printer`, `Application Keys` and generate a new App Key. This will result
in a QR Code being displayed in the UI that you then capture with your phone.

## PETG Cura configuration

- Settings
- Printers
- Manage Printers
- Materials
- Select 'Generic PETG'
- Click the 'hamburger' in the top-right corner
- Click 'Duplicate'
- Rename to "Custom PETG"
- Print Settings
- Set "Default printing temperature" to `225`
- Set "Retraction Distance" to `7mm`
- Set "Retraction Speed" to `55mm/s`
- Click the 'hamburger'
- Click 'Activate'