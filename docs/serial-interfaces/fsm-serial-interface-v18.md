# State Machine Serial Interface

Firmware version **18-22**

## Description

Allows software (i.e. Matlab, Python) to communicate with a Bpod state machine via its USB serial port.

This document describes the format of byte strings to send to the state machine's USB serial port, and what bytes to expect in return.

## Discovery byte

While not yet connected to software, the Bpod state machine sends a "discovery byte" to any software that connects to its serial port. This allows PC-side software to identify the correct port, without sending probe bytes that could be handled in unexpected ways by other instruments.

The discovery byte identifying a Bpod finite state machine is: **222**, and may be read by simply opening the port and waiting 150ms for the byte to arrive.

## Command Menu

The first byte sent to the Bpod state machine accesses a command menu, where different bytes specify different functions.

The command bytes are:

- '**6**' (ASCII 54): **Hand shake** (used to confirm a valid connection to the state machine).
    - Sending this byte disables discovery byte transmission until it is explicitly restored with the 'Z' command (see below).
    - The state machine returns a byte: '5' (ASCII 53) to confirm the connection.
    - The state machine also resets the session clock (see '\*' command below)
    - Note: You may find an extra discovery byte in the buffer after sending the hand shake. Be sure to clear it before sending the next command.
- '**F**' (ASCII 70): **Return firmware version and machine type**
    - The state machine replies with the following bytes:
    - FirmwareVersion(2 bytes; 16-bit int)
    - MachineType (2 bytes; 16-bit int). _MachineType is: 1 (State Machine 0.5) 2 (State Machine 0.7-0.9) or 3 (State Machine 2)_
- '**\***' (ASCII 42): **Reset session clock**
    - This sets the session clock to 0.
    - Trial start and end times will be provided in reference to the last clock reset.
    - The state machine returns a byte (1) to confirm that it has finished resetting the session clock.
