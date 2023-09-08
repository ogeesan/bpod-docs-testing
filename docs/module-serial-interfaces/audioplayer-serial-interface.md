# AudioPlayer Serial Interface

## Description

Allows the state machine or PC to play mono or stereo sounds using Ch1 + Ch2 of the [analog output module](../assembly/analog-output-module-assembly.md).

Requires a 4-channel analog output module board with BpodAudioPlayer or BpodAudioPlayerLive firmware loaded from:

- [https://github.com/sanworks/Bpod\_AnalogOutput\_Firmware/tree/master/AnalogOutputModule\_4ch](https://www.google.com/url?q=https%3A%2F%2Fgithub.com%2Fsanworks%2FBpod_AnalogOutput_Firmware%2Ftree%2Fmaster%2FAnalogOutputModule_4ch&sa=D&sntz=1&usg=AOvVaw2GrQjO1NLxDy9EtBSjsEUu)
    - Note: The 'Live' version of BpodAudioPlayer allows sounds to be loaded while playback is in progress, but is limited to 44.1kHz sampling.
The analog output module must be connected to a module port on the state machine.

NOTE: the Analog Output Module is not a sound card! If you need high quality audio playback, use the [Bpod HiFi module](../assembly/hifi-module-assembly.md).

## State Machine Command Interface

The state machine command interface consists of bytes sent from the Bpod state machine to the AudioPlayer module to start and stop playback.

- Byte 255 (reserved): Returns module info to state machine
- '**P**' (ASCII 80): **Plays a sound**.
    - 'P' (byte 0) must be followed by one byte:
        - Byte 1: The sound to play (zero-indexed).
- '**X**' (ASCII 88): **Stop all playback**
- '**x**' (ASCII 120): **Stop playing a specific sound** (if that sound happens to be playing)
    - 'x' (byte 0) must be followed by one byte:
        - Byte 1: The sound to stop (zero-indexed).

## SerialUSB Command Interface

The SerialUSB command interface allows configuration of the AudioPlayer module from MATLAB or Python before a trial begins. The [AudioPlayer](../module-documentation/audioplayer.md) class for Bpod/MATLAB wraps this interface. The first two commands are the same as for the state machine interface, and additional commands follow.

- (ASCII 229): **Handshake and reset**. The module replies with the following sequence of bytes:
    - handshakeReply (1 byte); Equal to 230
    - firmwareVersion(4 bytes; 32-bit int); the current firmware version
    - Note: This op will cancel any currently playing or loading sounds, reset the sampling rate to default 44.1kHz, and clear any previously loaded sounds.
- '**L**' (ASCII 76): **Load an audio waveform**. 'L' (byte 0) is followed by:
    - Byte 1: The waveform to load (0-19). Note: In the AudioPlayer plugin, these are corrected for MATLAB's indexing (1-20).
    - Byte 2: isStereo (set to 1 if loading a stereo/2ch waveform, or 0 if mono/1ch). Note: Both speakers play mono waveforms.
    - Bytes 3 - 6: The number of samples in the audio waveform expressed as a 32-bit integer (1-1,000,000).
    - Bytes 7 - (7+((2 + (2\*isStereo))\*nSamples)): The waveform. Each byte is a 16-bit integer coding for a voltage in the range -5 to +5.
    - The AudioPlayer module returns a byte (1) to confirm that it has finished reading the last sample.
- '**\>**' (ASCII 76): **Load an audio waveform in safe mode (slower than 'L')**. '>' (byte 0) is followed by bytes as described in op 'L' above.
    - The AudioPlayer module returns a byte (1) to confirm that it has finished reading the last sample.
- '**S**' (ASCII 83): **Set sampling period** (units = microseconds, default = 22.675737; sampling rate = 44.1kHz). 'S' (byte 0) is followed by:
    - Bytes 1-4: Sampling period in microseconds (32-bit float)
    - The [AudioPlayer class](../module-documentation/audioplayer.md) (MATLAB) exposes sampling _rate_ to the user, and computes sampling period before transmitting.
    - The AudioPlayer module returns a byte (1) to confirm that it has finished setting the sampling period.
    -*'\*'** (ASCII 42): **Push any loaded sounds to current playback buffers**.
    - Note: This op will make any sounds loaded recently current at the loaded positions, replacing any sounds at those positions.
        - Thus, if new sounds for positions 2 and 3 are loaded during a trial for use on the next trial, sending '\*' will make these waveforms the current sounds at positions 2 and 3.
    - The AudioPlayer module returns a byte (1) to confirm that it has finished pushing loaded sounds.
- '**E**' (ASCII 69): **Enable AM envelope**. 'E' (byte 0) is followed by:
    - Byte 1: UseAMEnvelope: 1 if using AM envelope, 0 if not.
    - The AudioPlayer module returns a byte (1) to confirm that it has finished setting the loop mode.
- '**M**' (ASCII 77): **Load AM envelope**. 'M' (byte 0) is followed by:
    - Bytes 1-2: EnvelopeSize (16-bit unsigned int): The number of samples in the envelope
    - Bytes 3-(3+EnvelopeSize\*4): samples of the 16-bit AM envelope. These are 4-byte floats (attenuating factors) in range \[0 to 1\].
    - The AudioPlayer module returns a byte (1) to confirm that it has finished storing the AM envelope.
    -*'Y'** (ASCII 89): **Setup MicroSD card**.
    - Note: This op must be run before use with a new microSD card. It may take up to 1 minute to complete.
    - The AudioPlayer module returns a byte (1) to confirm that it has finished setting up the microSD card.
- '**O**' (ASCII 79): **Set loop mode for a waveform**. 'O' (byte 0) is followed by:
    - Bytes 1-20: Loop mode bytes. Each byte represents a waveform: 1 if using loop mode, 0 if not.
    - The AudioPlayer module returns a byte (1) to confirm that it has finished setting the loop mode.
    -*'-'** (ASCII 45): **Set duration for looping playback on a single trigger.** '-' (byte 0) is followed by:
    - Bytes 1-21: Duration of looped playback on trigger for each waveform (expressed in samples) as a 32-bit integer.
        - Note: Because loop duration is expressed in samples, it must be re-loaded following sampling rate changes (see op above, 'S')
    - The AudioPlayer module returns a byte (1) to confirm that it has finished setting the loop duration.
- '**V**' (ASCII 86): **Set start/stop event reporting to the state machine**. 'V' (byte 0) is followed by:
    - Byte 1: 1 to report waveform on and off events, 0 if not.
    - Note: start and stop events are returned as bytes to indicate each waveform's playback start OR stop:
        - Bytes 0-19 = started playback of waveform 1-20
        - Bytes 20-39 = stopped playback of waveform 1-20
    - The AudioPlayer module returns a byte (1) to confirm that it has finished setting events.
- '**T**' (ASCII 84): **Set trigger mode**. 'T' (byte 0) is followed by:
    - Byte 1: The trigger mode index:
        - 0 = Standard mode ('P' command starts playback, 'P' commands received during playback are ignored)
        - 1 = Master mode ('P' command starts playback, 'P' commands received during playback start different waveforms)
        - 2 = Toggle mode ('P' command starts playback, 'P' commands received during playback ends playback)
    - The AudioPlayer module returns a byte (1) to confirm that it has finished setting the trigger mode.
- '**N**' (ASCII 78): **Return system constants**. Nothing follows byte 'N'. The module replies with the following sequence of bytes:
    - liveMode (1 byte); Corresponds to firmware type (0 = AudioPlayer, 1 = AudioPlayerLive)
    - maxWaves (2 bytes; 16-bit int); maximum number of waveforms supported
    - maxEnvelopeSize (2 bytes; 16-bit int); maximum number of samples in AM envelope
    - maxSamplingRate (4 bytes; 32-bit int); maximum sampling rate permitted

## Examples

Play sound #4 using an [ArCOM](http://www.google.com/url?q=http%3A%2F%2Fsites.google.com%2Fsite%2Fsanworksdocs%2Farcom&sa=D&sntz=1&usg=AOvVaw0q9tKPNJMCdKV2qsdKk90n) serial object in MATLAB:

```matlab
D = ArCOMObject('COM3', 115200);
D.write(['P' 3], 'uint8'); % Remember that sound position is 0-indexed!
clear D
```

Trigger sound #4 from the Bpod state machine

```matlab
LoadSerialMessages('AudioPlayer1', {['P' 3]}); % Set serial message 1

sma = NewStateMachine();

sma = AddState(sma, 'Name', 'PlaySound', ...
    'Timer', 0.1,...
    'StateChangeConditions', {'Tup', 'exit'},...
    'OutputActions', {'AudioPlayer1', 1}); % Sends serial message 1

SendStateMachine(sma);
RawEvents = RunStateMachine;
```