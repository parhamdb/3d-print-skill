# 3d-print-skill

AI agent skill for parametric 3D model design and printing with CadQuery.

This plugin provides the `3d-print` skill which guides the complete workflow from requirements gathering through CAD design, verification, interactive presentation, slicing, and printing.

## Usage

Invoke the skill when the user asks to design or print a 3D part:
- "I need a mount for X"
- "Design a bracket for Y"
- "Print an enclosure for Z"

The skill handles everything: setup verification, requirements gathering, constraint analysis, parametric modeling with CadQuery, cut verification, interactive Three.js presentation, PrusaSlicer slicing, and Moonraker upload.
