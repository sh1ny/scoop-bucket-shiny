# Repository Guidelines

## Project Overview

This Scoop bucket distributes stable Windows x86_64 releases of Godot bundled with LimboAI and the Flow Control text editor. It packages upstream archives; it does not build either application.

## Architecture & Data Flow

`bucket/godot-limboai.json` packages the bundled Godot editor:

- **Release discovery:** `checkver` reads GitHub's latest LimboAI release assets, captures both component versions, and produces `<limbo>-godot-<godot>`.
- **Manifest updates:** `autoupdate` interpolates `$matchLimbo` and `$matchGodot` into the Windows editor URL. Its hash lookup selects the exact asset's `digest` from the release-tag API.
- **Installation:** Scoop downloads and verifies the archive. Inline `pre_install` PowerShell renames the console and GUI executables to `godot.console.exe` and `godot.exe`, then creates `._sc_`.
- **User integration:** `bin` exposes the `godot-limboai` command; `shortcuts` creates `Godot LimboAI`; `persist` preserves `editor_data` across upgrades.

The release tag contains only the LimboAI version; the asset filename contains both LimboAI and Godot versions. Keep discovery, download URL, checksum, and exact-name digest selection aligned to the same asset.

`bucket/flow.json` uses GitHub's latest stable release, the Windows x86_64 ZIP, and its `.zip.sha256` sidecar. The archive contains `flow.exe` and `flow-gui.exe` at its root. Scoop exposes the `flow` command and a `Flow Control` GUI shortcut; no install hooks or additional runtime packages are needed. Flow stores configuration and state outside the installation directory, so the manifest does not declare `persist`.

## Key Directories

- `bucket/`: `godot-limboai.json` and `flow.json`.
- `docs/plans/`: implementation plans; plans are not verification receipts.
- `docs/superpowers/specs/`: maintenance design notes. These describe intended procedures, not completed verification.

There are no application source, test, or standalone script directories.

## Development Commands

Run from the repository root. There are no repository-defined build, test, or lint commands.

```powershell
# Validate JSON syntax (does not validate the Scoop schema or download).
Get-ChildItem .\bucket\*.json | ForEach-Object {
    Get-Content -Raw $_.FullName | ConvertFrom-Json | Out-Null
}

# Discover and apply an upstream update; this MODIFIES the manifest.
# Run in PowerShell Core with Scoop's checkver.ps1 available.
checkver.ps1 godot-limboai .\bucket -u
checkver.ps1 flow .\bucket -u

# After installation or upgrade, verify the packaged versions.
godot-limboai --version
flow --version
```

`checkver.ps1` is external Scoop tooling, not a checked-in script. Locate it in the local Scoop tooling installation rather than assuming it is on `PATH`.

## Code Conventions & Common Patterns

- Preserve the existing four-space JSON indentation and field ordering. Use valid JSON without comments or trailing commas.
- Package identifiers use lowercase hyphenated names. LimboAI uses `<limbo>-godot-<godot>` versions; Flow uses its upstream version without the `v` prefix.
- Keep install hooks as PowerShell strings in `pre_install`. Preserve JSON escaping in regexes and the distinction between `${limbo}`/`${godot}` replacement captures and `$matchLimbo`/`$matchGodot` update variables.
- Routine version bumps should change only `version`, the current download `url`, and `hash`. Preserve install hooks, shim, shortcut, persistence, and update definitions unless the task requires changing them.
- The install hook selects the first recursive console/non-console executable match and guards renaming with `if`; it does not explicitly fail when a binary is missing. Check archive layout and installed paths rather than treating hook completion as success.
- LimboAI uses Scoop's `persist: "editor_data"`. Flow manages its own user-profile configuration and state; do not redirect these into the package directory.

## Important Files

- `bucket/godot-limboai.json`: bundled editor metadata, install hooks, persistence, and update templates.
- `bucket/flow.json`: Flow metadata, command and GUI integration, and checksum-sidecar update template.
- `docs/superpowers/specs/2026-07-10-godot-limboai-1.8.0-upgrade-design.md`: historical upgrade design. Its target version is not the current manifest baseline; do not treat it as an execution receipt.

## Runtime/Tooling Preferences

Use Windows and Scoop for installation checks. The documented updater workflow uses PowerShell Core (`pwsh`); no exact runtime version is pinned. This repository has no Node/Bun dependency manifest, JavaScript package manager, compiler configuration, or CI workflow. Do not introduce application build tooling for manifest-only maintenance.

## Testing & QA

There is no automated test framework or coverage requirement. Validate package behavior instead:

1. Parse the JSON, then confirm the official release asset and SHA-256 match the manifest.
2. Exercise `checkver` for both packages: LimboAI's composite version and Flow's stable upstream version. Force autoupdate with the installed tooling's supported options; confirm the expected digest or sidecar checksum and unchanged manifest values.
3. Before replacing an installed version, complete repository-level verification. For an authorized rollout, refresh the registered bucket and install or upgrade through Scoop.
4. Run `godot-limboai --version` and check both component versions. Check the command shim, GUI shortcut, expected executable paths, and persistence of `editor_data`.
5. Run `flow --version`, open a file in its terminal and GUI interfaces, and verify editing and saving. Check the `Flow Control` shortcut. Use isolated configuration and state directories for smoke checks; do not modify the user's editor settings.

Do not publish a manifest with mismatched release metadata or failed verification. Retain the previous installed version until repository-level checks pass. Report which checks actually ran; JSON parsing alone does not prove update or installation behavior.
