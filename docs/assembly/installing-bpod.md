# Installing Bpod
This section details installing the Bpod software on the governing computer, and the Bpod firmware on the Bpod device.

## PC (Windows 10)
### The governing computer must have:
1. Windows 10
2. 8GB+ RAM
3. Preferably a 2.5GHz+ multi-core CPU (i.e. intel corei5/i7 series).
4. MATLAB r2013a or newer. 

> [!NOTE]
> If you are using Windows 7 and State Machine r2, you need to install the Teensy [serial port driver](https://wwwo.pjrc.com/teensy/serial_install.exe).

### Clone the MATLAB software repositry


### Install the MATLAB software

1. Open MATLAB
2. Set the Path
   1. In r2012a or later, choose "Set Path" from the "Environment" cluster in the "Home" tab. In r2011b or earlier, choose "File > Set path".
   2. For **Bpod_Gen2**
      1. Choose "Add" (NOT with subfolders)
      2. Select C:\Bpod_Gen2\ (or wherever your /Bpod_Gen2/ root folder is) and click "Add"
   3. For **Legacy Bpod**
      1. Choose "Add with subfolders"
      2. Select C:\Bpod\Bpod System Files\ and click "Add".
   4. Click "Save" at the bottom so MATLAB will know where Bpod is for all future sessions.
3. To verify that you were successful, make sure Bpod is unplugged and type Bpod at the MATLAB command prompt. 
   1. If all went well, Bpod will attempt to start and then fail with an error: "Bpod device not found". If something went wrong, you will get a different error.
   2. If the error says "Unidentified function or variable 'Bpod', you did not successfully add Bpod to the MATLAB path.
4. Install [PsychToolbox](http://psychtoolbox.org/download)
5. If you build the state machine yourself, upload Bpod's firmware[^1]

You should be able to run Bpod from that MATLAB prompt with `Bpod`

> [!WARNING]
> **For Bpod r0.5 - 1 users**
> On first plugging in the state machine, if MATLAB is open you will see a message: 
> ```
> Arduino Due detected.
> To use this device with MATLAB, install MATLAB Support Package for Arduino Hardware.
> ```
> DO NOT install the Arduino support package and definitely do not overwrite the state machine firmware! Bpod firmware communicates with MATLAB via MATLAB's built-in [serialport](https://au.mathworks.com/help/matlab/ref/serialport.html) interface.

[^1]: State machines purchased from the Sanworks Assembly Surface come with firmware pre-installed

## Ubuntu
