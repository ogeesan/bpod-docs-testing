# Serial message setup
Like other output actions, the serial messages are released when the state begins.

### `LoadSerialMessages()`
**Description**

When building a state machine, The output action {"Serial1", N} can trigger byte N to be sent to a connected module named 'Serial1'.

N can also specify a string of 1-3 bytes. 

This function loads byte strings for different output bytes on the UART serial channels.

**Syntax**

```matlab
Acknowledged = LoadSerialMessages(SerialPort, Messages, [MessageIndexes])
```

**Parameters**
- SerialPort: The UART serial port number (1-2 on Bpod 0.5, 1-3 on Bpod 0.7, 1-5 on Bpod Pocket State Machine)
    - If a recognized Bpod module is on the port, you can also use its name as a string (e.g. 'ValveModule1')
- Messages: A cell array of messages
- (optional) MessageIndexes: A list of indexes for the byte strings in the Messages argument.
    - By default, the indexes of Messages are consecutive.

**Returns**

- Acknowledged: 1 if messages successfully transmitted, 0 if not

**Examples**

Example1: Loads [5 8] as message#1, and [2 3 4] as message#2 on UART serial port 1
```matlab
LoadSerialMessages(1, {[5 8], [2 3 4]});
```


Example2: Loads ['X' 3] as message#8 on UART serial port 3 
```matlab
LoadSerialMessages(3, ['X' 3], 8); 
```

### `ResetSerialMessages()`
**Description**

When building a state machine, The output action `{"Serial1", N}` can trigger byte N to be sent.

`LoadSerialMessages()` loads byte strings to transmit instead of each Byte N.

`ResetSerialMessages()` returns each message to its default (Byte N).

**Syntax**

```matlab
Acknowledged = ResetSerialMessages()
```

**Parameters**

- None

**Returns**

- Acknowledged: 1 if messages successfully reset to defaults, 0 if not

**Example**

```matlab
% Loads ['X' 3] as message#7 on UART serial port 3 
LoadSerialMessages(3, ['X' 3], 7); 

% Resets message#7 to '7', and message#N to 'N' (default) as on all serial ports
ResetSerialMessages; 
```

### Implicit serial messages
**Description**

Messages to be sent from the state machine to its modules are stored in a library onboard the state machine. The library can be explicitly programmed with `LoadSerialMessages()`.

As of Bpod Console v1.70 and Bpod Firmware v23, serial messages can be added implicitly in the state description.

**Example**

Create a state named 'Error'. After a 3 second delay, the system exits the state. On entering the state, the byte sequence `['P' 2]` is sent to the HiFi module (to play an error sound loaded at sound position 2).

```matlab
sma = AddState(sma, 'Name', 'Error', ...
     'Timer', 3,...
     'StateChangeConditions', {'Tup', '>exit'},...
     'OutputActions', {'HiFi1', ['P' 2]});
```
