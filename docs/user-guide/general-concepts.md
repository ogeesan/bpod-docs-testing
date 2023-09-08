---
icon: material/state-machine
---
# General concepts

## Bpod files


```
Bpod Local/ # Contains user-specific files like protcols, calibrations, saved data
    Calibration files/
    Data/
    Protocols/
    Settings/    # Bpod saves 
Bpod_Gen2/    # Contains files required for Bpod to run
```

!!! note
    The expected location for the 'Bpod Local' folder is in same folder as 'Bpod_Gen2' is located.

## Bpod Console
The Bpod Console will appear when `Bpod` is launched and the device successfully connects:
![Alt text](../images/bpod-console.png)

The exact buttons displayed on the console will reflect channels and features available on your state machine version.

The console is your starting point for running protocols with Bpod. It has 5 sections:

- Live Info
    - Displays current and previous states while running an experimental trial
    - Displays the last event recorded during a trial
    - Displays the start time of the current trial in the session
    - Displays the status of the USB serial connection - Idle or Transfer
- Manual Override
    - This section contains an array of tabs. Each tab selects a panel of - override controls for either the state machine, or a connected module.
    - If no module is connected, the tab will display "Serial N", where N is - the number of the physical port (see Serial3 in image above)
    - If a module is connected, it displays as ModuleNameN, where N is the Nth - instance of the module type found.
        - For instance, two SNES modules on module ports 2 and 3 would appear in - separate tabs as SNES1 and SNES2. 
    - The state machine panel provides buttons to override behavior ports, BNC - and Wire interfaces.
    - The module panel defaults to a serial terminal, which exchanges data - between the state machine and the selected module.
    - Custom override panels for modules can be created, and stored in /Bpod/- Functions/Override Panels/
    - The number of panels shown will depend on the configuration of the state - machine hardware and firmware.
- Config
    - Contains four buttons: Module Refresh, Settings, Module USB configuration, System Properties
    - Module Refresh (top-left) 
        - Requests a self-description from connected modules
        - Updates the state machine's list of valid events and outputs. 
        - Displays each module in a tab in the Manual Override section of the GUI
    - Settings (top-right)
        - Launches a menu to configure Bpod settings:
        - Liquid Calibration
        - Audio Calibration
        - Bonsai (TCP/IP) configuration
        - Behavior port enable/disable
        - Flex I/O channel configuration
        - Sync line configuration
        - Data/settings path configuration
    - Module USB configuration (bottom-left)
        - Launches a UI for pairing connected modules with available USB ports
        The resulting paired USB ports are stored in BpodSystem.ModuleUSB.(module - name)
            - i.e. BpodSystem.ModuleUSB.WavePlayer1 could contain the value 'COM4', for - use in your protocol.
    - System Properties (bottom-right)
        - Launches a panel that displays details of the state machine and its - modules:
            - State machine firmware and hardware versions
            - List of the state machine's onboard hardware and functions
            - List of connected modules containing:
                - Name
                - Firmware version
                - Paired USB port
            - List of valid event names for the state machine assembler
            - List of valid output action names for the state machine assembler
    - Clicking the "Module Refresh" button (above) will update the info panel
- Session
    - Session contains two buttons:
        - Play/Pause (top)
            - If idle, opens the launch manager.
            - If running a protocol, schedules a pause after the current trial ends.
        - Stop (bottom)
            - Stops a running protocol, dropping the unfinished trial's data.
- Help
    - A help button on the console's top-right launches a web browser to view this wiki.


**Additional information**

- While the console is open, the state machine's indicator LED will glow green. This indicates that the state machine is ready to communicate with the Bpod software. 
- If you close the Bpod console, the state machine's indicator will glow blue, to indicate that it is disconnected from the program. Running the Bpod command while the console window is open results in an error.
- The console may contain some channels that appear grayed out. These channels are not available on the state machine you have connected.
<!-- - Different state machine models are supported. For instance, the pocket state machine generates the following console: image is not included in the original wiki-->

## Launch manager
The Launch Manager appears when a protocol is run from the Bpod Console.

![Alt text](../images/launch-manager.png)

- **Protocol panel**
    - Lists the protocols in /Bpod Local/Protocols/
    - Folders appear with angle brackets: <MyFolder>. Double click the folder to navigate.
    - Selecting a protocol will display the subjects registered for the protocol in the subject panel.
    - Clicking "+" creates a new protocol from a blank template.
    - Clicking "-" deletes a protocol
    - Clicking the "edit" icon launches the protocol's main .m file in the MATLAB editor.
