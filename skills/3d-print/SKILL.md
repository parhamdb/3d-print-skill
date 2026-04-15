---
name: 3d-print
description: "Use when the user wants to design, model, or 3D print a part. Guides the complete workflow: requirements, CAD design with CadQuery, verification, interactive Three.js presentation, slicing, and printing."
---

# 3D Print — Parametric CAD Design & Printing

Design parametric 3D models for FDM 3D printing using Python and CadQuery. This skill covers the complete workflow from gathering requirements through slicing and uploading G-code to the printer.

**This is a rigid process. Every part goes through every step. No shortcuts.**

## Step 0: Environment check

Before any design work, verify the toolchain is installed:

```bash
python3 --version          # Need 3.10-3.12
python3 -c "import cadquery; print(cadquery.__version__)"
prusa-slicer --version     # Optional — needed for slicing
ls ~/.config/3d-print-skill/printers/   # Optional — needed for printing
```

If anything is missing, read `setup.md` in this skill directory and run the bootstrap. This only happens once.

## Step 1: Gather requirements — ASK before designing

Before touching code, gather ALL information needed. Don't assume. Don't guess.

**Always ask the user:**

1. **Reference images** — "Can you show me an example?" Pictures eliminate ambiguity.
2. **What does it mate with?** — Bearing model, shaft diameter, extrusion type, mounting surface geometry.
3. **Load and forces** — "How much weight will this carry? What direction? Static or dynamic? Impact loads?"
4. **Orientation on the machine** — "Is this on top, bottom, or side? Which direction does the axle point? Which direction is gravity?"
5. **Assembly method** — "What fasteners? From which direction? Do you need to remove it easily?"
6. **Material** — PETG, PLA, TPU, ABS. This affects clearances, strength, and temperature limits.
7. **Print constraints** — "Preferred print orientation? Avoid supports?"

**For motors, servos, sensors, and other non-uniform mating parts — DO NOT accept a single "diameter" number.**

A motor is not a cylinder. Collect the **full envelope**:

8. **Face diameter** — outermost diameter of the mating face
9. **Body diameter** — main cylindrical section behind the face
10. **Rear-section diameter** — encoder, heatsink, or gearbox housing
11. **Radial protrusions** — terminal block, wire exit, mounting ears. For each: OD past body, and which side.
12. **Shaft axis offset from body center** — "Is the output shaft centered or offset?"
13. **Shaft axis offset from bolt-pattern center**
14. **Maximum downward reach from shaft axis** — the single most important number for mounting clearance. Include everything: encoder OD, wire exit, terminal box.

Prefer a dimensioned side-view photo over verbal dimensions.

**Mounting-surface clearance rule:**

```
shaft_axis_z = mounting_surface_z + max_downward_reach_from_shaft_axis + safety_margin
```

NOT `face_radius + safety_margin` (wrong — ignores encoder, protrusions, shaft offset).

**Annotate axes** before designing: extrusion direction, axle direction, up/gravity direction.

## Step 2: Constraint analysis

**For every dimension: "What constrains this? What is the minimum? Is there an upper limit?"**

| Type | Meaning | Action |
|------|---------|--------|
| **Fixed** | Determined by mating part | Exact value + tolerance |
| **Minimum** | Must be at least X for strength | Calculate from physics, add margin |
| **Free** | No upper constraint | **Be generous.** Never default to compact. |

Document every parameter's constraint in code:

```python
# [Fixed] bearing OD — from datasheet
bearing_od = 17.0

# [Minimum] wall thickness — plastic needs 2-3x metal equivalent
wall = 6.0  # minimum 5mm for load-bearing

# [Free — be generous] foot length — no upper limit
foot_extend = 35
```

Derive values from physics, not intuition. Add constraint validation assertions.

Read `tolerances.md` in this skill directory for fit type classification.
Read `fasteners.md` for edge distance rules and wrench clearances.

## Step 3: Force analysis

Read `materials.md` in this skill directory for material property tables.

Calculate:
1. Load per bolt (static analysis)
2. Bearing stress on housing wall
3. Bolt shear stress
4. Moment arms and bolt tension

All stresses must be well below material yield with safety margin. Include the force calculations in the CadQuery script output.

**Mounting orientation matters:**
- Top mount: bolts in shear. Strong.
- Bottom mount: bolts in tension, pulling out of plastic. Weak. Need more bolts, more edge distance.
- Side mount: asymmetric loading. Need analysis.

## Step 4: Design for physical reality

1. **Assembly sequence — WRITE IT OUT** and print it in the script. Walk through every step: pick up, insert, place, fasten. If you can't describe a valid sequence, the design is broken.
2. **Tool-access ray-trace** — for every fastener, trace a straight line from outside the part to the bolt head. If it passes through solid material, the fastener is inaccessible.
3. **Interference check** — edge-to-edge distances, not center-to-center.
4. **Bolt edge distance** — minimum 2.5x bolt diameter from center to nearest edge.
5. **Wrench access** — clear circle around bolt head (read `fasteners.md` for sizes).
6. **Classify the fit type FIRST** — read `tolerances.md`. Never blanket-apply clearances.
7. **PCB/board mounting rules** — standoffs (not flush faces), inspect both sides, cable routing path, minimal material for lightweight boards.
8. **Sensor FoV verification** — for optical/acoustic sensors, draw the FoV cone. No mount material in the cone. Multipath check for ToF sensors (>30mm from window).

