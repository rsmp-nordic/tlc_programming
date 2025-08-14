# Stage-based Strategy
A stage-based program consists of stages separated by transitions. All state changes occur during transitions, while state remain static during stages.

## Prerequisites
Running a stage-based program depends on regional, controller, and intersection [configurations](configurations.md), which are defined outside the signal program.

Specifically, signal groups and the conflict matrix are defined in the intersection configuration, not in the signal program.

Similarly, regional settings like red-yellow time are defined in the regional/controller/intersection configuration, not in the signal program.

Controllers must use UTC [synchronized](synchronization.md) using NTP.

## Structure
Stage-based programs first define stages and transitions, which are shared between programs:

```yaml
groups: ["a1", "a2", "b1", "b2", "a1_l"]
stages:
  main:
    open: ["a1", "a2"]
    duration: { default: 20, max: 29 }
  side:
    open: ["b1", "b2"]
    duration: { min: 10, default: 20, max: 26 }
  turn:
    open: ["a1_l"]
    duration: { default: 10 }
  oneway:
    open: ["a1"]
transitions:
  main:
    side:
      default:  ["11000", 3, "00220", 2]
      quick:    ["11000", 5, "00220", 4]
    turn:       ["11002", 3]
    oneway:     ["11000", 3]
  side:
    turn:
      default:  ["11000", 5]
      quick:    ["11000", 3]
  turn:
    main:       ["11000", 3]
  oneway:
    main:       ["00110", 4]
```

A program defines the stages and possible transitions, as well as how you can enter and leave the program.

Here a program for quiet periods:

```yaml
name: quiet
flow:
  main:
    side:
    turn:
  side:
    turn:
  turn:
    main:
enter:
  main:
leave:
  side:
```

A program for busy periods:
```yaml
name: busy
flow:
  main:
    side: { using: quick }
  side:
    main: { using: quick }
enter:
  side:
leave:
  main:
    side: { using: quick }
```


A program for a special event:
```yaml
name: event
flow:
  oneway:
enter:
  main:
    oneway:
leave:
  oneway:
    main:
```

### Stages
Each stage is defined by which groups are open and for how long. All other groups must be closed.
Open typically means green, while closed typically means red. But depending on the type of group it could be e.g. a white horizontal or vertical bar for public transport.

All groups remain in the same state throughout the stage. All state changes happen during transitions.

You can define how much the stage can be shortened or extended by setting `min` and/or `max` durations.

A stage is defined as a map with:
  - `open`: List of signal groups that are open (typically green).
  - `duration`: Durations in seconds. `default` is required, while `min` and `max` are optional and specify possible shortening and extension.
  - `switch`: True if program switch can happen at the beginning of this stage.

### Transitions
A transition defines how to move from one stage to another by explicitly listing all state changes, including intermediate states like yellow.

A transition does not include the start and end states, as these are defined by the stages you come from and go to.

A particular transition always goes through the same state changes with the same duration.

If a transition between stages A and B is not defined, the program cannot transition directly from A to B, although it might be possible to reach B via other stages.

When designing a stage-based program, it must be ensured that all transitions are valid.


A transition is defined by a source/destination stage pair. The destination can be a transition array:

```yaml
  main:
    side: ["11000", 3, "00220", 2]
```

Or another map if you want to define different transitions for the same source/destination pair:

```yaml
  main:
    side:
      default:  ["11000", 3, "00220", 2]
      busy:     ["11000", 5, "00220", 4]
```

The transition array must have even number of elements. It starts with a state string followed by a duration, and then repeat this:

```yaml
 ["11000", 2, "00220", 4]
```

Here a1 and a2 groups are in state "1" for 5s, then b1 and b2 are in state "2" for 7 seconds.

### Programs
A program is defined by which stages and transitions be be used.
It's defined as a map of source/transtions, with transitions  listed using an array of strings, e.g:

```yaml
flow:
  main:
    side:
    turn:
  side:
    turn:
  turn:
    main:
```

Here the the controller can go from the main stage to either the side or the turn stage.
From side it can go only to side, and from turn it can go only to main.

In case different transitions are defined for the same source/destination stage pair,
the variant is specified with 'using':

```yaml
  main:
    side: { using: quick }
```

Note: No logic is yet defined for how to choose which transition to use. This will be expanded later.

## Switching Programs
A program switch can occur at a stage that's present in both the origin and destination programs.

Switch stages are specified as:
```yaml
enter:
  main:
leave:
  side:
```

If you need to switch to a program that does not share any stages, you can istead use a transition:

```yaml
enter:
  main:
    oneway:
leave:
  oneway:
    main:
```yaml

In this example, the transitions between main and oneway stages are not used in the normal flow of the program, but can be used to switch to a program that has uses the main stage.

You can specify a transition variant if needed:

```yaml
leave:
  main:
    side: { using: quick }
```


It's allowed to switch between programs using different control strategies. But whether you switch to a program with the same or different control strategy, the two programs must have compatible signal states at the switch point. This mean there must be a stage with matching states. This state does not have be be used in the normal flow of any stage-based program, but can be used just in enter/leave definitions.

When a switch is requested the controller continues until the switch point, then continues from the switch point in the target program.

Once the program has switched, a new target offset is determined and the offset is gradually moved to the target, using the mechanism defined for the type of strategy of the target program.

## Example
For the program shown at the top, going through the stages main-side-turn can be visualized as:

```
stage  |main              |      |side             |   |turn    |   |
a1     |AAAAAAAAAAAAAAAAAA|111   |                 |   |        |   |
a2     |AAAAAAAAAAAAAAAAAA|111   |                 |   |        |   |
b1     |                  |   222|AAAAAAAAAAAAAAAAA|111|        |   |
b2     |                  |   222|AAAAAAAAAAAAAAAAA|111|        |   |
a1_l   |                  |      |                 |   |AAAAAAAA|111|
switch |*                 |      |                 |   |        |   |
       0s                                                           60s
```

## Changing Offset
Intersections that use the same cycle length can be coordinated by modifying their offsets. But a change in offset can happen for other reasons, e.g.:

- manual change of the offset
- change between signal programs
- synchronization of the underlying UTC time
- leap seconds

However, the offset cannot be changed abruptly, as this might cause invalid state changes or might violate constraints like minimum or intergreen times.

Instead the offset must be moved by shortening or extending stages. Since all groups remain in the same state during a stage, this is guaranteed to never cause invalid state changes.

### Extending and Shortening Stages
Shortening stages will move the offset forward, while extending stages will move it backward.

Since the program is cyclic, reaching the target offset can be achieved either by shortening or extending stages. The quickest way should be chosen.

Only stages with `min` defined can be shortened and only stages with `max` defined can be extended.

Stages with neither `min` nor `max` are fixed in duration. If no stage defines a `min` or `max` the offset cannot be moved and the controller cannot be coordinated with other controllers.

When trying to reach the target offset, shortening/extending is done one stage at a time, while respecting min/max durations. Reaching the target offset might take more than one cycle, depending on the min/max values defined and the target offset.

Once the target offset is reached, the controller must adjust stage durations to ensure that the actual cycle time matches the cycle time defined in the program, taking into account the transition durations.

A program is invalid if the stages cannot be extended/shortened so that the actual cycle time matches the cycle time defined for the program.

## Faults
The controller must respect all safety constraints (minimum green, intergreen, etc.). Invalid configurations or unsafe transitions result in a fault.
Programs should be validated using a simulator or test tool before deployment.

