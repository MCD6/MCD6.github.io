
<html>
<head>
<meta charset="UTF-8">
<title>Stellar Drift</title>
<style>
    body {
        margin: 0;
        overflow: hidden;
        background: black;
        font-family: Arial, sans-serif;
        color: white;
    }
    canvas {
        display: block;
        margin: auto;
        background: radial-gradient(circle at center, #0a0f2e, #000);
    }
    #ui, #homescreen {
        position: absolute;
        top: 10px;
        left: 50%;
        transform: translateX(-50%);
        text-align: center;
        font-size: 18px;
    }
    #ui { display: none; }
    #rankDisplay, #homeRank {
        font-size: 24px;
        font-weight: bold;
        margin-bottom: 10px;
        text-shadow: 0 0 10px currentColor;
    }
    #gems {
        font-size: 18px;
        color: #00ffd0;
        font-weight: bold;
        margin: 5px auto 5px auto;
        text-shadow: 0 0 8px #0ff;
    }
    .cosmeticShip {
        display: inline-block;
        margin: 8px;
        cursor: pointer;
        border: 2px solid rgba(255,255,255,0.07);
        border-radius: 8px;
        transition: box-shadow 0.2s, border 0.2s;
        background: #111;
        vertical-align: top;
        position: relative;
    }
    .cosmeticShip.selected {
        box-shadow: 0 0 16px #fff;
        border: 2px solid #fcd34d;
    }
    .cosmeticShip.locked {
        opacity: 0.33;
        cursor: default;
    }
    .booster {
        position: absolute;
        top: 4px;
        right: 8px;
        background: #0ff;
        color: #222;
        font-size: 11px;
        font-weight: bold;
        border-radius: 6px;
        padding: 1px 5px;
        box-shadow: 0 0 2px #333;
        z-index: 2;
        text-shadow: 1px 1px 6px #fff;
    }
    #boosterBtn {
        margin-bottom: 8px;
        padding: 7px 24px;
        font-size: 17px;
        background: #222;
        color: #00ffd0;
        border: 2px solid #00ffd0;
        border-radius: 9px;
        cursor: pointer;
        font-weight: bold;
        transition: background .2s;
    }
    #boosterBtn:disabled {
        color: #444 !important;
        border: 2px solid #444 !important;
        background: #222 !important;
        opacity: .5;
        cursor: not-allowed;
    }
    #boosterResult {
        display: none;
        margin: 6px auto 10px auto;
        font-size: 16px;
        font-weight: bold;
        color: #fff770;
        text-shadow: 0 0 10px #0ff,0 0 3px #fff;
    }
    #startBtn {
        padding: 12px 36px;
        font-size: 22px;
        border: 2px solid #00ffcc;
        background: #222;
        color: #00ffcc;
        border-radius: 8px;
        margin-top: 18px;
        cursor: pointer;
        transition: background .2s;
    }
    #startBtn:hover {
        background: #00ffcc;
        color: black;
    }
    #xpDisplay { font-size: 18px; font-weight: bold; margin: 4px auto 10px auto; color: #00ffcc; text-shadow:0 0 8px #00ffcc; }
    #rankProgressBar { width: 300px; height: 20px; background: #333; border: 2px solid white; margin: 5px auto; position: relative; overflow: hidden;}
    #rankProgressFill { height: 100%; background: linear-gradient(90deg, #00ff00, #ffff00); width: 0%; transition: width 0.3s; }
    #rankProgressText { position: absolute; top: 50%; left: 50%; transform: translate(-50%, -50%); font-size: 12px; color: white; font-weight: bold;}
    #cosmeticPowerDesc { margin:12px 0 10px 0; font-size:16px; color:#0ff; text-shadow:0 0 10px #000;}
    #rankUpAnimation { display: none; position: absolute; top: 50%; left: 50%; transform: translate(-50%, -50%); font-size: 48px; font-weight: bold; color: gold; text-shadow: 0 0 20px gold, 0 0 40px gold; animation: rankUpPulse 1s ease-out; z-index: 1000; pointer-events: none;}
    @keyframes rankUpPulse { 0% {transform: translate(-50%, -50%) scale(1); opacity: 1;} 100% {transform: translate(-50%, -150%) scale(1.5); opacity: 0;} }
    #reviveFlash { display:none; position: absolute; left: 0; top: 0; width:100vw; height:100vh; background:rgba(0,255,255,0.25); z-index:99999;}
    #gameOver { display: none; margin-top: 8px; font-size: 22px; color: #ff4444; }
    #godlyCooldown {
        position: absolute;
        left: 50%;
        top: 70px;
        transform: translateX(-50%);
        font-size: 18px;
        font-weight: bold;
        color: #00fff7;
        background: rgba(0,0,0,0.6);
        border-radius: 7px;
        padding: 2px 10px;
        display: none;
        z-index: 999;
        box-shadow: 0 0 10px #00fff7;
    }
