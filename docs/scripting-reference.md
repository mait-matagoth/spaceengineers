# Space Engineers Programmable Block - Scripting Reference

> A comprehensive guide to writing C# scripts for the Space Engineers Programmable Block.
> This is a living document - we'll expand it as we build more things.

---

## Quick Navigation

| Guide | What's Inside |
|-------|---------------|
| [Getting Started](getting-started.md) | Prerequisites, script anatomy, built-in properties, MDK setup |
| [Grid Terminal System](grid-terminal-system.md) | Finding blocks, filtering, groups, best practices |
| [Block Interfaces](block-interfaces.md) | Full API reference for every block type |
| [Runtime & Continuous Scripts](runtime-and-continuous-scripts.md) | Update frequencies, state machines, continuous execution |
| [LCD & Text Surfaces](lcd-and-text-surfaces.md) | Writing to screens, cockpit displays, progress bars |
| [Storage & Persistence](storage-and-persistence.md) | `Storage`, `CustomData`, `MyIni` configuration |
| [Intergrid Communication](intergrid-communication.md) | IGC broadcasting, unicast messaging between grids |
| [Utility Classes](utility-classes.md) | `MyCommandLine`, `MyIni`, `VRageMath` |
| [Sandbox Limitations](sandbox-limitations.md) | Whitelisted namespaces, hard limits, instruction counting |
| [Performance Best Practices](performance-best-practices.md) | Caching, update rates, spreading work, StringBuilder |
| [Common Patterns](common-patterns.md) | Toggle, multi-command, safe block access, inventory summary |
| [Worked Examples](worked-examples.md) | Thrust-to-weight calculator with full walkthrough |

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

As we build things, we'll expand these docs with:

- [ ] Complete block property listings (every property for every interface)
- [ ] PID controller implementation for autopilot
- [ ] Coordinate system transformations (world vs grid vs block space)
- [ ] Orbit/trajectory calculations
- [ ] Automated mining drone logic
- [ ] Airlock sequencing patterns
- [ ] Power management and load balancing
- [ ] Radar/detection systems with cameras
- [ ] Detailed item type IDs for inventory management
