# Running a state machine

!!!note
    The sending and running of a state matrix can be done in one of two ways. The "classic" method is to use `SendStateMachine()` and `RunStateMachine()`, where the state machine is suspended during inter-trial MATLAB updates. The "TrialManager" method is to use `BpodTrialManager()` which allows updates in parallel. Protocols based on both RunStateMachine() and BpodTrialManager() will be supported for the indefinite future.


### `SendStateMachine()`
**Description**

Sends a state machine description to a Bpod state machine device. 

- The state machine description is checked first for sanity, and an error is thrown if parameters are invalid, states were referred to but not subsequently defined, etc.
- The function returns an acknowledgement that the data was properly formatted and cued for transmission. 
    - SuccessfulTransmission is verified internally on the next call to RunStateMachine().

**Syntax**

```matlab
Acknowledged = SendStateMachine(StateMachineStruct)
```

**Parameters**

- StateMachineStruct: The state machine struct to be sent.
    - Note: state machine structs are created with NewStateMachine() and states are added with AddState().

**Returns**

- 1 if cued for transmission successfully, 0 if not.

**Example**

This code generates a simple state machine and sends it to Bpod. 

```matlab
sma = NewStateMachine();

sma = AddState(sma, 'Name', 'State1', ...
    'Timer', 1,...
    'StateChangeConditions', {'Tup', 'exit'},...
    'OutputActions', {});

SendStateMachine(sma);
```

### `RunStateMachine()`
**Description**

Runs the last state machine that was loaded to Bpod with `SendStateMachine()`, and returns events, state transitions and timestamps after the state machine reaches the exit state. 

- This command will block MATLAB execution until data is returned. Scripts to be run during a trial can be added as SoftCodeHandlers and triggered from the state machine.
- The events returned are in a raw format (not human-readable), and should be subsequently packaged into your session data with the `AddTrialEvents()` function.

**Syntax**
```matlab
RawEvents = RunStateMachine()
```

**Parameters**

- None

**Returns**

A struct with the following fields:

- States: a vector listing the integer codes of states visited, in sequential order.
    - The list of codes can be found in BpodSystem.StateMatrix.StateNames
- Events: a vector listing the byte codes of events captured.
    - The list of events can be found in BpodSystem.StateMachineInfo.EventNames
- StateTimestamps: a vector listing the entry times of each state listed in the "States" field (in seconds from matrix start).
- EventTimestamps: a vector listing the times of each event listed in the "Events" field (in seconds from matrix start).
- TrialStartTimestamp: The time the matrix started (in seconds from session start).

**Example**

This code sends "sma" (a state machine description) to Bpod, runs it 10 times, and packages the raw events for analysis. 

```matlab
SendStateMachine(sma);

SessionData = struct;
for i = 1:10
    RawEvents = RunStateMachine;
    SessionData = AddTrialEvents(SessionData, RawEvents);
end
```

### `BpodTrialManager()`
!!! note
    Former syntax: `TrialManagerObject()`

**Description**

In earlier releases of Bpod, the function RunStateMachine() was necessary to run each trial. RunStateMachine() blocks the MATLAB command line for the entire trial, so MATLAB cannot use in-trial time to prepare the next trial's state machine, update online plots or save data. MATLAB must do these things between trials, resulting in a "dead-time" period where the state machine is not recording events or controlling the environment. When MATLAB-side code is efficient, this dead-time is often acceptable - the example protocols included with Bpod have ~15ms of dead time on a modern processor, which occurs during the subject's motion to initiate the next trial. However if your protocol requires complex online analysis or other time-costly computer-side processing, the BpodTrialManager class provides a way for most of this processing to occur in parallel with the trial. 

A few points to consider before using TrialManager:

- Because the state machine uses a single-core Arduino processor, a small dead-time is still necessary for inter-trial data transmission. This dead time is on the order of 200 microseconds, and depends on how many states and state transitions are defined in the next trial. 
- With TrialManager, the code for an experimental protocol becomes slightly more complicated (e.g. this versus this).
- Because TrialManager requires Bpod's governing computer to multitask instead of simply checking for incoming bytes in a loop, the computer may process soft codes with increased latency and jitter. 


**Syntax**

```matlab
TrialManager = BpodTrialManager()
```

**Object Fields**

