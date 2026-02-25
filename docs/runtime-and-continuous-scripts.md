# Runtime API & Continuous Scripts

[< Back to Index](scripting-reference.md)

---

## The Runtime API

Access via `Runtime` property. Provides execution information and control.

| Property | Type | Description |
|----------|------|-------------|
| `UpdateFrequency` | `UpdateFrequency` | Set self-update rate |
| `TimeSinceLastRun` | `TimeSpan` | Game time since last `Main()` call |
| `LastRunTimeMs` | `double` | Real-time execution duration of last run (ms) |
| `CurrentInstructionCount` | `int` | Instructions executed so far this run |
| `MaxInstructionCount` | `int` | Maximum allowed instructions per run |
| `CurrentCallChainDepth` | `int` | Current method nesting depth |
| `MaxCallChainDepth` | `int` | Maximum allowed nesting |

**`UpdateFrequency` flags** (can be combined with `|`):

| Flag | Frequency | Use Case |
|------|-----------|----------|
| `None` | Never (manual only) | One-shot scripts |
| `Once` | Next tick only, then stops | Deferred initialisation |
| `Update1` | Every tick (60/s) | Precise control (autopilot, PID) |
| `Update10` | Every 10 ticks (6/s) | Responsive monitoring |
| `Update100` | Every 100 ticks (~1.67/s) | Status displays, slow checks |

---

## Continuous Running Scripts

By default, scripts only run when manually triggered. To run continuously:

```csharp
public Program()
{
    // Run every 100 ticks (~1.67 seconds)
    Runtime.UpdateFrequency = UpdateFrequency.Update100;
}

public void Main(string argument, UpdateType updateSource)
{
    // Check what triggered us
    if ((updateSource & UpdateType.Update100) != 0)
    {
        // Periodic update logic
        UpdateStatusDisplay();
    }

    if ((updateSource & UpdateType.Trigger) != 0)
    {
        // Triggered by timer/toolbar — handle the argument
        HandleCommand(argument);
    }
}
```

---

## State Machines for Complex Operations

For operations that take many ticks (e.g., automated docking sequences):

```csharp
IEnumerator<bool> _stateMachine;

public void Main(string argument, UpdateType updateSource)
{
    if (argument == "start")
    {
        _stateMachine = RunDockingSequence();
        Runtime.UpdateFrequency = UpdateFrequency.Update10;
    }

    if (_stateMachine != null)
    {
        if (!_stateMachine.MoveNext() || !_stateMachine.Current)
        {
            _stateMachine.Dispose();
            _stateMachine = null;
            Runtime.UpdateFrequency = UpdateFrequency.None;
        }
    }
}

IEnumerator<bool> RunDockingSequence()
{
    // Step 1: Approach
    Echo("Approaching...");
    // ... set thrusters ...
    yield return true; // continue next tick

    // Step 2: Align
    Echo("Aligning...");
    // ... set gyros ...
    yield return true;

    // Step 3: Connect
    Echo("Connecting...");
    // ... activate connector ...
    yield return false; // done
}
```