</style>
</head>
<body>
<!-- HOMESCREEN -->
<div id="homescreen">
    <div id="homeRank">Rank: CADET</div>
    <div id="homeXP">XP: 0</div>
    <div id="homeProgressBar">
        <div id="rankProgressBar">
            <div id="rankProgressFill"></div>
            <div id="rankProgressText">0 / 100 XP</div>
        </div>
    </div>
    <div id="gems">Gems: <span id="gemCount">0</span></div>
    <button id="boosterBtn">BOOSTERS</button>
    <div id="boosterResult"></div>
    <div style="margin-top:10px;font-size:21px;">Choose a Ship:</div>
    <div id="cosmeticList"></div>
    <div id="cosmeticPowerDesc"></div>
    <button id="startBtn">Start Game</button>
</div>
<div id="reviveFlash"></div>
<div id="godlyCooldown"></div>
<!-- IN-GAME UI -->
<div id="ui">
    <div id="rankDisplay">CADET</div>
    <div id="rankProgressBar">
        <div id="rankProgressFill"></div>
        <div id="rankProgressText">0 / 100 XP</div>
    </div>
    <div id="xpDisplay">XP: 0</div>
    <div id="gems">Gems: <span id="gemCount">0</span></div>
    <div>Score: <span id="score">0</span></div>
    <div>High Score: <span id="highScore">0</span></div>
    <div id="rankUpAnimation">⭐ RANK UP ⭐</div>
    <div id="gameOver">💥 GAME OVER – Press R to Restart</div>
</div>
<canvas id="gameCanvas" width="480" height="720"></canvas>

<script>
// ----------- RANKS AND COSMETICS ----------
const XP_RANKS = [
    { name: "Cadet", xp: 0, color: "#A0A0A0", cosmeticId: 0 },
    { name: "Pilot", xp: 100, color: "#00FF00", cosmeticId: 1 },
    { name: "Commander", xp: 300, color: "#0099FF", cosmeticId: 2 },
    { name: "Captain", xp: 600, color: "#FF6600", cosmeticId: 3 },
    { name: "Admiral", xp: 1200, color: "#FF00FF", cosmeticId: 4 },
    { name: "Legend", xp: 2500, color: "#FFFF00", cosmeticId: 5 },
    { name: "Godly", xp: 4000, color: "#00fff7", cosmeticId: 6 }
];
const COSMETICS = [
  {
    id: 0,
    name: "Cadet Viper",
    color: "#00ffcc",
    unlockXp: 0,
    svg: `<polygon points="35,15 16,70 54,70" />`,
    powerName: "Standard",
    powerDesc: "No special power."
  },
  {
    id: 1,
    name: "Pilot Falcon",
    color: "#00FF00",
    unlockXp: 100,
    svg: `<polygon points="35,14 12,54 22,76 48,76 58,54" />
          <rect x="27" y="40" width="16" height="18" rx="5" />`,
    powerName: "Double Boost",
    powerDesc: "Spacebar boost is twice as fast."
  },
  {
    id: 2,
    name: "Commander Nova",
    color: "#0099FF",
    unlockXp: 300,
    svg: `<ellipse cx="35" cy="38" rx="16" ry="29"/>
          <polygon points="35,18 19,70 51,70" />`,
    powerName: "Shield on Dodge",
    powerDesc: "Gain a shield every 5 asteroids dodged."
  },
  {
    id: 3,
    name: "Captain Phoenix",
    color: "#FF6600",
    unlockXp: 600,
    svg: `<polygon points="35,14 16,54 35,85 54,54" style="stroke:#fff;stroke-width:1.5"/>
          <polygon points="30,36 40,36 35,55" />`,
    powerName: "Milestone XP+",
    powerDesc: "+50% XP for score milestones."
  },
  {
    id: 4,
    name: "Admiral Eclipse",
    color: "#FF00FF",
    unlockXp: 1200,
    svg: `<ellipse cx="35" cy="42" rx="18" ry="24" />
          <circle cx="35" cy="32" r="13" />`,
    powerName: "Slow Asteroids",
    powerDesc: "Asteroid speed -20%."
  },
  {
    id: 5,
    name: "Legend Starcruiser",
    color: "#FFFF00",
    unlockXp: 2500,
    svg: `<rect x="19" y="24" rx="9" width="32" height="42"/>
          <polygon points="35,6 25,24 45,24" />
          <ellipse cx="35" cy="70" rx="11" ry="7" />`,
    powerName: "Revive",
    powerDesc: "Survive first crash once per game."
  },
  {
    id: 6,
    name: "Godly Dragon",
    color: "#00fff7",
    unlockXp: 4000,
    svg: `<ellipse cx="35" cy="45" rx="22" ry="30" />
          <polygon points="35,8 16,42 54,42" />
          <ellipse cx="35" cy="70" rx="12" ry="8" />
          <polygon points="16,47 13,62 27,66 21,55" style="fill:#fff;"/>
          <polygon points="54,47 57,62 43,66 49,55" style="fill:#fff;"/>`,
    powerName: "Time Annihilate",
    powerDesc: "Press F every 30sec: destroy all asteroids, freeze for 3s."
  }
];

