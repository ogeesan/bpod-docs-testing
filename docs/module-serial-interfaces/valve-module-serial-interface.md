# Valve Module Serial Interface


## Description

Allows the state machine or PC to control 8 solenoid valves, using the [Bpod Valve Driver Module](/site/bpoddocumentation/assembling-bpod/valve-driver-module?authuser=0).

Requires a valve module board with firmware loaded from:

- [https://github.com/sanworks/Bpod\_ValveDriver\_Firmware](https://www.google.com/url?q=https%3A%2F%2Fgithub.com%2Fsanworks%2FBpod_ValveDriver_Firmware&sa=D&sntz=1&usg=AOvVaw3EDkf9SYbLlmRxUlD3bAKl)

The valve module board must be connected to a serial port on the state machine module.

## Command Interface

- Byte 255 (reserved): Returns module info to state machine
- '**O**' (Byte 79): **Opens a valve**.
    - 'O' must be followed by:
        - A valve number (1-8) OR
        - The ASCII character of a valve number ('1' - '8') = (49 - 56)
- '**C**' (Byte 67): **Closes a valve**
    - 'C' must be followed by:
        - A valve number (1-8) OR
            - The ASCII character of a valve number ('1' - '8') = (49 - 56)
- '**B**' (Byte 66): **Sets the state of multiple valves**
    - 'B' must be followed by a byte. The 8 bits of the byte indicate valve states, for instance:
        - 1 = binary 00000001: valve 1 open, all others closed
        - 22 = binary 00010110: valves 2, 3 and 5 open, all others closed
- **All other bytes**: **Toggles state of valve**
    - If byte 1-8, toggles state of valve 1-8
    - If ASCII character of '1' to '8' (49-56), toggles state of numeric equivalent valve (1-8)

## Examples

Open and close valve 2 from the Arduino Serial Terminal:

> At the terminal, type 'O2', then 'C2'


Open and close valve 2 from an [ArCOM](http://www.google.com/url?q=http%3A%2F%2Fsites.google.com%2Fsite%2Fsanworksdocs%2Farcom&sa=D&sntz=1&usg=AOvVaw0q9tKPNJMCdKV2qsdKk90n) serial object in MATLAB:
```matlab
D = ArCOMObject('COM43', 115200);

D.write('O2', 'uint8');
D.write('C2', 'uint8');
clear D
```

Open and close valve 2 from the Bpod state machine
```matlab
LoadSerialMessages('ValveModule1', {['O' 2], ['C' 2]}); % Set serial messages 1 and 2

sma = NewStateMachine();
sma = AddState(sma, 'Name', 'OpenValve', ...
    'Timer', 0.1,...
    'StateChangeConditions', {'Tup', 'CloseValve'},...
    'OutputActions', {'ValveModule1', 1}); % Sends serial message 1

sma = AddState(sma, 'Name', 'CloseValve', ...
    'Timer', 0.1,...
    'StateChangeConditions', {'Tup', 'end'},...
    'OutputActions', {'ValveModule1', 2}); % Sends serial message 2

SendStateMachine(sma);
RawEvents = RunStateMachine;
```


This code is a loop that runs 8 times. On each iteration, it creates and runs a state machine with 2 states. The 2 states use the "toggle" op codes to open and close a valve. This code assumes all valves are initially closed.

```matlab
for i = 1:8
    sma = NewStateMachine();

    sma = AddState(sma, 'Name', 'OpenValve', ...
        'Timer', 0.1,...
        'StateChangeConditions', {'Tup', 'CloseValve'},...
        'OutputActions', {'ValveModule1', i}); % Toggle valve i Open

    sma = AddState(sma, 'Name', 'CloseValve', ...
        'Timer', 0.1,...
        'StateChangeConditions', {'Tup', 'end'},...
        'OutputActions', {'ValveModule1', i}); % Toggle valve i Closed

    SendStateMachine(sma);
    RawEvents = RunStateMachine;
end
```