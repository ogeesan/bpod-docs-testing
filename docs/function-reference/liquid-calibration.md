# Liquid calibration
### `GetValveTimes()`
**Description**

Converts liquid amounts (in microliters) to time a solenoid valve should be open to deliver the desired amount (in seconds).

- Uses the calibration functions generated with the [Liquid Calibrator](../user-guide/bpod-gui.md#liquid-calibration).

**Syntax**
```matlab
ValveTimes = GetValveTimes(LiquidAmount, TargetValves)
```

**Parameters**

- LiquidAmount: amount of liquid to deliver (in microliters).
- TargetValves: a vector of integers listing the valves to return.

**Returns**

- ValveTimes : A vector containing the valve times for all valves listed in the TargetValves parameter (in seconds)

**Example**

This code gets the time valves 1 and 3 must be open to deliver 20ul of liquid. 
```matlab
ValveTimes = GetValveTimes(20, [1 3]); 
LeftValveTime = ValveTimes(1); 
RightValveTime = ValveTimes(2);
```
