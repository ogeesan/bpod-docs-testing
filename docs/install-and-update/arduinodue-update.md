# Arduino Due (State Machine v0.5-1.0)
**To update state machine firmware for Bpod hardware v0.5-0.9:**

1. Plug the Bpod device into the governing computer's USB port.
2. (Windows only) If the drivers are not yet installed (or if you're not sure), follow Arduino Due's [Windows driver installation page](http://www.google.com/url?q=http%3A%2F%2Farduino.cc%2Fen%2FGuide%2FArduinoDue%23toc8&sa=D&sntz=1&usg=AOvVaw1QRxjl0FOoWYTqks4F8waN).
3. Open the Arduino program folder and run Arduino.exe.
4. Install support for Arduino Due (if you haven't done this already):
    - From the "Tools" menu, choose "Board" and then "Boards Manager".
    - In the boards manager, install "Arduino SAM boards (32-bits ARM Cortex M3).
    - Restart Arduino
5. From the "Tools" menu, choose "Board" and then "Arduino Due (Native USB Port)".
6. From the "Serial Port" menu, choose "COMX" (win) or "/dev/ttySX" (linux) where X is the port number. To find your port number in Win7, choose "Start" and type "device manager" in the search window. In the device manager, scroll down to "Ports (COM & LPT)" and expand the menu. The COM port will be listed as "Arduino Due Native USB Port (COMX)" where COMX is a serial port name. On Unix this will not be "COM".
7. From the File menu in Arduino, choose "Open" and select the firmware.
    - For Bpod 0.5, use: "/Bpod/Firmware/Bpod0_X/Bpod_StateMachine_0_X_Y/Bpod_StateMachine_0_X_Y.ino.
    - For older Bpod software, use Bpod_MainModule_0_X_Y.
    - For Bpod 0.7+ use: /Bpod/Firmware/Gen2/BpodStateMachine_0_X_Y
    - A new window should open with the firmware.
8. In the new window, click the "upload" button (the right-pointing arrow roughly under the "edit" menu).

If all went well, the green progress indicator should finish, and be replaced with a message: "Done uploading". In orange text below, should read "Verify successful".