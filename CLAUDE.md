# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is an NVIDIA Omniverse Kit SDK application based on the `kit-app-template`. It contains a custom application called "SphereFlake" — a base editor application built on top of the Kit SDK for GPU-accelerated, OpenUSD-based 3D application development. Currently bundled with **Kit SDK 110.1.0**; the bundled version moves forward over time, so the repo is named `sf4ov` (version-neutral) rather than tied to any one Kit release.

## History

The Kit 106.4 lineage of this project lives at [`MikeWise2718/sf4ovc-106-4`](https://github.com/MikeWise2718/sf4ovc-106-4) (tag `v106.4-working`). This repo started from that one's `kit-110` branch. The full upgrade trail is in the old repo's `specs/kit-110-upgrade.md`.

## Build Commands

All commands use `repo.bat` on Windows or `repo.sh` on Linux.

```powershell
# Build the application
.\repo.bat build

# Clean build
.\repo.bat build -c

# Rebuild from scratch
.\repo.bat build -x

# Launch the application
.\repo.bat launch

# Launch a specific app directly (skips interactive picker — required in non-Windows-console shells like Git Bash/MSYS/CI)
.\repo.bat launch --name msft.sphereflake.kit

# Launch with developer tools (Script Editor, Extension Manager, etc.)
.\repo.bat launch -d

# Run tests
.\repo.bat test

# Package for distribution
.\repo.bat package

# Create thin package (custom extensions only)
.\repo.bat package --thin

# Pass arguments directly to Kit executable
.\repo.bat launch -- --clear-cache
```

## Architecture

### Key Files

- **`premake5.lua`**: Build configuration. Kit 110 dropped the per-app `define_app()` model — apps are now picked up directly from `repo.toml`'s precache list.

- **`repo.toml`**: Repository configuration including extension registries, packaging settings, and the `[repo_precache_exts].apps` array listing apps for extension precaching.

- **`source/apps/*.kit`**: Application definition files. These TOML-like files define dependencies (extensions), settings, and metadata. The active app is `msft.sphereflake.kit`.

- **`tools/deps/*.packman.xml`**: Packman dependencies — `kit-sdk.packman.xml` pins the Kit kernel version; `repo-deps.packman.xml` pins the repo tooling versions; `host-deps.packman.xml` pins build-host tools (premake).

### Directory Structure

- `source/apps/` - Application .kit files
- `source/extensions/` - Custom extensions (currently empty)
- `templates/` - Application and extension templates for scaffolding new projects
- `tools/` - Repository-specific tooling and dependencies
- `_build/` - Build output (generated)

### Build Configuration Sync

The .kit file must be listed in:
1. `repo.toml` - `[repo_precache_exts].apps` array
2. `source/apps/` - actual .kit file

(Pre-Kit-110 also required a `define_app()` entry in `premake5.lua`; that's gone now.)

### Creating New Applications/Extensions

```powershell
# Interactive wizard for creating new apps or extensions
.\repo.bat template new

# List available templates
.\repo.bat template list
```

## SphereFlake Extension Setup

The `sphereflake22` extension is located in `D:/ov/exts`. On first install, it must be manually loaded:

1. Launch with the developer bundle to access the Extension Manager:
   ```powershell
   .\repo.bat launch -d
   ```

2. Open **Window → Extensions** to open the Extension Manager

3. Click the gear icon (settings) and add `D:/ov/exts` to the extension search paths

4. Search for `sphereflake22`, then enable and **autoload**

After autoload is enabled, subsequent regular launches (without `-d`) will load it automatically. The benchmark writes one JSON line per generation to `log.txt` in the repo root.

## Important Notes

- First RTX renderer launch takes 5-8 minutes for shader compilation; subsequent launches are faster (~15s)
- Windows: Place repository near drive root to avoid path length issues
- Repository requires NTFS (not exFAT) for symlink/junction support
- C++ development on Windows requires Visual Studio 2019/2022 with "Desktop development with C++" workload

## Status

- Last verified working: 2026-04-26 on Windows 11, Kit SDK 110.1.0. Both `.\repo.bat build` and `.\repo.bat launch --name msft.sphereflake.kit` succeed; sphereflake22 generation completes and appends to `log.txt`. Tagged `v110.0-working`.

## Known issues

- **Cosmetic warning at startup:** `[Error] [carb.dictionary.serializer-toml.plugin] Redefine of existing key /<from TOML string>/dependencies/omni.kit.actions.core` fires at ~5ms during extension search. Root cause unidentified, but harmless — build and launch both complete normally.
- **Cosmetic warning at startup:** `omni.kit.tool.asset_importer-2.10.3+106.4.0/utils.py:82: SyntaxWarning: invalid escape sequence '\.'` — NVIDIA hasn't published a Kit 110 / Python 3.12 rebuild of `omni.kit.tool.asset_importer-2.10.3`; the solver picks up the cached 106.4 build as the only available `2.10.3`. Will resolve when NVIDIA updates the registry.
- **Latent bugs in `sphereflake22`** (pre-existing, not fired in practice): `extension.py` calls `omni.kit.app.get_app()` and `omni.usd.get_context()` without importing those modules — works because other extensions import them transitively first. Also uses deprecated `asyncio.ensure_future()` and `asyncio.get_event_loop()` patterns. Both currently emit DeprecationWarnings under Python 3.12 but don't break.
