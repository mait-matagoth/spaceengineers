# LCD & Text Surfaces

[< Back to Index](scripting-reference.md)

---

## Writing to a Standalone LCD

```csharp
IMyTextPanel lcd = GridTerminalSystem.GetBlockWithName("Status LCD") as IMyTextPanel;
lcd.ContentType = ContentType.TEXT_AND_IMAGE;
lcd.FontSize = 1.2f;
lcd.FontColor = Color.Green;
lcd.WriteText("Systems Nominal\n");
lcd.WriteText($"Speed: {speed:F1} m/s\n", true); // true = append
```

---

## Writing to the Programmable Block's Own Screen

```csharp
IMyTextSurface surface = Me.GetSurface(0);
surface.ContentType = ContentType.TEXT_AND_IMAGE;
surface.WriteText("Script Running...");
```

---

## Writing to Cockpit Screens

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

---

## Building Progress Bars

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
