# Intergrid Communication (IGC)

[< Back to Index](scripting-reference.md)

---

Send messages between programmable blocks on different grids via antennas.

## Broadcasting (one-to-many)

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

---

## Unicast (one-to-one)

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