## Step 4.5: Sketch before code — MANDATORY

**Before writing ANY CadQuery geometry, draw a YZ cross-section sketch as ASCII art in the code comments.** Must show:

1. Mount body outline
2. Mating component in position
3. All fastener paths with tool access lines
4. Cable routing path
5. Sensor FoV cone (if applicable)
6. Mounting surface

This catches 90% of design failures before any code is written.

## Step 5: Design for 3D printing

1. **Print orientation FIRST** — constrains geometry. Layers parallel to load direction.
2. **Layer adhesion analysis** — loads WITHIN layers (strong), not pulling layers APART (weak).
3. **Bridge spans** — unsupported horizontal span > 10mm needs redesign or supports.
4. **Wall thickness** — 2-3x metal equivalent. 6mm minimum around bearings.
5. **Material around fasteners** — minimum 2.5x bolt diameter from bolt to any edge.

## Step 6: Build parametric model

Structure every CadQuery script as:

```python
import sys
from pathlib import Path
import cadquery as cq

# === CONSTRAINTS (from physical requirements) ===
# Every parameter documented with constraint type and source

# === CONSTRAINT VALIDATION ===
# Assertions that catch design errors before geometry

# === FORCE ANALYSIS ===
# Calculate stresses, safety factors

# === MODEL ===
# Build geometry from validated parameters

# === CUT VERIFICATION ===
# Point-in-solid checks for every boolean cut

# === EXPORT ===
output = Path(__file__).parent / "output"
output.mkdir(exist_ok=True)
cq.exporters.export(result, str(output / "part_name.stl"))
```

### CadQuery gotchas

- **XZ workplane extrudes in -Y.** `workplane(offset=-N)` puts you at Y=+N.
- **Never use `.faces().workplane().center()` for cuts** — offsets are relative to face centroid. Use explicit cylinders at global coordinates with `.cut()`.
- **Fillet before cutting holes.** Never silently catch fillet exceptions.
- **Never rewrite the entire model at once.** Make ONE change, re-run, verify, then next.
- **Stick to XY workplane boxes** for primary bodies (+Z extrusion is unambiguous).
- **Print-orientation rotation:** `-90` about X puts max-Y face on bed. `+90` puts min-Y face on bed.
- **`radiusArc` is ambiguous** — use `threePointArc` with explicit midpoint and probe-verify.

## Step 7: Verify every cut

Read `verification.md` in this skill directory for full procedures and code templates.

**For every hole, bore, and through-cut, perform ALL THREE checks:**

1. **Interior probe** — cut center is empty, adjacent walls are solid
2. **Ray-trace through opening** — sample every 0.5mm from outside through opening into hole. AIR->SOLID->AIR = covered hole.
3. **Opening-face connectivity** — point just outside opening face is AIR

**Common causes of covered holes:** union after cut, fillet on hole edge, coplanar faces, wrong extrusion direction. Always cut holes LAST, fillet BEFORE cutting, extend cuts 1mm past each face.

## Step 8: Render and present

Read `viewer-template.md` in this skill directory for the complete Three.js viewer template.

**Every time you deliver a part:**

1. Generate an interactive Three.js STL viewer HTML file
2. Serve locally: `cd output && python -m http.server 8765 --bind 127.0.0.1`
3. Verify with curl that the viewer HTML and STL return 200
4. Present the URL to the user
5. **Ask for feedback** — the user needs to rotate, zoom, and inspect in 3D
6. Iterate until the user approves

Do not skip this step because "the checks passed" — numerical verification is a prerequisite for presenting, not a substitute.

## Step 9: Slice and print

Read `slicer.md` in this skill directory for PrusaSlicer CLI usage and Moonraker upload.

1. Read printer config from `~/.config/3d-print-skill/printers/<name>.toml`
2. Validate part fits the build volume
3. Slice with PrusaSlicer CLI
4. Upload G-code to Moonraker (if printer has a host configured)
5. Report to user: filament (g, cm3, m), print time, layer height, brim, printer name
6. **NEVER auto-start the print.** User confirms.

If no printer is configured, tell the user the STL is ready and guide them to set up a printer (read `setup.md`).

## Step 10: Mechanical checklist

**All must pass before declaring done:**

- [ ] Fit type classified and correct clearances applied
- [ ] Bolt-feature interference (edge-to-edge > 5mm)
- [ ] Bolt edge distance (>= 2.5x bolt diameter)
- [ ] Bolt accessibility with wrench
- [ ] Tool-access ray-trace clear for every fastener
- [ ] Mounting surface compatibility
- [ ] Wall integrity verified (point-in-solid)
- [ ] Through-holes verified (ray-trace)
- [ ] Assembly sequence written and every step physically possible
- [ ] Print orientation: loads within layers
- [ ] No unsupported bridges > 10mm
- [ ] Force analysis: stresses below material limits
- [ ] Load path through solid material
- [ ] Proportions match reference
- [ ] PCB mounts: standoffs, back-side clearance, cable routing (if applicable)
- [ ] Sensor mounts: FoV cone clear, multipath >30mm (if applicable)
- [ ] Interactive viewer presented, user feedback collected
- [ ] G-code sliced, uploaded, print NOT started (if printer configured)

## Step 11: Iterate until perfect

If ANY check fails: fix -> verify cuts -> render -> checklist. **Never deliver with known issues.**
