# 3d-print-skill

AI agent skill for parametric 3D model design and 3D printing. Teaches AI coding assistants (Claude Code, Gemini CLI, Copilot CLI) to design mechanically sound parts using CadQuery, verify them, present interactive 3D viewers, and slice/print via PrusaSlicer and Moonraker.

## What it does

When you ask your AI assistant to design a 3D-printable part, this skill guides it through a rigorous engineering process:

1. **Gathers requirements** — asks about mating parts, loads, forces, material, print constraints
2. **Analyzes constraints** — classifies every dimension (fixed/minimum/free), derives values from physics
3. **Calculates forces** — bearing stress, bolt shear, safety factors against material limits
4. **Designs for reality** — assembly sequence, tool access, interference checks, proper tolerances per fit type
5. **Sketches before coding** — ASCII cross-section to catch design failures before geometry
6. **Builds parametric model** — CadQuery (Python) with constraint validation and assertions
7. **Verifies every cut** — ray-trace through openings to catch covered-hole boolean failures
8. **Presents interactively** — Three.js viewer with orbit controls, edge overlay, view presets
9. **Slices and uploads** — PrusaSlicer CLI to G-code, Moonraker upload to printer (never auto-starts)

## Requirements

- **Python 3.10-3.12** — CadQuery requires this range
- **A 3D printer** — the skill is designed for FDM printer owners
- **CadQuery** — installed automatically on first use (or manually if auto-install fails)
- **PrusaSlicer** (optional) — only needed for slicing; installed automatically or manually
- **Claude Code**, **Gemini CLI**, or **Copilot CLI** — any AI coding assistant that supports skills/plugins

## Installation

### Claude Code

```bash
claude plugins install parhamdb/3d-print-skill
```

### Gemini CLI

Clone the repo and point Gemini to it via your `GEMINI.md` configuration.

### Copilot CLI

Clone the repo and configure as a skill source in your Copilot CLI setup.

## First-time setup

On first use, the skill checks for Python, CadQuery, and PrusaSlicer. If anything is missing, it detects your platform (Linux/macOS/Windows, x86/ARM) and either:

- **Auto-installs** with your confirmation, or
- **Provides step-by-step instructions** if auto-install fails (common for CadQuery on Apple Silicon)

The skill also walks you through adding your first printer if you haven't configured one yet.

## Quick start

After installing the plugin, start a conversation:

```
You: I need a pillow block mount for a KP08 bearing on a 2020 aluminum extrusion.

Claude: [asks about load, direction, material, fasteners, orientation...]
Claude: [designs the part with constraint analysis and force calculations]
Claude: [generates CadQuery script, runs it, verifies all cuts]
Claude: [serves interactive 3D viewer at http://127.0.0.1:8765/pillow_block_viewer.html]
Claude: Here's the interactive viewer. Please rotate and inspect the model. Any changes?

You: Looks good, print it.

Claude: [slices with PrusaSlicer, uploads to your printer via Moonraker]
Claude: Uploaded. Filament: 23g, estimated time: 1h 15m. Ready to start?
```

## Printer configuration

Printers are configured as TOML files in `~/.config/3d-print-skill/printers/`.

```bash
mkdir -p ~/.config/3d-print-skill/printers
```

Create a file for each printer (e.g., `~/.config/3d-print-skill/printers/ender3.toml`):

```toml
[printer]
name = "Ender 3 V2"
description = "Stock Ender 3 V2, 0.4mm nozzle"
host = "192.168.1.100"       # Moonraker IP (optional)

[build_volume]
x = 220    # mm
y = 220    # mm
z = 250    # mm

[hotend]
nozzle_diameter = 0.4        # mm
filament_diameter = 1.75     # mm
max_temp = 260               # C

[bed]
max_temp = 100               # C
```

See `config/example-printer.toml` for a fully commented example.

## Supported materials

The skill includes engineering data for:

| Material | Best for |
|----------|----------|
| PETG | General purpose, good strength, moisture resistant |
| PLA | Easy to print, stiff, good for prototypes |
| TPU | Flexible, impact resistant, vibration dampening |
| ABS | Heat resistant, high-temperature environments |

## Supported platforms

| Platform | CadQuery | PrusaSlicer |
|----------|----------|-------------|
| Linux x86_64 | pip | apt / AppImage |
| Linux ARM64 | pip / conda | AppImage |
| macOS Intel | pip | Homebrew cask |
| macOS ARM (Apple Silicon) | conda-forge (pip may fail) | Homebrew cask |
| Windows | pip | winget |

## How it works

This is a **purely prompt-based** skill. There's no Python library to install — the skill is markdown files that teach the AI assistant:

- The complete mechanical design process
- Material properties and tolerance tables
- CadQuery patterns and gotchas
- Cut verification procedures
- Three.js viewer template
- PrusaSlicer CLI usage
- Moonraker upload API

The AI writes all the code from scratch each time, informed by the skill's engineering knowledge.

## Contributing

Contributions welcome. The skill content is based on real-world experience designing and printing 10+ production parts (motor mounts, battery trays, sensor brackets, display holders, etc.).

If you've found a CadQuery gotcha, a better tolerance for a specific fit type, or a material property that needs updating, please open a PR.

## License

MIT