// ----------- GAME VARS ----------
const XP_PER_ASTEROID = 10, XP_MILESTONE_STEP = 100, XP_PER_MILESTONE = 40, XP_PASSIVE_SEC = 1;
const canvas = document.getElementById("gameCanvas");
const ctx = canvas.getContext("2d");
let score, highScore, gameOver, baseSpeed, boost, difficultyTimer;
let xp, xpMilestoneNext, xpPassiveTimer, shieldActive, dodgedCount, reviveUsed, legendRevives = 0;
let freezeUsed = false, frozenFrames = 0, godlyCooldown = 0;
let boostedShips = [], gems = 0;
let asteroids, stars, particles;
let chosenCosmetic = 0;

// Cadet shielded start effect
let startShieldActive = false, startShieldFrames = 0;

const lanes = [
    canvas.width * 0.25 - 20,
    canvas.width * 0.5 - 20,
    canvas.width * 0.75 - 20
];
const ship = { width:40, height:60, x:lanes[1], targetX:lanes[1], y:canvas.height-100, lane:1, moveSpeed:0.11, tilt:0 };
let animationFrameId = null;

// ----------- RANK & XP -------------
function getXpRank(xpVal) {
    let rank = XP_RANKS[0];
    for (let i = XP_RANKS.length - 1; i >= 0; i--) {
        if (xpVal >= XP_RANKS[i].xp) return XP_RANKS[i];
    }
    return rank;
}
function nextXpRank() {
    const currIndex = XP_RANKS.findIndex(r => r.xp === getXpRank(xp).xp);
    return XP_RANKS[currIndex+1] || null;
}
function getXpRankProgress() {
    const next = nextXpRank();
    if (!next) return 100;
    const curXp = getXpRank(xp).xp, nextXp = next.xp;
    return Math.min(100, Math.max(0, ((xp-curXp)/(nextXp-curXp))*100));
}
function getXpRankProgressText() {
    const next = nextXpRank();
    if (!next) {
        const maxRank = XP_RANKS[XP_RANKS.length - 1];
        return `${xp} / ${maxRank.xp} XP`;
    }
    return `${xp} / ${next.xp} XP`;
}
function saveProgress() {
    localStorage.setItem("stellarDriftXP", xp);
    localStorage.setItem("stellarDriftScore", score);
    localStorage.setItem("stellarDriftRankName", getXpRank(xp).name);
    localStorage.setItem("stellarDriftCosmetic", chosenCosmetic);
    localStorage.setItem("stellarDriftBoostedShips", JSON.stringify(boostedShips));
    localStorage.setItem("stellarDriftGems", gems);
}
function loadProgress() {
    xp = parseInt(localStorage.getItem("stellarDriftXP")||"0",10) || 0;
    chosenCosmetic = parseInt(localStorage.getItem("stellarDriftCosmetic")||"0",10) || 0;
    boostedShips = JSON.parse(localStorage.getItem("stellarDriftBoostedShips")||"[]");
    gems = parseInt(localStorage.getItem("stellarDriftGems")||"0",10) || 0;
    xpMilestoneNext = XP_MILESTONE_STEP;
}

