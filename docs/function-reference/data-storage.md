# Data storage

### `SaveBpodSessionData()`
**Description**

Saves the struct BpodSystem.Data to a data file.

- The file name is determined automatically based on selections made in the launch manager, and the current time.
- The file is formatted as a MATLAB .mat file.

**Syntax**

```matlab
SaveBpodSessionData()
```

**Parameters**

- None

**Returns**

- None

**Example**

This code sends "sma" (an existing state matrix) to Bpod, runs it 10 times, and packages the raw events for analysis. On each trial, the session data is saved to disk.

```matlab
SendStateMatrix(sma);

for i = 1:10
    RawEvents = RunStateMachine;
    BpodSystem.Data = AddTrialEvents(BpodSystem.Data, RawEvents);
    SaveBpodSessionData;
end
```

### `SaveProtocolSettings()`
**Description**

Saves the struct BpodSystem.ProtocolSettings to disk.

- The settings in BpodSystem.ProtocolSettings are saved over the file targeted when selecting settings in the launch manager.

**Syntax**

```matlab
SaveProtocolSettings()
```

**Parameters**

- None

**Returns**

- None

**Example**

This code checks the selected settings file to see if it has been populated - and if not, adds default values.

```matlab
S = BpodSystem.ProtocolSettings; % Load settings chosen in launch manager into current workspace as a struct called S

if isempty(fieldnames(S))  % If settings file was an empty struct, populate struct with default settings
    S.SoundDuration = 0.5; % Duration of sound (s)
    S.RiotDuration = 7.5; % Duration of riot(s)
end

BpodSystem.ProtocolSettings = S;
SaveProtocolSettings; % Saves the default settings to the disk location selected in the launch manager.
```

### `AddFlexIOAnalogData()`
**Description**

Reads the current Flex I/O analog data file into memory, and adds it to a Bpod behavior data structure.

Adding analog data is a long operation and is typically run once at the end of each session.

This function will only work with state machine r2+ or other models with Flex I/O channels.

**Syntax**

```matlab
BehaviorDataOut = AddFlexIOAnalogData(BehaviorDataIn, [sampleFormat], [addSamplesByTrial]) 
% Note: [ ] indicates an optional argument
```

**Parameters**

- **BehaviorDataIn**: a Bpod behavior data structure. 
    - During a session, use BpodSystem.Data. 
    - To add analog data post-hoc (e.g. for a crashed session) BehaviorDataIn is the data structure loaded to the workspace
- **sampleFormat**: a string indicating the format to import
    - 'Volts' (default): Samples are imported as double type (8 bytes per sample). Units = volts.
    - 'Bits': Samples are imported as uint16 type in range 0-4095 (2 bytes per sample). Bits represent volts in range 0-5.
    - Note: Store samples as bits for smaller data files, and convert to volts at analysis time: 
          - Volts = (double(Bits)/4095)*5
- **addSamplesByTrial**:
    - 1 = add a cell array with a cell for each experimental trial, where each cell contains all samples acquired during that trial. This can also be done by user code at analysis time to save disk space.
    - 0 = Do not add trial-aligned duplicate data

**Returns**

- BehaviorDataOut: the Bpod behavior data structure passed into the function, with added analog data

**Examples**

This code will import the current Flex I/O analog data into the primary behavior data structure at the end of a behavior session. Samples are stored as bits to save space, and a cell array of samples per trial is added.

```matlab
if BpodSystem.Status.BeingUsed == 0
   BpodSystem.Data = AddFlexIOAnalogData(BpodSystem.Data, 'Bits', 1); Adds FlexI/O analog data to BpodSystem.Data
   SaveBpodSessionData; % Saves BpodSystem.Data to the current data file
   break
end
```

This code will add previously acquired analog data offline. Note that his must be done on the same PC that acquired the data, and assumes that the data has not been moved from its original location on the disk.
```matlab
load MyDataFile;  % MyDataFile is a .mat file of saved Bpod session data. A struct called SessionData is  		 
                  % created in the local workspace.

SessionData = AddFlexIOAnalogData(SessionData, 'Volts'); % Import the analog data as volts
```