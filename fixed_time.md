# Fixed Time Program
A fixed time program is a basic form of control strategy which define a fixed cycle length and a fixed schedule of when signal grooups change.

All cycles are exactly the same, and there is no use of detectors. The controller is just playing back a predefined program.

## Prerequisites
Running a fixed time program depends on separate intersection, controller and region configurations.

Specifiically, signal groups and the conflict matrix are defines in the intersection configurarion, not in the signal program.

Similary, regional settings like red-yellow time is defined in the regional/controller/intersection configuration, not in the signal program.

## Structure
A fixed time program defines a cycle length, and has a number of commands at specific times.

Each command lists the signal groups it affects.

```yaml
length: 60
commands:
  0: {"start": ["a1]}
  ...
```

- `length` defines the cycle length in seconds, in this case 60s. When this time is reached, the program starts over.
- `commands` is a hash of commands. The key indicating the time in seconds, while the value is a hash of commands,
each with a command key and a list of affected singal groups. See below for available commands.

## Commands
Consider a simple four-legged interesection with A and B directions. The intersection has four singal groups; a1, a2, b1 and b2.

We want a simple fixed time program with a cycle length of 60s The A direction must be green for the first 30s and the B direction must green for the last 30s, with some yellow transitions in between:

```
a1  |--------        |
a2  |--------        |
b1  |        --------|
b2  |        --------|
    0s       30s     60s
```

Let's see how a program for this intersection can be written using either smart or manual commands.

### Smart Commands
With smart commands, you request that groups reach a specific state, and let the controller determines how, e.g. by first going to yellow.

- `start`: Start groups. Depending on the type of a signal group, this could mean green, flashing., etc.
- `stop`: Stop groups. Depending on the type of signal group, this could mean red, dark, etc.

Example:

```yaml
length: 60
commands:
  0: { "start": ["a1","a2"], "stop": ["b1", "b2"] }
  30: { "stop": ["a1","a2"], "start": ["b1", "b2"] }
```

Smart commands are convenient and the program definition is concise, but you don't have the option
to custome transitions, since they are handled automatically by the controller.

### Manual Commands
Manual commands are used to force a particular state immediately, making it possible to customize every transition and state:

- `green`
- `red`
- `red-yellow`

You are responsible for making sure the appropriate safety times, etc. are respected. If not, a fault will occur when you
run the program.

Example:

```yaml
length: 60
commands:
  0: { "red-yellow": ["a1", "a2"], ""red": ["b1", "b2"] }
  3: { "green": ["a1", "a2"] }
  30: { "stop": ["a1","a2"], "red-yellow": ["b1", "b2"] }
  40: { "green": ["b1", "b2"] }
```

Here we use manual commands to but groups to red-yellow for three seconds before going green.
Manual commands lets you can customize every transition and state, but you're responsible for taking into account safety times, 
yellow transitions, etc. and program definitions are not as concise as when using smart commands.

### Mixing Smart and Manual Command
You can mix smart and manual commands. This is convenient when you only need to customize specific things.

Example:

```yaml
length: 60
commands:
  0: { "start": ["a1","a2"], "stop": ["b1", "b2"] }
  30: { "stop": ["a1", "a2"], "red-yellow": ["b1", "b2"] }
  33: { "green": ["b1", "b2"] }
```

Here we use manual commands to keep b1 and b2 are in red-yellow for 10s before going to green, but use smart commands
for everything else.

## Failures
If running a program results in conflict or inconsistencies, a fault will occur.

Faults can occur with both manual and smart commands if they are used in a way that lead to conflict or
violation of transition/timing rules.

Depending on the configuration of the controller, a fault can lead e.g. dark mode, yellow flah or switch to a fault program.

To avoid failures, programs should be validated using a suitable simulator or test tool before deployment.