function updateGemsUi() {
    document.getElementById("gemCount").innerText = gems;
}

function updateXpUi() {
    const rank = getXpRank(xp);
    updateGemsUi();
    document.getElementById("xpDisplay").innerText = `XP: ${xp}`;
    document.getElementById("rankProgressFill").style.width = getXpRankProgress()+"%";
    document.getElementById("rankProgressText").innerText = getXpRankProgressText();
    document.getElementById("rankDisplay").innerText = rank.name.toUpperCase();
    document.getElementById("rankDisplay").style.color = rank.color;
    document.getElementById("rankProgressFill").style.background = `linear-gradient(90deg, ${rank.color}, #ffff00)`;
}
function addXp(amount) {
    const oldRankObj = getXpRank(xp);
    xp += amount;
    const newRankObj = getXpRank(xp);
    if (newRankObj.name !== oldRankObj.name) {
        gems++;
        updateGemsUi();
        showBoosterGemGain();
        saveProgress();
    }
    saveProgress();
    updateXpUi();
    updateHomeUi();
    if (newRankObj.name !== oldRankObj.name) triggerRankUpAnimation();
}
function triggerRankUpAnimation() {
    const animation = document.getElementById("rankUpAnimation");
    animation.style.display = "block";
    animation.offsetHeight;
    setTimeout(() => { animation.style.display = "none"; }, 1000);
}

function showBoosterGemGain() {
    document.getElementById("boosterResult").style.display = "block";
    document.getElementById("boosterResult").innerText = "+1 Gem for Rank Up!";
    setTimeout(()=>{document.getElementById("boosterResult").style.display='none';},1800);
}

// ------------- BOOSTERS SHOP (random ship boost) -------------
function isBoosted(shipId) {
    return boostedShips.includes(shipId);
}
function getBoostedDesc(shipId) {
    switch (shipId) {
        case 0: return "Start invulnerable (shield) for 10s";
        case 1: return "Boost is 4x";
        case 2: return "Shield every 2 asteroids dodged";
        case 3: return "+100% XP for score milestones";
        case 4: return "Asteroid speed -40%";
        case 5: return "Two revives per game";
        case 6: return "Half cooldown: use F every 15s";
    }
    return "";
}
document.getElementById("boosterBtn").onclick = function() {
    if (gems < 1) return;
    let available = COSMETICS.map(c=>c.id).filter(id=>!isBoosted(id));
    if (available.length === 0) {
        document.getElementById("boosterResult").innerText = "All ships already BOOSTED!";
        document.getElementById("boosterResult").style.display = "block";
        setTimeout(()=>{document.getElementById("boosterResult").style.display='none';},2000);
        return;
    }
    let r = available[Math.floor(Math.random()*available.length)];
    boostedShips.push(r);
    drawHomeCosmetics();
    drawPowerDesc();
    gems--;
    saveProgress();
    updateGemsUi();
    document.getElementById("boosterResult").innerText = `BOOSTED: ${COSMETICS[r].name} — ${getBoostedDesc(r)}`;
    document.getElementById("boosterResult").style.display = "block";
    setTimeout(()=>{document.getElementById("boosterResult").style.display='none';},3800);
    document.getElementById('boosterBtn').disabled = (gems < 1);
};

