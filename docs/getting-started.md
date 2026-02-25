# Getting Started

[< Back to Index](scripting-reference.md)

---

## Prerequisites

Before you can run scripts:

1. **Experimental Mode** must be enabled in game settings
2. **In-game Scripts** must be allowed in the world settings
3. Place a **Programmable Block** on your grid
4. Open it, write/paste code in the editor, click **Check Code**, then **Run**

For development outside the game, the recommended setup is:
- **Visual Studio Community** (free) with the [MDK2 extension](https://github.com/malforge/mdk2)
- This gives you IntelliSense, syntax checking, and deployment to the game

---

## Script Anatomy

Every script runs inside an implicit class that extends `MyGridProgram`. The class provides access to all the game APIs. There are three methods you can define:

### The Constructor: `Program()`

Runs **once** when the script is first compiled or the game is loaded. Use it for one-time setup.

```csharp
public Program()
{
    // Runs once at startup
    // Good for: caching block references, setting update frequency, loading state
    Runtime.UpdateFrequency = UpdateFrequency.Update100;
}
```

### The Main Method: `Main(string argument, UpdateType updateSource)`

The **required** entry point. Runs each time the script is triggered (manually, by timer, by toolbar, or by self-update).

```csharp
public void Main(string argument, UpdateType updateSource)
{
    // argument: string passed from toolbar/timer actions
    // updateSource: why this execution was triggered

    if (argument == "toggle")
    {
        // Handle toolbar argument
    }

    if ((updateSource & UpdateType.Update100) != 0)
    {
        // Handle periodic self-update
    }
}
```

**`UpdateType` flags:**
| Flag | Meaning |
|------|---------|
| `None` | No update |
| `Terminal` | Run button pressed in terminal |
| `Trigger` | Triggered by timer/toolbar |
| `Mod` | Called by a mod |
| `Script` | Called by another script (IGC) |
| `Update1` | Self-update every tick |
| `Update10` | Self-update every 10 ticks |
| `Update100` | Self-update every 100 ticks |
| `Once` | One-shot self-update |
| `IGC` | Intergrid communication message |

### The Save Method: `Save()`

Runs when the game saves or when the script is about to recompile. Use it to persist state.

```csharp
public void Save()
{
    // Persist data to the Storage string
    Storage = $"{counter};{isActive}";
}
```

---

## Built-in Properties

These are available everywhere in your script (inherited from `MyGridProgram`):

| Property | Type | Description |
|----------|------|-------------|
| `GridTerminalSystem` | `IMyGridTerminalSystem` | Access to all blocks on the grid |
| `Me` | `IMyProgrammableBlock` | Reference to this programmable block |
| `Runtime` | `IMyGridProgramRuntimeInfo` | Execution stats and update frequency |
| `Storage` | `string` | Persistent string that survives game saves |
| `Echo` | `Action<string>` | Writes text to the programmable block's detail area |
| `IGC` | `IMyIntergridCommunicationSystem` | Send/receive messages between grids |

---

## MDK Project Structure

When developing externally with MDK, your code is wrapped in:

```csharp
namespace IngameScript
{
    partial class Program : MyGridProgram
    {
        public Program() { }
        public void Save() { }
        public void Main(string argument, UpdateType updateSource) { }
    }
}
```
