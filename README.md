<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8" />
<meta name="viewport" content="width=device-width, initial-scale=1.0, viewport-fit=cover" />
<title>Plane Mirror Optics — VR Lab</title>
<link rel="preconnect" href="https://fonts.googleapis.com">
<link href="https://fonts.googleapis.com/css2?family=JetBrains+Mono:wght@400;500;700&family=Inter:wght@400;500;600;700&display=swap" rel="stylesheet">
<style>
  :root{
    --bg:#0B0E14;
    --panel:#11151D;
    --ink:#E8E6DE;
    --ink-dim:#8B92A0;
    --object:#FF6A3D;
    --image:#3DD9FF;
    --structure:#5A6270;
    --accent:#2A8F5B;
    --line:#262C38;
  }
  *{box-sizing:border-box;}
  html,body{
    margin:0; padding:0; width:100%; height:100%;
    background:var(--bg); color:var(--ink);
    font-family:'Inter',system-ui,sans-serif;
    overflow:hidden;
    -webkit-tap-highlight-color:transparent;
  }
  #app{position:fixed; inset:0;}
  canvas{display:block; touch-action:none;}

  /* ---------- Top bar ---------- */
  #topbar{
    position:fixed; top:0; left:0; right:0;
    display:flex; align-items:center; justify-content:space-between;
    padding:14px 18px;
    background:linear-gradient(to bottom, rgba(11,14,20,0.92), rgba(11,14,20,0));
    pointer-events:none;
    z-index:10;
  }
  #topbar .brand{
    pointer-events:auto;
    display:flex; align-items:baseline; gap:8px;
  }
  #topbar .brand .mark{
    width:9px; height:9px; border-radius:50%;
    background:var(--accent);
    box-shadow:0 0 8px var(--accent);
  }
  #topbar h1{
    font-size:14px; font-weight:600; margin:0; letter-spacing:0.01em;
  }
  #topbar .sub{
    font-family:'JetBrains Mono',monospace;
    font-size:11px; color:var(--ink-dim);
  }
  #vrButton{
    pointer-events:auto;
    background:var(--accent);
    color:#06140C;
    border:none;
    font-family:'JetBrains Mono',monospace;
    font-weight:700;
    font-size:12px;
    letter-spacing:0.04em;
    padding:10px 18px;
    border-radius:3px;
    cursor:pointer;
    text-transform:uppercase;
    transition:filter .15s ease, transform .1s ease;
  }
  #vrButton:hover{filter:brightness(1.12);}
  #vrButton:active{transform:scale(0.97);}
  #vrButton:disabled{
    background:var(--structure); color:var(--ink-dim); cursor:not-allowed;
  }
  #vrButton:focus-visible{outline:2px solid var(--image); outline-offset:2px;}

  /* ---------- Bottom instrument dock ---------- */
  #dock{
    position:fixed; left:0; right:0; bottom:0;
    padding:16px 18px 18px;
    background:linear-gradient(to top, rgba(11,14,20,0.96), rgba(11,14,20,0.65) 70%, rgba(11,14,20,0));
    z-index:10;
  }
  #dockInner{
    max-width:920px; margin:0 auto;
    display:flex; flex-direction:column; gap:12px;
  }
  .readouts{
    display:flex; gap:10px; flex-wrap:wrap;
  }
  .readout{
    flex:1 1 140px;
    background:var(--panel);
    border:1px solid var(--line);
    border-radius:4px;
    padding:9px 12px;
    min-width:120px;
  }
  .readout .label{
    font-size:10px; text-transform:uppercase; letter-spacing:0.08em;
    color:var(--ink-dim); margin-bottom:3px;
  }
  .readout .value{
    font-family:'JetBrains Mono',monospace;
    font-size:17px; font-weight:500;
  }
  .readout.object .value{color:var(--object);}
  .readout.image .value{color:var(--image);}

  .controls-row{
    display:flex; align-items:center; gap:14px; flex-wrap:wrap;
  }
  .slider-wrap{
    flex:1 1 260px;
    display:flex; align-items:center; gap:10px;
    background:var(--panel);
    border:1px solid var(--line);
    border-radius:4px;
    padding:10px 14px;
  }
  .slider-wrap label{
    font-size:11px; color:var(--ink-dim); white-space:nowrap;
    font-family:'JetBrains Mono',monospace;
  }
  input[type=range]{
    flex:1;
    -webkit-appearance:none;
    height:3px;
    background:var(--line);
    border-radius:2px;
    outline:none;
  }
  input[type=range]::-webkit-slider-thumb{
    -webkit-appearance:none;
    width:16px; height:16px; border-radius:50%;
    background:var(--object);
    cursor:pointer;
    box-shadow:0 0 0 3px rgba(255,106,61,0.18);
  }
  input[type=range]::-moz-range-thumb{
    width:16px; height:16px; border-radius:50%;
    background:var(--object); border:none; cursor:pointer;
  }

  .toggles{
    display:flex; gap:8px; flex-wrap:wrap;
  }
  .toggle{
    background:var(--panel);
    border:1px solid var(--line);
    color:var(--ink-dim);
    font-family:'JetBrains Mono',monospace;
    font-size:11px;
    letter-spacing:0.02em;
    padding:9px 13px;
    border-radius:4px;
    cursor:pointer;
    display:flex; align-items:center; gap:7px;
    transition:border-color .15s ease, color .15s ease;
    white-space:nowrap;
  }
  .toggle .dot{
    width:7px; height:7px; border-radius:50%;
    background:var(--structure);
    transition:background .15s ease, box-shadow .15s ease;
  }
  .toggle.on{ color:var(--ink); border-color:var(--accent); }
  .toggle.on .dot{ background:var(--accent); box-shadow:0 0 6px var(--accent); }
  .toggle:focus-visible{outline:2px solid var(--image); outline-offset:1px;}

  /* ---------- Hint / first-load helper ---------- */
  #hint{
    position:fixed; top:64px; left:50%; transform:translateX(-50%);
    background:var(--panel); border:1px solid var(--line);
    color:var(--ink-dim); font-size:12px;
    padding:8px 14px; border-radius:4px;
    pointer-events:none;
    z-index:9;
    transition:opacity .6s ease;
  }

  #vrNotice{
    position:fixed; bottom:90px; left:50%; transform:translateX(-50%);
    background:#2A1414; border:1px solid #5A2A2A; color:#E8B8B8;
    font-size:12px; padding:8px 14px; border-radius:4px;
    z-index:9; display:none; max-width:80vw; text-align:center;
  }

  @media (prefers-reduced-motion: reduce){
    *{ transition:none !important; }
  }

  @media (max-width:600px){
    #topbar h1{font-size:12px;}
    .readout{flex:1 1 100px; padding:7px 10px;}
    .readout .value{font-size:14px;}
  }