- Timer
    - A MATLAB [timer object](https://www.google.com/url?q=https%3A%2F%2Fwww.mathworks.com%2Fhelp%2Fmatlab%2Fref%2Ftimer-class.html&sa=D&sntz=1&usg=AOvVaw06imv4yvjQCxbZkAectj-k), used to scan for incoming bytes from the state machine.
    - By default, the timer runs at 1kHz.

**Object Functions**

- **startTrial**(StateMatrix)
    - Sends the next trial's state matrix to the Bpod state machine device, and immediately begins running the trial. 
    - StateMatrix = a valid state machine definition, created with `AddState()`.
    - This function is non-blocking; after state matrix transmission is complete, MATLAB executes the next line of code in your protocol, while the trial proceeds in parallel.
    - In the background, a call to startTrial starts TrialManager's MATLAB timer, which checks constantly for new incoming bytes from the state machine.
- currentTrialEvents = **getCurrentEvents**(TriggerStates)
    - This is an optional function that stalls MATLAB until a specified trigger state is reached, and then returns all states visited and events captured up to that point in the trial. This can be useful for computing the next trial's state machine while the current trial is still running in an adaptive task (e.g. a task with an anti-bias algorithm).
    - TriggerStates = a cell array of strings specifying the names of trigger states, any of which will trigger the current states and events to be returned.
    - currentTrialEvents is a struct with 3 fields:
          - StatesVisited = cell array of strings listing names of states visited, in order of their occurrence
          - EventsCaptured = cell array of strings listing names of events captured, in order of their occurrence
          - RawData = a struct with numerical codes for the states visited and events captured
- RawEvents = **getTrialData**()
    - This function stalls until the trial is complete, then retrieves the trial data.
    - It should  be called after the next trial's state machine is computed and sent, plots are updated, and data is saved.
    - RawEvents is a struct with raw trial data, formatted exactly like the output of `RunStateMachine()`

**Cleanup**

- The TrialManager object and its associated timer object are cleared when you end the protocol.

**Examples**

1. An example visual 2AFC protocol using TrialManagerObject is included in [/Examples/Protocols/Light/Light2AFC_TrialManager](https://www.google.com/url?q=https%3A%2F%2Fgithub.com%2Fsanworks%2FBpod_Gen2%2Fblob%2Fmaster%2FExamples%2FProtocols%2FLight%2FLight2AFC_TrialManager%2FLight2AFC_TrialManager.m&sa=D&sntz=1&usg=AOvVaw2rCsUNZ2YyY8B8hrF22tIl).

For comparison, [/Light2AFC](https://www.google.com/url?q=https%3A%2F%2Fgithub.com%2Fsanworks%2FBpod_Gen2%2Fblob%2Fmaster%2FExamples%2FProtocols%2FLight%2FLight2AFC%2FLight2AFC.m&sa=D&sntz=1&usg=AOvVaw313gEKtjEgDGW4Gn1Cczdr) is an earlier protocol with ~identical functionality, programmed with `RunStateMachine()`.

2. Template for simple protocol setup (the task does not use adaptive contingencies, so `TrialManager.getCurrentEvents()` is not used)

```matlab
function myProtocol % Main protocol file, runs once when session is launched
    global BpodSystem % Import the BpodSystem object (used here to detect when the user ends the protocol)
    nTrials = 1000; % Number of trials in session

    TrialManager = BpodTrialManager; % Create trial manager object
    sma = prepareStateMachine; % Prepare first trial's state machine (see function below)
    TrialManager.startTrial(sma); % Start first trial

    for i = 1:nTrials
        sma = prepareStateMachine; % Prepare next trial's state machine
        RawEvents = TrialManager.getTrialData; % Hangs here until trial end, then returns the trial's raw data
        % // Code to update Bpod modules with the next trial's parameters (if necessary) goes here.
        if BpodSystem.Status.BeingUsed == 0; break; end % If user hit console "stop" button, end session 
        TrialManager.startTrial(sma); % Start next trial's state machine
        % // Code to compute online behavior metrics goes here.
        % // Code to update online plots goes here.
        % // Code to format and save data goes here.
    end
end

function sma = prepareStateMachine
    sma = NewStateMatrix();
    sma = AddState(sma, 'Name', 'MyRandomDelay', ...
        'Timer', ceil(rand*1000)/1000,...
        'StateChangeConditions', {'Tup', 'exit'},...
        'OutputActions', {});
end
```


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

- UpdatedSessionData: A struct containing data from all trials. It has the following fields:
    - nTrials: The number of trials that have been added
    - RawEvents: A struct containing re-organized state and event timestamps for each trial, labeled so they are human-readable.
          - RawEvents.Trial{n} has two sub-fields, populated depending on what occurred during the trial:
                  - States: the times when each state was entered and exited (in seconds). States that were not visited show NaN.
                  - Events: the times when each event was detected (in seconds).
    - RawData: A struct containing three fields:
          - OriginalStateNamesByNumber: A cell array of strings listing the names of each state (for matching states up with state numbers sent via the sync port)
                  - Note that state numbers are assigned automatically depending on the order of states added with AddState - so if you programmed your protocol to add states (or refer to not-yet-added states) in a different order on each trial, state numbers on each trial may be different.
          - OriginalStateData: A cell array containing the original state codes returned from RunStateMatrix on each trial
          - OriginalEventData: A cell array containing the original event codes returned from RunStateMatrix on each trial
    - TrialStartTimestamp: The time when each trial started (measured from the last time Bpod was initialized)
    - Settings: A cell array of strings containing the settings struct as it existed when each trial's state matrix was sent.


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