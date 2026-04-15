# Fastener Reference for 3D Printed Parts

## Clearance Holes and Edge Distances

| Fastener | Hole dia | Min edge distance | Wrench clearance | Socket head OD | Socket head height |
|----------|----------|-------------------|------------------|----------------|--------------------|
| M3 | 3.4mm | 8.5mm | 7mm | 5.5mm | 3.0mm |
| M4 | 4.5mm | 11mm | 8mm | 7.0mm | 4.0mm |
| M5 | 5.5mm | 14mm | 10mm | 8.5mm | 5.0mm |
| M6 | 6.6mm | 16.5mm | 12mm | 10.0mm | 6.0mm |

### Edge Distance Rule

Minimum distance from bolt center to nearest free edge of material = 2.5 x bolt diameter. This prevents bolt pullout under load in plastic parts.

- M3: 2.5 x 3 = 7.5mm, rounded up to 8.5mm for safety
- M4: 2.5 x 4 = 10mm, rounded up to 11mm
- M5: 2.5 x 5 = 12.5mm, rounded up to 14mm
- M6: 2.5 x 6 = 15mm, rounded up to 16.5mm

### Wrench Clearance

The wrench clearance column gives the minimum clear circle diameter around the bolt head needed for a socket wrench or hex key to operate. No solid material (arms, bosses, ribs, gussets) may intrude into this circle.

For every fastener, also verify tool-access: trace a straight line from outside the part to the bolt head. If that line passes through solid material at any point, the fastener is inaccessible regardless of wrench clearance.

## Counterbore Dimensions (Socket Head Cap Screws)

| Bolt | Counterbore dia | Counterbore depth |
|------|----------------|-------------------|
| M3 | 6.5mm | 3.5mm |
| M4 | 8.0mm | 4.5mm |
| M5 | 9.5mm | 5.5mm |
| M6 | 11.0mm | 6.5mm |

Counterbore diameter includes ~1mm clearance around the socket head OD. Counterbore depth is socket head height + 0.5mm so the head sits fully below the surface.

## T-Nut Specs for Aluminum Extrusions

| Extrusion | Slot width | Common bolt | T-nut type |
|-----------|-----------|-------------|------------|
| 2020 | 6mm | M5 | Drop-in or slide-in |
| 3030 | 8mm | M6 | Drop-in or slide-in |
| 4040 | 10mm | M8 | Drop-in or slide-in |

Drop-in T-nuts can be inserted from any point along the extrusion slot without sliding from the end. Slide-in T-nuts must be inserted from an open end of the extrusion. Drop-in is preferred for assembly convenience.

## Heat-Set Insert Dimensions

For threaded fastening into plastic parts, use brass knurled heat-set inserts installed with a soldering iron. Never cut threads directly into PETG/PLA.

| Insert | Hole dia in plastic | Insert OD | Insert length | Pull-out force (PETG) |
|--------|--------------------|-----------|--------------|-----------------------|
| M3 | 4.0mm | 4.6mm | 4-6mm | ~200N |
| M4 | 5.0mm | 5.6mm | 5-8mm | ~300N |
| M5 | 6.0mm | 6.8mm | 6-10mm | ~400N |

Hole diameter is slightly undersized relative to insert OD so the insert melts into the surrounding plastic for a secure bond.

## Bolt Selection Guide

- **M3**: Small electronics, PCB mounting, light brackets. Max recommended load in PETG: ~100N per bolt.
- **M4**: Medium brackets, sensor mounts, moderate loads. Max recommended load in PETG: ~200N per bolt.
- **M5**: Primary structural fastening to 2020 extrusion, motor mounts, bearing blocks. Max recommended load in PETG: ~350N per bolt.
- **M6**: Heavy structural, 3030 extrusion mounting. Max recommended load in PETG: ~500N per bolt.

These load limits are approximate and assume adequate edge distance, 100% infill around the bolt, and loads primarily in shear (not tension pulling out of plastic).

## Bolt Interference Checklist

For every bolt in the design, verify:

1. Edge distance from bolt center to nearest free edge >= 2.5 x bolt diameter
2. Clear circle around bolt head >= wrench clearance (no adjacent features intruding)
3. Straight-line tool access from outside the part to the bolt head (no solid material blocking)
4. Counterbore (if used) does not overlap any adjacent solid feature: bolt_center - feature_edge > counterbore_radius + 1mm
5. Bolt insertion path is clear -- no arm, rib, or boss blocks the hex key or wrench approach
