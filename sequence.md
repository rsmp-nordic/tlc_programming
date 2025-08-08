# Sequence
A sequence is a a basic sequence of states that signal groups goes through from start to end.

A sequence does not cycle and has no offset.

Switching to/from a sequence always happens atthe beginning or end.

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
