# Block Interfaces Reference

[< Back to Index](scripting-reference.md)

---

All in-game blocks implement interfaces from `Sandbox.ModAPI.Ingame`. Every block is at minimum an `IMyTerminalBlock`. The hierarchy is:

```
IMyTerminalBlock          (base: name, custom data, actions)
  └── IMyFunctionalBlock  (adds: Enabled on/off)
        └── [specific block interfaces]
```

---

## Base Interfaces

### IMyTerminalBlock

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

### IMyFunctionalBlock (extends IMyTerminalBlock)

| Property | Type | Description |
|----------|------|-------------|
| `Enabled` | `bool` | Turn block on/off |

---

## Movement & Control

### IMyThrust

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

### IMyGyro

Controls gyroscopes for rotation.

| Property | Type | Description |
|----------|------|-------------|
| `GyroOverride` | `bool` | Enable manual override |
| `Yaw` | `float` | Yaw rotation speed (rad/s) |
| `Pitch` | `float` | Pitch rotation speed (rad/s) |
| `Roll` | `float` | Roll rotation speed (rad/s) |
| `GyroPower` | `float` | Power multiplier (0.0 to 1.0) |

**Note:** Override values are in the gyro's local coordinate space, not the ship's. You'll need to transform between coordinate systems for precise control.

### IMyShipController (cockpits, remote controls, etc.)

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

### IMyLandingGear

| Property | Type | Description |
|----------|------|-------------|
| `IsLocked` | `bool` | Currently locked to a surface |
| `AutoLock` | `bool` | Auto-lock when near surface |
| `LockMode` | `LandingGearMode` | `Unlocked`, `ReadyToLock`, `Locked` |
| `Lock()` | `void` | Attempt to lock |
| `Unlock()` | `void` | Unlock |

---

## Power & Energy

### IMyBatteryBlock

| Property | Type | Description |
|----------|------|-------------|
| `ChargeMode` | `ChargeMode` | `Auto`, `Recharge`, `Discharge` |
| `CurrentStoredPower` | `float` | Current charge (MWh) |
| `MaxStoredPower` | `float` | Maximum capacity (MWh) |
| `CurrentInput` | `float` | Power being drawn (MW) |
| `CurrentOutput` | `float` | Power being supplied (MW) |
| `IsCharging` | `bool` | Currently charging |

### IMyReactor

| Property | Type | Description |
|----------|------|-------------|
| `CurrentOutput` | `float` | Current power output (MW) |
| `MaxOutput` | `float` | Maximum power output (MW) |

### IMySolarPanel / IMyWindTurbine

| Property | Type | Description |
|----------|------|-------------|
| `CurrentOutput` | `float` | Current power output (MW) |
| `MaxOutput` | `float` | Maximum power output (MW) |

### IMyPowerProducer (category interface)

All power-producing blocks implement this. Use with `GetBlocksOfType<IMyPowerProducer>()` to get all generators regardless of type.

---

## Storage & Production

### IMyCargoContainer

Primarily accessed through the `IMyInventory` interface:

```csharp
IMyCargoContainer cargo = /* get block */;
IMyInventory inv = cargo.GetInventory(0);
float currentVolume = (float)inv.CurrentVolume;
float maxVolume = (float)inv.MaxVolume;
List<MyInventoryItem> items = new List<MyInventoryItem>();
inv.GetItems(items);
```

### IMyInventory

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

### IMyAssembler

| Property | Type | Description |
|----------|------|-------------|
| `Mode` | `MyAssemblerMode` | `Assembly` or `Disassembly` |
| `IsQueueEmpty` | `bool` | Production queue empty |
| `IsProducing` | `bool` | Currently producing |
| `CooperativeMode` | `bool` | Share work with other assemblers |
| `AddQueueItem(MyDefinitionId, decimal)` | `void` | Add item to production queue |
| `GetQueue(List<MyProductionItem>)` | `void` | Get current queue |
| `ClearQueue()` | `void` | Clear production queue |

### IMyRefinery

