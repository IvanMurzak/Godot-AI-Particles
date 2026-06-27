# Particles Tools

AI MCP tools for Godot **GpuParticles** (2D & 3D), for [Godot-MCP / AI Game Developer](https://github.com/IvanMurzak/Godot-MCP).

A **source-only** MCP tool extension: the package ships C# source (no compiled DLL, no bundled Godot)
that compiles inside your Godot project against your own GodotSharp, so it never locks you to a Godot
version. Verified to compile into a consumer project across Godot 4.3 … 4.5.

## Install

Requires the core [`godot_mcp`](https://github.com/IvanMurzak/Godot-MCP) addon in your Godot C# project.

```bash
# via the godot-cli (resolves from the shared catalog, edits your .csproj, rebuilds)
godot-cli install-extension com.IvanMurzak.Godot.MCP.Particles

# …or add the reference manually and rebuild:
#   <PackageReference Include="com.IvanMurzak.Godot.MCP.Particles" Version="0.1.0" />
```

…or pick it from the **Extensions** dock inside the Godot editor.

After a rebuild, the extension's `[AiToolType]` tool family is auto-discovered — no registry edit.

## Tools

| Tool | Kind | Description |
| --- | --- | --- |
| `particles-defaults` | pure-managed | Return the recommended starter config (amount, lifetime, …) for a 2D/3D emitter. |
| `particles-create` | editor | Create a `GpuParticles2D`/`GpuParticles3D` node in the edited scene; seed `amount`/`lifetime`. |
| `particles-configure` | editor | Update an emitter's scalar properties (amount, lifetime, one-shot, explosiveness, randomness, speed scale, preprocess, local coords); each clamped to a valid range. |
| `particles-set-emitting` | editor | Start/stop emission (`emitting`), optionally restarting first. |
| `particles-get` | editor | Read an emitter's scalar config (read-only). |

All editor tools marshal every Godot call onto the editor main thread; values are clamped to valid
ranges before they touch a node.

License: Apache-2.0.
