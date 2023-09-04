# BpodWavePlayer()

## Description

`BpodWavePlayer` plays analog waveforms on trigger, using the [Analog Output Module]() (a precision voltage DAC with a dedicated microcontroller).

The analog output module must have WavePlayer firmware loaded to use this class.

After running Bpod, a `BpodWavePlayer` object is initialized with the following syntax:

```matlab
W = BpodWavePlayer('COM3');
```

Where COM3 is the analog output module's serial port.

The BpodWavePlayer device is controlled in 2 ways: 

- Setting the `BpodWavePlayer` object's fields
- Calling the `BpodWavePlayer` object's functions

## Object Fields

- Port 
    - [ArCOM](http://sites.google.com/site/sanworksdocs/arcom) Serial port object
- Info
    - A struct containing the connected module's firmware and hardware versions
- SamplingRate (Hz)
    - 1Hz-10kHz, affects all channels. 
    - Can be set beyond 10kHz, up to to 20kHz, automatically disabling channels 3-4.
- OutputRange (String)
    - A string specifying voltage output range for all channels: 
        - '0V:5V'
        - '0V:10V'
        - '0V:12V'
        - '-5V:5V'
        - '-10V:10V'
        - '-12V:12V'
    - For best signal quality, use the smallest voltage range necessary for your application. 
    - An error is raised if a voltage sample in a currently loaded waveform is outside of the new range. 
    - All currently loaded voltage waveforms are automatically re-coded into bits, and sent to the device.
- Waveforms (Cell)
    - Local copy of all waveforms loaded to microSD with loadWaveform() function.
- TriggerMode (String)
    - A string specifying how to handle incoming bytes from Bpod:
        - 'Normal' - plays the triggered wave(s), and ignores triggers on the same channel during playback
        - 'Master' - triggers can force-start a new wave during playback.
        - 'Toggle' - starts playback. Stops playback if a channel is triggered during playback.
- TriggerProfiles (Matrix)
    - Allow a single trigger byte to specify different waveforms to play on any subset of channels.
    - TriggerProfiles must be an maxTriggerProfiles X nChannels matrix. Columns are channels, rows are trigger profiles.
    - Each row specifies the waveform to play on each channel when the row's trigger byte arrives.
    - TriggerProfiles are disabled by default. Must be enabled by setting TriggerProfileEnable to 'On'
- TriggerProfileEnable (String)
    - A string that controls how bytes from the state machine are handled, following byte 'P' (for play).
    - 'On' = second byte specifies trigger profile to play (range = 1 : maxTriggerProfiles).
    - 'Off' = default trigger scheme: one waveform on multiple channels. 
        - second byte specifies bits corresponding to playback channels (bytes 1-16 = subsets of channels 1-4)
        - A third byte must follow, specifying the waveform index (range = 1 :maxWaveforms).
- LoopMode (String)
    - A string specifying whether loop mode is on or off for each channel
        - 'On' loops the waveform until LoopDuration seconds, or until toggled off
        - 'Off' = plays the waveform once
- LoopDuration (Seconds)
    - In loop mode, specifies the duration to loop the waveform following a trigger.
    - Units = seconds
- BpodEvents (String)
    - 'Off': No feedback to the Bpod state machine
    - 'On' generates a serial event when starting playback, and again when playback finishes.
        - The event byte specifies which channels started or stopped playback during the current playback sample
        - e.g. byte 0x3 = binary 11; event indicates that channels 1 and 2 simultaneously started (or stopped) playback.
        - e.g. byte 15 = binary 1000; event indicates that channel 4 started (or stopped) playback.

## Object Functions

- loadWaveform(waveNumber, waveform)
    - Transmits a voltage waveform to the device
    - waveNumber = index of the waveform (1-64)
    - waveform = a 1xN MATLAB array of voltages (samples in volts)
        - Voltages must be in the range specified by the OutputRange field.
        - An error is raised if the waveform exceeds the allowed maximum (1M samples)
    - Function returns an error if waveform was not successfully transmitted.
- play(channels, waveNumber)
    - Plays a waveform on any subset of channels.
    - channels = a vector indicating the output channels to trigger (e.g. [2 4] = channels 2 and 4)
    - waveNumber = index of the waveform to play on targeted channels (1-64)
    - If TriggerMode is set to 'Normal' and a waveform is playing, the play() command is ignored by the device.
    - If TriggerMode is set to 'Toggle' and a waveform is playing on a targeted channel, the waveform is stopped.
    - If TriggerMode is set to 'Master' and a waveform is playing, it is immediately replaced by the new waveform.
    - Returns an error if TriggerProfileEnable is set to 'On'; see play(triggerProfile)
        - Plays only 1 waveform per function call. For simultaneous triggering of multiple waveforms, see play(triggerProfile)
- play(triggerProfile)
    - Plays waveforms on target channels specified by a trigger profile
    - triggerProfile = index of the trigger profile to play (1-64)
    - triggerProfileEnable must be set to 'On'
- play(listOfWaveforms)
    - Requires firmware v5 or newer
    - listOfWaveforms is a 1xnChannels array specifying which waveform to play on which output channel
    - to indicate 'No Waveform', use 0
- stop()
    - Stops all ongoing playback
- setFixedVoltage(Channels, Voltage)
    - Sets a persistent voltage on a subset of output channels.
    - channels = a vector indicating the output channels to set (e.g. [2 4] = channels 2 and 4).
    - Voltage = the voltage to set. The voltage must be within the currently configured OutputRange (see OutputRange field above).
    - Playing a waveform on the channel(s) will return the voltage to 0V after playback completes.
- set2Defaults()
    - Loads default values for all parameters. As of Bpod_Gen2 v1.73, set2Defaults is run on creating a BpodWavePlayer object, to ensure that the device begins each session in a fixed state.
    - set2Defaults can be run explicitly by the user to clear any changes to the parameters.
    - In versions of Bpod_Gen2 prior to 1.73, parameters were read from the device to populate the BpodWavePlayer object on init. This could create issues on shared rigs if one experiment leaves the device in an unexpected state. Updating to Bpod_Gen2 v1.73 or newer is strongly recommended.
- setupSDCard()
    - Must be run before first use with a new card if using firmware v3 or older
    - Creates an empty data file on the module's microSD card, where waveforms will be stored.
    - Setup may take ~1 minute, depending on the microSD card's speed. A SanDisk Industrial card is strongly recommended.

## Cleanup

- Clear the BpodWavePlayer object with clear:

```matlab
W = BpodWavePlayer('COM3');

% ...Use the wave player

clear W
```

- Clearing the object releases the serial port, so other applications can access it.
- If a BpodWavePlayer object is created inside a MATLAB function, the object is cleared automatically when the function returns.