| Property | Type | Description |
|----------|------|-------------|
| `IsQueueEmpty` | `bool` | Processing queue empty |
| `IsProducing` | `bool` | Currently processing |

---

## Display & Communication

### IMyTextPanel / IMyTextSurface

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

### IMyRadioAntenna

| Property | Type | Description |
|----------|------|-------------|
| `Radius` | `float` | Broadcast range (m) |
| `EnableBroadcasting` | `bool` | Broadcasting on/off |
| `HudText` | `string` | Text shown on HUD |

### IMyBeacon

| Property | Type | Description |
|----------|------|-------------|
| `Radius` | `float` | Broadcast range (m) |
| `HudText` | `string` | Text shown on HUD |

### IMyLaserAntenna

| Property | Type | Description |
|----------|------|-------------|
| `Range` | `float` | Maximum range (m) |
| `Status` | `MyLaserAntennaStatus` | Connection status |

---

## Doors & Access

### IMyDoor

| Property | Type | Description |
|----------|------|-------------|
| `Status` | `DoorStatus` | `Open`, `Closed`, `Opening`, `Closing` |
| `OpenRatio` | `float` | How open (0.0 to 1.0) |
| `OpenDoor()` | `void` | Open the door |
| `CloseDoor()` | `void` | Close the door |
| `ToggleDoor()` | `void` | Toggle open/closed |

---

## Weapons & Defence

### IMyLargeTurretBase

| Property | Type | Description |
|----------|------|-------------|
| `Range` | `float` | Targeting range |
| `IsAimed` | `bool` | Currently aimed at target |
| `HasTarget` | `bool` | Has a valid target |
| `Elevation` | `float` | Vertical angle (rad) |
| `Azimuth` | `float` | Horizontal angle (rad) |

### IMyWarhead

| Property | Type | Description |
|----------|------|-------------|
| `IsCountingDown` | `bool` | Timer active |
| `DetonationTime` | `float` | Countdown time (seconds) |
| `IsArmed` | `bool` | Armed state |
| `Detonate()` | `void` | Immediate detonation |
| `StartCountdown()` | `void` | Start countdown |
| `StopCountdown()` | `void` | Stop countdown |

---

## Mechanical Blocks

### IMyMotorStator (rotors)

| Property | Type | Description |
|----------|------|-------------|
| `Angle` | `float` | Current angle (rad) |
| `TargetVelocityRPM` | `float` | Target rotation speed |
| `Torque` | `float` | Applied torque |
| `UpperLimitRad` | `float` | Upper rotation limit |
| `LowerLimitRad` | `float` | Lower rotation limit |
| `IsAttached` | `bool` | Rotor head attached |

### IMyPistonBase

| Property | Type | Description |
|----------|------|-------------|
| `CurrentPosition` | `float` | Current extension (m) |
| `MinLimit` | `float` | Minimum extension |
| `MaxLimit` | `float` | Maximum extension |
| `Velocity` | `float` | Extension speed (m/s, negative = retract) |
| `Extend()` | `void` | Start extending |
| `Retract()` | `void` | Start retracting |

---

## Sensors & Detection

### IMySensorBlock

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

### IMyCameraBlock

| Property | Type | Description |
|----------|------|-------------|
| `CanScan(double)` | `bool` | Can raycast at distance |
| `Raycast(double, float, float)` | `MyDetectedEntityInfo` | Perform raycast |
| `AvailableScanRange` | `double` | Maximum scan range (charges over time) |

---

## Connectors & Merge Blocks

### IMyShipConnector

| Property | Type | Description |
|----------|------|-------------|
| `Status` | `MyShipConnectorStatus` | `Unconnected`, `Connectable`, `Connected` |
| `OtherConnector` | `IMyShipConnector` | The connected block |
| `Connect()` | `void` | Attempt connection |
| `Disconnect()` | `void` | Disconnect |
| `ToggleConnect()` | `void` | Toggle state |
| `IsParkingEnabled` | `bool` | Parking mode |
