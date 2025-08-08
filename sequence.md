# Sequence
A sequence is a a basic sequence of states that signal groups goes through from start to end. A sequence does not cycle and has no offset.

Sequences are used e.g. for [startup/shutdown](startup_shutdown.md) and when a [fault](fault.md) occurs.

Example:
```yaml
groups: ["a1","a2","b1","b2"]
states: [
  "0000", 4, 
  "1100", 4,
  "AA00", 2
]
```

Where:
- groups: list of signal groups
- states: an array of alternating state and duration items. Must have an even number of items.

## Program switching
Switching to a sequence always happens at the beginning or end of the sequence.
Switching from a sequece can ony happen at the end of the sequence, except if a fault happens in which case the controller immediately switches to the [fault](fault.md) sequence.

If no target program is set when reaching the end of a sequence, it's a fault. 

