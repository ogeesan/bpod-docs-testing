# USB Soft Codes, PC --> FSM
### `SendBpodSoftCode()`
**Description**

Sends a byte code via USB to the Bpod state machine. The byte code can be handled during a trial like any other behavior event. By default, 15 bytes are reserved for soft codes (1-15). 

!!! note
    To access the MATLAB command line during a trial, you must use the `BpodTrialManager` class to run the trial's state machine.

**Syntax**

```matlab
SendBpodSoftCode(SoftCodeByte) 
```

**Parameters**

- SoftCodeByte: A byte to send to the state machine
    - Note: The byte must be in the range of supported soft code bytes. By default the range is 1-15.

**Returns**

- None

**Examples**

Send a 5 to the state machine after a random delay, triggering a state change

```matlab
sma = NewStateMachine;
sma = AddState(sma, 'Name', 'MyState', ...
   'Timer', 0,...
   'StateChangeConditions', {'SoftCode5', 'MyNextState'},...
   'OutputActions', {});

sma = AddState(sma, 'Name', 'MyNextState', ...
   'Timer', 0,...
   'StateChangeConditions', {'Tup', '>exit'},...
   'OutputActions', {});

T = BpodTrialManager; % Create an instance of the trial manager
T.startTrial(sma); % Start running the state machine
pause(rand*5); 
SendBpodSoftCode(5);
RawEvents = T.getTrialData;
clear T
```