</style>
</head>
<body>
<div id="app"></div>

<div id="topbar">
  <div class="brand">
    <span class="mark"></span>
    <div>
      <h1>Plane Mirror — Optics Bench</h1>
      <div class="sub">VR-CAPABLE · HEAD-TRACKED</div>
    </div>
  </div>
  <button id="vrButton">Enter VR Mode</button>
</div>

<div id="hint">Drag the orange object along the bench, or use the slider below.</div>
<div id="vrNotice">This browser/device doesn't support WebXR VR. Try Chrome on a Quest, or any WebXR-compatible headset browser.</div>

<div id="dock">
  <div id="dockInner">
    <div class="readouts">
      <div class="readout object">
        <div class="label">Object distance</div>
        <div class="value" id="readObj">1.00 m</div>
      </div>
      <div class="readout image">
        <div class="label">Image distance</div>
        <div class="value" id="readImg">1.00 m</div>
      </div>
      <div class="readout">
        <div class="label">Image type</div>
        <div class="value" id="readType">VIRTUAL · UPRIGHT · 1:1</div>
      </div>
    </div>

    <div class="controls-row">
      <div class="slider-wrap">
        <label for="distSlider">DIST</label>
        <input type="range" id="distSlider" min="0.3" max="3" step="0.01" value="1.0" />
      </div>
      <div class="toggles">
        <button class="toggle on" id="toggleRays" data-on="true"><span class="dot"></span>Light rays</button>
        <button class="toggle on" id="toggleMarkers" data-on="true"><span class="dot"></span>Distance markers</button>
        <button class="toggle on" id="toggleGuides" data-on="true"><span class="dot"></span>Guide lines</button>
      </div>
    </div>
  </div>
</div>

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
import { VRButton } from 'three/addons/webxr/VRButton.js';

