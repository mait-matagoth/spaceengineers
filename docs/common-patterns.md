# Common Patterns

[< Back to Index](scripting-reference.md)

---

## Toggle Pattern

```csharp
bool _isActive = false;

public void Main(string argument)
{
    if (argument == "toggle")
    {
        _isActive = !_isActive;
        List<IMyLightingBlock> lights = new List<IMyLightingBlock>();
        GridTerminalSystem.GetBlocksOfType(lights);
        foreach (var light in lights)
            light.Enabled = _isActive;
    }
}
```

---

## Multi-Command Pattern

```csharp
public void Main(string argument, UpdateType updateSource)
{
    switch (argument.ToLower())
    {
        case "start":
            Runtime.UpdateFrequency = UpdateFrequency.Update10;
            break;
        case "stop":
            Runtime.UpdateFrequency = UpdateFrequency.None;
            break;
        case "reset":
            ResetAll();
            break;
        default:
            if ((updateSource & UpdateType.Update10) != 0)
                RunContinuousLogic();
            break;
    }
}
```

---

## Safe Block Access Pattern

```csharp
T GetBlock<T>(string name) where T : class, IMyTerminalBlock
{
    var block = GridTerminalSystem.GetBlockWithName(name) as T;
    if (block == null)
        Echo($"WARNING: Block '{name}' not found or wrong type");
    return block;
}

// Usage
IMyTextPanel lcd = GetBlock<IMyTextPanel>("Status LCD");
lcd?.WriteText("Safe access");
```

---

## Inventory Summary Pattern

```csharp
void ShowInventorySummary(IMyTextSurface surface)
{
    List<IMyCargoContainer> containers = new List<IMyCargoContainer>();
    GridTerminalSystem.GetBlocksOfType(containers);

    float totalVolume = 0;
    float maxVolume = 0;

    foreach (var container in containers)
    {
        var inv = container.GetInventory(0);
        totalVolume += (float)inv.CurrentVolume;
        maxVolume += (float)inv.MaxVolume;
    }

    float ratio = maxVolume > 0 ? totalVolume / maxVolume : 0;
    StringBuilder sb = new StringBuilder();
    sb.AppendLine("=== Cargo Status ===");
    sb.AppendLine($"Fill: {BuildProgressBar(ratio)} {ratio * 100:F0}%");
    sb.AppendLine($"Volume: {totalVolume:F1} / {maxVolume:F1} L");
    surface.WriteText(sb.ToString());
}
```