// ------------- HOMESCREEN LOGIC -------------
function showHomescreen() {
    if (animationFrameId !== null) {
        cancelAnimationFrame(animationFrameId);
        animationFrameId = null;
    }
    document.getElementById("homescreen").style.display = "block";
    document.getElementById("ui").style.display = "none";
    document.getElementById("boosterResult").style.display = "none";
    updateHomeUi();
    drawHomeCosmetics();
    drawPowerDesc();
    ctx.clearRect(0,0,canvas.width,canvas.height);
    document.getElementById("godlyCooldown").style.display = "none";
}
function hideHomescreen() {
    document.getElementById("homescreen").style.display = "none";
    document.getElementById("ui").style.display = "block";
}
function updateHomeUi() {
    const rank = getXpRank(xp);
    updateGemsUi();
    document.getElementById("homeRank").innerText = "Rank: " + rank.name.toUpperCase();
    document.getElementById("homeRank").style.color = rank.color;
    document.getElementById("homeXP").innerText = "XP: " + xp;
    document.getElementById("rankProgressFill").style.width = getXpRankProgress()+"%";
    document.getElementById("rankProgressText").innerText = getXpRankProgressText();
    document.getElementById('boosterBtn').disabled = (gems < 1);
}
function drawHomeCosmetics() {
    const list = document.getElementById("cosmeticList");
    list.innerHTML = "";
    COSMETICS.forEach(shipCos => {
        let unlocked = xp >= shipCos.unlockXp;
        let div = document.createElement("div");
        div.className = "cosmeticShip" + (chosenCosmetic === shipCos.id ? " selected":"") + (unlocked ? "" : " locked");
        div.title = shipCos.name + (unlocked ? "" : " (Locked)");
        div.style.width = "80px";
        div.style.height = "100px";
        div.innerHTML = `
          <svg width="70" height="80" style="margin-bottom:1px;">
           <g fill="${shipCos.color}" stroke="#222" stroke-width="2">
             ${shipCos.svg}
           </g>
          </svg>
          <div style="font-size:13px;"><b>${shipCos.name}</b></div>
          <div style="font-size:12px;color:#eee;">${unlocked?shipCos.powerName:"🔒"}</div>
        `;
        if (isBoosted(shipCos.id)) {
            let b = document.createElement("span");
            b.className = "booster";
            b.innerText = "BOOSTED!";
            div.appendChild(b);
        }
        if (unlocked) {
            div.onclick = () => { chosenCosmetic = shipCos.id; saveProgress(); drawHomeCosmetics(); drawPowerDesc(); };
        }
        list.appendChild(div);
    });
}
function drawPowerDesc() {
    document.getElementById("cosmeticPowerDesc").innerText = isBoosted(chosenCosmetic) ? "BOOSTED: " + getBoostedDesc(chosenCosmetic) : COSMETICS[chosenCosmetic].powerDesc;
}

// ------------- SHIP POWERS LOGIC -------------
function getPowerActive(name) { return COSMETICS[chosenCosmetic].powerName === name; }
function getBoostMultiplier() {
    if (chosenCosmetic === 1 && isBoosted(1)) return 4;
    if (getPowerActive("Double Boost")) return 2;
    return 1;
}
function getAsteroidSpeedMultiplier() {
    if (chosenCosmetic === 4 && isBoosted(4)) return 0.6;
    if (getPowerActive("Slow Asteroids")) return 0.8;
    return 1;
}
function getMilestoneXpBonus() {
    if (chosenCosmetic === 3 && isBoosted(3)) return 2.0;
    if (getPowerActive("Milestone XP+")) return 1.5;
    return 1;
}
function getShieldEveryDodged() {
    if (chosenCosmetic === 2 && isBoosted(2)) return 2;
    if (getPowerActive("Shield on Dodge")) return 5;
    return null;
}
function getLegendRevives() {
    if (chosenCosmetic === 5 && isBoosted(5)) return 2;
    if (getPowerActive("Revive")) return 1;
    return 0;
}
function getGodlyCooldownFrames() {
    if (chosenCosmetic === 6 && isBoosted(6)) return 900; // 15 sec
    return 1800; // 30 sec
}

// ----------- GAME LOGIC -------------
function resetGame() {
    if (animationFrameId !== null) {
        cancelAnimationFrame(animationFrameId);
        animationFrameId = null;
    }
    baseSpeed = 4;
    score = 0; boost = false; difficultyTimer = 0;
    asteroids = []; particles = []; stars = [];
    shieldActive = false; xpPassiveTimer = 0; gameOver = false;
    dodgedCount = 0; reviveUsed = false; legendRevives = 0; freezeUsed = false; frozenFrames = 0; godlyCooldown = 0;
    // BOOSTER: Cadet shield
    if (chosenCosmetic === 0 && isBoosted(0)) {
        startShieldActive = true;
        startShieldFrames = 600;
    } else {
        startShieldActive = false;
        startShieldFrames = 0;
    }
    ship.lane = 1; ship.x = lanes[1]; ship.targetX = lanes[1]; ship.tilt = 0;
    highScore = localStorage.getItem("stellarHighScore") || 0;
    loadProgress();
    document.getElementById("score").innerText = score;
    document.getElementById("highScore").innerText = highScore;
    document.getElementById("gameOver").style.display = "none";
    updateXpUi();
    for (let i=0;i<100;i++) stars.push({x:Math.random()*canvas.width,y:Math.random()*canvas.height,size:Math.random()*2});
    document.getElementById("reviveFlash").style.display = "none";
    document.getElementById("godlyCooldown").style.display = "none";
}

