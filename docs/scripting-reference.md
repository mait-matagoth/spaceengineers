# Space Engineers Programmable Block - Scripting Reference

> A comprehensive guide to writing C# scripts for the Space Engineers Programmable Block.
> This is a living document - we'll expand it as we build more things.

---

## Table of Contents

1. [Prerequisites](#prerequisites)
2. [Script Anatomy](#script-anatomy)
3. [The Grid Terminal System](#the-grid-terminal-system)
4. [Block Interfaces Reference](#block-interfaces-reference)
5. [The Runtime API](#the-runtime-api)
6. [Continuous Running Scripts](#continuous-running-scripts)
7. [LCD and Text Surfaces](#lcd-and-text-surfaces)
8. [Storage and Persistence](#storage-and-persistence)
9. [Intergrid Communication (IGC)](#intergrid-communication-igc)
10. [Utility Classes](#utility-classes)
11. [Sandbox Limitations](#sandbox-limitations)
12. [Performance Best Practices](#performance-best-practices)
13. [Common Patterns](#common-patterns)
14. [Worked Example: Thrust-to-Weight Calculator](#worked-example-thrust-to-weight-calculator)
15. [External Resources](#external-resources)

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

### Built-in Properties

These are available everywhere in your script (inherited from `MyGridProgram`):

| Property | Type | Description |
|----------|------|-------------|
| `GridTerminalSystem` | `IMyGridTerminalSystem` | Access to all blocks on the grid |
| `Me` | `IMyProgrammableBlock` | Reference to this programmable block |
| `Runtime` | `IMyGridProgramRuntimeInfo` | Execution stats and update frequency |
| `Storage` | `string` | Persistent string that survives game saves |
| `Echo` | `Action<string>` | Writes text to the programmable block's detail area |
| `IGC` | `IMyIntergridCommunicationSystem` | Send/receive messages between grids |

### MDK Project Structure

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

---

## The Grid Terminal System

`GridTerminalSystem` is how you interact with every block on your grid. It can see all blocks on the same grid as the programmable block, plus blocks on sub-grids connected via rotors, pistons, or connectors — provided they share ownership.

### Getting Blocks

#### By Exact Name (case-sensitive)

```csharp
IMyTextPanel lcd = GridTerminalSystem.GetBlockWithName("Status LCD") as IMyTextPanel;
if (lcd != null)
{
    lcd.WriteText("Hello!");
}
```

#### By Type

```csharp
List<IMyThrust> thrusters = new List<IMyThrust>();
GridTerminalSystem.GetBlocksOfType(thrusters);
// thrusters now contains every thruster on the grid
```

#### By Type with Filter

```csharp
List<IMyThrust> hydrogenThrusters = new List<IMyThrust>();
GridTerminalSystem.GetBlocksOfType(hydrogenThrusters,
    t => t.CustomName.Contains("Hydrogen"));
```

#### By Group Name (case-sensitive)

```csharp
IMyBlockGroup group = GridTerminalSystem.GetBlockGroupWithName("Forward Thrusters");
List<IMyThrust> forwardThrusters = new List<IMyThrust>();
group?.GetBlocksOfType(forwardThrusters);
```

#### By Name Search (case-insensitive, partial match)

```csharp
List<IMyTerminalBlock> blocks = new List<IMyTerminalBlock>();
GridTerminalSystem.SearchBlocksOfName("Refinery", blocks);
// Finds "Refinery", "My Refinery", "refinery 2", etc.
```

#### By Entity ID (persists through renames)

```csharp
IMyTerminalBlock block = GridTerminalSystem.GetBlockWithId(entityId);
```

#### Get All Blocks

```csharp
List<IMyTerminalBlock> allBlocks = new List<IMyTerminalBlock>();
GridTerminalSystem.GetBlocks(allBlocks);
```

#### Get All Groups

```csharp
List<IMyBlockGroup> groups = new List<IMyBlockGroup>();
GridTerminalSystem.GetBlockGroups(groups);
```

### Filtering to Your Own Construct

If another ship is docked (via connector), its blocks appear in `GridTerminalSystem` too. To filter to only your own construct:

```csharp
GridTerminalSystem.GetBlocksOfType(thrusters,
    t => t.IsSameConstructAs(Me));
```

### Checking Access

```csharp
if (GridTerminalSystem.CanAccess(someBlock))
{
    // Block is still accessible (not destroyed, disconnected, or ownership changed)
}
```

### Best Practices

- **Cache block references** in `Program()` rather than fetching every `Main()` call
- Use `GetBlocksOfType()` with a predicate for efficient filtering
- Use `IsSameConstructAs(Me)` to exclude docked ships
- Use `EntityId` for stable references that survive block renaming

---

## Block Interfaces Reference

All in-game blocks implement interfaces from `Sandbox.ModAPI.Ingame`. Every block is at minimum an `IMyTerminalBlock`. The hierarchy is:

```
IMyTerminalBlock          (base: name, custom data, actions)
  └── IMyFunctionalBlock  (adds: Enabled on/off)
        └── [specific block interfaces]
```

### Base Interfaces

#### IMyTerminalBlock

Every controllable block implements this.

| Property/Method | Type | Description |
|-----------------|------|-------------|
| `CustomName` | `string` | Block name (read/write) |
| `CustomData` | `string` | Free-form string storage on the block |
| `DetailedInfo` | `string` | The info panel text (read-only) |
| `EntityId` | `long` | Unique identifier (persists through renames) |
| `IsWorking` | `bool` | Block is built and functional |
| `IsFunctional` | `bool` | Block is not damaged beyond function |
| `ShowOnHUD` | `bool` | Visible on player HUD |
| `Position` | `Vector3I` | Grid position |
| `CubeGrid` | `IMyCubeGrid` | The grid this block belongs to |
| `IsSameConstructAs(IMyTerminalBlock)` | `bool` | Same physical construct? |
| `GetActions(List<ITerminalAction>)` | `void` | Get available actions |
| `GetProperties(List<ITerminalProperty>)` | `void` | Get available properties |

#### IMyFunctionalBlock (extends IMyTerminalBlock)

| Property | Type | Description |
|----------|------|-------------|
| `Enabled` | `bool` | Turn block on/off |

### Movement & Control

#### IMyThrust

Controls thrusters (atmospheric, hydrogen, ion).

| Property | Type | Description |
|----------|------|-------------|
| `ThrustOverride` | `float` | Override thrust in Newtons (0 = no override) |
| `ThrustOverridePercentage` | `float` | Override as percentage (0.0 to 1.0) |
| `MaxThrust` | `float` | Maximum possible thrust (N) |
| `MaxEffectiveThrust` | `float` | Max thrust accounting for atmosphere/altitude |
| `CurrentThrust` | `float` | Current actual thrust output (N) |
| `CurrentThrustPercentage` | `float` | Current thrust as % of max |
| `GridThrustDirection` | `Vector3I` | Thrust direction relative to ship controller |

**Key distinction:** `MaxThrust` is the theoretical maximum. `MaxEffectiveThrust` accounts for atmospheric effects — atmospheric thrusters lose effectiveness at altitude, ion thrusters lose effectiveness in atmosphere.

#### IMyGyro

Controls gyroscopes for rotation.

| Property | Type | Description |
|----------|------|-------------|
| `GyroOverride` | `bool` | Enable manual override |
| `Yaw` | `float` | Yaw rotation speed (rad/s) |
| `Pitch` | `float` | Pitch rotation speed (rad/s) |
| `Roll` | `float` | Roll rotation speed (rad/s) |
| `GyroPower` | `float` | Power multiplier (0.0 to 1.0) |

**Note:** Override values are in the gyro's local coordinate space, not the ship's. You'll need to transform between coordinate systems for precise control.

#### IMyShipController (cockpits, remote controls, etc.)

| Property | Type | Description |
|----------|------|-------------|
| `MoveIndicator` | `Vector3` | WASD input direction |
| `RotationIndicator` | `Vector2` | Mouse rotation input |
| `RollIndicator` | `float` | Roll input |
| `IsUnderControl` | `bool` | Player is currently controlling |
| `CanControlShip` | `bool` | Can control the ship |
| `HandBrake` | `bool` | Handbrake state |
| `DampenersOverride` | `bool` | Inertia dampeners on/off |
| `GetNaturalGravity()` | `Vector3D` | Natural gravity vector at current position |
| `GetArtificialGravity()` | `Vector3D` | Artificial gravity vector |
| `GetTotalGravity()` | `Vector3D` | Combined gravity vector |
| `GetShipSpeed()` | `double` | Current speed (m/s) |
| `GetShipVelocities()` | `MyShipVelocities` | Linear and angular velocities |
| `CalculateShipMass()` | `MyShipMass` | Ship mass information |

**`MyShipMass` struct:**
| Property | Type | Description |
|----------|------|-------------|
| `BaseMass` | `float` | Mass of the grid without cargo (kg) |
| `TotalMass` | `float` | Mass including all cargo (kg) |
| `PhysicalMass` | `float` | Mass used for physics (may differ with artificial mass) |

#### IMyLandingGear

| Property | Type | Description |
|----------|------|-------------|
| `IsLocked` | `bool` | Currently locked to a surface |
| `AutoLock` | `bool` | Auto-lock when near surface |
| `LockMode` | `LandingGearMode` | `Unlocked`, `ReadyToLock`, `Locked` |
| `Lock()` | `void` | Attempt to lock |
| `Unlock()` | `void` | Unlock |

### Power & Energy

#### IMyBatteryBlock

| Property | Type | Description |
|----------|------|-------------|
| `ChargeMode` | `ChargeMode` | `Auto`, `Recharge`, `Discharge` |
| `CurrentStoredPower` | `float` | Current charge (MWh) |
| `MaxStoredPower` | `float` | Maximum capacity (MWh) |
| `CurrentInput` | `float` | Power being drawn (MW) |
| `CurrentOutput` | `float` | Power being supplied (MW) |
| `IsCharging` | `bool` | Currently charging |

#### IMyReactor

| Property | Type | Description |
|----------|------|-------------|
| `CurrentOutput` | `float` | Current power output (MW) |
| `MaxOutput` | `float` | Maximum power output (MW) |

#### IMySolarPanel / IMyWindTurbine

| Property | Type | Description |
|----------|------|-------------|
| `CurrentOutput` | `float` | Current power output (MW) |
| `MaxOutput` | `float` | Maximum power output (MW) |

#### IMyPowerProducer (category interface)

All power-producing blocks implement this. Use with `GetBlocksOfType<IMyPowerProducer>()` to get all generators regardless of type.

### Storage & Production

#### IMyCargoContainer

Primarily accessed through the `IMyInventory` interface:

```csharp
IMyCargoContainer cargo = /* get block */;
IMyInventory inv = cargo.GetInventory(0);
float currentVolume = (float)inv.CurrentVolume;
float maxVolume = (float)inv.MaxVolume;
List<MyInventoryItem> items = new List<MyInventoryItem>();
inv.GetItems(items);
```

#### IMyInventory

| Property/Method | Type | Description |
|-----------------|------|-------------|
| `CurrentVolume` | `MyFixedPoint` | Current fill volume |
| `MaxVolume` | `MyFixedPoint` | Maximum capacity |
| `CurrentMass` | `MyFixedPoint` | Current mass of contents |
| `ItemCount` | `int` | Number of item stacks |
| `GetItems(List<MyInventoryItem>)` | `void` | Get all items |
| `TransferItemTo(IMyInventory, int)` | `bool` | Move item to another inventory |
| `CanTransferItemTo(IMyInventory, MyItemType)` | `bool` | Check if transfer possible |
| `ContainItems(MyFixedPoint, MyItemType)` | `bool` | Contains specified amount? |

#### IMyAssembler

| Property | Type | Description |
|----------|------|-------------|
| `Mode` | `MyAssemblerMode` | `Assembly` or `Disassembly` |
| `IsQueueEmpty` | `bool` | Production queue empty |
| `IsProducing` | `bool` | Currently producing |
| `CooperativeMode` | `bool` | Share work with other assemblers |
| `AddQueueItem(MyDefinitionId, decimal)` | `void` | Add item to production queue |
| `GetQueue(List<MyProductionItem>)` | `void` | Get current queue |
| `ClearQueue()` | `void` | Clear production queue |

#### IMyRefinery

| Property | Type | Description |
|----------|------|-------------|
| `IsQueueEmpty` | `bool` | Processing queue empty |
| `IsProducing` | `bool` | Currently processing |

### Display & Communication

#### IMyTextPanel / IMyTextSurface

LCD panels and text surfaces for displaying information.

```csharp
// Standalone LCD panel
IMyTextPanel lcd = GridTerminalSystem.GetBlockWithName("LCD") as IMyTextPanel;
lcd.WriteText("Hello World!");

// Multi-surface block (cockpit, programmable block)
IMyTextSurfaceProvider provider = Me as IMyTextSurfaceProvider;
IMyTextSurface surface = provider.GetSurface(0); // first screen
surface.ContentType = ContentType.TEXT_AND_IMAGE;
surface.WriteText("Status: OK");
```

| Property/Method | Type | Description |
|-----------------|------|-------------|
| `WriteText(string, bool)` | `void` | Write text (false = clear first, true = append) |
| `GetText()` | `string` | Get current text |
| `ContentType` | `ContentType` | `NONE`, `TEXT_AND_IMAGE`, `SCRIPT` |
| `FontSize` | `float` | Text size |
| `FontColor` | `Color` | Text colour |
| `BackgroundColor` | `Color` | Background colour |
| `Font` | `string` | Font name |
| `TextPadding` | `float` | Padding percentage |
| `SurfaceSize` | `Vector2` | Pixel dimensions of the surface |

#### IMyRadioAntenna

| Property | Type | Description |
|----------|------|-------------|
| `Radius` | `float` | Broadcast range (m) |
| `EnableBroadcasting` | `bool` | Broadcasting on/off |
| `HudText` | `string` | Text shown on HUD |

#### IMyBeacon

| Property | Type | Description |
|----------|------|-------------|
| `Radius` | `float` | Broadcast range (m) |
| `HudText` | `string` | Text shown on HUD |

#### IMyLaserAntenna

| Property | Type | Description |
|----------|------|-------------|
| `Range` | `float` | Maximum range (m) |
| `Status` | `MyLaserAntennaStatus` | Connection status |

### Doors & Access

#### IMyDoor

| Property | Type | Description |
|----------|------|-------------|
| `Status` | `DoorStatus` | `Open`, `Closed`, `Opening`, `Closing` |
| `OpenRatio` | `float` | How open (0.0 to 1.0) |
| `OpenDoor()` | `void` | Open the door |
| `CloseDoor()` | `void` | Close the door |
| `ToggleDoor()` | `void` | Toggle open/closed |

### Weapons & Defence

#### IMyLargeTurretBase

| Property | Type | Description |
|----------|------|-------------|
| `Range` | `float` | Targeting range |
| `IsAimed` | `bool` | Currently aimed at target |
| `HasTarget` | `bool` | Has a valid target |
| `Elevation` | `float` | Vertical angle (rad) |
| `Azimuth` | `float` | Horizontal angle (rad) |

#### IMyWarhead

| Property | Type | Description |
|----------|------|-------------|
| `IsCountingDown` | `bool` | Timer active |
| `DetonationTime` | `float` | Countdown time (seconds) |
| `IsArmed` | `bool` | Armed state |
| `Detonate()` | `void` | Immediate detonation |
| `StartCountdown()` | `void` | Start countdown |
| `StopCountdown()` | `void` | Stop countdown |

### Mechanical Blocks

#### IMyMotorStator (rotors)

| Property | Type | Description |
|----------|------|-------------|
| `Angle` | `float` | Current angle (rad) |
| `TargetVelocityRPM` | `float` | Target rotation speed |
| `Torque` | `float` | Applied torque |
| `UpperLimitRad` | `float` | Upper rotation limit |
| `LowerLimitRad` | `float` | Lower rotation limit |
| `IsAttached` | `bool` | Rotor head attached |

#### IMyPistonBase

| Property | Type | Description |
|----------|------|-------------|
| `CurrentPosition` | `float` | Current extension (m) |
| `MinLimit` | `float` | Minimum extension |
| `MaxLimit` | `float` | Maximum extension |
| `Velocity` | `float` | Extension speed (m/s, negative = retract) |
| `Extend()` | `void` | Start extending |
| `Retract()` | `void` | Start retracting |

### Sensors & Detection

#### IMySensorBlock

| Property | Type | Description |
|----------|------|-------------|
| `IsActive` | `bool` | Currently detecting something |
| `LastDetectedEntity` | `MyDetectedEntityInfo` | Last detected entity info |
| `LeftExtend` / `RightExtend` | `float` | Detection field size |
| `TopExtend` / `BottomExtend` | `float` | Detection field size |
| `FrontExtend` / `BackExtend` | `float` | Detection field size |
| `DetectPlayers` | `bool` | Detect players |
| `DetectSmallShips` | `bool` | Detect small grids |
| `DetectLargeShips` | `bool` | Detect large grids |
| `DetectAsteroids` | `bool` | Detect asteroids |

#### IMyCameraBlock

| Property | Type | Description |
|----------|------|-------------|
| `CanScan(double)` | `bool` | Can raycast at distance |
| `Raycast(double, float, float)` | `MyDetectedEntityInfo` | Perform raycast |
| `AvailableScanRange` | `double` | Maximum scan range (charges over time) |

### Connectors & Merge Blocks

#### IMyShipConnector

| Property | Type | Description |
|----------|------|-------------|
| `Status` | `MyShipConnectorStatus` | `Unconnected`, `Connectable`, `Connected` |
| `OtherConnector` | `IMyShipConnector` | The connected block |
| `Connect()` | `void` | Attempt connection |
| `Disconnect()` | `void` | Disconnect |
| `ToggleConnect()` | `void` | Toggle state |
| `IsParkingEnabled` | `bool` | Parking mode |

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

### State Machines for Complex Operations

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

---

## LCD and Text Surfaces

### Writing to a Standalone LCD

```csharp
IMyTextPanel lcd = GridTerminalSystem.GetBlockWithName("Status LCD") as IMyTextPanel;
lcd.ContentType = ContentType.TEXT_AND_IMAGE;
lcd.FontSize = 1.2f;
lcd.FontColor = Color.Green;
lcd.WriteText("Systems Nominal\n");
lcd.WriteText($"Speed: {speed:F1} m/s\n", true); // true = append
```

### Writing to the Programmable Block's Own Screen

```csharp
IMyTextSurface surface = Me.GetSurface(0);
surface.ContentType = ContentType.TEXT_AND_IMAGE;
surface.WriteText("Script Running...");
```

### Writing to Cockpit Screens

```csharp
IMyShipController cockpit = GridTerminalSystem.GetBlockWithName("Cockpit") as IMyShipController;
IMyTextSurfaceProvider provider = cockpit as IMyTextSurfaceProvider;
if (provider != null)
{
    IMyTextSurface screen = provider.GetSurface(0); // centre screen
    screen.ContentType = ContentType.TEXT_AND_IMAGE;
    screen.WriteText("Dashboard");
}
```

### Building Progress Bars

```csharp
string BuildProgressBar(float ratio, int width = 20)
{
    int filled = (int)(ratio * width);
    return "[" + new string('|', filled) + new string('.', width - filled) + "]";
}

// Usage
float fuelRatio = currentFuel / maxFuel;
lcd.WriteText($"Fuel: {BuildProgressBar(fuelRatio)} {fuelRatio * 100:F0}%");
```

---

## Storage and Persistence

The `Storage` property is a single string that persists across game saves and script recompilations.

### Simple State Persistence

```csharp
int counter = 0;
bool isActive = false;

public Program()
{
    // Load state
    if (!string.IsNullOrEmpty(Storage))
    {
        string[] parts = Storage.Split(';');
        if (parts.Length >= 2)
        {
            int.TryParse(parts[0], out counter);
            bool.TryParse(parts[1], out isActive);
        }
    }
}

public void Save()
{
    Storage = $"{counter};{isActive}";
}
```

### Using CustomData for Block-Level Storage

Each block has a `CustomData` string you can use for configuration:

```csharp
// Write config
someBlock.CustomData = "speed=10\nmode=auto";

// Read config using the built-in MyIni parser
MyIni ini = new MyIni();
ini.TryParse(someBlock.CustomData);
int speed = ini.Get("config", "speed").ToInt32(10);
string mode = ini.Get("config", "mode").ToString("manual");
```

---

## Intergrid Communication (IGC)

Send messages between programmable blocks on different grids via antennas.

### Broadcasting (one-to-many)

```csharp
// Sender
IGC.SendBroadcastMessage("channel_tag", "Hello from Grid A!");

// Receiver (in Program constructor)
IGC.RegisterBroadcastListener("channel_tag");

// Receiver (in Main)
List<IMyBroadcastListener> listeners = new List<IMyBroadcastListener>();
IGC.GetBroadcastListeners(listeners);
foreach (var listener in listeners)
{
    while (listener.HasPendingMessage)
    {
        MyIGCMessage msg = listener.AcceptMessage();
        Echo($"From {msg.Source}: {msg.Data}");
    }
}
```

### Unicast (one-to-one)

```csharp
// Send to a specific programmable block's address
IGC.SendUnicastMessage(targetAddress, "tag", data);

// Receive
while (IGC.UnicastListener.HasPendingMessage)
{
    MyIGCMessage msg = IGC.UnicastListener.AcceptMessage();
    // handle message
}
```

---

## Utility Classes

### MyCommandLine — Argument Parsing

```csharp
MyCommandLine _commandLine = new MyCommandLine();

public void Main(string argument)
{
    if (_commandLine.TryParse(argument))
    {
        string command = _commandLine.Argument(0);
        switch (command)
        {
            case "start":
                Start();
                break;
            case "stop":
                Stop();
                break;
            case "set":
                string key = _commandLine.Argument(1);
                string value = _commandLine.Argument(2);
                SetValue(key, value);
                break;
        }

        // Named switches
        if (_commandLine.Switch("verbose"))
        {
            _verbose = true;
        }
    }
}
// Toolbar: run with argument "set speed 10 -verbose"
```

### MyIni — INI-Style Configuration

```csharp
MyIni _ini = new MyIni();

void LoadConfig()
{
    _ini.TryParse(Me.CustomData);

    float speed = _ini.Get("movement", "speed").ToSingle(5.0f);
    bool autoStart = _ini.Get("general", "auto_start").ToBoolean(false);
    string mode = _ini.Get("general", "mode").ToString("default");
}

void SaveConfig()
{
    _ini.Set("movement", "speed", 10.0f);
    _ini.Set("general", "auto_start", true);
    Me.CustomData = _ini.ToString();
}
```

### VRageMath — Maths Utilities

Key types available:

| Type | Description |
|------|-------------|
| `Vector3D` | 3D vector (double precision) |
| `Vector3` | 3D vector (single precision) |
| `Vector3I` | 3D vector (integer) |
| `MatrixD` | 4x4 transformation matrix |
| `QuaternionD` | Rotation quaternion |
| `BoundingBoxD` | Axis-aligned bounding box |
| `MathHelper` | Constants, clamp, lerp, angle conversions |

```csharp
// Distance between two points
double distance = Vector3D.Distance(posA, posB);

// Direction from A to B (normalised)
Vector3D direction = Vector3D.Normalize(posB - posA);

// Dot product (how aligned two vectors are: 1 = same, 0 = perpendicular, -1 = opposite)
double alignment = Vector3D.Dot(forward, targetDir);

// Angle between vectors
double angle = Math.Acos(Vector3D.Dot(
    Vector3D.Normalize(vecA),
    Vector3D.Normalize(vecB)));

// Degrees to radians and back
float rad = MathHelper.ToRadians(90f);
float deg = MathHelper.ToDegrees(MathHelper.Pi);
```

---

## Sandbox Limitations

### Whitelisted Namespaces

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

### Hard Limits

| Limit | Value |
|-------|-------|
| Script size | 100,000 characters |
| Instructions per run | 50,000 significant junctions |
| Call chain depth | Limited (exact value varies) |
| Tick budget (all scripts) | 16.67ms total per game tick |

### What Counts as an "Instruction"

Entering any of these counts as one instruction:
- Method calls
- Loop iterations
- Conditional branches (`if`, `switch`)
- Exception handlers

This is a **safeguard against infinite loops**, not a performance metric. You can have a low instruction count and still be slow if each instruction is expensive.

---

## Performance Best Practices

The game runs at 60 ticks per second, giving **16.67ms total** for all game logic per tick. Every programmable block shares this budget.

### 1. Cache Block References

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

### 2. Use the Slowest Adequate Update Rate

```csharp
// BAD: Update1 for a status display (60 updates/sec for text nobody reads that fast)
Runtime.UpdateFrequency = UpdateFrequency.Update1;

// GOOD: Update100 is fine for status displays
Runtime.UpdateFrequency = UpdateFrequency.Update100;

// Update1 only for: autopilot, PID controllers, precise manoeuvres
// Update10 for: responsive monitoring, combat scripts
// Update100 for: status displays, inventory management, slow checks
```

### 3. Spread Work Across Ticks

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

### 4. Avoid String Concatenation in Loops

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

### 5. Monitor Your Performance

```csharp
Echo($"Instructions: {Runtime.CurrentInstructionCount}/{Runtime.MaxInstructionCount}");
Echo($"Last run: {Runtime.LastRunTimeMs:F3}ms");
```

---

## Common Patterns

### Toggle Pattern

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

### Multi-Command Pattern

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

### Safe Block Access Pattern

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

### Inventory Summary Pattern

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

---

## Worked Example: Thrust-to-Weight Calculator

A practical script that calculates whether your ship can hover and displays the result on an LCD.

```csharp
// Thrust-to-Weight Ratio Calculator
// Displays on an LCD named "Thrust LCD"
// Requires at least one cockpit/remote control on the grid

List<IMyThrust> _thrusters = new List<IMyThrust>();
List<IMyShipController> _controllers = new List<IMyShipController>();
IMyTextPanel _lcd;

public Program()
{
    Runtime.UpdateFrequency = UpdateFrequency.Update100;

    GridTerminalSystem.GetBlocksOfType(_thrusters);
    GridTerminalSystem.GetBlocksOfType(_controllers);
    _lcd = GridTerminalSystem.GetBlockWithName("Thrust LCD") as IMyTextPanel;

    if (_lcd != null)
        _lcd.ContentType = ContentType.TEXT_AND_IMAGE;
}

public void Main(string argument, UpdateType updateSource)
{
    if (_controllers.Count == 0)
    {
        Echo("ERROR: No ship controller found!");
        return;
    }

    IMyShipController controller = _controllers[0];

    // Get ship mass
    MyShipMass mass = controller.CalculateShipMass();

    // Get gravity
    Vector3D gravity = controller.GetNaturalGravity();
    double gravityMagnitude = gravity.Length(); // m/s^2

    // Calculate gravitational force (F = m * g)
    double weight = mass.TotalMass * gravityMagnitude; // Newtons

    // Calculate upward thrust
    // We need to figure out which direction is "up" relative to gravity
    Vector3D gravityNorm = Vector3D.Normalize(gravity);

    float totalUpwardThrust = 0;
    float totalUpwardEffectiveThrust = 0;

    foreach (var thruster in _thrusters)
    {
        // Get the thruster's forward direction in world space
        // Thrusters push OPPOSITE to their forward direction
        Vector3D thrustDirection = -thruster.WorldMatrix.Forward;

        // How much of this thruster's force opposes gravity?
        // dot product: 1.0 = directly opposing gravity (pushing up)
        //              0.0 = perpendicular (no contribution)
        //             -1.0 = same direction as gravity (pushing down)
        double alignment = Vector3D.Dot(thrustDirection, -gravityNorm);

        if (alignment > 0) // Only count thrusters that push against gravity
        {
            totalUpwardThrust += thruster.MaxThrust * (float)alignment;
            totalUpwardEffectiveThrust += thruster.MaxEffectiveThrust * (float)alignment;
        }
    }

    // Calculate ratios
    double thrustToWeightMax = weight > 0 ? totalUpwardThrust / weight : 0;
    double thrustToWeightEffective = weight > 0 ? totalUpwardEffectiveThrust / weight : 0;

    // Build display
    StringBuilder sb = new StringBuilder();
    sb.AppendLine("=== Thrust-to-Weight Analysis ===");
    sb.AppendLine();
    sb.AppendLine($"Ship Mass (base):  {mass.BaseMass:N0} kg");
    sb.AppendLine($"Ship Mass (total): {mass.TotalMass:N0} kg");
    sb.AppendLine($"Gravity:           {gravityMagnitude:F2} m/s^2 ({gravityMagnitude / 9.81:F2}g)");
    sb.AppendLine($"Weight Force:      {weight:N0} N");
    sb.AppendLine();
    sb.AppendLine($"Upward Thrust (max):       {totalUpwardThrust:N0} N");
    sb.AppendLine($"Upward Thrust (effective): {totalUpwardEffectiveThrust:N0} N");
    sb.AppendLine();
    sb.AppendLine($"TWR (max):       {thrustToWeightMax:F2}");
    sb.AppendLine($"TWR (effective): {thrustToWeightEffective:F2}");
    sb.AppendLine();

    if (thrustToWeightEffective >= 1.0)
        sb.AppendLine("STATUS: CAN HOVER / TAKE OFF");
    else if (thrustToWeightEffective >= 0.8)
        sb.AppendLine("WARNING: MARGINAL - Reduce mass or add thrusters");
    else if (gravityMagnitude < 0.01)
        sb.AppendLine("STATUS: No significant gravity detected");
    else
        sb.AppendLine("DANGER: CANNOT HOVER - Insufficient thrust!");

    // Display
    if (_lcd != null)
        _lcd.WriteText(sb.ToString());

    Echo(sb.ToString());
}

string BuildProgressBar(float ratio, int width = 20)
{
    ratio = MathHelper.Clamp(ratio, 0f, 1f);
    int filled = (int)(ratio * width);
    return "[" + new string('|', filled) + new string('.', width - filled) + "]";
}
```

**How to use:**
1. Place a Programmable Block, a cockpit/remote control, and an LCD named "Thrust LCD" on your ship
2. Paste this script into the Programmable Block
3. Click Check Code, then Run
4. The LCD updates every ~1.67 seconds with thrust analysis

**What it teaches:**
- Caching blocks in `Program()`
- Using `CalculateShipMass()` and `GetNaturalGravity()`
- Vector maths to determine thruster alignment relative to gravity
- The difference between `MaxThrust` and `MaxEffectiveThrust`
- Building formatted output with `StringBuilder`

---

## External Resources

| Resource | URL | Notes |
|----------|-----|-------|
| Official SE Wiki - Scripting | https://spaceengineers.wiki.gg/wiki/Scripts | Best starting point |
| Malforge API Reference | https://malforge.github.io/spaceengineers/pbapi/ | Current API docs |
| MDK-SE Wiki (archived) | https://github.com/malware-dev/MDK-SE/wiki | Archived Dec 2025, still useful |
| MDK2 (current toolkit) | https://github.com/malforge/mdk2 | VS extension for SE scripting |
| Keen's SE Source | https://github.com/KeenSoftwareHouse/SpaceEngineers | Interface source code |
| SE Mod API Docs | https://keensoftwarehouse.github.io/SpaceEngineersModAPI | Official API reference |
| Steam Workshop Scripts | https://steamcommunity.com/app/244850/workshop/ | Filter by InGameScript |
| Keen Discord | #programmable-block channel | Community help |

---

## Future Additions

As we build things, we'll expand this doc with:

- [ ] Complete block property listings (every property for every interface)
- [ ] PID controller implementation for autopilot
- [ ] Coordinate system transformations (world vs grid vs block space)
- [ ] Orbit/trajectory calculations
- [ ] Automated mining drone logic
- [ ] Airlock sequencing patterns
- [ ] Power management and load balancing
- [ ] Radar/detection systems with cameras
- [ ] Detailed item type IDs for inventory management
