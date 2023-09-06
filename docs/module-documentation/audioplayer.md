# BpodAudioPlayer()

## Description

BpodAudioPlayer plays audio waveforms on trigger, using Ch1+2 of the[Analog Output Module](/site/bpoddocumentation/assembling-bpod/analog-output-module?authuser=0) (a precision voltage DAC with a dedicated microcontroller). This is a convenient way to deliver sound, in applications where anti-aliasing and other standard sound card features are not critical. For high resolution audio, use the [Bpod HiFi Module](../assembly/hifi-module-assembly.md).

To use AudioPlayer, you must [upload](../install-and-update/firmware-update.md) AudioPlayer firmware to the output module from [here](https://www.google.com/url?q=https%3A%2F%2Fgithub.com%2Fsanworks%2FBpod_AnalogOutput_Firmware&sa=D&sntz=1&usg=AOvVaw2mYfTij1ftDktlZRDZH-oN).

Two firmware versions are supported. Their differences are:

- AudioPlayer
    - 64 sounds max (up to 1M stereo samples each)
    - 96kHz sampling max
    - Loading sounds must occur while sounds are not playing (i.e. before the session or between trials)
- AudioPlayerLive (beta)
    - 20 sounds max (up to 1M stereo samples each)
    - 44.1kHz sampling max
    - Loading sounds permitted during playback
Both firmware versions feature very low latency playback on trigger (~100-200 microseconds).

Both also support arbitrary onset and offset AM envelopes up to 10k samples, useful for mitigating speaker "pop" and [spectral splatter](https://www.google.com/url?q=https%3A%2F%2Fen.wikipedia.org%2Fwiki%2FSpectral_splatter&sa=D&sntz=1&usg=AOvVaw1yuubncgr3cfg0_uLeXFC2).

After running Bpod, a BpodAudioPlayer object is initialized with the following syntax:

```matlab
A = BpodAudioPlayer('COM3');
```

Where COM3 is the analog output module's serial port.

The BpodAudioPlayer device is controlled in 2 ways:

- Setting the BpodAudioPlayer object's fields
- Calling the BpodAudioPlayer object's functions

## Object fields

- Port
    - [ArCOM](http://www.google.com/url?q=http%3A%2F%2Fsites.google.com%2Fsite%2Fsanworksdocs%2Farcom&sa=D&sntz=1&usg=AOvVaw0q9tKPNJMCdKV2qsdKk90n) Serial port object
- SamplingRate (Hz)
    - 1Hz-96kHz
    - With AudioPlayerLive firmware, this range is restricted to: 1Hz-44.1kHz.
- LoadMode
    - A string specifying a transmission scheme for loading sound data -> USB -> microSD card:
        - 'Fast' loads quickly, but may interrupt ongoing playback
        - 'Safe'\* loads more slowly, with care not to disrupt playback\*\*
        - \*AudioPlayerLive firmware only
        - \*\*Please verify this mode on your own system; USB host controllers and OS drivers may cause unexpected results.
            - To use 'Safe' mode effectively, you must have [PsychToolbox](http://www.google.com/url?q=http%3A%2F%2Fpsychtoolbox.org%2Fdownload%2F&sa=D&sntz=1&usg=AOvVaw0nKrMmbAW7F-ckOh8O3ATK) installed.
- Waveforms (Cell)
    - Local copy of all waveforms loaded to microSD with loadSound() function.
- TriggerMode (String)
    - A string specifying how to handle incoming bytes from the Bpod State Machine:
        - 'Normal' - plays the triggered sound, and ignores triggers that arrive during playback
        - 'Master' - triggers can force-start a new sound during playback.
        - 'Toggle' - starts playback. Stops playback if the same sound is triggered during playback.
- LoopMode (String)
    - A string specifying whether loop mode is on or off for each channel
        - 'On' loops the waveform until LoopDuration seconds, or until toggled off
        - 'Off' = plays the waveform once
- LoopDuration (Seconds)
    - In loop mode, specifies the duration to loop the waveform following a trigger.
    - Units = seconds
- AMEnvelope (Vector of fractional units in range [0 1])
    - An AM attenuation vector of up to 10,000 samples to apply on sound onset, and again in reverse on sound offset.
- BpodEvents (String)
    - 'Off': No feedback to the Bpod state machine
    - 'On' generates a serial event when starting playback, and again when playback finishes.
        - The event byte specifies which sound was started or stopped
        - e.g. byte 3 = sound 3 started (or stopped) playback.
## Object Functions

- loadSound(soundNumber, waveform)
    - Transmits a voltage waveform to the device.
    - soundNumber = index of the waveform (1-64, or 1-20 with AudioPlayerLive firmware)
    - waveform = a 1xN MATLAB array of voltages (samples in volts)
        - Voltages must be in the range [-5 to +5].
        - An error is raised if the waveform exceeds the allowed maximum (1M samples)
    - With AudioPlayerLive, the sound is stored in a holding buffer and cannot be triggered, until the next call to 'push' (see below).
        - This allows (for example), the next trial's sound#3 to be safely loaded while the previous sound#3 is still playing and triggerable.
    - Function returns an error if waveform was not successfully transmitted.
- push() \*AudioPlayerLive only
    - Sets all newly loaded sounds to be the current sounds, and frees the loading buffers for new sounds.
- play(soundNumber)
    - Plays a sound.
- stop()
    - Stops playback
- setupSDCard()
    - Must be run before first use with a new card.
    - Creates an empty data file on the module's microSD card, where waveforms will be stored.
    - Setup may take ~1 minute, depending on the microSD card's speed. A Sandisk U1 class card is strongly recommended.

## Cleanup

- Clear the BpodAudioPlayer object with clear:

```matlab
A = BpodAudioPlayer('COM3');

% ...Use the audio player

clear A
```

- Clearing the object releases the serial port, so other applications can access it.
- If a BpodAudioPlayer object is created locally inside a MATLAB function, the object is cleared automatically when the function returns.