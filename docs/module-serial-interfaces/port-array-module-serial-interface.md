# Port Array Module Serial Interface

## Description

Allows the state machine to open valves, set LED brightness and read poke events from ports on the Bpod port array module.

Requires a port array module board with firmware loaded from:

- [https://github.com/sanworks/Bpod\_PortArray\_Firmware](https://www.google.com/url?q=https%3A%2F%2Fgithub.com%2Fsanworks%2FBpod_PortArray_Firmware&sa=D&sntz=1&usg=AOvVaw25LU7gLB4HGkrFGSCOiWZI)

To use the state machine interface, the port array module must be connected to a free serial port on the Bpod state machine.

## State Machine Command Interface

The state machine command interface consists of bytes sent from the Bpod state machine to the port array module to open and close valves, and set LED brightness.

- Byte 255 (reserved): Returns module info to state machine
- '**V**' (ASCII 86): **Set the state of a valve**.
    - Following 'V', the module expects 2 bytes:
        - port# (1 byte; range = 0-3)
        - state (1 byte; range = 0-1) Note: 0 = closed, 1 = open
- '**B**' (ASCII 66): **Set the state of all valves at once, using bits of a byte**
    - Following 'B', the module expects 1 byte:
        - valveState (1 byte; range = 0-15) Note: 0 = all cosed, 15 = all open
- '**P**' (ASCII 80): **Set port LED intensity (pulse width modulated)**.
    - Following 'P', the module expects 2 bytes:
        - port# (1 byte; range = 0-3)
        - duty cycle (1 byte; range = 0-255) Note: 0 = off, 255 = max brightness
- '**W**' (ASCII 87): **Set all port LED intensities**
    - Following 'W', the module expects 4 bytes:
        - duty cycles of ports 1-4 (4 bytes; range = 0-255) Note: 0 = off, 255 = max brightness
- '**L**' (ASCII 76): **Set all port LEDs at once, using bits of a byte to indicate (0 = off) or (1 = max brightness)**
    - Following 'L', the module expects 1 byte:
        - ledState (1 byte; range = 0-15) Note: 0 = all off, 15 = all on
- '**R**' (ASCII 82): **Reset clock.**
    - The clock is used to return timestamps when streaming poke events directly over USB. When used with the state machine, the state machine clock timestamps all incoming events.

## SerialUSB Command Interface

The SerialUSB command interface allows the port array module to receive commands and return events directly to MATLAB. The PortArrayModule [plugin](https://www.google.com/url?q=https%3A%2F%2Fgithub.com%2Fsanworks%2FBpod_Gen2%2Ftree%2Fmaster%2FFunctions%2FModules%2FPortArray&sa=D&sntz=1&usg=AOvVaw19q7rkpkHnEHo04aoARB06) for Bpod/MATLAB wraps this interface. The first 6 commands are the same as for the state machine interface (though an acknowledgement byte = 1 is returned for ops B and W), and additional commands follow:

- '**S**' (ASCII 83): **Return the current state of all ports**. The module replies with the following bytes:
    - State (4 bytes) = for each port, 0 if port is open, 1 if port is bocked
- '**U**' (ASCII 85): **Start/Stop event stream**. 'U' (byte 0) is followed by:
    - Byte 1: 0 to stop event stream, 1 to start
    - while streaming, the module returns the following bytes for each port-in or port-out event:
        - currentTime (8 bytes; 64-bit unsigned int, units = microseconds)
        - eventCode (4 bytes, one for each port)
            - 0 = no event 1 = Port1In, 2 = Port1Out, 3 = Port2In,... 8 = Port4Out