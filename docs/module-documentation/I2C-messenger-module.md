# I2CMessenger()

## Description

Translates the Bpod serial interface (UART / RS-485) to the 2-wire I2C interface, using a Bpod I2C Messenger Module. 

- Enables message transmission to external instruments (e.g. ScanImage).
- Acts as an I2C master. By default, transmits all incoming messages from Bpod to slave#1.
- Output logic levels (3.3V or 5V) are configurable with a jumper on the device.
- Powered by the open source [SAMD21 breakout board](https://www.google.com/url?q=https%3A%2F%2Fwww.sparkfun.com%2Fproducts%2F13664&sa=D&sntz=1&usg=AOvVaw0XeduDYUT1Y6LCsRG8inUF), an Arduino-compatible ARM Cortex M0 microcontroller.
- By default, Messenger passes bytes in both directions.
- The USB connection can be used to load messages (up to 16 bytes) to transmit instead of each byte that arrives.
- Compatible with Bpod 0.7+

## Syntax

To initialize:
```matlab
I2C = I2CMessenger(portString); 
```

- portString = the I2C messenger's serial port (e.g. COM3)
- Returns an I2C Messenger object

To set the current slave:
```matlab
I2C.SlaveAddress = address;
```
- address = 1-255
- Default = 1

To load a 0-16 byte serial message:
```matlab
I2C.loadMessage(messageIndex, message, [messageSlaveAddress]);
```
- messageIndex = 1-256
- message = a 1-16 byte array of bytes
- optional: messageSlaveAddress = a fixed I2C slave address for the recipient (default = current slave)

To set the messaging mode:
```matlab
I2C.Mode = mode;
```
- mode = 'Relay', 'Message', 'USBRelay' or 'USBMessage
    - 'Relay' - Sends incoming bytes to current I2C slave
    - 'Message' - Sends incoming 1-16 byte messages to their I2C slave(s)
    - 'USBRelay' - Sends incoming bytes to the USB serial port (for diagnostics)
    - 'USBMessage' - Sends incoming 1-16 byte messages to the USB serial port (for diagnostics)

To set the transfer speed:
```matlab
I2C.TransferSpeed = speed;
```
- speed = 'Standard' or 'FastMode' (default). Standard = 100kb/s, FastMode = 400kb/s

To reset all parameters to defaults:
```matlab
I2C.reset;
```

To disconnect the serial port and clear the object:
```matlab
clear I2C
```

**Syntax** (diagnostics)

To force-send a byte from the I2C module to the Bpod state machine serial port:
```matlab
I2C.bpodWrite(byte);
```

To force-send a byte from the I2C module to the current I2C slave:
```matlab
I2C.I2Cwrite(byte);
```

To force-send a 1-16 byte message from the I2C module to its I2C slave:
```matlab
I2C.triggerMessage(messageIndex);
```

## Example

This code connects to the I2C module. A byte is sent to I2C slave#3, triggered by USB. Then, message#34 is set to a 13-byte message, to slave#5. The latter message is triggered from the state machine.

```matlab
I2C = I2CMessenger('COM6'); % Initialize I2C messenger on port COM6
I2C.SlaveAddress = 3;
I2C.Mode = 'Message';
I2C.I2Cwrite('A'); % Force-send 'A' to slaveAddress 3
I2C.setMessage(34, 'HelloI2Cslave', 5); % message#, message, slaveAddress

sma = NewStateMachine();
sma = AddState(sma, 'Name', 'SendI2CMessage', ...
    'Timer', 0,...
    'StateChangeConditions', {'Tup', 'exit'},...
    'OutputActions', {'I2C1', [1 34]});% 1 is the op code to send a message. 
                                       % Because the module is in 'Message' mode, message#34 will
                                       % be sent instead of byte 34.

SendStateMachine(sma);
RawEvents = RunStateMachine;
```