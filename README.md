<!DOCTYPE html>
<html>
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1">
<title>Stellar Drift 3D</title>
<style>
*{box-sizing:border-box}
body{margin:0;overflow:hidden;background:#000;font-family:'Segoe UI',Arial,sans-serif;color:#fff}
#renderer-container{position:fixed;top:0;left:0;width:100%;height:100%;z-index:0}
#renderer-container canvas{display:block;width:100%!important;height:100%!important}

/* HOMESCREEN */
#homescreen{
  position:fixed;top:50%;left:50%;transform:translate(-50%,-50%);
  text-align:center;z-index:100;
  background:rgba(2,4,20,.93);border:1px solid #00ffcc33;
  border-radius:14px;padding:16px 18px 18px;
  width:min(500px,96vw);max-height:92vh;overflow-y:auto;
  scrollbar-width:thin;scrollbar-color:#00ffcc33 transparent;
}
#homescreen::-webkit-scrollbar{width:5px}
#homescreen::-webkit-scrollbar-thumb{background:#00ffcc33;border-radius:3px}

/* IN-GAME UI */
#ui{
  position:fixed;top:8px;left:50%;transform:translateX(-50%);
  text-align:center;font-size:16px;z-index:10;display:none;
  min-width:320px;pointer-events:none;
}