/* =========================================================================
   PLANE MIRROR OPTICS — SCENE STATE
   Mirror sits in the YZ plane at x = 0 (facing +X, toward the object/viewer).
   Object lives at x = +objectDistance (positive side, "real" side).
   Image is computed at x = -objectDistance (mirrored across x=0).
   ========================================================================= */

const state = {
  objectDistance: 1.0,     // meters, always positive: distance from mirror
  showRays: true,
  showMarkers: true,
  showGuides: true,
  objectHeight: 0.5,       // meters — fixed; image height must match exactly
  mirrorHeight: 1.6,
  mirrorWidth: 1.0,
  dragging: false,
};

/* ---------- Renderer / Scene / Camera ---------- */
const container = document.getElementById('app');
const renderer = new THREE.WebGLRenderer({ antialias: true });
renderer.setPixelRatio(Math.min(window.devicePixelRatio, 2));
renderer.setSize(window.innerWidth, window.innerHeight);
renderer.xr.enabled = true;
renderer.outputColorSpace = THREE.SRGBColorSpace;
container.appendChild(renderer.domElement);

const scene = new THREE.Scene();
scene.background = new THREE.Color(0x0B0E14);
scene.fog = new THREE.Fog(0x0B0E14, 6, 14);

const camera = new THREE.PerspectiveCamera(
  55, window.innerWidth / window.innerHeight, 0.05, 100
);
// Desktop viewpoint: standing to the side, looking down the optical axis.
camera.position.set(2.4, 1.5, 2.6);
camera.lookAt(0, 1.0, 0);

// A rig the camera lives inside of. In VR, WebXR drives the camera's local
// transform directly (head tracking); we move THIS RIG for teleport/parallax
// instead of touching the camera's own transform, which XR owns at runtime.
const cameraRig = new THREE.Group();
cameraRig.add(camera);
scene.add(cameraRig);

/* ---------- Lighting: lab-style, directional + soft fill ---------- */
const hemi = new THREE.HemisphereLight(0x4A5568, 0x0B0E14, 0.6);
scene.add(hemi);

const key = new THREE.DirectionalLight(0xFFF4E8, 1.1);
key.position.set(3, 4, 4);
key.castShadow = true;
key.shadow.mapSize.set(1024, 1024);
key.shadow.camera.near = 0.5;
key.shadow.camera.far = 15;
scene.add(key);

const rim = new THREE.PointLight(0x3DD9FF, 0.5, 8);
rim.position.set(-2, 2, 1);
scene.add(rim);

/* ---------- Optical bench floor (subtle grid, like an optics table) --- */
const benchGroup = new THREE.Group();
scene.add(benchGroup);

const benchMat = new THREE.MeshStandardMaterial({
  color: 0x14181F, metalness: 0.4, roughness: 0.7,
});
const bench = new THREE.Mesh(new THREE.BoxGeometry(6, 0.08, 1.6), benchMat);
bench.position.y = -0.04;
bench.receiveShadow = true;
benchGroup.add(bench);

const gridHelper = new THREE.GridHelper(6, 30, 0x262C38, 0x1A1F28);
gridHelper.position.y = 0.001;
benchGroup.add(gridHelper);

// Optical axis line (the bench's centerline, where object/mirror/image align)
const axisGeo = new THREE.BufferGeometry().setFromPoints([
  new THREE.Vector3(-3, 0.02, 0), new THREE.Vector3(3, 0.02, 0)
]);
const axisMat = new THREE.LineDashedMaterial({ color: 0x5A6270, dashSize: 0.08, gapSize: 0.05 });
const axisLine = new THREE.Line(axisGeo, axisMat);
axisLine.computeLineDistances();
benchGroup.add(axisLine);

/* ---------- The mirror itself, at x = 0 ---------- */
const mirrorGroup = new THREE.Group();
scene.add(mirrorGroup);

const mirrorFrameMat = new THREE.MeshStandardMaterial({
  color: 0x5A6270, metalness: 0.8, roughness: 0.3,
});
const mirrorFrame = new THREE.Mesh(
  new THREE.BoxGeometry(0.04, state.mirrorHeight + 0.08, state.mirrorWidth + 0.08),
  mirrorFrameMat
);
mirrorFrame.position.y = state.mirrorHeight / 2;
mirrorFrame.castShadow = true;
mirrorGroup.add(mirrorFrame);

