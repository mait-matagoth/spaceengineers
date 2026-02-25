# Storage & Persistence

[< Back to Index](scripting-reference.md)

---

The `Storage` property is a single string that persists across game saves and script recompilations.

## Simple State Persistence

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

---

## Using CustomData for Block-Level Storage

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
