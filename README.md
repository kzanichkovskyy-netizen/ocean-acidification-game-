# ocean-acidification-game-
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Whiskey Creek Crisis — Save the Oysters</title>
<link rel="preconnect" href="https://fonts.googleapis.com">
<link href="https://fonts.googleapis.com/css2?family=Orbitron:wght@700;900&family=Exo+2:wght@300;400;600;700&display=swap" rel="stylesheet">
<style>
*, *::before, *::after { margin:0; padding:0; box-sizing:border-box; }
html, body { width:100%; height:100%; background:#020810; overflow:hidden; }

canvas { position:fixed; inset:0; width:100vw; height:100vh; cursor:crosshair; z-index:0; }

#ui { position:fixed; inset:0; pointer-events:none; z-index:10; }

/* ── HUD ── */
#hud { display:none; position:absolute; inset:0; }

#hud-ph { position:absolute; top:18px; left:20px; display:flex; flex-direction:column; gap:5px; }
#ph-row { position:relative; display:flex; align-items:center; gap:10px; }
#ph-bar-wrap { width:190px; height:16px; background:rgba(5,15,40,0.85); border-radius:8px; border:1.5px solid rgba(255,255,255,0.12); overflow:hidden; }
#ph-bar-fill { height:100%; border-radius:8px; transition:width 0.35s ease, background-color 0.35s ease; }
#ph-value { font-family:'Exo 2',sans-serif; font-size:13px; font-weight:600; color:#cce8ff; letter-spacing:0.05em; white-space:nowrap; }
#ph-label { font-size:10px; font-weight:400; color:rgba(150,200,255,0.5); letter-spacing:0.14em; text-transform:uppercase; }
#diff-badge { font-family:'Orbitron',sans-serif; font-size:10px; font-weight:700; letter-spacing:0.1em; }

#hud-score { position:absolute; top:14px; left:50%; transform:translateX(-50%); text-align:center; }
#score-num { font-family:'Orbitron',sans-serif; font-size:clamp(22px,2.5vw,38px); font-weight:900; color:#ffd020; letter-spacing:0.05em; text-shadow:0 0 16px rgba(255,208,32,0.65); line-height:1; }
#score-label { font-size:10px; font-weight:400; color:rgba(255,220,50,0.4); letter-spacing:0.2em; text-transform:uppercase; margin-top:2px; }

#hud-right { position:absolute; top:14px; right:20px; text-align:right; display:flex; flex-direction:column; gap:4px; }
#timer-val { font-family:'Exo 2',sans-serif; font-size:clamp(16px,1.8vw,26px); font-weight:700; color:#aad4ff; letter-spacing:0.04em; }
#level-val { font-size:10px; font-weight:400; color:rgba(150,200,255,0.45); letter-spacing:0.16em; text-transform:uppercase; }
#lives-row { display:flex; flex-direction:row-reverse; gap:6px; margin-top:4px; justify-content:flex-end; }
.life-dot { width:10px; height:10px; border-radius:2px; background:#20c4ff; box-shadow:0 0 8px rgba(32,196,255,0.7); }

#warning-bar { display:none; position:absolute; bottom:20px; left:50%; transform:translateX(-50%); font-family:'Exo 2',sans-serif; font-size:clamp(12px,1.2vw,16px); font-weight:700; color:#e02828; letter-spacing:0.06em; text-shadow:0 0 16px rgba(220,40,40,0.9); white-space:nowrap; animation:warnpulse 0.9s ease-in-out infinite; }
@keyframes warnpulse { 0%,100%{opacity:0.35} 50%{opacity:1} }

/* ── Screens ── */
.screen { display:none; position:absolute; inset:0; flex-direction:column; align-items:center; justify-content:center; pointer-events:auto; }

#menu-screen { background:rgba(2,8,22,0.8); }
#end-screen  { background:rgba(2,6,18,0.85); gap:7px; }

/* typography */
.screen-title { font-family:'Orbitron',sans-serif; font-size:clamp(36px,5vw,70px); font-weight:900; color:#ff5a3c; letter-spacing:0.06em; text-shadow:0 0 40px rgba(255,90,60,0.65),0 0 80px rgba(255,90,60,0.25); margin-bottom:8px; }
.screen-sub   { font-family:'Exo 2',sans-serif; font-size:clamp(12px,1.3vw,18px); font-weight:600; color:#4aa8ee; letter-spacing:0.04em; margin-bottom:16px; }
.screen-how   { font-family:'Exo 2',sans-serif; font-size:clamp(11px,1vw,15px); font-weight:400; color:rgba(200,230,255,0.78); line-height:1.85; text-align:center; margin-bottom:20px; }
.section-lbl  { font-family:'Orbitron',sans-serif; font-size:clamp(10px,1vw,14px); font-weight:700; color:rgba(200,225,255,0.8); letter-spacing:0.18em; margin-bottom:12px; }

/* diff cards */
#diff-cards { display:flex; gap:clamp(10px,2vw,24px); margin-bottom:20px; }
.diff-card {
  pointer-events:auto; cursor:pointer;
  width:clamp(120px,16vw,200px);
  padding:clamp(14px,1.8vw,22px) clamp(10px,1.2vw,16px);
  border-radius:14px;
  background:rgba(5,15,40,0.75);
  border:1.5px solid rgba(100,150,220,0.22);
  text-align:center;
  display:flex; flex-direction:column; align-items:center; gap:5px;
  transition:transform 0.18s, border-color 0.18s, background 0.18s, box-shadow 0.18s;
}
.diff-card:hover { transform:translateY(-3px); border-color:rgba(255,255,255,0.25); background:rgba(20,40,90,0.7); }
.diff-card.selected { transform:translateY(-4px) scale(1.03); }
.diff-card[data-key="easy"].selected     { border-color:#28d464; background:rgba(40,212,100,0.1); box-shadow:0 0 22px rgba(40,212,100,0.22); }
.diff-card[data-key="normal"].selected   { border-color:#ffd020; background:rgba(255,208,32,0.08); box-shadow:0 0 22px rgba(255,208,32,0.18); }
.diff-card[data-key="hardcore"].selected { border-color:#ff2020; background:rgba(255,32,32,0.1); box-shadow:0 0 22px rgba(255,32,32,0.22); }
.diff-emoji { font-size:clamp(22px,2.8vw,38px); line-height:1; }
.diff-name  { font-family:'Orbitron',sans-serif; font-size:clamp(10px,1vw,14px); font-weight:700; letter-spacing:0.1em; }
.diff-card[data-key="easy"]     .diff-name { color:#28d464; }
.diff-card[data-key="normal"]   .diff-name { color:#ffd020; }
.diff-card[data-key="hardcore"] .diff-name { color:#ff3030; }
.diff-tag { font-family:'Exo 2',sans-serif; font-size:clamp(9px,0.78vw,11px); font-weight:400; color:rgba(170,205,255,0.6); letter-spacing:0.04em; }

/* buttons */
.btn-start {
  pointer-events:auto; cursor:pointer;
  font-family:'Orbitron',sans-serif; font-size:clamp(12px,1.2vw,17px); font-weight:700; letter-spacing:0.1em;
  padding:clamp(10px,1.2vh,15px) clamp(28px,4vw,50px);
  border-radius:40px; border:2px solid currentColor; background:transparent;
  transition:transform 0.15s, box-shadow 0.15s, background 0.15s;
  animation:pulse 2.2s ease-in-out infinite;
}
.btn-start:hover { transform:scale(1.06); box-shadow:0 0 22px currentColor; background:rgba(255,255,255,0.07); animation:none; }
@keyframes pulse { 0%,100%{transform:scale(1)} 50%{transform:scale(1.025)} }
.btn-easy     { color:#28d464; }
.btn-normal   { color:#ffd020; }
.btn-hardcore { color:#ff3030; }
.fact-note { font-family:'Exo 2',sans-serif; font-size:clamp(9px,0.78vw,11px); font-weight:300; font-style:italic; color:rgba(120,175,255,0.45); margin-top:14px; text-align:center; max-width:640px; }

/* end screen */
.end-title { font-family:'Orbitron',sans-serif; font-size:clamp(28px,4.5vw,62px); font-weight:900; letter-spacing:0.05em; line-height:1; margin-bottom:3px; }
.end-title.win  { color:#28d464; text-shadow:0 0 30px rgba(40,212,100,0.55); }
.end-title.lose { color:#e02828; text-shadow:0 0 30px rgba(220,40,40,0.55); }
.end-badge { font-family:'Orbitron',sans-serif; font-size:clamp(10px,0.9vw,13px); font-weight:700; letter-spacing:0.14em; margin-bottom:3px; }
.end-sub  { font-family:'Exo 2',sans-serif; font-size:clamp(12px,1.2vw,17px); font-weight:600; color:rgba(200,225,255,0.85); margin-bottom:5px; }
.end-score{ font-family:'Orbitron',sans-serif; font-size:clamp(20px,2.8vw,42px); font-weight:900; color:#ffd020; letter-spacing:0.06em; text-shadow:0 0 16px rgba(255,208,32,0.55); margin-bottom:14px; }
.end-facts{ font-family:'Exo 2',sans-serif; font-size:clamp(10px,0.9vw,13px); font-weight:400; color:rgba(180,215,255,0.68); line-height:1.9; text-align:center; margin-bottom:18px; }
.end-btns { display:flex; gap:clamp(10px,2vw,18px); }
.btn-sec {
  pointer-events:auto; cursor:pointer;
  font-family:'Orbitron',sans-serif; font-size:clamp(10px,1vw,14px); font-weight:700; letter-spacing:0.1em;
  padding:clamp(8px,1vh,13px) clamp(20px,2.8vw,38px);
  border-radius:40px; border:1.8px solid; background:transparent;
  transition:transform 0.15s, background 0.15s;
}
.btn-sec:hover { transform:scale(1.05); background:rgba(255,255,255,0.06); }
.btn-menu { color:rgba(140,190,255,0.85); border-color:rgba(100,160,255,0.45); }

#fullscreen-btn {
  position:fixed; bottom:16px; right:16px;
  pointer-events:auto; cursor:pointer;
  background:rgba(10,30,80,0.6); border:1.5px solid rgba(40,120,200,0.35);
  color:rgba(160,210,255,0.75); font-family:'Exo 2',sans-serif; font-size:12px; font-weight:600;
  padding:6px 13px; border-radius:20px; transition:all 0.2s; letter-spacing:0.04em;
}
#fullscreen-btn:hover { background:rgba(20,60,160,0.8); color:#fff; border-color:rgba(80,160,255,0.7); }
</style>
</head>
<body>
<canvas id="c"></canvas>
<div id="ui">

  <!-- HUD -->
  <div id="hud">
    <div id="hud-ph">
      <div id="ph-row">
        <div id="ph-bar-wrap"><div id="ph-bar-fill"></div></div>
        <span id="ph-value">pH 8.20</span>
      </div>
      <span id="ph-label">Bay Acidity</span>
      <span id="diff-badge"></span>
    </div>
    <div id="hud-score">
      <div id="score-num">0</div>
      <div id="score-label">Larvae Saved</div>
    </div>
    <div id="hud-right">
      <div id="timer-val">60s</div>
      <div id="level-val">Level 1</div>
      <div id="lives-row"></div>
    </div>
    <div id="warning-bar">⚠ &nbsp;CRITICAL ACIDITY — STOP THE UPWELLING!</div>
  </div>

  <!-- Menu -->
  <div id="menu-screen" class="screen" style="display:flex;">
    <h1 class="screen-title">WHISKEY CREEK</h1>
    <p class="screen-sub">Save the Hatchery — Netarts Bay Acidification Crisis</p>
    <p class="screen-how">
      🔴 &nbsp;Click upwelled CO₂ plumes before they sink to the bay floor<br>
      🦪 &nbsp;Collect oyster larvae for bonus points<br>
      💧 &nbsp;Keep pH above 7.5 or the larvae dissolve and the hatchery collapses
    </p>
    <p class="section-lbl">SELECT DIFFICULTY</p>
    <div id="diff-cards">
      <div class="diff-card" data-key="easy">
        <div class="diff-emoji">🦪</div>
        <div class="diff-name">EASY</div>
        <div class="diff-tag">Calm upwelling · 5 tanks · 90s</div>
      </div>
      <div class="diff-card selected" data-key="normal">
        <div class="diff-emoji">🌊</div>
        <div class="diff-name">NORMAL</div>
        <div class="diff-tag">2007 conditions · 3 tanks · 60s</div>
      </div>
      <div class="diff-card" data-key="hardcore">
        <div class="diff-emoji">☠️</div>
        <div class="diff-name">HARDCORE</div>
        <div class="diff-tag">Mass die-off · 1 tank · 45s</div>
      </div>
    </div>
    <button id="start-btn" class="btn-start btn-normal">▶ &nbsp;START NORMAL</button>
    <p class="fact-note">🌊 In 2007–2008, Pacific upwelling brought water with pH as low as 7.6 into Whiskey Creek Hatchery, killing nearly all Pacific oyster larvae</p>
  </div>

  <!-- End -->
  <div id="end-screen" class="screen">
    <h1 class="end-title" id="end-title">HATCHERY SAVED! 🦪</h1>
    <div class="end-badge" id="end-badge"></div>
    <p class="end-sub" id="end-sub">You kept the ocean healthy!</p>
    <div class="end-score" id="end-score">Score: 0</div>
    <div class="end-facts">
      💡 Whiskey Creek Hatchery in Netarts Bay, Oregon suffered near-total larval die-offs in 2007–2009.<br>
      💡 Pacific upwelling pulls deep, CO₂-rich water onto the shelf — intensified by climate change.<br>
      💡 Oyster larvae can't form shells when seawater aragonite saturation drops below 1.0.<br>
      💡 The crisis spurred real-time pH monitoring still used by hatcheries on the Oregon coast today.
    </div>
    <div class="end-btns">
      <button id="retry-btn" class="btn-sec">↺ &nbsp;RETRY</button>
      <button id="menu-btn"  class="btn-sec btn-menu">⌂ &nbsp;MENU</button>
    </div>
  </div>

  <button id="fullscreen-btn">⛶ Fullscreen</button>
</div>

<script>
const canvas = document.getElementById('c');
const ctx    = canvas.getContext('2d');
let W, H;
function resize() { W = canvas.width = window.innerWidth; H = canvas.height = window.innerHeight; }
resize();
window.addEventListener('resize', () => { resize(); if (gs) gs.corals = buildCorals(); });

document.getElementById('fullscreen-btn').addEventListener('click', () => {
  if (!document.fullscreenElement) document.documentElement.requestFullscreen();
  else document.exitFullscreen();
});

// ── difficulty ────────────────────────────────────────────
const DIFF = {
  easy:     { label:'EASY',     emoji:'🦪', color:'#28d464',
              ph:8.2, lives:5, time:90,  spawnBase:2800, spawnMin:900, spawnLvl:180,
              spdBase:0.0018, spdRand:0.0018, spdLvl:0.0002,
              phDrop:0.055, phRec:0.00015, scoreHit:10, scoreShell:6, shellRate:2500, winPh:7.7 },
  normal:   { label:'NORMAL',   emoji:'🌊', color:'#ffd020',
              ph:8.2, lives:3, time:60,  spawnBase:1800, spawnMin:500, spawnLvl:220,
              spdBase:0.003,  spdRand:0.003,  spdLvl:0.0004,
              phDrop:0.09,  phRec:0.00007, scoreHit:10, scoreShell:5, shellRate:3200, winPh:7.8 },
  hardcore: { label:'HARDCORE', emoji:'💀', color:'#ff2020',
              ph:8.0, lives:1, time:45,  spawnBase:900,  spawnMin:200, spawnLvl:120,
              spdBase:0.005,  spdRand:0.005,  spdLvl:0.0008,
              phDrop:0.15,  phRec:0.00003, scoreHit:20, scoreShell:10,shellRate:4500, winPh:7.9 },
};
let currentDiff = 'normal';

const C = { coral:'#ff5a3c', green:'#28d464', red:'#e02828', co2:'#c04028', co2l:'#e05838', shell:'#e0c89a', shellD:'#706050', yellow:'#ffd020' };

// ── state ─────────────────────────────────────────────────
let gs;
function sandH() { return Math.max(50, H * 0.09); }
function buildCorals() {
  const n = Math.max(6, Math.floor(W / 100));
  return Array.from({length:n}, (_,i) => mkCoral((W/(n+1))*(i+1), H-sandH()-10));
}
function newState(diff) {
  const d = DIFF[diff];
  return { diff, ph:d.ph, score:0, lives:d.lives, level:1, timeLeft:d.time,
           tick:0, spawnT:0, shellT:0, co2s:[], shells:[], parts:[], corals:buildCorals(), state:'menu' };
}

// ── coral ─────────────────────────────────────────────────
function mkCoral(x,y) {
  return { x,y, health:1, sway:Math.random()*Math.PI*2, h:H*0.07+Math.random()*H*0.08, br:3+Math.floor(Math.random()*4) };
}
function updCoral(c) {
  const t = Math.max(0,(gs.ph-7.5)/0.7);
  c.health += (t-c.health)*0.004; c.health = Math.max(0,Math.min(1,c.health));
}
function drwCoral(c) {
  const sw = Math.sin(c.sway+gs.tick*0.018)*(W*0.002);
  // Health-based colour: healthy = blue-green, bleached = grey-brown
  const hr = lerp(0x90,0x20,c.health)|0, hg = lerp(0x80,0xc0,c.health)|0, hb = lerp(0x70,0xff,c.health)|0;
  const col = `rgb(${hr},${hg},${hb})`;
  const tw = Math.max(8, W*0.018), th = c.h;
  // Draw hatchery tank: vertical post + horizontal rack bars (oyster cage)
  ctx.strokeStyle=col; ctx.lineWidth=Math.max(2,W*0.003); ctx.lineCap='round';
  // center post
  ctx.beginPath(); ctx.moveTo(c.x+sw,c.y); ctx.lineTo(c.x+sw,c.y-th); ctx.stroke();
  // cross bars (cage rungs)
  const rungs = Math.max(3,c.br+1);
  ctx.lineWidth=Math.max(1,W*0.0015);
  for(let i=1;i<=rungs;i++){
    const ry = c.y-th*(i/(rungs+1));
    const rx = c.x+sw;
    ctx.beginPath(); ctx.moveTo(rx-tw,ry); ctx.lineTo(rx+tw,ry); ctx.stroke();
    // small larvae dots on healthy cages
    if(c.health>0.5 && i%2===0){
      ctx.fillStyle=`rgba(${hr},${hg},${hb},${c.health*0.7})`;
      ctx.beginPath(); ctx.arc(rx-tw*0.4,ry,Math.max(1.5,W*0.003),0,Math.PI*2); ctx.fill();
      ctx.beginPath(); ctx.arc(rx+tw*0.4,ry,Math.max(1.5,W*0.003),0,Math.PI*2); ctx.fill();
    }
  }
  // Float buoy on top
  ctx.fillStyle=`rgba(${hr},${hg},${hb},0.9)`;
  ctx.beginPath(); ctx.arc(c.x+sw,c.y-th,Math.max(3,W*0.005),0,Math.PI*2); ctx.fill();
}

// ── CO2 ───────────────────────────────────────────────────
function mkCO2() {
  const d=DIFF[gs.diff], r=W*0.022+Math.random()*W*0.015;
  const surge = gs.diff==='hardcore' && Math.random()<0.2;
  return { x:r+Math.random()*(W-r*2), y:-r*2, r:surge?r*0.7:r,
           spd:(H*d.spdBase)+Math.random()*(H*d.spdRand)+gs.level*(H*d.spdLvl)+(surge?H*0.006:0),
           wob:Math.random()*Math.PI*2, active:true, surge };
}
function updCO2(b) {
  b.y+=b.spd; b.wob+=0.065; b.x+=Math.sin(b.wob)*0.9;
  if(b.y>H+40){ gs.ph=Math.max(7.0,gs.ph-DIFF[gs.diff].phDrop); b.active=false; spawnParts(b.x,H-sandH(),C.co2,10); }
}
function drwCO2(b) {
  const x=b.x|0, y=b.y|0;
  const grd=ctx.createRadialGradient(x,y,2,x,y,b.r*1.7);
  grd.addColorStop(0,b.surge?'rgba(255,20,20,0.5)':'rgba(200,64,40,0.4)'); grd.addColorStop(1,'rgba(0,0,0,0)');
  ctx.fillStyle=grd; ctx.beginPath(); ctx.arc(x,y,b.r*1.7,0,Math.PI*2); ctx.fill();
  ctx.fillStyle=b.surge?'rgba(240,20,20,0.92)':'rgba(192,64,40,0.88)';
  ctx.beginPath(); ctx.arc(x,y,b.r,0,Math.PI*2); ctx.fill();
  ctx.strokeStyle=b.surge?'#ff6060':C.co2l; ctx.lineWidth=1.5;
  ctx.beginPath(); ctx.arc(x,y,b.r,0,Math.PI*2); ctx.stroke();
  ctx.fillStyle='#fff'; ctx.font=`bold ${b.r*0.7|0}px sans-serif`;
  ctx.textAlign='center'; ctx.textBaseline='middle';
  ctx.fillText(b.surge?'⚡UPWLL':'CO₂',x,y);
}

// ── shell ─────────────────────────────────────────────────
function mkShell() {
  return { x:40+Math.random()*(W-80), y:-20, spd:H*0.002+Math.random()*H*0.002, healthy:gs.ph>7.8, active:true };
}
function updShell(s) { s.y+=s.spd; if(s.y>H+30) s.active=false; }
function drwShell(s) {
  const x=s.x|0,y=s.y|0,r=W*0.018;
  // Oyster larva: small teardrop/oval shape
  const grd=ctx.createRadialGradient(x-r*0.3,y-r*0.3,1,x,y,r);
  if(s.healthy){
    grd.addColorStop(0,'rgba(230,240,255,0.95)'); grd.addColorStop(0.5,'rgba(180,220,240,0.85)'); grd.addColorStop(1,'rgba(100,160,200,0.6)');
  } else {
    grd.addColorStop(0,'rgba(160,140,110,0.9)'); grd.addColorStop(1,'rgba(80,65,45,0.7)');
  }
  ctx.fillStyle=grd;
  ctx.beginPath(); ctx.ellipse(x,y,r,r*0.75,0.3,0,Math.PI*2); ctx.fill();
  ctx.strokeStyle=s.healthy?'rgba(180,220,255,0.7)':'rgba(90,70,50,0.8)'; ctx.lineWidth=1;
  ctx.beginPath(); ctx.ellipse(x,y,r,r*0.75,0.3,0,Math.PI*2); ctx.stroke();
  // veliger shell hinge line
  ctx.strokeStyle=s.healthy?'rgba(120,180,220,0.5)':'rgba(60,50,35,0.5)'; ctx.lineWidth=0.8;
  ctx.beginPath(); ctx.moveTo(x-r*0.5,y); ctx.lineTo(x+r*0.5,y); ctx.stroke();
  // label
  ctx.fillStyle=s.healthy?'rgba(255,255,255,0.9)':'rgba(180,160,130,0.8)';
  ctx.font=`bold ${r*0.65|0}px sans-serif`; ctx.textAlign='center'; ctx.textBaseline='middle';
  ctx.fillText('🦪',x,y);
}

// ── particles ─────────────────────────────────────────────
function spawnParts(x,y,col,n) {
  for(let i=0;i<n;i++){const a=Math.random()*Math.PI*2,sp=1+Math.random()*4; gs.parts.push({x,y,vx:Math.cos(a)*sp,vy:Math.sin(a)*sp,life:30,col});}
}
function updParts() { gs.parts.forEach(p=>{p.x+=p.vx;p.y+=p.vy;p.vy+=0.12;p.life--;}); gs.parts=gs.parts.filter(p=>p.life>0); }
function drwParts() {
  gs.parts.forEach(p=>{ctx.globalAlpha=p.life/30;ctx.fillStyle=p.col;ctx.beginPath();ctx.arc(p.x|0,p.y|0,Math.max(1,p.life/7),0,Math.PI*2);ctx.fill();});
  ctx.globalAlpha=1;
}

// ── background ────────────────────────────────────────────
function drawBG() {
  const hc = gs.diff==='hardcore';
  const grd=ctx.createLinearGradient(0,0,0,H);
  grd.addColorStop(0,hc?'#041208':'#020d22'); grd.addColorStop(0.5,hc?'#081f10':'#071a50'); grd.addColorStop(1,hc?'#0a2812':'#0a2a6e');
  ctx.fillStyle=grd; ctx.fillRect(0,0,W,H);
  for(let i=0;i<6;i++){
    const a=Math.sin(gs.tick*0.008+i)*0.2+Math.PI/2,x0=W/6*i+W/12,rw=W*0.025;
    ctx.save(); ctx.globalAlpha=0.048; ctx.fillStyle=hc?'#ffb0a0':'#a0d4ff';
    ctx.beginPath(); ctx.moveTo(x0-rw,0); ctx.lineTo(x0+rw,0);
    const dx=Math.cos(a)*H*0.75;
    ctx.lineTo(x0+dx+rw*2,H*0.75); ctx.lineTo(x0+dx-rw*2,H*0.75); ctx.closePath(); ctx.fill(); ctx.restore();
  }
  for(let i=0;i<8;i++){
    const cx=(W/8)*i+W/16+Math.sin(gs.tick*0.012+i*1.3)*W*0.04;
    const cy=H*0.2+Math.cos(gs.tick*0.01+i)*H*0.08;
    const g2=ctx.createRadialGradient(cx,cy,0,cx,cy,W*0.06);
    g2.addColorStop(0,hc?'rgba(255,100,80,0.05)':'rgba(100,180,255,0.055)'); g2.addColorStop(1,'rgba(0,0,0,0)');
    ctx.fillStyle=g2; ctx.beginPath(); ctx.arc(cx,cy,W*0.06,0,Math.PI*2); ctx.fill();
  }
  const sh=sandH(), sg=ctx.createLinearGradient(0,H-sh,0,H);
  sg.addColorStop(0,'#c4a86a'); sg.addColorStop(1,'#8a6030');
  ctx.fillStyle=sg; ctx.beginPath(); ctx.moveTo(0,H-sh);
  for(let x=0;x<=W;x+=60) ctx.lineTo(x,H-sh+Math.sin(x*0.03+gs.tick*0.004)*(sh*0.12));
  ctx.lineTo(W,H); ctx.lineTo(0,H); ctx.closePath(); ctx.fill();
  ctx.fillStyle='rgba(154,128,72,0.55)';
  ctx.beginPath(); ctx.ellipse(W*0.15,H-sh*0.4,W*0.12,sh*0.35,-0.05,0,Math.PI*2); ctx.fill();
  ctx.beginPath(); ctx.ellipse(W*0.55,H-sh*0.35,W*0.1,sh*0.28,0.04,0,Math.PI*2); ctx.fill();
  ctx.beginPath(); ctx.ellipse(W*0.83,H-sh*0.38,W*0.09,sh*0.3,-0.03,0,Math.PI*2); ctx.fill();
}

// ── HUD update ────────────────────────────────────────────
function updateHUD() {
  const d=DIFF[gs.diff], phN=Math.max(0,Math.min(1,(gs.ph-7.0)/1.4));
  const fill=document.getElementById('ph-bar-fill');
  fill.style.width=(phN*100)+'%';
  fill.style.backgroundColor=`rgb(${lerp(0xe0,0x28,phN)|0},${lerp(0x28,0xd4,phN)|0},${lerp(0x28,0x64,phN)|0})`;
  document.getElementById('ph-value').textContent=`pH  ${gs.ph.toFixed(2)}`;
  document.getElementById('score-num').textContent=gs.score;
  document.getElementById('timer-val').textContent=`${Math.ceil(gs.timeLeft)}s`;
  document.getElementById('level-val').textContent=`Level ${gs.level}`;
  const badge=document.getElementById('diff-badge'); badge.textContent=`${d.emoji} ${d.label}`; badge.style.color=d.color;
  document.getElementById('lives-row').innerHTML=Array.from({length:gs.lives},()=>'<div class="life-dot"></div>').join('');
  document.getElementById('warning-bar').style.display=gs.ph<7.7?'block':'none';
}

// ── screen management ─────────────────────────────────────
const menuEl  = document.getElementById('menu-screen');
const endEl   = document.getElementById('end-screen');
const hudEl   = document.getElementById('hud');

function showMenu() {
  hudEl.style.display='none'; endEl.style.display='none'; menuEl.style.display='flex';
  syncCards();
}
function showHUD() {
  menuEl.style.display='none'; endEl.style.display='none'; hudEl.style.display='block';
}
function showEnd(win) {
  hudEl.style.display='none'; menuEl.style.display='none'; endEl.style.display='flex';
  const d=DIFF[gs.diff];
  const t=document.getElementById('end-title');
  t.textContent=win?'HATCHERY SAVED! 🦪':'LARVAE LOST 💀'; t.className='end-title '+(win?'win':'lose');
  const b=document.getElementById('end-badge'); b.textContent=`${d.emoji} ${d.label} MODE`; b.style.color=d.color;
  document.getElementById('end-sub').textContent=win?'You kept the bay water healthy enough for the larvae!':'The acidity dissolved the oyster larvae.';
  document.getElementById('end-score').textContent=`Score: ${gs.score}`;
  const rb=document.getElementById('retry-btn'); rb.style.color=d.color; rb.style.borderColor=d.color;
}
function syncCards() {
  document.querySelectorAll('.diff-card').forEach(c=>c.classList.toggle('selected',c.dataset.key===currentDiff));
  const d=DIFF[currentDiff], sb=document.getElementById('start-btn');
  sb.textContent=`▶  START ${d.label}`; sb.className=`btn-start btn-${currentDiff}`;
}

// ── events ────────────────────────────────────────────────
document.querySelectorAll('.diff-card').forEach(c=>c.addEventListener('click',()=>{ currentDiff=c.dataset.key; syncCards(); }));
document.getElementById('start-btn').addEventListener('click',()=>{ gs=newState(currentDiff); gs.state='play'; showHUD(); });
document.getElementById('retry-btn').addEventListener('click',()=>{ gs=newState(gs.diff); gs.state='play'; showHUD(); });
document.getElementById('menu-btn').addEventListener('click',()=>{ gs=newState(currentDiff); showMenu(); });

canvas.addEventListener('click', e=>{
  if(gs.state!=='play') return;
  const r=canvas.getBoundingClientRect(), mx=(e.clientX-r.left)*(W/r.width), my=(e.clientY-r.top)*(H/r.height);
  for(const b of gs.co2s){ if(b.active&&Math.hypot(b.x-mx,b.y-my)<b.r+6){ b.active=false; gs.score+=DIFF[gs.diff].scoreHit; gs.ph=Math.min(8.4,gs.ph+0.045); spawnParts(b.x,b.y,C.green,14); return; } }
  for(const s of gs.shells){ if(s.active&&Math.abs(s.x-mx)<W*0.025&&Math.abs(s.y-my)<H*0.02){ s.active=false; gs.score+=s.healthy?DIFF[gs.diff].scoreShell:2; spawnParts(s.x,s.y,C.yellow,8); return; } }
});

// ── helpers ───────────────────────────────────────────────
function lerp(a,b,t){return a+(b-a)*t;}

// ── loop ──────────────────────────────────────────────────
gs = newState('normal');
showMenu();
let last=performance.now();

function loop(now) {
  const dt=Math.min(now-last,50); last=now; gs.tick++;
  if(gs.state==='play'){
    const d=DIFF[gs.diff];
    gs.timeLeft-=dt/1000;
    gs.level=Math.max(1,Math.floor((d.time-gs.timeLeft)/Math.max(1,d.time/3))+1);
    gs.spawnT+=dt;
    if(gs.spawnT>=Math.max(d.spawnMin,d.spawnBase-gs.level*d.spawnLvl)){ gs.spawnT=0; gs.co2s.push(mkCO2()); }
    gs.shellT+=dt;
    if(gs.shellT>=d.shellRate){ gs.shellT=0; gs.shells.push(mkShell()); }
    gs.co2s.forEach(updCO2); gs.shells.forEach(updShell); gs.corals.forEach(updCoral); updParts();
    gs.co2s=gs.co2s.filter(b=>b.active); gs.shells=gs.shells.filter(s=>s.active);
    gs.ph=Math.min(d.ph,gs.ph+d.phRec*dt);
    updateHUD();
    if(gs.ph<=7.5){ gs.state='over'; showEnd(false); }
    else if(gs.timeLeft<=0){ const w=gs.ph>d.winPh; gs.state=w?'win':'over'; showEnd(w); }
  }
  drawBG();
  gs.corals.forEach(drwCoral); gs.shells.forEach(drwShell); gs.co2s.forEach(drwCO2); drwParts();
  requestAnimationFrame(loop);
}
requestAnimationFrame(loop);
</script>
</body>
</html>
