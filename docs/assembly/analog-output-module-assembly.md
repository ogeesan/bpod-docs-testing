# Analog Output Module
The analog output module plays voltage sequences on 4 or 8 output channels. Output is driven by the Analog Devices [AD5754R](https://www.google.com/url?q=https%3A%2F%2Fwww.analog.com%2Fen%2Fproducts%2Fad5754r.html&sa=D&sntz=1&usg=AOvVaw0OBIu8VgtX56aNPqnp5G2O) DAC.

Version 1 was released in 2017. It is powered by PJRC [Teensy 3.6](http://www.google.com/url?q=http%3A%2F%2Fhackaday.com%2F2016%2F08%2F17%2Fintroducing-the-teensy-3-5-and-3-6%2F&sa=D&sntz=1&usg=AOvVaw0eK_7K2oxeJegAxaUk4naE).

Version 2 was released in 2023. It is powered by PJRC [Teensy 4.1](https://www.google.com/url?q=https%3A%2F%2Fwww.pjrc.com%2Fstore%2Fteensy41.html&sa=D&sntz=1&usg=AOvVaw0Ix4K9Z2Inj9R6DoE9DxJP).

Several firmware versions are available for the device:
<!-- todo: links to firmwares -->
- WavePlayer firmware (default) plays from a programmable library of sampled waveforms on trigger.
- AudioPlayer firmware is specialized for stereo sound\*, sampled at up to 96kHz.
- PulsePal firmware plays parametric waveforms using the Pulse Pal Parameters.

Hardware Specs:

- V1: Arduino-compatible 180MHz ARM Cortex M4 processor with 12Mb/s USB data transfer
- V2: Arduino-compatible 600MHz ARM Cortex M7 processor with 480Mb/s USB data transfer
- Highly integrated digital to analog converter (DAC) - [AD5754R](http://www.google.com/url?q=http%3A%2F%2Fwww.analog.com%2Fmedia%2Fen%2Ftechnical-documentation%2Fdata-sheets%2FAD5724R_5734R_5754R.pdf&sa=D&sntz=1&usg=AOvVaw3BiGFJGIWHPp5t-uvALLf8).
- 16 bit voltage precision
- 6 output range settings: 0:5V, 0:10V, 0:12V, -5:+5V, -10:+10V, -12:+12V
- 16GB microSD memory for waveform data

Specs with WavePlayer firmware (arbitrary waveform playback on trigger):

- Max sampling rate: 100kHz (2 channels active), 50kHz (4 channels active)
- Playback latency on trigger: <100μs (waveform pre-selection not required)
- Max waveform size: 1M samples
- Trigger signals from Bpod can replace or toggle the currently playing waveform

Firmware for the analog output module is available [here](https://www.google.com/url?q=https%3A%2F%2Fgithub.com%2Fsanworks%2FBpod_AnalogOutput_Firmware&sa=D&sntz=1&usg=AOvVaw2mYfTij1ftDktlZRDZH-oN).

\*The analog output module is a general purpose DAC. High definition sound playback is available via the [HiFi module](../serial-interfaceshifi-module-serial-interface.md).
## Bill of Materials
### 4Ch AOM v1
<iframe width=1000 height=300 jsname="L5Fo6c" jscontroller="usmiIb" jsaction="rcuQ6b:WYd;" class="YMEQtf L6cTce-purZT L6cTce-pSzOP KfXz0b" sandbox="allow-scripts allow-popups allow-forms allow-same-origin allow-popups-to-escape-sandbox allow-downloads allow-modals" frameborder="0" aria-label="Spreadsheet, Raspberry Pi Shim BOM" allowfullscreen="" src="https://docs.google.com/spreadsheets/d/1sg_CjjEeOa-BNOvEDrPs1JLkYYPr1-5xbWjqEkAero8/htmlembed?authuser=0"></iframe>

### 4Ch AOM v2
<iframe width=1000 height=600 jsname="L5Fo6c" jscontroller="usmiIb" jsaction="rcuQ6b:WYd;" class="YMEQtf L6cTce-purZT L6cTce-pSzOP KfXz0b" sandbox="allow-scripts allow-popups allow-forms allow-same-origin allow-popups-to-escape-sandbox allow-downloads allow-modals" frameborder="0" aria-label="Spreadsheet, AnalogOutputModule v2 BOM" allowfullscreen="" src="https://docs.google.com/spreadsheets/d/1Du9rV7v4Srbf4028XDVEv7t-nSe9Na5oGJ34DrM0urQ/htmlembed?authuser=0"></iframe>

### 8Ch AOM
<iframe width=1000 height=700 jsname="L5Fo6c" jscontroller="usmiIb" jsaction="rcuQ6b:WYd;" class="YMEQtf L6cTce-purZT L6cTce-pSzOP KfXz0b" sandbox="allow-scripts allow-popups allow-forms allow-same-origin allow-popups-to-escape-sandbox allow-downloads allow-modals" frameborder="0" aria-label="Spreadsheet, AnalogOutputModule BOM 8ch" allowfullscreen="" src="https://docs.google.com/spreadsheets/d/1yOUCxAatLmd47a3QM8tikrkpaoZApwOKP3F7127gq_w/htmlembed?authuser=0"></iframe>