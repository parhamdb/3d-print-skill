# Interactive 3D Viewer Template

## When to use

Read this file at **Step 8 (Present the model)** of the design process, after the STL has been exported and all verification checks have passed.

## How to use

1. Copy the HTML template below into a new file: `output/<part>_viewer.html`
2. Replace all `{{PLACEHOLDERS}}` with the actual values for your part:
   - `{{TITLE}}` — human-readable part name, e.g. "Motor Mount" or "Bearing Pillow Block"
   - `{{DIMENSIONS}}` — envelope dimensions string, e.g. "80 x 60 x 35 mm"
   - `{{FEATURES}}` — key feature descriptions as HTML `<div>` elements, one per feature (e.g. `<div>2x M5 mounting holes on base</div>`)
   - `{{STL_FILE}}` — primary (design-orientation) STL filename, e.g. "motor_mount.stl"
   - `{{PRINTABLE_STL_FILE}}` — print-oriented STL filename, e.g. "motor_mount_printable.stl"
3. Save the file and serve it:
   ```bash
   cd output && python -m http.server 8765 --bind 127.0.0.1
   ```
   (Use `run_in_background: true` so the server stays alive.)
4. Verify with curl before giving the user any URLs:
   ```bash
   curl -s -o /dev/null -w "%{http_code}" http://127.0.0.1:8765/<part>_viewer.html
   curl -s -o /dev/null -w "%{http_code}" http://127.0.0.1:8765/<stl_file>.stl
   ```
   Both must return `200`.

## Single-orientation parts (no printable STL)

If the part does not need a separate print-oriented STL (same orientation for design and printing), remove the mode toggle entirely:

1. Delete the `<div id="mode">` block from the HTML.
2. In the JavaScript, remove the `meshes.print` loader block and the mode-switching logic.
3. Simplify `onAllLoaded` to load only the single STL — change the `loaded` counter check from `=== 2` to `=== 1`.
4. Replace the `setMode` function body to just show the single mesh directly.

## HTML Template

