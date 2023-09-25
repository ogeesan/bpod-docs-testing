# Creating a state machine
!!! note
    More examples of state machine creation can be found in the Examples/ folder of the repository. [Examples/State Machines/](https://github.com/sanworks/Bpod_Gen2/tree/master/Examples/State%20Machines) contains minimal (building block) examples, while [Examples/Protocols/](https://github.com/sanworks/Bpod_Gen2/tree/master/Examples/Protocols) contains complete examples of a protocol state matrix construction.

### `NewStateMachine()`
**Description**

Creates a new, empty state machine. 

States can be added to the empty state machine description with the `AddState()` function. The state machine description returned is a struct with 21 subfields:

- Meta: a struct with information about the state machine (sizes of variables, precomputed for speed)
- nStates: the number of states that have been added to the state machine
- Manifest: a cell array listing the names of states that have been referenced by other states (though not explicitly added yet)
- nStatesInManifest: the number of states in the manifest
- StateNames: a cell array of strings containing the name of each state added to the state machine with `AddState()`; initialized as a 1x1 cell with the string "Placeholder"
- InputMatrix: each row is a state, each column is an input event. The matrix specifies the new state to go to if each event occurrs in each state.
- OutputMatrix: each row is a state, each column is an output action. The matrix specifies the value of each output action in each state.
- StateTimerMatrix: for each state, specifies the new state to go to if the state's internal timer elapses.
- GlobalTimerStartMatrix: each row is a state, each column is a global timer. The matrix specifies the new state to go to when the global timer starts.
- GlobalTimerEndMatrix: each row is a state, each column is a global timer. The matrix specifies the new state to go to when the global timer elapses.
- GlobalTimers: each column is a timer. This vector specifies the current setting of each timer in seconds.
- GlobalCounterMatrix: each row is a state, each column is a global counter. The matrix specifies the new state to go to if the counter's threshold is exceeded.
- GlobalCounterEvents: each column is a counter. This vector specifies the input event being counted. It defaults to 255 (no input event).
- GlobalCounterThresholds: each column is a counter. This vector specifies the number of events recorded before each counter is exceeded.
- GlobalCounterSet: each column is a counter. This vector specifies whether each counter was used in the current matrix (1) or not (0).
- ConditionMatrix: each row is a state, each column is a configurable input channel condition. The matrix specifies the new state to go to if the condition is satisfied.
- ConditionChannels: The input channel index linked to each condition.
- ConditionValues: The value of the input channel (specified in ConditionChannels) for each condition to be satisfied.
- StateTimers: each column is a state. This vector specifies the setting of each state's internal timer in seconds.
- StatesDefined: each column is a state. States appear in the matrix as blank states when first referred to as the target of a state transition. StatesDefined is then set from 0 to 1 when the state is finally added.

**Syntax**

```matlab
StateMachine = NewStateMachine();
```

**Parameters**

- None

**Returns**

- An empty state machine struct

**Example**

This code initializes a state matrix and then adds one state:

```matlab
sma = NewStateMachine();
sma = AddState(sma, 'Name', 'MyState', ...
    'Timer', 1,...
    'StateChangeConditions', {'Tup', 'exit'},...
    'OutputActions', {});
```

### `AddState()`
**Description**

Adds a state to an existing state machine. 

**Syntax**

```matlab
NewStateMachine = AddState(StateMachineStruct, 'Name', StateName,...
    'Timer', TimerDuration,...
    'StateChangeConditions', Conditions,...
    'OutputActions', Actions)
```
**Parameters**

- StateMachineStruct: The state machine you are adding to. If this is the first state, StateMachineStruct is the output of `NewStateMachine()`.
- StateName: A character string containing the unique name of the state.
    - The state will automatically be assigned a number for internal use and state synchronization via the sync port.
- Timer: The state timer value, given in seconds
    - This value must be zero or positive, and can range between 0-3600s.
    - If set to 0s and linked to a state transition (see next bullet), the state will still take ~100us to execute the state's output actions before the transition completes.
- StateChangeConditions: A cell array of strings listing pairs of input events and the state changes they trigger.
    - Each odd cell should contain the name of a valid input event.
    - Each even cell should contain the name of the new state to enter if the previously listed event occurs, or 'exit' to exit the matrix and return all captured data..
- OutputActions: A cell array listing the output actions and corresponding values for the current state.
    - Each odd cell should contain the name of a valid output action.
    - Each even cell should contain the value of the previously listed output action (see output actions for valid values).

**Returns**

- A state machine struct, updated with the new state.

**Examples**

This code generates a simple state matrix that drives BNC output channel 1 to 5V (high) for 1 second before exiting. 
```matlab
sma = NewStateMachine();

sma = AddState(sma, 'Name', 'MyState', ...
    'Timer', 1,...
    'StateChangeConditions', {'Tup', 'exit'},... 
    'OutputActions', {'BNCState', 1}); 
% Tup occurs when the state's internal timer elapses
```

This code generates a simple state matrix that flashes the port LEDs of ports 1-3 for 0.1 second each (assuming an LED is connected to the port's PWM line). 
```matlab
sma = NewStateMachine();

sma = AddState(sma, 'Name', 'LightPort1', ...
    'Timer', 0.1,...
    'StateChangeConditions', {'Tup', 'LightPort2'},...
    'OutputActions', {'PWM1', 255}); 

sma = AddState(sma, 'Name', 'LightPort2', ...
    'Timer', 0.1,...
    'StateChangeConditions', {'Tup', 'LightPort3'},...
    'OutputActions', {'PWM2', 255}); 

sma = AddState(sma, 'Name', 'LightPort3', ...
    'Timer', 0.1,...
    'StateChangeConditions', {'Tup', 'exit'},...
    'OutputActions', {'PWM3', 255}); 
```

### `EditState()`
**Description**

Edits a state in an existing state machine. 

**Syntax**

```matlab
NewMatrix = EditState(StateMachineStruct, StateName, ParameterName, ParameterValue)
```

**Parameters**

- StateMachineStruct: The state machine you are editing.
- StateName: A character string specifying the name of the state you are editing.
- ParameterName: A character string specifying the parameter you are editing. Valid values are:
    - 'Timer'
    - 'StateChangeConditions'
    - 'OutputActions'
- ParameterValue: The new value of the parameter.
    - For timer, this is the new timer duration in seconds. 
    - For StateChangeConditions, this is a cell array of strings formatted with pair-wise arguments as in `AddState()`.
    - For OutputActions, this is a cell array of strings formatted with pair-wise arguments as in `AddState()`.

**Returns**

- A state machine struct, updated with the new parameter.

**Example**

This code generates a simple state machine that drives BNC output channel 1 to 5V (high) for 1 second before exiting. Next, EditState is used to make the state high for 10 seconds instead of 1. Finally, EditState is used to make the state machine end immediately if BNC input channel 1 receives a trigger pulse during the 10 second state.

```matlab
sma = NewStateMachine();
sma = AddState(sma, 'Name', 'MyState', ...
    'Timer', 1,...
    'StateChangeConditions', {'Tup', 'exit'},...
    'OutputActions', {'BNCState', 1}); 

sma = EditState(sma, 'MyState', 'Timer', 10);

sma = EditState(sma, 'MyState', 'StateChangeConditions',...
                     {'Tup', 'exit', 'BNC1High', 'exit');
```

### `SetGlobalTimer()`
**Description**

Sets the parameters of a global timer.

- Unlike state timers, global timers can be triggered from any state (as an output action), and handled from any state (by causing a state change). Any subset of global timers can be triggered or canceled from any state.
- An optional onset latency can be configured, following the timer trigger
- Following the onset latency, a "start" event is generated, and can trigger a state change.
- Then, following the timer duration, an "end" event is generated, which can also trigger a state change.
- A digital or PWM (LED) output channel can be linked to the timer. 
    - The linked channel is set "high" when the timer starts, and "low" when it ends. PWM values may be specified for onset/offset. 
- Separate serial output messages can be linked to the timer start and end events to control modules. 
- Global timers can be set to 'Loop'; repeat until they are explicitly canceled, or until a fixed number of iterations.
    - Each loop iteration generates a start and stop event (this can be disabled for high-frequency loops)
    - A configurable interval separates loop iterations (default = 0 seconds)
- Global timers can be linked to trigger other global timers. Following the timer's onset delay, any linked timers will be triggered.
- The number of available global timers is a configurable parameter specified in the state machine firmware.

**Syntax**

The function uses argument-value pairs. These must be listed in order (for efficiency), up to the last argument you need. Beyond that, optional arguments (denoted by [ ]) may be omitted):

```matlab
NewStateMachine = SetGlobalTimer(StateMachineStruct, 'TimerID', TimerNumber,... 
        'Duration', TimerDuration, ['OnsetDelay', OnsetDelay],...
        ['Channel', OutputChannel], ['OnsetValue', OnsetValue],... 
        ['OffsetValue', OffsetValue], ['Loop', LoopMode],...
        ['GlobalTimerEvents', EventsEnabled], ['LoopInterval', LoopInterval],...
        ['OnsetTrigger', OnsetTriggerByte])
```
where [ ] = optional argument
<!-- Check syntax docs to see if there's a better way of indicating optional argument -->

**Parameters**

- StateMachineStruct: The state machine description whose global timer you are setting (typically named 'sma').
- TimerNumber: The number of the timer you are setting (an integer, 1-5).
- TimerDuration: The duration of the timer, following timer start (0-3600 seconds)
- OnsetDelay: A fixed interval following timer trigger, before the timer start event (default = 0 seconds) 
    - If set to 0, the timer starts immediately on trigger and no separate start event is generated.
- OutputChannel: A string specifying an output channel to link to the timer (default = none)
    - Valid output channels can be viewed from the "inspect" icon on the Bpod Console.
- OnsetValue: The value to write to the output channel on timer start (default = none)
    - If the linked output channel is a digital output (BNC, Wire), set to 1 = High; 5V or 0 = Low, 0V
    - If the linked output channel is a pulse width modulated line (port LED), set between 0-255.
    - If the linked output channel is a serial module, OnsetValue specifies a byte message to send on timer start.
- OffsetValue: The value to write to the output channel on timer end (default = none)
- LoopMode: 0 = off (default). If set to 1, global timer loops until canceled or until trial end. If >1, indicates a fixed number of loop iterations to execute (up to 255).
- EventsEnabled: 1 = on (default). If set to 0, timer onset and offset events are not generated. Disabling events is useful for cases where the global timer is rapidly cycling to control a stimulus, and would otherwise generate a huge number of ignored behavior events.
- LoopInterval: A configurable delay between the end of a timer loop and the beginning of the next one (default = 0 seconds)
- OnsetTrigger: A byte whose bits indicate other global timers to trigger when the timer starts (following its onset delay).
    - Instead of an integer, the assembler will recognize a character string of 1s and 0s (i.e. '101001' to trigger timers 1,4 and 6)

**Returns**

- A state machine struct, updated with the new global timer settings.

**Examples**

The two examples below are for simple use cases. More complex global timer examples can be found in the Bpod repository: /Bpod_Gen2/Examples/State Machines/GlobalTimers/

This code generates a state machine that sets a global timer for 3 seconds, triggers it in the first state, and handles it in the second and third states. 
```matlab
sma = NewStateMachine();

sma = SetGlobalTimer(sma, 'TimerID', 1, 'Duration', 3); 

sma = AddState(sma, 'Name', 'State1', ...
    'Timer', 0,...
    'StateChangeConditions', {'Tup', 'State2'},...
    'OutputActions', {'GlobalTimerTrig', 1});

sma = AddState(sma, 'Name', 'State2', ...
    'Timer', 0,...
    'StateChangeConditions', {'Port1In', 'State3', 'GlobalTimer1_End', 'exit'},...
    'OutputActions', {});

sma = AddState(sma, 'Name', 'State3', ...
    'Timer', 0,...
    'StateChangeConditions', {'Port1Out', 'State2', 'GlobalTimer1_End', 'exit'},...
    'OutputActions', {});
```


This code generates a state machine that sets global timer#2 for 2 seconds with a 1.5 second onset delay. The timer is linked to a BNC channel. The timer is triggered in the first state, and handled it in the second and third states.

```matlab
sma = NewStateMachine;

sma = SetGlobalTimer(sma, 'TimerID', 2, 'Duration', 2,...
                          'OnsetDelay', 1.5, 'Channel', 'BNC2'); 

sma = AddState(sma, 'Name', 'TimerTrig', ...
    'Timer', 0,...
    'StateChangeConditions', {'Tup', 'Port1Lit'},...
    'OutputActions', {'GlobalTimerTrig', 1});

sma = AddState(sma, 'Name', 'Port1Lit', ...
    'Timer', .25,...
    'StateChangeConditions', {'Tup', 'Port3Lit', 'GlobalTimer1_End', 'exit'},...
    'OutputActions', {'PWM1', 255});

sma = AddState(sma, 'Name', 'Port3Lit', ...
    'Timer', .25,...
    'StateChangeConditions', {'Tup', 'Port1Lit', 'GlobalTimer1_End', 'exit'},...
    'OutputActions', {'PWM3', 255}); 
```

### `SetGlobalCounter()`
**Description**

Sets the threshold and monitored event for one of the 5 global counters. 

- Global counters can count instances of events, and handle when the count exceeds a threshold from any state (by triggering a state change).
- The number of possible counters is a configurable parameter specified in the state machine firmware.
- For more on global counters, see [Using State Matrices](../user-guide/index.md#state-matrix).

**Syntax**

```matlab
NewStateMachine = SetGlobalCounter(StateMachineStruct, CounterNumber, TargetEventName, Threshold)
```

**Parameters**

- StateMachineStruct: The state matrix whose global timer you are setting.
- CounterNumber: The number of the counter you are setting (an integer, 1-5).
- TargetEventName: The name of the event to count (a string; see Bpod Console's magnifying glass)
- Threshold: The number of event instances to count. (an integer).

**Returns**

- A state machine struct, updated with the new global counter setting.

**Example**

This code generates a state machine that sets a global counter to count 5 BNC1High events, resets the count to 0 in the second state, and handles it in the third and fourth states. 

```matlab
sma = NewStateMachine();

sma = SetGlobalCounter(sma, 1, 'BNC1High', 5);

sma = AddState(sma, 'Name', 'State1', ... % BNC1High Events in this state are not counted because the count will be reset.
    'Timer', 1,...
    'StateChangeConditions', {'Tup', 'State2'},...
    'OutputActions', {});

sma = AddState(sma, 'Name', 'State2', ... % This state resets the global counter.
    'Timer', 0,...
    'StateChangeConditions', {'Tup', 'State3'},...
    'OutputActions', {'GlobalCounterReset', 1});

sma = AddState(sma, 'Name', 'State3', ...
    'Timer', 0,...
    'StateChangeConditions', {'Port1In', 'State4', 'GlobalCounter1_End', 'exit'},...
    'OutputActions', {});

sma = AddState(sma, 'Name', 'State4', ...
    'Timer', 0,...
    'StateChangeConditions', {'Port1Out', 'State3', 'GlobalCounter1_End', 'exit'},...
    'OutputActions', {});
```

### `SetCondition()`
**Description**

Sets an input channel condition to handle on entering a state

- Each condition is true if an input channel's state matches the condition's value.
- The number of possible conditions is a configurable parameter specified in the state machine firmware.

**Syntax**
```matlab
NewStateMachine = SetCondition(StateMachineStruct, ConditionNumber, ConditionChannel, ConditionValue)
```

**Parameters**

- StateMachineStruct: The state machine description whose global timer you are setting.
- ConditionNumber: The number of the condition you are setting (an integer).
- ConditionChannel: The name of the input channel attached to the condition.
    - Input channel names are listed in BpodSystem.StateMachineInfo.InputChannelNames
    - The channel can also be a global timer, indicated as 'GlobalTimerN' where N is the index of the global timer.
- ConditionValue: The value of the condition channel if the condition is met (1 = high, 0 = low)
    - If using a global timer, the timer is "high" (1) between its "start" and "end" events, and 0 otherwise.

**Returns**

- A state machine struct, updated with the new condition description.

**Example**

This code generates a state machine with three states. Each state lights up a behavior port. A condition is set to be valid if the IR channel of port2 is high (1). It is then used to skip state 2 if true.

```matlab
sma = NewStateMachine;

sma = SetCondition(sma, 2, 'Port2', 1);

sma = AddState(sma, 'Name', 'Port1Light', ...
    'Timer', 1,...
    'StateChangeConditions', {'Tup', 'Port2Light'},...
    'OutputActions', {'PWM1', 255});

sma = AddState(sma, 'Name', 'Port2Light', ...
    'Timer', 1,...
    'StateChangeConditions', {'Tup', 'Port3Light', 'Condition2', 'Port3Light'},...
    'OutputActions', {'PWM2', 255});

sma = AddState(sma, 'Name', 'Port3Light', ...
    'Timer', 1,...
    'StateChangeConditions', {'Tup', 'exit'},...
    'OutputActions', {'PWM3', 255});
```