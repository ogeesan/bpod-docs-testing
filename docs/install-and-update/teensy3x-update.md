# Teensy3_X - 4_X
To update firmware for a Bpod module or state machine using Teensy 3.X or 4.X:

1. Plug the module into the governing computer's USB port. If it is your first time uploading firmware, press Teensy's 'Reset' button once, to make sure its bootloader mode is detected by the OS.
2. Unplug USB cables of all other Bpod modules from the same computer, and unplug all CAT5 (Ethernet) cables from the device you're updating. This is to avoid sending the firmware to the wrong board.
3. Follow the instructions to [install Teensyduino](https://www.pjrc.com/teensy/td_download.html) into your Arduino program folder. The installer includes Windows drivers.
4. Open the Arduino program folder and run Arduino.exe (in Linux/Mac, open the Arduino application). 
5. From the "Tools" menu, choose "Board" and then the Teensy board you want to program. For Bpod modules these are: 
    - Teensy 3.2 (for DDS Module, Ethernet Module and Port Array Module)
    - Teensy 3.5 (for Rotary Encoder Module v1)
    - Teensy 3.6 (for State Machine v2.0-2.4, Analog Input v1 and Analog Output v1 modules)
    - Teensy 4.0 (for Rotary Encoder v2 and Valve Driver v2)
    - Teensy 4.1 (for State Machine v2.5 and 2+, HiFi, Analog Input v2 and Analog Output v2)
6. From the "Serial Port" menu, choose "COMX" (win) or "/dev/ttySX" (linux) where X is the port number. If it is your first time uploading, the port may not be displayed - this is OK. Otherwise, to find your port number in Win7-10, choose "Start" and type "device manager" in the search window. In the device manager, scroll down to "Ports (COM & LPT)" and expand the menu. The COM port will be listed as "Teensy USB Serial (COMX)" where COMX is a serial port name. On Unix this will not be "COM".
7. Important: Check the top of the firmware file for instructions regarding CPU Speed (selectable from the Arduino 'Tools' menu). The default is OK if no note is given.
8. From the File menu in Arduino, choose "Open" and select the firmware file. 
    - A new window should open with the firmware.
9. In the new window, click the "upload" button (the right-pointing arrow roughly under the "edit" menu).

The Teensyduino application will launch, and a small window with a progress indicator will open and close.

If all went well, the blue bar at the bottom of the firmware window should display "Done uploading".