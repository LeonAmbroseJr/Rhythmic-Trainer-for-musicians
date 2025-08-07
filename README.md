python -m venv venv
source venv/bin/activate  # or venv\Scripts\activate on Windows
pip install fastapi uvicorn[standard] aiofiles



-------------------------------------------------
3. Backend (main.py)
------------------------------------------------
```python
from fastapi import FastAPI, WebSocket, WebSocketDisconnect
from fastapi.staticfiles import StaticFiles
import json, asyncio, pathlib, random

app = FastAPI(title="Pow-wow Rhythm Trainer")
app.mount("/static", StaticFiles(directory="static"), name="static")
app.mount("/samples", StaticFiles(directory="samples"), name="samples")

# ----------------------
# Exercise definitions
# ----------------------
EXERCISES = [
    {
        "id": "straight4",
        "bpm": 100,
        "pattern": "K---S---K---S---",  # K=Kick, S=Snare, -=rest
        "time_signature": "4/4",
        "description": "Four-on-the-floor pow-wow basic"
    },
    {
        "id": "honor_beats",
        "bpm": 110,
        "pattern": "K-S-K-S-K-S-KS--",
        "time_signature": "4/4",
        "description": "Northern style honor beats"
    },
    {
        "id": "round_dance_6",
        "bpm": 120,
        "pattern": "K-S-KK-S-K-S-",
        "time_signature": "6/8",
        "description": "Round dance 6/8 feel"
    }
]

@app.websocket("/ws")
async def websocket_endpoint(ws: WebSocket):
    await ws.accept()
    try:
        while True:
            data = await ws.receive_text()
            msg = json.loads(data)
            if msg["type"] == "next":
                ex = random.choice(EXERCISES)
                await ws.send_json({"type":"exercise", "payload": ex})
            elif msg["type"] == "ping":
                await ws.send_json({"type":"pong"})
            else:
                await ws.send_json({"type":"error", "text":"unknown"})
    except WebSocketDisconnect:
        pass

# Optional: REST endpoint if you want to fetch list
@app.get("/api/exercises")
def list_exercises():
    return EXERCISES

uvicorn main:app --reload --host 0.0.0.0 --port 8080


<!doctype html>
<html lang="en">
<head>
  <meta charset="utf-8"/>
  <title>Pow-wow Rhythm Trainer</title>
  <link rel="stylesheet" href="style.css"/>
</head>
<body>
  <h1>Pow-wow Rhythm Trainer</h1>

  <div id="controls">
    <button id="playBtn">▶ Start</button>
    <button id="nextBtn">Next Pattern</button>
    <label>BPM <input type="range" id="bpmSlider" min="60" max="180" value="100"/></label>
    <span id="bpmLabel">100</span>
  </div>

  <div id="patternBox">
    <pre id="pattern"></pre>
    <p id="description"></p>
  </div>

  <script src="app.js"></script>
</body>
</html>


/* Browser side metronome using WebAudio clock */
let audioCtx, schedulerId;
let lookahead = 25;        // ms
let scheduleAhead = 0.1;   // seconds
let nextNoteTime = 0;
let currentPattern = "";
let noteIndex = 0;
let bpm = 100;
let isPlaying = false;

const kick = new Audio("/samples/kick.wav");
const snare = new Audio("/samples/snare.wav");
const shaker = new Audio("/samples/shaker.wav");

kick.preload = snare.preload = shaker.preload = "auto";

const playBtn = document.getElementById("playBtn");
const nextBtn = document.getElementById("nextBtn");
const bpmSlider = document.getElementById("bpmSlider");
const bpmLabel = document.getElementById("bpmLabel");
const patternEl = document.getElementById("pattern");
const descEl = document.getElementById("description");

// -------------------------
// WebSocket to backend
// -------------------------
const ws = new WebSocket(`ws://${location.host}/ws`);
ws.onmessage = (e) => {
  const msg = JSON.parse(e.data);
  if (msg.type === "exercise") loadExercise(msg.payload);
};

function loadExercise(ex) {
  bpm = ex.bpm;
  bpmSlider.value = bpm;
  bpmLabel.textContent = bpm;
  currentPattern = ex.pattern;
  patternEl.textContent = currentPattern;
  descEl.textContent = ex.description;
  if (isPlaying) stop();
}

// -------------------------
// Metronome engine
// -------------------------
function nextNote() {
  const secondsPerBeat = 60.0 / bpm;
  nextNoteTime += secondsPerBeat;
  noteIndex = (noteIndex + 1) % currentPattern.length;
}

