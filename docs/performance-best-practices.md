# Performance Best Practices

[< Back to Index](scripting-reference.md)

---

The game runs at 60 ticks per second, giving **16.67ms total** for all game logic per tick. Every programmable block shares this budget.

## 1. Cache Block References

```csharp
// BAD: Fetches every single run
public void Main()
{
    List<IMyThrust> thrusters = new List<IMyThrust>();
    GridTerminalSystem.GetBlocksOfType(thrusters);
}

// GOOD: Fetch once, reuse
List<IMyThrust> _thrusters = new List<IMyThrust>();

public Program()
{
    GridTerminalSystem.GetBlocksOfType(_thrusters);
}
```

## 2. Use the Slowest Adequate Update Rate

```csharp
// BAD: Update1 for a status display (60 updates/sec for text nobody reads that fast)
Runtime.UpdateFrequency = UpdateFrequency.Update1;

// GOOD: Update100 is fine for status displays
Runtime.UpdateFrequency = UpdateFrequency.Update100;

// Update1 only for: autopilot, PID controllers, precise manoeuvres
// Update10 for: responsive monitoring, combat scripts
// Update100 for: status displays, inventory management, slow checks
```

## 3. Spread Work Across Ticks

```csharp
int _currentIndex = 0;

public void Main()
{
    // Process 5 blocks per tick instead of all 200
    int end = Math.Min(_currentIndex + 5, _allBlocks.Count);
    for (int i = _currentIndex; i < end; i++)
    {
        ProcessBlock(_allBlocks[i]);
    }
    _currentIndex = end >= _allBlocks.Count ? 0 : end;
}
```

## 4. Avoid String Concatenation in Loops

```csharp
// BAD: Creates many intermediate strings
string output = "";
foreach (var item in items)
    output += item.ToString() + "\n";

// GOOD: Use StringBuilder
StringBuilder sb = new StringBuilder();
foreach (var item in items)
    sb.AppendLine(item.ToString());
lcd.WriteText(sb.ToString());
```

## 5. Monitor Your Performance

```csharp
Echo($"Instructions: {Runtime.CurrentInstructionCount}/{Runtime.MaxInstructionCount}");
Echo($"Last run: {Runtime.LastRunTimeMs:F3}ms");
```
