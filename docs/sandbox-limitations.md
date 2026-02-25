# Sandbox Limitations

[< Back to Index](scripting-reference.md)

---

## Whitelisted Namespaces

You can **only** use:

| Namespace | Purpose |
|-----------|---------|
| `Sandbox.ModAPI.Ingame` | All in-game block interfaces |
| `Sandbox.ModAPI.Interfaces` | Terminal properties and actions |
| `Sandbox.Common.ObjectBuilders` | Item/block type definitions |
| `VRageMath` | Vectors, matrices, maths |
| `VRage` | Core types (`MyFixedPoint`, etc.) |
| `VRage.Game` | Game enums and definitions |
| Standard .NET (limited) | `System`, `System.Collections.Generic`, `System.Text`, `System.Linq` |

You **cannot** use: file I/O, networking, reflection, threading, `System.Diagnostics`, or any `Sandbox.ModAPI` (non-Ingame) namespace.

---

## Hard Limits

| Limit | Value |
|-------|-------|
| Script size | 100,000 characters |
| Instructions per run | 50,000 significant junctions |
| Call chain depth | Limited (exact value varies) |
| Tick budget (all scripts) | 16.67ms total per game tick |

---

## What Counts as an "Instruction"

Entering any of these counts as one instruction:
- Method calls
- Loop iterations
- Conditional branches (`if`, `switch`)
- Exception handlers

This is a **safeguard against infinite loops**, not a performance metric. You can have a low instruction count and still be slow if each instruction is expensive.
