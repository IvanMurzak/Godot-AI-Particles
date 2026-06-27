# CI / CD

Workflows ship **inert** as `.github/workflows/*.yml-sample` so they never run on the template repo.
`commands/init.ps1` / `init.py` activate them (rename `*.yml-sample` → `*.yml`).

| Workflow | Trigger | What it does |
| --- | --- | --- |
| `test.yml` | `workflow_call` (reusable) | **Leg 1 (deterministic, no Godot binary):** build the source-only package, run pure-managed unit tests, `dotnet pack`, assert the nupkg is source-only with no GodotSharp dep, then build the **testbed consuming the local package** (proves the source-injection recipe). **Leg 2 (live):** boot headless Godot, install the core addon, `--build-solutions`, then call each tool via `godot-cli run-tool` (finish wiring the local server). |
| `test-pull-request.yml` | PR to `main`/`dev` | Fans `test.yml` across a multi-Godot-version matrix. |
| `release.yml` | push to `main` | **Version-gated** (package `<Version>` increased AND tag absent) → full test matrix → pack at the real version → publish source-only NuGet → **atomic** GitHub Release (tag + nupkg + notes). |
| `bump_version.yml` | manual dispatch | Runs `bump-version.ps1`, opens a PR bumping `<Version>`. |

## Required secret

- `NUGET_API_KEY` — nuget.org API key with push rights for `com.IvanMurzak.Godot.MCP.*`.
  `gh secret set NUGET_API_KEY --repo <owner>/<repo>`

## Multi-Godot matrix

Each matrix entry sets `godot-version` (the editor for the live E2E) and `godot-sdk-version` (the
`Godot.NET.Sdk`/GodotSharp the testbed compiles against). Keep them equal. `test.yml` rewrites the
testbed's `Godot.NET.Sdk/<x>` line per entry, so one testbed covers every version. Edit the `matrix` in
`test-pull-request.yml` + `release.yml` to the Godot versions you support.

## The version gate

`release.yml` reads the package csproj `<Version>` and publishes only when no tag of that name exists.
Bump with `commands/bump-version.ps1 -NewVersion x.y.z` (or `bump_version.yml`), merge to `main`, and the
release runs once. The testbed's `<PackageReference Version="0.0.0-ci">` is intentionally decoupled (CI
packs the local feed as `0.0.0-ci`), so bumping the release version never touches the testbed.
