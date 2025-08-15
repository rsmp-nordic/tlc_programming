# Sequence
A sequence is an array of explicit state/duration pairs that the controller goes through from start to end. A sequence does not cycle and has no offset.

```
[<state>, <duration>, <state>, <duration>, ....]
```

A state sequence always have an even number of items. An empty array is allowed.

A state is a string with one character for each signal group. The order of groups must be defined previously, typically in a field named `types`.
For exa, here's how a sequence is used in a startup program:

```yaml
groups: ["a1","a2","b1","b2"]
states: [
  "0000", 4,
  "1100", 4,
  "AA00", 2
]
```

Sequences are used for [startup/shutdown](startup_shutdown.md) program and also when definining transitions in [stage-based](stage-based.md) programs.
