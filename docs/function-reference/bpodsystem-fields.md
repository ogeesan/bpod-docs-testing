# BpodSystem fields
`BpodSystem` exists in the workspace as a [global object](https://mathworks.com/help/matlab/ref/global.html). It has the following fields:

### Data
**Description**

Stores session data as a `struct`.

- Typically stores the output of [`AddTrialEvents`](#addtrialevents) and other data added as subfields.
- Can be saved automatically with the `SaveBpodSessionData` function.

**Example**

This code sends "sma" (an existing state matrix) to Bpod, runs it 10 times, and packages the raw events for analysis. 
It then saves them to disk on each trial. 

```matlab
SendStateMatrix(sma);
for i = 1:10
    RawEvents = RunStateMatrix;
    BpodSystem.Data = AddTrialEvents(BpodSystem.Data, RawEvents);
    BpodSystem.Data.arbitrarydata = rand(1,10); % Anything can be added
    SaveBpodSessionData;
end
```

### ProtocolSettings
**Description**

Saves the struct BpodSystem.ProtocolSettings to disk.

- The settings in BpodSystem.ProtocolSettings are saved over the file targeted when selecting settings in the launch manager.

**Syntax**

`SaveProtocolSettings()`

**Parameters**

- None

**Returns**

- None

**Example**

This code checks the selected settings file to see if it has been populated - and if not, adds default values.
```matlab
S = BpodSystem.ProtocolSettings; % Load settings chosen in launch manager into current workspace as a struct called S

if isempty(fieldnames(S))   % If settings file was an empty struct,
                            % populate struct with default settings
    S.SoundDuration = 0.5;  % Duration of sound (s)
    S.RiotDuration = 7.5;   % Duration of riot(s)
end

BpodSystem.ProtocolSettings = S;
SaveProtocolSettings; % Saves the default settings to the disk location selected in the launch manager.
```

### SoftCodeHandlerFunction
**Description**

Equal to the full path of a soft code handler m-file to use with the current protocol.

**Example**

For another example of a soft code handler, see [Examples/Protocols/](https://github.com/sanworks/Bpod_Gen2/tree/master/Examples/Protocols). PsychToolboxSound/ contains complete examples of a protocol state matrix construction using SoftCodeHandler_PlaySound.m

```matlab
% 1. (main protocol file): This single-state matrix sends a byte (3) back to the
% governing computer on state entry by setting a 'SoftCode' in Output Actions.
sma = NewStateMachine();
sma = AddState(sma, 'Name', 'State1', ...
    'Timer', 1,...
    'StateChangeConditions', {'Tup', 'exit'},...
    'OutputActions', {'SoftCode', 3});

% Part 2 (in main protocol file): This code specifies which file will handle the
% byte when it comes back.
BpodSystem.SoftCodeHandlerFunction = 'SoftCodeHandler_Printout';

% Part 3 (separate file in protocol folder): This code handles the byte by 
% printing it to the MATLAB command window.
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
BpodSystem.ProtocolFigures.MyPlotFig = figure('Position', [200 200 1000 200],...
                                              'name', 'My Plots');
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
    SerialEthernet('Init', 'COM65'); % Set COM port for Arduino Leonardo
    pause(1);
    RemoteIP = [192 168 0 104]; RemotePort = 3336;
    SerialEthernet('Connect', RemoteIP , RemotePort);
else
    disp('Fake-connected to a fake TCP server')
end
```

### StateMachineInfo
**Description**

A struct containing human-readable information about the currently connected state machine hardware.

Different state machine devices will populate this struct differently, resulting in different possible events and outputs.

It has the following fields:

- nEvents: the total number of unique events monitored from input channels
- EventNames: A character string naming each event 
    - Each event is a state of an input channel
    - Use these with the `AddState()` function when writing state machines.
- InputChannelNames: A character string naming each input channel
    - Each channel is capable of generating multiple states
    - Use these with the `SetCondition()` function
- nOutputChannels: the total number of unique output channels
- OutputChannelNames: A character string naming each output channel
- Use these with the `AddState()` and `SetGlobalTimer()` functions
- MaxStates: The maximum number of states supported by the connected state machine

**Example**

`>> BpodSystem.StateMachineInfo.OutputChannelNames`
```
ans = 

  Columns 1 through 7
    'Serial1'    'Serial2'    'Serial3'    'SoftCode'    'ValveState'    'BNC1'    'BNC2'

  Columns 8 through 15
    'Wire1'    'Wire2'    'Wire3'    'PWM1'    'PWM2'    'PWM3'    'PWM4'    'PWM5'

  Columns 16 through 20
    'PWM6'    'PWM7'    'PWM8'    'GlobalTimerTrig'    'GlobalTimerCancel'

  Column 21
    'GlobalCounterReset'
```
### Status
**Description**

`BpodSystem.Status` is a struct populated with system status variables. Most subfields are used internally by the GUI. 

`BpodSystem.Status.BeingUsed` indicates whether a behavior session is running.

- BeingUsed is set to 1 when:
    - A state machine is run using `RunStateMachine()` or `BpodTrialManager`
    - A session is started with `RunProtocol()` or the Launch Manager
- BeingUsed is set to 0 when:
    - The 'Stop' or 'Pause' buttons are pressed on the Bpod Console GUI
    - The session is stopped with `RunProtocol('Stop')`

**Example**

When the user ends the session, this code calls a user-defined cleanup function before exiting

```matlab
if BpodSystem.Status.BeingUsed == 0
    break
end
```

### FlexIOConfig
**Description**

FlexIOConfig is a struct used to configure Flex I/O channels. Flex I/O channels can each be configured as:

- Digital Input
- Digital Output
- Analog Input (12-bit, 0-5V)
- Analog Output (12-bit, 0-5V)
- High Impedance (channel disabled)

Other configuration settings in FlexIOConfig concern channel(s) configured as analog input:

- Sampling Rate
- ADC measurements per sample
- Threshold configuration

FlexIOconfig is attached to a callback function. Any changes to the struct are automatically sent to the state machine.

**Properties**

- **channelTypes**: An array containing the Flex I/O configuration. The array must specify a configuration for each Flex I/O channel.
    - Configuration values
          - 0: Digital Input
          - 1: Digital Output
          - 2: Analog Input
          - 3: Analog Output
          - 4: Disabled (High Impedance)
    - On calling FlexIOConfig.channelTypes(), the composition of valid state machine events and output actions will be updated to match the requested channel types.
    - Note: a GUI is provided to set the Flex I/O configuration manually, from the settings menu on the Bpod Console. If set from the GUI, the configuration will be loaded automatically to the device each time Bpod is started. FlexIOconfig.channelTypes is useful for reconfiguring the Flex I/O channels when launching an experimental protocol.
- **analogSamplingRate**: The rate of sample acquisition for all Flex I/O channels configured as analog input
    - Units: Hz
    - Range: [1, 1000]
- **nReadsPerSample**: The number of ADC reads to average for each sample acquired. ADC reads are measured consecutively, and each read takes ~2 microseconds. Increasing reads per sample reduces the effect of high-frequency noise.
    - Range: [1, 4]
- **threshold1, threshold2**: A 1x4 array specifying an event threshold for each channel. Each Flex I/O channel has two configurable thresholds, contained in threshold1 and threshold2 respectively.
- Unit  s: Volts
- **polarity1, polarity2**: A 1x4 array specifying the polarity of each channel's threshold. 
    - 0: An event is generated when voltage is above the threshold
    - 1: An event is generated when voltage is below the threshold
- **thresholdMode**: A 1x4 array specifying the mode of each threshold. All thresholds are disabled when reached.
    - 0: Thresholds must be manually re-enabled using the 'AnalogThreshEnable' output action.
    - 1: For a single Flex I/O channel, crossing threshold 1 enables threshold 2. Crossing threshold 2 enables threshold 1.

**Examples**

This code configures channel 1 as an analog input and channel 2 as an analog output. All other channels are configured as high impedance.
```matlab
BpodSystem.FlexIOConfig.channelTypes = [2 3 4 4];
```


This code configures channel 1's two thresholds to 4V (threshold 1) and 2V (threshold 2). Threshold 2 is set to detect voltage below 2V. Thresholds are set to enable each other when crossed. This setup can be used to generate an event on the rising and falling phases of a cyclic signal.

```matlab
Chan = 1;
BpodSystem.FlexIOConfig.channelTypes(Chan) = 2; % Set Flex I/O Ch1 as analog input
BpodSystem.FlexIOConfig.threshold1(Chan) = 4;
BpodSystem.FlexIOConfig.threshold2(Chan) = 2;
BpodSystem.FlexIOConfig.polarity1(Chan) = 0;
BpodSystem.FlexIOConfig.polarity2(Chan) = 1;
BpodSystem.FlexIOConfig.thresholdMode(Chan) = 1;
```

This code configures the sampling rate and number of ADC reads per sample for all analog inputs.
```matlab
BpodSystem.FlexIOConfig.analogSamplingRate = 100; % Set Flex I/O analog input sampling rate to 100Hz
BpodSystem.FlexIOConfig.nReadsPerSample = 2; % Set Flex I/O analog input to 2 averaged ADC reads per sample
```

A behavior protocol demonstrating use of Flex I/O channels for analog acquisition is given in the Bpod_Gen2 repository [here](https://www.google.com/url?q=https%3A%2F%2Fgithub.com%2Fsanworks%2FBpod_Gen2%2Fblob%2Fmaster%2FExamples%2FProtocols%2FLight%2FFlexIOAnalogLight2AFC%2FFlexIOAnalogLight2AFC.m&sa=D&sntz=1&usg=AOvVaw3Hhg52UiVwwLIqlC_2oK8a).

!!! note
    Flex I/O channels were introduced with Bpod State Machine 2+ in 2022, and may also exist on newer models.