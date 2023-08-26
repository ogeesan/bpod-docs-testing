# Function reference
> [!NOTE]
> :construction: This file contains all function references, with each section likely to be moved into its own file into the future.

## Table of Contents
- [Initialization](#initialization)
  - [`Bpod()`](#bpod)
- [BpodSystem fields](#bpodsystem-fields)
  - [Data](#data)
  - [ProtocolSettings](#protocolsettings)
  - [SoftCodeHandlerFunction](#softcodehandlerfunction)
  - [ProtocolFigures](#protocolfigures)
  - [EmulatorMode](#emulatormode)
  - [StateMachineInfo](#statemachineinfo)
  - [Status](#status)
  - [FlexIOConfig](#flexioconfig)
- [BpodSystem functions](#bpodsystem-functions)
  - [`assertModule()`](#assertmodule)
  - [`setStatusLED()`](#setstatusled)
  - [`startAnalogViewer()`](#startanalogviewer)
- [Creating a state machine](#creating-a-state-machine)
  - [`NewStateMachine()`](#newstatemachine)
  - [`AddState()`](#addstate)
  - [`EditState()`](#editstate)
  - [`SetGlobalTimer()`](#setglobaltimer)
  - [`SetGlobalCounter()`](#setglobalcounter)
  - [`SetCondition()`](#setcondition)
- [Running a state machine](#running-a-state-machine)
  - [`SendStateMachine()`](#sendstatemachine)
  - [`RunStateMachine()`](#runstatemachine)
  - [`BpodTrialManager()`](#bpodtrialmanager)
  - [`AddTrialEvents()`](#addtrialevents)
- [Running a protocol](#running-a-protocol)
  - [From the Bpod console GUI:](#from-the-bpod-console-gui)
  - [From the command line:](#from-the-command-line)
    - [`RunProtocol()`](#runprotocol)
- [Data storage](#data-storage)
  - [`SaveBpodSessionData()`](#savebpodsessiondata)
  - [`SaveProtocolSettings()`](#saveprotocolsettings)
  - [`AddFlexIOAnalogData()`](#addflexioanalogdata)
- [General Plugins](#general-plugins)
  - [`BpodParameterGUI()`](#bpodparametergui)
  - [`PsychToolboxSoundServer()`](#psychtoolboxsoundserver)
  - [`PsychToolboxVideoPlayer()`](#psychtoolboxvideoplayer)
  - [`BpodNotebook`](#bpodnotebook)
  - [`SideOutcomePlot()`](#sideoutcomeplot)
  - [`TrialTypeOutcomePlot()`](#trialtypeoutcomeplot)
  - [`StateTiming()`](#statetiming)
- [Serial message seup](#serial-message-seup)
  - [`LoadSerialMessages()`](#loadserialmessages)
  - [`ResetSerialMessages()`](#resetserialmessages)
  - [Implicit serial messages](#implicit-serial-messages)
- [Module \<-\> MATLAB (via USB)](#module---matlab-via-usb)
  - [`I2CMessenger()`](#i2cmessenger)
  - [`BpodAnalogIn()`](#bpodanalogin)
  - [`BpodWavePlayer()`](#bpodwaveplayer)
  - [`BpodAudioPlayer()`](#bpodaudioplayer)
  - [`PulsePalModule()`](#pulsepalmodule)
  - [`DDSModule()`](#ddsmodule)
  - [`RotaryEncoderModule()`](#rotaryencodermodule)
  - [`BpodHiFi()`](#bpodhifi)
  - [`BpodStepperModule()`](#bpodsteppermodule)
- [Module \<-\> MATLAB (via FSM)](#module---matlab-via-fsm)
  - [`ModuleWrite()`](#modulewrite)
  - [`ModuleRead()`](#moduleread)
- [USB Soft Codes, PC --\> FSM](#usb-soft-codes-pc----fsm)
  - [`SendBpodSoftCode()`](#sendbpodsoftcode)
- [Liquid calibration](#liquid-calibration)
  - [`GetValveTimes()`](#getvalvetimes)
- [Updating Bpod](#updating-bpod)
  - [`LoadBpodFirmware()`](#loadbpodfirmware)
  - [`UpdateBpodSoftware()`](#updatebpodsoftware)


## Initialization

### `Bpod()`
**Description**
Initializes Bpod and creates a global object representing the Bpod device (`BpodSystem`) in the base workspace.

- The function automatically searches all available serial ports and finds Bpod if one is connected.
- It then creates a `BpodSystem` object.

## BpodSystem fields
`BpodSystem` exists in the workspace as a [global object](https://mathworks.com/help/matlab/ref/global.html). It has the following fields:

### Data
**Description**

Stores session data as a `struct`.

- Typically stores the output of `AddTrialEvents` and other data added as subfields.
- Can be saved automatically with the `SaveBpodSessionData` function.

**Example**
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
**Description**

Equal to the full path of a soft code handler m-file to use with the current protocol.

**Example**

For another example of a soft code handler, see /Bpod/Examples/Protocols/PsychToolboxSound/SoftCodeHandler_PlaySound.m

```matlab
% Part 1 (in main protocol file): This single-state matrix sends a byte (3) back to the 
% governing computer on state entry by setting a 'SoftCode' in Output Actions.
sma = NewStateMachine();
sma = AddState(sma, 'Name', 'State1', ...
    'Timer', 1,...
    'StateChangeConditions', {'Tup', 'exit'},...
    'OutputActions', {'SoftCode', 3});

% Part 2 (in main protocol file): This code specifies which file will handle the byte when it 
% comes back.
BpodSystem.SoftCodeHandlerFunction = 'SoftCodeHandler_Printout';

% Part 3 (separate file in protocol folder): This code handles the byte by printing it to the
% MATLAB command window.
function SoftCodeHandler_Printout(Byte)
disp(Byte);
```

### ProtocolFigures
**Description**

A struct for handles of figures that display online data.

- Since `BpodSystem` is a global variable, the figure handles in this struct will be available in figure update functions and in the main protocol function.
- All figures in `BpodSystem.ProtocolFigures` will automatically be closed when the protocol ends.

**Example**
This code initializes a new figure to show data, and adds its handle to `BpodSystem.ProtocolFigures`.
At the conclusion of the session it will be closed.
```matlab
BpodSystem.ProtocolFigures.MyPlotFig = figure('Position', [200 200 1000 200],'name','My Plots');
```

### EmulatorMode
**Description**

When no Bpod device is found, the user can launch the Bpod software in emulator mode.

- BpodSystem.EmulatorMode is automatically set to 1 in emulator mode (0 = default, actual hardware).
- Emulator mode is a tool for protocol development; timing data and sound / visual display timing may be imprecise depending on the platform. Protocols developed with the emulator should be validated on hardware prior to any experiment.
- Inputs are read and the hardware state is indicated on the Bpod console while running a state machine.
- When designing a protocol that can be run in emulator mode, block off code that interfaces with unavailable hardware (see example).

**Example**
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
**Description**

Throws an error if a given Bpod module is not present.
- An optional argument specifies whether the module must also be paired with its USB serial port via the USB pairing UI.
- After connecting a new module, you must press the 'refresh' button on the [Bpod console GUI] to make it visible to `assertModule()`.

**Syntax**

`BpodSystem.assertModule(moduleNames, USBPaired)`

**Parameters**

- `moduleNames`: a character array containing the name of the module. A cell array of strings may also be provided to assert multiple modules. Note: The names of connected modules are given in BpodSystem.Modules
- `USBPaired`: optional, an array of 1s and 0s with one value for each module in moduleNames. 
  - 1 = the module must be paired with its USB serial port. 
  - 0 = the module does not have to be paired.

**Return**
- None

**Example**

This code will throw an error if the HiFi or ValveDriver modules are missing.
It will also throw an error if the HiFi module is not paired with its USB serial port.
```matlab
BpodSystem.AssertModule({'HiFi', 'ValveDriver'}, [1 0]);
```

### `setStatusLED()`

### `startAnalogViewer()`

## Creating a state machine

### `NewStateMachine()`

### `AddState()`
**Description**

Adds a state to an existing state machine. 

**Syntax**
```matlab
NewStateMachine = AddState(StateMachineStruct, 'Name', StateName, 'Timer', TimerDuration, 'StateChangeConditions', Conditions, 'OutputActions', Actions)
```
**Parameters**

- StateMachineStruct: The state machine you are adding to. If this is the first state, StateMachineStruct is the output of NewStateMachine().
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

### `SetGlobalTimer()`

### `SetGlobalCounter()`

### `SetCondition()`

## Running a state machine

> **Note:** The sending and running of a state matrix can be done in one of two ways. The "classic" method is to use `SendStateMachine()` and `RunStateMachine()`, where the state machine is suspended during inter-trial MATLAB updates. The "TrialManager" method is to use `BpodTrialManager()` which allows updates in parallel.

### `SendStateMachine()`

### `RunStateMachine()`

### `BpodTrialManager()`

### `AddTrialEvents()`
**Description**

Packages raw events returned from RunStateMatrix() into a session data struct. 

State codes and event codes are decoded so the session data is human-readable

**Syntax**
```matlab
UpdatedSessionData = AddTrialEvents(PreviousSessionData, RawEvents)
```

**Parameters**

- PreviousSessionData: The session data struct (or an empty struct for the first trial).
- RawEvents: The struct of raw events returned from RunStateMatrix().

**Return**

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

**Example**
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

### `AddFlexIOAnalogData()`

## General Plugins
### `BpodParameterGUI()`
### `PsychToolboxSoundServer()`
### `PsychToolboxVideoPlayer()`
### `BpodNotebook`
### `SideOutcomePlot()`
### `TrialTypeOutcomePlot()`
### `StateTiming()`

## Serial message seup
### `LoadSerialMessages()`
### `ResetSerialMessages()`
### Implicit serial messages

## Module <-> MATLAB (via USB)
### `I2CMessenger()`
### `BpodAnalogIn()`
### `BpodWavePlayer()`
### `BpodAudioPlayer()`
### `PulsePalModule()`
### `DDSModule()`
### `RotaryEncoderModule()`
### `BpodHiFi()`
### `BpodStepperModule()`

## Module <-> MATLAB (via FSM)
### `ModuleWrite()`
### `ModuleRead()`

## USB Soft Codes, PC --> FSM
### `SendBpodSoftCode()`

## Liquid calibration
### `GetValveTimes()`

## Updating Bpod
### `LoadBpodFirmware()`
### `UpdateBpodSoftware()`