function scheduleNote(time) {
  const sym = currentPattern[noteIndex];
  if (sym === "K") kick.cloneNode().start(time);
  else if (sym === "S") snare.cloneNode().start(time);
  else if (sym === "-") {} // rest
}

function scheduler() {
  while (nextNoteTime < audioCtx.currentTime + scheduleAhead) {
    scheduleNote(nextNoteTime);
    nextNote();
  }
  schedulerId = setTimeout(scheduler, lookahead);
}

function start() {
  if (!audioCtx) audioCtx = new (window.AudioContext || window.webkitAudioContext)();
  nextNoteTime = audioCtx.currentTime;
  noteIndex = 0;
  isPlaying = true;
  playBtn.textContent = "⏸ Pause";
  scheduler();
}

function stop() {
  isPlaying = false;
  playBtn.textContent = "▶ Start";
  clearTimeout(schedulerId);
}

playBtn.onclick = () => isPlaying ? stop() : start();
nextBtn.onclick = () => ws.send(JSON.stringify({type:"next"}));

bpmSlider.oninput = () => {
  bpm = +bpmSlider.value;
  bpmLabel.textContent = bpm;
};

// Ask for first exercise on load
ws.onopen = () => ws.send(JSON.stringify({type:"next"}));


body {
  font-family: sans-serif;
  background: #111;
  color: #eee;
  text-align: center;
  padding: 2rem;
}
button, input {
  font-size: 1.2rem;
  margin: 0.5rem;
}
#patternBox {
  margin-top: 2rem;
  font-size: 1.8rem;
  letter-spacing: 0.6rem;
}


python -m http.server --directory static 8080   # or use uvicorn above


# Python
__pycache__/
*.pyc
*.pyo
*.pyd
venv/
.env

# IDE
.vscode/
.idea/

# OS
.DS_Store
Thumbs.db

# Build / packaging
*.egg-info/
dist/
build/
*.whl

# Logs
*.log


touch .gitignore

git add .gitignore
git commit -m "ignore more build artifacts"

git rm --cached <file-or-folder>
git commit -m "stop tracking now-ignored file"

[
  {
    "id": "southern_basic",
    "style": "Southern",
    "bpm": 92,
    "time_signature": "4/4",
    "pattern": "K---S-K-S--K-S---",
    "description": "Southern host drum basic"
  },
  {
    "id": "northern_heartbeat",
    "style": "Northern",
    "bpm": 118,
    "time_signature": "4/4",
    "pattern": "K-S-K-K-S-K-S-KS--",
    "description": "Northern heartbeat honor beats"
  },
  {
    "id": "grass_dance",
    "style": "Grass Dance",
    "bpm": 132,
    "time_signature": "2/4",
    "pattern": "KSSKSSKSSKSS",
    "description": "Fast double-time feel"
  },
  {
    "id": "fancy_dance",
    "style": "Fancy Dance",
    "bpm": 144,
    "time_signature": "4/4",
    "pattern": "K-K-S-K-S-K-K-SKS-",
    "description": "Fancy dance trick beats"
  },
  {
    "id": "jingle_dress",
    "style": "Jingle Dress",
    "bpm": 105,
    "time_signature": "6/8",
    "pattern": "K-S-KK-S-K-S-",
    "description": "6/8 round-dance feel"
  }
]


import json, pathlib
PATTERNS = json.loads(pathlib.Path("patterns.json").read_text())
# … in websocket handler …
    if msg["type"] == "next":
        style = msg.get("style")          # optional filter
        pool = [p for p in PATTERNS if style is None or p["style"]==style]
        ex = random.choice(pool)
        await ws.send_json({"type":"exercise", "payload": ex})


<select id="styleFilter">
  <option value="">All</option>
  <option>Southern</option>
  <option>Northern</option>
  <option>Grass Dance</option>
  <option>Fancy Dance</option>
  <option>Jingle Dress</option>
</select>


styleFilter.onchange = () =>
  ws.send(JSON.stringify({type:"next", style: styleFilter.value}));

<canvas id="timeline" width="600" height="120"></canvas>

const canvas = document.getElementById("timeline");
const ctx = canvas.getContext("2d");
const CELL_W = 40, CELL_H = 40;

function drawGrid() {
  ctx.clearRect(0,0,canvas.width,canvas.height);
  ctx.strokeStyle = "#555";
  for (let i=0;i<currentPattern.length;i++){
    ctx.strokeRect(i*CELL_W, 0, CELL_W, CELL_H);
    const sym = currentPattern[i];
    if (sym==="K") { ctx.fillStyle="red";  ctx.fillRect(i*CELL_W,0,CELL_W,CELL_H);}
    if (sym==="S") { ctx.fillStyle="blue"; ctx.fillRect(i*CELL_W,0,CELL_W,CELL_H);}
  }
}


