# gioco1
<!DOCTYPE html>
<html lang="it">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0, user-scalable=no">
<meta name="apple-mobile-web-app-capable" content="yes">
<meta name="apple-mobile-web-app-status-bar-style" content="black-translucent">
<meta name="apple-mobile-web-app-title" content="Torino Drive">
<title>Torino Drive 🏎️</title>
<style>
  * { margin: 0; padding: 0; box-sizing: border-box; }
  body { background: #0a0a0f; overflow: hidden; font-family: 'Arial', sans-serif; touch-action: none; }
  canvas { display: block; }

  #splash {
    position: fixed; inset: 0; z-index: 100;
    background: linear-gradient(135deg, #0a0a1a 0%, #1a0a2e 50%, #0d1a2e 100%);
    display: flex; flex-direction: column; align-items: center; justify-content: center;
    transition: opacity 0.8s ease;
  }
  #splash.hidden { opacity: 0; pointer-events: none; }

  .splash-logo {
    font-size: clamp(3rem, 10vw, 6rem);
    font-weight: 900;
    letter-spacing: -2px;
    background: linear-gradient(90deg, #e8b84b, #ff6b35, #e8b84b);
    background-size: 200%;
    -webkit-background-clip: text;
    -webkit-text-fill-color: transparent;
    animation: shimmer 2s infinite linear;
    text-align: center;
    line-height: 1;
  }
  .splash-sub {
    color: #7a9bbf;
    font-size: clamp(0.9rem, 3vw, 1.2rem);
    letter-spacing: 6px;
    text-transform: uppercase;
    margin-top: 0.5rem;
  }
  .splash-tagline {
    color: #4a6a8f;
    font-size: 0.85rem;
    margin-top: 1.5rem;
    letter-spacing: 2px;
    text-transform: uppercase;
  }
  .start-btn {
    margin-top: 3rem;
    padding: 1rem 3rem;
    background: linear-gradient(135deg, #e8b84b, #ff6b35);
    border: none; border-radius: 50px;
    color: #0a0a1a; font-size: 1.1rem; font-weight: 800;
    letter-spacing: 3px; text-transform: uppercase;
    cursor: pointer;
    box-shadow: 0 0 30px rgba(232,184,75,0.4);
    transition: transform 0.15s, box-shadow 0.15s;
    -webkit-tap-highlight-color: transparent;
  }
  .start-btn:active { transform: scale(0.96); box-shadow: 0 0 15px rgba(232,184,75,0.2); }

  .install-hint {
    margin-top: 1.5rem;
    color: #3a5a7f;
    font-size: 0.75rem;
    text-align: center;
    line-height: 1.6;
    max-width: 280px;
  }
  .install-hint span { color: #7a9bbf; }

  @keyframes shimmer { 0%{background-position:0%} 100%{background-position:200%} }

  /* HUD */
  #hud {
    position: fixed; inset: 0; pointer-events: none; z-index: 10;
    display: none;
  }
  #hud.active { display: block; }

  .speed-panel {
    position: absolute; bottom: 130px; right: 16px;
    background: rgba(0,0,0,0.65);
    border: 1px solid rgba(232,184,75,0.3);
    border-radius: 16px;
    padding: 10px 18px;
    text-align: center;
    backdrop-filter: blur(10px);
  }
  .speed-num {
    font-size: 2.8rem; font-weight: 900;
    color: #e8b84b; line-height: 1;
    font-variant-numeric: tabular-nums;
  }
  .speed-unit { color: #7a9bbf; font-size: 0.7rem; letter-spacing: 2px; }

  .gear-panel {
    position: absolute; bottom: 130px; right: 120px;
    background: rgba(0,0,0,0.55);
    border: 1px solid rgba(255,255,255,0.1);
    border-radius: 12px;
    padding: 8px 14px;
    text-align: center;
    backdrop-filter: blur(8px);
  }
  .gear-num { font-size: 1.8rem; font-weight: 900; color: #fff; }
  .gear-label { color: #7a9bbf; font-size: 0.6rem; letter-spacing: 2px; }

  .location-badge {
    position: absolute; top: 20px; left: 50%; transform: translateX(-50%);
    background: rgba(0,0,0,0.6);
    border: 1px solid rgba(232,184,75,0.3);
    border-radius: 20px;
    padding: 6px 18px;
    color: #e8b84b;
    font-size: 0.75rem;
    letter-spacing: 2px;
    text-transform: uppercase;
    backdrop-filter: blur(8px);
  }

  .minimap {
    position: absolute; top: 16px; right: 16px;
    width: 110px; height: 110px;
    background: rgba(0,0,0,0.7);
    border: 1px solid rgba(232,184,75,0.3);
    border-radius: 12px;
    overflow: hidden;
    backdrop-filter: blur(8px);
  }
  #minimapCanvas { width: 100%; height: 100%; }

  .rpm-bar-wrap {
    position: absolute; bottom: 105px; left: 50%; transform: translateX(-50%);
    width: min(220px, 60vw);
    display: flex; flex-direction: column; align-items: center; gap: 4px;
  }
  .rpm-label { color: #4a6a8f; font-size: 0.6rem; letter-spacing: 2px; }
  .rpm-track {
    width: 100%; height: 6px;
    background: rgba(255,255,255,0.08);
    border-radius: 3px; overflow: hidden;
  }
  .rpm-fill {
    height: 100%; width: 0%;
    background: linear-gradient(90deg, #2aff7a, #e8b84b, #ff3a3a);
    border-radius: 3px;
    transition: width 0.05s;
  }

  /* Touch controls */
  #controls {
    position: fixed; bottom: 0; left: 0; right: 0;
    height: 120px;
    display: none;
    pointer-events: auto;
    z-index: 20;
    padding: 10px 16px;
  }
  #controls.active { display: flex; align-items: center; justify-content: space-between; }

  .ctrl-group { display: flex; gap: 10px; align-items: center; }

  .ctrl-btn {
    width: 56px; height: 56px;
    background: rgba(255,255,255,0.08);
    border: 1.5px solid rgba(255,255,255,0.18);
    border-radius: 14px;
    display: flex; align-items: center; justify-content: center;
    font-size: 1.4rem;
    color: #fff;
    -webkit-tap-highlight-color: transparent;
    user-select: none;
    transition: background 0.1s;
    cursor: pointer;
  }
  .ctrl-btn.active-btn { background: rgba(232,184,75,0.25); border-color: rgba(232,184,75,0.6); }
  .ctrl-btn.gas { background: rgba(46,213,115,0.15); border-color: rgba(46,213,115,0.4); width: 70px; height: 70px; font-size: 1.8rem; }
  .ctrl-btn.brake { background: rgba(255,71,87,0.15); border-color: rgba(255,71,87,0.4); width: 58px; height: 58px; }
  .ctrl-btn.gas.active-btn { background: rgba(46,213,115,0.35); }
  .ctrl-btn.brake.active-btn { background: rgba(255,71,87,0.35); }

  .steer-track {
    width: 160px; height: 56px;
    background: rgba(255,255,255,0.05);
    border: 1.5px solid rgba(255,255,255,0.12);
    border-radius: 28px;
    position: relative;
    overflow: hidden;
  }
  .steer-knob {
    width: 52px; height: 48px;
    background: rgba(232,184,75,0.2);
    border: 1.5px solid rgba(232,184,75,0.5);
    border-radius: 24px;
    position: absolute;
    top: 3px; left: 54px;
    transition: left 0.06s;
    display: flex; align-items: center; justify-content: center;
    font-size: 1.2rem;
  }
</style>
</head>
<body>

<!-- SPLASH -->
<div id="splash">
  <div class="splash-logo">TORINO<br>DRIVE</div>
  <div class="splash-sub">Città di Torino</div>
  <div class="splash-tagline">Open World • 3D • Free Roam</div>
  <button class="start-btn" id="startBtn">▶ INIZIA</button>
  <div class="install-hint">
    📲 Per installare su iPhone:<br>
    <span>Safari → Condividi → "Aggiungi a schermata Home"</span>
  </div>
</div>

<!-- HUD -->
<div id="hud">
  <div class="location-badge" id="locationBadge">🏔️ Piazza Castello</div>
  <div class="minimap"><canvas id="minimapCanvas" width="110" height="110"></canvas></div>
  <div class="speed-panel">
    <div class="speed-num" id="speedNum">0</div>
    <div class="speed-unit">KM/H</div>
  </div>
  <div class="gear-panel">
    <div class="gear-num" id="gearNum">N</div>
    <div class="gear-label">MARCIA</div>
  </div>
  <div class="rpm-bar-wrap">
    <div class="rpm-label">RPM</div>
    <div class="rpm-track"><div class="rpm-fill" id="rpmFill"></div></div>
  </div>
</div>

<!-- TOUCH CONTROLS -->
<div id="controls">
  <div class="ctrl-group">
    <div class="ctrl-btn" id="btnLeft" data-key="left">◀</div>
    <div class="steer-track" id="steerTrack">
      <div class="steer-knob" id="steerKnob">🚗</div>
    </div>
    <div class="ctrl-btn" id="btnRight" data-key="right">▶</div>
  </div>
  <div class="ctrl-group" style="flex-direction:column; gap:12px;">
    <div class="ctrl-btn gas" id="btnGas">⬆</div>
    <div class="ctrl-btn brake" id="btnBrake">⬇</div>
  </div>
</div>

<script type="importmap">
{
  "imports": {
    "three": "https://cdnjs.cloudflare.com/ajax/libs/three.js/r128/three.module.js"
  }
}
</script>

<script type="module">
import * as THREE from 'three';

// ─── STATE ───────────────────────────────────────────────────
const keys = { up:false, down:false, left:false, right:false };
let gameStarted = false;

// ─── SCENE SETUP ─────────────────────────────────────────────
const scene = new THREE.Scene();
scene.background = new THREE.Color(0x87CEEB);
scene.fog = new THREE.FogExp2(0xc0d8f0, 0.018);

const renderer = new THREE.WebGLRenderer({ antialias: true, powerPreference:'high-performance' });
renderer.setPixelRatio(Math.min(devicePixelRatio, 2));
renderer.setSize(innerWidth, innerHeight);
renderer.shadowMap.enabled = true;
renderer.shadowMap.type = THREE.PCFSoftShadowMap;
document.body.appendChild(renderer.domElement);

const camera = new THREE.PerspectiveCamera(65, innerWidth/innerHeight, 0.1, 400);
camera.position.set(0, 4, -10);

// ─── LIGHTING ─────────────────────────────────────────────────
const ambient = new THREE.AmbientLight(0xfff5e0, 0.7);
scene.add(ambient);

const sun = new THREE.DirectionalLight(0xfffde7, 1.4);
sun.position.set(80, 120, 60);
sun.castShadow = true;
sun.shadow.mapSize.set(2048, 2048);
sun.shadow.camera.near = 0.1;
sun.shadow.camera.far = 400;
sun.shadow.camera.left = -150; sun.shadow.camera.right = 150;
sun.shadow.camera.top = 150; sun.shadow.camera.bottom = -150;
scene.add(sun);

const hemi = new THREE.HemisphereLight(0x87ceeb, 0x556B2F, 0.4);
scene.add(hemi);

// ─── TORINO MAP LAYOUT ────────────────────────────────────────
// Inspired by real Turin grid: Corso Vittorio Emanuele II axis,
// Via Roma, Po riverbank, Piazza Castello, etc.

const ROAD_W = 7;
const BLD_COLORS = [0xd4a574, 0xc4956a, 0xe8c99a, 0xb8825a, 0xddb88a, 0xa0785a, 0xcc9f7a];
const ROOF_COLORS = [0x8B4513, 0x6B3410, 0xa05020, 0x7a3a18, 0x954820];

// Road network: [x1,z1,x2,z2]
const roads = [
  // Corso Vittorio Emanuele II (E-W main)
  [-200,0,200,0],
  // Via Roma (N-S main)
  [0,-200,0,200],
  // Corso Re Umberto
  [-200,30,200,30],
  // Corso Galileo Ferraris
  [-200,-30,200,-30],
  // Corso Oporto
  [-200,60,200,60],
  // Via Nizza
  [-200,-60,200,-60],
  // Corso Matteotti
  [40,-200,40,200],
  // Via Lagrange
  [-40,-200,-40,200],
  // Corso Massimo d'Azeglio
  [80,-200,80,200],
  // Via Sacchi
  [-80,-200,-80,200],
  // Po riverbank
  [120,-200,120,200],
  // Corso Galileo Galilei
  [-120,-200,-120,200],
  // Diagonali piemontesi
  [-200,-200,200,200], // Corso Francia style diagonal
];

// ─── GROUND ──────────────────────────────────────────────────
const groundGeo = new THREE.PlaneGeometry(500, 500);
const groundMat = new THREE.MeshLambertMaterial({ color: 0x5a8a3a });
const ground = new THREE.Mesh(groundGeo, groundMat);
ground.rotation.x = -Math.PI/2;
ground.receiveShadow = true;
scene.add(ground);

// ─── ROAD SURFACE ────────────────────────────────────────────
function makeRoad(x1,z1,x2,z2) {
  const dx = x2-x1, dz = z2-z1;
  const len = Math.sqrt(dx*dx+dz*dz);
  const geo = new THREE.PlaneGeometry(ROAD_W, len);
  const mat = new THREE.MeshLambertMaterial({ color: 0x2a2a2a });
  const mesh = new THREE.Mesh(geo, mat);
  mesh.rotation.x = -Math.PI/2;
  mesh.rotation.z = Math.atan2(dx, dz);
  mesh.position.set((x1+x2)/2, 0.02, (z1+z2)/2);
  mesh.receiveShadow = true;
  scene.add(mesh);

  // Center line dashes
  const dashCount = Math.floor(len / 8);
  for (let i = 0; i < dashCount; i++) {
    const t = (i+0.5)/dashCount;
    const px = x1 + dx*t, pz = z1 + dz*t;
    const dGeo = new THREE.PlaneGeometry(0.25, 3);
    const dMat = new THREE.MeshLambertMaterial({ color: 0xffdd00 });
    const dash = new THREE.Mesh(dGeo, dMat);
    dash.rotation.x = -Math.PI/2;
    dash.rotation.z = Math.atan2(dx, dz);
    dash.position.set(px, 0.03, pz);
    scene.add(dash);
  }
}

roads.forEach(r => makeRoad(r[0],r[1],r[2],r[3]));

// Intersections (smooth)
function makeIntersection(x,z,size=ROAD_W+1) {
  const geo = new THREE.PlaneGeometry(size, size);
  const mat = new THREE.MeshLambertMaterial({ color: 0x2a2a2a });
  const mesh = new THREE.Mesh(geo, mat);
  mesh.rotation.x = -Math.PI/2;
  mesh.position.set(x, 0.025, z);
  mesh.receiveShadow = true;
  scene.add(mesh);
}
// Major intersections
const intPoints = [];
for(let x of [-80,-40,0,40,80,120]) for(let z of [-60,-30,0,30,60]) intPoints.push([x,z]);
intPoints.forEach(([x,z])=>makeIntersection(x,z));

// ─── BUILDINGS ───────────────────────────────────────────────
function makeBuilding(x,z,w,d,h, colorIdx) {
  const geo = new THREE.BoxGeometry(w,h,d);
  const mat = new THREE.MeshLambertMaterial({ color: BLD_COLORS[colorIdx % BLD_COLORS.length] });
  const mesh = new THREE.Mesh(geo, mat);
  mesh.position.set(x, h/2, z);
  mesh.castShadow = true;
  mesh.receiveShadow = true;
  scene.add(mesh);

  // Roof
  const rGeo = new THREE.BoxGeometry(w+0.3, 0.6, d+0.3);
  const rMat = new THREE.MeshLambertMaterial({ color: ROOF_COLORS[colorIdx % ROOF_COLORS.length] });
  const roof = new THREE.Mesh(rGeo, rMat);
  roof.position.set(x, h+0.3, z);
  roof.castShadow = true;
  scene.add(roof);

  // Windows (simple planes)
  if(h > 5) {
    const wMat = new THREE.MeshLambertMaterial({ color: 0x88aacc, emissive:0x112233, emissiveIntensity:0.3 });
    const floors = Math.floor(h/3);
    for(let fl=1; fl<=floors; fl++) {
      for(let wi=0; wi<2; wi++) {
        const wGeo = new THREE.PlaneGeometry(1.2, 1.5);
        const win = new THREE.Mesh(wGeo, wMat);
        win.position.set(x + (wi===0?-w/4:w/4), fl*3-1, z + d/2 + 0.01);
        scene.add(win);
        const win2 = new THREE.Mesh(wGeo, wMat);
        win2.position.set(x + (wi===0?-w/4:w/4), fl*3-1, z - d/2 - 0.01);
        win2.rotation.y = Math.PI;
        scene.add(win2);
      }
    }
  }
}

// City blocks between roads
const rng = (min,max)=>Math.random()*(max-min)+min;
let bldIdx=0;
// Fill city blocks
const blockPositions = [
  // Between Via Roma and Corso Matteotti, various latitudes
  [20,-50,14,10,rng(8,18)], [20,-70,14,10,rng(6,14)],[20,-80,14,10,rng(10,20)],
  [20,10,14,10,rng(8,18)],[20,20,14,10,rng(6,14)],[20,50,14,10,rng(10,20)],
  [20,70,14,10,rng(8,16)],[20,90,14,10,rng(10,18)],
  [-20,-50,14,10,rng(8,18)],[-20,-70,14,10,rng(6,14)],[-20,-80,14,10,rng(10,20)],
  [-20,10,14,10,rng(8,18)],[-20,20,14,10,rng(6,14)],[-20,50,14,10,rng(10,20)],
  [60,-45,14,10,rng(10,22)],[60,-70,14,10,rng(8,16)],[60,-80,14,10,rng(12,20)],
  [60,15,14,10,rng(8,18)],[60,50,14,10,rng(10,22)],[60,70,14,10,rng(8,16)],
  [-60,-45,14,10,rng(10,22)],[-60,-70,14,10,rng(8,16)],[-60,-80,14,10,rng(12,20)],
  [-60,15,14,10,rng(8,18)],[-60,50,14,10,rng(10,22)],[-60,70,14,10,rng(8,16)],
  [100,-45,12,10,rng(6,16)],[100,15,12,10,rng(8,18)],[100,50,12,10,rng(6,14)],
  [-100,-45,12,10,rng(6,16)],[-100,15,12,10,rng(8,18)],[-100,50,12,10,rng(6,14)],
  // Extra blocks
  [20,-100,14,10,rng(8,16)],[-20,-100,14,10,rng(8,16)],
  [60,-100,14,10,rng(10,20)],[-60,-100,14,10,rng(10,20)],
  [20,100,14,10,rng(8,16)],[-20,100,14,10,rng(8,16)],
  [60,100,14,10,rng(10,20)],[-60,100,14,10,rng(10,20)],
];
blockPositions.forEach(([x,z,w,d,h])=>{ makeBuilding(x,z,w,d,h,bldIdx++); });

// Random extra buildings in outer districts
for(let i=0;i<80;i++){
  const x = (Math.random()-0.5)*220;
  const z = (Math.random()-0.5)*220;
  // avoid road centers
  if(Math.abs(x)<5||Math.abs(z)<5||Math.abs(x-40)<5||Math.abs(x+40)<5) continue;
  makeBuilding(x,z,rng(6,16),rng(6,12),rng(4,20),bldIdx++);
}

// ─── LANDMARKS ───────────────────────────────────────────────

// Mole Antonelliana (iconic Turin tower)
function makeMole(x,z) {
  const base = new THREE.Mesh(new THREE.BoxGeometry(8,14,8),
    new THREE.MeshLambertMaterial({color:0xf5e6c8}));
  base.position.set(x,7,z); base.castShadow=true; scene.add(base);
  const mid = new THREE.Mesh(new THREE.BoxGeometry(4,20,4),
    new THREE.MeshLambertMaterial({color:0xf0ddb0}));
  mid.position.set(x,24,z); mid.castShadow=true; scene.add(mid);
  const narrow = new THREE.Mesh(new THREE.BoxGeometry(2,15,2),
    new THREE.MeshLambertMaterial({color:0xead9a0}));
  narrow.position.set(x,41,z); narrow.castShadow=true; scene.add(narrow);
  // Spire
  const sGeo = new THREE.CylinderGeometry(0.1,1,18,8);
  const spire = new THREE.Mesh(sGeo, new THREE.MeshLambertMaterial({color:0xd4c090}));
  spire.position.set(x,57,z); spire.castShadow=true; scene.add(spire);
  // Star on top
  const starGeo = new THREE.SphereGeometry(0.6,8,8);
  const star = new THREE.Mesh(starGeo, new THREE.MeshLambertMaterial({color:0xffd700,emissive:0xffaa00,emissiveIntensity:0.5}));
  star.position.set(x,67,z); scene.add(star);
}
makeMole(90, -85);

// Palazzo Reale (Royal Palace) near Piazza Castello
function makePalazzoReale(x,z) {
  const main = new THREE.Mesh(new THREE.BoxGeometry(30,12,16),
    new THREE.MeshLambertMaterial({color:0xf2e0b0}));
  main.position.set(x,6,z); main.castShadow=true; scene.add(main);
  const roof2 = new THREE.Mesh(new THREE.BoxGeometry(30.5,1.5,16.5),
    new THREE.MeshLambertMaterial({color:0x8B6914}));
  roof2.position.set(x,12.75,z); scene.add(roof2);
  // Columns
  for(let ci=-6;ci<=6;ci+=2) {
    const col = new THREE.Mesh(new THREE.CylinderGeometry(0.3,0.35,11,8),
      new THREE.MeshLambertMaterial({color:0xf8f0d8}));
    col.position.set(x+ci*2,5.5,z+8); col.castShadow=true; scene.add(col);
  }
}
makePalazzoReale(-5,-88);

// Piazza Castello (central plaza)
const pGeo = new THREE.PlaneGeometry(28,28);
const pMat = new THREE.MeshLambertMaterial({color:0xe8d8b8});
const piazza = new THREE.Mesh(pGeo, pMat);
piazza.rotation.x=-Math.PI/2; piazza.position.set(0,-80,0.04); scene.add(piazza);

// Po River (blue strip on east side)
const riverGeo = new THREE.PlaneGeometry(18,500);
const riverMat = new THREE.MeshLambertMaterial({color:0x4a90d9,emissive:0x1a4a8a,emissiveIntensity:0.2});
const river = new THREE.Mesh(riverGeo, riverMat);
river.rotation.x=-Math.PI/2; river.position.set(145,0.01,0); scene.add(river);

// Superga hill in distance
const hillGeo = new THREE.ConeGeometry(60,35,12);
const hillMat = new THREE.MeshLambertMaterial({color:0x3a6a2a});
const hill = new THREE.Mesh(hillGeo, hillMat);
hill.position.set(180,17,-120); scene.add(hill);
// Basilica di Superga on hill
const basilica = new THREE.Mesh(new THREE.CylinderGeometry(4,4,12,12),
  new THREE.MeshLambertMaterial({color:0xf0d8b0}));
basilica.position.set(180,44,-120); scene.add(basilica);

// ─── TREES ───────────────────────────────────────────────────
function makeTree(x,z) {
  const trunk = new THREE.Mesh(
    new THREE.CylinderGeometry(0.15,0.2,2.5,6),
    new THREE.MeshLambertMaterial({color:0x5a3a1a})
  );
  trunk.position.set(x,1.25,z); trunk.castShadow=true; scene.add(trunk);
  const crown = new THREE.Mesh(
    new THREE.SphereGeometry(1.5+Math.random(),7,7),
    new THREE.MeshLambertMaterial({color: Math.random()>0.5 ? 0x2d7a2d : 0x3a8a3a})
  );
  crown.position.set(x,4+Math.random(),z); crown.castShadow=true; scene.add(crown);
}

// Corso Vittorio trees
for(let x=-180;x<=180;x+=12) { makeTree(x, 5); makeTree(x, -5); }
// Via Roma trees
for(let z=-180;z<=180;z+=12) { makeTree(5, z); makeTree(-5, z); }
// Random park trees
for(let i=0;i<60;i++) {
  const tx=(Math.random()-0.5)*180, tz=(Math.random()-0.5)*180;
  if(Math.abs(tx)>10&&Math.abs(tz)>10) makeTree(tx,tz);
}

// ─── STREET LIGHTS ───────────────────────────────────────────
function makeStreetLight(x,z,ry=0) {
  const pole = new THREE.Mesh(new THREE.CylinderGeometry(0.08,0.1,6,6),
    new THREE.MeshLambertMaterial({color:0x555566}));
  pole.position.set(x,3,z); scene.add(pole);
  const arm = new THREE.Mesh(new THREE.BoxGeometry(1.5,0.1,0.1),
    new THREE.MeshLambertMaterial({color:0x555566}));
  arm.position.set(x+0.75*Math.cos(ry),6,z+0.75*Math.sin(ry)); scene.add(arm);
  const bulb = new THREE.Mesh(new THREE.SphereGeometry(0.2,6,6),
    new THREE.MeshLambertMaterial({color:0xffffaa,emissive:0xffffaa,emissiveIntensity:1}));
  bulb.position.set(x+1.5*Math.cos(ry),6,z+1.5*Math.sin(ry)); scene.add(bulb);
}
for(let x=-160;x<=160;x+=20) { makeStreetLight(x,4.5); makeStreetLight(x,-4.5,Math.PI); }
for(let z=-160;z<=160;z+=20) { makeStreetLight(4.5,z,Math.PI/2); makeStreetLight(-4.5,z,-Math.PI/2); }

// ─── CAR ─────────────────────────────────────────────────────
const car = new THREE.Group();
scene.add(car);

// Body
const bodyGeo = new THREE.BoxGeometry(2.0, 0.7, 4.4);
const bodyMat = new THREE.MeshLambertMaterial({color:0xff3a3a});
const body = new THREE.Mesh(bodyGeo, bodyMat);
body.position.y = 0.5; body.castShadow=true; car.add(body);

// Cabin
const cabinGeo = new THREE.BoxGeometry(1.6,0.6,2.0);
const cabinMat = new THREE.MeshLambertMaterial({color:0xcc2020});
const cabin = new THREE.Mesh(cabinGeo, cabinMat);
cabin.position.set(0,1.05,-0.2); cabin.castShadow=true; car.add(cabin);

// Windshield
const wsGeo = new THREE.PlaneGeometry(1.5,0.5);
const wsMat = new THREE.MeshLambertMaterial({color:0x88ccff,transparent:true,opacity:0.6,side:THREE.DoubleSide});
const ws = new THREE.Mesh(wsGeo,wsMat);
ws.position.set(0,1.1,0.82); ws.rotation.x=-0.3; car.add(ws);

// Spoiler
const spoilerGeo = new THREE.BoxGeometry(2.1,0.1,0.3);
const spoilerMat = new THREE.MeshLambertMaterial({color:0x990000});
const spoiler = new THREE.Mesh(spoilerGeo,spoilerMat);
spoiler.position.set(0,1.1,-2.1); car.add(spoiler);

// Wheels
const wheelGeo = new THREE.CylinderGeometry(0.38,0.38,0.28,16);
const wheelMat = new THREE.MeshLambertMaterial({color:0x111111});
const rimGeo = new THREE.CylinderGeometry(0.22,0.22,0.29,8);
const rimMat = new THREE.MeshLambertMaterial({color:0xcccccc});

const wheelPositions = [
  [0.95,0.38,1.4],[-0.95,0.38,1.4],
  [0.95,0.38,-1.4],[-0.95,0.38,-1.4]
];
const wheels = [];
wheelPositions.forEach(([x,y,z]) => {
  const wg = new THREE.Group();
  const w = new THREE.Mesh(wheelGeo, wheelMat);
  w.rotation.z=Math.PI/2;
  const r = new THREE.Mesh(rimGeo, rimMat);
  r.rotation.z=Math.PI/2;
  wg.add(w); wg.add(r);
  wg.position.set(x,y,z);
  car.add(wg);
  wheels.push(wg);
});

// Headlights
const hlMat = new THREE.MeshLambertMaterial({color:0xffffee,emissive:0xffffaa,emissiveIntensity:0.8});
[[-0.6,0.55,2.21],[0.6,0.55,2.21]].forEach(([x,y,z])=>{
  const hl = new THREE.Mesh(new THREE.BoxGeometry(0.5,0.2,0.05),hlMat);
  hl.position.set(x,y,z); car.add(hl);
  const light = new THREE.SpotLight(0xffffcc,1.2,30,0.3);
  light.position.set(x,y,z+0.1);
  light.target.position.set(x,y-0.3,z+15);
  car.add(light); car.add(light.target);
});

// Taillights
const tlMat = new THREE.MeshLambertMaterial({color:0xff0000,emissive:0xff0000,emissiveIntensity:0.9});
[[-0.6,0.55,-2.21],[0.6,0.55,-2.21]].forEach(([x,y,z])=>{
  const tl = new THREE.Mesh(new THREE.BoxGeometry(0.45,0.18,0.05),tlMat);
  tl.position.set(x,y,z); car.add(tl);
});

car.position.set(0,0,0);

// ─── NPC CARS ─────────────────────────────────────────────────
const npcColors = [0x2255ff,0x22aa44,0xffaa22,0xaa22ff,0x22aaff,0xaaaaaa];
const npcs = [];
function makeNPC(x,z,dx,dz,speed,col) {
  const g = new THREE.Group();
  const b = new THREE.Mesh(new THREE.BoxGeometry(1.8,0.65,4),
    new THREE.MeshLambertMaterial({color:col}));
  b.position.y=0.5; b.castShadow=true; g.add(b);
  const c = new THREE.Mesh(new THREE.BoxGeometry(1.4,0.55,1.8),
    new THREE.MeshLambertMaterial({color:col}));
  c.position.set(0,0.95,-0.1); g.add(c);
  // wheels
  [[0.9,0.35,1.2],[-0.9,0.35,1.2],[0.9,0.35,-1.2],[-0.9,0.35,-1.2]].forEach(([wx,wy,wz])=>{
    const wh = new THREE.Mesh(new THREE.CylinderGeometry(0.32,0.32,0.25,10),
      new THREE.MeshLambertMaterial({color:0x111111}));
    wh.rotation.z=Math.PI/2; wh.position.set(wx,wy,wz); g.add(wh);
  });
  g.position.set(x,0,z);
  // Orient toward travel direction
  g.rotation.y = Math.atan2(dx,dz);
  scene.add(g);
  npcs.push({mesh:g, x, z, dx, dz, speed, baseX:x, baseZ:z, range:80});
}

// Horizontal NPC cars
for(let zi of [-30,0,30,60,-60]) {
  makeNPC(-80, zi+2.5, 1,0, 0.08+Math.random()*0.04, npcColors[Math.floor(Math.random()*6)]);
  makeNPC(60,  zi-2.5,-1,0, 0.07+Math.random()*0.04, npcColors[Math.floor(Math.random()*6)]);
}
// Vertical NPC cars
for(let xi of [-40,0,40,80,-80]) {
  makeNPC(xi+2.5,-80, 0,1, 0.08+Math.random()*0.04, npcColors[Math.floor(Math.random()*6)]);
  makeNPC(xi-2.5, 60, 0,-1,0.07+Math.random()*0.04, npcColors[Math.floor(Math.random()*6)]);
}

// ─── SKY & CLOUDS ─────────────────────────────────────────────
function makeCloud(x,y,z) {
  const g = new THREE.Group();
  [[0,0,0],[1.5,0.3,0.5],[-1.2,0.2,0.3],[0.5,0.4,-0.8]].forEach(([cx,cy,cz])=>{
    const s = new THREE.Mesh(
      new THREE.SphereGeometry(1.8+Math.random()*0.8,7,7),
      new THREE.MeshLambertMaterial({color:0xffffff,transparent:true,opacity:0.85})
    );
    s.position.set(cx*2,cy,cz*2); g.add(s);
  });
  g.position.set(x,y,z); scene.add(g);
}
for(let i=0;i<15;i++) makeCloud((Math.random()-0.5)*300, 60+Math.random()*30, (Math.random()-0.5)*300);

// ─── PHYSICS STATE ────────────────────────────────────────────
let speed=0, angle=0, steer=0;
const MAX_SPEED=0.45, ACCEL=0.012, DECEL=0.009, BRAKE=0.022, STEER_SPEED=0.035, STEER_RETURN=0.05;
let cameraAngle=0, cameraHeight=4, cameraDist=11;
let rpm=0;

// ─── LOCATIONS ────────────────────────────────────────────────
const locations = [
  {x:0,z:0,name:"Corso Vittorio Emanuele II"},
  {x:0,z:-80,name:"Piazza Castello"},
  {x:90,z:-85,name:"Mole Antonelliana"},
  {x:0,z:60,name:"Quartiere Crocetta"},
  {x:80,z:0,name:"Corso Massimo d'Azeglio"},
  {x:-80,z:0,name:"Corso Galileo Galilei"},
  {x:145,z:0,name:"Lungo Po"},
  {x:0,z:-60,name:"Via Nizza"},
  {x:40,z:0,name:"Corso Matteotti"},
];
let lastLocation="";

function updateLocation() {
  const cx=car.position.x, cz=car.position.z;
  let best=locations[0], bestD=Infinity;
  locations.forEach(l=>{
    const d=Math.sqrt((cx-l.x)**2+(cz-l.z)**2);
    if(d<bestD){bestD=d;best=l;}
  });
  if(best.name!==lastLocation) {
    lastLocation=best.name;
    document.getElementById('locationBadge').textContent='📍 '+best.name;
  }
}

// ─── MINIMAP ─────────────────────────────────────────────────
const mmCanvas=document.getElementById('minimapCanvas');
const mmCtx=mmCanvas.getContext('2d');
const MM=110, MM_SCALE=110/220; // 220 world units -> 110px

function drawMinimap() {
  mmCtx.fillStyle='#111';
  mmCtx.fillRect(0,0,MM,MM);
  // Roads
  mmCtx.strokeStyle='#333'; mmCtx.lineWidth=4;
  roads.forEach(([x1,z1,x2,z2])=>{
    mmCtx.beginPath();
    mmCtx.moveTo(x1*MM_SCALE+MM/2, z1*MM_SCALE+MM/2);
    mmCtx.lineTo(x2*MM_SCALE+MM/2, z2*MM_SCALE+MM/2);
    mmCtx.stroke();
  });
  // NPC dots
  mmCtx.fillStyle='#4488ff';
  npcs.forEach(n=>{
    const nx=n.mesh.position.x*MM_SCALE+MM/2;
    const nz=n.mesh.position.z*MM_SCALE+MM/2;
    mmCtx.fillRect(nx-2,nz-2,4,4);
  });
  // Player dot
  const px=car.position.x*MM_SCALE+MM/2;
  const pz=car.position.z*MM_SCALE+MM/2;
  mmCtx.fillStyle='#ff3a3a';
  mmCtx.beginPath();
  mmCtx.arc(px,pz,4,0,Math.PI*2); mmCtx.fill();
  // Arrow
  mmCtx.save();
  mmCtx.translate(px,pz);
  mmCtx.rotate(-car.rotation.y);
  mmCtx.fillStyle='#fff';
  mmCtx.beginPath(); mmCtx.moveTo(0,-7); mmCtx.lineTo(4,4); mmCtx.lineTo(-4,4); mmCtx.closePath();
  mmCtx.fill();
  mmCtx.restore();
}

// ─── INPUT (keyboard) ─────────────────────────────────────────
window.addEventListener('keydown', e=>{
  if(e.key==='ArrowUp'||e.key==='w'||e.key==='W') keys.up=true;
  if(e.key==='ArrowDown'||e.key==='s'||e.key==='S') keys.down=true;
  if(e.key==='ArrowLeft'||e.key==='a'||e.key==='A') keys.left=true;
  if(e.key==='ArrowRight'||e.key==='d'||e.key==='D') keys.right=true;
});
window.addEventListener('keyup', e=>{
  if(e.key==='ArrowUp'||e.key==='w'||e.key==='W') keys.up=false;
  if(e.key==='ArrowDown'||e.key==='s'||e.key==='S') keys.down=false;
  if(e.key==='ArrowLeft'||e.key==='a'||e.key==='A') keys.left=false;
  if(e.key==='ArrowRight'||e.key==='d'||e.key==='D') keys.right=false;
});

// ─── TOUCH CONTROLS ──────────────────────────────────────────
function setupTouchBtn(el, keyName) {
  el.addEventListener('touchstart', e=>{ e.preventDefault(); keys[keyName]=true; el.classList.add('active-btn'); }, {passive:false});
  el.addEventListener('touchend',   e=>{ e.preventDefault(); keys[keyName]=false; el.classList.remove('active-btn'); }, {passive:false});
  el.addEventListener('mousedown',  ()=>{ keys[keyName]=true; el.classList.add('active-btn'); });
  el.addEventListener('mouseup',    ()=>{ keys[keyName]=false; el.classList.remove('active-btn'); });
}
setupTouchBtn(document.getElementById('btnGas'),   'up');
setupTouchBtn(document.getElementById('btnBrake'), 'down');
setupTouchBtn(document.getElementById('btnLeft'),  'left');
setupTouchBtn(document.getElementById('btnRight'), 'right');

// Steer track swipe
const steerTrack=document.getElementById('steerTrack');
const steerKnob=document.getElementById('steerKnob');
let steerTouchActive=false, steerStartX=0;
steerTrack.addEventListener('touchstart', e=>{ e.preventDefault(); steerTouchActive=true; steerStartX=e.touches[0].clientX; },{passive:false});
steerTrack.addEventListener('touchmove', e=>{ e.preventDefault();
  if(!steerTouchActive)return;
  const dx=(e.touches[0].clientX-steerStartX)/50;
  keys.left=dx<-0.3; keys.right=dx>0.3;
  const knobX=Math.max(4,Math.min(104,54+dx*54));
  steerKnob.style.left=knobX+'px';
},{passive:false});
steerTrack.addEventListener('touchend', e=>{ e.preventDefault();
  steerTouchActive=false; keys.left=false; keys.right=false;
  steerKnob.style.left='54px';
},{passive:false});

// ─── GAME LOOP ────────────────────────────────────────────────
function getGear(spd) {
  if(Math.abs(spd)<0.02)return 'N';
  if(spd<0)return 'R';
  const s=Math.abs(spd)/MAX_SPEED;
  if(s<0.15)return '1';
  if(s<0.3)return '2';
  if(s<0.5)return '3';
  if(s<0.7)return '4';
  if(s<0.85)return '5';
  return '6';
}

let frameCount=0;
function animate() {
  if(!gameStarted){ requestAnimationFrame(animate); return; }
  requestAnimationFrame(animate);
  frameCount++;

  // Physics
  if(keys.up)   speed=Math.min(speed+ACCEL, MAX_SPEED);
  else if(keys.down) speed=Math.max(speed-BRAKE, -MAX_SPEED*0.4);
  else speed=speed>0?Math.max(0,speed-DECEL):Math.min(0,speed+DECEL);

  if(Math.abs(speed)>0.005) {
    if(keys.left)  steer=Math.min(steer+STEER_SPEED,1);
    if(keys.right) steer=Math.max(steer-STEER_SPEED,-1);
  }
  if(!keys.left&&!keys.right) steer*=(1-STEER_RETURN);

  angle += steer * speed * 2.2;
  car.rotation.y = angle;
  car.position.x += Math.sin(angle)*speed;
  car.position.z += Math.cos(angle)*speed;

  // Bounds
  car.position.x=Math.max(-220,Math.min(220,car.position.x));
  car.position.z=Math.max(-220,Math.min(220,car.position.z));

  // Wheel rotation
  const wheelRot=(speed/0.38)*1.5;
  wheels.forEach((w,i)=>{ w.children[0].rotation.x+=wheelRot; w.children[1].rotation.x+=wheelRot; });
  // Front wheels steer
  wheels[0].rotation.y=steer*0.4;
  wheels[1].rotation.y=steer*0.4;

  // Camera
  const camTargetX=car.position.x-Math.sin(angle)*cameraDist;
  const camTargetZ=car.position.z-Math.cos(angle)*cameraDist;
  camera.position.x+=(camTargetX-camera.position.x)*0.1;
  camera.position.y+=(cameraHeight-camera.position.y)*0.1;
  camera.position.z+=(camTargetZ-camera.position.z)*0.1;
  camera.lookAt(car.position.x, 1.2, car.position.z);

  // NPC movement
  npcs.forEach(n=>{
    n.x+=n.dx*n.speed; n.z+=n.dz*n.speed;
    if(Math.abs(n.x-n.baseX)>n.range||Math.abs(n.z-n.baseZ)>n.range){
      n.dx*=-1; n.dz*=-1;
      n.mesh.rotation.y=Math.atan2(n.dx,n.dz);
    }
    n.mesh.position.x=n.x; n.mesh.position.z=n.z;
  });

  // RPM
  rpm+=(Math.abs(speed)/MAX_SPEED-rpm)*0.08;
  if(keys.up&&speed<MAX_SPEED) rpm=Math.min(1,rpm+0.02);

  // HUD update (every 3 frames)
  if(frameCount%3===0) {
    const kmh=Math.abs(Math.round(speed/MAX_SPEED*160));
    document.getElementById('speedNum').textContent=kmh;
    document.getElementById('gearNum').textContent=getGear(speed);
    document.getElementById('rpmFill').style.width=(rpm*100)+'%';
    updateLocation();
    drawMinimap();
  }

  renderer.render(scene, camera);
}

// ─── START ────────────────────────────────────────────────────
document.getElementById('startBtn').addEventListener('click', ()=>{
  gameStarted=true;
  document.getElementById('splash').classList.add('hidden');
  document.getElementById('hud').classList.add('active');
  document.getElementById('controls').classList.add('active');
  setTimeout(()=>document.getElementById('splash').remove(),900);
});

window.addEventListener('resize',()=>{
  camera.aspect=innerWidth/innerHeight;
  camera.updateProjectionMatrix();
  renderer.setSize(innerWidth,innerHeight);
});

animate();
</script>
</body>
</html>