<h1 align="center">Godot AI Particles</h1>

<p align="center">
  AI <b>MCP tools</b> for Godot <b>GpuParticles</b> (2D &amp; 3D) — an extension for
  <a href="https://github.com/IvanMurzak/Godot-MCP">Godot-MCP / AI Game Developer</a>.
</p>

`Godot-AI-Particles` is the first real Godot-MCP extension. It adds a focused MCP tool family for
Godot's built-in `GpuParticles2D` / `GpuParticles3D` nodes, authored in C# with `[AiToolType]` /
`[AiTool]` (the same model as Unity-MCP and the core Godot-MCP addon), and shipped as a **source-only
NuGet package** that compiles inside any consumer's Godot project against the consumer's own GodotSharp
— no bundled Godot, no version lock. Created from
[`Godot-AI-Tools-Template`](https://github.com/IvanMurzak/Godot-AI-Tools-Template).

## Tools

| Tool | Kind | Description |
| --- | --- | --- |
| `particles-defaults` | pure-managed | Return the recommended starter config (amount, lifetime, explosiveness, …) for a 2D/3D emitter. |
| `particles-create` | editor (`#if TOOLS`) | Create a `GpuParticles2D`/`GpuParticles3D` node in the edited scene; optional parent, name, `amount`, `lifetime`. |
| `particles-configure` | editor (`#if TOOLS`) | Update an emitter's scalar properties; each clamped to a valid range. |
| `particles-set-emitting` | editor (`#if TOOLS`) | Start/stop emission, optionally restarting first. |
| `particles-get` | editor (`#if TOOLS`) | Read an emitter's scalar config (read-only). |

Pure-managed tools (no Godot native API) live under `src/Godot-AI-Particles/Runtime/` and are
CI-unit-tested; editor-driving tools live under `Editor/` behind `#if TOOLS` and marshal every Godot
call onto the editor main thread via `MainThread.Instance.Run(...)`.

## Install (in a consumer Godot project)

Requires the core [`godot_mcp`](https://github.com/IvanMurzak/Godot-MCP) addon. Then either:

- **Extensions dock** — pick it inside the Godot editor (Install → adds the `<PackageReference>` → rebuild).
- **CLI** — `godot-cli install-extension com.IvanMurzak.Godot.MCP.Particles`.
- **By hand** — add `<PackageReference Include="com.IvanMurzak.Godot.MCP.Particles" Version="x.y.z" />`
  to the consumer `.csproj` and rebuild.

After a rebuild the `[AiToolType]` tool family is auto-discovered — no registry edit.

## Build & test (no Godot binary needed)

`Godot.NET.Sdk` pulls GodotSharp from NuGet, so the package builds and unit-tests headless:

```bash
dotnet build src/Godot-AI-Particles/Godot-AI-Particles.csproj            # compiles tools (Godot API resolves)
dotnet test  tests/Godot-AI-Particles.Tests/Godot-AI-Particles.Tests.csproj   # pure-managed unit tests
dotnet pack  src/Godot-AI-Particles/Godot-AI-Particles.csproj -p:Version=0.0.0-ci -o local-nuget
dotnet build testbed/Particles-Testbed.csproj                            # consumer build = source-injection proof
```

The testbed build proves the source-injection recipe: the package's `.cs` are injected as `<Compile>`
items into the consumer and compile against the consumer's own GodotSharp. CI runs this across a
multi-Godot-version matrix; an end-to-end leg additionally boots real headless Godot, installs the core
addon, and (once a local MCP server is wired) calls each tool via `godot-cli run-tool`.

## Docs

- `docs/source-only-nuget-recipe.md` — the packaging recipe (the centerpiece).
- `docs/ci.md` — workflows, the version gate, the multi-Godot matrix, required secrets.
- `CLAUDE.md` — maintainer notes.

## Publish

Source-only, version-gated release (see `docs/ci.md`): set the `NUGET_API_KEY` secret, bump
`<Version>` (`commands/bump-version.ps1 -NewVersion x.y.z`), merge to `main`; `release.yml` runs the
full matrix, publishes the package to NuGet, and cuts an atomic GitHub Release.

License: **Apache-2.0**.