// Reflective face: a real-time mirror using a Reflector-style approach is
// possible, but a stylized "glass" plane keeps performance high in VR and
// keeps focus on the geometric construction (rays + markers) rather than a
// literal raytraced reflection.
const mirrorGlassMat = new THREE.MeshPhysicalMaterial({
  color: 0x3DD9FF,
  metalness: 0.9,
  roughness: 0.05,
  transparent: true,
  opacity: 0.22,
  envMapIntensity: 1.5,
  side: THREE.DoubleSide,
});
const mirrorGlass = new THREE.Mesh(
  new THREE.PlaneGeometry(state.mirrorWidth, state.mirrorHeight),
  mirrorGlassMat
);
mirrorGlass.rotation.y = Math.PI / 2;
mirrorGlass.position.set(0.001, state.mirrorHeight / 2, 0);
mirrorGroup.add(mirrorGlass);

/* ---------- Object: the "real" thing on the +X side ---------- */
function buildFigure(color, emissiveColor) {
  const g = new THREE.Group();
  const mat = new THREE.MeshStandardMaterial({
    color, emissive: emissiveColor, emissiveIntensity: 0.25,
    roughness: 0.45, metalness: 0.1,
  });

  const headRadius = 0.09;
  const bodyHeight = state.objectHeight - headRadius * 2;

  const body = new THREE.Mesh(
    new THREE.CapsuleGeometry(0.07, Math.max(bodyHeight, 0.05), 6, 12),
    mat
  );
  body.position.y = bodyHeight / 2 + headRadius * 2 - 0.0; // base sits at y=0
  body.castShadow = true;
  g.add(body);

  const head = new THREE.Mesh(new THREE.SphereGeometry(headRadius, 20, 16), mat);
  head.position.y = bodyHeight + headRadius * 2;
  head.castShadow = true;
  g.add(head);

  // Base marker disc (helps depth perception on the bench grid)
  const baseDisc = new THREE.Mesh(
    new THREE.CircleGeometry(0.12, 24),
    new THREE.MeshBasicMaterial({ color: emissiveColor, transparent: true, opacity: 0.35 })
  );
  baseDisc.rotation.x = -Math.PI / 2;
  baseDisc.position.y = 0.003;
  g.add(baseDisc);

  g.userData.totalHeight = bodyHeight + headRadius * 2;
  return g;
}

const objectFigure = buildFigure(0xFF6A3D, 0x331100);
scene.add(objectFigure);

const imageFigure = buildFigure(0x3DD9FF, 0x002233);
// Image renders as translucent + wireframe-tinged to read as "virtual" (not solid/real)
imageFigure.traverse((child) => {
  if (child.isMesh && child.material && child.material.transparent !== true) {
    child.material = child.material.clone();
    child.material.transparent = true;
    child.material.opacity = 0.55;
  }
});
scene.add(imageFigure);

/* ---------- Distance marker group (the "signature" element) ---------- */
const markerGroup = new THREE.Group();
scene.add(markerGroup);

function makeTickLabel(text, color) {
  // Canvas-based billboard sprite — counter-rotates to always face the camera,
  // like a real readout floating in space above the optical bench.
  const canvas = document.createElement('canvas');
  canvas.width = 256; canvas.height = 96;
  const ctx = canvas.getContext('2d');
  const tex = new THREE.CanvasTexture(canvas);
  const mat = new THREE.SpriteMaterial({ map: tex, depthTest: false, transparent: true });
  const sprite = new THREE.Sprite(mat);
  sprite.scale.set(0.5, 0.19, 1);
  sprite.userData.draw = (txt) => {
    ctx.clearRect(0, 0, canvas.width, canvas.height);
    ctx.font = '600 40px "JetBrains Mono", monospace';
    ctx.fillStyle = color;
    ctx.textAlign = 'center';
    ctx.textBaseline = 'middle';
    ctx.fillText(txt, canvas.width / 2, canvas.height / 2);
    tex.needsUpdate = true;
  };
  sprite.userData.draw(text);
  return sprite;
}

