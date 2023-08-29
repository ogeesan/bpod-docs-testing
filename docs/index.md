# Bpod Github Markdown Wiki Test
Welcome to the Bpod Wiki.

!!! warning

    :stop_sign: This is a test version of the Bpod wiki to examine the feasability of moving it to Github. The official wiki maintained by Sanworks is hosted [here](https://sites.google.com/site/bpoddocumentation/home?authuser=0).
    
    :construction: This test wiki does not contain all items, and some of the items included have been modified.

    :bulb: Some new additions exist.

Bpod is an open source system for real-time behavior measurement in tasks consisting of multiple experimental trials. Experiment software is written in MATLAB, and device firmware is written in [Arduino](https://www.arduino.cc/). Hardware can be assembled with DIY desktop manufacturing methods - hand-soldering, 3-D printing, laser cutting and hand-tapping. The system architecture is low cost, and supremely hackable - precisely what is necessary to explore a space of behavioral metrics, or to train test subjects with high throughput. This wiki contains instructions for assembly and programming.

<p align="center">
<img src="./images/state-machine.jpg" alt="Alt text" width="300"/>
</p>

### Download
The Bpod repository is available here:

- Current release (Mandatory for state machine r0.7+, compatible with r0.5)
  - MATLAB software: https://github.com/sanworks/Bpod_Gen2 
  - CAD: https://github.com/sanworks/Bpod-CAD
- Legacy release (Bpod state machine 0.5 only)
  - https://github.com/sanworks/Bpod/tree/master

## About Bpod

Bpod was initially developed in [Kepecs Lab](http://kepecslab.cshl.edu/) at Cold Spring Harbor Laboratory, as a project alongside the lead developer's thesis research. It is maintained by [Sanworks LLC](https://sanworks.io/), a company dedicated to developing Bpod and other open neuroscience tools.

Bpod builds on the central design concept of [B-control](http://brodywiki.princeton.edu/bcontrol/index.php/Main_Page), a system provided by [Brody Lab](http://brodylab.org/) at Princeton University for rodent behavior measurement. Experimental trials are constructed in MATLAB as [finite state machines](https://en.wikipedia.org/wiki/Finite-state_machine), and executed on a separate real-time Linux computer. Bpod combines this parallel processing model with the accessibility of embedded computing in the Arduino language. Bpod provides a rich suite of software tools in high level interpreted computing environments for protocol development and online analysis, while real-time processing is delegated to an Arduino microcontroller network governed by finite state machine firmware.

```mermaid
---
title: Bpod's design concept
---
sequenceDiagram
actor e as Experimenter
participant matlab as Computer
participant bpod as Bpod State Machine
participant external as External devices

e->>matlab: Start protocol
Note over matlab: Prepare session
matlab ->> external: Configure with user-friendly functions
loop
Note over matlab: Prepare state machine
matlab ->> bpod: Run state machine
activate bpod
Note over bpod: Rodent behavior
bpod -->external: High speed interaction
bpod ->> matlab: Send trial data
deactivate bpod
Note over matlab: Save data
end
```


We love hearing about the awesome [science](https://sanworks.io/science/science.php) that is generated with Bpod! 

Please post on the [Forums](https://sanworks.io/forums/) with your questions and feedback, or [email us](https://sanworks.io/about/contact.php) directly.