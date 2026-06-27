# CLAUDE.md — Godot-AI-Particles

First real **Godot-MCP extension**: an MCP tool family for Godot's built-in `GpuParticles2D` /
`GpuParticles3D` nodes, shipped as a **source-only NuGet package** (`com.IvanMurzak.Godot.MCP.Particles`)
that compiles inside a consumer's Godot project against the consumer's own GodotSharp. Created from
[`Godot-AI-Tools-Template`](https://github.com/IvanMurzak/Godot-AI-Tools-Template). The packaging recipe
is the load-bearing detail — read `docs/source-only-nuget-recipe.md`.

## Layout

- `src/Godot-AI-Particles/` — the source-only package (`Godot.NET.Sdk`).
  - `Runtime/Tools/Tool_Particles.cs` — the `[AiToolType]` family (one partial class).
  - `Runtime/Tools/Tool_Particles.Ids.cs` — all tool-id consts (pure-managed; pinned by tests).
  - `Runtime/Tools/Tool_Particles.Defaults.cs` — `particles-defaults` (pure-managed tool).
  - `Runtime/Particles/` — pure-managed support types: `ParticlesDimension`, `ParticlesNodeInfo`,
    `ParticlesDefaults` (starter config + value-clamping rules, all unit-tested).
  - `Editor/Tools/Tool_Particles.{Editor,Create,Configure,SetEmitting,Get}.cs` — editor tools behind
    `#if TOOLS` (touch `EditorInterface`/live nodes; main-thread-marshalled; E2E-verified).
  - `build/com.IvanMurzak.Godot.MCP.Particles.props` — the source-injection props (auto-imported by NuGet
    in the consumer; MUST stay named `<PackageId>.props`).
- `tests/Godot-AI-Particles.Tests/` — xUnit specs for the pure-managed sources only (no Godot binary).
- `testbed/Particles-Testbed.csproj` — a consumer `Godot.NET.Sdk` project that restores the local-packed
  package; `dotnet build` of it is the source-injection proof.

## Tools

| Tool | Kind | File |
| --- | --- | --- |
| `particles-defaults` | pure-managed | `Runtime/Tools/Tool_Particles.Defaults.cs` |
| `particles-create` | editor | `Editor/Tools/Tool_Particles.Create.cs` |
| `particles-configure` | editor | `Editor/Tools/Tool_Particles.Configure.cs` |
| `particles-set-emitting` | editor | `Editor/Tools/Tool_Particles.SetEmitting.cs` |
| `particles-get` | editor | `Editor/Tools/Tool_Particles.Get.cs` |

## Build / test (no Godot binary)

```bash
dotnet build src/Godot-AI-Particles/Godot-AI-Particles.csproj   # source-only package compiles tools
dotnet test  tests/Godot-AI-Particles.Tests/Godot-AI-Particles.Tests.csproj
dotnet pack  src/Godot-AI-Particles/Godot-AI-Particles.csproj -p:Version=0.0.0-ci -o local-nuget
dotnet build testbed/Particles-Testbed.csproj                  # consumes the local package (injection proof)
```

`Godot.NET.Sdk` supplies GodotSharp from NuGet, so no Godot install is needed to build/test/pack or to
prove the source-injection recipe (the testbed build is a faithful proxy for `godot --build-solutions`).
The recipe is verified to compile into the consumer across **Godot 4.3 and 4.5** (the CI matrix). When
proving locally, note `dotnet pack` re-uses the **global NuGet cache** for an already-cached version: if
you re-pack the same `Version`, clear `~/.nuget/packages/com.ivanmurzak.godot.mcp.particles/<ver>` (or pack
a unique version) before re-restoring the testbed, or you'll silently build the stale cached source.

## Conventions

- Root namespace `com.IvanMurzak.Godot.MCP.Particles`. Every `.cs` starts with the Apache-2.0 header.
- Pure-managed tools (no Godot native API) → `Runtime/` (outside `#if TOOLS`, unit-testable); editor-driving
  tools → `Editor/` (behind `#if TOOLS`, every Godot call via `MainThread.Instance.Run(...)`, E2E-verified).
- The package declares ONLY the `com.IvanMurzak.McpPlugin` / `com.IvanMurzak.ReflectorNet` min-version
  deps; **GodotSharp must never become a package dependency** (CI asserts the nuspec). Keep the MCP pins in
  lockstep with the core Godot-MCP addon; bump with `commands/update-core.ps1`.
- One `[AiToolType] partial class Tool_Particles`; one `[AiTool]` method per partial-class file. New
  pure-managed sources must be added to the test csproj `<Compile Include>` list to be unit-tested.

## Find detail in

- `docs/source-only-nuget-recipe.md` — the packaging recipe (the centerpiece) + the consumer story.
- `docs/ci.md` — workflows, the version gate, multi-Godot matrix, the `NUGET_API_KEY` secret.
