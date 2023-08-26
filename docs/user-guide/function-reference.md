# Function reference


## Initialization

### `Bpod()`
#### Description
Initializes Bpod and creates a global object representing the Bpod device (`BpodSystem`) in the base workspace.

- The function automatically searches all available serial ports and finds Bpod if one is connected.
- It then creates a `BpodSystem` object.

## BpodSystem fields
`BpodSystem` exists in the workspace as a [global object](https://mathworks.com/help/matlab/ref/global.html). It has the following fields:

### Data
#### Description

Stores session data as a `struct`.

- Typically stores the output of `AddTrialEvents` and other data added as subfields.
- Can be saved automatically with the `SaveBpodSessionData` function.

#### Example
This code sends "sma" (an existing state matrix) to Bpod, runs it 10 times, and packages the raw events for analysis. 
It then saves them to disk on each trial. 

```matlab
SendStateMatrix(sma);
for i = 1:10
    RawEvents = RunStateMatrix;
    BpodSystem.Data = AddTrialEvents(BpodSystem.Data, RawEvents);
    BpodSystem.Data.arbitrarydata = rand(1,10);  % you can add anything into the data
    SaveBpodSessionData;
end
```

### ProtocolSettings

### SoftCodeHandlerFunction
#### Description

Equal to the full path of a soft code handler m-file to use with the current protocol.

#### Example

For another example of a soft code handler, see /Bpod/Examples/Protocols/PsychToolboxSound/SoftCodeHandler_PlaySound.m

```matlab
% Part 1 (in main protocol file): This single-state matrix sends a byte (3) back to the governing computer on state entry by setting a 'SoftCode' in Output Actions.
sma = NewStateMachine();
sma = AddState(sma, 'Name', 'State1', ...
    'Timer', 1,...
    'StateChangeConditions', {'Tup', 'exit'},...
    'OutputActions', {'SoftCode', 3});

% Part 2 (in main protocol file): This code specifies which file will handle the byte when it comes back.
BpodSystem.SoftCodeHandlerFunction = 'SoftCodeHandler_Printout';

% Part 3 (separate file in protocol folder): This code handles the byte by printing it to the MATLAB command window.
function SoftCodeHandler_Printout(Byte)
disp(Byte);
```

### ProtocolFigures
#### Description

A struct for handles of figures that display online data.

- Since `BpodSystem` is a global variable, the figure handles in this struct will be available in figure update functions and in the main protocol function.
- All figures in `BpodSystem.ProtocolFigures` will automatically be closed when the protocol ends.

#### Example
This code initializes a new figure to show data, and adds its handle to `BpodSystem.ProtocolFigures`.
At the conclusion of the session it will be closed.
```matlab
BpodSystem.ProtocolFigures.MyPlotFig = figure('Position', [200 200 1000 200],'name','My Plots');
```

### EmulatorMode
#### Description

When no Bpod device is found, the user can launch the Bpod software in emulator mode.

- BpodSystem.EmulatorMode is automatically set to 1 in emulator mode (0 = default, actual hardware).
- Emulator mode is a tool for protocol development; timing data and sound / visual display timing may be imprecise depending on the platform. Protocols developed with the emulator should be validated on hardware prior to any experiment.
- Inputs are read and the hardware state is indicated on the Bpod console while running a state machine.
- When designing a protocol that can be run in emulator mode, block off code that interfaces with unavailable hardware (see example).

#### Example
This code connects to a remote TCP server using the SerialEthernet plugin, only if NOT in emulator mode.
```matlab
if BpodSystem.EmulatorMode == 0
    SerialEthernet('Init', 'COM65'); % Set this to the correct COM port for Arduino Leonardo
    pause(1);
    RemoteIP = [192 168 0 104]; RemotePort = 3336;
    SerialEthernet('Connect', RemoteIP , RemotePort);
else
    disp('Fake-connected to a fake TCP server')
end
```

### StateMachineInfo

### Status

### FlexIOConfig

## BpodSystem functions

### `assertModule()`

### `setStatusLED()`

### `startAnalogViewer()`

## Creating a state machine

### `NewStateMachine()`

### `AddState()`

### `EditState()`

### `SetGlobalTimer()`

### `SetGlobalCounter()`

### `SetCondition()`

## Running a state machine

> **Note:** The sending and running of a state matrix can be done in one of two ways. The "classic" method is to use `SendStateMachine()` and `RunStateMachine()`, where the state machine is suspended during inter-trial MATLAB updates. The "TrialManager" method is to use `BpodTrialManager()` which allows updates in parallel.

### `SendStateMachine()`

### `RunStateMachine()`

### `BpodTrialManager()`

### `AddTrialEvents()`
#### Description

Packages raw events returned from RunStateMatrix() into a session data struct. 

State codes and event codes are decoded so the session data is human-readable

#### Syntax
```matlab
UpdatedSessionData = AddTrialEvents(PreviousSessionData, RawEvents)
```

#### Parameters

- PreviousSessionData: The session data struct (or an empty struct for the first trial).
- RawEvents: The struct of raw events returned from RunStateMatrix().

#### Returns

* UpdatedSessionData: A struct contatining data from all trials. It has the following fields:
** nTrials: The number of trials that have been added
** RawEvents: A struct containing re-organized state and event timestamps for each trial, labeled so they are human-readable.
*** RawEvents.Trial{n} has two sub-fields

, populated depending on what occurred during the trial:
**** States: the times when each state was entered and exited (in seconds). States that were not visited show NaN.
**** Events: the times when each event was detected (in seconds).
** RawData: A struct containing three fields:
*** OriginalStateNamesByNumber: A cell array of strings listing the names of each state (for matching states up with state numbers sent via the sync port)
**** Note that state numbers are assigned automatically depending on the order of states added with AddState - so if you programmed your protocol to add states (or refer to not-yet-added states) in a different order on each trial, state numbers on each trial may be different.
*** OriginalStateData: A cell array containing the original state codes returned from RunStateMatrix on each trial
*** OriginalEventData: A cell array containing the original event codes returned from RunStateMatrix on each trial
** TrialStartTimestamp: The time when each trial started (measured from the last time Bpod was initialized)
** Settings: A cell array of strings containing the settings struct as it existed when each trial's state matrix was sent.

#### Example
This code sends "sma" (an existing state matrix) to Bpod, runs it 10 times, and packages the raw events for analysis. 
```matlab
SendStateMatrix(sma);
SessionData = struct;  % equivalent to BpodSystem.Data 
for i = 1:10
    RawEvents = RunStateMatrix;
    SessionData = AddTrialEvents(SessionData, RawEvents);
end
```

## Running a protocol

### From the Bpod console GUI:

Click 'Play' for launch manager.

### From the command line:

#### `RunProtocol()`

## Data storage

### `SaveBpodSessionData()`
Saves information in `BpodSystem.Data` to disk.

### `SaveProtocolSettings()`
