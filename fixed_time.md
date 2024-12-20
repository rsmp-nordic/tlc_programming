# Fixed Time Program
A fixed time program is a basic form of control strategy which define a fixed cycle length and a fixed schedule of when signal groops change.

The controller is playing back a predefined program. There is no use of detectors and each cycle is exactly the same.

## Prerequisites
Running a fixed time program depends on regional, controller and intersection [configurations](configurations.md), which are defined outside the
signal program.

Specifiically, signal groups and the conflict matrix are defines in the intersection configurarion, not in the signal program.

Similary, regional settings like red-yellow time is defined in the regional/controller/intersection configuration, not in the signal program.

## Structure
A fixed-time program defines a cycle length and the transitions:

```yaml
length: 60
groups: ["a1","a2","b1","b2"]
states:
  0:    "00AA"
  2.5:  "11AA"
  30:   "AA00"
  34:   "AA11"
```

- `length` defines the cycle length in seconds, in this case 60s. When this time is reached, the program starts over.
- `groups` is an ordered lists of signal groups.
- `states` is a hash of signal states, with keys indicating the time in seconds and the representing the group states.
  Each character in the string represent the state of a signal group;
  the first character corresponds to the first group in the `groups` list,
  the second characater to the second item, etc. The allowed state values are explained below.

The above program is for a simple four-legged interesection with A and B directions. The intersection has four singal groups; a1, a2, b1 and b2. The A direction is green for the first 30s and the B direction is green (1) for the last 30s, with some red-yellow (0) transitions in between:

```
a1  |011111111111111AAAAAAAAAAAAAAA|
a2  |011111111111111AAAAAAAAAAAAAAA|
b1  |AAAAAAAAAAAAAAA001111111111111|
b2  |AAAAAAAAAAAAAAA001111111111111|
    0s              30s            60s
```

## States
Avaiable states:

- a: Disabled, dark
- 0: Red-yellow
- 1: Minimum green
- A: Red rest without start order

(These states are taken from https://rsmp-nordic.github.io/rsmp_sxl_traffic_lights/1.2.1/signal_group_status.html)

## Failures
You are responsible for making sure the appropriate safety times, etc. are respected.
If running a program results in conflict or inconsistencies, a fault will occur.

Depending on the configuration of the controller, a fault can lead e.g. dark mode, yellow flah or switch to a fault program.

To avoid failures, programs should be validated using a suitable simulator or test tool before deployment.
