# HiFi Module
The Bpod HiFi Module stores audio waveforms and play them back on trigger.

Audio waveforms are rendered by [HiFiBerry](https://www.google.com/url?q=https%3A%2F%2Fwww.hifiberry.com%2F&sa=D&sntz=1&usg=AOvVaw0MmWWvePk-wEiPqorUFfaO) [DAC2 Pro](https://www.google.com/url?q=https%3A%2F%2Fwww.hifiberry.com%2Fdocs%2Fdata-sheets%2Fdatasheet-dac2-pro%2F&sa=D&sntz=1&usg=AOvVaw2UpAYi7CokH-lqQp_sSOAo) and [DAC2 HD](https://www.google.com/url?q=https%3A%2F%2Fwww.hifiberry.com%2Fdocs%2Fdata-sheets%2Fdatasheet-dac-hd%2F&sa=D&sntz=1&usg=AOvVaw15LwCjfSITr3cyBx33L5hl) cards.

Our [firmware](https://www.google.com/url?q=https%3A%2F%2Fgithub.com%2Fsanworks%2FBpod_HiFi_Firmware&sa=D&sntz=1&usg=AOvVaw2ZMAy5LNY3KO1RRxuJC9g3) directly interfaces these cards with [Teensy 4.1](https://www.google.com/url?q=https%3A%2F%2Fwww.pjrc.com%2Fstore%2Fteensy41.html&sa=D&sntz=1&usg=AOvVaw0Ix4K9Z2Inj9R6DoE9DxJP) for superior timing\n\s+\n precision.

An isolated TTL output channel signals audio playback onset and offset.

Synth functions are provided for programmatic control of white noise and pure tones.

Hardware for the two modules is identical except for the enclosure and HiFiBerry card used.

A single firmware file can be set to compile for either module.

Key specs are:

- Sampling rate: 44.1, 48, 96 or 192kHz
- Bit depth: 16
- Audio channels: 2 (Stereo)
- Audio waveform slots: 20
- Max audio samples per slot: 5,760,000
- Playback latency on trigger: 0.22ms +/- 0.01ms
- USB audio data transfer speed: 40-50Mb/s
- Audio signal voltage range: +/- 3V
- TTL sync voltage: 3.3V
- Max AM onset/offset Envelope samples: 2000
- Synth waveforms supported: White Noise, Sinef

The HiFi module is controlled from MATLAB with the [BpodHiFi class](../module-documentation/hifi-module.md).

Its interfaces to the Bpod State Machine and USB are documented [here](../module-serial-interfaces/hifi-module-serial-interface.md).

This table contrasts the HiFiBerry cards in the 'SD' and 'HD' versions of the module:

<iframe width=400 jsname="L5Fo6c" jscontroller="usmiIb" jsaction="rcuQ6b:WYd;" class="YMEQtf DnR2hf L6cTce-purZT L6cTce-pSzOP KfXz0b" sandbox="allow-scripts allow-popups allow-forms allow-same-origin allow-popups-to-escape-sandbox allow-downloads allow-modals" frameborder="0" aria-label="Spreadsheet, HiFi Module Comparison" style="height: 268px" allowfullscreen="" src="https://docs.google.com/spreadsheets/d/1sHyfGqV-IkvTB1UVjjvFqv3ITArC2krG1VzMo9Z8gLc/htmlembed?authuser=0"></iframe>

## Bill of Materials
<iframe height=700 width=1000 jsname="L5Fo6c" jscontroller="usmiIb" jsaction="rcuQ6b:WYd;" class="YMEQtf L6cTce-purZT L6cTce-pSzOP KfXz0b" sandbox="allow-scripts allow-popups allow-forms allow-same-origin allow-popups-to-escape-sandbox allow-downloads allow-modals" frameborder="0" aria-label="Spreadsheet, HiFi Module BOM" allowfullscreen="" src="https://docs.google.com/spreadsheets/d/12IV6EH0wJ04lvYyQvK4MSJ5uNonhKb-8kxIhhijEA9U/htmlembed?authuser=0"></iframe>