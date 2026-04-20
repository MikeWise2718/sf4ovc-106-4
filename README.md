# SphereFlake 106.4

A GPU-accelerated benchmarking application built on the NVIDIA Omniverse Kit SDK 106.4. SphereFlake is designed for benchmarking Omniverse applications across various platforms, providing a consistent workload for performance testing and comparison.

**Status:** Last verified working on 2026-04-20 (Windows 11, Kit SDK 106.4, build `106.4+main.0.ddbfb79b.local`). `.\repo.bat build` and `.\repo.bat launch --name msft.sphereflake1064.kit` both succeed. Tagged [`v106.4-working`](https://github.com/MikeWise2718/sf4ovc-106-4/releases/tag/v106.4-working).

## Overview

SphereFlake 106.4 is a base editor application that leverages the Omniverse Kit SDK for OpenUSD-based 3D rendering. It includes:

- **Desktop Application** (`msft.sphereflake1064.kit`): Full-featured editor with viewport, stage hierarchy, property panels, and content browser

> A streaming variant was removed ahead of the Kit 110 upgrade. It will be reintroduced against 110's `streaming_configs/` templates (CloudXR for Quest/Vision Pro).

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

1. **Clone the repository:**
   ```bash
   git clone <repository-url>
   cd sf4ovc-106-4
   ```

2. **Build the application:**

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

When prompted, select `msft.sphereflake1064.kit` for the desktop editor.

To skip the interactive picker (useful in non-Windows-console shells like Git Bash, MSYS, or CI), pass `--name`:

```powershell
.\repo.bat launch --name msft.sphereflake1064.kit
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
.\repo.bat launch -d --name msft.sphereflake1064.kit
```

## Important Notes

- **First Launch**: The initial startup takes 5-8 minutes for RTX shader compilation. Subsequent launches are much faster.
- **Windows Path Length**: Place the repository near the drive root to avoid path length issues.
- **Filesystem**: Requires NTFS (not exFAT) for symlink/junction support.

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
