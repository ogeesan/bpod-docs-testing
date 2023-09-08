# HiFi Module Serial Interface

## Description

Allows the state machine or PC to load and play sounds using the [Bpod HiFi Module](../module-documentation/hifi-module.md).

Requires a Bpod HiFi module with firmware loaded from:

- [https://github.com/sanworks/](https://www.google.com/url?q=https%3A%2F%2Fgithub.com%2Fsanworks%2FBpod_HiFi_Firmware&sa=D&sntz=1&usg=AOvVaw2ZMAy5LNY3KO1RRxuJC9g3)[Bpod\_HiFi\_Firmware](https://www.google.com/url?q=https%3A%2F%2Fgithub.com%2Fsanworks%2FBpod_HiFi_Firmware&sa=D&sntz=1&usg=AOvVaw2ZMAy5LNY3KO1RRxuJC9g3)

The HiFi module must be connected to a module port on the state machine to use the state machine interface.

## State Machine Command Interface

The state machine command interface consists of bytes sent from the Bpod state machine to the HiFi module to start and stop playback, and control synth.

- Byte 255 (reserved): Returns module info to state machine
- '**P**' (ASCII 80): **Plays a sound**.
    - 'P' (byte 0) must be followed by one byte:
        - Byte 1: The sound to play (zero-indexed).
- **'\*'** (ASCII 42): **Push any loaded sounds to current playback buffers**.
    - Note: This op will make any sounds loaded recently current at the loaded positions, replacing any sounds at those positions.
        - Thus, if new sounds for positions 2 and 3 are loaded during a trial for use on the next trial, sending '\*' will make these waveforms the current sounds at positions 2 and 3.
    - If called from the PC via USB, The HiFi module returns a byte (1) to confirm that it has finished pushing loaded sounds.
- '**X**' (ASCII 88): **Stop all playback**
- '**x**' (ASCII 120): **Stop playing a specific sound** (if that sound happens to be playing)
    - 'x' (byte 0) must be followed by one byte:
        - Byte 1: The sound to stop (zero-indexed).
- **'N'** (ASCII 78): **Set Synth Amplitude.** 'N' (byte 0) is followed by:
    - Bytes 1-2: synthAmplitude (16-bit unsigned integer)
    - synthAmplitude must be in range 0-32767 where 0 = no synth, and 32767 = max amplitude (i.e. this is an attenuation factor in bits)
    - If called from the PC via USB, The HiFi module returns a byte (1) to confirm that it has finished setting synth amplitude.
- **'F'** (ASCII 78): **Set Synth Frequency.** 'F' (byte 0) is followed by:
    - Bytes 1-4: synthFrequency (32-bit unsigned integer)
    - synthFrequency is the target frequency \*1000 (e.g. to set 100Hz, use 100000)
    - synthFrequency is ignored if the current waveform is white noise.
    - If called from the PC via USB, The HiFi module returns a byte (1) to confirm that it has finished setting synth frequency.
- **'W'** (ASCII 78): **Set Synth Waveform.** 'W' (byte 0) is followed by:
    - Byte 1: Waveform type. Values are 0: White Noise, 1: Sine
    - If called from the PC via USB, The HiFi module returns a byte (1) to confirm that it has finished setting synth waveform.

## SerialUSB Command Interface

The SerialUSB command interface allows configuration of the HiFi module from MATLAB or Python before a trial begins. The [BpodHiFi](../module-documentation/hifi-module.md) class for Bpod/MATLAB wraps this interface. The first several commands are the same as for the state machine interface, and additional commands follow.

- (ASCII 243): **Handshake**. The module replies with the following sequence of bytes:
    - handshakeReply (1 byte); Equal to 244
- '**L**' (ASCII 76): **Load an audio waveform**. 'L' (byte 0) is followed by:
    - Byte 1: The waveform to load (0-19). Note: In the HiFiModule class, these are corrected for MATLAB's indexing (1-20).
    - Byte 2: isStereo (set to 1 if loading a stereo/2ch waveform, or 0 if mono/1ch). Note: Both speakers play mono waveforms.
    - Bytes 3 - 6: The number of samples in the audio waveform expressed as a 32-bit integer (1-1,000,000).
    - Bytes 7 - (7+((2 + (2\*isStereo))\*nSamples)): The waveform. Each byte is a 16-bit signed integer coding for an audio sample.
    - The HiFi module returns a byte (1) to confirm that it has finished reading the last sample.
- '**S**' (ASCII 83): **Set sampling rate** (units = Hz). 'S' (byte 0) is followed by:
    - Bytes 1-4: Sampling rate in hz (32-bit unsigned integer)
    - Note that only 44100, 48000, 96000 and 192000 are valid sampling rates
    - The HiFi module returns a byte (1) to confirm that it has finished setting the sampling period.
- '**E**' (ASCII 69): **Enable AM envelope**. 'E' (byte 0) is followed by:
    - Byte 1: UseAMEnvelope: 1 if using AM envelope, 0 if not.
    - The HiFi module returns a byte (1) to confirm that it has finished setting the AM envelope.
- '**M**' (ASCII 77): **Load AM envelope**. 'M' (byte 0) is followed by:
    - Bytes 1-2: EnvelopeSize (16-bit unsigned int): The number of samples in the envelope
    - Bytes 3-(3+EnvelopeSize\*4): samples of the 16-bit AM envelope. These are 4-byte floats (attenuating factors) in range \[0 to 1\].
    - The HiFi module returns a byte (1) to confirm that it has finished storing the AM envelope.
- '**O**' (ASCII 79): **Set loop mode for a waveform**. 'O' (byte 0) is followed by:
    - Bytes 1-20: Loop mode bytes. Each byte represents a waveform: 1 if using loop mode, 0 if not.
    - The HiFi module returns a byte (1) to confirm that it has finished setting the loop mode.
- **'-'** (ASCII 45): **Set duration for looping playback on a single trigger.** '-' (byte 0) is followed by:
    - Bytes 1-21: Duration of looped playback on trigger for each waveform (expressed in samples) as a 32-bit integer.
        - Note: Because loop duration is expressed in samples, it must be re-loaded following sampling rate changes (see op above, 'S')
    - The HiFi module returns a byte (1) to confirm that it has finished setting the loop duration.
- **'A'** (ASCII 65): **Set digital attenuation.** 'A' (byte 0) is followed by:
    - Byte 1: Digital attenuation. Valid values are in range 0-240
    - The HiFi module returns a byte (1) to confirm that it has finished setting the digital attenuation.
- '**I**' (ASCII 78): **Return system info**. Nothing follows byte 'I'. The module replies with the following sequence of bytes:
    - isHD(1 byte); Corresponds to HiFiBerry DAC board (0 = DAC2 Pro, 1 = DAC2 HD)
    - bitDepth (1 byte); Bit depth of audio samples. Possible values are 16 or 24 (current firmware supports 16 bit only)
    - maxWaves (1 byte); Maximum number of waveforms that can be loaded
    - digitalAttenuation (1 byte); Digital attenuation. dB = digitalAttenuation\*-0.5
    - samplingRate (4 bytes; 32-bit int); Current sampling rate in Hz
    - maxSecondsPerWaveform (4 bytes; 32-bit int); Maximum number of seconds that can be stored for a 192kHz stereo waveform
    - maxEnvelopeSize (4 bytes; 32-bit int); Maximum number of samples in the configurable onset/offset AM envelope

## Examples

Play sound #4 using an [ArCOM](http://www.google.com/url?q=http%3A%2F%2Fsites.google.com%2Fsite%2Fsanworksdocs%2Farcom&sa=D&sntz=1&usg=AOvVaw0q9tKPNJMCdKV2qsdKk90n) serial object in MATLAB:

```matlab
D = ArCOMObject('COM3', 115200);

D.write(['P' 3], 'uint8'); % Remember that sound position is 0-indexed!

clear D
```

Trigger sound #4 from the Bpod state machine
```matlab
LoadSerialMessages('HiFi1', {['P' 3]}); % Set serial message 1

sma = NewStateMachine();

sma = AddState(sma, 'Name', 'PlaySound', ...
    'Timer', 0.1,...
    'StateChangeConditions', {'Tup', 'exit'},...
    'OutputActions', {'HiFi1', 1}); % Sends serial message 1

SendStateMachine(sma);
RawEvents = RunStateMachine;
```
