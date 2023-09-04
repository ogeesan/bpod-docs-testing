# ArCOM
Original site: https://sites.google.com/site/sanworksdocs/arcom?authuser=0


**Ar**duino **Com**munication, or ArCOM, is a set of classes developed by Sanworks to simplify two types of data transaction:

1. Arduino <--USB--> MATLAB/GNU Octave
2. Arduino <--UART--> Arduino.

It provides single-line commands on both ends to transmit and receive scalars or arrays of different data types. You can find the rationale for this in the [Sanworks release announcement](https://sanworks.io/news/viewArticle?articleID=ArCOM1) for ArCOM.

Currently, ArCOM transmits scalar integers and arrays of integers:

- 8-bit types
    - uint8, int8, char
- 16-bit types
    - uint16, int16
- 32-bit types
    - uint32, int32

On the MATLAB side, ArCOM wraps three serial interfaces so your code can be used with any of: 
- Matlab's built-in Java-based serial interface
    - The PsychToolbox IOPort serial interface (significantly faster and lower latency!) 
- The GNU Octave serial interface

ArCOM configures serial port communication settings for you on both sides automatically (byte order, RTS/request to send, etc, to match the default configurations for MATLAB and Arduino).

ArCOM is under development, and may contain bugs. Please submit any bugs you encounter to: admin@sanworks.io

## Examples
Here's how you'd use ArCOM to send an unsigned 16-bit integer array from Arduino to MATLAB: 

Arduino code:
```arduino
#include "ArCOM.h" // Import the ArCOM library
ArCOM myUSB(SerialUSB); // Create an ArCOM wrapper for the SerialUSB interface

unsigned short myDataArray[10] = {0}; // Create a 1x10 uint16 array

void setup() {
        SerialUSB.begin(115200); // Initialize the USB serial port
    myUSB.writeUint16Array(myDataArray,10); // Send the array to MATLAB's buffer
}

void loop() {}
```

MATLAB code (with ArCOM.m in the MATLAB path):

```matlab
SerialPort1 = ArCOM('open', 'COM3', 115200); % Create and open the serial port
MyData = ArCOM('read', SerialPort1, 10, 'uint16'); % Read the array from the buffer
```

Another example, sending an array of 100 signed 32-bit ints from MATLAB to Arduino, 

```matlab
MyData = -151340:-151241;
ArCOM('write', SerialPort1, MyData, 'int16');
```

Arduino code:
```arduino
long myDataArray[100] = {0}; // Create a 1x100 int16 array
  ...
myUSB.readint16Array(myDataArray,10);
```

## Installation
### Download
License

ArCOM is free software, copyrighted by [Sanworks LLC](https://www.google.com/url?q=https%3A%2F%2Fsanworks.io%2F&sa=D&sntz=1&usg=AOvVaw2GJ0C0ZKoJZouWa9XwhIjC)

(C) 2021, Sanworks LLC, Rochester, NY, USA. 

Sanworks LLC has made ArCOM available to you under the terms of [GPL 3.0](http://www.google.com/url?q=http%3A%2F%2Fwww.gnu.org%2Flicenses%2Fgpl-3.0.en.html&sa=D&sntz=1&usg=AOvVaw0DSqmUfz_NB2fteqEA95x1). 

By downloading, you affirm that you have read the terms of this license and agreed to them.

Download from [Github](https://www.google.com/url?q=https%3A%2F%2Fgithub.com%2Fsanworks%2FArCOM&sa=D&sntz=1&usg=AOvVaw2OHUdnSsYwO-uBGXj0R92G)

### Setup
#### Arduino
- Extract /ArCOM/Arduino Library/ArCOM.zip
- Copy the extracted folder to your [Arduino libraries folder](https://www.google.com/url?q=https%3A%2F%2Fwww.arduino.cc%2Fen%2FGuide%2FLibraries%23toc5&sa=D&sntz=1&usg=AOvVaw216qiNSt24eIRXT_TyLnjT).
- Alternatively, import the .zip directly in Arduino: Sketch > Include Library > Add .Zip Library

#### MATLAB
- Add /ArCOM/Matlab/ to the [MATLAB path](http://www.google.com/url?q=http%3A%2F%2Fwww.mathworks.com%2Fhelp%2Fmatlab%2Fref%2Fpathtool.html&sa=D&sntz=1&usg=AOvVaw23WQiRH4ADewLctjtX_tC0).

## Usage
### Arduino
### MATLAB/Octave