/* RANK BAR */
.rankBar{width:280px;height:16px;background:#333;border:2px solid #555;margin:3px auto;position:relative;overflow:hidden;border-radius:3px}
.rankFill{height:100%;background:linear-gradient(90deg,#00ff00,#ffff00);width:0%;transition:width .3s}
.rankText{position:absolute;top:50%;left:50%;transform:translate(-50%,-50%);font-size:10px;color:#fff;font-weight:bold}
.rankLabel{font-size:22px;font-weight:bold;margin-bottom:4px;text-shadow:0 0 10px currentColor}
.xpLabel{font-size:13px;font-weight:bold;color:#00ffcc;margin:2px auto;text-shadow:0 0 8px #00ffcc}
.gemsLabel{font-size:15px;color:#00ffd0;font-weight:bold;margin:3px auto;text-shadow:0 0 8px #0ff}

/* TABS */
#homeTabs{display:flex;gap:4px;justify-content:center;margin:8px 0 0;flex-wrap:wrap}
.tabBtn{padding:5px 11px;background:#1a1a2e;color:#777;border:1px solid #333;border-radius:6px;cursor:pointer;font-size:12px;font-weight:bold;transition:all .15s}
.tabBtn.active{background:#0d2137;color:#00ffcc;border-color:#00ffcc;box-shadow:0 0 8px #00ffcc44}
.tabPanel{display:none}.tabPanel.active{display:block}

/* SHIP SELECT */
.cosmeticShip{display:inline-block;margin:6px;cursor:pointer;border:2px solid rgba(255,255,255,.07);border-radius:8px;transition:box-shadow .2s,border .2s;background:#111;vertical-align:top;position:relative}
.cosmeticShip.selected{box-shadow:0 0 16px #fff;border:2px solid #fcd34d}
.cosmeticShip.locked{opacity:.33;cursor:default}
.booster-badge{position:absolute;top:3px;right:5px;background:#0ff;color:#222;font-size:9px;font-weight:bold;border-radius:4px;padding:1px 4px}

/* RUN BOOSTS */
#boostsGrid{display:flex;flex-wrap:wrap;justify-content:center;gap:7px;margin-top:6px;max-width:460px}
.boostCard{background:#0d1a2e;border:1px solid #334;border-radius:9px;padding:9px 10px;width:120px;text-align:center}
.boostCard.equipped{border-color:#00ffcc;box-shadow:0 0 8px #00ffcc44}
.boostCard .bn{font-size:13px;font-weight:bold;color:#fff;margin-bottom:3px}
.boostCard .bd{font-size:10px;color:#aaa;margin-bottom:6px;min-height:28px}
.boostCard .bc{font-size:11px;color:#00ffd0;margin-bottom:5px}
.boostCard button{padding:3px 10px;font-size:11px;border-radius:5px;cursor:pointer;font-weight:bold;border:1px solid #00ffcc;background:#222;color:#00ffcc}
.boostCard button:disabled{border-color:#444;color:#555;cursor:not-allowed}
.boostCard button.unequip{border-color:#ff6666;color:#ff6666}
#activeBoostsList{font-size:12px;color:#0ff;margin:4px 0 2px;min-height:16px}

/* QUESTS */
#questsPanel{max-width:420px;margin:0 auto}
#questsHeader{font-size:12px;color:#888;margin:4px 0 6px}
.questCard{background:#0d1520;border:1px solid #223;border-radius:10px;padding:9px 12px;margin:6px 0;text-align:left;position:relative}
.questCard.done{border-color:#00ff88;background:#091409}
.questCard .qt{font-size:13px;font-weight:bold;color:#fff;margin-bottom:3px}
.questCard .qd{font-size:11px;color:#aaa;margin-bottom:5px}
.questCard .qr{font-size:11px;color:#ffd700;position:absolute;top:9px;right:10px}
.questCard .qbar{height:8px;background:#1a1a2a;border-radius:4px;overflow:hidden;margin-bottom:3px}
.questCard .qfill{height:100%;border-radius:4px;transition:width .3s}
.questCard .qprog{font-size:10px;color:#888}
.questCard .qdone{font-size:12px;color:#00ff88;font-weight:bold}
.questRarity-common .qfill{background:#00cc88}.questRarity-rare .qfill{background:#4488ff}.questRarity-epic .qfill{background:#cc44ff}
.questRarity-common{border-left:3px solid #00cc88}.questRarity-rare{border-left:3px solid #4488ff}.questRarity-epic{border-left:3px solid #cc44ff}

/* NAME / MISC */
#nameRow{display:flex;align-items:center;gap:6px;justify-content:center;margin:5px 0}
#playerNameInput{background:#111;border:1px solid #00ffcc;color:#fff;border-radius:5px;padding:3px 9px;font-size:13px;width:130px;text-align:center}
#boosterBtn{padding:5px 14px;font-size:13px;background:#222;color:#00ffd0;border:1px solid #00ffd0;border-radius:7px;cursor:pointer;font-weight:bold}
#boosterBtn:disabled{color:#444!important;border-color:#444!important;opacity:.5;cursor:not-allowed}
#boosterResult{display:none;margin:3px auto;font-size:13px;font-weight:bold;color:#fff770;text-shadow:0 0 8px #0ff}
#cosmeticPowerDesc{margin:6px 0 4px;font-size:13px;color:#0ff}
#startBtn{padding:11px 34px;font-size:20px;border:2px solid #00ffcc;background:#222;color:#00ffcc;border-radius:8px;margin-top:12px;cursor:pointer;transition:background .2s;pointer-events:all}
#startBtn:hover{background:#00ffcc;color:black}

/* IN-GAME */
#rankUpAnimation{display:none;position:fixed;top:50%;left:50%;transform:translate(-50%,-50%);font-size:48px;font-weight:bold;color:gold;text-shadow:0 0 20px gold,0 0 40px gold;animation:rankUpPulse 1s ease-out;z-index:1000;pointer-events:none}
@keyframes rankUpPulse{0%{transform:translate(-50%,-50%) scale(1);opacity:1}100%{transform:translate(-50%,-150%) scale(1.5);opacity:0}}
#reviveFlash{display:none;position:fixed;left:0;top:0;width:100vw;height:100vh;background:rgba(0,255,255,.25);z-index:99999;pointer-events:none}
#gameOver{display:none;margin-top:5px;font-size:18px;color:#ff4444}
#godlyCooldown{position:fixed;left:50%;top:66px;transform:translateX(-50%);font-size:14px;font-weight:bold;color:#00fff7;background:rgba(0,0,0,.65);border-radius:6px;padding:2px 10px;display:none;z-index:999;box-shadow:0 0 8px #00fff7}
#soulBurstDisplay{position:fixed;left:50%;top:88px;transform:translateX(-50%);font-size:13px;font-weight:bold;color:#fffacd;background:rgba(0,0,0,.65);border-radius:6px;padding:2px 10px;display:none;z-index:999;box-shadow:0 0 10px #fffacd88}
#laserDisplay{position:fixed;right:8px;top:8px;font-size:14px;font-weight:bold;color:#ff6644;text-shadow:0 0 8px #ff4400;display:none;z-index:999;pointer-events:none}
#teleportFlash{display:none;position:fixed;left:0;top:0;width:100vw;height:100vh;pointer-events:none;z-index:99998}
#toast{display:none;position:fixed;left:50%;bottom:40px;transform:translateX(-50%);background:rgba(0,0,0,.88);border:1px solid #00ffcc;border-radius:8px;padding:8px 18px;font-size:14px;font-weight:bold;color:#fff770;text-shadow:0 0 8px #0ff;z-index:9999;white-space:nowrap}
#controls-hint{position:fixed;bottom:10px;left:50%;transform:translateX(-50%);font-size:11px;color:#334;z-index:999;pointer-events:none}

/* Soul Burst flash */
@keyframes soulBurstFlash{
  0%{opacity:0.9;transform:scale(1)}
  50%{opacity:0.6}
  100%{opacity:0;transform:scale(1.3)}
}
.soul-burst-anim{animation:soulBurstFlash .5s ease-out forwards}

/* Teleport flash keyframe */
@keyframes teleportBurst{
  0%{opacity:.85;transform:scale(1)}
  60%{opacity:.4;transform:scale(1.08)}
  100%{opacity:0;transform:scale(1.18)}
}
.teleport-anim{animation:teleportBurst .28s ease-out forwards}
</style>
</head>
<body>

<div id="renderer-container"></div>

<!-- FLASH / TOAST / OVERLAYS -->
<div id="reviveFlash"></div>
<div id="teleportFlash"></div>
<div id="rankUpAnimation">⭐ RANK UP ⭐</div>
<div id="toast"></div>
<div id="godlyCooldown"></div>
<div id="soulBurstDisplay">✨ Soul Burst: <span id="soulBurstCount">20</span> dodges</div>
<div id="laserDisplay">🔥 Laser: <span id="laserLeft">3</span></div>
<div id="controls-hint">← → / A D = Move &nbsp;|&nbsp; SPACE = Boost &nbsp;|&nbsp; L = Laser &nbsp;|&nbsp; F = Annihilate &nbsp;|&nbsp; R = Restart</div>

<!-- HOMESCREEN -->
<div id="homescreen">
  <div class="rankLabel" id="homeRank" style="color:#A0A0A0">CADET</div>
  <div class="rankBar"><div class="rankFill" id="homeRankFill"></div><div class="rankText" id="homeRankText">0 / 100 XP</div></div>
  <div class="xpLabel" id="homeXpLabel">XP: 0</div>
  <div class="gemsLabel">💎 <span class="gemCount">0</span> Gems</div>
  <div id="nameRow">
    <span style="font-size:12px;color:#888">Pilot Name:</span>
    <input id="playerNameInput" type="text" maxlength="16" placeholder="Enter name…"/>
  </div>
  <div id="homeTabs">
    <button class="tabBtn active" data-tab="shipTab">🚀 Ship</button>
    <button class="tabBtn" data-tab="boostsTab">⚡ Run Boosts</button>
    <button class="tabBtn" data-tab="questsTab">📜 Quests</button>
  </div>
  <div id="shipTab" class="tabPanel active">
    <div style="margin:5px 0 3px">
      <button id="boosterBtn">🎲 Random Ship Booster (1 💎)</button>
      <div id="boosterResult"></div>
    </div>
    <div id="cosmeticList"></div>
    <div id="cosmeticPowerDesc"></div>
  </div>
  <div id="boostsTab" class="tabPanel">
    <div style="font-size:12px;color:#888;margin:5px 0 2px">Buy boosts before a run — spent on use.</div>
    <div id="activeBoostsList"></div>
    <div id="boostsGrid"></div>
  </div>
  <div id="questsTab" class="tabPanel">
    <div id="questsPanel">
      <div id="questsHeader">Complete quests to earn 💎 Gems. New quests unlock as you finish them!</div>
      <div id="questsList"></div>
    </div>
  </div>
  <button id="startBtn">▶ Start Game</button>
</div>

<!-- IN-GAME UI -->
<div id="ui">
  <div class="rankLabel" id="rankDisplay" style="color:#A0A0A0">CADET</div>
  <div class="rankBar"><div class="rankFill" id="gameRankFill"></div><div class="rankText" id="gameRankText">0 / 100 XP</div></div>
  <div class="xpLabel" id="xpDisplay">XP: 0</div>
  <div class="gemsLabel">💎 <span class="gemCount">0</span></div>
  <div>Score: <span id="score">0</span> &nbsp;|&nbsp; Best: <span id="highScore">0</span></div>
  <div id="gameOver">💥 GAME OVER — Press R to return Home</div>
</div>

<script src="https://cdnjs.cloudflare.com/ajax/libs/three.js/r128/three.min.js"></script>
<script>
// ══════════════════════════════════════════════════════════
// DATA
// ══════════════════════════════════════════════════════════
const XP_RANKS=[
  {name:"Cadet",xp:0,color:"#A0A0A0"},{name:"Pilot",xp:100,color:"#00FF00"},
  {name:"Commander",xp:300,color:"#0099FF"},{name:"Captain",xp:600,color:"#FF6600"},
  {name:"Admiral",xp:1200,color:"#FF00FF"},{name:"Legend",xp:2500,color:"#FFFF00"},
  {name:"Godly",xp:4000,color:"#00fff7"},{name:"Mythic",xp:6000,color:"#ff77ff"},
  {name:"Eternal",xp:9000,color:"#fffacd"},{name:"Cosmic",xp:13000,color:"#ff6666"},
  {name:"Transcendent",xp:20000,color:"#ffffff"},
];
const COSMETICS=[
  {id:0,name:"Cadet Viper",color:"#00ffcc",unlockXp:0,powerName:"Standard",powerDesc:"No special power."},
  {id:1,name:"Pilot Falcon",color:"#00FF00",unlockXp:100,powerName:"Double Boost",powerDesc:"Spacebar boost is twice as fast."},
  {id:2,name:"Commander Nova",color:"#0099FF",unlockXp:300,powerName:"Shield on Dodge",powerDesc:"Gain a shield every 5 asteroids dodged."},
  {id:3,name:"Captain Phoenix",color:"#FF6600",unlockXp:600,powerName:"Milestone XP+",powerDesc:"+50% XP for score milestones."},
  {id:4,name:"Admiral Eclipse",color:"#FF00FF",unlockXp:1200,powerName:"Slow Asteroids",powerDesc:"Asteroid speed −20%."},
  {id:5,name:"Legend Starcruiser",color:"#FFFF00",unlockXp:2500,powerName:"Revive",powerDesc:"Survive first crash once per game."},
  {id:6,name:"Godly Dragon",color:"#00fff7",unlockXp:4000,powerName:"Time Annihilate",powerDesc:"Press F every 30s: destroy all asteroids + freeze 3s."},
  {id:7,name:"Mythic Phantom",color:"#ff77ff",unlockXp:6000,powerName:"Void Teleport",powerDesc:"Instantly teleport between lanes — no sliding. Teleport through asteroids harmlessly!"},
  {id:8,name:"Eternal Seraph",color:"#fffacd",unlockXp:9000,powerName:"Soul Burst",powerDesc:"Every 20 asteroids dodged, your soul erupts — all asteroids annihilated + burst of XP."},
];
const RUN_BOOSTS=[
  {id:"hyperdrive",name:"⚡ Hyperdrive",desc:"Move 2× faster between lanes",cost:1},
  {id:"iron_shell",name:"🛡️ Iron Shell",desc:"Start the run with a shield",cost:1},
  {id:"xp_surge",name:"✨ XP Surge",desc:"Earn 3× XP this entire run",cost:2},
  {id:"null_field",name:"❄️ Null Field",desc:"Asteroids 35% slower this run",cost:2},
  {id:"laser_shot",name:"🔥 Laser Shot",desc:"Press L to blast your lane (3 shots)",cost:2},
];
const QUEST_TIERS=[
  {targets:[100,150,200],gems:1},{targets:[200,250,300],gems:1},
  {targets:[300,400,500],gems:2},{targets:[500,600,750],gems:2},
  {targets:[750,900,1100],gems:3},{targets:[1000,1250,1500],gems:3},
  {targets:[1500,1750,2000],gems:4},{targets:[2000,2500,3000],gems:4},
  {targets:[3000,4000,5000],gems:5},
];

// ══════════════════════════════════════════════════════════
// THREE.JS SETUP
// ══════════════════════════════════════════════════════════
const scene = new THREE.Scene();
scene.background = new THREE.Color(0x000510);
scene.fog = new THREE.FogExp2(0x000510, 0.008);

const renderer = new THREE.WebGLRenderer({antialias:true,powerPreference:'high-performance'});
renderer.setPixelRatio(Math.min(window.devicePixelRatio,2));
renderer.setSize(window.innerWidth,window.innerHeight);
renderer.shadowMap.enabled = true;
document.getElementById("renderer-container").appendChild(renderer.domElement);

const camera = new THREE.PerspectiveCamera(72, window.innerWidth/window.innerHeight, 0.1, 600);
camera.position.set(0, 6, 12);
camera.lookAt(0, 0, -20);

window.addEventListener("resize",()=>{
  renderer.setSize(window.innerWidth,window.innerHeight);
  camera.aspect = window.innerWidth/window.innerHeight;
  camera.updateProjectionMatrix();
});

// Lights
const ambientLight = new THREE.AmbientLight(0x111233, 2);
scene.add(ambientLight);
const sunLight = new THREE.DirectionalLight(0x6688cc, 1.5);
sunLight.position.set(-5, 10, 5);
scene.add(sunLight);
const rimLight = new THREE.DirectionalLight(0x00ffcc, 0.5);
rimLight.position.set(5, -5, -10);
scene.add(rimLight);
const shipPointLight = new THREE.PointLight(0xffffff, 2, 12);
shipPointLight.position.set(0, 3, 2);
scene.add(shipPointLight);
const engineLight = new THREE.PointLight(0x00ffcc, 3, 6);
scene.add(engineLight);

// Void teleport flash point light (purple, only active during teleport)
const voidLight = new THREE.PointLight(0xff44ff, 0, 14);
scene.add(voidLight);

// Soul Burst halo light (warm gold, only active during soul burst)
const soulLight = new THREE.PointLight(0xfffacd, 0, 20);
scene.add(soulLight);

// ══════════════════════════════════════════════════════════
// GAME WORLD CONSTANTS
// ══════════════════════════════════════════════════════════
const LANES_3D = [-5, 0, 5];
const SPAWN_Z = -130;
const PASS_Z = 5;
const SPEED_SCALE = 0.175;
const XP_PER_ASTEROID=10, XP_MILESTONE_STEP=100, XP_PER_MILESTONE=40, XP_PASSIVE_SEC=1;

// ══════════════════════════════════════════════════════════
// SCENE OBJECTS (persistent)
// ══════════════════════════════════════════════════════════
let shipGroup = null;
let shieldMesh = null;
let laserBeam = null;
let engineExhaust = [];
let particles3D = [];
let asteroids = [];
let stars3D = null;

// Seraph halo ring reference (for spinning)
let seraphHaloRing = null;
let seraphWingGlow = 0; // 0–1, pulses during soul burst
let seraphSoulBurstFlashFrames = 0;

// Phantom ghost decoy (boosted Mythic Phantom only)
let ghostDecoy = null;
let ghostDecoyLane = -1;
let ghostDecoyFrames = 0;

// Teleport state
let isTeleporting = false;
let teleportFrames = 0;
const TELEPORT_DURATION = 8; // frames ship is invisible mid-teleport

// Void burst afterimage particles
let voidParticles = [];

// Soul burst particles (Eternal Seraph)
let soulParticles = [];

// ══════════════════════════════════════════════════════════
// STARFIELD
// ══════════════════════════════════════════════════════════
function createStarfield(){
  const geo = new THREE.BufferGeometry();
  const positions = [], colors = [];
  for(let i=0;i<3000;i++){
    positions.push((Math.random()-.5)*250,(Math.random()-.5)*140,Math.random()*-250);
    const bright=0.5+Math.random()*0.5;
    colors.push(bright,bright,Math.min(1,bright+Math.random()*0.3));
  }
  geo.setAttribute("position",new THREE.Float32BufferAttribute(positions,3));
  geo.setAttribute("color",new THREE.Float32BufferAttribute(colors,3));
  const mat=new THREE.PointsMaterial({size:0.35,sizeAttenuation:true,vertexColors:true,transparent:true,opacity:0.9});
  return new THREE.Points(geo,mat);
}
stars3D = createStarfield();
scene.add(stars3D);

function updateStarfield(speed2d){
  const s=speed2d*SPEED_SCALE*1.8;
  const pos=stars3D.geometry.attributes.position.array;
  for(let i=0;i<pos.length;i+=3){
    pos[i+2]+=s;
    if(pos[i+2]>80){pos[i]=(Math.random()-.5)*250;pos[i+1]=(Math.random()-.5)*140;pos[i+2]=-250+Math.random()*20;}
  }
  stars3D.geometry.attributes.position.needsUpdate=true;
}

// ══════════════════════════════════════════════════════════
// LANE MARKERS
// ══════════════════════════════════════════════════════════
function createLaneScene(){
  const group=new THREE.Group();
  const floorGeo=new THREE.PlaneGeometry(22,300);
  const floorMat=new THREE.MeshBasicMaterial({color:0x010815,side:THREE.DoubleSide,transparent:true,opacity:0.8});
  const floor=new THREE.Mesh(floorGeo,floorMat);
  floor.rotation.x=Math.PI/2; floor.position.set(0,-1.8,-100); group.add(floor);
  for(let z=-130;z<=10;z+=12){
    const pts=[new THREE.Vector3(-11,-1.8,z),new THREE.Vector3(11,-1.8,z)];
    const g=new THREE.BufferGeometry().setFromPoints(pts);
    group.add(new THREE.Line(g,new THREE.LineBasicMaterial({color:0x001830,transparent:true,opacity:0.4})));
  }
  [-2.5,2.5].forEach(x=>{
    const pts=[new THREE.Vector3(x,-1.8,-130),new THREE.Vector3(x,-1.8,10)];
    const g=new THREE.BufferGeometry().setFromPoints(pts);
    group.add(new THREE.Line(g,new THREE.LineBasicMaterial({color:0x004488,transparent:true,opacity:0.7})));
    group.add(new THREE.Line(g,new THREE.LineBasicMaterial({color:0x0088ff,transparent:true,opacity:0.25})));
  });
  [-7.5,7.5].forEach(x=>{
    const pts=[new THREE.Vector3(x,-1.8,-130),new THREE.Vector3(x,-1.8,10)];
    const g=new THREE.BufferGeometry().setFromPoints(pts);
    group.add(new THREE.Line(g,new THREE.LineBasicMaterial({color:0x002244,transparent:true,opacity:0.5})));
  });
  LANES_3D.forEach(x=>{
    const pts=[new THREE.Vector3(x,-1.8,-130),new THREE.Vector3(x,-1.8,10)];
    const g=new THREE.BufferGeometry().setFromPoints(pts);
    group.add(new THREE.Line(g,new THREE.LineBasicMaterial({color:0x003355,transparent:true,opacity:0.3})));
  });
  const horizonGeo=new THREE.SphereGeometry(4,16,16);
  const horizonMat=new THREE.MeshBasicMaterial({color:0x0044aa,transparent:true,opacity:0.12,side:THREE.BackSide});
  const horizon=new THREE.Mesh(horizonGeo,horizonMat);
  horizon.position.set(0,0,-140); group.add(horizon);
  return group;
}
scene.add(createLaneScene());

// ══════════════════════════════════════════════════════════
// SHIP MESH FACTORY
// ══════════════════════════════════════════════════════════
function makeMat(hex,emissiveScale=0.35,metalness=0.6,roughness=0.25){
  const c=new THREE.Color(hex);
  return new THREE.MeshStandardMaterial({color:c,emissive:c.clone().multiplyScalar(emissiveScale),metalness,roughness});
}

function createShipMesh(id){
  const group=new THREE.Group();
  const col=COSMETICS[id].color;
  const mat=makeMat(col);
  seraphHaloRing = null; // reset

  if(id===0){
    const body=new THREE.Mesh(new THREE.ConeGeometry(0.75,3.5,5),mat);
    body.rotation.x=-Math.PI/2; body.position.z=-0.3; group.add(body);
    const finGeo=new THREE.BoxGeometry(0.12,1.1,1.4);
    [-1,1].forEach(s=>{const fin=new THREE.Mesh(finGeo,mat.clone());fin.position.set(s*1.1,-0.3,1.0);group.add(fin);});
    const cockpit=new THREE.Mesh(new THREE.SphereGeometry(0.35,6,6),makeMat(col,0.8,0.3,0.1));
    cockpit.position.set(0,0.3,-0.5); group.add(cockpit);
  } else if(id===1){
    const body=new THREE.Mesh(new THREE.ConeGeometry(0.65,4,4),mat);
    body.rotation.x=-Math.PI/2; body.position.z=-0.2; group.add(body);
    const wingShape=new THREE.Shape();
    wingShape.moveTo(0,0);wingShape.lineTo(3.5,0.5);wingShape.lineTo(2.8,2.5);wingShape.lineTo(0,1.5);
    const wingGeo=new THREE.ShapeGeometry(wingShape);
    const wingMat=new THREE.MeshStandardMaterial({color:new THREE.Color(col),emissive:new THREE.Color(col).multiplyScalar(0.3),side:THREE.DoubleSide,metalness:0.5,roughness:0.3});
    [-1,1].forEach(s=>{
      const wing=new THREE.Mesh(wingGeo,wingMat.clone());
      wing.rotation.x=Math.PI/2; wing.position.set(s*0.65,-0.1,0.5);
      if(s===-1){wing.rotation.y=Math.PI;wing.position.x=-0.65;} wing.scale.x=s; group.add(wing);
    });
  } else if(id===2){
    const body=new THREE.Mesh(new THREE.SphereGeometry(1,10,8),mat);
    body.scale.set(0.9,0.7,2.2); group.add(body);
    const nubGeo=new THREE.SphereGeometry(0.38,6,6);
    [-1.3,1.3].forEach(x=>{const nub=new THREE.Mesh(nubGeo,mat.clone());nub.position.set(x,0,0.6);group.add(nub);});
    const nose=new THREE.Mesh(new THREE.ConeGeometry(0.45,1.8,6),mat.clone());
    nose.rotation.x=-Math.PI/2; nose.position.set(0,0,-2.5); group.add(nose);
  } else if(id===3){
    const bodyGeo=new THREE.OctahedronGeometry(1.6,0);
    const body=new THREE.Mesh(bodyGeo,mat);
    body.scale.set(0.8,0.55,2.2); group.add(body);
    const tailGeo=new THREE.ConeGeometry(0.5,1.8,4);
    const tail=new THREE.Mesh(tailGeo,mat.clone());
    tail.rotation.x=Math.PI/2; tail.position.set(0,0,2.2); group.add(tail);
    const wingGeo=new THREE.BoxGeometry(3.2,0.12,1.2);
    const wing=new THREE.Mesh(wingGeo,mat.clone());
    wing.position.set(0,0,0.4); group.add(wing);
  } else if(id===4){
    const disc=new THREE.Mesh(new THREE.CylinderGeometry(1.8,1.4,0.6,12),mat);
    disc.rotation.x=Math.PI/2; group.add(disc);
    const dome=new THREE.Mesh(new THREE.SphereGeometry(0.9,8,6,0,Math.PI*2,0,Math.PI/2),mat.clone());
    dome.position.set(0,0,0); group.add(dome);
    const ring=new THREE.Mesh(new THREE.TorusGeometry(2.4,0.18,7,28),mat.clone());
    ring.rotation.x=Math.PI/2; group.add(ring);
    const nose2=new THREE.Mesh(new THREE.ConeGeometry(0.3,1.2,5),mat.clone());
    nose2.rotation.x=-Math.PI/2; nose2.position.set(0,0,-1.9); group.add(nose2);
  } else if(id===5){
    const body=new THREE.Mesh(new THREE.BoxGeometry(2.0,0.85,3.5),mat);
    group.add(body);
    const nose3=new THREE.Mesh(new THREE.ConeGeometry(1.05,2,4),mat.clone());
    nose3.rotation.x=-Math.PI/2; nose3.position.set(0,0,-2.6); group.add(nose3);
    const blade=new THREE.Mesh(new THREE.BoxGeometry(4.5,0.12,1.8),mat.clone());
    blade.position.set(0,0,0.3); group.add(blade);
    const gun=new THREE.Mesh(new THREE.CylinderGeometry(0.15,0.15,1.5,6),mat.clone());
    gun.rotation.x=Math.PI/2; gun.position.set(0,0.5,-1.8); group.add(gun);
  } else if(id===6){
    const body2=new THREE.Mesh(new THREE.CylinderGeometry(0.65,1.2,4,7),mat);
    body2.rotation.x=Math.PI/2; group.add(body2);
    const neck=new THREE.Mesh(new THREE.ConeGeometry(0.5,2.5,6),mat.clone());
    neck.rotation.x=-Math.PI/2; neck.position.set(0,0.3,-2.8); group.add(neck);
    const wShape=new THREE.Shape();
    wShape.moveTo(0,0);wShape.lineTo(3.2,-0.8);wShape.lineTo(2.2,2.5);wShape.lineTo(0.8,2.8);wShape.lineTo(0,1.5);
    const wGeo=new THREE.ShapeGeometry(wShape);
    const wMat=new THREE.MeshStandardMaterial({color:new THREE.Color(col),emissive:new THREE.Color(col).multiplyScalar(0.4),side:THREE.DoubleSide,metalness:0.3,roughness:0.4,transparent:true,opacity:0.92});
    [-1,1].forEach(s=>{const w=new THREE.Mesh(wGeo,wMat.clone());w.rotation.x=Math.PI/2;w.scale.x=s;w.position.set(s*0.65,-0.2,0.5);group.add(w);});
    const wMark=new THREE.MeshBasicMaterial({color:0xffffff,transparent:true,opacity:0.35,side:THREE.DoubleSide});
    const wm=new THREE.Shape();wm.moveTo(0.3,0.3);wm.lineTo(2,0.2);wm.lineTo(1.5,1.8);wm.lineTo(0.3,1.5);
    const wmGeo=new THREE.ShapeGeometry(wm);
    [-1,1].forEach(s=>{const wmark=new THREE.Mesh(wmGeo,wMark.clone());wmark.rotation.x=Math.PI/2;wmark.scale.x=s;wmark.position.set(s*0.66,-0.18,0.5);group.add(wmark);});
  } else if(id===7){
    // ── Mythic Phantom ──────────────────────────────────────
    const hullGeo = new THREE.ConeGeometry(1.1, 4.5, 3);
    const hull = new THREE.Mesh(hullGeo, mat);
    hull.rotation.x = -Math.PI/2;
    hull.position.z = -0.2;
    group.add(hull);
    const vwShape = new THREE.Shape();
    vwShape.moveTo(0, 0);vwShape.lineTo(3.8, 0.2);vwShape.lineTo(3.0, 1.6);vwShape.lineTo(1.2, 2.2);vwShape.lineTo(0, 1.4);
    const vwGeo = new THREE.ShapeGeometry(vwShape);
    const vwMat = new THREE.MeshStandardMaterial({color:new THREE.Color(col),emissive:new THREE.Color(col).multiplyScalar(0.55),side:THREE.DoubleSide,metalness:0.2,roughness:0.3,transparent:true,opacity:0.88});
    [-1,1].forEach(s=>{const vw=new THREE.Mesh(vwGeo,vwMat.clone());vw.rotation.x=Math.PI/2;vw.scale.x=s;vw.position.set(s*0.55,-0.05,0.6);group.add(vw);});
    const orbGeo = new THREE.SphereGeometry(0.42, 10, 10);
    const orbMat = new THREE.MeshStandardMaterial({color:0x220033,emissive:new THREE.Color(col),emissiveIntensity:1.2,metalness:0.0,roughness:0.0,transparent:true,opacity:0.95});
    const orb = new THREE.Mesh(orbGeo, orbMat);
    orb.position.set(0, 0.22, -0.6);group.add(orb);
    const vRingGeo = new THREE.TorusGeometry(0.62, 0.06, 7, 22);
    const vRingMat = new THREE.MeshBasicMaterial({color:new THREE.Color(col),transparent:true,opacity:0.8});
    const vRing = new THREE.Mesh(vRingGeo, vRingMat);
    vRing.rotation.x=Math.PI/2;vRing.position.set(0,0.22,-0.6);group.add(vRing);
    const tfGeo = new THREE.BoxGeometry(0.1, 1.0, 1.0);
    [-0.9,0.9].forEach(x=>{const tf=new THREE.Mesh(tfGeo,mat.clone());tf.position.set(x,0.1,1.5);group.add(tf);});
    const pgGeo = new THREE.CircleGeometry(0.65, 14);
    const pgMat = new THREE.MeshBasicMaterial({color:new THREE.Color(col),transparent:true,opacity:0.9,side:THREE.DoubleSide});
    const pg = new THREE.Mesh(pgGeo, pgMat);
    pg.rotation.y=Math.PI;pg.position.set(0,0,1.85);group.add(pg);

  } else if(id===8){
    // ══════════════════════════════════════════════════════
    // ── Eternal Seraph ────────────────────────────────────
    // Sleek diamond hull — celestial, angelic, luminous
    // ══════════════════════════════════════════════════════
    const warmGold = new THREE.Color(col);

    // Main hull — tall diamond/chevron body
    const hullGeo = new THREE.OctahedronGeometry(1.5, 0);
    const hull = new THREE.Mesh(hullGeo, makeMat(col, 0.55, 0.35, 0.15));
    hull.scale.set(0.65, 0.42, 2.1);
    hull.position.z = -0.1;
    group.add(hull);

    // Forward prow — elongated spike
    const prowGeo = new THREE.ConeGeometry(0.32, 2.2, 5);
    const prow = new THREE.Mesh(prowGeo, makeMat(col, 0.9, 0.2, 0.08));
    prow.rotation.x = -Math.PI / 2;
    prow.position.set(0, 0, -2.4);
    group.add(prow);

    // Swept seraph wings — large, feather-like panels
    const swShape = new THREE.Shape();
    swShape.moveTo(0,   0  );
    swShape.lineTo(4.2, -0.3);
    swShape.lineTo(3.4,  2.0);
    swShape.lineTo(2.0,  3.2);
    swShape.lineTo(0.6,  3.0);
    swShape.lineTo(0,    1.6);
    const swGeo = new THREE.ShapeGeometry(swShape);
    const swMat = new THREE.MeshStandardMaterial({
      color: warmGold,
      emissive: warmGold.clone().multiplyScalar(0.35),
      side: THREE.DoubleSide, metalness: 0.25, roughness: 0.2,
      transparent: true, opacity: 0.86
    });
    [-1, 1].forEach(s => {
      const sw = new THREE.Mesh(swGeo, swMat.clone());
      sw.rotation.x = Math.PI / 2;
      sw.scale.x = s;
      sw.position.set(s * 0.6, -0.08, 0.2);
      group.add(sw);
    });

    // Wing light veins — thin bright lines across each wing
    const veinMat = new THREE.MeshBasicMaterial({
      color: 0xffffff, transparent: true, opacity: 0.5, side: THREE.DoubleSide
    });
    const vShape = new THREE.Shape();
    vShape.moveTo(0.2, 0.4); vShape.lineTo(3.6, 0.1); vShape.lineTo(2.8, 1.5); vShape.lineTo(0.2, 1.4);
    const vGeo = new THREE.ShapeGeometry(vShape);
    [-1, 1].forEach(s => {
      const vein = new THREE.Mesh(vGeo, veinMat.clone());
      vein.rotation.x = Math.PI / 2;
      vein.scale.x = s;
      vein.position.set(s * 0.61, -0.07, 0.21);
      group.add(vein);
    });

    // Secondary upper feather wings — smaller, higher
    const sf2Shape = new THREE.Shape();
    sf2Shape.moveTo(0,   0  );
    sf2Shape.lineTo(2.8, 0.1);
    sf2Shape.lineTo(2.0, 1.6);
    sf2Shape.lineTo(0.4, 1.8);
    sf2Shape.lineTo(0,   0.8);
    const sf2Geo = new THREE.ShapeGeometry(sf2Shape);
    const sf2Mat = new THREE.MeshStandardMaterial({
      color: warmGold,
      emissive: warmGold.clone().multiplyScalar(0.5),
      side: THREE.DoubleSide, metalness: 0.15, roughness: 0.15,
      transparent: true, opacity: 0.72
    });
    [-1, 1].forEach(s => {
      const sf2 = new THREE.Mesh(sf2Geo, sf2Mat.clone());
      sf2.rotation.x = Math.PI / 2;
      sf2.rotation.z = -s * 0.18; // slight upward cant
      sf2.scale.x = s;
      sf2.position.set(s * 0.58, 0.5, -0.3);
      group.add(sf2);
    });

    // Halo ring — the Eternal's iconic signature
    const haloGeo = new THREE.TorusGeometry(2.1, 0.1, 10, 36);
    const haloMat = new THREE.MeshBasicMaterial({
      color: 0xfffacd, transparent: true, opacity: 0.88
    });
    const halo = new THREE.Mesh(haloGeo, haloMat);
    halo.rotation.x = Math.PI * 0.1; // slight forward tilt
    halo.position.set(0, 0.7, -0.5);
    group.add(halo);
    seraphHaloRing = halo; // track for spinning

    // Halo outer glow ring (slightly larger, dimmer)
    const haloGlowGeo = new THREE.TorusGeometry(2.5, 0.05, 6, 32);
    const haloGlowMat = new THREE.MeshBasicMaterial({
      color: 0xffd080, transparent: true, opacity: 0.35
    });
    const haloGlow = new THREE.Mesh(haloGlowGeo, haloGlowMat);
    haloGlow.rotation.x = Math.PI * 0.1;
    haloGlow.position.set(0, 0.7, -0.5);
    group.add(haloGlow);

    // Soul orb — luminous heart at cockpit
    const sOrbGeo = new THREE.SphereGeometry(0.45, 14, 14);
    const sOrbMat = new THREE.MeshStandardMaterial({
      color: new THREE.Color(0xffffff),
      emissive: warmGold.clone(),
      emissiveIntensity: 2.2,
      metalness: 0.0, roughness: 0.0,
      transparent: true, opacity: 1.0
    });
    const sOrb = new THREE.Mesh(sOrbGeo, sOrbMat);
    sOrb.position.set(0, 0.28, -0.7);
    group.add(sOrb);

    // Orb inner core (bright white)
    const coreGeo = new THREE.SphereGeometry(0.22, 8, 8);
    const coreMat = new THREE.MeshBasicMaterial({ color: 0xffffff, transparent: true, opacity: 0.95 });
    const core = new THREE.Mesh(coreGeo, coreMat);
    core.position.set(0, 0.28, -0.7);
    group.add(core);

    // Tail fin pair — swept upward
    const tfGeo = new THREE.BoxGeometry(0.08, 1.1, 0.9);
    [-0.8, 0.8].forEach(x => {
      const tf = new THREE.Mesh(tfGeo, makeMat(col, 0.4, 0.4, 0.2));
      tf.position.set(x, 0.25, 1.55);
      tf.rotation.z = x > 0 ? -0.15 : 0.15;
      group.add(tf);
    });
  }

  // Engine nozzle (shared for all)
  const nozzleGeo=new THREE.CylinderGeometry(0.3,0.5,0.4,7);
  const nozzleMat=new THREE.MeshStandardMaterial({color:0x222244,emissive:new THREE.Color(col).multiplyScalar(0.5),metalness:0.8,roughness:0.2});
  const nozzle=new THREE.Mesh(nozzleGeo,nozzleMat);
  nozzle.rotation.x=Math.PI/2; nozzle.position.set(0,0,1.5); group.add(nozzle);

  const glowGeo=new THREE.CircleGeometry(0.55,12);
  const glowMat=new THREE.MeshBasicMaterial({color:new THREE.Color(col),transparent:true,opacity:0.7,side:THREE.DoubleSide});
  const glow=new THREE.Mesh(glowGeo,glowMat);
  glow.rotation.y=Math.PI; glow.position.set(0,0,1.75); group.add(glow);

  group.scale.set(0.85,0.85,0.85);
  return group;
}

// ══════════════════════════════════════════════════════════
// ASTEROID MESH FACTORY
// ══════════════════════════════════════════════════════════
const ASTEROID_GEO=(()=>{
  const g=new THREE.IcosahedronGeometry(1.1,1);
  const pos=g.attributes.position;
  for(let i=0;i<pos.count;i++){const s=0.7+Math.random()*0.5;pos.setXYZ(i,pos.getX(i)*s,pos.getY(i)*s,pos.getZ(i)*s);}
  g.computeVertexNormals(); return g;
})();
const ASTEROID_MATS=[
  new THREE.MeshStandardMaterial({color:0x776655,roughness:0.95,metalness:0.05}),
  new THREE.MeshStandardMaterial({color:0x665544,roughness:0.9,metalness:0.1}),
  new THREE.MeshStandardMaterial({color:0x888877,roughness:0.92,metalness:0.08}),
];
function createAsteroidMesh(){
  const mat=ASTEROID_MATS[Math.floor(Math.random()*ASTEROID_MATS.length)].clone();
  const mesh=new THREE.Mesh(ASTEROID_GEO,mat);
  const s=0.8+Math.random()*0.6;
  mesh.scale.set(s,s*(0.8+Math.random()*0.4),s*(0.8+Math.random()*0.4));
  mesh.rotation.set(Math.random()*Math.PI*2,Math.random()*Math.PI*2,Math.random()*Math.PI*2);
  return mesh;
}

// ══════════════════════════════════════════════════════════
// SHIELD MESH
// ══════════════════════════════════════════════════════════
function createShieldMesh(){
  const geo=new THREE.SphereGeometry(2.2,18,14);
  const mat=new THREE.MeshBasicMaterial({color:0x00ffff,wireframe:true,transparent:true,opacity:0.35});
  const mesh=new THREE.Mesh(geo,mat); mesh.visible=false; return mesh;
}

// ══════════════════════════════════════════════════════════
// GHOST DECOY (Boosted Mythic Phantom)
// ══════════════════════════════════════════════════════════
function createGhostDecoy(fromLane){
  if(ghostDecoy){scene.remove(ghostDecoy);ghostDecoy=null;}
  const g = new THREE.Group();
  const col = "#ff77ff";
  const ghostMat = new THREE.MeshStandardMaterial({
    color:new THREE.Color(col),emissive:new THREE.Color(col).multiplyScalar(0.8),
    transparent:true,opacity:0.38,metalness:0.1,roughness:0.5,wireframe:false
  });
  const hullGeo = new THREE.ConeGeometry(1.1, 4.5, 3);
  const hull = new THREE.Mesh(hullGeo, ghostMat.clone());
  hull.rotation.x=-Math.PI/2;hull.position.z=-0.2;g.add(hull);
  const sphereGeo = new THREE.SphereGeometry(0.42, 8, 8);
  const sphereMat = ghostMat.clone();sphereMat.opacity=0.6;
  const orb = new THREE.Mesh(sphereGeo, sphereMat);
  orb.position.set(0,0.22,-0.6);g.add(orb);
  g.scale.set(0.85,0.85,0.85);
  g.position.set(LANES_3D[fromLane],0,0);
  scene.add(g);
  ghostDecoy=g;ghostDecoyLane=fromLane;ghostDecoyFrames=300;
}

function updateGhostDecoy(){
  if(!ghostDecoy)return;
  ghostDecoyFrames--;
  const pulse=0.2+Math.sin(Date.now()*0.006)*0.18;
  ghostDecoy.traverse(child=>{
    if(child.material&&child.material.opacity!==undefined){
      child.material.opacity=Math.max(0,pulse*(ghostDecoyFrames/300));
    }
  });
  ghostDecoy.rotation.y+=0.015;
  if(ghostDecoyFrames<=0){scene.remove(ghostDecoy);ghostDecoy=null;ghostDecoyLane=-1;}
}

// ══════════════════════════════════════════════════════════
// VOID BURST PARTICLES (teleport effect)
// ══════════════════════════════════════════════════════════
function spawnVoidBurst(x,y,z){
  const colors=[0xff44ff,0xaa00ff,0xffffff,0xff99ff,0x6600ff];
  for(let i=0;i<28;i++){
    const geo=new THREE.SphereGeometry(0.06+Math.random()*0.12,4,4);
    const mat=new THREE.MeshBasicMaterial({color:colors[Math.floor(Math.random()*colors.length)],transparent:true,opacity:0.95,blending:THREE.AdditiveBlending});
    const mesh=new THREE.Mesh(geo,mat);
    mesh.position.set(x+(Math.random()-.5)*1.5,y+(Math.random()-.5)*1.5,z+(Math.random()-.5)*1.5);
    const sp=0.2+Math.random()*0.18;const angle=Math.random()*Math.PI*2;
    voidParticles.push({mesh,vx:Math.cos(angle)*sp*(Math.random()-.5)*2,vy:(Math.random()-.5)*sp,vz:Math.sin(angle)*sp*(Math.random()-.5)*2,life:30+Math.floor(Math.random()*20),maxLife:50});
    scene.add(mesh);
  }
}

function updateVoidParticles(){
  for(let i=voidParticles.length-1;i>=0;i--){
    const p=voidParticles[i];
    p.mesh.position.x+=p.vx;p.mesh.position.y+=p.vy;p.mesh.position.z+=p.vz;
    p.vx*=0.92;p.vy*=0.92;p.vz*=0.92;p.life--;
    p.mesh.material.opacity=(p.life/p.maxLife)*0.9;
    p.mesh.scale.setScalar(Math.max(0.05,p.life/p.maxLife));
    if(p.life<=0){scene.remove(p.mesh);voidParticles.splice(i,1);}
  }
}

// ══════════════════════════════════════════════════════════
// SOUL BURST PARTICLES (Eternal Seraph explosion)
// ══════════════════════════════════════════════════════════
function spawnSoulBurst(){
  // Radiating golden rays from ship position
  const colors=[0xfffacd, 0xfff080, 0xffd700, 0xffffff, 0xffe4a0, 0xffcc44];
  const sx=ship.x, sy=0, sz=0;
  for(let i=0;i<48;i++){
    const geo=new THREE.SphereGeometry(0.1+Math.random()*0.18,4,4);
    const mat=new THREE.MeshBasicMaterial({
      color:colors[Math.floor(Math.random()*colors.length)],
      transparent:true,opacity:1.0,blending:THREE.AdditiveBlending
    });
    const mesh=new THREE.Mesh(geo,mat);
    mesh.position.set(sx+(Math.random()-.5)*2,sy+(Math.random()-.5)*2,sz+(Math.random()-.5)*4);
    const speed=0.28+Math.random()*0.24;
    const angle=Math.random()*Math.PI*2;
    const elevation=(Math.random()-.5)*Math.PI;
    soulParticles.push({
      mesh,
      vx:Math.cos(angle)*Math.cos(elevation)*speed,
      vy:Math.sin(elevation)*speed*0.6,
      vz:Math.sin(angle)*Math.cos(elevation)*speed - 0.08,
      life:50+Math.floor(Math.random()*20),
      maxLife:70
    });
    scene.add(mesh);
  }
  // Seraph-specific: flash halo brighter
  seraphSoulBurstFlashFrames = 45;
  soulLight.position.set(sx, sy+1, sz);
  soulLight.intensity = 22;
}

function updateSoulParticles(){
  for(let i=soulParticles.length-1;i>=0;i--){
    const p=soulParticles[i];
    p.mesh.position.x+=p.vx;p.mesh.position.y+=p.vy;p.mesh.position.z+=p.vz;
    p.vx*=0.93;p.vy*=0.93;p.vz*=0.93;p.life--;
    p.mesh.material.opacity=Math.pow(p.life/p.maxLife,0.7)*0.95;
    p.mesh.scale.setScalar(Math.max(0.04,p.life/p.maxLife*1.3));
    if(p.life<=0){scene.remove(p.mesh);soulParticles.splice(i,1);}
  }
  // Decay halo flash and soul light
  if(seraphSoulBurstFlashFrames>0){seraphSoulBurstFlashFrames--;}
  if(soulLight.intensity>0)soulLight.intensity=Math.max(0,soulLight.intensity*0.88);
}

// ══════════════════════════════════════════════════════════
// ENGINE EXHAUST PARTICLES
// ══════════════════════════════════════════════════════════
function spawnEngineParticle(x,col){
  const geo=new THREE.SphereGeometry(0.08,4,4);
  const c=new THREE.Color(col);
  const mat=new THREE.MeshBasicMaterial({color:c,transparent:true,opacity:0.9});
  const mesh=new THREE.Mesh(geo,mat);
  mesh.position.set(x+(Math.random()-.5)*0.5,(Math.random()-.5)*0.4,1.8+(Math.random()*.5));
  scene.add(mesh);
  engineExhaust.push({mesh,life:18,maxLife:18,vz:(0.1+Math.random()*0.15)});
}
function updateEngineExhaust(){
  for(let i=engineExhaust.length-1;i>=0;i--){
    const p=engineExhaust[i];
    p.mesh.position.z+=p.vz;p.mesh.position.y-=0.02;p.life--;
    p.mesh.material.opacity=(p.life/p.maxLife)*0.7;
    p.mesh.scale.setScalar(p.life/p.maxLife);
    if(p.life<=0){scene.remove(p.mesh);engineExhaust.splice(i,1);}
  }
}

// ══════════════════════════════════════════════════════════
// 3D PARTICLES (explosion)
// ══════════════════════════════════════════════════════════
function spawnParticles3D(x,y,z,n){
  for(let i=0;i<n;i++){
    const geo=new THREE.SphereGeometry(0.14,3,3);
    const mat=new THREE.MeshBasicMaterial({color:Math.random()>.5?0xff6600:0xffaa00,transparent:true,opacity:0.95});
    const mesh=new THREE.Mesh(geo,mat);
    mesh.position.set(x+(Math.random()-.5)*2,y+(Math.random()-.5)*2,z+(Math.random()-.5)*2);
    const sp=0.12;
    particles3D.push({mesh,vx:(Math.random()-.5)*sp*2,vy:(Math.random()-.5)*sp*2,vz:(Math.random()-.5)*sp*2,life:40,maxLife:40});
    scene.add(mesh);
  }
}
function updateParticles3D(){
  for(let i=particles3D.length-1;i>=0;i--){
    const p=particles3D[i];
    p.mesh.position.x+=p.vx;p.mesh.position.y+=p.vy;p.mesh.position.z+=p.vz;p.life--;
    p.mesh.material.opacity=(p.life/p.maxLife)*0.9;
    p.mesh.scale.setScalar(Math.max(0.1,p.life/p.maxLife));
    if(p.life<=0){scene.remove(p.mesh);particles3D.splice(i,1);}
  }
}

// ══════════════════════════════════════════════════════════
// LASER BEAM
// ══════════════════════════════════════════════════════════
let laserFrames3D=0,laserLane3D=0;
function updateLaser3D(){
  if(laserBeam){
    if(laserFrames3D>0){laserBeam.material.opacity=laserFrames3D/20;laserFrames3D--;}
    else{scene.remove(laserBeam);laserBeam=null;}
  }
}
function showLaserBeam(laneX){
  if(laserBeam)scene.remove(laserBeam);
  const geo=new THREE.CylinderGeometry(0.08,0.08,140,6);
  const mat=new THREE.MeshBasicMaterial({color:0xff4400,transparent:true,opacity:1,blending:THREE.AdditiveBlending});
  laserBeam=new THREE.Mesh(geo,mat);
  laserBeam.rotation.x=Math.PI/2;laserBeam.position.set(laneX,0,-60);
  scene.add(laserBeam);laserFrames3D=20;
}

// ══════════════════════════════════════════════════════════
// SCREEN TELEPORT FLASH
// ══════════════════════════════════════════════════════════
function doTeleportScreenFlash(){
  const el=document.getElementById("teleportFlash");
  el.style.background="radial-gradient(ellipse at center, rgba(255,80,255,0.7) 0%, rgba(120,0,200,0.4) 50%, transparent 100%)";
  el.style.display="block";el.classList.remove("teleport-anim");void el.offsetWidth;
  el.classList.add("teleport-anim");
  setTimeout(()=>{el.style.display="none";el.classList.remove("teleport-anim");},300);
}

// Soul burst screen flash (gold)
function doSoulBurstScreenFlash(){
  const el=document.getElementById("teleportFlash");
  el.style.background="radial-gradient(ellipse at center, rgba(255,250,150,0.75) 0%, rgba(255,200,50,0.45) 45%, transparent 100%)";
  el.style.display="block";el.classList.remove("teleport-anim");void el.offsetWidth;
  el.classList.add("teleport-anim");
  setTimeout(()=>{el.style.display="none";el.classList.remove("teleport-anim");},500);
}

// ══════════════════════════════════════════════════════════
// GAME STATE
// ══════════════════════════════════════════════════════════
let score,highScore,gameOver,baseSpeed,boostKey,difficultyTimer;
let xp,xpMilestoneNext,xpPassiveTimer,shieldActive,dodgedCount;
let legendRevives=0,frozenFrames=0,godlyCooldown=0;
let boostedShips=[],gems=0;
let chosenCosmetic=0;
let startShieldActive=false,startShieldFrames=0;
let activeRunBoosts=[];
let laserShots=0,laserFrames=0,laserLane=0;
let playerName="Pilot";
let questRunScore=0;
let animationFrameId=null,idleAnimId=null;

const ship={lane:1,x:0,targetX:0,moveSpeed:.11,tilt:0};

// ══════════════════════════════════════════════════════════
// HELPERS
// ══════════════════════════════════════════════════════════
function isSeraph(){return chosenCosmetic===8;}
function isPhantomatic(){return chosenCosmetic===7;}
function hasRunBoost(id){return activeRunBoosts.includes(id);}
function getPowerActive(name){return COSMETICS[chosenCosmetic].powerName===name;}
function isBoosted(id){return boostedShips.includes(id);}
function getBoostMultiplier(){if(chosenCosmetic===1&&isBoosted(1))return 4;if(getPowerActive("Double Boost"))return 2;return 1;}
function getAsteroidSpeedMult(){let m=1;if(chosenCosmetic===4&&isBoosted(4))m*=.6;else if(getPowerActive("Slow Asteroids"))m*=.8;if(hasRunBoost("null_field"))m*=.65;return m;}
function getMilestoneXpBonus(){if(chosenCosmetic===3&&isBoosted(3))return 2;if(getPowerActive("Milestone XP+"))return 1.5;return 1;}
function getShieldEveryDodged(){if(chosenCosmetic===2&&isBoosted(2))return 2;if(getPowerActive("Shield on Dodge"))return 5;return null;}
function getLegendRevives(){if(chosenCosmetic===5&&isBoosted(5))return 2;if(getPowerActive("Revive"))return 1;return 0;}
function getGodlyCooldownFrames(){return(chosenCosmetic===6&&isBoosted(6))?900:1800;}
// Soul Burst triggers every N dodges
function getSoulBurstEvery(){
  if(!isSeraph())return 999999;
  return(isBoosted(8))?10:20;
}
function getBoostedDesc(id){
  return[
    "Start invulnerable (shield) for 10s",
    "Boost is 4×",
    "Shield every 2 dodges",
    "+100% XP for milestones",
    "Asteroid speed −40%",
    "Two revives per game",
    "Half cooldown: F every 15s",
    "Ghost decoy on teleport — absorbs 1 hit",
    "Soul Burst every 10 dodges + double XP burst"
  ][id]||"";
}

function flashScreen(color){
  const f=document.getElementById("reviveFlash");
  f.style.background=color;f.style.display="block";
  setTimeout(()=>f.style.display="none",300);
}

// ══════════════════════════════════════════════════════════
// SOUL BURST (Eternal Seraph power)
// ══════════════════════════════════════════════════════════
function triggerSoulBurst(){
  // Annihilate all asteroids
  const count=asteroids.length;
  asteroids.forEach(a=>{
    spawnParticles3D(a.x,0,a.z,6);
    if(a.mesh)scene.remove(a.mesh);
  });
  asteroids=[];
  // XP burst
  const xpGain=isBoosted(8)?count*30:count*15;
  if(xpGain>0)addXp(xpGain);
  // VFX
  spawnSoulBurst();
  doSoulBurstScreenFlash();
  const msg=isBoosted(8)?`✨ SOUL BURST! +${xpGain} XP (BOOSTED)`:`✨ SOUL BURST! +${xpGain} XP`;
  showToast(msg,2200);
}

// ══════════════════════════════════════════════════════════
// TELEPORT MECHANICS (Mythic Phantom)
// ══════════════════════════════════════════════════════════
function doPhantomTeleport(targetLane){
  if(isTeleporting||gameOver)return;
  const fromLane=ship.lane;
  if(fromLane===targetLane)return;
  const fromX=LANES_3D[fromLane],toX=LANES_3D[targetLane];
  isTeleporting=true;teleportFrames=TELEPORT_DURATION;
  spawnVoidBurst(fromX,0,0);
  if(isBoosted(7))createGhostDecoy(fromLane);
  if(shipGroup)shipGroup.visible=false;
  doTeleportScreenFlash();
  voidLight.position.set(fromX,0,0);voidLight.intensity=18;
  setTimeout(()=>{
    ship.lane=targetLane;ship.x=toX;ship.targetX=toX;
    if(shipGroup){shipGroup.position.x=toX;shipGroup.visible=true;}
    spawnVoidBurst(toX,0,0);doTeleportScreenFlash();
    voidLight.position.set(toX,0,0);voidLight.intensity=14;
    isTeleporting=false;
  },TELEPORT_DURATION*16);
}

// ══════════════════════════════════════════════════════════
// PERSISTENCE
// ══════════════════════════════════════════════════════════
function saveProgress(){
  localStorage.setItem("sd2_xp",xp);
  localStorage.setItem("sd2_cosmetic",chosenCosmetic);
  localStorage.setItem("sd2_boosted",JSON.stringify(boostedShips));
  localStorage.setItem("sd2_gems",gems);
  localStorage.setItem("sd2_name",playerName);
}
function loadProgress(){
  xp=parseInt(localStorage.getItem("sd2_xp")||"0")||0;
  chosenCosmetic=parseInt(localStorage.getItem("sd2_cosmetic")||"0")||0;
  boostedShips=JSON.parse(localStorage.getItem("sd2_boosted")||"[]");
  gems=parseInt(localStorage.getItem("sd2_gems")||"0")||0;
  playerName=localStorage.getItem("sd2_name")||"Pilot";
  xpMilestoneNext=XP_MILESTONE_STEP;
  document.getElementById("playerNameInput").value=playerName;
}

// ══════════════════════════════════════════════════════════
// GEM / XP UI
// ══════════════════════════════════════════════════════════
function updateGemsUi(){document.querySelectorAll(".gemCount").forEach(el=>el.innerText=gems);}
function getXpRank(v){for(let i=XP_RANKS.length-1;i>=0;i--)if(v>=XP_RANKS[i].xp)return XP_RANKS[i];return XP_RANKS[0];}
function nextXpRank(){const idx=XP_RANKS.findIndex(r=>r.xp===getXpRank(xp).xp);return XP_RANKS[idx+1]||null;}
function getXpProgress(){const next=nextXpRank();if(!next)return 100;const cur=getXpRank(xp);return Math.min(100,Math.max(0,(xp-cur.xp)/(next.xp-cur.xp)*100));}
function getXpProgressText(){const next=nextXpRank();return next?`${xp} / ${next.xp} XP`:`${xp} / MAX`;}
function updateAllRankUi(){
  const rank=getXpRank(xp),pct=getXpProgress(),txt=getXpProgressText();
  updateGemsUi();
  const hr=document.getElementById("homeRank");hr.innerText=rank.name.toUpperCase();hr.style.color=rank.color;
  document.getElementById("homeXpLabel").innerText="XP: "+xp;
  document.getElementById("homeRankFill").style.width=pct+"%";
  document.getElementById("homeRankFill").style.background=`linear-gradient(90deg,${rank.color},#ffff00)`;
  document.getElementById("homeRankText").innerText=txt;
  const rd=document.getElementById("rankDisplay");rd.innerText=rank.name.toUpperCase();rd.style.color=rank.color;
  document.getElementById("xpDisplay").innerText="XP: "+xp;
  document.getElementById("gameRankFill").style.width=pct+"%";
  document.getElementById("gameRankFill").style.background=`linear-gradient(90deg,${rank.color},#ffff00)`;
  document.getElementById("gameRankText").innerText=txt;
  document.getElementById("boosterBtn").disabled=(gems<1);
}
function addXp(amount){
  const mult=hasRunBoost("xp_surge")?3:1;
  const oldRank=getXpRank(xp);xp+=amount*mult;const newRank=getXpRank(xp);
  if(newRank.name!==oldRank.name){gems++;updateGemsUi();saveProgress();showToast("+1 💎 for reaching "+newRank.name+"!");triggerRankUpAnimation();}
  saveProgress();updateAllRankUi();
}
function triggerRankUpAnimation(){const el=document.getElementById("rankUpAnimation");el.style.display="block";el.offsetHeight;setTimeout(()=>el.style.display="none",1100);}
function showToast(msg,duration=2200){const t=document.getElementById("toast");t.innerText=msg;t.style.display="block";clearTimeout(t._tid);t._tid=setTimeout(()=>t.style.display="none",duration);}

// ══════════════════════════════════════════════════════════
// SHIP BOOSTER
// ══════════════════════════════════════════════════════════
document.getElementById("boosterBtn").onclick=function(){
  if(gems<1)return;
  const avail=COSMETICS.map(c=>c.id).filter(id=>!isBoosted(id));
  if(!avail.length){showToast("All ships already BOOSTED!");return;}
  const r=avail[Math.floor(Math.random()*avail.length)];
  boostedShips.push(r);gems--;saveProgress();updateAllRankUi();
  drawHomeCosmetics();drawPowerDesc();
  showToast(`BOOSTED: ${COSMETICS[r].name} — ${getBoostedDesc(r)}`,4000);
  document.getElementById("boosterBtn").disabled=(gems<1);
};

// ══════════════════════════════════════════════════════════
// RUN BOOSTS
// ══════════════════════════════════════════════════════════
function toggleRunBoost(id){
  const def=RUN_BOOSTS.find(b=>b.id===id);if(!def)return;
  if(hasRunBoost(id)){activeRunBoosts=activeRunBoosts.filter(b=>b!==id);gems+=def.cost;saveProgress();updateAllRankUi();}
  else{if(gems<def.cost){showToast("Not enough 💎 Gems!");return;}gems-=def.cost;activeRunBoosts.push(id);saveProgress();updateAllRankUi();}
  renderBoostsTab();
}
function renderBoostsTab(){
  const grid=document.getElementById("boostsGrid");grid.innerHTML="";
  RUN_BOOSTS.forEach(b=>{
    const eq=hasRunBoost(b.id),afford=gems>=b.cost;
    const d=document.createElement("div");
    d.className="boostCard"+(eq?" equipped":"");
    d.innerHTML=`<div class="bn">${b.name}</div><div class="bd">${b.desc}</div><div class="bc">${eq?"✅ Equipped":"Cost: "+b.cost+" 💎"}</div><button class="${eq?'unequip':''}" ${!eq&&!afford?"disabled":""} onclick="toggleRunBoost('${b.id}')">${eq?"Unequip":"Equip"}</button>`;
    grid.appendChild(d);
  });
  const el=document.getElementById("activeBoostsList");
  if(!activeRunBoosts.length){el.innerText="No run boosts equipped.";return;}
  el.innerText="Active: "+activeRunBoosts.map(id=>RUN_BOOSTS.find(b=>b.id===id).name).join(" | ");
}

// ══════════════════════════════════════════════════════════
// QUESTS
// ══════════════════════════════════════════════════════════
function loadQuestState(){try{return JSON.parse(localStorage.getItem("sd2_qstate")||"null")}catch(e){return null}}
function saveQuestState(s){localStorage.setItem("sd2_qstate",JSON.stringify(s));}
function seededRng(seed){let s=seed;return()=>{s=(s*9301+49297)%233280;return s/233280;};}
function generateBatch(batchIndex){
  const tier=QUEST_TIERS[Math.min(batchIndex,QUEST_TIERS.length-1)];
  const rng=seededRng(batchIndex*997+42);
  const unlocked=COSMETICS.filter(c=>xp>=c.unlockXp);
  const pool=(unlocked.length>=3?unlocked:COSMETICS).map(c=>c.id);
  const shuffled=[...pool].sort(()=>rng()-.5);
  const picked=shuffled.slice(0,3);
  const tgts=[...tier.targets].sort(()=>rng()-.5);
  return picked.map((shipId,i)=>({shipId,shipName:COSMETICS[shipId].name,target:tgts[i],gems:tier.gems,done:false}));
}
function getOrCreateQuestState(){
  let state=loadQuestState();
  if(!state){state={batchIndex:0,batch:generateBatch(0),totalCompleted:0};saveQuestState(state);}
  return state;
}
function checkAndAdvanceBatch(state){
  if(state.batch.every(q=>q.done)){state.batchIndex++;state.batch=generateBatch(state.batchIndex);saveQuestState(state);showToast("🎉 All quests done! New batch unlocked!",3000);renderQuestsTab();}
}
function checkQuestsAfterRun(){
  const state=getOrCreateQuestState();let anyCompleted=false;
  state.batch.forEach(q=>{
    if(q.done)return;
    if(questRunScore>=q.target&&chosenCosmetic===q.shipId){
      q.done=true;state.totalCompleted++;gems+=q.gems;saveProgress();updateAllRankUi();
      showToast(`✅ Quest: Score ${q.target} with ${q.shipName} +${q.gems}💎`,3000);anyCompleted=true;
    }
  });
  if(anyCompleted){saveQuestState(state);checkAndAdvanceBatch(state);}
}
function renderQuestsTab(){
  const list=document.getElementById("questsList");list.innerHTML="";
  const state=getOrCreateQuestState();
  const doneCount=state.batch.filter(q=>q.done).length;
  const tier=QUEST_TIERS[Math.min(state.batchIndex,QUEST_TIERS.length-1)];
  document.getElementById("questsHeader").innerHTML=`Batch ${state.batchIndex+1} · ${doneCount}/3 complete · Each quest: ${tier.gems}💎<br>Complete all 3 to unlock the next batch`;
  state.batch.forEach((q,i)=>{
    const sc=COSMETICS[q.shipId];
    const prog=Math.min(q.target,questRunScore);
    const pct=q.done?100:Math.min(100,prog/q.target*100);
    const rarity=i===2?"epic":i===1?"rare":"common";
    const d=document.createElement("div");
    d.className=`questCard questRarity-${rarity}${q.done?" done":""}`;
    d.innerHTML=`<div class="qr">${q.gems}💎</div><div class="qt">Score ${q.target.toLocaleString()} with ${sc.name}</div><div class="qd">${sc.powerDesc}</div>${q.done?`<div class="qdone">✅ Completed!</div>`:`<div class="qbar"><div class="qfill" style="width:${pct}%"></div></div><div class="qprog">${chosenCosmetic===q.shipId?prog:0} / ${q.target} ${chosenCosmetic!==q.shipId?"(select this ship)":""}</div>`}`;
    list.appendChild(d);
  });
  const note=document.createElement("div");
  note.style.cssText="font-size:10px;color:#444;margin-top:10px;text-align:center";
  note.innerText=`Total quests completed: ${state.totalCompleted}`;
  list.appendChild(note);
}

// ══════════════════════════════════════════════════════════
// TABS
// ══════════════════════════════════════════════════════════
document.querySelectorAll(".tabBtn").forEach(btn=>{
  btn.addEventListener("click",()=>{
    document.querySelectorAll(".tabBtn").forEach(b=>b.classList.remove("active"));
    document.querySelectorAll(".tabPanel").forEach(p=>p.classList.remove("active"));
    btn.classList.add("active");document.getElementById(btn.dataset.tab).classList.add("active");
    if(btn.dataset.tab==="questsTab")renderQuestsTab();
    if(btn.dataset.tab==="boostsTab")renderBoostsTab();
  });
});
document.getElementById("playerNameInput").addEventListener("input",function(){playerName=this.value;saveProgress();});

// ══════════════════════════════════════════════════════════
// HOMESCREEN SHIP PREVIEW
// ══════════════════════════════════════════════════════════
function drawHomeCosmetics(){
  const list=document.getElementById("cosmeticList");list.innerHTML="";
  const svgByShip=[
    `<svg viewBox="-22 -30 44 68" width="60" height="60" xmlns="http://www.w3.org/2000/svg"><polygon points="0,-28 -18,28 18,28" fill="#00ffcc" opacity="0.9"/><rect x="-4" y="-4" width="8" height="16" fill="#00aaaa" rx="2"/><polygon points="-18,28 -22,38 -12,30" fill="#00ddaa"/><polygon points="18,28 22,38 12,30" fill="#00ddaa"/></svg>`,
    `<svg viewBox="-26 -30 52 68" width="60" height="60" xmlns="http://www.w3.org/2000/svg"><polygon points="0,-28 -20,20 -10,36 10,36 20,20" fill="#00FF00" opacity="0.9"/><polygon points="-26,8 -20,20 0,0" fill="#00cc00"/><polygon points="26,8 20,20 0,0" fill="#00cc00"/></svg>`,
    `<svg viewBox="-20 -30 40 68" width="60" height="60" xmlns="http://www.w3.org/2000/svg"><ellipse cx="0" cy="0" rx="14" ry="26" fill="#0099FF" opacity="0.9"/><ellipse cx="-14" cy="4" rx="6" ry="4" fill="#0077cc"/><ellipse cx="14" cy="4" rx="6" ry="4" fill="#0077cc"/></svg>`,
    `<svg viewBox="-22 -32 44 70" width="60" height="60" xmlns="http://www.w3.org/2000/svg"><polygon points="0,-30 -20,16 0,38 20,16" fill="#FF6600" opacity="0.9"/><polygon points="-20,16 -28,22 -10,18" fill="#cc4400"/><polygon points="20,16 28,22 10,18" fill="#cc4400"/></svg>`,
    `<svg viewBox="-24 -20 48 52" width="60" height="60" xmlns="http://www.w3.org/2000/svg"><ellipse cx="0" cy="0" rx="20" ry="12" fill="#FF00FF" opacity="0.85"/><ellipse cx="0" cy="0" rx="24" ry="4" fill="none" stroke="#FF00FF" stroke-width="2.5" opacity="0.7"/><ellipse cx="0" cy="-4" rx="10" ry="7" fill="#cc00cc" opacity="0.9"/></svg>`,
    `<svg viewBox="-20 -32 40 68" width="60" height="60" xmlns="http://www.w3.org/2000/svg"><rect x="-16" y="-16" width="32" height="44" fill="#FFFF00" opacity="0.9" rx="3"/><polygon points="0,-32 -12,-16 12,-16" fill="#ffdd00"/><rect x="-20" y="-4" width="40" height="10" fill="#cccc00" rx="2"/></svg>`,
    `<svg viewBox="-26 -34 52 72" width="60" height="60" xmlns="http://www.w3.org/2000/svg"><ellipse cx="0" cy="0" rx="16" ry="26" fill="#00fff7" opacity="0.9"/><polygon points="0,-34 -18,8 18,8" fill="#00ddee" opacity="0.8"/><polygon points="-16,4 -26,20 -6,24 -12,10" fill="#ffffff" opacity="0.6"/><polygon points="16,4 26,20 6,24 12,10" fill="#ffffff" opacity="0.6"/></svg>`,
    // Mythic Phantom
    `<svg viewBox="-26 -36 52 76" width="60" height="60" xmlns="http://www.w3.org/2000/svg">
      <defs><radialGradient id="vg" cx="50%" cy="50%" r="50%"><stop offset="0%" stop-color="#ffffff" stop-opacity="0.9"/><stop offset="100%" stop-color="#ff77ff" stop-opacity="0.2"/></radialGradient></defs>
      <polygon points="0,-34 -13,24 13,24" fill="#ff77ff" opacity="0.92"/>
      <polygon points="-13,12 -26,28 -4,30" fill="#cc44cc" opacity="0.8"/>
      <polygon points="13,12 26,28 4,30" fill="#cc44cc" opacity="0.8"/>
      <polygon points="-13,12 -22,24 -8,26" fill="#ffffff" opacity="0.25"/>
      <polygon points="13,12 22,24 8,26" fill="#ffffff" opacity="0.25"/>
      <circle cx="0" cy="-4" r="7" fill="url(#vg)" opacity="0.95"/>
      <circle cx="0" cy="-4" r="7" fill="none" stroke="#ff77ff" stroke-width="1.5" opacity="0.8"/>
      <ellipse cx="0" cy="-4" rx="10" ry="3.5" fill="none" stroke="#ff99ff" stroke-width="1" opacity="0.6"/>
    </svg>`,
    // Eternal Seraph — diamond hull, swept angel wings, halo ring, soul orb
    `<svg viewBox="-28 -40 56 82" width="60" height="60" xmlns="http://www.w3.org/2000/svg">
      <defs>
        <radialGradient id="sg" cx="50%" cy="50%" r="50%">
          <stop offset="0%" stop-color="#ffffff" stop-opacity="1"/>
          <stop offset="60%" stop-color="#fffacd" stop-opacity="0.8"/>
          <stop offset="100%" stop-color="#ffd700" stop-opacity="0.1"/>
        </radialGradient>
        <radialGradient id="hg" cx="50%" cy="50%" r="50%">
          <stop offset="0%" stop-color="#fffacd" stop-opacity="0.6"/>
          <stop offset="100%" stop-color="#ffd080" stop-opacity="0"/>
        </radialGradient>
      </defs>
      <!-- hull diamond -->
      <polygon points="0,-37 -11,18 0,32 11,18" fill="#fffacd" opacity="0.93"/>
      <!-- hull highlight -->
      <polygon points="0,-37 -5,0 5,0" fill="#ffffff" opacity="0.35"/>
      <!-- large swept wings -->
      <polygon points="-11,4 -28,22 -10,30 -4,20" fill="#e8d880" opacity="0.82"/>
      <polygon points="11,4 28,22 10,30 4,20" fill="#e8d880" opacity="0.82"/>
      <!-- wing veins -->
      <line x1="-11" y1="4" x2="-24" y2="20" stroke="#ffffff" stroke-width="0.8" opacity="0.5"/>
      <line x1="11" y1="4" x2="24" y2="20" stroke="#ffffff" stroke-width="0.8" opacity="0.5"/>
      <!-- upper mini wings -->
      <polygon points="-11,0 -22,10 -6,14" fill="#fff0a0" opacity="0.65"/>
      <polygon points="11,0 22,10 6,14" fill="#fff0a0" opacity="0.65"/>
      <!-- halo ring (tilted ellipse) -->
      <ellipse cx="0" cy="-16" rx="20" ry="5.5" fill="url(#hg)" opacity="0.4"/>
      <ellipse cx="0" cy="-16" rx="20" ry="5.5" fill="none" stroke="#fffacd" stroke-width="2" opacity="0.95"/>
      <ellipse cx="0" cy="-16" rx="23" ry="6.5" fill="none" stroke="#ffd080" stroke-width="0.8" opacity="0.4"/>
      <!-- soul orb -->
      <circle cx="0" cy="-10" r="8" fill="url(#sg)" opacity="1"/>
      <circle cx="0" cy="-10" r="8" fill="none" stroke="#fffacd" stroke-width="1.2" opacity="0.85"/>
      <!-- orb inner core -->
      <circle cx="0" cy="-10" r="3.5" fill="#ffffff" opacity="0.95"/>
      <!-- engine glow -->
      <ellipse cx="0" cy="34" rx="5" ry="2.5" fill="#fffacd" opacity="0.55"/>
    </svg>`,
  ];
  COSMETICS.forEach(sc=>{
    const unlocked=xp>=sc.unlockXp;
    const div=document.createElement("div");
    div.className="cosmeticShip"+(chosenCosmetic===sc.id?" selected":"")+(unlocked?"":" locked");
    div.title=sc.name+(unlocked?"":" (Locked — needs "+sc.unlockXp+" XP)");
    div.style.cssText="width:82px;height:100px;";
    div.innerHTML=`<div style="padding:6px 4px 0">${svgByShip[sc.id]||""}</div><div style="font-size:9px;color:#aaa;margin-top:2px;padding:0 3px">${sc.name}</div><div style="font-size:8px;color:${sc.color};margin:2px 0;font-weight:bold">${unlocked?sc.powerName:"🔒 "+sc.unlockXp+" XP"}</div>`;
    if(isBoosted(sc.id)){const b=document.createElement("span");b.className="booster-badge";b.innerText="BOOST";div.appendChild(b);}
    if(unlocked)div.onclick=()=>{chosenCosmetic=sc.id;saveProgress();drawHomeCosmetics();drawPowerDesc();};
    list.appendChild(div);
  });
}
function drawPowerDesc(){
  document.getElementById("cosmeticPowerDesc").innerText=isBoosted(chosenCosmetic)?"BOOSTED: "+getBoostedDesc(chosenCosmetic):COSMETICS[chosenCosmetic].powerDesc;
}

// ══════════════════════════════════════════════════════════
// CLEAR 3D OBJECTS
// ══════════════════════════════════════════════════════════
function clearGameObjects(){
  asteroids.forEach(a=>{if(a.mesh)scene.remove(a.mesh)});asteroids=[];
  particles3D.forEach(p=>scene.remove(p.mesh));particles3D=[];
  voidParticles.forEach(p=>scene.remove(p.mesh));voidParticles=[];
  soulParticles.forEach(p=>scene.remove(p.mesh));soulParticles=[];
  engineExhaust.forEach(p=>scene.remove(p.mesh));engineExhaust=[];
  if(laserBeam){scene.remove(laserBeam);laserBeam=null;}
  if(shipGroup){scene.remove(shipGroup);shipGroup=null;}
  if(shieldMesh){scene.remove(shieldMesh);shieldMesh=null;}
  if(ghostDecoy){scene.remove(ghostDecoy);ghostDecoy=null;ghostDecoyLane=-1;ghostDecoyFrames=0;}
  isTeleporting=false;teleportFrames=0;
  voidLight.intensity=0;soulLight.intensity=0;
  seraphHaloRing=null;seraphSoulBurstFlashFrames=0;
}

// ══════════════════════════════════════════════════════════
// HOMESCREEN SHOW/HIDE
// ══════════════════════════════════════════════════════════
function showHomescreen(){
  if(animationFrameId!==null){cancelAnimationFrame(animationFrameId);animationFrameId=null;}
  clearGameObjects();
  document.getElementById("homescreen").style.display="block";
  document.getElementById("ui").style.display="none";
  document.getElementById("laserDisplay").style.display="none";
  document.getElementById("godlyCooldown").style.display="none";
  document.getElementById("soulBurstDisplay").style.display="none";
  document.getElementById("controls-hint").style.display="none";
  updateAllRankUi();drawHomeCosmetics();drawPowerDesc();renderQuestsTab();
  if(idleAnimId)cancelAnimationFrame(idleAnimId);
  const idleAnimate=()=>{updateStarfield(3);renderer.render(scene,camera);idleAnimId=requestAnimationFrame(idleAnimate);};
  idleAnimate();
}
function hideHomescreen(){
  if(idleAnimId){cancelAnimationFrame(idleAnimId);idleAnimId=null;}
  document.getElementById("homescreen").style.display="none";
  document.getElementById("ui").style.display="block";
  document.getElementById("controls-hint").style.display="block";
}

// ══════════════════════════════════════════════════════════
// GAME RESET
// ══════════════════════════════════════════════════════════
function resetGame(){
  if(animationFrameId!==null){cancelAnimationFrame(animationFrameId);animationFrameId=null;}
  clearGameObjects();
  baseSpeed=4;score=0;boostKey=false;difficultyTimer=0;
  shieldActive=false;xpPassiveTimer=0;gameOver=false;
  dodgedCount=0;legendRevives=0;frozenFrames=0;godlyCooldown=0;
  questRunScore=0;laserShots=0;laserFrames3D=0;laserLane3D=0;
  seraphWingGlow=0;seraphSoulBurstFlashFrames=0;
  loadProgress();
  const boostedCadet=(chosenCosmetic===0&&isBoosted(0));
  startShieldActive=boostedCadet;startShieldFrames=boostedCadet?600:0;
  if(hasRunBoost("iron_shell"))shieldActive=true;
  ship.moveSpeed=hasRunBoost("hyperdrive")?.22:.11;
  laserShots=hasRunBoost("laser_shot")?3:0;
  activeRunBoosts=[];saveProgress();
  ship.lane=1;ship.x=LANES_3D[1];ship.targetX=LANES_3D[1];ship.tilt=0;
  highScore=parseInt(localStorage.getItem("sd2_hs")||"0")||0;
  document.getElementById("score").innerText=0;
  document.getElementById("highScore").innerText=highScore;
  document.getElementById("gameOver").style.display="none";
  document.getElementById("reviveFlash").style.display="none";
  document.getElementById("godlyCooldown").style.display="none";
  const ld=document.getElementById("laserDisplay");
  if(laserShots>0){ld.style.display="block";document.getElementById("laserLeft").innerText=laserShots;}else{ld.style.display="none";}
  // Soul Burst counter display
  const sd=document.getElementById("soulBurstDisplay");
  if(isSeraph()){
    sd.style.display="block";
    document.getElementById("soulBurstCount").innerText=getSoulBurstEvery();
  } else {sd.style.display="none";}
  shipGroup=createShipMesh(chosenCosmetic);
  shipGroup.position.set(LANES_3D[1],0,0);scene.add(shipGroup);
  shieldMesh=createShieldMesh();shieldMesh.position.set(LANES_3D[1],0,0);scene.add(shieldMesh);
  engineLight.color.set(COSMETICS[chosenCosmetic].color);
  if(isPhantomatic())voidLight.color.set(0xff44ff);
  if(isSeraph())soulLight.color.set(0xfffacd);
  updateAllRankUi();renderBoostsTab();
}

// ══════════════════════════════════════════════════════════
// LASER
// ══════════════════════════════════════════════════════════
function fireLaser(){
  if(laserShots<=0||gameOver)return;
  laserShots--;laserLane3D=ship.lane;
  showLaserBeam(LANES_3D[ship.lane]);
  for(let i=asteroids.length-1;i>=0;i--){
    if(asteroids[i].x===LANES_3D[ship.lane]){scene.remove(asteroids[i].mesh);asteroids.splice(i,1);}
  }
  document.getElementById("laserLeft").innerText=laserShots;
  if(laserShots<=0)document.getElementById("laserDisplay").style.display="none";
  showToast("🔥 LASER FIRED"+(laserShots>0?" ("+laserShots+" left)":"— all shots used!"),1200);
}

// ══════════════════════════════════════════════════════════
// ASTEROIDS
// ══════════════════════════════════════════════════════════
function spawnAsteroid(){
  const count=Math.random()<.8?1:2;
  const shuffled=[...LANES_3D].sort(()=>Math.random()-.5);
  let spawned=0;
  for(let i=0;i<shuffled.length&&spawned<count;i++){
    const lx=shuffled[i];
    if(!asteroids.find(a=>a.x===lx&&a.z<SPAWN_Z+30)){
      const mesh=createAsteroidMesh();mesh.position.set(lx,0,SPAWN_Z);scene.add(mesh);
      asteroids.push({x:lx,z:SPAWN_Z,mesh,rotX:(Math.random()-.5)*0.04,rotY:(Math.random()-.5)*0.04});
      spawned++;
    }
  }
}
function preventImpossibleRows(){
  const tops=LANES_3D.map(lx=>asteroids.filter(a=>a.x===lx&&a.z<SPAWN_Z+40));
  if(tops.every(arr=>arr.length>0)){
    const li=Math.floor(Math.random()*3),ar=tops[li][0];
    if(ar){const idx=asteroids.indexOf(ar);if(idx>-1){scene.remove(ar.mesh);asteroids.splice(idx,1);}}
  }
}

function updateAsteroids3D(speed3D){
  for(let index=asteroids.length-1;index>=0;index--){
    const a=asteroids[index];
    const aspd=frozenFrames>0?0:speed3D*getAsteroidSpeedMult();
    a.z+=aspd;
    if(a.mesh){a.mesh.position.z=a.z;a.mesh.rotation.x+=a.rotX;a.mesh.rotation.y+=a.rotY;}
    if(a.z>PASS_Z){
      scene.remove(a.mesh);asteroids.splice(index,1);
      if(!gameOver){
        score+=10;questRunScore=score;
        document.getElementById("score").innerText=score;
        addXp(XP_PER_ASTEROID);dodgedCount++;
        const every=getShieldEveryDodged();
        if(every&&dodgedCount%every===0)shieldActive=true;
        while(score>=xpMilestoneNext){addXp(Math.floor(XP_PER_MILESTONE*getMilestoneXpBonus()));xpMilestoneNext+=XP_MILESTONE_STEP;}
        saveProgress();
        if(score>highScore){highScore=score;localStorage.setItem("sd2_hs",highScore);document.getElementById("highScore").innerText=highScore;}

        // ── Soul Burst check (Eternal Seraph) ─────────────────
        if(isSeraph()&&dodgedCount>0&&dodgedCount%getSoulBurstEvery()===0){
          triggerSoulBurst();
        }
        // Update Soul Burst countdown display
        if(isSeraph()){
          const remaining=getSoulBurstEvery()-(dodgedCount%getSoulBurstEvery());
          document.getElementById("soulBurstCount").innerText=remaining===getSoulBurstEvery()?getSoulBurstEvery():remaining;
        }
      }
      continue;
    }
    // ── Collision check ───────────────────────────────────
    if(Math.abs(a.x-ship.x)<2.0&&a.z>-2&&a.z<2.5){
      if(isPhantomatic()&&isTeleporting)continue;
      if(isPhantomatic()&&isBoosted(7)&&ghostDecoy&&ghostDecoyLane===ship.lane){
        spawnVoidBurst(a.x,0,a.z);
        scene.remove(ghostDecoy);ghostDecoy=null;ghostDecoyLane=-1;ghostDecoyFrames=0;
        scene.remove(a.mesh);asteroids.splice(index,1);
        flashScreen("rgba(200,0,255,.3)");
        showToast("👻 Ghost Decoy absorbed the hit!",1400);
        continue;
      }
      if(chosenCosmetic===0&&isBoosted(0)&&startShieldActive)continue;
      if(shieldActive){
        shieldActive=false;flashScreen("rgba(255,150,0,.4)");
        spawnParticles3D(ship.x,0,0,10);scene.remove(a.mesh);asteroids.splice(index,1);continue;
      }
      if(getPowerActive("Revive")&&legendRevives<getLegendRevives()){
        legendRevives++;
        flashScreen("rgba(0,255,200,.3)");
        document.getElementById("reviveFlash").style.display="block";
        setTimeout(()=>document.getElementById("reviveFlash").style.display="none",300);
        spawnParticles3D(a.x,0,a.z,15);scene.remove(a.mesh);asteroids.splice(index,1);continue;
      }
      spawnParticles3D(ship.x,0,0,25);
      gameOver=true;
      document.getElementById("gameOver").style.display="block";
      checkQuestsAfterRun();
    }
  }
  if(frozenFrames>0)frozenFrames--;
  if(startShieldActive&&startShieldFrames>0){startShieldFrames--;if(!startShieldFrames)startShieldActive=false;}
}

// ══════════════════════════════════════════════════════════
// SHIP UPDATE
// ══════════════════════════════════════════════════════════
function updateShip3D(){
  if(!isPhantomatic()){
    ship.x+=(ship.targetX-ship.x)*ship.moveSpeed;
    ship.tilt=(ship.targetX-ship.x)*.04;
  } else {
    ship.tilt=0;
  }

  if(shipGroup){
    shipGroup.position.x=ship.x;
    shipGroup.rotation.z=-ship.tilt;
    shipGroup.position.y=Math.sin(Date.now()*0.002)*0.12;

    // Phantom void orb ring spin
    if(isPhantomatic()){
      shipGroup.children.forEach(child=>{
        if(child.geometry&&child.geometry.type==="TorusGeometry"){child.rotation.z+=0.04;}
      });
    }

    // Eternal Seraph — halo spin + soul glow pulse
    if(isSeraph()&&seraphHaloRing){
      seraphHaloRing.rotation.z+=0.018;
      // Pulse halo brightness during soul burst flash
      const burstGlow=seraphSoulBurstFlashFrames>0?(seraphSoulBurstFlashFrames/45):0;
      const baseGlow=0.7+Math.sin(Date.now()*0.003)*0.15;
      seraphHaloRing.material.opacity=Math.min(1.0,baseGlow+burstGlow*0.9);
      // Gentle wing pulsing light
      seraphWingGlow=0.5+Math.sin(Date.now()*0.0025)*0.3+burstGlow*0.5;
      if(soulLight.intensity<1){
        soulLight.intensity=seraphWingGlow*1.8;
        soulLight.position.set(ship.x,1,-0.5);
      }
    }
  }

  // Shield
  const shieldOn=(chosenCosmetic===0&&isBoosted(0)&&startShieldActive)||(shieldActive);
  if(shieldMesh){
    shieldMesh.position.x=ship.x;
    shieldMesh.position.y=shipGroup?shipGroup.position.y:0;
    shieldMesh.visible=shieldOn;
    if(shieldOn){shieldMesh.rotation.y+=0.02;shieldMesh.rotation.x+=0.01;}
  }

  engineLight.position.set(ship.x,shipGroup?shipGroup.position.y:0,2);
  const pulse=1.5+Math.sin(Date.now()*0.01)*0.7+(boostKey?2:0);
  engineLight.intensity=pulse;
  shipPointLight.position.x=ship.x;

  if(voidLight.intensity>0)voidLight.intensity=Math.max(0,voidLight.intensity*0.85);

  if(Math.random()<0.35&&!isTeleporting){
    spawnEngineParticle(ship.x,COSMETICS[chosenCosmetic].color);
  }
}

// ══════════════════════════════════════════════════════════
// PASSIVE XP + GODLY COOLDOWN
// ══════════════════════════════════════════════════════════
function updatePassiveXp(){
  if(!gameOver){xpPassiveTimer++;if(xpPassiveTimer>=60){addXp(XP_PASSIVE_SEC);xpPassiveTimer=0;}}
}
function updateGodlyCooldown(){
  if(getPowerActive("Time Annihilate")&&!gameOver){
    const el=document.getElementById("godlyCooldown");
    el.style.display="block";
    el.innerText=godlyCooldown>0?`⏳ Godly: ${Math.ceil(godlyCooldown/60)}s`:`Press F — Time Annihilate!`;
  } else {
    document.getElementById("godlyCooldown").style.display="none";
  }
}

// ══════════════════════════════════════════════════════════
// CAMERA
// ══════════════════════════════════════════════════════════
function updateCamera(){
  camera.position.x+=(ship.x*0.18-camera.position.x)*0.04;
  camera.lookAt(new THREE.Vector3(ship.x*0.12,-0.5,-20));
}

// ══════════════════════════════════════════════════════════
// MAIN GAME LOOP
// ══════════════════════════════════════════════════════════
function update(){
  const spd2d=frozenFrames>0?0:baseSpeed+(baseSpeed*.5*getBoostMultiplier())+(boostKey?4*getBoostMultiplier():0);
  const spd3D=spd2d*SPEED_SCALE;
  updateStarfield(spd2d);
  if(!gameOver){
    difficultyTimer++;
    if(frozenFrames===0&&difficultyTimer%300===0)baseSpeed+=.5;
    updateShip3D();
    updateAsteroids3D(spd3D);
    preventImpossibleRows();
    if(frozenFrames===0&&Math.random()<.03)spawnAsteroid();
    if(godlyCooldown>0)godlyCooldown--;
  }
  updateLaser3D();
  updateEngineExhaust();
  updateParticles3D();
  updateVoidParticles();
  updateSoulParticles();
  updateGhostDecoy();
  updatePassiveXp();
  updateGodlyCooldown();
  updateCamera();
  renderer.render(scene,camera);
  animationFrameId=requestAnimationFrame(update);
}

// ══════════════════════════════════════════════════════════
// START
// ══════════════════════════════════════════════════════════
document.getElementById("startBtn").onclick=()=>{
  hideHomescreen();resetGame();animationFrameId=requestAnimationFrame(update);
};

loadProgress();showHomescreen();

// ══════════════════════════════════════════════════════════
// INPUT
// ══════════════════════════════════════════════════════════
document.addEventListener("keydown",e=>{
  if(document.getElementById("ui").style.display==="none")return;
  if(e.key==="ArrowLeft"||e.key==="a"){
    if(isPhantomatic()){if(ship.lane>0)doPhantomTeleport(ship.lane-1);}
    else{if(ship.lane>0){ship.lane--;ship.targetX=LANES_3D[ship.lane];}}
  }
  if(e.key==="ArrowRight"||e.key==="d"){
    if(isPhantomatic()){if(ship.lane<2)doPhantomTeleport(ship.lane+1);}
    else{if(ship.lane<2){ship.lane++;ship.targetX=LANES_3D[ship.lane];}}
  }
  if(e.key===" "){e.preventDefault();boostKey=true;}
  if(e.key==="r"&&gameOver){showHomescreen();}
  if(e.key.toLowerCase()==="l"&&laserShots>0&&!gameOver)fireLaser();
  if(getPowerActive("Time Annihilate")&&e.key.toLowerCase()==="f"&&godlyCooldown===0&&!gameOver){
    asteroids.forEach(a=>scene.remove(a.mesh));asteroids=[];
    godlyCooldown=getGodlyCooldownFrames();frozenFrames=180;
    flashScreen("rgba(0,255,255,.25)");showToast("⚡ TIME ANNIHILATE!",1500);
  }
});
document.addEventListener("keyup",e=>{if(e.key===" ")boostKey=false;});
</script>
</body>
</html>
