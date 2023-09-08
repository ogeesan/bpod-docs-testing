# Rotary Encoder Module
## Description
Allows the state machine or PC to log rotary motion and detect threshold crossings using the rotary encoder module.
Requires a rotary encoder module board with firmware loaded from:
[https://github.com/sanworks/Bpod_RotaryEncoder_Firmware](https://www.google.com/url?q=https%3A%2F%2Fgithub.com%2Fsanworks%2FBpod_RotaryEncoder_Firmware&sa=D&sntz=1&usg=AOvVaw3P2j3QYw8vJD4SYkDNsaNT)

The Rotary Encoder Module must be connected to a free serial port on the Bpod state machine.

!!! important
    There are two versions of the Rotary Encoder Module with non-overlapping feature sets. Both modules are controlled by the `RotaryEncoderModule` class. Functions specific to each version are indicated with:
    
    - Module v1 only: :one:
    
    - Module v2 only: :two:

## State Machine Command Interface
- Byte **255** (reserved): Returns module info to state machine
- '**L**' (ASCII 76):one:: **Start logging position+time data to the microSD card**. 
    - Resets logging position to 0, overwriting previously logged data.
- '**F**' (ASCII 70):one:: **Finish logging position+time data to the microSD card**. 
    - Stops logging data. A call to 'R' (see below) must be made to return data to the PC before the next call to 'L'.
- '**Z**' (ASCII 90): **Set current rotary encoder position to zero**. 
- '**E**' (ASCII 69): **Enable all position thresholds**.
    - Position thresholds are disabled individually once they are crossed, generating a behavior event. Call 'E' to re-enable all of them.
- '**O**' (ASCII 79):one:: **Start / Stop module output stream**.
    - Following 'O', the module expects 1 byte:
        - 1 to start the module stream
        - 0 to stop the module stream
- '**#**' (ASCII 35): **Timestamp a byte-message and return via ongoing USB stream.** (Note: Req. Firmware v2)
    - Following '#', the module expects 1 byte:
        - a message (0-255)
- '**\***' (ASCII 42):two:: **'Push' command**. If advanced thresholds were loaded to the device with 't' command (below), make them the current thresholds.
- '**X**' (ASCII 88): **Stops streaming + logging**.

## SerialUSB Command Interface
The SerialUSB command interface allows configuration of the rotary encoder module with MATLAB or Python before a trial begins. It also allows data return from the module's onboard microSD card. The `RotaryEncoderModule` class/plugin for Bpod/MATLAB wraps this interface. The first 6 commands are the same as for the state machine interface (though an acknowledgement byte = 1 is returned in each case), and additional commands follow.

- '**R**' (ASCII 82):one:: **Retrieve captured position data from the module's microSD card**. The module replies with the following bytes:
    - nPositionsAcquired (4 bytes; 32-bit int); number of samples acquired since last call to 'L' (see above)
    - for (each position)
        - a position (2 bytes; 16-bit signed int)
        - a timestamp (4 bytes; 32-bit unsigned int)
    - Positions are signed integers in units of rotary encoder tics (1024 = 1 full rotation)
    - Times are given in milliseconds
    - If no data is available, 0 is returned for nPositionsAcquired.
- '**Q**' (ASCII 81): **Return the encoder's current position**. The module replies with the following bytes:
    - currentPosition (2 bytes; 16-bit signed int);
    - units = rotary encoder tics (1024 / full rotation)
- '**P**' (ASCII 80): **Set current rotary encoder position**. 'P' (byte 0) is followed by:
    - Bytes 1-2: a 16-bit signed integer indicating the new position.
        - units = rotary encoder tics (1024 / full rotation).
        - Negative values are permitted, but their absolute value must not exceed wrapPoint (see 'W' below).
    - The rotary encoder module returns a byte (1) to confirm that it has finished setting the current position.
- '**S**' (ASCII 83): **Start/Stop streaming position and time measurements to the USB port**. 'S' (byte 0) is followed by:
    - streamingEnabled (1 byte): 0 to stop USB streaming, 1 to start USB streaming. Also see '#' command above.
    - If enabled, bytes arrive at the USB serial port as follows:
    - while (streamingEnabled)
        - ---For firmware v1:---
            - for (each new position)
                - currentPosition (2 bytes; 16-bit signed int; units = rotary encoder tics (1024 / full rotation)
                - currentTime (4 bytes; 32-bit unsigned int; units = ms)
        - ---For firmware v2:---
            - whichData (1 byte):
                - 'P' (ASCII 80) if position data follows
                - 'E' (ASCII 69) if event data follows
            - IF 'P' was received (position data):
                - nPositions (1 byte): number of positions to read
                - FOR each position in nPositions
                        - currentPosition (2 bytes; 16-bit signed int; units = rotary encoder tics (1024 / full rotation)
                        - currentTime (4 bytes; 32-bit unsigned int; units = ms)
            - IF 'E' was received (event data):
                    - eventOrigin (1 byte): 0 if from state machine, 1-3 reserved for events from TTL and I2C
                    - eventCode (1 byte): For state machine events, the "event" byte that followed '#' (above)
                        - currentTime (4 bytes; 32-bit unsigned int; units = ms)
        - ---For firmware v3 or newer:---
            - whichData (1 byte):
                - 'P' (ASCII 80) if position data follows
                - 'E' (ASCII 69) if event data follows
            - IF 'P' was received (position data):
                - currentPosition (2 bytes; 16-bit signed int; units = rotary encoder tics (1024 / full rotation)
                - currentTime (4 bytes; 32-bit unsigned int; units = ms)
         - IF 'E' was received (event data):
                - eventOrigin (1 byte): 0 if from state machine, 1-3 reserved for events from TTL and I2C
                - eventCode (1 byte): For state machine events, the "event" byte that followed '#' (above)
                - currentTime (4 bytes; 32-bit unsigned int; units = ms)
- '**V**' (ASCII 86): **Enable/Disable event transmission to state machine**. 'V' (byte 0) is followed by:
    - eventsEnabled (1 byte): 0 to disable sending threshold events, 1 to enable.
    - The rotary encoder module returns a byte (1) to confirm that it has finished enabling/disabling events.
- **'T'** (ASCII 84): **Program position thresholds (used to generate behavior events).** 'T' (byte 0) is followed by:
    - nThresholds (1 byte): number of thresholds to use. This must be less than maxThresholds (defined in firmware)
    - for (each threshold between 0 and nThresholds)
        - threshold (16-bit signed integer; units = rotary encoder tics (1024 / full rotation))
        - The absolute value of all thresholds must be less than wrap point (see 'W' below)
        - The rotary encoder module returns a byte (1) to confirm that it has finished programming the list of thresholds.
- '**t**' (ASCII 116):two:: **Program 'advanced' position thresholds**. Advanced thresholds are not made current when loaded to the device, until the device receives a **\*** command (see above). Threshold type can be type 0 (position threshold) or type 1 (threshold reached by remaining within position range for a set amount of time). 't' (byte 0) is followed by:
    - nThresholds (1 byte): number of thresholds to program
    - for (each threshold between 0 and nThresholds)
        - thresholdType (1 byte): Either 0 (position threshold) or 1 (time within range)
    - for (each threshold between 0 and nThresholds)
        - threshold (16-bit signed integer): Position threshold. If thresholdType == 1, this is the range boundary (+/- with respect to position 0)
    - for (each threshold between 0 and nThresholds)
        - thresholdTimes (32-bit unsigned integer): Time for thresholdType1 (unit = increments of 100 microseconds)
- '**W**' (ASCII 87): **Set wrap point (number of tics in a half-rotation)**. 'W' (byte 0) is followed by:
    - Bytes 1-2: a 16-bit signed integer indicating the wrapPoint.
        - units = rotary encoder tics (512 / half rotation).
            - wrapPoint is the point at which the lowest permissible negative position is wrapped to the highest permissible positive position.
                - At 512 (default), position wraps from -512 to +512 (for a total of 1024 tics per rotation).
                - At 1024, two full rotations are required in either direction to wrap the current position (i.e. positions between -1024 and +1024 are permissible).
                - wrapPoint can be used to allow thresholds more distant than 1 rotation away from the start point (position 0).
    - The rotary encoder module returns a byte (1) to confirm that it has finished setting the wrapPoint.
- '**I**' (ASCII 73):one:: **Set 1-character prefix for module output stream**. 'I' (byte 0) is followed by:
    - prefix (1 byte): a prefix-byte sent before each 16-bit position in the output data stream
        - The prefix should match the data format expected by the receiving module
        - The rotary encoder module returns a byte (1) to confirm that it has finished setting the prefix.
- '**;**' (ASCII 59): **Enable / disable specific event thresholds**. ';' (byte 0) is followed by:
    - 1 byte whose bits indicate whether each threshold up to nThresholds is enabled (1) or disabled (0) (for nThresholds, see 'T' command)