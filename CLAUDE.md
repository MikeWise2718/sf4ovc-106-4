# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is an NVIDIA Omniverse Kit SDK application based on the `kit-app-template`. It contains a custom application called "SphereFlake 106.4" - a base editor application built on top of the Kit SDK for GPU-accelerated, OpenUSD-based 3D application development.

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
.\repo.bat launch --name msft.sphereflake1064.kit

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

- **`premake5.lua`**: Build configuration defining which apps to build via `define_app()`. Current apps:
  - `msft.sphereflake1064.kit`
  - `msft.sphereflake1064_streaming.kit`

- **`repo.toml`**: Repository configuration including extension registries, packaging settings, and the `[repo_precache_exts].apps` array listing apps for extension precaching.

- **`source/apps/*.kit`**: Application definition files. These TOML-like files define dependencies (extensions), settings, and metadata.

### Directory Structure

- `source/apps/` - Application .kit files
- `source/extensions/` - Custom extensions (currently empty)
- `templates/` - Application and extension templates for scaffolding new projects
- `tools/` - Repository-specific tooling and dependencies
- `_build/` - Build output (generated)

### Build Configuration Sync

The same `.kit` files must be listed in three places:
1. `premake5.lua` - `define_app()` calls
2. `repo.toml` - `[repo_precache_exts].apps` array
3. `source/apps/` - actual .kit files

### Creating New Applications/Extensions

```powershell
# Interactive wizard for creating new apps or extensions
.\repo.bat template new

# List available templates
.\repo.bat template list
```

## SphereFlake Extension Setup

The `sphereflake22` extension is located in `D:/ov/exts` and must be manually loaded:

1. Launch with the developer bundle to access the Extension Manager:
   ```powershell
   .\repo.bat launch -d
   ```

2. Open **Window → Extensions** to open the Extension Manager

3. Click the gear icon (settings) and add `D:/ov/exts` to the extension search paths

4. Search for `sphereflake22`, then enable and activate it

## Important Notes

- First RTX renderer launch takes 5-8 minutes for shader compilation; subsequent launches are faster
- Windows: Place repository near drive root to avoid path length issues
- Repository requires NTFS (not exFAT) for symlink/junction support
- C++ development on Windows requires Visual Studio 2019/2022 with "Desktop development with C++" workload

## Status

- Last verified working: 2026-04-20 on Windows 11, Kit SDK 106.4. Both `.\repo.bat build` and `.\repo.bat launch --name msft.sphereflake1064.kit` succeed; app reaches RTX renderer startup and runs.
