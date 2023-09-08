# Blink Module Serial Interface
## Description
Blinks the Arduino board LED (pin 13) to indicate the values of bytes arriving from the Bpod state machine.

The blink module is intended for educational and debugging purposes.

Requires an Arduino with BlinkModule firmware loaded from:

- [/Bpod_Gen2/Examples/Firmware/Bpod Shield/BlinkModule/](https://www.google.com/url?q=https%3A%2F%2Fgithub.com%2Fsanworks%2FBpod_Gen2%2Ftree%2Fmaster%2FExamples%2FFirmware%2FBpod%2520Shield%2FBlinkModule&sa=D&sntz=1&usg=AOvVaw1RRU0rLWGcz2SO5StvGBGY)
  
## Command Interface
- Byte 255 (reserved): Returns module info to state machine
- All other bytes: blinks LED to indicate byte value (byte 1 = 1 blink, byte 4 = 4 blinks, etc.)

## Example
This code creates, sends and runs a state machine with 1 state. The single state causes the Arduino board to blink 5 times. Set nBlinks to customize blink number. 
```matlab
nBlinks = 5; 
sma = NewStateMachine();

sma = AddState(sma, 'Name', 'TriggerBlink', ...
    'Timer', 0,...
    'StateChangeConditions', {'Tup', 'end'},...
    'OutputActions', {'BlinkModule1', nBlinks});

SendStateMachine(sma);
RawEvents = RunStateMachine
```