const objTickLine = new THREE.Mesh(
  new THREE.BoxGeometry(1, 0.01, 0.01),
  new THREE.MeshBasicMaterial({ color: 0xFF6A3D })
);
const imgTickLine = new THREE.Mesh(
  new THREE.BoxGeometry(1, 0.01, 0.01),
  new THREE.MeshBasicMaterial({ color: 0x3DD9FF })
);
markerGroup.add(objTickLine, imgTickLine);

const objLabel = makeTickLabel('1.00 m', '#FF6A3D');
const imgLabel = makeTickLabel('1.00 m', '#3DD9FF');
markerGroup.add(objLabel, imgLabel);

/* ---------- Light ray group (object -> mirror -> eye, + virtual extension) */
const rayGroup = new THREE.Group();
scene.add(rayGroup);

function makeRay(color, dashed = false) {
  const geo = new THREE.BufferGeometry().setFromPoints([new THREE.Vector3(), new THREE.Vector3()]);
  const mat = dashed
    ? new THREE.LineDashedMaterial({ color, dashSize: 0.05, gapSize: 0.04, transparent: true, opacity: 0.85 })
    : new THREE.LineBasicMaterial({ color, transparent: true, opacity: 0.9 });
  const line = new THREE.Line(geo, mat);
  line.userData.dashed = dashed;
  return line;
}

// Two representative rays from the top of the object to two points on the
// mirror, then onward to the (approximate) eye position, plus their dashed
// backward extensions that converge at the virtual image.
const rays = {
  incidentA: makeRay(0xFFD08A),
  incidentB: makeRay(0xFFD08A),
  reflectedA: makeRay(0xFFD08A),
  reflectedB: makeRay(0xFFD08A),
  virtualA: makeRay(0x3DD9FF, true),
  virtualB: makeRay(0x3DD9FF, true),
};
Object.values(rays).forEach((r) => rayGroup.add(r));

function setLine(line, a, b) {
  const pos = line.geometry.attributes.position;
  pos.setXYZ(0, a.x, a.y, a.z);
  pos.setXYZ(1, b.x, b.y, b.z);
  pos.needsUpdate = true;
  line.geometry.computeBoundingSphere();
  if (line.userData.dashed) line.computeLineDistances();
}

/* ---------- Eye / viewer reference point for ray construction ---------- */
// In desktop mode this tracks the camera. In VR it tracks the head (XR camera).
const eyePoint = new THREE.Vector3();

/* =========================================================================
   CORE PHYSICS UPDATE — the only function that encodes mirror law.
   Plane mirror rule: image distance behind mirror == object distance in
   front; image height == object height; image is upright; not inverted.
   ========================================================================= */
function updateOptics() {
  const d = state.objectDistance;
  const h = state.objectHeight;

  objectFigure.position.set(d, 0, 0);
  imageFigure.position.set(-d, 0, 0); // exact mirror of object across x=0

  // ---- Readouts ----
  document.getElementById('readObj').textContent = `${d.toFixed(2)} m`;
  document.getElementById('readImg').textContent = `${d.toFixed(2)} m`;

  // ---- Distance markers (billboard ticks along the bench) ----
  markerGroup.visible = state.showMarkers;
  if (state.showMarkers) {
    objTickLine.scale.x = Math.max(d, 0.001);
    objTickLine.position.set(d / 2, 0.015, 0.5);
    imgTickLine.scale.x = Math.max(d, 0.001);
    imgTickLine.position.set(-d / 2, 0.015, 0.5);

    objLabel.position.set(d / 2, 0.12, 0.5);
    imgLabel.position.set(-d / 2, 0.12, 0.5);
    objLabel.userData.draw(`${d.toFixed(2)} m`);
    imgLabel.userData.draw(`${d.toFixed(2)} m`);
  }

  // ---- Guide lines (dashed projection from image back through mirror) ----
  // handled together with rays below for shared geometry.

  // ---- Light rays ----
  rayGroup.visible = state.showRays;
  if (state.showRays) {
    const objTop = new THREE.Vector3(d, h, 0);
    const imgTop = new THREE.Vector3(-d, h, 0);

    // Two mirror-intersection points (upper and lower on the mirror face)
    // chosen so reflected rays plausibly converge toward the eye.
    const mirrorY_A = state.mirrorHeight * 0.78;
    const mirrorY_B = state.mirrorHeight * 0.45;
    const mirrorPtA = new THREE.Vector3(0, mirrorY_A, lerpZ(objTop, eyePoint, mirrorY_A));
    const mirrorPtB = new THREE.Vector3(0, mirrorY_B, lerpZ(objTop, eyePoint, mirrorY_B));

    setLine(rays.incidentA, objTop, mirrorPtA);
    setLine(rays.incidentB, objTop, mirrorPtB);
    setLine(rays.reflectedA, mirrorPtA, eyePoint);
    setLine(rays.reflectedB, mirrorPtB, eyePoint);

    // Dashed virtual extensions: straight line from image point through the
    // same mirror point — this is the "your brain thinks the light came
    // from back there" construction.
    setLine(rays.virtualA, imgTop, mirrorPtA);
    setLine(rays.virtualB, imgTop, mirrorPtB);
  }

  // Guide lines: simple dashed vertical + connecting line emphasizing
  // object-height == image-height and the mirror-plane symmetry.
  guideLines.visible = state.showGuides;
  if (state.showGuides) {
    setLine(guideTop, new THREE.Vector3(d, h, 0), new THREE.Vector3(-d, h, 0));
    setLine(guideBase, new THREE.Vector3(d, 0.001, 0), new THREE.Vector3(-d, 0.001, 0));
  }
}

