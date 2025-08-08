# Faults
In case a fault happens, the controller must immediatly run the fault [sequence](sequence.md).

Ater the fault sequence has run, the controller switches to standby mode.

Example fault sequence:

```yaml
length: 8
groups: ["a1","a2","b1","b2"]
states:
  0: "1111"
  4: "AAAA"
```

If a another fault happens while running the fault sequence, the controller goes immedately to the standby mode.

