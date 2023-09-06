# ArduinoM0

To update state machine firmware for a module using Arduino M0:
1. Plug the module into the governing computer's USB port.
2. (Windows only) If the drivers are not yet installed (or if you're not sure), follow Arduino M0's [Windows driver installation page](https://www.google.com/url?q=https%3A%2F%2Fwww.arduino.cc%2Fen%2FGuide%2FArduinoM0%23toc3&sa=D&sntz=1&usg=AOvVaw0f_rLeyViiHur_9XYH43GZ).
3. Open the Arduino program folder and run Arduino.exe.
4. Install support for Arduino Due (if you haven't done this already):
    - From the "Tools" menu, choose "Board" and then "Boards Manager".
    - In the boards manager, install "Arduino SAMD boards (32-bits ARM Cortex M0).
    - Restart Arduino
5. From the "Tools" menu, choose "Board" and then "Arduino M0".
6. From the "Serial Port" menu, choose "COMX" (win) or "/dev/ttySX" (linux) where X is the port number. To find your port number in Win7, choose "Start" and type "device manager" in the search window. In the device manager, scroll down to "Ports (COM & LPT)" and expand the menu. The COM port will be listed as "Arduino M0 Native Port (COMX)" where COMX is a serial port name. On Unix this will not be "COM".
7. From the File menu in Arduino, choose "Open" and select the firmware file.
    - A new window should open with the firmware.
8. In the new window, click the "upload" button (the right-pointing arrow roughly under the "edit" menu).

If all went well, the green progress indicator should finish, and be replaced with a message: "Done uploading". In orange text below, should read "AVRDude Done. Thank you".