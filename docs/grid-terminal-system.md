# The Grid Terminal System

[< Back to Index](scripting-reference.md)

---

`GridTerminalSystem` is how you interact with every block on your grid. It can see all blocks on the same grid as the programmable block, plus blocks on sub-grids connected via rotors, pistons, or connectors — provided they share ownership.

## Getting Blocks

### By Exact Name (case-sensitive)

```csharp
IMyTextPanel lcd = GridTerminalSystem.GetBlockWithName("Status LCD") as IMyTextPanel;
if (lcd != null)
{
    lcd.WriteText("Hello!");
}
```

### By Type

```csharp
List<IMyThrust> thrusters = new List<IMyThrust>();
GridTerminalSystem.GetBlocksOfType(thrusters);
// thrusters now contains every thruster on the grid
```

### By Type with Filter

```csharp
List<IMyThrust> hydrogenThrusters = new List<IMyThrust>();
GridTerminalSystem.GetBlocksOfType(hydrogenThrusters,
    t => t.CustomName.Contains("Hydrogen"));
```

### By Group Name (case-sensitive)

```csharp
IMyBlockGroup group = GridTerminalSystem.GetBlockGroupWithName("Forward Thrusters");
List<IMyThrust> forwardThrusters = new List<IMyThrust>();
group?.GetBlocksOfType(forwardThrusters);
```

### By Name Search (case-insensitive, partial match)

```csharp
List<IMyTerminalBlock> blocks = new List<IMyTerminalBlock>();
GridTerminalSystem.SearchBlocksOfName("Refinery", blocks);
// Finds "Refinery", "My Refinery", "refinery 2", etc.
```

### By Entity ID (persists through renames)

```csharp
IMyTerminalBlock block = GridTerminalSystem.GetBlockWithId(entityId);
```

### Get All Blocks

```csharp
List<IMyTerminalBlock> allBlocks = new List<IMyTerminalBlock>();
GridTerminalSystem.GetBlocks(allBlocks);
```

### Get All Groups

```csharp
List<IMyBlockGroup> groups = new List<IMyBlockGroup>();
GridTerminalSystem.GetBlockGroups(groups);
```

---

## Filtering to Your Own Construct

If another ship is docked (via connector), its blocks appear in `GridTerminalSystem` too. To filter to only your own construct:

```csharp
GridTerminalSystem.GetBlocksOfType(thrusters,
    t => t.IsSameConstructAs(Me));
```

---

## Checking Access

```csharp
if (GridTerminalSystem.CanAccess(someBlock))
{
    // Block is still accessible (not destroyed, disconnected, or ownership changed)
}
```

---

## Best Practices

- **Cache block references** in `Program()` rather than fetching every `Main()` call
- Use `GetBlocksOfType()` with a predicate for efficient filtering
- Use `IsSameConstructAs(Me)` to exclude docked ships
- Use `EntityId` for stable references that survive block renaming
