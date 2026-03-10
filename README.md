
<html>
<head>
<meta charset="UTF-8">
<title>Stellar Drift v2</title>
<style>
body { margin:0; overflow:hidden; background:black; font-family:Arial,sans-serif; color:white; }
canvas { display:block; margin:auto; background:radial-gradient(circle at center,#0a0f2e,#000); }
#ui, #homescreen {
    position:absolute; top:8px; left:50%; transform:translateX(-50%); text-align:center; font-size:16px;
}
#ui { display:none; min-width:320px; }

/* ── RANK BAR ── */
.rankBar { width:280px; height:16px; background:#333; border:2px solid #555; margin:3px auto; position:relative; overflow:hidden; border-radius:3px; }
.rankFill { height:100%; background:linear-gradient(90deg,#00ff00,#ffff00); width:0%; transition:width .3s; }
.rankText { position:absolute; top:50%; left:50%; transform:translate(-50%,-50%); font-size:10px; color:#fff; font-weight:bold; }
.rankLabel { font-size:22px; font-weight:bold; margin-bottom:4px; text-shadow:0 0 10px currentColor; }
.xpLabel { font-size:13px; font-weight:bold; color:#00ffcc; margin:2px auto; text-shadow:0 0 8px #00ffcc; }
.gemsLabel { font-size:15px; color:#00ffd0; font-weight:bold; margin:3px auto; text-shadow:0 0 8px #0ff; }

/* ── TABS ── */
#homeTabs { display:flex; gap:4px; justify-content:center; margin:8px 0 0; flex-wrap:wrap; }
.tabBtn {
    padding:5px 11px; background:#1a1a2e; color:#777; border:1px solid #333;
    border-radius:6px; cursor:pointer; font-size:12px; font-weight:bold; transition:all .15s;
}
.tabBtn.active { background:#0d2137; color:#00ffcc; border-color:#00ffcc; box-shadow:0 0 8px #00ffcc44; }
.tabPanel { display:none; }
.tabPanel.active { display:block; }

/* ── SHIP SELECT ── */
.cosmeticShip {
    display:inline-block; margin:6px; cursor:pointer; border:2px solid rgba(255,255,255,.07);
    border-radius:8px; transition:box-shadow .2s,border .2s; background:#111; vertical-align:top; position:relative;
}
.cosmeticShip.selected { box-shadow:0 0 16px #fff; border:2px solid #fcd34d; }
.cosmeticShip.locked { opacity:.33; cursor:default; }
.booster-badge {
    position:absolute; top:3px; right:5px; background:#0ff; color:#222;
    font-size:9px; font-weight:bold; border-radius:4px; padding:1px 4px;
}

/* ── RUN BOOSTS ── */
#boostsGrid { display:flex; flex-wrap:wrap; justify-content:center; gap:7px; margin-top:6px; max-width:460px; }
.boostCard {
    background:#0d1a2e; border:1px solid #334; border-radius:9px;
    padding:9px 10px; width:120px; text-align:center;
}
.boostCard.equipped { border-color:#00ffcc; box-shadow:0 0 8px #00ffcc44; }
.boostCard .bn { font-size:13px; font-weight:bold; color:#fff; margin-bottom:3px; }
.boostCard .bd { font-size:10px; color:#aaa; margin-bottom:6px; min-height:28px; }
.boostCard .bc { font-size:11px; color:#00ffd0; margin-bottom:5px; }
.boostCard button {
    padding:3px 10px; font-size:11px; border-radius:5px; cursor:pointer;
    font-weight:bold; border:1px solid #00ffcc; background:#222; color:#00ffcc;
}
.boostCard button:disabled { border-color:#444; color:#555; cursor:not-allowed; }
.boostCard button.unequip { border-color:#ff6666; color:#ff6666; }
#activeBoostsList { font-size:12px; color:#0ff; margin:4px 0 2px; min-height:16px; }

/* ── DAILY ── */
#dailyCard {
    background:#0a1a0a; border:2px solid #00ff88; border-radius:12px;
    padding:14px 18px; max-width:320px; margin:10px auto; text-align:center;
}
#dailyCard .dt { font-size:18px; font-weight:bold; color:#00ff88; margin-bottom:6px; }
#dailyCard .dd { font-size:14px; color:#ccc; margin-bottom:6px; }
#dailyCard .dr { font-size:13px; color:#ffd700; }
#dailyCard .ddone { font-size:15px; color:#00ff88; font-weight:bold; margin-top:5px; }
#dailyProgressBar { width:250px; height:12px; background:#1a2a1a; border:1px solid #00ff88; margin:8px auto; border-radius:3px; overflow:hidden; }
#dailyProgressFill { height:100%; background:#00ff88; width:0%; transition:width .3s; }
#dailyProgressText { font-size:11px; color:#aaa; margin-top:2px; }

/* ── LEADERBOARD ── */
#lbPanel { max-width:380px; margin:0 auto; }
#lbTable { width:100%; border-collapse:collapse; font-size:13px; margin-top:6px; }
#lbTable th { color:#00ffcc; border-bottom:1px solid #333; padding:3px 7px; text-align:left; }
#lbTable td { padding:3px 7px; border-bottom:1px solid #1a1a1a; }
#lbTable tr.my-score td { color:#ffd700; font-weight:bold; }
#lbTable tr:nth-child(2) td:first-child { color:#ffd700; }
#lbTable tr:nth-child(3) td:first-child { color:#c0c0c0; }
#lbTable tr:nth-child(4) td:first-child { color:#cd7f32; }
#lbEmpty { color:#666; font-size:13px; margin-top:16px; }
#lbRefreshBtn {
    padding:4px 12px; background:#111; color:#00ffcc; border:1px solid #00ffcc;
    border-radius:6px; cursor:pointer; font-size:12px; margin-top:6px;
}

/* ── NAME / MISC ── */
#nameRow { display:flex; align-items:center; gap:6px; justify-content:center; margin:5px 0; }
#playerNameInput {
    background:#111; border:1px solid #00ffcc; color:#fff; border-radius:5px;
    padding:3px 9px; font-size:13px; width:130px; text-align:center;
}
#boosterBtn {
    padding:5px 14px; font-size:13px; background:#222; color:#00ffd0;
    border:1px solid #00ffd0; border-radius:7px; cursor:pointer; font-weight:bold;
}
#boosterBtn:disabled { color:#444!important; border-color:#444!important; opacity:.5; cursor:not-allowed; }
#boosterResult { display:none; margin:3px auto; font-size:13px; font-weight:bold; color:#fff770; text-shadow:0 0 8px #0ff; }
#cosmeticPowerDesc { margin:6px 0 4px; font-size:13px; color:#0ff; }
#startBtn {
    padding:11px 34px; font-size:20px; border:2px solid #00ffcc; background:#222;
    color:#00ffcc; border-radius:8px; margin-top:12px; cursor:pointer; transition:background .2s;
}
#startBtn:hover { background:#00ffcc; color:black; }

/* ── IN-GAME ── */
#rankUpAnimation {
    display:none; position:absolute; top:50%; left:50%; transform:translate(-50%,-50%);
    font-size:48px; font-weight:bold; color:gold;
    text-shadow:0 0 20px gold,0 0 40px gold;
    animation:rankUpPulse 1s ease-out; z-index:1000; pointer-events:none;
}
@keyframes rankUpPulse {
    0%{transform:translate(-50%,-50%) scale(1);opacity:1;}
    100%{transform:translate(-50%,-150%) scale(1.5);opacity:0;}
}
#reviveFlash { display:none; position:absolute; left:0; top:0; width:100vw; height:100vh; background:rgba(0,255,255,.25); z-index:99999; pointer-events:none; }
#gameOver { display:none; margin-top:5px; font-size:18px; color:#ff4444; }
#godlyCooldown {
    position:absolute; left:50%; top:66px; transform:translateX(-50%);
    font-size:14px; font-weight:bold; color:#00fff7; background:rgba(0,0,0,.65);
    border-radius:6px; padding:2px 10px; display:none; z-index:999; box-shadow:0 0 8px #00fff7;
}
#laserDisplay {
    position:absolute; right:8px; top:8px; font-size:14px; font-weight:bold;
    color:#ff6644; text-shadow:0 0 8px #ff4400; display:none;
}

/* ── SCORE SUBMIT ── */
#scoreSubmit {
    display:none; position:absolute; left:50%; top:50%; transform:translate(-50%,-50%);
    background:rgba(0,0,0,.93); border:2px solid #00ffcc; border-radius:12px;
    padding:18px 24px; text-align:center; z-index:500; min-width:230px;
}
#scoreSubmit h3 { margin:0 0 8px; color:#ffd700; font-size:18px; }
#scoreSubmit .sv { font-size:22px; font-weight:bold; color:#ffd700; margin:4px 0; }
#scoreSubmit .sn { font-size:12px; color:#aaa; margin-bottom:8px; }
#submitBtn {
    padding:7px 20px; background:#00ffcc; color:#000; border:none;
    border-radius:7px; font-size:14px; font-weight:bold; cursor:pointer; display:block; margin:8px auto 4px;
}
#skipBtn {
    padding:4px 14px; background:transparent; color:#666; border:1px solid #444;
    border-radius:6px; font-size:12px; cursor:pointer; display:block; margin:0 auto;
}

/* ── NOTIFICATION TOAST ── */
#toast {
    display:none; position:absolute; left:50%; bottom:40px; transform:translateX(-50%);
    background:rgba(0,0,0,.88); border:1px solid #00ffcc; border-radius:8px;
    padding:8px 18px; font-size:14px; font-weight:bold; color:#fff770;
    text-shadow:0 0 8px #0ff; z-index:9999; white-space:nowrap;
}
</style>
</head>
<body>

<!-- ══════════ HOMESCREEN ══════════ -->
<div id="homescreen">
    <div class="rankLabel" id="homeRank">CADET</div>
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
        <button class="tabBtn" data-tab="dailyTab">📅 Daily</button>

    </div>

    <!-- SHIP TAB -->
    <div id="shipTab" class="tabPanel active">
        <div style="margin:5px 0 3px">
            <button id="boosterBtn">🎲 Random Ship Booster (1 💎)</button>
            <div id="boosterResult"></div>
        </div>
        <div id="cosmeticList"></div>
        <div id="cosmeticPowerDesc"></div>
    </div>

    <!-- RUN BOOSTS TAB -->
    <div id="boostsTab" class="tabPanel">
        <div style="font-size:12px;color:#888;margin:5px 0 2px">Buy boosts before a run — spent on use.</div>
        <div id="activeBoostsList"></div>
        <div id="boostsGrid"></div>
    </div>

    <!-- DAILY TAB -->
    <div id="dailyTab" class="tabPanel">
        <div id="dailyCard">
            <div class="dt">📅 Daily Challenge</div>
            <div class="dd" id="dailyDesc">Loading…</div>
            <div id="dailyProgressBar"><div id="dailyProgressFill"></div></div>
            <div id="dailyProgressText"></div>
            <div class="dr">Reward: 🪙 3 Gems</div>
            <div class="ddone" id="dailyDone" style="display:none">✅ Completed today!</div>
        </div>
    </div>



    <button id="startBtn">▶ Start Game</button>
</div>

<!-- ══════════ OVERLAYS ══════════ -->
<div id="reviveFlash"></div>
<div id="godlyCooldown"></div>
<div id="laserDisplay">🔥 Laser: <span id="laserLeft">3</span></div>
<div id="toast"></div>

<!-- ══════════ IN-GAME UI ══════════ -->
<div id="ui">
    <div class="rankLabel" id="rankDisplay">CADET</div>
    <div class="rankBar"><div class="rankFill" id="gameRankFill"></div><div class="rankText" id="gameRankText">0 / 100 XP</div></div>
    <div class="xpLabel" id="xpDisplay">XP: 0</div>
    <div class="gemsLabel">💎 <span class="gemCount">0</span></div>
    <div style="font-size:14px">Score: <span id="score">0</span> &nbsp;|&nbsp; Best: <span id="highScore">0</span></div>
    <div id="rankUpAnimation">⭐ RANK UP ⭐</div>
    <div id="gameOver">💥 GAME OVER — Press R to Restart</div>
</div>

<!-- SCORE SUBMIT -->
<div id="scoreSubmit">
    <h3>🏆 Submit Score</h3>
    <div class="sv" id="submitVal"></div>
    <div class="sn">Pilot: <strong id="submitName"></strong></div>
    <button id="submitBtn">Post to Leaderboard</button>
    <button id="skipBtn">Skip</button>
</div>

<canvas id="gameCanvas" width="480" height="720"></canvas>

<script>
// ══════════════════════════════════════════════════════════
//  DATA
// ══════════════════════════════════════════════════════════
const XP_RANKS = [
    { name:"Cadet",        xp:0,     color:"#A0A0A0" },
    { name:"Pilot",        xp:100,   color:"#00FF00" },
    { name:"Commander",    xp:300,   color:"#0099FF" },
    { name:"Captain",      xp:600,   color:"#FF6600" },
    { name:"Admiral",      xp:1200,  color:"#FF00FF" },
    { name:"Legend",       xp:2500,  color:"#FFFF00" },
    { name:"Godly",        xp:4000,  color:"#00fff7" },
    { name:"Mythic",       xp:6000,  color:"#ff77ff" },
    { name:"Eternal",      xp:9000,  color:"#fffacd" },
    { name:"Cosmic",       xp:13000, color:"#ff6666" },
    { name:"Transcendent", xp:20000, color:"#ffffff" },
];

const COSMETICS = [
  { id:0, name:"Cadet Viper",        color:"#00ffcc", unlockXp:0,
    svg:`<polygon points="35,15 16,70 54,70"/>`,
    powerName:"Standard", powerDesc:"No special power." },
  { id:1, name:"Pilot Falcon",       color:"#00FF00", unlockXp:100,
    svg:`<polygon points="35,14 12,54 22,76 48,76 58,54"/><rect x="27" y="40" width="16" height="18" rx="5"/>`,
    powerName:"Double Boost", powerDesc:"Spacebar boost is twice as fast." },
  { id:2, name:"Commander Nova",     color:"#0099FF", unlockXp:300,
    svg:`<ellipse cx="35" cy="38" rx="16" ry="29"/><polygon points="35,18 19,70 51,70"/>`,
    powerName:"Shield on Dodge", powerDesc:"Gain a shield every 5 asteroids dodged." },
  { id:3, name:"Captain Phoenix",    color:"#FF6600", unlockXp:600,
    svg:`<polygon points="35,14 16,54 35,85 54,54" style="stroke:#fff;stroke-width:1.5"/><polygon points="30,36 40,36 35,55"/>`,
    powerName:"Milestone XP+", powerDesc:"+50% XP for score milestones." },
  { id:4, name:"Admiral Eclipse",    color:"#FF00FF", unlockXp:1200,
    svg:`<ellipse cx="35" cy="42" rx="18" ry="24"/><circle cx="35" cy="32" r="13"/>`,
    powerName:"Slow Asteroids", powerDesc:"Asteroid speed −20%." },
  { id:5, name:"Legend Starcruiser", color:"#FFFF00", unlockXp:2500,
    svg:`<rect x="19" y="24" rx="9" width="32" height="42"/><polygon points="35,6 25,24 45,24"/><ellipse cx="35" cy="70" rx="11" ry="7"/>`,
    powerName:"Revive", powerDesc:"Survive first crash once per game." },
  { id:6, name:"Godly Dragon",       color:"#00fff7", unlockXp:4000,
    svg:`<ellipse cx="35" cy="45" rx="22" ry="30"/><polygon points="35,8 16,42 54,42"/><ellipse cx="35" cy="70" rx="12" ry="8"/>
         <polygon points="16,47 13,62 27,66 21,55" style="fill:#fff;"/><polygon points="54,47 57,62 43,66 49,55" style="fill:#fff;"/>`,
    powerName:"Time Annihilate", powerDesc:"Press F every 30s: destroy all asteroids + freeze 3s." },
];

const RUN_BOOSTS = [
    { id:"hyperdrive", name:"⚡ Hyperdrive",  desc:"Move 2× faster between lanes",         cost:1 },
    { id:"iron_shell", name:"🛡️ Iron Shell",  desc:"Start the run with a shield",          cost:1 },
    { id:"xp_surge",   name:"✨ XP Surge",    desc:"Earn 3× XP this entire run",           cost:2 },
    { id:"null_field", name:"❄️ Null Field",  desc:"Asteroids 35% slower this run",        cost:2 },
    { id:"laser_shot", name:"🔥 Laser Shot",  desc:"Press L to blast your lane (3 shots)", cost:2 },
];

// ══════════════════════════════════════════════════════════
//  GAME STATE
// ══════════════════════════════════════════════════════════
const XP_PER_ASTEROID=10, XP_MILESTONE_STEP=100, XP_PER_MILESTONE=40, XP_PASSIVE_SEC=1;
const canvas = document.getElementById("gameCanvas");
const ctx    = canvas.getContext("2d");

let score, highScore, gameOver, baseSpeed, boostKey, difficultyTimer;
let xp, xpMilestoneNext, xpPassiveTimer, shieldActive, dodgedCount;
let legendRevives=0, frozenFrames=0, godlyCooldown=0;
let boostedShips=[], gems=0;
let asteroids, stars, particles;
let chosenCosmetic=0;
let startShieldActive=false, startShieldFrames=0;
let activeRunBoosts=[];
let laserShots=0, laserFrames=0, laserLane=0;
let playerName="Pilot";
let dailySessionScore=0;

const lanes = [canvas.width*.25-20, canvas.width*.5-20, canvas.width*.75-20];
const ship = { width:40, height:60, x:lanes[1], targetX:lanes[1], y:canvas.height-120, lane:1, moveSpeed:.11, tilt:0 };
let animationFrameId=null;

// ══════════════════════════════════════════════════════════
//  HELPERS
// ══════════════════════════════════════════════════════════
function hasRunBoost(id){ return activeRunBoosts.includes(id); }
function getPowerActive(name){ return COSMETICS[chosenCosmetic].powerName===name; }
function isBoosted(id){ return boostedShips.includes(id); }

function getBoostMultiplier(){
    if(chosenCosmetic===1&&isBoosted(1)) return 4;
    if(getPowerActive("Double Boost")) return 2;
    return 1;
}
function getAsteroidSpeedMult(){
    let m=1;
    if(chosenCosmetic===4&&isBoosted(4)) m*=.6;
    else if(getPowerActive("Slow Asteroids")) m*=.8;
    if(hasRunBoost("null_field")) m*=.65;
    return m;
}
function getMilestoneXpBonus(){
    if(chosenCosmetic===3&&isBoosted(3)) return 2;
    if(getPowerActive("Milestone XP+")) return 1.5;
    return 1;
}
function getShieldEveryDodged(){
    if(chosenCosmetic===2&&isBoosted(2)) return 2;
    if(getPowerActive("Shield on Dodge")) return 5;
    return null;
}
function getLegendRevives(){
    if(chosenCosmetic===5&&isBoosted(5)) return 2;
    if(getPowerActive("Revive")) return 1;
    return 0;
}
function getGodlyCooldownFrames(){ return (chosenCosmetic===6&&isBoosted(6))?900:1800; }

function getBoostedDesc(id){
    return ["Start invulnerable (shield) for 10s","Boost is 4×","Shield every 2 dodges",
            "+100% XP for milestones","Asteroid speed −40%","Two revives per game",
            "Half cooldown: F every 15s"][id]||"";
}

// ══════════════════════════════════════════════════════════
//  PERSISTENCE
// ══════════════════════════════════════════════════════════
function saveProgress(){
    localStorage.setItem("sd2_xp", xp);
    localStorage.setItem("sd2_cosmetic", chosenCosmetic);
    localStorage.setItem("sd2_boosted", JSON.stringify(boostedShips));
    localStorage.setItem("sd2_gems", gems);
    localStorage.setItem("sd2_name", playerName);
}
function loadProgress(){
    xp           = parseInt(localStorage.getItem("sd2_xp")||"0")||0;
    chosenCosmetic=parseInt(localStorage.getItem("sd2_cosmetic")||"0")||0;
    boostedShips = JSON.parse(localStorage.getItem("sd2_boosted")||"[]");
    gems         = parseInt(localStorage.getItem("sd2_gems")||"0")||0;
    playerName   = localStorage.getItem("sd2_name")||"Pilot";
    xpMilestoneNext = XP_MILESTONE_STEP;
    document.getElementById("playerNameInput").value = playerName;
}

// ══════════════════════════════════════════════════════════
//  GEM / XP UI
// ══════════════════════════════════════════════════════════
function updateGemsUi(){ document.querySelectorAll(".gemCount").forEach(el=>el.innerText=gems); }

function getXpRank(v){
    for(let i=XP_RANKS.length-1;i>=0;i--) if(v>=XP_RANKS[i].xp) return XP_RANKS[i];
    return XP_RANKS[0];
}
function nextXpRank(){
    const idx=XP_RANKS.findIndex(r=>r.xp===getXpRank(xp).xp);
    return XP_RANKS[idx+1]||null;
}
function getXpProgress(){
    const next=nextXpRank(); if(!next) return 100;
    const cur=getXpRank(xp); return Math.min(100,Math.max(0,(xp-cur.xp)/(next.xp-cur.xp)*100));
}
function getXpProgressText(){
    const next=nextXpRank(); return next ? `${xp} / ${next.xp} XP` : `${xp} / MAX`;
}

function updateAllRankUi(){
    const rank=getXpRank(xp); const pct=getXpProgress(); const txt=getXpProgressText();
    updateGemsUi();
    // homescreen
    const hr=document.getElementById("homeRank");
    hr.innerText=rank.name.toUpperCase(); hr.style.color=rank.color;
    document.getElementById("homeXpLabel").innerText="XP: "+xp;
    document.getElementById("homeRankFill").style.width=pct+"%";
    document.getElementById("homeRankFill").style.background=`linear-gradient(90deg,${rank.color},#ffff00)`;
    document.getElementById("homeRankText").innerText=txt;
    // in-game
    const rd=document.getElementById("rankDisplay");
    rd.innerText=rank.name.toUpperCase(); rd.style.color=rank.color;
    document.getElementById("xpDisplay").innerText="XP: "+xp;
    document.getElementById("gameRankFill").style.width=pct+"%";
    document.getElementById("gameRankFill").style.background=`linear-gradient(90deg,${rank.color},#ffff00)`;
    document.getElementById("gameRankText").innerText=txt;
    // booster btn
    document.getElementById("boosterBtn").disabled=(gems<1);
}

function addXp(amount){
    const mult = hasRunBoost("xp_surge")?3:1;
    const oldRank=getXpRank(xp);
    xp+=amount*mult;
    const newRank=getXpRank(xp);
    if(newRank.name!==oldRank.name){
        gems++; updateGemsUi(); saveProgress();
        showToast("+1 💎 for reaching "+newRank.name+"!");
        triggerRankUpAnimation();
    }
    saveProgress(); updateAllRankUi();
}

function triggerRankUpAnimation(){
    const el=document.getElementById("rankUpAnimation");
    el.style.display="block"; el.offsetHeight;
    setTimeout(()=>el.style.display="none",1100);
}

function showToast(msg,duration=2200){
    const t=document.getElementById("toast");
    t.innerText=msg; t.style.display="block";
    clearTimeout(t._tid);
    t._tid=setTimeout(()=>t.style.display="none",duration);
}

// ══════════════════════════════════════════════════════════
//  SHIP BOOSTER (random cosmetic booster)
// ══════════════════════════════════════════════════════════
document.getElementById("boosterBtn").onclick=function(){
    if(gems<1) return;
    const avail=COSMETICS.map(c=>c.id).filter(id=>!isBoosted(id));
    if(!avail.length){ showToast("All ships already BOOSTED!"); return; }
    const r=avail[Math.floor(Math.random()*avail.length)];
    boostedShips.push(r); gems--; saveProgress(); updateAllRankUi();
    drawHomeCosmetics(); drawPowerDesc();
    showToast(`BOOSTED: ${COSMETICS[r].name} — ${getBoostedDesc(r)}`,4000);
    document.getElementById("boosterBtn").disabled=(gems<1);
};

// ══════════════════════════════════════════════════════════
//  RUN BOOSTS
// ══════════════════════════════════════════════════════════
function toggleRunBoost(id){
    const def=RUN_BOOSTS.find(b=>b.id===id); if(!def) return;
    if(hasRunBoost(id)){
        activeRunBoosts=activeRunBoosts.filter(b=>b!==id);
        gems+=def.cost; saveProgress(); updateAllRankUi();
    } else {
        if(gems<def.cost){ showToast("Not enough gems!"); return; }
        gems-=def.cost; activeRunBoosts.push(id); saveProgress(); updateAllRankUi();
    }
    renderBoostsTab();
}

function renderBoostsTab(){
    const grid=document.getElementById("boostsGrid"); grid.innerHTML="";
    RUN_BOOSTS.forEach(b=>{
        const eq=hasRunBoost(b.id), afford=gems>=b.cost;
        const d=document.createElement("div");
        d.className="boostCard"+(eq?" equipped":"");
        d.innerHTML=`<div class="bn">${b.name}</div>
                     <div class="bd">${b.desc}</div>
                     <div class="bc">${eq?"✅ Equipped":"Cost: "+b.cost+" 💎"}</div>
                     <button class="${eq?"unequip":""}" ${!eq&&!afford?"disabled":""}
                         onclick="toggleRunBoost('${b.id}')">
                         ${eq?"Unequip":"Equip"}</button>`;
        grid.appendChild(d);
    });
    const el=document.getElementById("activeBoostsList");
    if(!activeRunBoosts.length){ el.innerText="No run boosts equipped."; return; }
    el.innerText="Active: "+activeRunBoosts.map(id=>RUN_BOOSTS.find(b=>b.id===id).name).join("  |  ");
}

// ══════════════════════════════════════════════════════════
//  DAILY CHALLENGE
// ══════════════════════════════════════════════════════════
function getDailyChallenge(){
    const now=new Date();
    const seed=now.getFullYear()*10000+(now.getMonth()+1)*100+now.getDate();
    const targets=[150,200,250,300,400];
    return {
        shipId: seed%COSMETICS.length,
        shipName: COSMETICS[seed%COSMETICS.length].name,
        targetScore: targets[seed%targets.length],
        seed,
        dateStr: now.toDateString()
    };
}
function isDailyDone(){ return localStorage.getItem("sd2_daily_"+getDailyChallenge().seed)==="done"; }
function completeDailyChallenge(){
    if(isDailyDone()) return;
    const ch=getDailyChallenge();
    localStorage.setItem("sd2_daily_"+ch.seed,"done");
    gems+=3; saveProgress(); updateAllRankUi();
    renderDailyTab();
    showToast("🎉 Daily Challenge Complete! +3 💎",3000);
}

function renderDailyTab(){
    const ch=getDailyChallenge(), done=isDailyDone();
    document.getElementById("dailyDesc").innerText=
        `Score ${ch.targetScore}+ points using the ${ch.shipName}`;
    document.getElementById("dailyDone").style.display=done?"block":"none";
    const pct=done?100:Math.min(100,dailySessionScore/ch.targetScore*100);
    document.getElementById("dailyProgressFill").style.width=pct+"%";
    document.getElementById("dailyProgressText").innerText=
        done?"":`Progress this run: ${Math.min(dailySessionScore,ch.targetScore)} / ${ch.targetScore}`;
}

// ══════════════════════════════════════════════════════════
//  LEADERBOARD
// ══════════════════════════════════════════════════════════
function getLeaderboard(){
    try{ return JSON.parse(localStorage.getItem("sd2_lb")||"[]"); } catch(e){ return []; }
}
function submitScore(name,scoreVal){
    const lb=getLeaderboard();
    lb.push({ name:(name||"Anonymous").trim().substring(0,16), score:scoreVal, date:new Date().toLocaleDateString() });
    lb.sort((a,b)=>b.score-a.score); lb.splice(20);
    localStorage.setItem("sd2_lb",JSON.stringify(lb));
    renderLeaderboard();
}
function renderLeaderboard(){
    const lb=getLeaderboard(), content=document.getElementById("lbContent");
    if(!lb.length){ content.innerHTML=`<div id="lbEmpty">No scores yet — play a game!</div>`; return; }
    const myName=(playerName||"").trim();
    content.innerHTML=`<table id="lbTable">
        <tr><th>#</th><th>Pilot</th><th>Score</th><th>Date</th></tr>
        ${lb.map((e,i)=>`<tr class="${e.name===myName?"my-score":""}">
            <td>${i+1}</td><td>${e.name}</td><td>${e.score}</td>
            <td style="color:#555;font-size:11px">${e.date}</td></tr>`).join("")}
    </table>`;
}

// ══════════════════════════════════════════════════════════
//  SCORE SUBMIT OVERLAY
// ══════════════════════════════════════════════════════════
function showScoreSubmit(){
    if(score<=0){ showHomescreen(); return; }
    document.getElementById("submitVal").innerText="Score: "+score;
    document.getElementById("submitName").innerText=playerName||"Anonymous";
    document.getElementById("scoreSubmit").style.display="block";
}
document.getElementById("submitBtn").onclick=function(){
    submitScore(playerName,score);
    document.getElementById("scoreSubmit").style.display="none";
    showHomescreen();
};
document.getElementById("skipBtn").onclick=function(){
    document.getElementById("scoreSubmit").style.display="none";
    showHomescreen();
};

// ══════════════════════════════════════════════════════════
//  TABS
// ══════════════════════════════════════════════════════════
document.querySelectorAll(".tabBtn").forEach(btn=>{
    btn.addEventListener("click",()=>{
        document.querySelectorAll(".tabBtn").forEach(b=>b.classList.remove("active"));
        document.querySelectorAll(".tabPanel").forEach(p=>p.classList.remove("active"));
        btn.classList.add("active");
        document.getElementById(btn.dataset.tab).classList.add("active");
        if(btn.dataset.tab==="lbTab")    renderLeaderboard();
        if(btn.dataset.tab==="dailyTab") renderDailyTab();
        if(btn.dataset.tab==="boostsTab"){ renderBoostsTab(); }
    });
});
document.getElementById("playerNameInput").addEventListener("input",function(){
    playerName=this.value; saveProgress();
});

// ══════════════════════════════════════════════════════════
//  HOMESCREEN SHOW/HIDE
// ══════════════════════════════════════════════════════════
function showHomescreen(){
    if(animationFrameId!==null){ cancelAnimationFrame(animationFrameId); animationFrameId=null; }
    document.getElementById("homescreen").style.display="block";
    document.getElementById("ui").style.display="none";
    document.getElementById("scoreSubmit").style.display="none";
    document.getElementById("laserDisplay").style.display="none";
    document.getElementById("godlyCooldown").style.display="none";
    updateAllRankUi(); drawHomeCosmetics(); drawPowerDesc(); renderDailyTab();
    ctx.clearRect(0,0,canvas.width,canvas.height);
}
function hideHomescreen(){
    document.getElementById("homescreen").style.display="none";
    document.getElementById("ui").style.display="block";
}

function drawHomeCosmetics(){
    const list=document.getElementById("cosmeticList"); list.innerHTML="";
    COSMETICS.forEach(sc=>{
        const unlocked=xp>=sc.unlockXp;
        const div=document.createElement("div");
        div.className="cosmeticShip"+(chosenCosmetic===sc.id?" selected":"")+(unlocked?"":" locked");
        div.title=sc.name+(unlocked?"":" (Locked)");
        div.style.cssText="width:78px;height:96px;";
        div.innerHTML=`<svg width="68" height="78"><g fill="${sc.color}" stroke="#222" stroke-width="2">${sc.svg}</g></svg>
            <div style="font-size:11px"><b>${sc.name}</b></div>
            <div style="font-size:10px;color:#ccc">${unlocked?sc.powerName:"🔒"}</div>`;
        if(isBoosted(sc.id)){ const b=document.createElement("span"); b.className="booster-badge"; b.innerText="BOOST"; div.appendChild(b); }
        if(unlocked) div.onclick=()=>{ chosenCosmetic=sc.id; saveProgress(); drawHomeCosmetics(); drawPowerDesc(); };
        list.appendChild(div);
    });
}
function drawPowerDesc(){
    document.getElementById("cosmeticPowerDesc").innerText=
        isBoosted(chosenCosmetic)?"BOOSTED: "+getBoostedDesc(chosenCosmetic):COSMETICS[chosenCosmetic].powerDesc;
}

// ══════════════════════════════════════════════════════════
//  GAME RESET
// ══════════════════════════════════════════════════════════
function resetGame(){
    if(animationFrameId!==null){ cancelAnimationFrame(animationFrameId); animationFrameId=null; }
    baseSpeed=4; score=0; boostKey=false; difficultyTimer=0;
    asteroids=[]; particles=[]; stars=[];
    shieldActive=false; xpPassiveTimer=0; gameOver=false;
    dodgedCount=0; legendRevives=0; frozenFrames=0; godlyCooldown=0;
    dailySessionScore=0;
    laserShots=0; laserFrames=0; laserLane=0;

    loadProgress();

    // Apply run boosts (consumed this run)
    const boostedCadet=(chosenCosmetic===0&&isBoosted(0));
    startShieldActive=boostedCadet; startShieldFrames=boostedCadet?600:0;
    if(hasRunBoost("iron_shell")) shieldActive=true;
    ship.moveSpeed=hasRunBoost("hyperdrive")?.22:.11;
    laserShots=hasRunBoost("laser_shot")?3:0;

    // Consume boosts
    activeRunBoosts=[];
    saveProgress();

    ship.lane=1; ship.x=lanes[1]; ship.targetX=lanes[1]; ship.tilt=0;
    highScore=parseInt(localStorage.getItem("sd2_hs")||"0")||0;
    document.getElementById("score").innerText=0;
    document.getElementById("highScore").innerText=highScore;
    document.getElementById("gameOver").style.display="none";
    document.getElementById("scoreSubmit").style.display="none";
    document.getElementById("reviveFlash").style.display="none";
    document.getElementById("godlyCooldown").style.display="none";

    const ld=document.getElementById("laserDisplay");
    if(laserShots>0){ ld.style.display="block"; document.getElementById("laserLeft").innerText=laserShots; }
    else { ld.style.display="none"; }

    updateAllRankUi(); renderBoostsTab();
    for(let i=0;i<100;i++) stars.push({x:Math.random()*canvas.width,y:Math.random()*canvas.height,size:Math.random()*2});
}

// ══════════════════════════════════════════════════════════
//  LASER
// ══════════════════════════════════════════════════════════
function fireLaser(){
    if(laserShots<=0||gameOver) return;
    laserShots--; laserLane=ship.lane; laserFrames=12;
    const lx=lanes[ship.lane];
    asteroids=asteroids.filter(a=>a.x!==lx);
    document.getElementById("laserLeft").innerText=laserShots;
    if(laserShots<=0) document.getElementById("laserDisplay").style.display="none";
    showToast("🔥 LASER FIRED"+(laserShots>0?" ("+laserShots+" left)":"— all shots used!"),1200);
}
function drawLaser(){
    if(laserFrames>0){
        ctx.save();
        ctx.globalAlpha=laserFrames/12;
        ctx.strokeStyle="#ff6600"; ctx.lineWidth=5;
        ctx.shadowColor="#ff2200"; ctx.shadowBlur=24;
        const lx=lanes[laserLane]+20;
        ctx.beginPath(); ctx.moveTo(lx,ship.y); ctx.lineTo(lx,0); ctx.stroke();
        ctx.restore(); laserFrames--;
    }
}

// ══════════════════════════════════════════════════════════
//  ASTEROIDS
// ══════════════════════════════════════════════════════════
function updateAsteroids(speed){
    for(let index=asteroids.length-1;index>=0;index--){
        const a=asteroids[index];
        const aspd=frozenFrames>0?0:speed*getAsteroidSpeedMult();
        a.y+=aspd;
        if(!gameOver&&
            ship.x<a.x+a.width&&ship.x+ship.width>a.x&&
            ship.y<a.y+a.height&&ship.y+ship.height>a.y){
            if(chosenCosmetic===0&&isBoosted(0)&&startShieldActive) continue;
            if(getPowerActive("Revive")){
                if(legendRevives<getLegendRevives()){
                    legendRevives++;
                    asteroids=asteroids.filter(o=>o.x!==lanes[ship.lane]||o.y>ship.y+ship.height||o.y+o.height<ship.y);
                    flashScreen("rgba(255,255,0,.18)");
                    spawnParticles(ship.x+ship.width/2,ship.y+ship.height/2,30);
                    shieldActive=false; continue;
                }
            }
            if(getPowerActive("Shield on Dodge")&&shieldActive){
                shieldActive=false; asteroids.splice(index,1); saveProgress(); continue;
            }
            gameOver=true;
            spawnParticles(ship.x+ship.width/2,ship.y+ship.height/2,30);
            document.getElementById("gameOver").style.display="block";
            saveProgress();
            // Check daily
            const ch=getDailyChallenge();
            if(chosenCosmetic===ch.shipId&&score>=ch.targetScore) completeDailyChallenge();
            setTimeout(showScoreSubmit,950);
        }
        if(a.y>canvas.height){
            asteroids.splice(index,1);
            if(!gameOver){
                score+=10; dailySessionScore=score;
                document.getElementById("score").innerText=score;
                addXp(XP_PER_ASTEROID);
                dodgedCount++;
                const every=getShieldEveryDodged();
                if(every&&dodgedCount%every===0) shieldActive=true;
                while(score>=xpMilestoneNext){
                    addXp(Math.floor(XP_PER_MILESTONE*getMilestoneXpBonus()));
                    xpMilestoneNext+=XP_MILESTONE_STEP;
                }
                saveProgress();
                if(score>highScore){
                    highScore=score;
                    localStorage.setItem("sd2_hs",highScore);
                    document.getElementById("highScore").innerText=highScore;
                }
            }
        }
    }
    if(frozenFrames>0) frozenFrames--;
    if(startShieldActive&&startShieldFrames>0){ startShieldFrames--; if(!startShieldFrames) startShieldActive=false; }
}

function spawnAsteroid(){
    const count=Math.random()<.8?1:2;
    const shuffled=[...lanes].sort(()=>Math.random()-.5);
    let spawned=0;
    for(let i=0;i<shuffled.length&&spawned<count;i++){
        const lx=shuffled[i];
        if(!asteroids.some(a=>a.x===lx&&a.y<150)){ asteroids.push({x:lx,y:-60,width:40,height:40}); spawned++; }
    }
}
function preventImpossibleRows(){
    const tops=lanes.map(lx=>asteroids.filter(a=>a.x===lx&&a.y<150));
    if(tops.every(arr=>arr.length>0)){
        const li=Math.floor(Math.random()*3), ar=tops[li][0];
        if(ar){ const idx=asteroids.indexOf(ar); if(idx>-1) asteroids.splice(idx,1); }
    }
}

// ══════════════════════════════════════════════════════════
//  DRAW HELPERS
// ══════════════════════════════════════════════════════════
function flashScreen(color){
    const f=document.getElementById("reviveFlash");
    f.style.background=color; f.style.display="block";
    setTimeout(()=>f.style.display="none",300);
}
function spawnParticles(x,y,n){
    for(let i=0;i<n;i++) particles.push({x,y,vx:(Math.random()-.5)*6,vy:(Math.random()-.5)*6,life:30});
}

function drawStars(speed){
    ctx.fillStyle="white";
    stars.forEach(s=>{ ctx.fillRect(s.x,s.y,s.size,s.size); s.y+=speed*.5; if(s.y>canvas.height) s.y=0; });
}
function drawAsteroids(){
    // Draw rocky-looking asteroids
    asteroids.forEach(a=>{
        const cx=a.x+20, cy=a.y+20;
        ctx.fillStyle="#777"; ctx.beginPath(); ctx.arc(cx,cy,20,0,Math.PI*2); ctx.fill();
        ctx.fillStyle="#555"; ctx.beginPath(); ctx.arc(cx-5,cy-5,6,0,Math.PI*2); ctx.fill();
        ctx.fillStyle="#888"; ctx.beginPath(); ctx.arc(cx+6,cy+7,4,0,Math.PI*2); ctx.fill();
    });
}
function drawShip(){
    const sc=COSMETICS[chosenCosmetic];
    ctx.save(); ctx.translate(ship.x+ship.width/2,ship.y+ship.height/2); ctx.rotate(ship.tilt);
    ctx.beginPath();
    switch(chosenCosmetic){
        case 0: ctx.moveTo(0,-25);ctx.lineTo(-19,29);ctx.lineTo(19,29);ctx.closePath();ctx.fillStyle=sc.color;ctx.fill();break;
        case 1: ctx.moveTo(0,-24);ctx.lineTo(-23,18);ctx.lineTo(-13,38);ctx.lineTo(13,38);ctx.lineTo(23,18);ctx.closePath();ctx.fillStyle=sc.color;ctx.fill();ctx.beginPath();ctx.rect(-8,0,16,18);ctx.fill();break;
        case 2: ctx.ellipse(0,0,16,29,0,0,2*Math.PI);ctx.fillStyle=sc.color;ctx.fill();ctx.beginPath();ctx.moveTo(0,-20);ctx.lineTo(-16,32);ctx.lineTo(16,32);ctx.closePath();ctx.fill();break;
        case 3: ctx.moveTo(0,-27);ctx.lineTo(-18,17);ctx.lineTo(0,40);ctx.lineTo(18,17);ctx.closePath();ctx.fillStyle=sc.color;ctx.fill();ctx.beginPath();ctx.moveTo(-5,12);ctx.lineTo(5,12);ctx.lineTo(0,29);ctx.closePath();ctx.fill();break;
        case 4: ctx.ellipse(0,6,18,24,0,0,2*Math.PI);ctx.fillStyle=sc.color;ctx.fill();ctx.beginPath();ctx.arc(0,-4,13,0,2*Math.PI);ctx.fill();break;
        case 5: ctx.beginPath();ctx.rect(-16,-16,32,42);ctx.fillStyle=sc.color;ctx.fill();ctx.beginPath();ctx.moveTo(0,-34);ctx.lineTo(-10,-16);ctx.lineTo(10,-16);ctx.closePath();ctx.fill();ctx.beginPath();ctx.ellipse(0,26,11,7,0,0,2*Math.PI);ctx.fill();break;
        case 6: ctx.ellipse(0,4,22,30,0,0,2*Math.PI);ctx.fillStyle=sc.color;ctx.fill();ctx.beginPath();ctx.moveTo(0,-34);ctx.lineTo(-19,10);ctx.lineTo(19,10);ctx.closePath();ctx.fill();ctx.beginPath();ctx.ellipse(0,25,12,8,0,0,2*Math.PI);ctx.fill();ctx.beginPath();ctx.moveTo(-19,7);ctx.lineTo(-21,23);ctx.lineTo(-8,27);ctx.lineTo(-14,13);ctx.closePath();ctx.fillStyle="#fff";ctx.fill();ctx.beginPath();ctx.moveTo(19,7);ctx.lineTo(21,23);ctx.lineTo(8,27);ctx.lineTo(14,13);ctx.closePath();ctx.fillStyle="#fff";ctx.fill();break;
    }
    // Shield glow
    if((chosenCosmetic===0&&isBoosted(0)&&startShieldActive)||(getPowerActive("Shield on Dodge")&&shieldActive)){
        ctx.lineWidth=2;ctx.strokeStyle="#0ff";ctx.shadowColor="#0ff";ctx.shadowBlur=12;
        ctx.beginPath();ctx.arc(0,0,26,0,2*Math.PI);ctx.stroke();
    }
    // Iron Shell indicator
    if(hasRunBoost("iron_shell")&&shieldActive&&!getPowerActive("Shield on Dodge")&&!(chosenCosmetic===0&&isBoosted(0)&&startShieldActive)){
        ctx.lineWidth=2;ctx.strokeStyle="#ffaa00";ctx.shadowColor="#ffaa00";ctx.shadowBlur=10;
        ctx.beginPath();ctx.arc(0,0,26,0,2*Math.PI);ctx.stroke();
    }
    ctx.restore();
}
function updateShip(){
    ship.x+=(ship.targetX-ship.x)*ship.moveSpeed;
    ship.tilt=(ship.targetX-ship.x)*.005;
}
function updateParticles(){
    ctx.fillStyle="orange";
    particles.forEach((p,i)=>{ p.x+=p.vx;p.y+=p.vy;p.life--;ctx.fillRect(p.x,p.y,4,4);if(p.life<=0)particles.splice(i,1); });
}
function updatePassiveXp(){
    if(!gameOver){ xpPassiveTimer++; if(xpPassiveTimer>=60){addXp(XP_PASSIVE_SEC);xpPassiveTimer=0;} }
}
function updateGodlyCooldown(){
    if(getPowerActive("Time Annihilate")&&!gameOver){
        const el=document.getElementById("godlyCooldown"); el.style.display="block";
        el.innerText=godlyCooldown>0?`⏳ Godly: ${Math.ceil(godlyCooldown/60)}s`:`Press F — Time Annihilate!`;
    } else { document.getElementById("godlyCooldown").style.display="none"; }
}

// ══════════════════════════════════════════════════════════
//  MAIN LOOP
// ══════════════════════════════════════════════════════════
function update(){
    ctx.clearRect(0,0,canvas.width,canvas.height);
    const spd=frozenFrames>0?0:baseSpeed+(baseSpeed*.5*getBoostMultiplier())+(boostKey?4*getBoostMultiplier():0);
    drawStars(spd);
    if(!gameOver){
        difficultyTimer++;
        if(frozenFrames===0&&difficultyTimer%300===0) baseSpeed+=.5;
        updateShip(); updateAsteroids(spd); preventImpossibleRows();
        if(frozenFrames===0&&Math.random()<.03) spawnAsteroid();
        if(godlyCooldown>0) godlyCooldown--;
    }
    drawLaser(); drawShip(); drawAsteroids(); updateParticles();
    updatePassiveXp(); updateGodlyCooldown();
    animationFrameId=requestAnimationFrame(update);
}

// ══════════════════════════════════════════════════════════
//  START
// ══════════════════════════════════════════════════════════
document.getElementById("startBtn").onclick=()=>{
    hideHomescreen(); resetGame(); animationFrameId=requestAnimationFrame(update);
};

loadProgress();
showHomescreen();

// ══════════════════════════════════════════════════════════
//  INPUT
// ══════════════════════════════════════════════════════════
document.addEventListener("keydown",e=>{
    if(document.getElementById("ui").style.display==="none") return;
    if(e.key==="ArrowLeft"||e.key==="a"){ if(ship.lane>0){ship.lane--;ship.targetX=lanes[ship.lane];} }
    if(e.key==="ArrowRight"||e.key==="d"){ if(ship.lane<2){ship.lane++;ship.targetX=lanes[ship.lane];} }
    if(e.key===" ") boostKey=true;
    if(e.key==="r"&&gameOver){ document.getElementById("scoreSubmit").style.display="none"; resetGame(); animationFrameId=requestAnimationFrame(update); }
    if(e.key.toLowerCase()==="l"&&laserShots>0&&!gameOver) fireLaser();
    if(getPowerActive("Time Annihilate")&&e.key.toLowerCase()==="f"&&godlyCooldown===0&&!gameOver){
        asteroids=[]; godlyCooldown=getGodlyCooldownFrames(); frozenFrames=180;
        flashScreen("rgba(0,255,255,.25)");
    }
});
document.addEventListener("keyup",e=>{ if(e.key===" ") boostKey=false; });
</script>
</body>
</html>