// Helper: find a z-value along the line from `from` toward eyePoint, at a
// given y, used only to give the two illustrative rays slight visual spread.
function lerpZ(from, eye, atY) {
  const t = (atY - from.y) / ((eye.y - from.y) || 0.0001);
  const z = from.z + (eye.z - from.z) * THREE.MathUtils.clamp(t, 0.05, 0.95);
  return THREE.MathUtils.clamp(z, -0.4, 0.4);
}

/* ---------- Guide lines geometry ---------- */
const guideLines = new THREE.Group();
scene.add(guideLines);
const guideMat = new THREE.LineDashedMaterial({ color: 0x8B92A0, dashSize: 0.04, gapSize: 0.03, transparent: true, opacity: 0.6 });
function makeGuide() {
  const geo = new THREE.BufferGeometry().setFromPoints([new THREE.Vector3(), new THREE.Vector3()]);
  return new THREE.Line(geo, guideMat);
}
const guideTop = makeGuide();
const guideBase = makeGuide();
guideLines.add(guideTop, guideBase);

/* =========================================================================
   INTERACTION — drag the object along X on desktop/mobile
   ========================================================================= */
const raycaster = new THREE.Raycaster();
const pointer = new THREE.Vector2();

function pointerToWorldX(clientX, clientY) {
  pointer.x = (clientX / window.innerWidth) * 2 - 1;
  pointer.y = -(clientY / window.innerHeight) * 2 + 1;
  raycaster.setFromCamera(pointer, camera);
  const planeY = new THREE.Plane(new THREE.Vector3(0, 1, 0), 0); // ground plane y=0
  const hit = new THREE.Vector3();
  raycaster.ray.intersectPlane(planeY, hit);
  return hit ? hit.x : null;
}

function onPointerDown(e) {
  const x = e.touches ? e.touches[0].clientX : e.clientX;
  const y = e.touches ? e.touches[0].clientY : e.clientY;
  pointer.x = (x / window.innerWidth) * 2 - 1;
  pointer.y = -(y / window.innerHeight) * 2 + 1;
  raycaster.setFromCamera(pointer, camera);
  const hits = raycaster.intersectObject(objectFigure, true);
  if (hits.length) {
    state.dragging = true;
    document.getElementById('hint').style.opacity = '0';
  }
}
function onPointerMove(e) {
  if (!state.dragging) return;
  const x = e.touches ? e.touches[0].clientX : e.clientX;
  const y = e.touches ? e.touches[0].clientY : e.clientY;
  const worldX = pointerToWorldX(x, y);
  if (worldX !== null) {
    state.objectDistance = THREE.MathUtils.clamp(worldX, 0.3, 3.0);
    distSlider.value = state.objectDistance.toFixed(2);
    updateOptics();
  }
}
function onPointerUp() { state.dragging = false; }

renderer.domElement.addEventListener('mousedown', onPointerDown);
renderer.domElement.addEventListener('mousemove', onPointerMove);
window.addEventListener('mouseup', onPointerUp);
renderer.domElement.addEventListener('touchstart', onPointerDown, { passive: true });
renderer.domElement.addEventListener('touchmove', onPointerMove, { passive: true });
renderer.domElement.addEventListener('touchend', onPointerUp);

