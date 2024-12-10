# Overview
An RSMP TLC Program is a standardized way to describe a signal program for a traffic light controller.

```mermaid
graph LR
Editor --> Program@{shape: document}
Program --> Controller
Config@{shape: document} --> Controller
Controller --> Lamps
```

Programs can be created with any suitable editor, and then deployed to the controller.

To run a a program, other [configurations](configurations.md) must be present on the controller. including:
- regional configurations, e.g. red-yellow times.
- controller configurations, e.g. list of intersections.
- intersection configurations, e.g. signal groups and their conflict matrix.

When the controller runs the program it will change lamps over time according, e.g. change 
between red, yellow and green. Flashing yellow, dark, etc. are also possible, depending
on the signal group type.
 
## Format
Program are stored as YAML files, but can be transmitted as eg. JSON or CBOR.

## Control Strategies
Only [fixed-time](fixed_time.md) control is supported initially.

Additional control strategies will be added later.

