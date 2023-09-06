# BpodHiFi()

## Description

BpodHiFi plays audio waveforms on trigger, using the [Bpod HiFi Module](../assembly/hifi-module-assembly.md).

Example protocols using `BpodHiFi()` are given [here](https://www.google.com/url?q=https%3A%2F%2Fgithub.com%2Fsanworks%2FBpod_Gen2%2Fblob%2Fmaster%2FExamples%2FProtocols%2FSound%2FHiFiSound2AFC%2FHiFiSound2AFC.m&sa=D&sntz=1&usg=AOvVaw2x_38llCit2QSKRBKZuvW_), [here](https://www.google.com/url?q=https%3A%2F%2Fgithub.com%2Fsanworks%2FBpod_Gen2%2Fblob%2Fmaster%2FExamples%2FProtocols%2FSound%2FHiFiSound2AFC_TrialManager%2FHiFiSound2AFC_TrialManager.m&sa=D&sntz=1&usg=AOvVaw0T5u2m80bMIablE4Z79Ztc) and [here](https://www.google.com/url?q=https%3A%2F%2Fgithub.com%2Fsanworks%2FBpod_Gen2%2Fblob%2Fmaster%2FExamples%2FProtocols%2FSound%2FHiFiSound2AFC_Synth%2FHiFiSound2AFC_Synth.m&sa=D&sntz=1&usg=AOvVaw3XOE3SsWGE0TR6cpehFDbt).

After running Bpod, a BpodHiFi object is initialized with the following syntax:

```matlab
H = BpodHiFi('COM3');
```

Where COM3 is the HiFi module's serial port.

The HiFi module is controlled in 2 ways:

- Setting the BpodHiFi object's fields
- Calling the BpodHiFi object's functions

## Object Fields

- Port
    - [ArCOM](http://www.google.com/url?q=http%3A%2F%2Fsites.google.com%2Fsite%2Fsanworksdocs%2Farcom&sa=D&sntz=1&usg=AOvVaw0q9tKPNJMCdKV2qsdKk90n) Serial port object
- SamplingRate (Hz)
    - 44.1kHz, 48kHz, 96kHz and 192kHz (default) are supported.
    - The sampling rate is a global parameter, affecting all sounds loaded to the device.
- AMEnvelope (Vector of fractional units in range [0, 1])
    - An AM attenuation vector of up to 2,000 samples to apply on sound onset, and again in reverse on sound offset.
- HeadphoneAmpEnabled(String)
    - A string specifying whether the headphone amplifier on the base model HiFi module is enabled.
    - Note: The HD model does not have a headphone amp.
    - 'Off': Headphone amplifier is disabled. Audio output is still accessible via the RCA jacks.
    - 'On' Headphone amplifier enabled. Gain is controlled by setting the HeadphoneAmpGain field.
- HeadphoneAmpGain(Value)
    - Gain of the headphone amplifier (base model HiFi module only)
    - Value in range 0-63 sets the amplifier gain from 0 to its maximum value.
- DigitalAttenuation\_dB(dB)
        - An attenuation factor to constrain the output voltage range of the audio DAC (i.e. digital volume control)
        - dB must be in range [-103, 0] for the base model (-103dB = maximum attenuation, 0dB = no attenuation)
    - dB must be in range [-120, 0] for the HD model (-120dB = maximum attenuation, 0dB = no attenuation)
- SynthAmplitude(Value)
    - Amplitude of persistent synth waveform
    - Amplitude is in range [0, 1] where 0 = synth disabled and 1 = max amplitude
    - Note: synth amplitude will drop to 0 during playback of loaded waveforms, and resume its set point when no waveform is playing.
    - SynthFrequency(Value, Hz)
        - Frequency of synthesized waveform
        - Range = [20, 80,000]
        - SynthFrequency is ignored when the waveform selected is 'WhiteNoise'
    - SynthWaveform(String)
        - Waveform to synthesize continuously when SynthAmplitude is >0
        - Options are 'WhiteNoise' and 'Sine' (others may be added in future releases)
    - SynthAmplitudeFade(Samples)
        - Number of samples over which to execute a linear amplitude ramp to the new setpoint when a new SynthAmplitude is set
        - Range = [0, 1920000] where 0 is an instant transition

## Object Functions

- load(soundIndex, waveform, \*optionalArgs)
    - Transmits an audio waveform to the device.
    - soundIndex = index of the waveform (1-20) \* Note, in communications between the state machine and HiFi module these are 0-19
    - waveform = a 1xN (mono) or 2xN (stereo) MATLAB array of audio samples
        - Samples must be [double](https://www.google.com/url?q=https%3A%2F%2Fwww.mathworks.com%2Fhelp%2Fmatlab%2Fref%2Fdouble.html&sa=D&sntz=1&usg=AOvVaw17B9C0HEQVgIEpSw6t8UyN) type, in the range [-1, 1].
        - An error is raised if the waveform length exceeds the allowed maximum (5,760,000 samples)
    - The sound is stored in a holding buffer and cannot be triggered, until the next call to 'push' (see below).
        - This allows (for example), the next trial's sound#3 to be safely loaded while the previous sound#3 is still playing and triggerable.
    - Optional arguments are given as argument-value pairs:
        - (...'LoopMode', loopMode...)
            - loopMode = 0 (No looping)
            - loopMode = 1 (Use looping)
        - (...'LoopDuration', loopDuration...)
            - loopDuration is the amount of time to run looping, in seconds. 0 = infinite loop.
            - loopDuration is only valid at the current sampling rate - the sound should be reloaded if sampling rate is changed.
    - If used, optional arguments MUST be given in the order above (for efficiency)
    - Function will retry up to 5 times automatically if the waveform was not successfully transmitted.
- push()
    - Sets all newly loaded sounds to be the current sounds, and frees their loading buffers for new sounds.
- play(soundNumber)
    - Plays a sound.
- stop()
    - Stops playback

## Cleanup

- Clear the BpodHiFi object with clear:

```matlab
H = BpodHiFi('COM3');

% ...Use the HiFi module

clear H
```

- Clearing the object releases the serial port, so other applications can access it.
- If a BpodHiFi object is created locally inside a MATLAB function, the object is cleared automatically when the function returns.