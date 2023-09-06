# Installing Bpod
This section details installing the Bpod software on the governing computer, and the Bpod firmware on the Bpod device.

## PC (Windows 10)
### Requirements
1. Windows 10
2. 8GB+ RAM
3. Preferably a 2.5GHz+ multi-core CPU (i.e. intel corei5/i7 series).
4. MATLAB r2013a or newer. 

!!! note
    If you are using Windows 7 and State Machine r2, you need to install the Teensy [serial port driver](https://www.google.com/url?q=https%3A%2F%2Fwww.pjrc.com%2Fteensy%2Fserial_install.exe&sa=D&sntz=1&usg=AOvVaw25jfvYD6VWoNzSSCTbUj4Y).

### Clone the MATLAB software repository
Bpod's MATLAB software is frequently updated with new features and improvements. 
To keep Bpod current for everyone, we use a [revision control](http://www.google.com/url?q=http%3A%2F%2Fen.wikipedia.org%2Fwiki%2FRevision_control&sa=D&sntz=1&usg=AOvVaw1B_ySYzuOh-Dql7I0rO0fo) system called Git.

Git's command line usage can be tricky, so instead, we recommend a simple and powerful user interface to Git called SourceTree.
<!-- todo: Is this really the best suggestion for Git? Github Desktop works just fine -->

1. Download and install [SourceTree](http://www.google.com/url?q=http%3A%2F%2Fwww.sourcetreeapp.com%2F&sa=D&sntz=1&usg=AOvVaw0Z-GdVxu4303g9vODKJ6_K). The default options during install are correct - and you can use the embedded git client when prompted, if you don't already have it.
2. From SourceTree, clone the remote repository. 
    1. If you are using Bpod 0.5 (legacy), use: https://github.com/sanworks/Bpod.git
    2. If you are using Bpod 0.7+ or want to use Bpod 0.5 with current features, use: https://github.com/sanworks/Bpod_Gen2.git

If all went well, this should copy the latest Bpod software to your computer.

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

!!! warning
    **For Bpod r0.5 - 1 users**: 
    On first plugging in the state machine, if MATLAB is open you will see a message: 
    ```
    Arduino Due detected.
    To use this device with MATLAB, install MATLAB Support Package for Arduino Hardware.
    ```
    DO NOT install the Arduino support package and definitely do not overwrite the state machine firmware! Bpod firmware communicates with MATLAB via MATLAB's built-in [serialport](https://au.mathworks.com/help/matlab/ref/serialport.html) interface.

[^1]: State machines purchased from the Sanworks Assembly Surface come with firmware pre-installed

## Ubuntu
These are instructions for setting up Bpod on a computer running Ubuntu

We recommend at least an Intel Corei5 processor and 8GB of RAM.

This tutorial assumes you have loaded Bpod's firmware if you self-assembled the state machine.

1. Install Ubuntu (64bit) with >100GB partition
    1. Update Ubuntu to current version if necessary
2. Install MATLAB. 
    1. When prompted, check “install script”.
3. Run MATLAB (if default install location, from terminal: sudo /usr/local/MATLAB/RXXXX/bin/matlab
4. Install PsychToolbox:
    1. Download PsychToolbox by following instructions for linux [here](http://www.google.com/url?q=http%3A%2F%2Fpsychtoolbox.org%2Fdownload%2F%23Linux&sa=D&sntz=1&usg=AOvVaw3f0me0x_GWXOv64cwC4-lS). Use the SUBVERSION based installation.
    2. Allow all patches and use default settings when prompted.
5. Copy Bpod files from [here](https://www.google.com/url?q=https%3A%2F%2Fgithub.com%2Fsanworks%2FBpod_Gen2&sa=D&sntz=1&usg=AOvVaw0hZOqBP6mI4rPtPR76Nb5k) and add /Bpod_Gen2/Bpod System Files to MATLAB path
6. Close MATLAB
7. Open a terminal window and add yourself to the “dialout” group:
8. `sudo usermod -a -G dialout kepecslab` (if kepecslab is your username)
9. Restart matlab as root (same as step 3)
10. Run Bpod from the command prompt.

Note: Gnome ModemManager does not play well with Arduino; it discovers Arduino and probes it with bytes that interfere with communications, possibly leaving Bpod in a state where it expects bytes that will never arrive.
If you experience issues starting Bpod and you're using Gnome, consider [disabling the modem manager](https://www.google.com/search?ei=TojoWuHnKOam_QbF8bWIDw&q=Gnome+ModemManager+arduino&oq=Gnome+ModemManager+arduino).

Note: A previous installation step was automated in the current version.
If you are not using PsychToolbox, MATLAB needs to be instructed that ports of the form /dev/ttyACMx are valid serial ports.
On first run, Bpod should automatically handle this, and then ask you to restart MATLAB.
If it fails, do the following:

1. from terminal, launch the editor as root. Run: sudo gedit
2. Paste the following line into the text editor: -Dgnu.io.rxtx.SerialPorts=/dev/ttyS0:/dev/ttyS1:/dev/USB0:/dev/ttyACM0
3. Save the file as java.opts to the following location:  /usr/local/MATLAB/R2011a/bin/glnxa64