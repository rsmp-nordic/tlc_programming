# Phase-based Strategy
With the phase-based control strategy, the program consists of phases run in a specific order, separated by interphases.

## Prerequisites
Running a phase-based program depends on regional, controller, and intersection [configurations](configurations.md), which are defined outside the signal program.

Specifically, signal groups and the conflict matrix are defined in the intersection configuration, not in the signal program.

Similarly, regional settings like red-yellow time are defined in the regional/controller/intersection configuration, not in the signal program.

Controllers must use UTC [synchronized](synchronization.md) using NTP.

## Structure
A phase-based program has the following structure:

```yaml
cycle: 60
offset: 0
groups: ["a1", "a2", "b1", "b2"]
phases:
  main: 
    groups: ["a1", "a2"]
    duration: 20
    max: 30
  side:
    groups: ["b1", "b2"]
    duration: 20
    min: 10
    max: 25
  turn:
    open: ["b2"]
    duration: 10
order: [:main, :turn, :side]
switch: :main
```

- **cycle**: Cycle time
- **offset**: Default offset
- **groups**: List of all signal groups
- **phases**: Map of phases with phase name and options:
  - `groups`: List of signal groups that are open (green).
  - `duration`: Default duration in seconds.
  - `min`/`max`: Optional minimum/maximum durations when shortening or extending the phase.
- **order**:  sequence to run phases in.
- **switch**: Where a program switch is allowed. Switch happens at the beginning of the phase.

The above program can be visualized like this:

```
phase  |main      |..|turn |..|side     |..|
a1     |oooooooooo|  |     |  |         |  |
a2     |oooooooooo|  |     |  |         |  |
b1     |          |  |     |  |ooooooooo|  |
b2     |          |  |ooooo|  |ooooooooo|  |
switch |*         |  |     |  |         |  |
       0s                                  60s
```

Interphase are indicated with ".." and does not show group states because they are determined by the controller at run time.

## Phases
Each phase is defined by which groups are open and for how long. All other groups will be closed.

Open  typically means green, while closed typically means red. But depending on the type of group it could be e.g. a white horizontal or vertical bare for public transport.

All groups remain in the same state throughout the phase. All state changes happend during interphases.

You can define how much the phase can be shortened or extended, which will be used by the controller to adjust phase durations to ensure that the cycle time is respected.
The controller will also extend and shorten phases when the offset needs to be moved.

## Interphases
Interphases are not explicitly defined. The controller determines how to switch between phases at run time using intermediate states.
This will be based on configurations of intergreen times, geometry, safety times etc.

## Changing Offset
A changing in offset can happen for several reasons, e.g.:

- manual change of the offset
- change between signal programs
- synchronization of the underlying UTC time
- leap seconds

However, the offset cannot simply be abruptly changed, as this might cause invalid state changes, or might violate constrainst like minimum or inter-green times.

Instead the desired offset must be reached as quickly as possible while respecting all constraints.

### Extending and Shortening Phases
If the offset needs to move backward the controller can extend a phase up to the `max` set for the phase. If max is not set, the phase cannot be extended.
Conversely, if the offset needs to move forward the controller can shorten a phase down to the `min` set of the phase. If min is not set, the phase cannot be shorted.

Reaching the target offset will be typically be quicker if both min and max values are defines. If no minimum or maximum, values are defined, the controller canonot adjust the offset, and the program cannot be coordinated with other controllers.

Reaching the target offset might take more than one cycle, depending on the min/mx values defined.

Once the target offset is reached, the controller must adjust phase durations to ensure that the cycle time matches the value defined in the program.


### Proportial Offset Adjustment
The controller must used the default, min and max durations defines for each phase to distribute an adjustment proportially across phases.

Possible extension is computed as: max - default
Possible shortening is cmpute as: default - min.

In the program above, maximum shortenings and extensions are:

|Phase|Shorten|Extend|
|--|--|--|
|main||10s (66%)|
|side|10s (100%)|5s (33%)|
|turn||

The proportions are shown a percentages.

If the offset must be moved back by 9s the `main` phase will be extend by 6s, while `side` will be extended by 3s. 
After a single cycle, the offset target will be reached, and the phase will go back to durations that ensure that the cycle time is 60s (including interphases).

If the offset must be moved back to 21s  the `main` phase needs to be extend by 14s, while `side` needs to be extended by 7s. But the max extensions must be respected, so in the first cycle,
 `main` will be extended by 10s, and `side` by 5s, which will move the offset back by 15s, leaving if 6s from the target. In the second cycle, the `main` phase will be extend by 4s and while `side` by 2s. 

## Switching Programs
A program switch can occur only at the start of the phase defined in `switch`. Both the current and target programs must have compatible signal states at the switch point to ensure a safe transition.

It's allowed to switch between programs using different control strategies.

When a switch is requested the controller continues until the switch point, then transitions to the switch point in the target program.
A new target offset is determined, and the offset is grdually moved to the target, using the mechanism defined for tthe type of strategy of the targert program.

## Failures
The controller must respect all safety constraints (minimum green, intergreen, etc.). Invalid configurations or unsafe transitions results in a fault.
Programs should be validated using a simulator or test tool before deployment.

