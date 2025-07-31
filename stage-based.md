# Stage-based Strategy
A stage-based program consists of stages separated by transitions. All state changes occur during transitions, while state remain static during stages.

## Prerequisites
Running a stage-based program depends on regional, controller, and intersection [configurations](configurations.md), which are defined outside the signal program.

Specifically, signal groups and the conflict matrix are defined in the intersection configuration, not in the signal program.

Similarly, regional settings like red-yellow time are defined in the regional/controller/intersection configuration, not in the signal program.

Controllers must use UTC [synchronized](synchronization.md) using NTP.

## Structure
A stage-based program has the following structure:

```yaml
cycle: 60
offset: 0
groups: ["a1", "a2", "b1", "b2", "a1_l"]
stages:
  main:
    open: ["a1", "a2"]
    duration: { default: 20, max: 29}
    transitions:
      side: { 0: "11000", 3: "00220", 5: null }
      turn: { 0: "11000", 3: null }
  side:
    open: ["b1", "b2"]
    duration: { min: 10, default: 20, max: 26}
    transitions:
      turn: { 0: "11000", 3: null }
  turn:
    open: ["a1_l"]
    duration: { default: 10}
    transitions:
      main: { 0: "11000", 3: null }
```

- **cycle**: Cycle time
- **offset**: Default offset
- **groups**: List of all signal groups
- **stages**: Map of stages with stage name and options:
  - `open`: List of signal groups that are open (typically green).
  - `duration`: Durations in seconds. `default` is required, while `min` and `max` are optional and specify possible shortening and extension.
  - `transitions`: Possible transitions. The key is the name of the stage to transition to, while the value is a transition map.

A transition map defines all state changes during a transition:
  - keys: Transition time in seconds, measured from the start of the transition.
  - values: A string containing the state of all signal groups, with one character for each group, in the order defined in the 'groups' attribute.

Transition maps must include one element with the time set to 0, representing the start of the transition.
Transition maps must include one element with the value set to null/nil, which indicates the end of the transition. No item can have times later than the end.

In the program above, going through stages main-side-turn can be visualized as:

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

## Stages
Each stage is defined by which groups are open and for how long. All other groups must be closed.

Open typically means green, while closed typically means red. But depending on the type of group it could be e.g. a white horizontal or vertical bar for public transport.

All groups remain in the same state throughout the stage. All state changes happen during transitions.

You can define how much the stage can be shortened or extended by setting `min` and/or `max` durations.

## Transitions
A transition defines how to move from one stage to another by explicitly listing all state changes, including intermediate states like yellow.

A transition does not include the start and end states, as these are defined by the stages you come from and go to.

A particular transition always goes through the same state changes with the same duration.

If a transition between stages A and B is not defined, the program cannot transition directly from A to B, although it might be possible to reach B via other stages.

When designing a stage-based program, it must be ensured that all transitions are valid.

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

## Switching Programs
A program switch can occur only at the start of the stage defined in `switch`.

It's allowed to switch between programs using different control strategies. But whether you switch to a program with the same or different control strategy, the current and target programs must have compatible signal states at the switch point.

When a switch is requested the controller continues until the switch point, then continues from the switch point in the target program. When switching to a stage-based program, the program continues from the start of the stage defined as the switch point.

Once the program has switched, a new target offset is determined and the offset is gradually moved to the target, using the mechanism defined for the type of strategy of the target program.

## Failures
The controller must respect all safety constraints (minimum green, intergreen, etc.). Invalid configurations or unsafe transitions result in a fault.
Programs should be validated using a simulator or test tool before deployment.
