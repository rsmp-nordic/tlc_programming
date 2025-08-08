# Sequence
A sequence is a a basic sequence of states that signal groups goes through from start to end. A sequence does not cycle and has no offset.

Sequences are used e.g. for [startup/shutdown](startup_shutdown.md) and when a [fault](fault.md) occurs.

Example:
```yaml
length: 10
groups: ["a1","a2","b1","b2"]
states:
  0: "0000"
  4: "1100"
  8: "AA00"
```

Where:
- length: the length of the sequence
- groups: list of signal groups
- states: a map of time/state pairs

## Program switching
Switching to/from a sequence always happens at the beginning or end of the sequence.

Since a sequence does not cycle, a target program must be set before running the sequence, so that the controller knows where to continue when reaching the end of the sequence.

If the end of the sequence is reached and no target program is set, it's a fault.

