# Firmware update

## Automatic

!!! note
    An experimental auto-updater is included with the latest Bpod software. It is not available on all platforms, and as new software it carries the risk of malfunction in your particular MATLAB + PC configuration. If you choose to try it,

- To update the firmware of your state machine:
    - Ensure the Bpod software is [installed](../user-guide/software-update.md)
    - Start MATLAB
    - Run the following at the MATLAB command line:
        - BpodFirmwareUpdate(MyPort)
        - MyPort is the state machine's USB serial port (e.g. 'COM3')
        - The updater will automatically detect your state machine's firmware version and give you the option to update
- To update firmware of your module(s):
    - Connect each module you want to update to the state machine using a CAT5 (Ethernet) cable
    - Also connect each module you want to update to the PC via USB (even if a USB cable is not usually needed)
    - Run Bpod()
    - From the Bpod console, use the 'USB' button to pair each connected module with its USB serial port
    - Next, run the following at the MATLAB command line:
        - BpodFirmwareUpdate
- The updater will automatically find any connected modules and give you the option to proceed with the update

## Semi-Automatic

The semi-automatic option works on Windows only, and is only recommended for Teensy-based modules:

- State Machine r2
- Analog Output Module
- Analog Input Module
- HiFi Module
- Rotary Encoder Module
- Ethernet Module
- DDS Module
- Port Array Module
From the MATLAB-command prompt, run: LoadBpodFirmware;

A GUI will launch. Select the correct firmware and the target device's COM port, and click 'upload'.

## Manual

FOR ALL: 

- Download [Arduino 1.8.X](http://arduino.cc/en/Main/Software#toc3) non-admin / zip, extract the zip folder and save the extracted folder somewhere permanent on your PC.
- Download the firmware file to update. Firmware is an Arduino [sketch](https://www.arduino.cc/en/Tutorial/Sketch), hosted on the Sanworks Github account:
    - [Bpod State Machine](https://github.com/sanworks/Bpod_StateMachine_Firmware) (select your machine from the /Preconfigured/ folder)
- [Analog Output Module](https://github.com/sanworks/Bpod_AnalogOutput_Firmware)
- [Analog Input Module](https://github.com/sanworks/Bpod_AnalogInput_Firmware)
- [DDS Module](https://github.com/sanworks/Bpod_DDS_Firmware)
- [Rotary Encoder Module](https://github.com/sanworks/Bpod_RotaryEncoder_Firmware)
- [Valve Driver Module](https://github.com/sanworks/Bpod_ValveDriver_Firmware)
- [I2C Messenger Module](https://github.com/sanworks/Bpod_I2CMessenger_Firmware)
- [SNES Module](https://github.com/sanworks/Bpod_SNES_Firmware)
    - [Example firmware](https://github.com/sanworks/Bpod_Gen2/tree/master/Examples/Firmware) for Arduino / Teensy Shields

Next, continue to specific instructions for the device you want to update:

<!-- todo: fix links here -->
[Modules powered by Arduino Due](/site/bpoddocumentation/firmware-update/state-machine?authuser=0) (State Machine v0.5-1.0)

[Modules powered by Teensy 3.X and 4.X](/site/bpoddocumentation/firmware-update/teensy3_x?authuser=0) (State Machine v2, 2.5 and 2+, Analog Output, Analog Input, DDS, HiFi, Ethernet, Rotary Encoder, Port Array, Valve Driver v2, Examples using Teensy Shield)

[Modules powered by Sparkfun SAMD21 Mini](/site/bpoddocumentation/firmware-update/samd21mini?authuser=0) (ValveDriver v1, I2C, SNES)

[Modules powered by Arduino M0](/site/bpoddocumentation/firmware-update/arduinom0?authuser=0) (Examples using Arduino Shield)