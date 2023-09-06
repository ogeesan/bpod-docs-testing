# SAMD21Mini

To update state machine firmware for a module using Sparkfun SAMD21 Mini:

1. Plug the module into the governing computer's USB port. 
2. (Windows only) If the drivers are not yet installed (or if you're not sure), follow Sparkfun's [SAMD21 Mini Driver Installation instructions](https://learn.sparkfun.com/tutorials/samd21-minidev-breakout-hookup-guide/hardware-setup).
3. Open the Arduino program folder and run Arduino.exe (in Linux/Mac, open the Arduino application). 
4. Follow [the instructions](https://learn.sparkfun.com/tutorials/samd21-minidev-breakout-hookup-guide/setting-up-arduino) to install support for the SAMD21 Mini board.
5. Restart Arduino 
6. From the "Tools" menu, choose "Board" and then "Sparkfun SAMD21 Mini Breakout".
7. From the "Serial Port" menu, choose "COMX" (win) or "/dev/ttySX" (linux) where X is the port number. To find your port number in Win7-10, choose "Start" and type "device manager" in the search window. In the device manager, scroll down to "Ports (COM & LPT)" and expand the menu. The COM port will be listed as "Sparkfun SAMD21 Breakout (COMX)" where COMX is a serial port name. On Unix this will not be "COM". If you can't find your port, see the note below the instructions.
8. From the File menu in Arduino, choose "Open" and select the firmware file. 
    - A new window should open with the firmware.
9. In the new window, click the "upload" button (the right-pointing arrow roughly under the "edit" menu).

If all went well, the green progress indicator should finish, and be replaced with a message: "Done uploading". 

On some platforms, you may see a message "an error occurred while uploading the sketch". This does not mean the upload was unsuccessful; hopefully future versions of the uploader will fix this error. Try out the module, and see if it behaves as expected.

NOTE: In some circumstances, the SAMD21 Mini may get stuck during a firmware upload, in a state with a fading blue LED and no COM port visible to the PC. To fix this, unplug the board, then plug it in and double-click the SAMD21 Mini's black button. The COM port may change temporarily (e.g. from COM3 to COM4). Select the new port in Arduino and load the firmware.