# DDSModule()

## Description

DDSModule produces sine or triangle waves with adjustable frequency and amplitude, using an [AD9834](http://www.google.com/url?q=http%3A%2F%2Fwww.analog.com%2Fmedia%2Fen%2Ftechnical-documentation%2Fdata-sheets%2FAD9834.pdf&sa=D&sntz=1&usg=AOvVaw2ItKpIVxjJLVPCs9YTOUNh) [Direct Digital Synthesizer](https://www.google.com/url?q=https%3A%2F%2Fen.wikipedia.org%2Fwiki%2FDirect_digital_synthesizer&sa=D&sntz=1&usg=AOvVaw16HQxz0zV4KGiMlv6I9n6G) IC. Frequency can range from 1Hz to 100kHz. Amplitude can range from 0 to 600mV peak-to-peak.

The wave parameters can be adjusted with <100us latency from the state machine, or from a separate module that is configured to stream 16-bit values at up to 10kHz. An internal mapping function maps the 16-bit data stream to output frequencies.

The DDSModule MATLAB object allows frequency, amplitude and other parameters to be configured from MATLAB.

After running Bpod, a DDSModule object is initialized with the following syntax:

```matlab
D = DDSModule('COM3');
```

Where COM3 is the DDS module's serial port.

The DDSModule device is controlled by setting the DDSModule object's fields

## Object Fields

- Port
    - [ArCOM](http://www.google.com/url?q=http%3A%2F%2Fsites.google.com%2Fsite%2Fsanworksdocs%2Farcom&sa=D&sntz=1&usg=AOvVaw0q9tKPNJMCdKV2qsdKk90n) Serial port object
- Frequency (Hz)
    - Range = 1Hz : 100,000Hz.
    - Frequencies above 100,000Hz may be selected, with increasing prevalance of aliasing artifacts from the DDS IC's external 10MHz oscillator.
- Amplitude (Normalized Units)
    - Range = 0 : 1.
    - At range = 0, waveform peak to peak amplitude = ~0mV.
    - At range = 1, waveform peak to peak amplitude = ~600mV.
    - The waveform produced by the DDS IC has a DC offset. Waveform minima increase from 7mV to 40mV as the amplitude is increased from 0 to 1. Depending on demand, a future version of the device may include an output stage to center the output waveform on 0V.
- Waveform (String)
    - A string specifying the shape of the output waveform:
        - 'Sine' - a sinusoidal waveform
        - 'Triangle' - a triangle wave.
- MapFcn (String)
    - A string specifying a mapping function, used to convert a stream of 16-bit values arriving from the module input port into output frequencies in the range specified by OutputMapRange.
    - 'Linear' = Bits are mapped directly to frequencies in the range specified by OutputMapRange.
    - 'Exp' = The squares of 16-bit values are mapped to the linear space of frequencies.
        - The 'Exp' mapping function provides increased resolution in low frequencies.
- InputBitRange (1x2 Double, Bits)
    - A 1 x 2 vector specifying the range of input bits to map to the OutputMapRange (below).
    - Input bits arrive from a Bpod acquisition module (AnalogInput, RotaryEncoder) via the "Input Stream" connector.
- OutputMapRange (1x2 Double, Frequencies)
    - A 1 x 2 vector specifying the lower and upper frequency boundaries of the mapping function's output range.
        - [20 20000] maps the 16-bit input stream in range [0 2^16] to human auditory range of 20Hz-20kHz.
        - [2000 100000] maps the 16-bit input stream to the mouse auditory range of 2kHz-100kHz.

## Object Functions

- setAmplitudeBits(bits)
    - Sets the current output waveform amplitude by writing a specific bit value to the amplitude-control DAC.
        - This function is used in conjunction with setAmplitudeZeroCode() to calibrate the device's output amplitude
    - bits = An integer bit value, up to 13 bits max (Range = 0-8192)
- setAmplitudeZeroCode(bits)
    - Sets the amplitude-control DAC bit value that codes for a 0V p2p output waveform.
    - bits = An integer bit value, up to 13 bits max (Range = 0-8192)
    - The value is written to the device's EEPROM, and becomes the device's zero code for all subsequent use, even across power cycles.
    - NOTE: To calibrate, with the device plugged into an oscilloscope, use setAmplitudeBits() (above) to find the lowest bit-value at which the output waveform is 0V p2p. Then call setAmplitudeZeroCode with the bit value to store it.

## Cleanup

- Clear the DDSModule object with clear:
```matlab
D = DDSModule('COM3');

% ...Use the DDS Module

clear D
```

- Clearing the object releases the serial port, so other applications can access it.
- If a DDS Module object is created inside a MATLAB function, the object is cleared automatically when the function returns.