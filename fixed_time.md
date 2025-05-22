# Fixed Time Program
A fixed time program is a basic form of control strategy which define a fixed cycle length and a fixed schedule of when signal groops change. There is no use of detectors and each cycle is exactly the same.

## Prerequisites
Running a fixed time program depends on regional, controller and intersection [configurations](configurations.md), which are defined outside the
signal program.

Specifically, signal groups and the conflict matrix are defines in the intersection configurarion, not in the signal program.

Similary, regional settings like red-yellow time is defined in the regional/controller/intersection configuration, not in the signal program.

Controllers must use UTC [synchronized](synchronization.md) using NTP.

## Cycle counter
UTC is converted to Unix Time to get a global second counter.

The **base cycle counter** is defined as the qoutient of Unix Time divided by the cycle length.

The **cycle counter** is defined as the qoutient of Unix Time plus offset divided by the cycle length.

Multiple controllers can be coordinated if they all run fixed-time program with the same cycle length. Offset is then used to adjust the relative phasing of the fixed-time programs.

## Structure
A fixed-time program defines the cycle length and offset and all signal group transitions:

```yaml
length: 60
offset: 0
groups: ["a1","a2","b1","b2"]
states:
  0:    "00AA"
  2.5:  "11AA"
  30:   "AA00"
  34:   "AA11"
skips: { 2: 20 }
waits: { 22: 10, 32: 20 }
switch: 2
```

- `length` defines the cycle length in seconds. Must be positive. When this time is reached, the program starts over.
- `offset` defines the cycle offset in seconds. Must be in the range 0..length.
- `groups` is an ordered lists of signal groups. Must match the actual groups.
- `states` is a hash of signal states, with keys indicating the time in seconds and the representing the state of all groups.
  Each character in the string represent the state of a signal group;
  the first character corresponds to the first group in the `groups` list,
  the second characater to the second item, etc. The allowed state values are explained below.
  The string must specify the state of all groups, ie. the length must be equal to the number of items in `groups`.
- `skips` is an optional hash of skip points, with each skip point defined as <location>:<duration>.
  Locations must be equal to or greater than zero and less than `length`. Duration must be greater than zero and less than `length`.
- `waits` is a hash of wait points, with each wait point defined as <location>:<duration>. Must contain at least item.
  Locations must be equal to or greater than zero and less than `length`. Duration must be greater than zero and less than `length`.
- `switch` defines the location of the switch point, where the controller can switch to/from other program.
  Must be greater than zero and less than `length`

The program above is for a simple four-legged interesection with A and B directions. The intersection has four singal groups; a1, a2, b1 and b2. The A direction is green for the first 30s and the B direction is green (1) for the last 30s, with some red-yellow (0) transitions in between:

```
a1     |011111111111111AAAAAAAAAAAAAAA|
a2     |011111111111111AAAAAAAAAAAAAAA|
b1     |AAAAAAAAAAAAAAA001111111111111|
b2     |AAAAAAAAAAAAAAA001111111111111|
       0s             30s             60s
```

### States
Avaiable states:

- a: Disabled, dark
- 0: Red-yellow
- 1: Minimum green
- A: Red rest without start order

(These states are taken from https://rsmp-nordic.github.io/rsmp_sxl_traffic_lights/1.2.1/signal_group_status.html)


## Changing Offset
A changing in offset can happen for several reasons, e.g.:
- manual change of the offset
- change between signal programs
- synchronization of the underlying UTC time
- leap seconds

However, the offset cannot simply be abruptly changed, as this might cause invalid state changes, or might violate constrainst like minimum or inter-green times.

Instead the desired offset must be reached as quickly as possible while respecting all constraints.

### Skip and Wait Points
Skip and wait points are used to specify how the offset can be shifted. The program above can be shown as:

```
a1     |01111111111111AAAAAAAAAAAAA|
a2     |01111111111111AAAAAAAAAAAAA|
b1     |AAAAAAAAAAAAAA0011111111111|
b2     |AAAAAAAAAAAAAA0011111111111|
skip   | 10-------->               |
wait   |           10   20         |
switch |                           |
       0s            30s           60s
```

Adjusting the location and duration of skip and wait points can be used to control how different parts of the cycle is prioritized when changing offset,
and how many cycles is used to reach the target offset.

When the offset must shift the controller first determine the direction. Since fixed-time programs and the cycle counter are cyclic,
the phase difference between the currrent offset and the target offset is computed, which will be between zero and the cycle time.
If less than half the cycle time, the offset must be reduced, otherwise it must be increased.

A **skip point** is defined by a location and a duration.
If the offset needs to increase and a skip point is reached, the controllers moves the cycle counter ahead by the duration of the skip point.
A partial skip is not allowed. The start and end of a skip must therefore either have the same state, or the state change must be valid.

A **wait point** is also defined by a location and duration.
If the offset needs to decrease and a wait point is reached, the controller pauses the cycle counter. If the required shift if smaller than the duration of the wait point,
the controller waits for the required duration, after which the offset shift is complete. Otherwise it waits for the full duration of the wait point and continues.

## Switching Programs
A switch between programs can only happen at switch points. Switch points must be placed where state changes are valid.

P1:
```
a1     |01111111111111AAAAAAAAAAAAA|
a2     |01111111111111AAAAAAAAAAAAA|
b1     |AAAAAAAAAAAAAA0011111111111|
b2     |AAAAAAAAAAAAAA0011111111111|
switch | *                         |
       0s            30s           60s
```

P2:
```
a1     |AAAAAAAAAAAAA0111111111111111111111|
a2     |AAAAAAAAAAAAA0111111111111111111111|
b1     |0011111111111AAAAAAAAAAAAAAAAAAAAAA|
b2     |0011111111111AAAAAAAAAAAAAAAAAAAAAA|
switch |              *                    |
       0s            30s           60s
```

Notice how the group states are the same for the switch point in P1 and P2.

When you reach a switch point in P1 you can switch to a switch point in P2, and the cycle counter jumps to the location of the destination switch point in P2.

Based on the base counter, this new value of the cycle counter implies a new offset. To reach the offset defined in P2 the skip and wait points defined in P2 are used.

## Failures
You are responsible for making sure the appropriate safety times, etc. are respected.
If running a program results in conflict or inconsistencies, a fault will occur.

Depending on the configuration of the controller, a fault can lead e.g. dark mode, yellow flah or switch to a fault program.

To avoid failures, programs should be validated using a suitable simulator or test tool before deployment.
