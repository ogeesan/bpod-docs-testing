# WavePlayer Serial Interface

## Description

Allows the state machine or PC to play analog waveforms using the [analog output module]().

Requires a 4-channel analog output module board with firmware loaded from:
- [https://github.com/sanworks/Bpod_AnalogOutput_Firmware/tree/master/AnalogOutputModule_4ch/BpodWavePlayer](https://github.com/sanworks/Bpod_AnalogOutput_Firmware/tree/master/AnalogOutputModule_4ch/BpodWavePlayer)

OR, an 8-channel analog output module board with firmware loaded from:
- [https://github.com/sanworks/Bpod_AnalogOutput_Firmware/tree/master/AnalogOutputModule_8ch/BpodWavePlayer](https://github.com/sanworks/Bpod_AnalogOutput_Firmware/tree/master/AnalogOutputModule_8ch/BpodWavePlayer)

The analog output module must be connected to a serial port on the state machine module.

## State Machine Command Interface

The state machine command interface consists of bytes sent from the Bpod state machine to the WavePlayer module to start and stop playback.

- Byte **255** (reserved): **Returns module info to state machine**
- '**P**' (ASCII 80): **Plays a waveform**.
    - In standard trigger mode (default), 'P' (byte 0) must be followed by two bytes:
        - Byte 1: A byte whose bits indicate which channels to trigger (i.e. byte 5 = bits: 101 = channels 1 and 3).
        - Byte 2: A byte indicating the waveform to play on the channels specified by Byte 1. NOTE: This byte uses 0-indexing.
    - In trigger profile mode, 'P' (byte 0) must be followed by 1 byte:
        - Byte 1: The trigger profile to play (1-64)
- '**>**' (ASCII 62): **Plays a list of waveforms specified for each channel**
    - REQUIRES FIRMWARE v5 OR NEWER
    - '>' (byte 0) must be followed by one byte for each output channel on the device.
        - Byte 1: A byte indicating the waveform to trigger on channel 1. Use 255 for no waveform.
        - ...
        - Byte 4: A byte indicating the waveform to trigger on channel 4.
        - NOTE: The waveform bytes use 0-indexing, so if you loaded a waveform to the device in MATLAB with W.loadWaveform(Wave, 1), the wave is known to WavePlayer's Arduino firmware as wave 0.
    - A byte must be specified for every physical output channel. Use 255 for 'no waveform'.
    - On State Machine r2 or newer with firmware v23, the '>' op can be output from a single state for the 4-channel module as 5 bytes are available per serial message. Earlier machines, or state machine r2 with earlier firmware have only 3 bytes per serial message, so the 5 bytes for the '>' op must be broken across states. Consider using Trigger Profile mode in this case to save 100µs of latency.
- '**!**' (ASCII 33): **Sets a fixed voltage on a subset of channels**.
    - REQUIRES FIRMWARE v5 OR NEWER
    - '!' (Byte 0) must be followed by:
        - Byte 1: Bits of channels to set (e.g. 3 = 00000011, to set channels 1 and 2)
        - Bytes 2-3: 16-bit value encoding the voltage.
        - If sent from the PC, the module returns an acknowledgement byte (1)
- '**X**' (ASCII 88): **Stop all playback**

## SerialUSB Command Interface

The SerialUSB command interface allows configuration of the WavePlayer module from MATLAB or Python before a trial begins. The [WavePlayer](../module-documentation/waveplayer.md) plugin for Bpod/MATLAB wraps this interface. The first two commands are the same as for the state machine interface, and additional commands follow.

- '**P**' (ASCII 80): **Plays a waveform**.
    - In standard trigger mode (default), 'P' (byte 0) must be followed by two bytes:
        - Byte 1: A byte whose bits indicate which channels to trigger (i.e. byte 5 = bits: 101 = channels 1 and 3).
        - Byte 2: A byte indicating the waveform to play on the channels specified by Byte 1 (zero-indexed).
    - In trigger profile mode, 'P' (byte 0) must be followed by 1 byte:
        - Byte 1: The trigger profile to play (1-64)
- '**X**' (ASCII 88): **Stop all playback**.
- '**L**' (ASCII 76): **Load a waveform**. 'L' (byte 0) is followed by:
    - Byte 1: The waveform to load (0-63). Note: In the WavePlayer plugin, these are corrected for MATLAB's indexing (1-64).
    - Bytes 2 - 5: The number of samples in the waveform expressed as a 32-bit integer (1-1,000,000).
    - Bytes 6 - (6+(2\*nSamples)): The waveform. Each byte is a 16-bit integer coding for a voltage in the currently selected output range.
    - The WavePlayer module returns a byte (1) to confirm that it has finished reading the last sample.
- '**R**' (ASCII 82): **Set the output range** (default = +/-5V). 'R' (byte 0) is followed by:
    - Byte 1: The range index. Range indexes are:
        - 0: 0V to +5V
        - 1: 0V to +10V
        - 2: 0V to +12V
        - 3: -5V to +5V
        - 4: -10V to +10V
        - 5: -12V to +12V
    - The WavePlayer module returns a byte (1) to confirm that it has finished setting the range.
    - NOTE: The waveforms are stored on the device in bits. The bits are computed for the current range. When you change the output range of the device, the waveforms already loaded will be compressed or expanded in amplitude. The [WavePlayer](../module-documentation/waveplayer.md) plugin (MATLAB) re-loads every waveform on the device, following a range update, to enforce accurate playback of the waveform voltages.
- '**S**' (ASCII 83): **Set the sampling period** (units = microseconds, default = 100; sampling rate = 10kHz). 'S' (byte 0) is followed by:
    - Bytes 1-4: Sampling period in microseconds (32-bit integer)
    - The [WavePlayer](../module-documentation/waveplayer.md) plugin (MATLAB) exposes sampling rate to the user, and computes sampling period before transmitting.
    - If Loop mode is active, loop durations will be scaled with the sampling period as they are expressed in samples internally. They will need to be re-computed and re-transmitted (see next op, 'O')
- '**O**' (ASCII 79): **Set loop mode and loop duration**. 'O' (byte 0) is followed by:
    - Bytes 1-4 (or 1-8 for 8-channel module): Loop mode bytes. Each byte represents a channel: 1 if using loop mode, 0 if not.
    - Bytes 5-21 (or 9-41): Duration of looped playback on trigger for each channel (expressed in samples) as a 32-bit integer.
        - Note: Because loop duration is expressed in samples, it must be re-loaded following sampling rate changes (see op above, 'S')
- '**V**' (ASCII 86): **Set Bpod state machine event reporting**. 'V' (byte 0) is followed by:
    - Bytes 1-4 (or 1-8 for 8-channel module): Each byte represents a channel. 1 to report the channel's events, 0 if not.
    - Events are returned as bits to indicate each channel's playback start OR stop:
        - If channel 1 started (or stopped) playback on the current cycle, a byte (1) is returned to the state machine.
        - If channels 1 and 3 started (or stopped) playback simultaneously, (5) is returned since channels 1 and 3 in binary = 1 0 1 = 5.
- '**T**' (ASCII 84): **Set trigger mode**. 'T' (byte 0) is followed by:
    - Byte 1: The trigger mode index:
        - 0 = Standard mode ('P' command is followed by channel bits and waveform ID)
        - 1 = Trigger profile mode ('P' command is followed by a profile index uploaded previously with 'F' command (below)).
        - A trigger profile specifies which waveforms to play on which channels.
- '**F**' (ASCII 70): **Load trigger profile**. 'F' (byte 0) is followed by:
    - Bytes 1-64: The waveform (1-64) OR 255 (no waveform) to play on channel 1 for each of 64 trigger profiles.
    - Bytes 65:(65+(64\*nChannels)): The waveform to play for each profile on channels 2:nChannels
- '**N**' (ASCII 78): **Retrieve current parameters**. Nothing follows byte 'N'. The module replies with the following sequence of bytes:
    - nChannels (1 byte); number of output channels
    - maxWaves (2 bytes; 16-bit int); maximum number of waveforms supported
    - triggerMode (1 byte); current trigger mode (see 'T' above)
    - triggerProfileEnable (1 byte); 0 = standard trigger mode, 1 = trigger profile mode
    - maxTriggerProfiles (1 byte); maximum number of trigger profiles supported
    - rangeIndex (1 byte); index of the currently selected range (see 'R' above)
    - timerPeriod (4 bytes; 32-bit int); sampling period (in microseconds)
    - sendBpodEvents (1 byte per channel); 0 = off, 1 = bpod event reporting (see 'V' above)
    - loopMode (1 byte per channel) 0 = off, 1 = on (see 'O' above)
    - loop duration (4 bytes per channel; 32-bit ints) duration of loop playback in samples (see 'O' above)
- '**H**' (ASCII 72): **Load hardware information**. REQUIRES FIRMWARE v5 OR NEWER. 'H' (byte 0) is followed by:
    - hardwareVersion (1 byte); The major hardware version number
    - circuitVersion (1 byte): The circuit board revision number

## Examples

Play waveform# 4 on output channel 1 from an [ArCOM](http://sites.google.com/site/sanworksdocs/arcom) serial object in MATLAB:

```matlab
D = ArCOMObject('COM43', 115200);
D.write(['P' 1 3], 'uint8');
clear D
```

Play waveform# 4 on output channel 1 from the Bpod state machine:

```matlab
LoadSerialMessages('WavePlayer1', {['P' 1 3]});  % Set serial message 1
sma = NewStateMachine();
sma = AddState(sma, 'Name', 'PlaySound', ...
    'Timer', 0.1,...
    'StateChangeConditions', {'Tup', 'exit'},...
    'OutputActions', {'WavePlayer1', 1}); % Sends serial message 1
SendStateMachine(sma);
RawEvents = RunStateMachine;
```