- **Subject panel**
    - Lists the test subjects registered for the selected protocol. 
    - Clicking "+" adds a test subject to the list. It also creates a folder for the test subject in /Bpod Local/Data/
    - Clicking "-" removes the test subject's folder and all of the data and settings within it.
    - You will be warned before this happens, and forced to click two check-boxes confirming the deletion.
- **Settings panel**
    - Lists the settings files in /Bpod Local/Data/TestSubject/Protocol/Settings/
    - Each settings file is a .mat file, containing a struct with task parameters.
    - Clicking "+" adds a .mat file containing an empty struct to the selected subject's settings folder.
    - Clicking "-" removes the selected .mat file from the selected subject's settings folder.
    - Clicking "edit" loads the struct of the selected .mat file into the local workspace. You can change parameters in the struct, and then run `SaveProtocolSettings(ProtocolSettings)` to save it to its original location.
    - Clicking "import" allows you to copy a settings file to the list (from another test subject's folder).
- **Launch button**
    - Once you are ready to start the protocol, click "Launch".
    - This will run your protocol's main .m file
    - The settings struct you selected will be available in your protocol workspace as: `BpodSystem.ProtocolSettings`

!!! note
    The path to the folder may be different on your system. You can select where settings, protocols and data are stored from the Bpod console's settings menu.

## State matrix
### What's that?

Each Bpod trial is programmed as a [virtual finite state machine](http://www.google.com/url?q=http%3A%2F%2Fen.wikipedia.org%2Fwiki%2FVirtual_finite-state_machine&sa=D&sntz=1&usg=AOvVaw1DcMhaObbuC0WOA7hhGMCg). This ensures precise timing of events - for any state machine you program, state transitions will be completed in less than 100 microseconds - so inefficient coding won't reduce the precision of events in your data.

### Introduction to the Bpod state machine

- Each state describes Bpod's outputs (Valves, LEDs, BNC channels, wire terminals, serial ports, etc.).
- Events detected by Bpod's inputs can be set to trigger transitions between specific states.

Here is a simple finite state machine, describing a binary switch that controls a bulb with variable brightness:

```mermaid
stateDiagram
direction LR
onstate: On state\nBrightness 100
offstate: On state\nBrightness 0
onstate --> offstate: Switched on
offstate --> onstate: Switched off
```

- Each state contains a name ("On state" or "Off state"), a hardware description ("Brightness: X"), and transition events ("Switched on/off")

Here is the same diagram presented as a state matrix, written in proper syntax for Bpod:

```matlab
sma = NewStateMatrix();         % Initializes a new, empty state 
                                % matrix, and assigns it to the variable "sma".

sma = AddState(sma, 'Name', 'OnState', ...  % Adds a new state called "OnState" 
                                            % to the matrix. 
    'Timer', 0,...                          % Sets the internal timer of 
                                            % "On state" to 0 seconds. 
    'StateChangeConditions', {'Port1In', 'OffState'},...  % Causes a transition 
                % to "Off state" (not yet defined) if a "Port1In" event occurs. 
    'OutputActions', {'PWM1', 255});           % Outputs for "On state". PWM1 is
                % Port 1's PWM channel, value set to max LED brightness 
                % (range = 0-255). 

sma = AddState(sma, 'Name', 'OffState', ... % Adds a state called "Off state". 
                                            % PWM1 = 0, "Port1Out" returns to
                                            % first state. 
    'Timer', 0,...
    'StateChangeConditions', {'Port1Out', 'OnState'},...
    'OutputActions', {'PWM1', 0});
```

## Emulator mode
If the Bpod software can not connect to a Bpod device, it can be run in Emulator mode.

- In emulator mode, a software state machine on the computer takes the place of the Bpod device.    
- State matrices can be sent and run, and events returned without changing protocol code.
- The Bpod console shows the state of the device, and accepts manual inputs. Simply click port sensor, BNC and wire override buttons.
- The computer clock is used for timers and event timestamps. Thus, timing is far less precise than for measurements obtained with the Bpod device.
- The [PsychToolboxSoundServer plugin](../user-guide/function-reference.md#psychtoolboxsoundserver) detects when Bpod is in emulator mode, and plays sounds through the computer's default sound card.
    - In emulator mode, sounds are played with the [MATLAB sound function](http://www.google.com/url?q=http%3A%2F%2Fwww.mathworks.com%2Fhelp%2Fmatlab%2Fref%2Fsound.html&sa=D&sntz=1&usg=AOvVaw2FJb4XPhRZQubFll-UohrR). Performance and availability varies by platform.
- Emulator mode can be forced when hardware is connected, by running Bpod('EMU')
- To exit emulator mode, ensure that the Bpod device is plugged in and restart Bpod.
- Currently, only the state machine's onboard channels are supported. If your protocol depends on Bpod modules or other external hardware, you will have to run your protocol with the hardware present.

## Liquid calibration
Solenoid valves connected to each behavior port (we recommend [these](http://www.google.com/url?q=http%3A%2F%2Fwww.theleeco.com%2Felectro-fluidic-systems%2Fsolenoid-valves%2Flhd%2Fsoft-tube-ported-style.cfm&sa=D&sntz=1&usg=AOvVaw1w0EV-e7R4MRGhzhhuY39h) for their fast action) can gate the gravity flow of liquid reward from an elevated reservoir to the test subject below. When writing your protocol, you might want to deliver a 5µl of liquid reward to a mouse - but how long should you open the valve to achieve this? The `GetValveTimes()` function will solve this for you, by reading from a calibration curve you create. Here's how to create and manage calibration curves:

<!-- ### Step 1. Launch the calibration manager -->
<!-- Original wiki has no step 2 heading -->

- From the [Bpod console](../user-guide/general-concepts.md#bpod-console), click "Settings" (wrench icon). You will see a settings menu: 

![Alt text](../images/console-settings-menu.png)

- Next, click "Liquid reward calibration" (the faucet icon on the far left)
- You should now see the calibration manager:

![Alt text](../images/liquid-calibration-menu.png)

- In the left list box, you can select one of the 8 valves to view its measurements and calibration curve in the right panels.
- The "Plus" button will manually add a new amount to be measured. This will appear in red as a "Pending measurement".
- The "Minus" button will permanently delete the selected measurement. 
    - The "Suggest Points" button prompts you for a liquid amount range of interest, and automatically adds the best pending measurements: 

![Alt text](../images/liquid-measurements.png)

- The "#Pulses / measurement" button allows you to specify how many pulses will be delivered for each measurement. Larger numbers of pulses reduce measurement error, especially when calibrating for small liquid quantities.
- The "Measure Pending" button delivers pulses from each valve that has a pending measurement. You should capture the water dispensed from each valve in a separate [weigh boat](https://www.google.com/search?q=weigh+boat&source=lnms&tbm=isch). When dispensing is complete, the software prompts you to measure each boat on a laboratory balance, and enter the resulting liquid weights. It then updates the calibration curve.
- The "Test curve" button delivers 100 test pulses of a size you select from a valve you select, prompts for the resulting weight, and indicates whether the measurement is within a selected tolerance:

![Alt text](../images/liquid-curve-test.png)

<!-- this function could be improved if goal is just to see if one liquid port is doing its job -->

## Modules
Beyond solenoid valves, LEDs and TTL pulses, it's hard to anticipate what kinds of outputs Bpod will need to control hardware in future experiments.

As a general expansion framework, we have exposed 3 (or more) of the [UART serial ports](https://www.google.com/url?q=https%3A%2F%2Flearn.sparkfun.com%2Ftutorials%2Fserial-communication%2Fuarts&sa=D&sntz=1&usg=AOvVaw2e5bid8ez_clYR9sdmyEtv) of the state machine's microcontroller.

<!-- this is dope, more emphasis on the user-expandability of this would be good -->

- The serial ports are indicated on the enclosure as RJ45 ethernet jacks labeled "Modules" 1-N.
- The ports are configured to communicate with other microcontrollers at 1.3125Mb/s
- The state machine sends UART serial transmissions to modules using an RS485 IC at each end of the ethernet cable. This employs differential signaling over the Ethernet cable's twisted wire pairs, to make the digital messages more robust against noise.

We designed a special circuit board to interface between these ports and the UART on Arduino Leonardo: the [Bpod Arduino Shield](../assembly/arduino-shield-gen2-assembly.md).

To develop your own serial device with Arduino M0 or Arduino Due and the Bpod Arduino shield, use the BlinkModule sketch as a starting point.
- /Bpod_Gen2/Examples/Firmware/Gen2/BlinkModule/BlinkModule.ino

It will help to become familiar with the [Arduino language](http://www.google.com/url?q=http%3A%2F%2Farduino.cc%2Fen%2FReference%2FHomePage&sa=D&sntz=1&usg=AOvVaw1v-cPDNL0l0ua0s9yO_xvD).

An excellent intro to Arduino is located [here](https://www.google.com/url?q=https%3A%2F%2Flearn.sparkfun.com%2Ftutorials%2Fwhat-is-an-arduino&sa=D&sntz=1&usg=AOvVaw1od5YgunQFQgRDuuzRaBOE).

## Bonsai integration

[Bonsai](https://bonsai-rx.org/) is an open source software tool for processing data streams, developed by [Goncalo Lopes](https://neurogears.org/about-us/).

Among many applications in behavior measurement, it can be used for live video tracking, e.g. to trigger a Bpod state change based on the test subject's position in an arena.

Two methods exist to integrate Bpod with Bonsai, depending on your state machine model and firmware:

### State Machine r2.0 and newer with firmware v23

With Firmware v23 on Bpod State Machine 2.0 or newer, the machine creates two USB serial ports on the PC. The primary port (e.g. COM3) is used to communicate with MATLAB. The secondary "App" port (e.g. COM4) can be used by a third-party application to exchange events with the state machine. Bonsai can send bytes to the App serial port using the [SerialWrite](https://bonsai-rx.org/docs/api/Bonsai.IO.Ports.SerialWrite.html) sink, found under 'IO.Ports' in the sink menu. Bonsai can receive bytes from the App serial port using the [SerialRead](https://bonsai-rx.org/docs/api/Bonsai.IO.Ports.SerialRead.html) source, under 'IO.Ports' in the source menu. 

Note: From the Bpod console, click the Info (spyglass) icon to view the identity of the App serial port.

#### Bonsai --> Bpod State Machine

When bytes in range [0x1, 0x15] are sent from Bonsai's SerialWrite sink to the state machine's App serial port, the bytes are interpreted by the state machine as events App_SoftCode0 to App_SoftCode14. To handle these events, add them to the 'StateChangeConditions' section of a state. The following example state proceeds to the next state when byte 0x2 arrives from Bonsai:

```matlab
sma = AddState(sma, 'Name', 'WaitForBonsai', ...
    'Timer', 0,...
    'StateChangeConditions', {'APP\_SoftCode2', 'MyNextState'},...
    'OutputActions', {});
```

#### Bpod State Machine --> Bonsai

Bytes in range 1-255 can be sent to Bonsai's SerialRead source from any state using:

```matlab
ByteToSend = 3; % Send byte 0x3 to Bonsai

sma = AddState(sma, 'Name', 'SendToBonsai', ...
    'Timer', 0,...
    'StateChangeConditions', {'Tup', 'MyNextSTate'},...
    'OutputActions', {'AppSoftCode', ByteToSend});
```

An example Bonsai program [here](https://github.com/sanworks/Bpod_Gen2/tree/develop/Functions/Plugins/Bonsai/APP_SoftCode%20Example) uses the SerialRead and SerialWrite nodes to read incoming bytes from the state machine, and echo the same bytes back to the state machine generating Bpod events. Usage instructions are in the Readme file in the program folder.

### State Machine r0.5 - r1.0 (any firmware), and r2.0 with firmware v22

Since Bonsai is a separate program and MATLAB requires ownership of the Bpod state machine's only serial port, the soft codes must be passed via a local TCP socket. The Bonsai Socket Configurator manages this socket connection by creating a TCP server within MATLAB, and confirming when Bonsai is successfully attached as a client.

Notice: The current release should be regarded as an alpha. It works with example code as detailed below, but it is preliminary, and will be updated in a future revision with a more general solution for built-in TCP communication. Please report any bugs you encounter.

Step 1: Launch the settings and calibration manager.

- From the Bpod console, click "Settings" (the wrench icon)f

![Alt Text](../images/console-settings-menu.png)

- Next, click "Setup Bonsai Socket Connection" (the tree icon)
- You should now see the Bonsai socket configurator:

![Alt text](../images/bonsai-socket-configurator.png)

- Since you are creating the TCP server, you can use custom IP and Port. The defaults work fine.
- Click "Connect".
- In Bonsai, run an application that connects to your IP and Port. An example app is available in /Bpod/Bpod System Files/Plugins/Bonsai/App_SoftCode Example/. To demonstrate use of the app, after connecting to Bonsai, run the soft code example in /Bpod/Bpod System Files/ExampleMatrices: SoftCodeTriggeredStateChange.m. Shifting a bright light across your webcam's field of view from right to left should trigger a state change, visible as a change in port LEDs.
- If the connection was initiated successfully, the Bonsai Status should show "Connected" in green.
- Check "Auto-connect on Bpod start" to automatically connect to Bonsai when you run Bpod. Note: this will halt the Bpod launch routine until a Bonsai app connects.