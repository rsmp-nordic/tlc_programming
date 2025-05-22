# Startup and shutdown programs
A startup program is used to go from standby or fault to normal operation.
A shutdown program is used to go from normal mode to standby.

Startup and shutdown programs are similar to normal [fixed-time](fixed_time.md) programs, but have no offset, and no skip points or wait points:

The controller must have one startup program and one shutdown program.

Startup and shutdown programs should not be listed as normal programs and shoudl not targetable for normal program switching. Instead they are targeted automatically when the controller starts up or shuts down.

The **initial state** after a power on is always dark. The **standby state** is defined in the controller configuration, not as part of signal programs. It can be dark or some other state like yellow flash, since the power is still on.

## Startup
Example of a startup program:
```yaml
length: 10
groups: ["a1","a2","b1","b2"]
states:
  0: "0000"
  4: "1100"
  8: "AA00"
switch: 9
```

The startup program above can be visualized as:

```
a1     |00001111AA| # The main 'a' direction goes through red-yellow-green
a2     |00001111AA|
b1     |0000000000| # The side direction 'b' stays red
b2     |0000000000|
switch |         S|
       0s         10s
```

When powering up the controller:
- starts in the initial state, which is always dark, because the power is off
- goes to standby state
- sets a target program
- runs the startup program from the start
- switches program when it reaches the switch point

When the controller leaves fault mode it runs the same sequence, but starts from the standby state.

A startup program always runs from the start and never cycles. If it ever tries to go beyond the end it's a fault.

If the controller runs in coordinated mode, it can choose to stay a while in standby state before running the startup program. Or it can immediately run the startup program, switch to another program and then use the skip/wait mechanism to coordinate. Which method is used is controlled by configurations in the controller.

## Shutdown
Example of a shutdown program:
```yaml
length: 10
groups: ["a1","a2","b1","b2"]
states:
  0: "AA00"
  2: "1100"
  6: "0000"
switch: 0
```

The shutdown program above can be visualized as:
```
a1     |AA11110000| # The main 'a' direction goes through green-yellow-red
a2     |AA11110000|
b1     |0000000000| # The side direction 'b' stays red
b2     |0000000000|
switch |S         |
       0s         10s
```

When shutting down the controller:
- sets the shutdown program as the target
- switches when it reaches the switch point in the current program
- runs the shutdown program from the switch point
- when it reaches the end it goes to standby state and halts

When the end is reached, the controller goes to standby state and halts. A shutdown program never cycles.

## Switch Points
In a startup program the switch point is usually placed at the end, since there is no way to reach the part after the switch point.

In a shutdown program the switch point is usually placed at the start, since there is no way to reach the part before the switch point.
