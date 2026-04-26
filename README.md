# SphereFlake

A GPU-accelerated benchmarking application built on the NVIDIA Omniverse Kit SDK. SphereFlake is designed for benchmarking Omniverse applications across various platforms, providing a consistent workload for performance testing and comparison.

**Status:** Last verified working on 2026-04-26 (Windows 11, Kit SDK 110.1.0, build `110.1.0+feature.293547.8e364693.gl`). `.\repo.bat build` and `.\repo.bat launch --name msft.sphereflake.kit` both succeed; sphereflake22 generation logs to `log.txt`. Tagged `v110.0-working`.

> The Kit 106.4 lineage lives at [`MikeWise2718/sf4ovc-106-4`](https://github.com/MikeWise2718/sf4ovc-106-4) (tag `v106.4-working`). This repo took the `kit-110` branch from there as its starting point — see that repo's `specs/kit-110-upgrade.md` for the upgrade trail.

## Overview

SphereFlake is a base editor application that leverages the Omniverse Kit SDK for OpenUSD-based 3D rendering. It includes:

- **Desktop Application** (`msft.sphereflake.kit`): Full-featured editor with viewport, stage hierarchy, property panels, and content browser

> A streaming variant existed in the 106.4 lineage but was dropped ahead of the Kit 110 upgrade. It will be reintroduced cleanly using 110's `streaming_configs/` templates (likely CloudXR for Quest/Vision Pro).

## Prerequisites

### System Requirements

- **Operating System**: Windows 10/11 or Linux (Ubuntu 20.04/22.04 recommended)
- **GPU**: NVIDIA RTX capable GPU (Turing architecture or newer)
- **Driver**: Latest NVIDIA driver compatible with your GPU
- **Internet Access**: Required for downloading Kit SDK and extensions on first build

### Required Software

- [Git](https://git-scm.com/downloads)
- [Git LFS](https://git-lfs.com/)

### Platform-Specific Requirements

**Windows (C++ development only):**
- Microsoft Visual Studio 2019 or 2022 with "Desktop development with C++" workload
- Windows SDK (installed via Visual Studio Installer)

**Linux:**
- build-essential package: `sudo apt-get install build-essential`

## Installation

### From a fresh clone

These steps reproduce the verified-working state on a clean machine. Tested on Windows 11.

1. **Clone the repo:**
   ```powershell
   git clone https://github.com/MikeWise2718/sf4ov.git
   cd sf4ov
   ```

2. **Clone the `sphereflake22` extension** (required — loaded manually from the Extension Manager at runtime):
   ```powershell
   # Place alongside this repo or anywhere stable; the path is added to the
   # Extension Manager search paths later.
   git clone https://github.com/mikewise2718/sphereflake22.git D:/ov/exts/sphereflake22
   ```

3. **Build:**

   **Windows:**
   ```powershell
   .\repo.bat build
   ```

   **Linux:**
   ```bash
   ./repo.sh build
   ```

   A successful build displays:
   ```
   BUILD (RELEASE) SUCCEEDED (Took XX.XX seconds)
   ```

   First build downloads the Kit SDK and several hundred MB of extensions — expect a few minutes on a fast connection.

4. **Launch** (see [Running the Application](#running-the-application) below).

5. **Load the `sphereflake22` extension:**
   - Launch with developer tools: `.\repo.bat launch -d --name msft.sphereflake.kit`
   - **Window → Extensions** → gear icon → add the parent of `sphereflake22` (e.g. `D:/ov/exts`) to the extension search paths
   - Search `sphereflake22`, enable + autoload
   - Once enabled with autoload, subsequent regular launches (without `-d`) will load it automatically.

### Path and filesystem constraints

- **Windows path length**: place the repo near the drive root (e.g. `D:\ov\apps\…`). Deep paths break Kit's internal file resolution.
- **Filesystem**: NTFS required — the build uses symlinks/junctions. exFAT and FAT32 do not work.

## Running the Application

### Desktop Application

**Windows:**
```powershell
.\repo.bat launch
```

**Linux:**
```bash
./repo.sh launch
```

When prompted, select `msft.sphereflake.kit` for the desktop editor.

To skip the interactive picker (useful in non-Windows-console shells like Git Bash, MSYS, or CI), pass `--name`:

```powershell
.\repo.bat launch --name msft.sphereflake.kit
```

### With Developer Tools

To launch with Script Editor, Extension Manager, and other developer tools:

**Windows:**
```powershell
.\repo.bat launch -d
```

**Linux:**
```bash
./repo.sh launch -d
```

Combine with `--name` to skip the picker:

```powershell
.\repo.bat launch -d --name msft.sphereflake.kit
```

## Important Notes

- **First Launch**: The initial startup takes 5-8 minutes for RTX shader compilation. Subsequent launches are much faster.
- Path-length and filesystem constraints are covered in the [Installation](#path-and-filesystem-constraints) section.
- **Kit version**: The window title shows the bundled Kit version (e.g. "SphereFlake (Kit 110.1.0)"). **Help → About** opens a dialog with the kernel version.

## Additional Commands

```powershell
# Clean build
.\repo.bat build -c

# Rebuild from scratch
.\repo.bat build -x

# Run tests
.\repo.bat test

# Package for distribution
.\repo.bat package

# Pass arguments to Kit executable
.\repo.bat launch -- --clear-cache
```

## License

Development using the Omniverse Kit SDK is subject to the [NVIDIA Omniverse License Agreement](https://docs.omniverse.nvidia.com/dev-guide/latest/common/NVIDIA_Omniverse_License_Agreement.html).
