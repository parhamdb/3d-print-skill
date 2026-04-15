# Environment Setup

Read this file when Step 0 of SKILL.md detects missing tools. Run the setup once, then you're good for all future designs.

## Detection

Run these checks and note what's missing:

```bash
# 1. Python version (need 3.10-3.12)
python3 --version

# 2. CadQuery
python3 -c "import cadquery; print(cadquery.__version__)"

# 3. PrusaSlicer CLI (optional — only needed for slicing/printing)
prusa-slicer --version

# 4. Printer config (optional — only needed for printing)
ls ~/.config/3d-print-skill/printers/
```

## Platform detection

```bash
# OS
uname -s   # Linux, Darwin (macOS), or check via python: platform.system()

# Architecture
uname -m   # x86_64, aarch64 (Linux ARM), arm64 (macOS ARM)
```

## Install Python 3.12

Skip if `python3 --version` shows 3.10-3.12.

| Platform | Command |
|----------|---------|
| Linux (apt) | `sudo apt update && sudo apt install -y python3.12 python3.12-venv python3.12-dev` |
| Linux (dnf) | `sudo dnf install -y python3.12 python3.12-devel` |
| macOS (Homebrew) | `brew install python@3.12` |
| Windows | `winget install Python.Python.3.12` |

## Install CadQuery

CadQuery depends on OpenCascade (C++ CAD kernel). Installation varies by platform.

### Try pip first (works on most platforms)

```bash
# Create a project venv
python3.12 -m venv .venv
source .venv/bin/activate   # Linux/macOS
# .venv\Scripts\activate    # Windows

pip install cadquery
```

### If pip fails (common on macOS ARM / Apple Silicon)

**Option 1: conda-forge**

```bash
# Install miniforge if needed
brew install miniforge   # macOS
# Or: curl -L -O https://github.com/conda-forge/miniforge/releases/latest/download/Miniforge3-$(uname)-$(uname -m).sh && bash Miniforge3-*.sh

conda create -n cadquery python=3.12
conda activate cadquery
conda install -c conda-forge cadquery
```

**Option 2: Build from source**

See the CadQuery installation guide: https://cadquery.readthedocs.io/en/latest/installation.html

### Verify CadQuery works

```bash
python3 -c "
import cadquery as cq
result = cq.Workplane('XY').box(10, 10, 10)
print(f'CadQuery {cq.__version__} OK — built a test box')
print(f'BoundingBox: {result.val().BoundingBox()}')
"
```

## Install PrusaSlicer CLI (optional)

Only needed if the user wants to slice STL files and upload G-code to their printer. Skip if they just want to design and export STL.

| Platform | Command |
|----------|---------|
| Linux (AppImage) | Download from https://github.com/prusa3d/PrusaSlicer/releases — the AppImage includes the CLI |
| Linux (apt) | `sudo apt install prusa-slicer` (may be older version) |
| macOS | `brew install --cask prusaslicer` — CLI available at `/Applications/PrusaSlicer.app/Contents/MacOS/PrusaSlicer` |
| Windows | `winget install Prusa3D.PrusaSlicer` |

**macOS note:** After installing via Homebrew, the CLI may need a symlink:

```bash
sudo ln -sf /Applications/PrusaSlicer.app/Contents/MacOS/PrusaSlicer /usr/local/bin/prusa-slicer
```

### Verify PrusaSlicer

```bash
prusa-slicer --version
```

## Configure printer (optional)

Only needed if the user wants to slice and upload to a printer. Create the config directory and add a printer:

```bash
mkdir -p ~/.config/3d-print-skill/printers
```

Ask the user for:

1. **Printer name** — a short identifier (e.g., "ender3", "prusa-mk4")
2. **Build volume** — X, Y, Z in mm
3. **Nozzle diameter** — typically 0.4mm or 0.6mm
4. **Filament diameter** — 1.75mm (most common)
5. **Max hotend temp** — from printer specs
6. **Max bed temp** — from printer specs
7. **Moonraker host IP** — if running Klipper with Moonraker (optional)

Write the config:

```toml
# ~/.config/3d-print-skill/printers/<name>.toml

[printer]
name = "My Printer"
description = "Ender 3 V2, stock hotend"
# host = "192.168.1.100"   # Uncomment if using Moonraker

[build_volume]
x = 220
y = 220
z = 250

[hotend]
nozzle_diameter = 0.4
filament_diameter = 1.75
max_temp = 260

[bed]
max_temp = 100
```

### Optional: slicer profiles

If the user has custom PrusaSlicer INI profiles, they can place them in:

```bash
mkdir -p ~/.config/3d-print-skill/slicer-profiles
# Copy .ini files here
```

## After setup

Return to Step 1 of SKILL.md and proceed with the design.
