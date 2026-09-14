<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0, user-scalable=no">
<title>RALLY RIDER — 50+ Tracks</title>
<style>
* { margin: 0; padding: 0; box-sizing: border-box; }
body { font-family: 'Segoe UI', sans-serif; overflow: hidden; background: #1a1a2e; color: white; user-select: none; }
canvas { display: block; }

#hud { position: fixed; top: 20px; left: 20px; z-index: 10; }
#hud div { background: rgba(0,0,0,0.7); padding: 10px 20px; margin: 5px 0; border-radius: 8px; font-size: 18px; font-weight: bold; }
#minimap { position: fixed; bottom: 20px; right: 20px; width: 200px; height: 200px; background: rgba(0,0,0,0.8); border: 2px solid #ff6b35; border-radius: 8px; z-index: 10; }

#controls { position: fixed; bottom: 20px; left: 20px; z-index: 10; display: flex; gap: 10px; flex-wrap: wrap; max-width: 300px; }
.ctrl-btn { background: rgba(255,107,53,0.8); border: none; color: white; padding: 15px 20px; border-radius: 8px; font-size: 14px; font-weight: bold; cursor: pointer; min-width: 80px; }
.ctrl-btn:active { background: rgba(255,107,53,1); transform: scale(0.95); }

#trackSelect { position: fixed; inset: 0; background: rgba(0,0,0,0.95); z-index: 100; display: flex; flex-direction: column; align-items: center; justify-content: center; padding: 40px; overflow-y: auto; }
#trackSelect h1 { font-size: 48px; margin-bottom: 10px; color: #ff6b35; }
#trackSelect p { font-size: 18px; margin-bottom: 30px; opacity: 0.8; }

.track-grid { display: grid; grid-template-columns: repeat(auto-fill, minmax(280px, 1fr)); gap: 20px; width: 100%; max-width: 1400px; }
.track-card { background: linear-gradient(135deg, rgba(255,107,53,0.1), rgba(0,0,0,0.3)); border: 2px solid rgba(255,107,53,0.3); border-radius: 12px; padding: 20px; cursor: pointer; transition: all 0.3s; position: relative; overflow: hidden; }
.track-card:hover { border-color: #ff6b35; transform: translateY(-4px); box-shadow: 0 10px 30px rgba(255,107,53,0.3); }
.track-card canvas { width: 100%; height: 150px; border-radius: 8px; margin-bottom: 15px; background: #2a2a3e; }
.track-card h3 { font-size: 20px; margin-bottom: 8px; }
.track-card .tags { display: flex; gap: 6px; flex-wrap: wrap; margin-top: 10px; }
.tag { font-size: 11px; padding: 3px 8px; background: rgba(255,107,53,0.3); border-radius: 4px; }

#preview { position: fixed; inset: 0; background: rgba(0,0,0,0.95); z-index: 200; display: none; flex-direction: column; align-items: center; justify-content: center; padding: 40px; }
#preview.active { display: flex; }
#preview h2 { font-size: 42px; margin-bottom: 20px; }
#previewCanvas { width: 90%; max-width: 900px; height: 600px; background: #1a1a2e; border: 2px solid #ff6b35; border-radius: 12px; }
#preview button { margin-top: 30px; padding: 18px 40px; font-size: 20px; background: #ff6b35; border: none; color: white; border-radius: 8px; cursor: pointer; font-weight: bold; }

#gameOver { position: fixed; inset: 0; background: rgba(0,0,0,0.9); z-index: 150; display: none; align-items: center; justify-content: center; }
#gameOver.active { display: flex; }
#gameOver div { background: rgba(26,26,46,0.95); padding: 40px; border-radius: 12px; text-align: center; border: 2px solid #ff6b35; }
#gameOver h2 { font-size: 36px; margin-bottom: 20px; }
#gameOver button { margin: 10px; padding: 15px 30px; font-size: 16px; background: #ff6b35; border: none; color: white; border-radius: 8px; cursor: pointer; }

.gear-display { position: fixed; top: 20px; right: 20px; background: rgba(0,0,0,0.8); padding: 15px 25px; border-radius: 8px; font-size: 24px; font-weight: bold; z-index: 10; border: 2px solid #ff6b35; }
.rpm-bar { width: 200px; height: 20px; background: rgba(255,255,255,0.1); border-radius: 10px; margin-top: 10px; overflow: hidden; }
.rpm-fill { height: 100%; background: linear-gradient(90deg, #00ff00, #ffff00, #ff0000); transition: width 0.1s; }

.stalled { animation: shake 0.5s; }
@keyframes shake { 0%, 100% { transform: translateX(0); } 25% { transform: translateX(-5px); } 75% { transform: translateX(5px); } }

.touch-controls { position: fixed; bottom: 0; left: 0; right: 0; padding: 20px; z-index: 50; display: none; }
.touch-controls.active { display: flex; justify-content: space-between; }
.touch-pad { display: flex; gap: 10px; }
.touch-btn { width: 70px; height: 70px; background: rgba(255,107,53,0.6); border: 2px solid rgba(255,107,53,0.8); border-radius: 50%; display: flex; align-items: center; justify-content: center; font-size: 24px; font-weight: bold; }
</style>
</head>
<body>
<canvas id="gameCanvas"></canvas>

<div id="hud">
<div>SPEED: <span id="speed">0</span> km/h</div>
<div>CHECKPOINT: <span id="checkpoint">0</span> / <span id="totalCP">0</span></div>
<div>DISTANCE: <span id="distance">0</span> m</div>
</div>

<div class="gear-display">
GEAR: <span id="gear">N</span>
<div class="rpm-bar"><div class="rpm-fill" id="rpmFill"></div></div>
</div>

<canvas id="minimap"></canvas>

<div id="controls">
<button class="ctrl-btn" onclick="recoverCar()">RESET [R]</button>
<button class="ctrl-btn" onclick="restartTrack()">RESTART [F]</button>
<button class="ctrl-btn" onclick="toggleManual()">MANUAL [M]</button>
<button class="ctrl-btn" onclick="showTrackSelect()">TRACKS [T]</button>
</div>

<div class="touch-controls" id="touchControls">
<div class="touch-pad">
<div class="touch-btn" ontouchstart="touchInput.left=true" ontouchend="touchInput.left=false">◀</div>
<div class="touch-btn" ontouchstart="touchInput.right=true" ontouchend="touchInput.right=false">▶</div>
</div>
<div class="touch-pad">
<div class="touch-btn" ontouchstart="touchInput.up=true" ontouchend="touchInput.up=false">▲</div>
<div class="touch-btn" ontouchstart="touchInput.down=true" ontouchend="touchInput.down=false">▼</div>
</div>
</div>

<div id="trackSelect">
<h1>🏁 RALLY RIDER</h1>
<p>Choose from 50+ tracks across different environments</p>
<div class="track-grid" id="trackGrid"></div>
</div>

<div id="preview">
<h2 id="previewTitle">Track Preview</h2>
<canvas id="previewCanvas"></canvas>
<p id="previewDesc" style="margin-top:20px; font-size:16px; max-width:700px; text-align:center;"></p>
<button onclick="startRace()">START RACE</button>
<button onclick="closePreview()" style="background:#444; margin-left:10px;">BACK</button>
</div>

<div id="gameOver">
<div>
<h2 id="gameOverTitle">Track Complete!</h2>
<p id="gameOverText"></p>
<button onclick="restartTrack()">RETRY</button>
<button onclick="showTrackSelect()">SELECT TRACK</button>
</div>
</div>

<script>
// ═══════════════════════════════════════════════════════════════
// TRACK DEFINITIONS - 50+ tracks with different themes
// ═══════════════════════════════════════════════════════════════
const TRACKS = [];

function generateTrack(type, theme, difficulty, length) {
    const points = [];
    let x = 0, y = 0, angle = 0;
    for(let i = 0; i < length; i++) {
        const turn = (Math.random() - 0.5) * (difficulty * 0.3);
        angle += turn;
        const dist = 80 + Math.random() * 40;
        x += Math.cos(angle) * dist;
        y += Math.sin(angle) * dist;
        points.push({x, y, type, angle});
    }
    return points;
}

const themes = [
    {name: 'Forest', color: '#2d5016', tags: ['Trees', 'Natural']},
    {name: 'Desert', color: '#c19a3e', tags: ['Sand', 'Dunes']},
    {name: 'Snow', color: '#a8d5e8', tags: ['Ice', 'Cold']},
    {name: 'Volcano', color: '#8b2500', tags: ['Lava', 'Rocks']},
    {name: 'Coast', color: '#1e7a8f', tags: ['Water', 'Cliffs']},
    {name: 'Mountain', color: '#5d5d5d', tags: ['Steep', 'Winding']},
    {name: 'City', color: '#4a4a4a', tags: ['Urban', 'Buildings']},
    {name: 'F1 Circuit', color: '#1a1a1a', tags: ['Professional', 'Fast']}
];

const trackTypes = ['sprint', 'circuit', 'rally', 'drift', 'endurance'];

for(let i = 0; i < 50; i++) {
    const theme = themes[i % themes.length];
    const type = trackTypes[i % trackTypes.length];
    const difficulty = 1 + (i % 5);
    const length = 15 + Math.floor(i / 5);
    TRACKS.push({
        id: i,
        name: `${theme.name} ${type.charAt(0).toUpperCase() + type.slice(1)} #${i + 1}`,
        theme: theme.name,
        themeColor: theme.color,
        type, difficulty,
        points: generateTrack(type, theme, difficulty, length),
        tags: theme.tags.concat([type, `Difficulty ${difficulty}`]),
        description: `A ${difficulty}-star ${type} track through ${theme.name.toLowerCase()} terrain with ${length} sections.`
    });
}

// ═══════════════════════════════════════════════════════════════
// GAME STATE
// ═══════════════════════════════════════════════════════════════
const canvas = document.getElementById('gameCanvas');
const ctx = canvas.getContext('2d');
const minimapCanvas = document.getElementById('minimap');
const minimapCtx = minimapCanvas.getContext('2d');

let gameState = 'menu';
let currentTrack = null;
let camera = {x: 0, y: 0, zoom: 0.5};

const car = {
    x: 0, y: 0, angle: 0, speed: 0, angularVel: 0,
    gear: 0, rpm: 0, maxSpeed: 200, acceleration: 0,
    width: 30, height: 50, checkpoints: [], currentCP: 0,
    manual: false, stalled: false, distance: 0
};

const keys = {};
const touchInput = {up: false, down: false, left: false, right: false};

function resizeCanvas() {
    canvas.width = window.innerWidth;
    canvas.height = window.innerHeight;
    minimapCanvas.width = 200;
    minimapCanvas.height = 200;
}
window.addEventListener('resize', resizeCanvas);
resizeCanvas();
showTrackSelect();

// ═══════════════════════════════════════════════════════════════
// TRACK SELECT UI
// ═══════════════════════════════════════════════════════════════
function showTrackSelect() {
    gameState = 'menu';
    document.getElementById('trackSelect').style.display = 'flex';
    document.getElementById('preview').classList.remove('active');
    document.getElementById('gameOver').classList.remove('active');
    const grid = document.getElementById('trackGrid');
    grid.innerHTML = '';
    TRACKS.forEach((track, idx) => {
        const card = document.createElement('div');
        card.className = 'track-card';
        card.onclick = () => showPreview(idx);
        const cv = document.createElement('canvas');
        cv.width = 280; cv.height = 150;
        drawTrackPreview(cv, track);
        card.appendChild(cv);
        const info = document.createElement('div');
        info.innerHTML = `<h3>${track.name}</h3><p style="font-size:13px;opacity:0.7;">${track.description}</p>
        <div class="tags">${track.tags.map(t => `<span class="tag">${t}</span>`).join('')}</div>`;
        card.appendChild(info);
        grid.appendChild(card);
    });
}

function drawTrackPreview(canvas, track) {
    const ctx = canvas.getContext('2d');
    ctx.fillStyle = '#2a2a3e'; ctx.fillRect(0, 0, canvas.width, canvas.height);
    if(track.points.length === 0) return;
    let minX = Infinity, maxX = -Infinity, minY = Infinity, maxY = -Infinity;
    track.points.forEach(p => { minX = Math.min(minX, p.x); maxX = Math.max(maxX, p.x); minY = Math.min(minY, p.y); maxY = Math.max(maxY, p.y); });
    const rangeX = maxX - minX, rangeY = maxY - minY;
    const scale = Math.min(canvas.width / rangeX, canvas.height / rangeY) * 0.8;
    const offsetX = (canvas.width - rangeX * scale) / 2, offsetY = (canvas.height - rangeY * scale) / 2;
    ctx.strokeStyle = track.themeColor; ctx.lineWidth = 3; ctx.lineCap = 'round'; ctx.lineJoin = 'round';
    ctx.beginPath();
    track.points.forEach((p, i) => {
        const x = (p.x - minX) * scale + offsetX, y = (p.y - minY) * scale + offsetY;
        if(i === 0) ctx.moveTo(x, y); else ctx.lineTo(x, y);
    });
    ctx.stroke();
    track.points.forEach((p, i) => {
        if(i % 3 === 0) {
            const x = (p.x - minX) * scale + offsetX, y = (p.y - minY) * scale + offsetY;
            ctx.fillStyle = '#ff6b35'; ctx.beginPath(); ctx.arc(x, y, 3, 0, Math.PI * 2); ctx.fill();
        }
    });
}

function showPreview(idx) {
    previewTrackIdx = idx; currentTrack = TRACKS[idx];
    document.getElementById('preview').classList.add('active');
    document.getElementById('previewTitle').textContent = currentTrack.name;
    document.getElementById('previewDesc').textContent = currentTrack.description;
    const cv = document.getElementById('previewCanvas');
    cv.width = cv.offsetWidth; cv.height = cv.offsetHeight;
    drawLargePreview(cv, currentTrack);
}

function drawLargePreview(canvas, track) {
    const ctx = canvas.getContext('2d');
    ctx.fillStyle = '#1a1a2e'; ctx.fillRect(0, 0, canvas.width, canvas.height);
    if(track.points.length === 0) return;
    let minX = Infinity, maxX = -Infinity, minY = Infinity, maxY = -Infinity;
    track.points.forEach(p => { minX = Math.min(minX, p.x); maxX = Math.max(maxX, p.x); minY = Math.min(minY, p.y); maxY = Math.max(maxY, p.y); });
    const rangeX = maxX - minX, rangeY = maxY - minY;
    const scale = Math.min(canvas.width / rangeX, canvas.height / rangeY) * 0.85;
    const offsetX = (canvas.width - rangeX * scale) / 2, offsetY = (canvas.height - rangeY * scale) / 2;
    ctx.shadowBlur = 20; ctx.shadowColor = track.themeColor;
    ctx.strokeStyle = track.themeColor; ctx.lineWidth = 8; ctx.lineCap = 'round'; ctx.lineJoin = 'round';
    ctx.beginPath();
    track.points.forEach((p, i) => {
        const x = (p.x - minX) * scale + offsetX, y = (p.y - minY) * scale + offsetY;
        if(i === 0) ctx.moveTo(x, y); else ctx.lineTo(x, y);
    });
    ctx.stroke(); ctx.shadowBlur = 0;
    track.points.forEach((p, i) => {
        if(i % 3 === 0) {
            const x = (p.x - minX) * scale + offsetX, y = (p.y - minY) * scale + offsetY;
            ctx.fillStyle = i === 0 ? '#00ff00' : '#ff6b35';
            ctx.strokeStyle = 'white'; ctx.lineWidth = 2;
            ctx.beginPath(); ctx.arc(x, y, 6, 0, Math.PI * 2); ctx.fill(); ctx.stroke();
            ctx.fillStyle = 'white'; ctx.font = '12px sans-serif'; ctx.textAlign = 'center';
            ctx.fillText(`CP ${i / 3 + 1}`, x, y - 12);
        }
    });
    const start = track.points[0];
    const sx = (start.x - minX) * scale + offsetX, sy = (start.y - minY) * scale + offsetY;
    ctx.fillStyle = '#00ff00'; ctx.font = 'bold 14px sans-serif';
    ctx.fillText('START', sx, sy + 30);
}

function closePreview() { document.getElementById('preview').classList.remove('active'); showTrackSelect(); }

// ═══════════════════════════════════════════════════════════════
// GAME LOGIC
// ═══════════════════════════════════════════════════════════════
let previewTrackIdx = 0;

function startRace() {
    gameState = 'playing';
    document.getElementById('preview').classList.remove('active');
    document.getElementById('trackSelect').style.display = 'none';
    const start = currentTrack.points[0];
    car.x = start.x; car.y = start.y; car.angle = start.angle || 0;
    car.speed = 0; car.gear = 1; car.rpm = 0; car.currentCP = 0; car.distance = 0;
    car.checkpoints = currentTrack.points.filter((p, i) => i % 3 === 0);
    document.getElementById('totalCP').textContent = car.checkpoints.length;
    document.getElementById('checkpoint').textContent = 0;
    camera.x = car.x; camera.y = car.y;
    requestAnimationFrame(gameLoop);
}

function gameLoop() {
    if(gameState !== 'playing') return;
    updateCar(); render(); renderMinimap(); updateHUD();
    requestAnimationFrame(gameLoop);
}

// ═══════════════════════════════════════════════════════════════
// CAR PHYSICS - JIGGLY & WEIGHTY
// ═══════════════════════════════════════════════════════════════
function updateCar() {
    const input = {
        up: keys['w'] || keys['arrowup'] || touchInput.up,
        down: keys['s'] || keys['arrowdown'] || touchInput.down,
        left: keys['a'] || keys['arrowleft'] || touchInput.left,
        right: keys['d'] || keys['arrowright'] || touchInput.right,
        handbrake: keys[' ']
    };
    if(car.manual) handleManualTransmission(input); else handleAutoTransmission(input);
    const friction = 0.98;
    let accel = 0;
    if(!car.stalled) {
        const gearRatio = [0, 3.5, 2.5, 2.0, 1.6, 1.3][Math.abs(car.gear)];
        const rpmFactor = Math.max(0, (car.rpm - 1000) / 7000);
        if(input.up && car.gear > 0) accel = rpmFactor * gearRatio * 0.8;
        else if(input.down && car.gear < 0) accel = -rpmFactor * 3.0 * 0.6;
        else if(input.down && car.gear > 0) accel = -0.5;
    }
    car.acceleration = accel;
    car.speed += accel * 0.016;
    car.speed *= friction;
    const maxSpeedForGear = [0, 40, 80, 120, 160, 200][Math.abs(car.gear)] || 200;
    car.speed = Math.max(-30, Math.min(car.speed, maxSpeedForGear));
    let turnRate = 0;
    if(Math.abs(car.speed) > 0.5) {
        const speedFactor = Math.min(1, Math.abs(car.speed) / 50);
        turnRate = (input.right ? 1 : 0) - (input.left ? 1 : 0);
        turnRate *= speedFactor * 0.04;
        if(input.handbrake) { turnRate *= 1.8; car.speed *= 0.96; }
    }
    car.angularVel += turnRate;
    car.angularVel *= 0.85;
    car.angle += car.angularVel;
    const moveX = Math.cos(car.angle) * car.speed, moveY = Math.sin(car.angle) * car.speed;
    car.x += moveX; car.y += moveY;
    car.distance += Math.abs(car.speed) * 0.1;
    const targetRPM = Math.abs(car.speed) * 80 * (Math.abs(car.gear) || 1);
    car.rpm += (targetRPM - car.rpm) * 0.1;
    car.rpm = Math.max(0, Math.min(8000, car.rpm));
    if(car.currentCP < car.checkpoints.length) {
        const cp = car.checkpoints[car.currentCP];
        const dx = cp.x - car.x, dy = cp.y - car.y;
        if(Math.sqrt(dx * dx + dy * dy) < 60) {
            car.currentCP++;
            document.getElementById('checkpoint').textContent = car.currentCP;
            if(car.currentCP >= car.checkpoints.length) finishTrack();
        }
    }
    camera.x += (car.x - camera.x) * 0.1;
    camera.y += (car.y - camera.y) * 0.1;
    camera.zoom = 0.5 + Math.abs(car.speed) * 0.002;
}

function handleManualTransmission(input) {
    if(keys['q']) { if(car.gear > -1) { car.gear = Math.max(-1, car.gear - 1); keys['q'] = false; checkStall(); } }
    if(keys['e']) { if(car.gear < 5) { car.gear = Math.min(5, car.gear + 1); keys['e'] = false; checkStall(); } }
}

function handleAutoTransmission(input) {
    if(car.speed > 160) car.gear = 5;
    else if(car.speed > 120) car.gear = 4;
    else if(car.speed > 80) car.gear = 3;
    else if(car.speed > 40) car.gear = 2;
    else if(car.speed > 5) car.gear = 1;
    else if(car.speed < -5) car.gear = -1;
    else car.gear = 0;
}

function checkStall() {
    const speed = Math.abs(car.speed), gear = Math.abs(car.gear);
    if(gear > 0) {
        const minSpeed = gear * 15;
        if(speed < minSpeed && car.rpm < 2000) {
            car.stalled = true;
            document.getElementById('gear').parentElement.classList.add('stalled');
            setTimeout(() => { car.stalled = false; document.getElementById('gear').parentElement.classList.remove('stalled'); }, 500);
        }
    }
}

// ═══════════════════════════════════════════════════════════════
// RENDERING
// ═══════════════════════════════════════════════════════════════
function render() {
    ctx.fillStyle = '#1a1a2e'; ctx.fillRect(0, 0, canvas.width, canvas.height);
    ctx.save();
    ctx.translate(canvas.width / 2, canvas.height / 2);
    ctx.scale(camera.zoom, camera.zoom);
    ctx.translate(-camera.x, -camera.y);
    drawTrack(); drawCar();
    ctx.restore();
}

function drawTrack() {
    if(!currentTrack) return;
    ctx.strokeStyle = currentTrack.themeColor; ctx.lineWidth = 40;
    ctx.lineCap = 'round'; ctx.lineJoin = 'round';
    ctx.shadowBlur = 15; ctx.shadowColor = currentTrack.themeColor;
    ctx.beginPath();
    currentTrack.points.forEach((p, i) => { if(i === 0) ctx.moveTo(p.x, p.y); else ctx.lineTo(p.x, p.y); });
    ctx.stroke(); ctx.shadowBlur = 0;
    ctx.strokeStyle = 'rgba(255,255,255,0.3)'; ctx.lineWidth = 2; ctx.setLineDash([20, 20]);
    ctx.beginPath();
    currentTrack.points.forEach((p, i) => { if(i === 0) ctx.moveTo(p.x, p.y); else ctx.lineTo(p.x, p.y); });
    ctx.stroke(); ctx.setLineDash([]);
    car.checkpoints.forEach((cp, i) => {
        const reached = i < car.currentCP, current = i === car.currentCP;
        ctx.fillStyle = reached ? '#00ff00' : (current ? '#ffff00' : '#ff6b35');
        ctx.strokeStyle = 'white'; ctx.lineWidth = 3;
        ctx.beginPath(); ctx.arc(cp.x, cp.y, 25, 0, Math.PI * 2); ctx.fill(); ctx.stroke();
        ctx.fillStyle = 'white'; ctx.font = 'bold 16px sans-serif'; ctx.textAlign = 'center'; ctx.textBaseline = 'middle';
        ctx.fillText(i + 1, cp.x, cp.y);
    });
}

function drawCar() {
    ctx.save(); ctx.translate(car.x, car.y); ctx.rotate(car.angle);
    ctx.fillStyle = car.stalled ? '#ff0000' : '#ff6b35';
    ctx.strokeStyle = '#000'; ctx.lineWidth = 2;
    ctx.fillRect(-car.width / 2, -car.height / 2, car.width, car.height);
    ctx.strokeRect(-car.width / 2, -car.height / 2, car.width, car.height);
    ctx.fillStyle = '#87ceeb';
    ctx.fillRect(-car.width / 2 + 4, -car.height / 2 + 10, car.width - 8, 12);
    ctx.fillRect(-car.width / 2 + 4, car.height / 2 - 22, car.width - 8, 12);
    ctx.fillStyle = '#000';
    ctx.fillRect(-car.width / 2 - 3, -car.height / 2 + 5, 4, 15);
    ctx.fillRect(car.width / 2 - 1, -car.height / 2 + 5, 4, 15);
    ctx.fillRect(-car.width / 2 - 3, car.height / 2 - 20, 4, 15);
    ctx.fillRect(car.width / 2 - 1, car.height / 2 - 20, 4, 15);
    ctx.fillStyle = '#ffff00';
    ctx.beginPath(); ctx.moveTo(0, -car.height / 2 - 5);
    ctx.lineTo(-5, -car.height / 2 + 5); ctx.lineTo(5, -car.height / 2 + 5);
    ctx.closePath(); ctx.fill();
    ctx.restore();
}

function renderMinimap() {
    if(!currentTrack) return;
    minimapCtx.fillStyle = 'rgba(0,0,0,0.8)'; minimapCtx.fillRect(0, 0, 200, 200);
    let minX = Infinity, maxX = -Infinity, minY = Infinity, maxY = -Infinity;
    currentTrack.points.forEach(p => { minX = Math.min(minX, p.x); maxX = Math.max(maxX, p.x); minY = Math.min(minY, p.y); maxY = Math.max(maxY, p.y); });
    const rangeX = maxX - minX, rangeY = maxY - minY;
    const scale = Math.min(180 / rangeX, 180 / rangeY);
    const offsetX = (200 - rangeX * scale) / 2, offsetY = (200 - rangeY * scale) / 2;
    minimapCtx.strokeStyle = currentTrack.themeColor; minimapCtx.lineWidth = 3;
    minimapCtx.beginPath();
    currentTrack.points.forEach((p, i) => {
        const x = (p.x - minX) * scale + offsetX, y = (p.y - minY) * scale + offsetY;
        if(i === 0) minimapCtx.moveTo(x, y); else minimapCtx.lineTo(x, y);
    });
    minimapCtx.stroke();
    const cx = (car.x - minX) * scale + offsetX, cy = (car.y - minY) * scale + offsetY;
    minimapCtx.fillStyle = '#ff6b35';
    minimapCtx.beginPath(); minimapCtx.arc(cx, cy, 5, 0, Math.PI * 2); minimapCtx.fill();
}

function updateHUD() {
    document.getElementById('speed').textContent = Math.abs(Math.round(car.speed));
    document.getElementById('distance').textContent = Math.round(car.distance);
    document.getElementById('gear').textContent = car.gear === 0 ? 'N' : (car.gear < 0 ? 'R' : car.gear);
    document.getElementById('rpmFill').style.width = (car.rpm / 8000 * 100) + '%';
}

function recoverCar() {
    const targetCP = Math.max(0, car.currentCP - 1);
    const cp = car.checkpoints[targetCP] || currentTrack.points[0];
    car.x = cp.x; car.y = cp.y; car.angle = cp.angle || 0;
    car.speed = 0; car.rpm = 0; car.stalled = false;
}

function restartTrack() { startRace(); }
function toggleManual() { car.manual = !car.manual; }

function finishTrack() {
    gameState = 'menu';
    document.getElementById('gameOver').classList.add('active');
    document.getElementById('gameOverTitle').textContent = '🏁 Track Complete!';
    document.getElementById('gameOverText').textContent = `You completed ${currentTrack.name}! Distance: ${Math.round(car.distance)}m`;
}

window.addEventListener('keydown', e => {
    keys[e.key.toLowerCase()] = true;
    if(e.key.toLowerCase() === 'r') recoverCar();
    if(e.key.toLowerCase() === 'f') restartTrack();
    if(e.key.toLowerCase() === 'm') toggleManual();
    if(e.key.toLowerCase() === 't') showTrackSelect();
    if(e.key === 'Escape' && gameState === 'playing') showTrackSelect();
});
window.addEventListener('keyup', e => { keys[e.key.toLowerCase()] = false; });
if('ontouchstart' in window) document.getElementById('touchControls').classList.add('active');
window.addEventListener('contextmenu', e => e.preventDefault());
</script>
</body>
</html>
