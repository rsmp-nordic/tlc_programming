# Startup and Shutdown Programs
A startup program is used to go from standby or fault to running, while a shutdown program is used to go from running to standby.

The controller must have one startup program and one shutdown program.

Startup and shutdown program should not be listed as normal programs and should not be targetable for normal program switching. Instead, they are targeted automatically when the controller starts up or shuts down.

The **initial state** after a power-on is always dark. The **standby state** is defined in the controller configuration, not as part of signal programs. It can be dark or some other state like yellow flash, since the power is still on.

The states to run through is defined using a [state sequence](sequence.md).

## Startup
Structure:
```yaml
groups: ["a1","a2","b1","b2"]
states: [
  "0000", 4,
  "1100", 4,
  "AA00", 2
```

This can be visualized as:

```
a1     |00001111AA| # The main 'a' direction goes through red-yellow-green
a2     |00001111AA|
b1     |0000000000| # The side direction 'b' stays red
b2     |0000000000|
       0s         10s
```

When powering up the controller:
- starts in the initial state, which is always dark, because the power was off
- goes to standby state
- sets a normal program as the target
- runs the startup program from the start
- switches to the set target program when it reaches the end

When the controller leaves fault mode it runs the same sequence, but starts from the standby state.

A startup program always runs from the start to the end and never cycles.

If the controller runs in coordinated mode, it can choose to stay a while in standby state before running the startup program. Or it can immediately run the startup program, switch to another program and then use the skip/wait mechanism to coordinate. Which method is used is controlled by configurations in the controller.

## Shutdown
Structure:
```yaml
groups: ["a1","a2","b1","b2"]
states:
  "AA00", 2,
  "1100", 4,
  "0000", 4,
```

This can be visualized as:
```
a1     |AA11110000| # The main 'a' direction goes through green-yellow-red
a2     |AA11110000|
b1     |0000000000| # The side direction 'b' stays red
b2     |0000000000|
       0s         10s
```

When shutting down the controller:
- sets the shutdown program as the target
- switches when it reaches a switch point in the current program
- runs the shutdown program from the start
- when it reaches the end it goes to standby state and halts, i.e. stops the cycle counter

A shutdown program always runs from the start to the end and never cycles.

## Switch points
A startup program can only switch at the end of the program, while a shutdown program can only switch at the start of the program.
