# Cut Verification Procedures for CadQuery

Boolean operations in CadQuery/OCCT can silently fail, producing holes that appear correct in renders but are actually covered or incomplete. Every hole, bore, and through-cut must pass all three verification checks before delivery.

## Three Mandatory Checks

### 1. Interior Probe

Verify the cut center is empty (air) and adjacent walls are solid.

```python
from OCP.BRepClass3d import BRepClass3d_SolidClassifier
from OCP.gp import gp_Pnt

def is_solid(result, x, y, z):
    """Return True if point (x,y,z) is inside the solid."""
    c = BRepClass3d_SolidClassifier(result.val().wrapped, gp_Pnt(x, y, z), 0.01)
    return c.State() == 0

# Hole center should be air
assert not is_solid(result, hole_x, hole_y, hole_z), "Hole center is solid -- cut failed"

# Adjacent wall should be solid (confirms the hole has material around it)
assert is_solid(result, hole_x + hole_radius + wall/2, hole_y, hole_z), "Wall missing next to hole"
```

### 2. Ray-Trace Through Opening

Sample points from outside the solid, through the expected opening face, and into the hole interior. Space points every 0.5mm. If any point transitions from AIR to SOLID to AIR, the hole is covered by a thin shell -- the most common CadQuery boolean failure mode.

```python
def verify_hole_open(result, probe_points, description):
    """Trace a line of (x,y,z) points from outside through opening into hole.
    Any SOLID point between two AIR points means the hole is covered."""
    from OCP.BRepClass3d import BRepClass3d_SolidClassifier
    from OCP.gp import gp_Pnt
    states = []
    for x, y, z in probe_points:
        c = BRepClass3d_SolidClassifier(result.val().wrapped, gp_Pnt(x, y, z), 0.01)
        states.append(c.State())  # 0=IN(solid), 1=OUT(air)
    saw_air = False
    saw_solid_after_air = False
    for s in states:
        if s == 1:
            if saw_solid_after_air:
                assert False, f"COVERED HOLE: {description}"
            saw_air = True
        elif s == 0 and saw_air:
            saw_solid_after_air = True
```

**Usage examples:**

```python
# Socket bore: trace from below part (Z=-1) up through bottom opening into bore
verify_hole_open(result,
    [(0, 20, z) for z in [z * 0.5 for z in range(-2, 60)]],
    "socket bore bottom opening")

# M5 bolt hole: trace from outside backplate through hole to other side
verify_hole_open(result,
    [(-15, y, 10) for y in [y * 0.5 for y in range(-2, 20)]],
    "M5 left bolt through-hole")

# Counterbore: trace from above part down into counterbore and through bolt hole
verify_hole_open(result,
    [(bolt_x, bolt_y, z) for z in [z * 0.5 for z in range(60, -4, -1)]],
    "M5 counterbore from top")
```

### 3. Opening-Face Connectivity

Verify a point just outside the opening face is AIR (not solid). This confirms the hole is accessible from outside the part.

```python
# Through-hole: both faces must be open
assert not is_solid(result, bolt_x, -0.5, bolt_z), "Hole covered at entry face"
assert not is_solid(result, bolt_x, thickness + 0.5, bolt_z), "Hole covered at exit face"

# Blind bore: entry face must be open
assert not is_solid(result, 0, stem_y, bore_open_z - 0.5), "Bore opening covered"

# Counterbore: top must be open
assert not is_solid(result, bolt_x, bolt_y, top_z + 0.5), "Counterbore covered at top"
```

## Common Causes of Covered Holes

### 1. Union after cut
Adding material (union, fillet, chamfer) after cutting a hole can fill the hole back in. The fix: always perform boolean cuts LAST, after all additive geometry is complete.

### 2. Fillet on hole edge
CadQuery fillet operations on small hole edges can create degenerate geometry that seals the hole. The fix: apply fillets BEFORE cutting holes.

### 3. Coplanar faces
When a cut cylinder face is exactly coplanar with a solid body face, OCCT may keep the solid face instead of creating an opening. The fix: extend cuts 1mm past each face of the body.

```python
# BAD: cut exactly flush with face
result = body.cut(cq.Workplane("XY").cylinder(height=wall_thickness, radius=hole_r))

# GOOD: extend cut 1mm past each face
result = body.cut(cq.Workplane("XY").cylinder(height=wall_thickness + 2, radius=hole_r)
    .translate((0, 0, -1)))
```

### 4. Wrong extrusion direction
The XZ workplane extrudes in the -Y direction. Getting the offset or direction wrong puts the cut in entirely the wrong location, and since it misses the body, no material is removed. Always verify which direction your workplane extrudes.

| Workplane | Extrusion direction |
|-----------|-------------------|
| XY | +Z (default), -Z with `cutBlind` |
| XZ | -Y |
| YZ | -X |

### 5. Undersized cut
A cut cylinder that is slightly smaller than intended (due to parameter error) may leave a thin ring of material around the opening. Always verify the cut radius matches the intended dimension.

## Full Verification Template

```python
# === CUT VERIFICATION ===
from OCP.BRepClass3d import BRepClass3d_SolidClassifier
from OCP.gp import gp_Pnt

def is_solid(result, x, y, z):
    c = BRepClass3d_SolidClassifier(result.val().wrapped, gp_Pnt(x, y, z), 0.01)
    return c.State() == 0

def verify_hole_open(result, probe_points, description):
    from OCP.BRepClass3d import BRepClass3d_SolidClassifier
    from OCP.gp import gp_Pnt
    states = []
    for x, y, z in probe_points:
        c = BRepClass3d_SolidClassifier(result.val().wrapped, gp_Pnt(x, y, z), 0.01)
        states.append(c.State())
    saw_air = False
    saw_solid_after_air = False
    for s in states:
        if s == 1:
            if saw_solid_after_air:
                assert False, f"COVERED HOLE: {description}"
            saw_air = True
        elif s == 0 and saw_air:
            saw_solid_after_air = True

# For each hole in the design:
# 1. Interior probe
assert not is_solid(result, hole_x, hole_y, hole_z), "Hole center is solid"
assert is_solid(result, hole_x + radius + 1, hole_y, hole_z), "Wall missing"

# 2. Ray-trace through opening
verify_hole_open(result,
    [(hole_x, hole_y, z) for z in [i * 0.5 for i in range(-2, 40)]],
    "hole description")

# 3. Opening-face connectivity
assert not is_solid(result, hole_x, hole_y, entry_z - 0.5), "Entry covered"
assert not is_solid(result, hole_x, hole_y, exit_z + 0.5), "Exit covered"

print("All cut verifications passed")
```
