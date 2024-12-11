# RSMP TLC Programming

This repository is used for work on making programming Traffic Light Controllers easier and more vendor-neutral.

Work will include drafting an open RSMP specification for Traffic Light Controller programs.

See the [overview](overview.md).

The goa is to crate a common data structure containing traffic programming parameters and
describe the way these parameters should be interpreted in a traffic light controller.

The data structure should provide enough information to produce identical traffic behaviour on any controller executing it,
given the same detector and communication input.
