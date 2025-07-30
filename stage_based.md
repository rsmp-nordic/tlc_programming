# Stage-based Strategy
With the stage-based control strategy, the program consists of stages run in a specific order, separated by interstages.

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
groups: ["a1", "a2", "b1", "b2"]
stages:
  main: 
    groups: ["a1", "a2"]
    duration: 20
    max: 29
  side:
    groups: ["b1", "b2"]
    duration: 20
    min: 10
    max: 26
  turn:
    groups: ["b2"]
    duration: 10
order: [:main, :turn, :side]
switch: :main
```

- **cycle**: Cycle time
- **offset**: Default offset
- **groups**: List of all signal groups
- **stages**: Map of stages with stage name and options:
  - `groups`: List of signal groups that are open (green).
  - `duration`: Default duration in seconds.
  - `min`/`max`: Optional minimum/maximum durations when shortening or extending the stage.
- **order**: Sequence to run stages in.
- **switch**: Where a program switch is allowed. Switch happens at the beginning of the stage.

The above program can be visualized like this:

```
stage  |main      |..|turn |..|side     |..|
a1     |oooooooooo|  |     |  |         |  |
a2     |oooooooooo|  |     |  |         |  |
b1     |          |  |     |  |ooooooooo|  |
b2     |          |  |ooooo|  |ooooooooo|  |
switch |*         |  |     |  |         |  |
       0s                                  60s
```

Interstages are indicated with ".." and do not show group states because they are determined by the controller at run time.

## Stages
Each stage is defined by which groups are open and for how long. All other groups will be closed.

Open typically means green, while closed typically means red. But depending on the type of group it could be e.g. a white horizontal or vertical bar for public transport.

All groups remain in the same state throughout the stage. All state changes happen during interstages.

You can define how much the stage can be shortened or extended, which will be used by the controller to adjust stage durations to ensure that the cycle time is respected.
The controller will also extend and shorten stages when the offset needs to be moved.

## Interstages
Interstages are not explicitly defined. The controller determines how to switch between stages at run time using intermediate states.
This will be based on configurations of intergreen times, geometry, safety times etc.

## Changing Offset
A change in offset can happen for several reasons, e.g.:

- manual change of the offset
- change between signal programs
- synchronization of the underlying UTC time
- leap seconds

However, the offset cannot simply be abruptly changed, as this might cause invalid state changes, or might violate constraints like minimum or inter-green times.

Instead the offset must be moved by shortening of extending stages. Since all groups remain in the same stage, this is guaranteed to never cause invalid state changes. The interstages are not affected when moving the offset.

### Extending and Shortening Stages
Shortening stages will move the offset forward, while extending stages will move the backward.

Since the program is cyclic, reaching the target offset get can be done either by shorting or extending stages, but the fastest way should be chosen.

Only stages with `min` set can be shortened, while only stages with `max`set can be extended. Stages with neither `min`nor `max` are fixed in duration. If no `min`or `max` is set for any stage, the offset cannot be moved, and the controller cannot be coordinated with other controllers.

Reaching the target offset might take more than one cycle, depending on the min/max values defined.

Once the target offset is reached, the controller must adjust stage durations to ensure that the cycle time matches the value defined in the program, taking into account the interstage durations.

### Proportional Offset Adjustment
Shortening and extending stages must be done proportionally, while respecing min and max values.


The possible shortening of a stage is computed as `default - min`, while the possible extension is computed as `max - default`.

The possible shortening/extension of each stage is calculated as:
- extension_possible = max - default
- shortening_possible = default - min

The total possible shortening/extension of all stages is calculated as:
- total_extension_possible = sum of all extension_possible values
- total_shortening_possible = sum of all shortening_possible

The proportion for each stage is calculated as:
- extension_proportion = extension_possible / total_extension_possible
- shortening_proportion = shortening_possible / total_shortening_possible


For the program above, the proportions for extending are calculated as:

- extension_possible (main) = max - default = 29 - 20 = 9
- extension_possible (side) = max - default = 26 - 20 = 6
- total_extension_possible = sum of all extension_possible values = 9 + 6 = 15
- extension_proportion (main) = extension_possible / total_extension_possible = 9 / 15 = 6/10 = 60%
- extension_proportion (side) = extension_possible / total_extension_possible = 6 / 15 = 4/10 = 40%

The proportions for shortening are calculated as:

- shortening_possible (side) = duration - min = 20 - 10 = 10
- total_shortening_possible = sum of all shortening_possible values = 10
- shortening_proportion (side) = shortening_possible / total_shortening_possible =  10/10 = 100%


|Stage|Shorten|Extend|
|--|--|--|
|main||9s (60%)|
|side|10s (100%)|6s (40%)|
|turn|||

The proportions are shown as percentages.


If the offset must be moved back (i.e. stages extended), then the duration is distributed according to the percentages.

For example, if the offset must be moved back by 10s, then the `main` stage will be extended by 6s (60% of 10s), while `side` will be extended by 4s (40% of 10s).

After a single cycle, the offset target will be reached, and the stage will go back to durations that ensure that the cycle time is 60s (including interstages).

Max value for extension and shortening must be respected, which might mean that several cycles are require to reach the target offset.

For example, if the offset must be moved back (and thus stages extended) by 25s, then the `main` stage needs to be extended by 15s (60% of 25s), while `side` needs to be extended by 10s (40% of 25s). But the max extensions must be respected, so in the first cycle, `main` can only be extended by 9s, and `side` by 6s, which will move the offset back by 15s, leaving it 10s from the target. In the second cycle, `main` will be extended by 6s, while `side` is extended by 4s, after which the target offset is reached.

## Switching Programs
A program switch can occur only at the start of the stage defined in `switch`. Both the current and target programs must have compatible signal states at the switch point to ensure a safe transition.

It's allowed to switch between programs using different control strategies.

When a switch is requested the controller continues until the switch point, then transitions to the switch point in the target program.
A new target offset is determined, and the offset is gradually moved to the target, using the mechanism defined for the type of strategy of the target program.

## Failures
The controller must respect all safety constraints (minimum green, intergreen, etc.). Invalid configurations or unsafe transitions result in a fault.
Programs should be validated using a simulator or test tool before deployment.

