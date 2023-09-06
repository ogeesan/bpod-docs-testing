---
icon: material/arrow-left-right
---

# Serial interfaces
A "serial interface" is a communication protocol that involves sending and receiving data one byte at a time, serially. 

This allows computer software (e.g. MATLAB, Python) to communicate with the hardware via USB, or for the hardware to communicate with other hardware via serial ports (via ethernet CAT5e cables).

Bpod uses the [ArCOM](https://github.com/sanworks/Bpod_Gen2/blob/master/Functions/Internal%20Functions/ArCOM/ArCOMObject_Bpod.m) object to interact with hardware over USB. In effect, modules are communicated with using the serial byte code approach, with Bpod/Functions/ providing a wrapper (a user friendly interface) for this communication method.

UART vs USB

## State machine serial interface

- Firmware v17 [State Machine Serial Interface](https://sites.google.com/site/bpoddocumentation/user-guide/serial-interfaces/statemachineserialinterface?authuser=0)
- Firmware v18-22 [State Machine Serial Interface](https://sites.google.com/site/bpoddocumentation/user-guide/serial-interfaces/fsm_interface_v18?authuser=0)


## Module serial interfaces
- Explain `ArCOM`/Module Class vs State Machine interface