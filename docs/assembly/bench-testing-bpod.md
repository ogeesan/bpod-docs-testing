# Bench-testing Bpod
This Bench-testing procedure will verify connectivity, but will not validate reliability or detect problems with Arduino. An improved bench testing procedure will be added soon. 

## Setup

1. Connect a BNC to bare-wire breakout (or a probe set to 1X) to an oscilloscope, and view the channel with DC coupling, 1ms per div, 5V per div. Set the trigger to "Normal", and the threshold to ~2.5V. Make sure all filters and persistent display features are off.
2. Connect a Bpod Arduino shield to an Arduino Leonardo and upload the "Blink-server" sketch in /Bpod/Firmware/SerialBlinkServer. Then unplug the Leonardo and put it aside.
3. Connect a Bpod mouse port and solenoid valve to a port interface board. All three should be known-good.
4. Connect the interface board + valve + port to Bpod with a known-good Ethernet cable.
5. Run `Bpod`; at the MATLAB command prompt
6. From the Settings menu, select the "Port" icon, and select the port that is plugged in. All other channels should be unchecked.

## Ports

1. From the Bpod console, click the "H2O" button ("VLV" on Bpod 0.7+), corresponding to the connected port. The valve should open each time the button is pressed, and close when depressed. (with your mouse, click to press and click again to depress).
2. From the Bpod console, click the "LED" button. The port light should turn on with each press, and off with each depress.
3. Run the "operant" protocol from the console, using "dummy subject" and "default settings". Break the port photogate with an opaque object, and observe the "poke" icon on the console. It should turn green when the object is inside the port, and red when it is outside. The "Last Event" should display "Port1In" when the object enters, and "Port1Out" when it leaves.  There should be no spurious pokes on any of the ports.
4. Repeat this process for the next 7 ports. Each time, update Settings > Port to make sure only the connected poke is checked. If you have more than 1 port, proceed in parallel.

## Sync port (Bpod 0.5 only)

1. Connect the oscilloscope BNC wire ground to the right-most spring terminal ground. 
2. Run /Bpod/Functions/Example Matrices/Cycle_128_States.m. This will generate a state matrix with 128 states.
3. Upload the state matrix: SendStateMatrix(sma);
4. Run the state matrix: `RunStateMatrix`;
5. Each time you run the state matrix, the system passes through 128 states for 1ms each.
6. Touch the signal pin of the BNC probe to each pin on the RJ45 connector, accessed from the top. The left-most pin is ground, and should have no signal when you run the matrix. The next pin is bit 1, which should go high every 2ms. The next is bit 2; 4ms... and so forth until the right most pin, bit 7. If all pins have the correct signal reliably, the sync port is working.

## Hardware serial ports (for Bpod 0.5)
1. Connect Arduino Leonardo to power, and connect an Ethernet cable between the Bpod Arduino shield and Serial port 1. Reboot Leonardo with the "reset" button.
2. Run /Bpod/Functions/Example Matrices/TriggerSerialDevice.m
3. Run SendStateMatrix(sma);
4. When you run the state matrix, you should see the LED on Arduino Leonardo flash twice, indicating that byte "2" was received. 
5. Plug Leonardo into Serial port 2. When you run the state matrix, it should flash 4 times, indicating that byte "4" was received.

## Hardware serial ports (for Bpod 0.7+)

1. Connect Arduino M0 to power, and connect an Ethernet cable between the Bpod Arduino shield and Serial port 1. Reboot M0 with the "reset" button.
2. Press the "Refresh" button on the console GUI. A tab should appear, labeled "EchoModule1".
3. Open the EchoModule1 tab, and type a message into the terminal. The echo module should receive it, and echo it back. 
4. Verify that the echo module is visible on all 3 ports.

## BNC and wire terminal outputs

1. Connect the BNC outputs to the oscilloscope, and toggle them with the BNC Override > Out buttons on the Bpod console. You should see 5V when high, and 0V when low.
2. Connect each wire terminal output (spring terminals 2-5 from the left) to the oscilloscope and override them from the console. You should see 3.3V when high and 0V when low.

## BNC and wire terminal inputs
1. Connect a TTL source (we use Pulse Pal) to Bpod's BNC input channel 1. 
2. Run the Operant protocol. 
3. You should see the BNC 1 icon turn green each time the BNC line is high, and "Last Event" should display "BNC1High".  
4. Repeat for BNC input 2.
5. To test the wire input channels (also 5V tolerant), select each one from Console > Settings > Port, and un-check all others. Then repeat the BNC input channel test.