# Utility Classes

[< Back to Index](scripting-reference.md)

---

## MyCommandLine — Argument Parsing

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

---

## MyIni — INI-Style Configuration

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

---

## VRageMath — Maths Utilities

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