function updateAsteroids(speed) {
    asteroids.forEach((a, index) => {
        let asteroidSpeed = frozenFrames > 0 ? 0 : speed * getAsteroidSpeedMultiplier();
        a.y += asteroidSpeed;
        if (!gameOver &&
            ship.x < a.x + a.width &&
            ship.x + ship.width > a.x &&
            ship.y < a.y + a.height &&
            ship.y + ship.height > a.y
        ) {
            // BOOSTED Cadet Viper: shield start for 10s
            if (chosenCosmetic === 0 && isBoosted(0) && startShieldActive) return;
            // BOOSTER: Legend Starcruiser — two revives
            if (getPowerActive("Revive")) {
                const maxRevives = getLegendRevives();
                if (legendRevives < maxRevives) {
                    legendRevives++;
                    let safeLaneX = lanes[ship.lane];
                    asteroids = asteroids.filter(other =>
                        other.x !== safeLaneX || other.y > ship.y + ship.height || other.y + other.height < ship.y
                    );
                    document.getElementById("reviveFlash").style.background = "rgba(255,255,0,0.18)";
                    document.getElementById("reviveFlash").style.display = "block";
                    setTimeout(function () {
                        document.getElementById("reviveFlash").style.display = "none";
                    }, 300);
                    for(let i=0;i<30;i++) {
                        particles.push({
                            x: ship.x + ship.width/2,
                            y: ship.y + ship.height/2,
                            vx: (Math.random()-0.5)*6,
                            vy: (Math.random()-0.5)*6,
                            life: 14
                        });
                    }
                    shieldActive = false;
                    return;
                }
            }
            if (getPowerActive("Shield on Dodge") && shieldActive) {
                shieldActive = false;
                asteroids.splice(index, 1);
                saveProgress();
                return;
            }
            gameOver = true;
            createExplosion();
            document.getElementById("gameOver").style.display = "block";
            saveProgress();
            setTimeout(showHomescreen,1100);
        }
        if (a.y > canvas.height) {
            asteroids.splice(index, 1);
            if (!gameOver) {
                score += 10;
                document.getElementById("score").innerText = score;
                addXp(XP_PER_ASTEROID);
                dodgedCount++;
                let every = getShieldEveryDodged();
                if (every && dodgedCount % every === 0) {
                    shieldActive = true;
                }
                while (score >= xpMilestoneNext) {
                    addXp(Math.floor(XP_PER_MILESTONE * getMilestoneXpBonus()));
                    xpMilestoneNext += XP_MILESTONE_STEP;
                }
                saveProgress();
                if (score > highScore) {
                    highScore = score;
                    localStorage.setItem("stellarHighScore", highScore);
                    document.getElementById("highScore").innerText = highScore;
                }
            }
        }
    });
    if (frozenFrames > 0) frozenFrames--;
    if (startShieldActive && startShieldFrames > 0) {
        startShieldFrames--;
        if (startShieldFrames === 0) startShieldActive = false;
    }
}

function createExplosion() {
    for (let i = 0; i < 30; i++) {
        particles.push({
            x: ship.x + ship.width/2,
            y: ship.y + ship.height/2,
            vx: (Math.random() - 0.5) * 6,
            vy: (Math.random() - 0.5) * 6,
            life: 30
        });
    }
}