```html
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="utf-8">
<title>{{TITLE}} - Interactive 3D Viewer</title>
<style>
  html, body { margin: 0; padding: 0; overflow: hidden; background: #1a1a1a; color: #eee; font-family: -apple-system, system-ui, monospace; height: 100%; }
  #info {
    position: fixed; top: 12px; left: 12px; padding: 12px 16px;
    background: rgba(0,0,0,0.65); border-radius: 6px; z-index: 10;
    font-size: 13px; line-height: 1.5; max-width: 320px;
  }
  #info h2 { margin: 0 0 6px; font-size: 15px; }
  #info .hint { color: #aaa; font-size: 11px; margin-top: 6px; }
  #info .dim { color: #e67e22; }
  #controls { margin-top: 10px; display: flex; gap: 6px; flex-wrap: wrap; }
  #controls button {
    background: #333; color: #eee; border: 1px solid #555; border-radius: 4px;
    padding: 5px 9px; cursor: pointer; font: inherit; font-size: 11px;
  }
  #controls button:hover { background: #444; }
  #controls button.active { background: #e67e22; color: #111; border-color: #e67e22; }
  #mode {
    position: fixed; top: 12px; right: 12px; padding: 10px 14px;
    background: rgba(0,0,0,0.65); border-radius: 6px; z-index: 10; font-size: 12px;
  }
  #mode label { display: block; margin-bottom: 4px; color: #aaa; font-size: 10px; text-transform: uppercase; }
  #mode button {
    background: #333; color: #eee; border: 1px solid #555; border-radius: 4px;
    padding: 5px 9px; cursor: pointer; font: inherit; font-size: 11px; margin-right: 4px;
  }
  #mode button.active { background: #e67e22; color: #111; border-color: #e67e22; }
  #loading { position: fixed; top: 50%; left: 50%; transform: translate(-50%, -50%); font-size: 16px; color: #e67e22; }
</style>
</head>
<body>
<div id="info">
  <h2>{{TITLE}} (PETG)</h2>
  <div>Envelope: <span class="dim">{{DIMENSIONS}}</span></div>
  {{FEATURES}}
  <div class="hint">Left-drag: rotate · Scroll: zoom · Right-drag: pan</div>
  <div id="controls">
    <button data-view="iso">Iso</button>
    <button data-view="front">Front</button>
    <button data-view="back">Back</button>
    <button data-view="left">Left</button>
    <button data-view="right">Right</button>
    <button data-view="top">Top</button>
    <button data-view="bottom">Bottom</button>
  </div>
</div>
<div id="mode">
  <label>Orientation</label>
  <button data-mode="design" class="active">Design frame</button>
  <button data-mode="print">Print</button>
</div>
<div id="loading">Loading STL...</div>

<script type="importmap">
{
  "imports": {
    "three": "https://unpkg.com/three@0.160.0/build/three.module.js",
    "three/addons/": "https://unpkg.com/three@0.160.0/examples/jsm/"
  }
}
</script>
<script type="module">
import * as THREE from 'three';
import { OrbitControls } from 'three/addons/controls/OrbitControls.js';
import { STLLoader } from 'three/addons/loaders/STLLoader.js';

const scene = new THREE.Scene();
scene.background = new THREE.Color(0x1a1a1a);

const camera = new THREE.PerspectiveCamera(40, window.innerWidth / window.innerHeight, 0.1, 2000);

const renderer = new THREE.WebGLRenderer({ antialias: true });
renderer.setSize(window.innerWidth, window.innerHeight);
renderer.setPixelRatio(window.devicePixelRatio);
renderer.shadowMap.enabled = true;
document.body.appendChild(renderer.domElement);

scene.add(new THREE.AmbientLight(0xffffff, 0.35));
const key = new THREE.DirectionalLight(0xffffff, 0.9);
key.position.set(80, 120, 100);
key.castShadow = true;
scene.add(key);
const fill = new THREE.DirectionalLight(0x88aaff, 0.35);
fill.position.set(-80, -50, 60);
scene.add(fill);
const rim = new THREE.DirectionalLight(0xffccaa, 0.25);
rim.position.set(-60, 80, -60);
scene.add(rim);

const grid = new THREE.GridHelper(200, 20, 0x555555, 0x2a2a2a);
scene.add(grid);
const axes = new THREE.AxesHelper(30);
scene.add(axes);

const controls = new OrbitControls(camera, renderer.domElement);
controls.enableDamping = true;
controls.dampingFactor = 0.08;

const loader = new STLLoader();
let meshBBox = null;
let meshCenter = new THREE.Vector3();
let meshSize = new THREE.Vector3();
const meshes = {};
let currentMode = 'design';

const material = new THREE.MeshStandardMaterial({
  color: 0xe67e22,
  metalness: 0.15,
  roughness: 0.55,
  flatShading: false,
});
const edgeMat = new THREE.LineBasicMaterial({ color: 0x000000, transparent: true, opacity: 0.45 });

function buildFromGeometry(geometry, mode) {
  geometry.computeVertexNormals();
  const mesh = new THREE.Mesh(geometry, material);
  mesh.castShadow = true;
  mesh.receiveShadow = true;
  mesh.rotation.x = -Math.PI / 2;
  const edges = new THREE.EdgesGeometry(geometry, 20);
  const edgeLines = new THREE.LineSegments(edges, edgeMat);
  edgeLines.rotation.x = -Math.PI / 2;
  return { mesh, edges: edgeLines };
}

function computeBBox(geometry) {
  geometry.computeBoundingBox();
  const gb = geometry.boundingBox.clone();
  const rmin = new THREE.Vector3(gb.min.x, gb.min.z, -gb.max.y);
  const rmax = new THREE.Vector3(gb.max.x, gb.max.z, -gb.min.y);
  return new THREE.Box3(rmin, rmax);
}

function setMode(mode) {
  if (!meshes[mode]) return;
  for (const m of Object.values(meshes)) { m.mesh.visible = false; m.edges.visible = false; }
  meshes[mode].mesh.visible = true;
  meshes[mode].edges.visible = true;
  currentMode = mode;
  const box = computeBBox(meshes[mode].mesh.geometry);
  box.getCenter(meshCenter); box.getSize(meshSize); meshBBox = box;
  controls.target.copy(meshCenter);
  setView('iso');
  document.querySelectorAll('#mode button').forEach(btn => btn.classList.toggle('active', btn.dataset.mode === mode));
}

let loaded = 0;
function onAllLoaded() { document.getElementById('loading').remove(); setMode('design'); }

loader.load('{{STL_FILE}}', (geometry) => {
  meshes.design = buildFromGeometry(geometry, 'design');
  scene.add(meshes.design.mesh); scene.add(meshes.design.edges);
  meshes.design.mesh.visible = false; meshes.design.edges.visible = false;
  if (++loaded === 2) onAllLoaded();
});

loader.load('{{PRINTABLE_STL_FILE}}', (geometry) => {
  meshes.print = buildFromGeometry(geometry, 'print');
  scene.add(meshes.print.mesh); scene.add(meshes.print.edges);
  meshes.print.mesh.visible = false; meshes.print.edges.visible = false;
  if (++loaded === 2) onAllLoaded();
});

document.querySelectorAll('#mode button').forEach(btn => btn.addEventListener('click', () => setMode(btn.dataset.mode)));

function setView(name) {
  if (!meshBBox) return;
  const size = Math.max(meshSize.x, meshSize.y, meshSize.z);
  const dist = size * 2.2;
  const center = meshCenter.clone();
  let offset;
  switch (name) {
    case 'iso':    offset = new THREE.Vector3( 1, 1, 1); break;
    case 'front':  offset = new THREE.Vector3( 0, 0, 1); break;
    case 'back':   offset = new THREE.Vector3( 0, 0,-1); break;
    case 'left':   offset = new THREE.Vector3(-1, 0, 0); break;
    case 'right':  offset = new THREE.Vector3( 1, 0, 0); break;
    case 'top':    offset = new THREE.Vector3( 0, 1, 0); break;
    case 'bottom': offset = new THREE.Vector3( 0,-1, 0); break;
    default:       offset = new THREE.Vector3( 1, 1, 1);
  }
  offset.normalize().multiplyScalar(dist);
  camera.position.copy(center).add(offset);
  camera.up.set(0, 1, 0);
  camera.lookAt(center);
  controls.target.copy(center);
  controls.update();
}

document.querySelectorAll('#controls button').forEach(btn => btn.addEventListener('click', () => setView(btn.dataset.view)));

window.addEventListener('resize', () => {
  camera.aspect = window.innerWidth / window.innerHeight;
  camera.updateProjectionMatrix();
  renderer.setSize(window.innerWidth, window.innerHeight);
});

function animate() { requestAnimationFrame(animate); controls.update(); renderer.render(scene, camera); }
animate();
</script>
</body>
</html>
```
