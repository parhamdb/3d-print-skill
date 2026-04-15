# Fit Type Classification for FDM Printed Parts

Never blanket-apply a single clearance value to all interfaces. Classify the fit type first, then apply the correct radial clearance for that interface.

## Fit Type Table

| Fit type | Use case | Radial clearance | Notes |
|----------|----------|------------------|-------|
| Press fit | Bearing OD into pocket, pin into plastic | 0.10-0.15mm | Interference with plastic elasticity |
| Snap/retention | Snap lip holding ring/cap | 0.10-0.15mm | Elastic deflection holds |
| Bolt clearance (static) | M-bolt clearance hole | 0.15-0.25mm | Self-centers under preload |
| Stationary shaft/dowel | Alignment pin, press axle | 0.25-0.50mm | No rotation |
| Rotating shaft through plastic | Motor output shaft, unconstrained axle | 0.75-1.25mm | Must never touch -- friction damages plastic |
| Accommodation fit | Precision boss into printed recess | 0.50-0.75mm | Precision feature entering sloppy-tolerance pocket |
| Cable/wire passage | Cables through wall | 1-2mm + cable OD | Generous for assembly |

## Key Rules

### FDM hole shrinkage compensation
FDM holes print 0.15-0.25mm undersized diametrically (varies by material -- see materials.md). Subtract this shrinkage from your designed clearance to estimate real clearance. A "0.3mm diametral clearance" on paper can become 0mm actual clearance after print shrink, resulting in interference. Always design with the real clearance you need after accounting for shrink.

### Do not reuse tolerances across different interface types
A pillow-block bearing pocket is a press fit. A motor-shaft through-hole is a rotating-shaft clearance. A bolt hole is a static clearance. These have different radial clearance values. Copy-pasting one tolerance into another interface type causes binding or slop.

### No threads in PETG/PLA
Printed threads in soft plastics strip immediately under any real load. Use heat-set inserts (M3-M6 brass knurled inserts, installed with soldering iron) or T-nuts for aluminum extrusion mounting. Heat-set inserts provide metal threads in a plastic body with excellent pull-out resistance.

### Rotating shaft pass-through rule
Any time a shaft rotates inside a printed hole with no bearing between them, that is a rotating-shaft class fit. Requires at least 0.75mm radial clearance (at least 1.5mm diametral). Never less, even if the fit looks fine on paper. Real printers and real motors have enough tolerance stack-up (print shrink + shaft runout + mounting misalignment + bearing play) to consume anything smaller.

## Worked Example: Motor Shaft Through-Hole

Motor shaft diameter: 6mm

1. Required radial clearance (rotating shaft): 0.75-1.25mm
2. Choose 1.0mm radial clearance
3. Designed hole diameter: 6 + 2(1.0) = 8.0mm
4. PETG print shrink: ~0.2mm diametral reduction
5. Actual printed hole: ~7.8mm
6. Actual radial clearance: (7.8 - 6) / 2 = 0.9mm -- still within acceptable range

## Worked Example: M5 Bolt Clearance Hole

Bolt shank diameter: 5.0mm

1. Required radial clearance (bolt static): 0.15-0.25mm
2. Choose 0.25mm radial clearance
3. Designed hole diameter: 5.0 + 2(0.25) = 5.5mm
4. PETG print shrink: ~0.2mm diametral reduction
5. Actual printed hole: ~5.3mm
6. Actual radial clearance: (5.3 - 5.0) / 2 = 0.15mm -- tight but acceptable for static bolt

## Worked Example: Bearing Press Fit

Bearing OD: 17.0mm

1. Required radial clearance (press fit): 0.10-0.15mm interference conceptually, but with print shrink the pocket prints smaller than designed
2. Designed pocket diameter: 17.0 + 2(0.10) = 17.2mm diametral
3. PETG print shrink: ~0.2mm diametral reduction
4. Actual printed pocket: ~17.0mm -- snug press fit as intended
