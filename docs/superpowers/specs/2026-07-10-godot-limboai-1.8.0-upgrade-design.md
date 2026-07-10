# Godot LimboAI 1.8.0 Upgrade Design

## Goal

Upgrade the `godot-limboai` Scoop manifest from LimboAI 1.7.1 / Godot 4.6.3 to LimboAI 1.8.0 / Godot 4.7, publish the update, and upgrade the locally installed package.

## Source Release

- Release: `v1.8.0`
- Windows asset: `limboai+v1.8.0.godot-4.7.editor.windows.x86_64.zip`
- Composite Scoop version: `1.8.0-godot-4.7`
- SHA-256: `1f2a0069b9b042d0aba135a829c84e5030d151b8cd4986773bf45f3f72f414ae`

The asset name and digest are supplied by the official GitHub release API.

## Approach

Run Scoop's `checkver.ps1 godot-limboai .\bucket -u` through PowerShell Core. This exercises the existing `checkver` and `autoupdate` contract rather than duplicating it with manual edits.

The updater must change only the manifest's current version, download URL, and hash. Existing executable renaming, shim, shortcut, persistence, `checkver`, and `autoupdate` definitions remain unchanged.

## Verification

1. Parse the resulting manifest as JSON.
2. Run `checkver.ps1` and confirm `1.8.0-godot-4.7`.
3. Run forced autoupdate and confirm it is idempotent and resolves the expected GitHub digest.
4. Commit and push the verified manifest to `origin/main`.
5. Update the registered `godot-limboai` bucket.
6. Upgrade the installed package through Scoop.
7. Run `godot-limboai --version`; it must report Godot 4.7 with LimboAI 1.8.0.

## Failure Handling

Do not publish if the release asset, digest, JSON manifest, checkver result, or installation verification differs from the expected values. Retain the previously installed version until the new manifest has passed repository-level verification.
