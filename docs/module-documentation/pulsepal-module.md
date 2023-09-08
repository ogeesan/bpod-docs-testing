# PulsePalModule()

## Description

PulsePalModule plays stimulation patterns on trigger, using the [Bpod Analog Output Module](../assembly/analog-input-module-assembly.md) (a precision voltage DAC with a dedicated microcontroller).

Pulse trains are [parametric](http://www.google.com/url?q=http%3A%2F%2Fsites.google.com%2Fsite%2Fpulsepalwiki%2Fparameter-guide&sa=D&sntz=1&usg=AOvVaw1ZWmqWtNp5a9EmGr7xA_v9). The parameters are identical to the [Beta version](https://www.google.com/url?q=https%3A%2F%2Fgithub.com%2Fsanworks%2FPulsePal%2Ftree%2Fbeta%2FMATLAB_GEN2&sa=D&sntz=1&usg=AOvVaw1zt-fb28L_J_GRnNAyx7CW) of [Pulse Pal](http://www.google.com/url?q=http%3A%2F%2Fsites.google.com%2Fsite%2Fpulsepalwiki%2Fhome&sa=D&sntz=1&usg=AOvVaw18RXRTwRYC7wH5orxsQg3j)'s MATLAB API.

To use PulsePalModule, you must [upload](../install-and-update/firmware-update.md) PulsePalModule firmware to the output module from [here](https://www.google.com/url?q=https%3A%2F%2Fgithub.com%2Fsanworks%2FBpod_AnalogOutput_Firmware&sa=D&sntz=1&usg=AOvVaw2mYfTij1ftDktlZRDZH-oN).

After running Bpod, a PulsePalModule object is initialized with the following syntax:

```matlab
P = PulsePalModule('COM3');
```

Where COM3 is the analog output module's serial port.

The PulsePalModule device is controlled in 2 ways:

- Setting the PulsePalModule object's fields
- Calling the PulsePalModule object's functions

## Object Fields

- Port
    - [ArCOM](https://www.google.com/url?q=https%3A%2F%2Fgithub.com%2Fsanworks%2FArCOM&sa=D&sntz=1&usg=AOvVaw2OHUdnSsYwO-uBGXj0R92G) Serial port
- nChannels
    - Number of output channels on the connected Analog output module
- Parameters
    - 1 x nChannels vector, specifying the value of a pulse train parameter.
    - Pulse Pal parameters and their units are described [here](http://www.google.com/url?q=http%3A%2F%2Fsites.google.com%2Fsite%2Fpulsepalwiki%2Fparameter-guide&sa=D&sntz=1&usg=AOvVaw1ZWmqWtNp5a9EmGr7xA_v9).
    - When modifying parameters, specify channels by index:
        - P.phase1Voltage(2) = 5 % sets output channel 2's pulse voltage to 5V
        - P.interPulseInterval(1:4) = 0.001 % Sets the inter-pulse interval on ch1-4 to 1ms
        - P.restingVoltage = 3 % Returns an error: no channels specified
- TriggerMode (integer)
    - An integer specifying how to handle incoming bytes from Bpod:
        - 0 - plays the triggered pulse train, and ignores triggers on the same channel during playback
        - 1 - starts playback. Stops playback if a channel is triggered during playback.
- autoSync (string)
    - 'On': updates the device immediately each time a pulse train parameter is changed in MATLAB
    - 'Off': updates to the device are handled manually by calling the sync() function.

## Object Functions

- trigger(channels)
    - Triggers the output channels specified in (channels) to play their currently loaded pulse train
    - Useful for testing pulse trains. During behavior, pulse trains are triggered directly from the Bpod device.
- abort()
    - Aborts ongoing pulse train playback on all channels.
- setVoltage(channel, voltage)
    - Sets a fixed voltage on an output channel
- sync()
    - If autoSync is disabled, this function manually updates the pulse train parameters on the device to match the MATLAB object.
- sendCustomPulseTrain(trainID, pulseTimes, voltages)
    - Sends a custom pulse train to the device, overriding many parameters.
        - Set an output channel to use a custom pulse train by setting the object's customTrainID field.
        - trainID: Index of the pulse train (1-4)
    - pulseTimes: onset times of each pulse in the train (seconds)
    - voltages: voltages of each pulse in the train (volts)
- sendCustomWaveform(trainID, samplePeriod, Voltages)
    - Sends a custom pulse train to the device, overriding many parameters.
        - Sequential pulses are assumed, and their width is set by the samplePeriod argument.
    - samplePeriod: The width of each pulse (seconds)
    - voltages: voltages in the waveform (volts)
- saveParameters(filename)
    - Saves the current object's pulse train parameters to a MATLAB .mat file
- loadParameters(filename)
    - Loads pulse train parameters from a MATLAB file previously saved with the saveParameters() function.
    - The PulsePalModule object's fields are populated from the file, and the parameters are synchronized to the device.
- setDefaultParams()
    - Sets all pulse train parameters to their defaults (a 1-second train of 1ms, 5V pulses on all 4 output channels, at 100Hz)

## Triggering from the state machine
From the state machine's output actions, use:
{'PulsePal1', N}
N is a byte, whose bits indicate which channel(s) to trigger.

## Cleanup

- Clear the PulsePalModule object with clear:
```matlab
P = PulsePalModule('COM3');

... % Use the module

clear P
```

- Clearing the object releases the serial port, so other applications can access it.
- If a PulsePalModule object is created inside a MATLAB function, the object is cleared automatically when the function returns.