canvas.addEventListener("click", e => {
  const col = Math.floor(e.offsetX/CELL_W);
  const hit = currentPattern[col] !== "-";
  ctx.fillStyle = hit ? "#0f0" : "#f00";
  ctx.fillRect(col*CELL_W, CELL_H, CELL_W, CELL_H/2);
});


<label>Humanize ±<input type="range" id="human" min="0" max="20" value="0"> ms</label>


const human = +document.getElementById("human").value;
const jitter = (Math.random()-0.5)*2*human/1000; // seconds
scheduleNote(nextNoteTime + jitter);

const swingSlider = document.getElementById("swing"); // 0-100
function nextNote() {
  const secondsPerBeat = 60.0/bpm;
  const swingRatio = swingSlider.value/100; // 0.5 = triplet
  const delay = noteIndex%2 ? secondsPerBeat*swingRatio : secondsPerBeat*(2-swingRatio);
  nextNoteTime += delay;
  noteIndex = (noteIndex+1)%currentPattern.length;
}


from sqlmodel import SQLModel, Field
class User(SQLModel, table=True):
    id: int | None = Field(default=None, primary_key=True)
    username: str
class Progress(SQLModel, table=True):
    id: int | None = Field(default=None, primary_key=True)
    user_id: int
    pattern_id: str
    tempo: int
    score: float
    date: str

npm init -y
npm i @capacitor/core @capacitor/cli @capacitor/android @capacitor/ios
npx cap init "RhythmTrainer" com.yourorg.rhythm --web-dir static
npx cap add android
npx cap add ios


npm i @capacitor-community/native-audio

<input type="file" id="uploadWav" accept="audio/wav">

navigator.mediaDevices.getUserMedia({audio:true})
  .then(stream => {
    const rec = new MediaRecorder(stream);
    rec.ondataavailable = e => {
      const fd = new FormData();
      fd.append("file", e.data, "user.wav");
      fetch("/api/upload", {method:"POST", body:fd});
    };
    // start/stop logic…
  });


from fastapi import UploadFile
@app.post("/api/upload")
async def upload(file: UploadFile, user: str = Depends(get_current_user)):
    out = pathlib.Path("uploads") / f"{user.id}_{uuid4()}.wav"
    out.write_bytes(await file.read())
    return {"url": str(out)}

document.addEventListener("keydown", e=>{
  if (e.code==="Space") { e.preventDefault(); playBtn.click(); }
  if (e.code==="KeyN")   nextBtn.click();
});

@media (prefers-contrast: more) {
  body { background:#000; color:#fff; }
  button { border:2px solid #fff; }
}

@media (prefers-contrast: more) {
  body { background:#000; color:#fff; }
  button { border:2px solid #fff; }
}

<button id="contrastBtn">Toggle High Contrast</button>

*<button id="playBtn" aria-label="Start or pause playback">▶ Start</button>**

<label>BPM
  <input type="range" id="bpmSlider" min="60" max="180" value="100">
  <span id="bpmLabel">100</span>
</label>

const bpmSlider = document.getElementById("bpmSlider");
const bpmLabel  = document.getElementById("bpmLabel");

bpmSlider.oninput = () => {
  bpm = +bpmSlider.value;
  bpmLabel.textContent = bpm;
};


function loadExercise(ex) {
  bpm = ex.bpm;              // <─ new default
  bpmSlider.value = bpm;
  bpmLabel.textContent = bpm;
  ...
}
<button id="tapBtn">TAP</button>

const tapBtn = document.getElementById("tapBtn");
const taps = [];
tapBtn.addEventListener("click", () => {
  const now = Date.now();
  taps.push(now);
  if (taps.length > 4) taps.shift();
  if (taps.length >= 2) {
    const avg = (taps[taps.length-1] - taps[0]) / (taps.length-1);
    bpm = Math.round(60000 / avg);
    bpm = Math.min(200, Math.max(50, bpm));   // clamp
    bpmSlider.value = bpm;
    bpmLabel.textContent = bpm;
  }
});

<button id="tapBtn">TAP</button><button id="tapBtn">TAP</button><button id="tapBtn">TAP</button>

let rampBPM = 0;     // 0 = off
function scheduler() {
  if (rampBPM && bpm < rampBPM) bpm += 0.1;   // 0.1 BPM per tick
  ...
}






