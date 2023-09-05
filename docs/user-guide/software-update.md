# Software update

## Automatic

!!! note
    An experimental auto-updater is included with the latest Bpod software. It is not available on all platforms, and as new software it carries the risk of malfunction in your particular MATLAB + PC configuration. If you choose to use it,

- Manually back up your Bpod_Gen2 folder as a precaution.
    - Run `UpdateBpodSoftware()` at the MATLAB command line, and follow all prompts
    - Please report any bugs or unexpected behavior to support@sanworks.io

## Manual

To manually update your Bpod software to the newest stable release:

### If using SourceTree
<!-- check if SourceTree is the best wording to use here -->
- Close MATLAB and open SourceTree
    - Select the 'Bpod_Gen2' repository tab. If the tab is missing, click '+' and add it back.
    - Click 'Pull'
    - Double-click the latest version in the 'master' branch under 'Branches'


### If NOT using SourceTree

- Close MATLAB
- Backup your Bpod_Gen2 folder to a safe location
- Delete Bpod_Gen2 from its original location (Do not change the MATLAB path)
- Download the latest software from here: [https://github.com/sanworks/Bpod\_Gen2](https://www.google.com/url?q=https%3A%2F%2Fgithub.com%2Fsanworks%2FBpod_Gen2&sa=D&sntz=1&usg=AOvVaw0hZOqBP6mI4rPtPR76Nb5k)
    - Click 'Clone or Download' and select 'Download Zip'
    - Extract the downloaded Zip file to the original Bpod_Gen2 location
    - If desired, rename the root folder to remove '-master'
    - Make sure your MATLAB path includes the Bpod_Gen2 root folder (subfolders not required)