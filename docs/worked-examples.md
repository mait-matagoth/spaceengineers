# Worked Examples

[< Back to Index](scripting-reference.md)

---

## Thrust-to-Weight Calculator

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

### How to use

1. Place a Programmable Block, a cockpit/remote control, and an LCD named "Thrust LCD" on your ship
2. Paste this script into the Programmable Block
3. Click Check Code, then Run
4. The LCD updates every ~1.67 seconds with thrust analysis

### What it teaches

- Caching blocks in `Program()`
- Using `CalculateShipMass()` and `GetNaturalGravity()`
- Vector maths to determine thruster alignment relative to gravity
- The difference between `MaxThrust` and `MaxEffectiveThrust`
- Building formatted output with `StringBuilder`
