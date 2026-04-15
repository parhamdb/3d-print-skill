# FDM 3D Printing Material Properties

Reference data for common FDM filament materials. Use these values for force analysis, stress calculations, and print parameter selection.

## Mechanical and Thermal Properties

| Property | PETG | PLA | TPU (95A) | ABS |
|----------|------|-----|-----------|-----|
| Tensile yield (MPa) | ~50 | ~60 | ~30 | ~40 |
| Shear strength (MPa) | ~30 | ~35 | ~15 | ~25 |
| Layer adhesion Z (MPa) | 15-25 | 20-30 | ~20 | 15-20 |
| Max bearing stress (MPa) | 10 | 12 | 5 | 8 |
| Elastic modulus (MPa) | 2000 | 3500 | 50-100 | 2200 |
| Density (g/cm3) | 1.27 | 1.24 | 1.21 | 1.04 |
| Heat deflection (C) | 70 | 55 | - | 95 |
| Print shrink on holes (mm) | 0.15-0.25 | 0.10-0.20 | 0.20-0.40 | 0.30-0.50 |
| Bed temp (C) | 70-80 | 50-60 | 50-60 | 90-110 |
| Hotend temp (C) | 230-250 | 190-220 | 220-240 | 230-250 |

## Material Selection Guide

### PETG
General-purpose engineering material. Good balance of strength, layer adhesion, and moisture resistance. Suitable for functional parts, enclosures, brackets, and mounts. Moderate heat resistance (70C). Preferred default for structural parts.

### PLA
Easiest to print with high stiffness and dimensional accuracy. Poor heat resistance (55C) -- parts soften in direct sunlight or warm environments. Brittle under impact. Best for prototypes, jigs, fixtures, and parts that stay indoors at room temperature.

### TPU (95A)
Flexible and impact-resistant. Excellent vibration dampening. Difficult to print (requires direct-drive extruder, slow speeds). Use for wheels, tires, bumpers, gaskets, vibration isolators, and flexible couplings. Shore 95A is semi-rigid; lower durometers are softer but harder to print.

### ABS
Best heat resistance of the group (95C). Good for parts near heat sources, automotive applications, or outdoor use in hot climates. Prone to warping -- requires enclosed printer, high bed temp, and good bed adhesion strategy. Weaker layer adhesion than PETG.

## Notes

- **Layer adhesion Z values** represent inter-layer bond strength, which is always weaker than in-plane tensile strength. Orient prints so primary loads are within layers, not pulling layers apart.
- **Print shrink on holes** means FDM holes print undersized by this amount diametrically. Compensate in the design by adding this value to the nominal hole diameter.
- **Max bearing stress** is for sustained static loads on printed surfaces. Dynamic or impact loads should use a safety factor of 2-3x.
- All values are approximate and vary with print settings (layer height, infill, perimeter count, cooling). Use 100% infill for load-bearing features.
