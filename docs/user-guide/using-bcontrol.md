# Using BControl

## Status

As of firmware v18, Bpod supports [BControl software](http://www.google.com/url?q=http%3A%2F%2Fbrodywiki.princeton.edu%2Fbcontrol%2Findex.php%2FMain_Page&sa=D&sntz=1&usg=AOvVaw2s70xjKBiwq-O6G1vEp0LE), developed and maintained by Brody Lab at Princeton University.

BControl support is in public beta. Please report issues on the [Sanworks support forums](https://www.google.com/url?q=https%3A%2F%2Fsanworks.io%2Fforums%2F&sa=D&sntz=1&usg=AOvVaw3h3BjfrtOIbg6esu1U-Ex_).

To use BControl with Bpod hardware, you need to apply a patch to the latest BControl software. Follow these instructions:

## Setup

1. Download a fresh copy of B-control, following the instructions [here](http://www.google.com/url?q=http%3A%2F%2Fbrodywiki.princeton.edu%2Fbcontrol%2Findex.php%2FInstallation_Guide%23Installation_Guide_for_Advanced_Users_.28core_code_and_behavioral_protocol_developers.29&sa=D&sntz=1&usg=AOvVaw2h1hRlM0u4aDdkXZpllmp9) (MATLAB-side instructions ONLY).
2. Make a backup copy of your BControl download, in case something fails.
3. If you have not already done so, install [PsychToolbox](http://www.google.com/url?q=http%3A%2F%2Fpsychtoolbox.org%2Fdownload%2F&sa=D&sntz=1&usg=AOvVaw0nKrMmbAW7F-ckOh8O3ATK).
4. [Load](../install-and-update/firmware-update.md) the B-control firmware variant for your state machine (from the state machine firmware repository, [here](https://www.google.com/url?q=https%3A%2F%2Fgithub.com%2Fsanworks%2FBpod_StateMachine_Firmware%2Ftree%2Fv22%2FPreconfigured%2FFor%2520BControl&sa=D&sntz=1&usg=AOvVaw0ITFEY6McFr4B1-TU54UA4)).
5. Open MATLAB, and from the command prompt run: `Bpod`;
6. Once Bpod opens, run: `PatchBControl`;
7. Follow the prompt to identify BControl's root directory, and confirm that you have made a backup copy
8. The software will install a patch, allowing you to run BControl whenever Bpod is open.
9. If you plan to use sound stimuli with BControl, load AudioPlayerLive firmware onto a 4-channel analog output module
10. If you plan to use analog scheduled waves, load WavePlayer firmware onto a separate 4-channel analog output module
11. If you added hardware modules in steps 9-10
    1. Press the "refresh" button on the Bpod console (or restart Bpod). Detected modules should appear in tabs on the console.
    2. Run "USB Pairing" from the Bpod console (USB icon), to associate each module with its USB port.

## Running BControl

1. With Bpod open, run newstartup; dispatcher('init');
2. Note for prior users: you no longer need to navigate to the BControl home directory.

## Tips

- Choice of Bpod state machine
    - State machine v0.7-1.0 provides:
        - 8 Nose pokes
        - 8 scheduled waves
        - 8 non-standard happening specs (non-standard = beyond the events in BControl's legacy state matrix)
        - 1 valve open at a time
    - State machine r2 provides:
        - 4 Nose pokes
        - 20 scheduled waves
        - 20 non-standard happening specs
        - 4 valves open at a time
        - Significantly faster USB data transmission
        - Significantly more CPU available for mods
- Choice of computer and MATLAB
    - Required: a PC running Windows 7 or 10
    - Core i7 + 16GB RAM recommended, especially if other apps (i.e. an IR camera) will run in parallel
    - MATLAB r2011a - r2014a supported, r2014a strongly recommended.
- Necessary Bpod Modules
    - For sound: [HiFi Module HD](../assembly/hifi-module-assembly.md)
    - For analog scheduled waves: [Analog Output Module](../assembly/analog-output-module-assembly.md)
- Limitations
    - Compiled C "on the fly" cannot be supported
    - untrigger\_on\_down is not currently supported
    - scheduled wave refraction is not currently supported
- Settings\_Custom.conf
    - When running the patch, a settings\_custom.conf file is auto-generated for the state machine version connected, incl. the correct path to your protocol and data folders.
    - 3 ports are enabled. More can be added by uncommenting the relevant DIO lines.
    - 2 stimulator channels are enabled: stim1, stim2. These are mapped to Bpod's BNC output channels.
    - At the top of the conf file, 'use\_bdata' is set to 0. If you are in Brody Lab, set this variable to 1, to use the local SQL server.
- VALIDATE
    - BControl support is in public beta. Please ensure that the system is doing what you think it is, before acquiring scientific data. This means verifying event timing in your protocol (with an oscilloscope), measuring audio latency and waveform quality, the correct flow of states and events, etc.