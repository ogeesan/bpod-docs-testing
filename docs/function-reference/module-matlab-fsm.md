# Module <-> MATLAB (via FSM)
### `ModuleWrite()`
**Description**

Writes values to a Bpod module, via its serial connection to the state machine. (i.e. MATLAB --> State Machine --> Module)

**Syntax**
```matlab
ModuleWrite(ModuleName, Values, [Datatype]) 
```

**Parameters**

- ModuleName: The module's name (a character array). See BpodSystem.Modules for the names of connected modules.
- Values: Value(s) to send to the module. By default, values are 'uint8'.
- (optional) DataType: An integer data type. Supported types are:
    - uint8
    - uint16
    - uin32
    - int8
    - int16
    - int32

**Returns**

- None

**Examples**

```matlab
% Example1: Sends the character array "Hi there" via the state machine, to EchoModule1
ModuleWrite('EchoModule1', 'Hi there');

% Example2: Sends two 32-bit integers to SillyModule2
ModuleWrite('SillyModule2', [102483 297438], 'uint32');
```

### `ModuleRead()`
**Description**

Reads values from a Bpod module, via its serial connection to the state machine. (i.e. Module --> State Machine --> MATLAB)

!!! important
    The state machine must be manually configured to relay bytes from the module to the USB port, in order for `ModuleRead()` to work. Set the current relayed module with: `BpodSystem.StartModuleRelay(ModuleName)`. When you are done exchanging data with the module, you must call `BpodSystem.StopModuleRelay()` before using the state machine. If you do not call `StopModuleRelay()`, bytes relayed from the module may interfere with expected USB transmissions. Example code below shows proper usage.

**Syntax**

```matlab
Values = ModuleRead(ModuleName, nValues, [Datatype]) 
```

**Parameters**

- ModuleName: The module's name (a character array). See BpodSystem.Modules for the names of connected modules.
- Values: Value(s) to send to the module. By default, values are 'uint8'.
- (optional) DataType: An integer data type. Supported types are:
    - uint8
    - uint16
    - uin32
    - int8
    - int16
    - int32

**Returns**

- Values: An array of values returned from the module

**Examples**

Sends the character array "Hi there" via the state machine, to EchoModule1. Then, read the echo module's reply

```matlab
BpodSystem.StartModuleRelay('EchoModule1'); % Relay bytes from EchoModule1
ModuleWrite('EchoModule1', 'Hi there'); % Write character string "Hi there" to EchoModule1
Reply = ModuleRead('EchoModule1, 8); % Read 8 bytes (the length of "Hi there") from EchoModule1
BpodSystem.StopModuleRelay; % Cancel the relay from EchoModule1
```

Sends two 32-bit integers to SillyModule2. Reads SillyModule's reply - four 16-bit unsigned integers.

```matlab
BpodSystem.StartModuleRelay('SillyModule2');
ModuleWrite('SillyModule2', [102483 297438], 'uint32');
Reply = ModuleRead('SillyModule2, 4, 'uint16');
BpodSystem.StopModuleRelay;
```