/* ---------- Slider + toggle wiring ---------- */
const distSlider = document.getElementById('distSlider');
distSlider.addEventListener('input', () => {
  state.objectDistance = parseFloat(distSlider.value);
  updateOptics();
});

function wireToggle(id, key) {
  const el = document.getElementById(id);
  el.addEventListener('click', () => {
    state[key] = !state[key];
    el.classList.toggle('on', state[key]);
    updateOptics();
  });
}
wireToggle('toggleRays', 'showRays');
wireToggle('toggleMarkers', 'showMarkers');
wireToggle('toggleGuides', 'showGuides');

/* =========================================================================
   WEBXR — VR mode, head tracking, parallax
   ========================================================================= */
const vrButton = document.getElementById('vrButton');
const vrNotice = document.getElementById('vrNotice');

if ('xr' in navigator) {
  navigator.xr.isSessionSupported('immersive-vr').then((supported) => {
    if (supported) {
      // VRButton.createButton() builds AND APPENDS its own button to
      // document.body, and owns the actual session request/end lifecycle.
      // We keep that button (for its lifecycle logic + headset prompt
      // handling) but hide it visually, and forward clicks from our own
      // styled lab-UI button into it.
      const builtIn = VRButton.createButton(renderer);
      builtIn.style.display = 'none';
      document.body.appendChild(builtIn);
      vrButton.addEventListener('click', () => builtIn.dispatchEvent(new MouseEvent('click')));
    } else {
      vrButton.disabled = true;
      vrButton.textContent = 'VR Not Supported';
      vrNotice.style.display = 'block';
      setTimeout(() => (vrNotice.style.display = 'none'), 6000);
    }
  }).catch(() => {
    vrButton.disabled = true;
    vrButton.textContent = 'VR Check Failed';
    vrNotice.style.display = 'block';
  });
} else {
  vrButton.disabled = true;
  vrButton.textContent = 'WebXR Unavailable';
  vrNotice.style.display = 'block';
}

renderer.xr.addEventListener('sessionstart', () => {
  // Reposition the rig so the player starts standing at a sensible spot
  // on the bench, facing the mirror, in VR's room-scale local space.
  cameraRig.position.set(2.2, 0, 1.4);
  cameraRig.rotation.y = -Math.PI / 2.4;
  document.getElementById('topbar').style.display = 'none';
  document.getElementById('dock').style.display = 'none';
});
renderer.xr.addEventListener('sessionend', () => {
  cameraRig.position.set(0, 0, 0);
  cameraRig.rotation.set(0, 0, 0);
  document.getElementById('topbar').style.display = '';
  document.getElementById('dock').style.display = '';
});

/* =========================================================================
   ANIMATION LOOP — updates eyePoint from whichever camera is "live"
   (desktop perspective camera, or the XR camera once a session starts),
   so light-ray construction always points at the viewer's actual head
   position. This is the head-tracking -> physics-visual link.
   ========================================================================= */
function getActiveCameraWorldPosition(target) {
  if (renderer.xr.isPresenting) {
    const xrCam = renderer.xr.getCamera();
    xrCam.getWorldPosition(target);
  } else {
    camera.getWorldPosition(target);
  }
  return target;
}

let lastEyeUpdate = 0;
function animate() {
  getActiveCameraWorldPosition(eyePoint);

  // Recompute optics-dependent visuals only when something meaningfully
  // changed (object moved, or eye moved enough to matter) — keeps VR frame
  // budget tight on standalone headsets.
  lastEyeUpdate += 1;
  if (lastEyeUpdate % 2 === 0) updateOptics();

  // Billboard the distance-marker sprites toward the active camera.
  const camPos = renderer.xr.isPresenting ? eyePoint : camera.position;
  [objLabel, imgLabel].forEach((s) => s.lookAt(camPos));

  renderer.render(scene, camera);
}
renderer.setAnimationLoop(animate);

/* ---------- Resize handling ---------- */
window.addEventListener('resize', () => {
  camera.aspect = window.innerWidth / window.innerHeight;
  camera.updateProjectionMatrix();
  renderer.setSize(window.innerWidth, window.innerHeight);
});

/* ---------- Initial paint ---------- */
updateOptics();
</script>
</body>
</html>
