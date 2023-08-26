Welcome to the Bpod Wiki.

Bpod is an open source system for real-time behavior measurement in tasks consisting of multiple experimental trials. Experiment software is written in MATLAB, and device firmware is written in [Arduino](https://www.arduino.cc/). Hardware can be assembled with DIY desktop manufacturing methods - hand-soldering, 3-D printing, laser cutting and hand-tapping. The system architecture is low cost, and supremely hackable - precisely what is necessary to explore a space of behavioral metrics, or to train test subjects with high throughput. This wiki contains instructions for assembly and programming.

```mermaid
sequenceDiagram
actor e as Experimenter
participant matlab as Computer
participant bpod as Bpod State Machine

e->>matlab: Start protocol
Note over matlab: Prepare session
loop
Note over matlab: Prepare state machine
matlab ->> bpod: Run state machine
activate bpod
Note over bpod: Rodent behavior
bpod ->> matlab: Send trial data
deactivate bpod
Note over matlab: Save data
end

```

```
readme.md
content/
    assembling-bpod.md
    installing-bpod.md
    firmware-update.md
    software-update.md
    bench-testing.md
    user-guide/
        hardware-overview.md
        general-concepts.md
        protocol-development.md
        function-reference.md
        using-bcontrol.md
        serial-interfaces.md
    module-guides/
        rotary-encoder-module.md
    stimulation.md
    license-and-liability.md
    download.md
    contact.md

```

- [ ] Improve module documentation
  - [ ] Move module function reference and serial interface into the same file


Bpod was initially developed in [Kepecs Lab](http://kepecslab.cshl.edu/) at Cold Spring Harbor Laboratory, as a project alongside the lead developer's thesis research. It is maintained by [Sanworks LLC](https://sanworks.io/), a company dedicated to developing Bpod and other open neuroscience tools.

![](docs/images/state-machine.jpg)

Bpod builds on the central design concept of [B-control](http://brodywiki.princeton.edu/bcontrol/index.php/Main_Page), a system provided by [Brody Lab](http://brodylab.org/) at Princeton University for rodent behavior measurement. Experimental trials are constructed in MATLAB as [finite state machines](https://en.wikipedia.org/wiki/Finite-state_machine), and executed on a separate real-time Linux computer. Bpod combines this parallel processing model with the accessibility of embedded computing in the Arduino language. Bpod provides a rich suite of software tools in high level interpreted computing environments for protocol development and online analysis, while real-time processing is delegated to an Arduino microcontroller network governed by finite state machine firmware.

We love hearing about the awesome [science] that is generated with Bpod! 

Please post on the [Forums](https://sanworks.io/forums/) with your questions and feedback, or [email us](https://sanworks.io/about/contact.php) directly.

# Table of Contents
- Assembly
  - Hardware
  - Installing  Bpod
  - Firmware updates
  - Software updates
- Module guides
  - Analog Input Module
  - Analog Output Module
  - DDS Module
  - Ethernet Module
  - HiFi Module
  - I2C Messenger Module
  - Rotary Encoder Module
  - Port Array Module
  - Valve Driver Module
- User guide
  - Function reference
  - Modules
  - Protocol development
  - Serial interfaces