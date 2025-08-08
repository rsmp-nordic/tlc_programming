# Sequence
A sequence is a a basic sequence of states that signal groups goes through from start to end.

A sequence doers not cycle and has no offset or switch points.

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