function drawShip() {
    const shipCos = COSMETICS[chosenCosmetic];
    ctx.save();
    ctx.translate(ship.x + ship.width/2, ship.y + ship.height/2);
    ctx.rotate(ship.tilt);
    ctx.beginPath();
    switch (chosenCosmetic) {
        case 0:
            ctx.moveTo(0, -25);
            ctx.lineTo(-19, 29);
            ctx.lineTo(19, 29);
            ctx.closePath();
            ctx.fillStyle = shipCos.color;
            ctx.fill();
            break;
        case 1:
            ctx.moveTo(0,-24);ctx.lineTo(-23,18);ctx.lineTo(-13,38);ctx.lineTo(13,38);ctx.lineTo(23,18);ctx.closePath();ctx.fillStyle=shipCos.color;ctx.fill();
            ctx.beginPath();ctx.rect(-8,0,16,18);ctx.fill();
            break;
        case 2:
            ctx.ellipse(0,0,16,29,0,0,2*Math.PI);
            ctx.fillStyle=shipCos.color;ctx.fill();
            ctx.beginPath();ctx.moveTo(0,-20);ctx.lineTo(-16,32);ctx.lineTo(16,32);ctx.closePath();ctx.fill();
            break;
        case 3:
            ctx.moveTo(0,-27);ctx.lineTo(-18,17);ctx.lineTo(0,40);ctx.lineTo(18,17);ctx.closePath();ctx.fillStyle=shipCos.color;ctx.fill();
            ctx.beginPath();ctx.moveTo(-5,12);ctx.lineTo(5,12);ctx.lineTo(0,29);ctx.closePath();ctx.fill();
            break;
        case 4:
            ctx.ellipse(0,6,18,24,0,0,2*Math.PI);ctx.fillStyle=shipCos.color;ctx.fill();
            ctx.beginPath();ctx.arc(0,-4,13,0,2*Math.PI);ctx.fill();
            break;
        case 5:
            ctx.beginPath();ctx.rect(-16, -16,32,42);ctx.fillStyle=shipCos.color;ctx.fill();
            ctx.beginPath();ctx.moveTo(0,-34);ctx.lineTo(-10,-16);ctx.lineTo(10,-16);ctx.closePath();ctx.fill();
            ctx.beginPath();ctx.ellipse(0,26,11,7,0,0,2*Math.PI);ctx.fill();
            break;
        case 6: // Godly Dragon
            ctx.ellipse(0,4,22,30,0,0,2*Math.PI); ctx.fillStyle=shipCos.color; ctx.fill();
            ctx.beginPath(); ctx.moveTo(0,-34); ctx.lineTo(-19,10); ctx.lineTo(19,10); ctx.closePath(); ctx.fill();
            ctx.beginPath(); ctx.ellipse(0,25,12,8,0,0,2*Math.PI); ctx.fill();
            ctx.beginPath(); ctx.moveTo(-19,7); ctx.lineTo(-21,23); ctx.lineTo(-8,27); ctx.lineTo(-14,13); ctx.closePath(); ctx.fillStyle="#fff"; ctx.fill();
            ctx.beginPath(); ctx.moveTo(19,7); ctx.lineTo(21,23); ctx.lineTo(8,27); ctx.lineTo(14,13); ctx.closePath(); ctx.fillStyle="#fff"; ctx.fill();
            break;
    }
    if ((chosenCosmetic === 0 && isBoosted(0) && startShieldActive) 
     || (getPowerActive("Shield on Dodge") && shieldActive)) {
        ctx.lineWidth = 2;
        ctx.strokeStyle = "#0ff";
        ctx.beginPath();
        ctx.arc(0,0,26,0,2*Math.PI);
        ctx.stroke();
    }
    ctx.restore();
}

