# Group-based Program
A group-based program is flexible type of control strategy based on defining contrainst and sensor input. The controller then determines
all state changes.

Constrains can be minimum and maximum times, groups that must or must not be green at the same time, etc.

If you define enough constrainst in can become a fixed-time program. Or if you define very few constrains it
can behave like a fully adaptive program. It's common to conceptually work with stages, although this is just one way to use the strategy.

## Prerequisites
Running a group-based  program depends on regional, controller and intersection [configurations](configurations.md), which are defined outside the
signal program.

Specifically, signal groups and the conflict matrix are defines in the intersection configurarion, not in the signal program.

Similary, regional settings like red-yellow time is defined in the regional/controller/intersection configuration, not in the signal program.

Controllers must use UTC [synchronized](synchronization.md) using NTP.

## Cycle counter
A group-based has no inherent cycle time but you can enforce with constrains.

## Structure
A fixed-time program defines the constraints:

```yaml
groups: ["a1","a2","b1","b2"]
constraints: {
  timing:
  relations:
  inputs:
}
```

- `groups` is an ordered lists of signal groups. Must match the actual groups.
- `constraints` is a hash of constraints

Conflicts does not have to be defined as constraints, as this is already defined in the conflict matrix.

## Switching Programs

## Failures
You are responsible for making sure the appropriate safety times, etc. are respected.
If running a program results in conflict or inconsistencies, a fault will occur.

Depending on the configuration of the controller, a fault can lead e.g. dark mode, yellow flah or switch to a fault program.

To avoid failures, programs should be validated using a suitable simulator or test tool before deployment.
