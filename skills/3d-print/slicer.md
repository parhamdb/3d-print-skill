# PrusaSlicer CLI and Moonraker Upload

Slice STL files with PrusaSlicer CLI and upload G-code to Klipper printers via Moonraker API.

## Slicing with PrusaSlicer CLI

```bash
# Basic slice with profile files
prusa-slicer \
  --load printer-profile.ini \
  --load filament-profile.ini \
  --load quality-profile.ini \
  --export-gcode \
  --output output/part.gcode \
  input.stl

# Override brim width (for parts with small Z=0 contact area)
prusa-slicer \
  --load printer-profile.ini \
  --load filament-profile.ini \
  --load quality-profile.ini \
  --brim-width 5.0 \
  --export-gcode \
  --output output/part.gcode \
  input.stl

# List available options
prusa-slicer --help
```

### Profile Discovery

Users can use PrusaSlicer's built-in profiles or custom INI files. Custom profiles are stored in:

```
~/.config/3d-print-skill/slicer-profiles/
```

Ask the user which printer, filament, and quality profiles to use before slicing. Common quality presets:
- `0.20mm` -- standard quality, good balance of speed and detail
- `0.32mm` -- draft quality, fast prints for prototypes
- `0.12mm` -- fine quality, slower but smoother surfaces

### Slicer Notes

- First layer height must be >= 0.25mm for reliable bed adhesion
- Use `M104 S0\nM140 S0` before `PRINT_START` in start G-code to prevent PrusaSlicer from injecting its own temperature commands (relevant for Klipper macro setups)
- Slicer profiles should use `EXTRUDER_TEMP`/`BED_TEMP` variables that match Klipper macros

## Reading Printer Config

```python
import tomllib
from pathlib import Path

config_dir = Path.home() / ".config" / "3d-print-skill" / "printers"
printer_file = config_dir / "myprinter.toml"
with open(printer_file, "rb") as f:
    printer = tomllib.load(f)

build_x = printer["build_volume"]["x"]
build_y = printer["build_volume"]["y"]
build_z = printer["build_volume"]["z"]
nozzle = printer["printer"].get("nozzle_diameter", 0.4)
host = printer["printer"].get("host")  # Optional -- Moonraker host for upload
```

## Build Volume Validation

Always validate that the part fits within the printer's build volume before slicing.

```python
bbox = result.val().BoundingBox()
dx = bbox.xmax - bbox.xmin
dy = bbox.ymax - bbox.ymin
dz = bbox.zmax - bbox.zmin
assert dx <= build_x and dy <= build_y and dz <= build_z, \
    f"Part ({dx:.1f}x{dy:.1f}x{dz:.1f}) exceeds build volume ({build_x}x{build_y}x{build_z})"
print(f"Part fits: {dx:.1f} x {dy:.1f} x {dz:.1f} mm")
```

## Uploading G-code to Moonraker

```python
import urllib.request
import json
from pathlib import Path

def upload_gcode(gcode_path, host, start=False):
    """Upload G-code to Klipper printer via Moonraker API."""
    boundary = "----PythonBoundary"
    filename = Path(gcode_path).name
    body = (
        f"--{boundary}\r\n"
        f'Content-Disposition: form-data; name="file"; filename="{filename}"\r\n'
        f"Content-Type: application/octet-stream\r\n\r\n"
    ).encode()
    body += Path(gcode_path).read_bytes()
    body += f"\r\n--{boundary}--\r\n".encode()

    url = f"http://{host}/server/files/upload"
    if start:
        url += "?print=true"

    req = urllib.request.Request(
        url,
        data=body,
        headers={"Content-Type": f"multipart/form-data; boundary={boundary}"},
        method="POST",
    )
    with urllib.request.urlopen(req, timeout=30) as resp:
        result = json.loads(resp.read())
        print(f"Uploaded: {result.get('result', {}).get('item', {}).get('path', filename)}")
```

**NEVER pass `start=True` unless the user explicitly confirms they want to start the print.** Always upload with `start=False` first, report the metrics, and wait for user confirmation.

## G-code Header Parsing

After slicing, read the G-code header to extract print metrics for reporting to the user.

```python
def parse_gcode_header(gcode_path):
    """Parse PrusaSlicer G-code header for print metrics."""
    metrics = {}
    with open(gcode_path) as f:
        for line in f:
            if not line.startswith(';'):
                break
            if 'filament used [g]' in line:
                metrics['filament_g'] = float(line.split('=')[1].strip())
            elif 'filament used [cm3]' in line:
                metrics['filament_cm3'] = float(line.split('=')[1].strip())
            elif 'filament used [mm]' in line:
                metrics['filament_mm'] = float(line.split('=')[1].strip())
            elif 'estimated printing time' in line:
                metrics['print_time'] = line.split('=')[1].strip()
            elif 'layer_height' in line and '=' in line:
                try:
                    metrics['layer_height'] = float(line.split('=')[1].strip())
                except ValueError:
                    pass
            elif 'brim_width' in line and '=' in line:
                try:
                    metrics['brim_width'] = float(line.split('=')[1].strip())
                except ValueError:
                    pass
    return metrics
```

## Post-Slice Report Template

After slicing and uploading, report the following to the user:

```
Slicing complete:
  Filament: {filament_g:.1f}g / {filament_cm3:.1f}cm3 / {filament_mm:.0f}mm
  Estimated print time: {print_time}
  Layer height: {layer_height}mm
  Brim width: {brim_width}mm
  Printer: {printer_name}
  File uploaded: {filename}
  Print started: NO (waiting for user confirmation)
```

## Extensibility: OctoPrint Support

The Moonraker upload is a simple HTTP POST. To support OctoPrint in the future, add an alternative upload function:

```python
def upload_gcode_octoprint(gcode_path, host, api_key, start=False):
    """Upload G-code to OctoPrint."""
    boundary = "----PythonBoundary"
    filename = Path(gcode_path).name
    
    select_and_print = "true" if start else "false"
    body = (
        f"--{boundary}\r\n"
        f'Content-Disposition: form-data; name="file"; filename="{filename}"\r\n'
        f"Content-Type: application/octet-stream\r\n\r\n"
    ).encode()
    body += Path(gcode_path).read_bytes()
    body += (
        f"\r\n--{boundary}\r\n"
        f'Content-Disposition: form-data; name="select"\r\n\r\n'
        f"{select_and_print}\r\n"
        f"--{boundary}\r\n"
        f'Content-Disposition: form-data; name="print"\r\n\r\n'
        f"{select_and_print}\r\n"
        f"--{boundary}--\r\n"
    ).encode()

    req = urllib.request.Request(
        f"http://{host}/api/files/local",
        data=body,
        headers={
            "Content-Type": f"multipart/form-data; boundary={boundary}",
            "X-Api-Key": api_key,
        },
        method="POST",
    )
    with urllib.request.urlopen(req, timeout=30) as resp:
        result = json.loads(resp.read())
        print(f"Uploaded to OctoPrint: {result.get('files', {}).get('local', {}).get('name', filename)}")
```