function spawnAsteroid() {
    const count = Math.random() < 0.8 ? 1 : 2;
    const shuffledLanes = [...lanes].sort(() => Math.random() - 0.5);
    let spawned = 0;
    for (let i = 0; i < shuffledLanes.length && spawned < count; i++) {
        const laneX = shuffledLanes[i];
        if (!asteroids.some(a => a.x === laneX && a.y < 150)) {
            asteroids.push({x: laneX, y: -60, width: 40, height: 40});
            spawned++;
        }
    }
}
function preventImpossibleRows() {
    const topAsteroids = lanes.map(laneX =>
        asteroids.filter(a => a.x === laneX && a.y < 150)
    );
    if (topAsteroids.every(arr => arr.length > 0)) {
        const laneToClear = Math.floor(Math.random() * 3);
        const asteroidToRemove = topAsteroids[laneToClear][0];
        if (asteroidToRemove) {
            const index = asteroids.indexOf(asteroidToRemove);
            if (index > -1) asteroids.splice(index, 1);
        }
    }
}
function drawStars(speed) {
    ctx.fillStyle = "white";
    stars.forEach(s => {
        ctx.fillRect(s.x, s.y, s.size, s.size);
        s.y += speed * 0.5;
        if (s.y > canvas.height) s.y = 0;
    });
}
function drawAsteroids() {
    ctx.fillStyle = "#888";
    asteroids.forEach(a => {
        ctx.beginPath();
        ctx.arc(a.x + 20, a.y + 20, 20, 0, Math.PI * 2);
        ctx.fill();
    });
}
function updateShip() {
    ship.x += (ship.targetX - ship.x) * ship.moveSpeed;
    ship.tilt = (ship.targetX - ship.x) * 0.005;
}
function updateParticles() {
    ctx.fillStyle = "orange";
    particles.forEach((p, i) => {
        p.x += p.vx;
        p.y += p.vy;
        p.life--;
        ctx.fillRect(p.x, p.y, 4, 4);
        if (p.life <= 0) particles.splice(i, 1);
    });
}
function updatePassiveXp() {
    if (!gameOver) {
        xpPassiveTimer++;
        if (xpPassiveTimer >= 60) { addXp(XP_PASSIVE_SEC); xpPassiveTimer = 0; }
    }
}
function updateGodlyCooldown() {
    if (getPowerActive("Time Annihilate") && !gameOver) {
        const cdElem = document.getElementById("godlyCooldown");
        if (godlyCooldown > 0) {
            cdElem.style.display = "block";
            cdElem.innerText = `Godly Power Ready In: ${Math.ceil(godlyCooldown/60)}s`;
        } else {
            cdElem.style.display = "block";
            cdElem.innerText = `Press F for Time Annihilate!`;
        }
    } else {
        document.getElementById("godlyCooldown").style.display = "none";
    }
}
function update() {
    ctx.clearRect(0, 0, canvas.width, canvas.height);
    let currentSpeed = frozenFrames > 0 ? 0 : baseSpeed + (baseSpeed * 0.5 * getBoostMultiplier()) + (boost ? 4 * getBoostMultiplier() : 0);
    drawStars(currentSpeed);
    if (!gameOver) {
        difficultyTimer++;
        if (frozenFrames === 0 && difficultyTimer % 300 === 0) baseSpeed += 0.5;
        updateShip();
        updateAsteroids(currentSpeed);
        preventImpossibleRows();
        if (frozenFrames === 0 && Math.random() < 0.03) spawnAsteroid();
        if (godlyCooldown > 0) godlyCooldown--;
    }
    drawShip();
    drawAsteroids();
    updateParticles();
    updatePassiveXp();
    updateGodlyCooldown();
    animationFrameId = requestAnimationFrame(update);
}
document.getElementById("startBtn").onclick = () => {
    hideHomescreen();
    resetGame();
    animationFrameId = requestAnimationFrame(update);
};
showHomescreen();
document.addEventListener("keydown", e => {
    if(!document.getElementById("ui").style.display || document.getElementById("ui").style.display==="none") return;
    if (e.key === "ArrowLeft" || e.key === "a") { if (ship.lane > 0) ship.lane--; }
    if (e.key === "ArrowRight" || e.key === "d") { if (ship.lane < 2) ship.lane++; }
    if (e.key === " ") boost = true;
    if (e.key === "r" && gameOver) { resetGame(); reviveUsed=false; freezeUsed=false; frozenFrames=0; godlyCooldown=0; legendRevives=0; animationFrameId = requestAnimationFrame(update); }
    if (getPowerActive("Time Annihilate") && e.key.toLowerCase() === "f" && godlyCooldown === 0 && !gameOver) {
        // Destroys all asteroids and freezes for 3s
        asteroids = [];
        godlyCooldown = getGodlyCooldownFrames();
        frozenFrames = 180; // 3s
        document.getElementById("reviveFlash").style.background = "rgba(0,255,255,0.25)";
        document.getElementById("reviveFlash").style.display = "block";
        setTimeout(() => {
            document.getElementById("reviveFlash").style.background = "rgba(255,255,0,0.18)";
            document.getElementById("reviveFlash").style.display = "none";
        }, 400);
    }
    ship.targetX = lanes[ship.lane];
});
document.addEventListener("keyup", e => { if (e.key === " ") boost = false; });
</script>
</body>
</html>
