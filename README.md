# paket-basic

## Probe metadata

| Field | Value |
|---|---|
| Pattern | paket-basic |
| Target framework | net8.0 |
| NuGet source | https://api.nuget.org/v3/index.json |
| Storage mode | none |
| Resolution strategy | max (default) |
| Dependency groups | Main only (no named groups) |
| Generated | 2026-04-22 |

## Feature exercised

This probe exercises the simplest possible Paket project structure: a single
`paket.dependencies` file with one unnamed (Main) group, four direct NuGet
dependencies, and a `paket.lock` that captures the fully resolved dependency
graph including transitive packages. It validates that Mend's Unified Agent
can parse `paket.lock` and correctly surface both direct and transitive
NuGet dependencies with accurate versions.

## Expected dependency tree

### Direct dependencies (Main group)

| Package | Resolved version | Has transitives |
|---|---|---|
| Newtonsoft.Json | 13.0.3 | No |
| Serilog | 3.1.1 | No |
| Microsoft.Extensions.Logging | 8.0.0 | Yes |
| AutoMapper | 12.0.1 | Yes |

### Transitive dependencies (resolved by lockfile)

| Package | Resolved version | Required by |
|---|---|---|
| Microsoft.Extensions.DependencyInjection.Abstractions | 8.0.1 | Microsoft.Extensions.Logging, Microsoft.Extensions.Logging.Abstractions, Microsoft.Extensions.Options, AutoMapper |
| Microsoft.Extensions.Logging.Abstractions | 8.0.0 | Microsoft.Extensions.Logging |
| Microsoft.Extensions.Options | 8.0.0 | Microsoft.Extensions.Logging |
| Microsoft.Extensions.Primitives | 8.0.0 | Microsoft.Extensions.Options |

### Detection expectations

- Mend should detect **8 unique packages** in total from `paket.lock`.
- All packages should be attributed to the `Main` (default) group.
- Source should be recorded as `https://api.nuget.org/v3/index.json`.
- `storage: none` must not prevent detection — Mend reads `paket.lock`, not
  the local packages folder.
- Transitive packages must appear as children of their direct-dependency
  parents in the dependency tree, not as top-level entries.

## File structure

```
paket-basic/
├── paket.dependencies       # Single Main group, 4 direct deps
├── paket.lock               # Fully resolved lockfile (8 packages)
├── expected-tree.json       # Expected Mend dependency tree (probe format)
├── README.md                # This file
└── src/
    └── MyProject/
        ├── MyProject.csproj # SDK-style net8.0 project
        └── paket.references # 4 direct package references
```