- '**G**' (ASCII 71): **Return timestamp transmission scheme**
    - The state machine replies with one byte, TimestampTransmissionScheme:
        - 0: Post-trial (event timestamps are stored in Bpod's RAM and sent at once when each trial ends)
        - 1: Live (during a trial, each cycle's list of event codes sent to the PC is followed by the cycle's timestamp).
    - Note: In either case, timestamps are returned in units of state machine cycles (default = 100 microseconds / cycle).
- '**H**' (ASCII 72): **Return the state machine's on-board hardware configuration (excluding modules)**.
    - The state machine replies with the following bytes:
        - MaxStates(2 bytes; 16-bit int); _maximum number of supported states in a single state machine description_
        - TimerPeriod(2 bytes; 16-bit int); _the period (in microseconds) of the state machine's refresh cycle during a trial_
        - maxSerialEvents(1 byte); _the maximum number of behavior events that can be allocated among connected modules_
        - nGlobalTimers (1 byte); _the number of global timers supported_
        - nGlobalCounters (1 byte); _the number of global counters supported_
        - nConditions (1 byte); _the number of condition-events supported_
        - nInputs (1 byte); _the number of channels in the state machine's input channel description array_
        - inputDescriptionArray (1 byte x nInputs); _an array indicating the state machine's onboard input channel types_
        - nOutputs (1 byte); _the number of channels in the state machine's output channel description array_
        - outputDescriptionArray (1 byte x nOutputs); _a byte array indicating the state machine's onboard output channel types_
- '**M**' (ASCII 77): **Return information describing the state machine's connected modules**
    - The state machine replies with the following bytes:
        - for each module (character 'U') in outputDescriptionArray (see 'H' above)
            - moduleConnected (1 byte); 1 if a module was found, 0 if not.
            - if moduleConnected == 1
                - moduleFirmwareVersion (4 bytes; 32-bit int) - _firmware version reported by the module_
                - moduleNameLength (1 byte); _length of module name, in characters_
                - moduleName (1 x moduleNameLength bytes); _a character array with the module name_
                - moreInfoFollows (1 byte); _0 if module description is complete, 1 if more data follows_
                - while moreInfoFollows == 1
                    - infoType (1 byte); _Type of info returned_
                    - if infoType == '**#**' (ASCII 35); _code to request for a specific number of serial events_
                        - nEvents(1 byte); _number of serial events requested_
                    - elseif infoType == '**E**' (ASCII 69) - _code to_ _assign names to event bytes returned from this module to the state machine_
                        - nEventNames (1 byte); number of event names to transmit
                        - for 1 to nEventNames
                            - eventNameLength (1 byte); length of this event name
                            - eventName (1 x eventNameLength bytes); a character array with the event name
                    - end
                - end
                    - moreInfoFollows (1 byte); 0 if module description is complete, 1 if more data follows
            - end
        - end
    - end
- '**%**' (ASCII 37): **Set number of behavior events allocated to each module**. '#' (byte 0) is followed by:
    - Bytes 1-nModules: a byte for each module, indicating the number of behavior events it can generate.
        - _NOTE: nModules is the sum of (character 'U') in outputDescriptionArray returned by the state machine (see 'H' above)_
    - The state machine returns a byte (1) to confirm that it has finished setting each module's event allocation.
    - _NOTE: By default, the maximum number of events (see 'H' -> maxSerialEvents above) is distributed equally among modules. If a module requests a specific number of events (see 'M' -> '#' above), the software (MATLAB / Python) must calculate the reallocation using the set of requests, and then use this command ('%') to update the state machine._
    - '**E**' (ASCII 69): **Set the state of each input channel (enabled/disabled)**. 'E' (byte 0) is followed by:
    - Bytes 1-nInputs: a byte for each input channel, indicating whether it is enabled (1) or disabled (0).
        - _nInputs must exist, having been returned from the 'H' command (above)_
        - Generally, low-impedance inputs that are not connected to a signal source or tied to a pull-down resistor should be disabled.
            - On the Bpod state machine, only the port input channels (i.e. photogates) are low impedance inputs.
    - The state machine returns a byte (1) to confirm that it has finished setting each channel's enabled property.
    - '**J**' (ASCII 74): **Enable/Disable relay of incoming bytes from one module to the USB port**. 'J' (byte 0) is followed by:
    - Byte 1: the module number to enable or disable _(indexed by 0)_
    - Byte 2: the state of the module (0 = relay off, 1 = relay on)
- '**K**' (ASCII 87): **Set a state synchronization channel**. 'K' (byte 0) is followed by:
    - Byte 1: the digital output channel to use for synchronization. _This channel is an index of outputDescriptionArray (see 'H' above)._
    - Byte 2: the synchronization mode. Valid modes are:
        - 0: Channel set high on trial start and low on trial end
        - 1: Channel switches logic states with each state transition
    - The state machine returns a byte (1) to confirm that it has finished setting the sync channel configuration.
- '**O**' (ASCII 79): **Override digital output line state**. 'O' (byte 0) is followed by:
    - Byte 1: the digital output channel to override. _This channel is an index of outputDescriptionArray (see 'H' above)._
    - Byte 2: the new state of the channel
        - If outputDescriptionArray\[index\] is a digital line (D,B,W), Byte 2 should be (1 = high, 0 = low)
        - If outputDescriptionArray\[index\] is a PWM line (P), Byte 2 should be the new PWM duty cycle (0-255)
        - If outputDescriptionArray\[index\] is a valve bank (S), Byte 2 should be a byte whose bits set the state of the 8 valves.
- '**I**' (ASCII 73): **Read the state of a digital input channel**. 'I' (byte 0) is followed by:
    - Byte 1: the digital input channel to read. _This channel is an index of inputDescriptionArray (see 'H' above)._
        - The state machine will return:
            - 0 if the channel's logic level is low
            - 1 if the channel's logic level is high.
- '**T**' (ASCII 84): **Transmit a string of bytes to a connected module**. 'T' (byte 0) is followed by:
    - Byte 1: the index of the targeted module.
        - The module index is in range 0 -> max number of modules (instances of 'U' in outputDescriptionArray; see 'H' above)
        - Byte 2: nBytes _(number of bytes in the message)_
        - Bytes 3 --> (2+nBytes): _The message to transmit_
- '**L**' (ASCII 76): **Store a list of 1-3 byte serial messages, which can later be sent to modules by message index**. 'L' (byte 0) is followed by:
    - Byte 1: the index of the targeted module. _(Each module has its own message library)_
    - _The module index is in range 0 -> max number of modules (instances of 'U' in outputDescriptionArray; see 'H' above)_
    - Byte 2: nMessages _(the number of messages to store)_
        - for each message between 1 and nMessages
            - MessageIndex (1 byte; 1-255)
            - MessageLength (1 byte; 1-3)
            - for 1 to MessageLength
                - 1 Byte (The next byte of the current message)
    - The state machine returns a byte (1) to confirm that the specified channel's message library has been updated.
- '**\>**' (ASCII 62): **Clear the serial message libraries**. No data follows.
        - The state machine restores each message to default - a message of length 1, whose value is equal to its index.
    - The state machine returns a byte (1) to confirm that the message libraries have been cleared.
- '**U**' (ASCII 85): **Transmit a stored serial message (by index) to a connected module**. 'U' (byte 0) is followed by:
    - Byte 1: the index of the targeted module.
    - The module index is in range 0 -> max number of modules (instances of 'U' in outputDescriptionArray; see 'H' above)
    - Byte 2: the index of the message to send.
- '**S**' (ASCII 83): **Echo a USB soft code targeting the PC**. 'S' (byte 0) is followed by:
    - Byte1: (The soft code to echo)
        - The state machine replies with the following bytes:
            - 2 (the op-code for a soft code; see command 'R' below)
            - Byte1
            - This function is useful for debugging and testing soft code handlers.
        **'~'** (ASCII 126): **Receive a USB soft code targeting the state machine.** '~' (byte 0) is followed by:
        - Byte1: (The soft code)
        - This will only have an effect if the state machine is running, and in a state that handles the specific soft code sent.
- '**V**' (ASCII 86): **Manually override an input channel, creating a virtual event**. 'V' (byte 0) is followed by:
    - Byte 1: the input channel to override. This channel is an index of inputDescriptionArray (see 'H' above).
    - Byte 2: the new value of the channel (0 = low, 1 = high).
    - NOTE: If a channel is overridden, it will remain in the overridden state regardless of what signals arrive. The channel must be reset with a second call to 'V' in order to return control to the hardware.
- '**C**' (ASCII 67): **Transmit a compressed state machine description to the state machine device**. 'C' (byte 0) is followed by:
    - Byte 1: RunStateMatrixASAP _(if 1, triggers the new SM to auto-run after the current one completes, without a call to 'R')_
        - Byte 2: using255BackSignal _(if 1, only 254 actual states are allowed and "going to state 255" jumps back to previous state)_
        - Bytes 3-4: nBytes _(the number of bytes in the remaining state machine description, NOT including the first 4 bytes and 'C')_
        - Byte 5: nStates _(the number of states in the state machine description)_
        - Byte 6: nGlobalTimersUsed _(The 1-based index of the highest global timer used in the state matrix; 0 if none)_
        - Byte 7: nGlobalCountersUsed _(The 1-based index of the highest global counter used in the state matrix; 0 if none)_
        - Byte 8: nConditionsUsed _(The 1-based index of the highest condition used in the state matrix; 0 if none)_
        - for each state s between 1 and nStates
            - TimerMatrix\[s\] (1 byte) - _The state to go to if the state timer elapses_
        - for each state s between 1 and nStates
            - nOverrides (1 byte) - The number of **events** handled in this state (overriding default of not-handled)
            - if nOverrides > 0
                - for each event e between 1 and nOverrides
                    - thisEvent (1 byte) - _(the numeric code of the event handled in state s)_
                    - thisState (1 byte) - _(the state to go to if thisEvent occurs in state s)_
        - for each state s between 1 and nStates
            - nOverrides (1 byte): _The number of_ _**outputs\***_ _controlled in this state (overriding default of no output)_
            - _\*note: As of firmware v18, global timer trigger and cancel output actions are sent separately (below)_
            - if nOverrides > 0
                - for each output channel between 1 and nOverrides
                    - thisOutputChannel (1 byte) - _(the index of the target output channel outputDescriptionArray)_
                    - thisOutputValue (1 byte) - _(the value of the output channel)_
        - for each state between 1 and nStates
            - nOverrides (1 byte): _The number of_ _**global timer start events**_ _handled in this state (overriding default of none)_
            - if nOverrides > 0
                - for each timer t between 1 and nOverrides
                    - thisTimer (1 byte) - _(the index of the global timer whose start-events are handled)_
                - thisState (1 byte) - _(the state to go to if thisTimer's start event occurs)_
        - for each state s between 1 and nStates
            - nOverrides (1 byte): _The number of_ _**global timer end events**_ _handled in this state (overriding default of none)_
            - if nOverrides > 0
                - for each timer t between 1 and nOverrides
                    - thisTimer (1 byte) - _(the index of the global timer whose end-events are handled)_
                - thisState (1 byte) - _(the state to go to if thisTimer's end event occurs)_
        - for each state s between 1 and nStates
            - nOverrides (1 byte): _The number of_ _**global counter threshold events**_ _handled in this state (overriding default of none)_
            - if nOverrides > 0
                - for each counter c between 1 and nOverrides
                    - thisCounter (1 byte) - _(the index of the global counter whose threshold events are handled)_
                - thisState (1 byte) - _(the state to go to if thisCounter's threshold event occurs)_
        - for each state s between 1 and nStates
            - nOverrides (1 byte): _The number of_ _**condition events**_ _handled in this state (overriding default of none)_
            - if nOverrides > 0
                - for each condition c between 1 and nOverrides
                    - thisCondition (1 byte) - _(the index of the condition whose events are handled)_
                - thisState (1 byte) - _(the state to go to if a thisCondition event occurs)_
        - for each global timer between 1 and nGlobalTimersUsed _(nGlobalTimersUsed was received earlier in the transmission)_
            - linkedOutputChannel (1-byte) - _index of an output channel in outputDescriptionArray. 255 = no channel linked)._
        - for each global timer between 1 and nGlobalTimersUsed _(nGlobalTimersUsed was received earlier in the transmission)_
            - onMessage (1-byte). _This is the index of a message set previously with command 'L' above, to send when the timer starts._
        - _Note: The target module for onMessage is defined by linkedOutputChannel (above), if linkedOutputChannel is a module port._
        - for each global timer between 1 and nGlobalTimersUsed _(nGlobalTimersUsed was received earlier in the transmission)_
            - offMessage (1-byte). _This is the index of a message set previously with command 'L' above, to send when the timer starts._
        - _Note: The target module for offMessage is defined by linkedOutputChannel (above), if linkedOutputChannel is a module port._
        - for each global timer between 1 and nGlobalTimersUsed _(nGlobalTimersUsed was received earlier in the transmission)_
            - loopMode (1-byte). _Set to 0 (default) if a one-shot timer, 1 to loop the timer until canceled, 2-255 to loop N times_
        - for each global timer between 1 and nGlobalTimersUsed _(nGlobalTimersUsed was received earlier in the transmission)_
            - sendGlobalTimerEvents (1-byte). _Set to 1 (default) to generate global timer on and off events. Set to 0 to disable them._
        - for each global counter between 1 and nGlobalCountersUsed _(nGlobalCountersUsed was received earlier in the transmission)_
            - attachedEvent(1-byte). _This is the index of a behavior event to count._
        - for each condition between 1 and nConditionsUsed _(nConditionsUsed was received earlier in the transmission)_
            - conditionChannel(1-byte). _This is an input channel index in inputDescriptionArray._
        - for each condition between 1 and nConditionsUsed _(nConditionsUsed was received earlier in the transmission)_
            - conditionValue(1-byte). 0 = low, 1 = high. _Defines the state of the channel when the condition is true._
    - for each state between 1 and nStates
        - globalCounterReset(1-byte). _Defines the index of the global counter to reset in each state._
    - _NOTE: The next section of the state machine description expects a different datatype for several variables, depending on how many global timers are supported by the state machine: byte (if < 9), 16-bit unsigned int (if < 17), 32-bit unsigned int (otherwise, up to 32). This compression allows Bpod firmware to support state machine versions with less internal memory._
    * if nGlobalTimers < 9 Note: _nGlobalTimers was received with the state machine hardware description; (see 'H' above)._
        - for each state between 1 and nStates
            - globalTimerTrig (1-byte). _Bits of globalTimerTrig indicate the global timers to trigger in each state_
        - for each state between 1 and nStates
            - globalTimerCancel (1-byte). _Bits of globalTimerTrig indicate the global timers to cancel in each state_
            - for each global timer between 1 and nGlobalTimersUsed _(nGlobalTimersUsed was received earlier in the transmission)_
                - globalTimerOnsetTriggers (1-byte) _Bits of globalTimerOnsetTriggers indicate other global timers to trigger after the current timer's onset delay._
    - else if nGlobalTimers < 17
        - for each state between 1 and nStates
            - globalTimerTrig (2-byte; 16-bit unsigned int). _Bits of globalTimerTrig indicate the global timers to trigger in each state_
        - for each state between 1 and nStates
            - globalTimerCancel (2-byte; 16-bit unsigned int). _Bits of globalTimerTrig indicate the global timers to cancel in each state_
            - for each global timer between 1 and nGlobalTimersUsed _(nGlobalTimersUsed was received earlier in the transmission)_
                - globalTimerOnsetTriggers (2-byte; 16-bit unsigned int) _Bits of globalTimerOnsetTriggers indicate other global timers to trigger after the current timer's onset delay._
    - else (up to 32 global timers theoretically possible)
        - for each state between 1 and nStates
            - globalTimerTrig (2-byte; 32-bit unsigned int). _Bits of globalTimerTrig indicate the global timers to trigger in each state_
        - for each state between 1 and nStates
            - globalTimerCancel (2-byte; 32-bit unsigned int). _Bits of globalTimerTrig indicate the global timers to cancel in each state_
            - for each global timer between 1 and nGlobalTimersUsed _(nGlobalTimersUsed was received earlier in the transmission)_
                - globalTimerOnsetTriggers (2-byte; 32-bit unsigned int) _Bits of globalTimerOnsetTriggers indicate other global timers to trigger after the current timer's onset delay._
    * end if

        - for each state s between 1 and nStates
            - stateTimer\[s\] (4 bytes; 32-bit int). _This state's internal timer (units = state machine cycles, 1 cycle = timerPeriod returned from 'H' above)._
        - for each global timer t between 1 and nGlobalTimersUsed _(nGlobalTimersUsed was received earlier in the transmission)_
            - globalTimer\[t\] (4 bytes; 32-bit int). _Duration of global timer t (units = state machine cycles)._
        - for each global timer t between 1 and nGlobalTimersUsed _(nGlobalTimersUsed was received earlier in the transmission)_
            - globalTimerOnsetDelay\[t\] (4 bytes; 32-bit int). _Onset delay of global timer t (units = state machine cycles)._
        - for each global timer t between 1 and nGlobalTimersUsed _(nGlobalTimersUsed was received earlier in the transmission)_
            - globalTimerLoopInterval\[t\] (4 bytes; 32-bit int). _Delay between timer loop iterations (units = state machine cycles)._
    - for each global counter between 1 and nGlobalCountersUsed _(nGlobalCountersUsed was received earlier in the transmission)_
            - globalCounterThreshold\[t\] (4 bytes; 32-bit int). _Threshold (units = instances of the event being counted)._
    - NOTE: The state machine will return a confirmation byte to indicate successful transmission of the state machine description, on the next call to RunStateMachine. This saves 1 read/write cycle, and reduces dead-time.
- **'R'** (ASCII 82): **Run state machine.**
    - if a new state machine was sent prior to calling 'R'
    - The state machine returns a byte (1) to confirm that it has finished receiving the new state machine.
    - The state machine returns TrialStartTimestamp (8 bytes; 64-bit unsigned int). _Time in microseconds since session start_
    - While the trial is running, event codes and soft-codes are returned via the USB serial port as follows:
        - Op-code (byte 1) = 1 if the message is an event, and 2 if the message is a soft code
        - If op-code 1,
            - nEvents (byte 2) - _number of events returned_
        - for each event between 1 and nEvents
            - Event code (1 byte)
            - if TimestampTransmissionScheme == 1 _See command 'G' above; 1 = during trial_
                - Timestamp (4 bytes; 32-bit int) _Units = state machine cycles (default=100us)_
        - If op-code 2,
            - Soft code (byte 2) = the soft code to pass to the [soft code handler function](../function-reference/bpodsystem-fields.md#softcodehandlerfunction).
        **Exit state**: If event code 255 was returned, the state machine has reached an exit state. It then sends the trial's timing data:
        - nCyclesCompleted (4 bytes; 32-bit integer) _\- the number of state machine cycles counted during the trial_
        - TrialEndTimestamp in microseconds (8 bytes; 64-bit unsigned integer) _- time the trial ended in microseconds_
        - if TimestampTransmissionScheme == 0 _See command 'G' above; 0 = after trial_
            - nTimestamps (2 bytes; 16-bit integer) - _the number of timestamps to be transmitted_
            - for each timestamp between 1 and nTimestamps:
                - Timestamp (4 bytes; 32-bit integer) - _for each event recorded during the trial, a 32-bit timestamp. Units = 100us state machine cycles following trial start._
- '**X**' (ASCII 88): **Force-exit the currently running state machine, and return the partial trial's data**.
    - No data follows.
    - The state machine then returns the data in same format as for 'R' command above.
    **'Z'** (ASCII 90): **Disconnect state machine from the USB serial port**.
    - No data follows.
    - This resets several program variables to prepare the state machine for its next connection