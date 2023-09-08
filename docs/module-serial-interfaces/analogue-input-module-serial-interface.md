# Analog Input Module Serial Interface

### Description

Allows the state machine or PC to acquire analog waveforms and extract voltage threshold events using the analog input module.

Requires an analog input module board with firmware loaded from:
- [https://github.com/sanworks/Bpod\_AnalogInput\_Firmware](https://github.com/sanworks/Bpod_AnalogInput_Firmware)
The analog input module must be connected to a free serial port on the state machine module.

### State Machine Command Interface

The state machine command interface consists of bytes sent from the Bpod state machine to the analog input module to start and stop acquisition, data streaming or threshold event transmission.

- Byte **255*- (reserved): Returns module info to state machine
- '**L**' (ASCII 80): **Start/Stop logging analog data**
    - 'L' (byte 0) must be followed by one byte:
        - Byte 1: A byte indicating start (1) or stop (0). 
- '**E**' (ASCII 70): **Start/Stop threshold event transmission to the state machine**
    - 'E' (byte 0) must be followed by two bytes:
        - Byte1: A byte indicating the event target: 0 = USB\*, 1 = state machine.
        - Byte 2: A byte indicating whether to start (1) or stop (0) event transmission.
    - Note: Channels must be selected previously, to transmit events (see op 'K' below).
- '**S**' (ASCII 83):  **Start/Stop raw data streaming to USB, or to a module*- (Analog Output, DDS, etc)
    - 'S' must be followed by two bytes:
        - Byte1: A byte indicating the data stream target: 0 = USB\*, 1 = output module.
        - Byte 2: A byte indicating whether to start (1) or stop (0) event transmission.
- '**#**' (ASCII 35): **Sync byte from the state machine**. This byte can be sent from any state to mark the time in the analog USB data stream. This simplifies alignment of the USB data stream with behavioral events. 
    - '#' must be followed by one byte:
        - Byte N: A byte with a user-determined value, passed from the state machine. From state machine output actions, use: {'AnalogIn1', ['#' N]} where N is any user-determined value that would be useful to have in the analog record  (an event code, a state number, etc)

### SerialUSB Command Interface

The SerialUSB command interface allows configuration of the analog input module with MATLAB or Python before a trial begins. It also allows data return from the module's onboard microSD card. The [`AnalogInputModule()`](../module-documentation//analog-input-module.md) plugin for Bpod/MATLAB wraps this interface. The first two commands are the same as for the state machine interface (though an acknowledgement byte = 1 is returned in each case), and additional commands follow:

- '**R**' (ASCII 82): **Set the input range for each channel*- (default = +/-10V). 'R' (byte 0) is followed by 1 byte for each channel (8 total):
    - Bytes 1-8: Each channel's range index. Range indexes are:
        - 0: -10V to +10V
        - 1: -5V to +5V
        - 2: -2.5V to +2.5V
        - 3: 0V to +10V
    - The analog input module returns a byte (1) to confirm that it has finished setting the range.
    - NOTE: The waveforms are stored on the device in bits. The bits are computed for the current range. When you change the output range of the device, the waveforms already stored will be compressed or expanded in amplitude with respect to the new range. 
- '**A**' (ASCII 65): **Set the number of actively sampled channels**  'A' (byte 0) is followed by:
    - Byte 1: The number of consecutive, actively sampled channels (beginning with Ch0).
    - The analog input module returns a byte (1) to confirm that it has finished setting nChannels.
    - Note: Sampling more channels takes more time, and reduces the maximum possible sampling rate. Set this value to the minimum number of channels used.
- '**F**' (ASCII 70): **Set sampling frequency**. 'F' (byte 0) is followed by:
    - Bytes 1-4: a 32-bit integer indicating the new sampling frequency (in Hz).
    - The analog input module returns a byte (1) to confirm that it has finished setting sampling frequency.
- '**O**' (ASCII 79): **Hand-shake, retrieve firmware version, and reset module parameters to defaults**.
    - The module replies with the following sequence of bytes:
        - 161 (1 byte;  a unique acknowledgement byte)
        - firmwareVersion (4 bytes; 32-bit int)
- '**D**' (ASCII 68): **Retrieve captured analog data

 from the module's microSD card**. The module replies with the following bytes:
    - nSamplesAcquired (4 bytes; 32-bit int); number of samples acquired since last call to ['L' 1] (see above)
    - for (each sample)
        - for (each actively sampled channel)
            - a sample (2 bytes; 16-bit int)
    - Samples are returned as bytes in the current voltage range, and must be converted to volts by the software.
- '**W**' (ASCII 87): **Set maximum number of samples to acquire**. 'W' (byte 0) is followed by:
    - Bytes 1-4: a 32-bit integer indicating the new maximum number of samples to acquire following a call to ['L' 1] (see above)
    - The analog input module returns a byte (1) to confirm that it has finished setting the maximum number of samples.
- '**K**' (ASCII 75): **Enable/disable threshold event transmission for each channel**. 'K' (byte 0) is followed by:
    - Bytes 1-8: each channel's setting: 1 (on) or 0 (off)
    - The analog input module returns a byte (1) to confirm the new configuration.
    - Note: Event transmission must be enabled globally for this configuration to be observed (see op 'E' above)
- '**T**' (ASCII 84): **Set voltage thresholds and reset voltages for each channel**. 'T' (byte 0) is followed by:
    - Bytes 1-16: Voltage threshold for each channel in bits (2 bytes; 16-bit int)\*8 channels. Bits indicate voltage in the current range.
        - Bytes 17-32: Reset voltage for each channel in bits (2 bytes; 16-bit int)\*8 channels. Bits indicate voltage in the current range.
    - The analog input module returns a byte (1) to confirm that it has finished setting the thresholds and reset voltages.
- '**Z**' (ASCII 90): **Set a channel's zero-code to its current voltage**. 'Z' (byte 0) is followed by:
    - Byte 1: Channel (1 byte); the channel to zero.
    - Note: The device will measure the channel 100 times, and offset its range so that the average measurement = 0V. Be sure to set the input signal to 0V first.
    - Note: This function helps compensate for the DAC's zero-code error (typically <7 LSBs, also dependent on range configuration).
    - Note: Any changes to range of any channel will reset zero-code offset to 0 for all channels (i.e. If using zero-code correction, this function must be called after each range adjustment).

### Examples

Acquire 1 channel of analog data at 10kHz from an [ArCOM](http://sites.google.com/site/sanworksdocs/arcom) serial object in MATLAB: 

```matlab
A = ArCOMObject('COM43', 115200);
A.write(['A' 1], 'uint8'); % Set the module to read only 1 channel (channel 1)
A.write('F', 'uint8', 10000, 'uint32'); % Set the module to sample at 10kHz
A.write(['L' 1], 'uint8'); % Start logging
Ack = A.read(3, 'uint8'); % Read acknowledgement bytes
pause(1); % Wait 1 second for data to be acquired
A.write(['L' 0], 'uint8'); % Stop logging
A.write('D', 'uint8'); % Request logged data
nSamples = A.read(1, 'uint32'); % read the number of samples acquired
Data = A.read(nSamples , 'uint32'); % read the data
clear A
```

Start+Stop logging analog data from the Bpod state machine, when the subject enters+exits port 2 (Assuming the subject has run Bpod and created a [`AnalogInputModule()`](../module-documentation//analog-input-module.md) object called A).

```matlab
LoadSerialMessages('AnalogIn1', {['L' 1], ['L' 0]});  % Set serial messages 1+2 to start+stop logging
sma = NewStateMachine();
sma = AddState(sma, 'Name', 'WaitForPort2Entry', ...
    'Timer', 0,...
    'StateChangeConditions', {'Port2In', 'WaitForPort2Exit'},...
    'OutputActions', {}); % Sends serial message 1 (Start logging)
sma = AddState(sma, 'Name', 'WaitForPort2Exit', ...
    'Timer', 0,...
    'StateChangeConditions', {'Port2Out', 'StopLogging'},...
    'OutputActions', {'AnalogIn1', 1}); % Sends serial message 1 (Start logging)
sma = AddState(sma, 'Name', 'StopLogging', ...
    'Timer', 0,...
    'StateChangeConditions', {'Tup', 'exit'},...
    'OutputActions', {'AnalogIn1', 2}); % Sends serial message 2 (Stop logging)
SendStateMachine(sma);
RawEvents = RunStateMachine;
Data = A.getData; % Now that the trial has ended, get data acquired while the subject was in the port
```
