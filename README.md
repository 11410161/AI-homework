<!DOCTYPE html>
<html lang="zh-TW">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>Neon Beats Edu-Strike v4.0</title>
    <style>
        :root {
            --perfect-color: #00ffcc;
            --good-color: #3b82f6;
            --miss-color: #ef4444;
            --gold-color: #fbbf24;
            --hold-color: #a855f7;
            --frenzy-color: #ff0055;
            --bg-dark: #09090b;
            --glass-bg: rgba(20, 20, 25, 0.85);
        }

        * { box-sizing: border-box; margin: 0; padding: 0; user-select: none; -webkit-user-select: none; touch-action: none; }

        body {
            font-family: 'Segoe UI', system-ui, -apple-system, sans-serif;
            background-color: var(--bg-dark); color: #fff; overflow: hidden;
            width: 100vw; height: 100vh; transition: background-color 0.5s, box-shadow 0.5s;
        }

        body.frenzy-mode {
            background-color: #1a0005;
            box-shadow: inset 0 0 150px rgba(255, 0, 85, 0.3);
        }

        .glass-panel {
            background: var(--glass-bg); backdrop-filter: blur(16px); -webkit-backdrop-filter: blur(16px);
            border: 1px solid rgba(255, 255, 255, 0.1); border-radius: 20px;
            box-shadow: 0 0 40px rgba(0, 255, 204, 0.1); padding: clamp(20px, 5vw, 40px);
        }

        .screen {
            position: absolute; top: 0; left: 0; width: 100%; height: 100%;
            display: flex; flex-direction: column; justify-content: center; align-items: center;
            opacity: 0; pointer-events: none; transition: opacity 0.3s ease; z-index: 10;
        }
        .screen.active { opacity: 1; pointer-events: auto; }

        .ui-box { width: 90%; max-width: 600px; text-align: center; }

        h1, h2 {
            font-size: clamp(28px, 6vw, 42px); margin-bottom: 10px; color: transparent;
            -webkit-text-stroke: 1px var(--perfect-color); text-shadow: 0 0 20px rgba(0, 255, 204, 0.5); letter-spacing: 2px;
        }

        .device-group { display: flex; flex-direction: column; gap: 15px; margin-top: 20px; }
        .device-btn {
            background: rgba(0,0,0,0.5); color: #fff; border: 2px solid #555;
            padding: 20px; font-size: 24px; border-radius: 15px; cursor: pointer;
            transition: all 0.2s; display: flex; align-items: center; justify-content: center; gap: 15px;
        }
        .device-btn:hover { border-color: var(--perfect-color); box-shadow: 0 0 20px rgba(0,255,204,0.3); transform: translateY(-2px); }
        .device-btn:active { transform: scale(0.95); }

        input[type="text"] {
            width: 100%; padding: 15px; border-radius: 12px; border: 2px solid rgba(255,255,255,0.2);
            background: rgba(0,0,0,0.5); color: #fff; font-size: 20px; text-align: center;
            margin: 15px 0; outline: none; transition: 0.3s;
        }
        input[type="text"]:focus { border-color: var(--perfect-color); box-shadow: 0 0 15px rgba(0, 255, 204, 0.3); }

        .btn {
            background: transparent; color: #fff; border: 2px solid var(--perfect-color);
            padding: 15px 30px; font-size: 20px; border-radius: 12px; cursor: pointer; width: 100%;
            font-weight: bold; text-transform: uppercase; letter-spacing: 2px; transition: all 0.2s;
            box-shadow: inset 0 0 10px rgba(0, 255, 204, 0.2), 0 0 15px rgba(0, 255, 204, 0.2); margin-top: 10px;
        }
        .btn:active { transform: scale(0.95); background: var(--perfect-color); color: #000; }
        
        .enter-hint {
            margin-top: 15px; font-size: clamp(12px, 3vw, 16px); color: rgba(255,255,255,0.5);
            animation: pulse 1.5s infinite; font-weight: bold; letter-spacing: 1px;
        }
        @keyframes pulse { 0%, 100% { opacity: 0.5; } 50% { opacity: 1; text-shadow: 0 0 10px #fff; } }

        .diff-group { display: flex; gap: 10px; margin: 20px 0; }
        .diff-btn {
            flex: 1; padding: 10px; border-radius: 8px; border: 2px solid #555; background: transparent;
            color: #ccc; cursor: pointer; font-weight: bold; transition: 0.3s;
        }
        .diff-btn.active[data-diff="easy"] { border-color: #22c55e; color: #22c55e; box-shadow: 0 0 15px rgba(34,197,94,0.4); }
        .diff-btn.active[data-diff="normal"] { border-color: #eab308; color: #eab308; box-shadow: 0 0 15px rgba(234,179,8,0.4); }
        .diff-btn.active[data-diff="hard"] { border-color: #ef4444; color: #ef4444; box-shadow: 0 0 15px rgba(239,68,68,0.4); }

        .instruction-list { text-align: left; font-size: clamp(14px, 3.5vw, 16px); line-height: 1.6; color: #ddd; margin-bottom: 10px;}
        .instruction-list li { margin-bottom: 8px; list-style: none; display: flex; align-items: center; gap: 10px;}
        
        #game-canvas { position: absolute; top: 0; left: 0; width: 100%; height: 100%; z-index: 1; }
        #game-hud { position: absolute; top: 0; left: 0; width: 100%; height: 100%; pointer-events: none; z-index: 5; }

        .top-hud { display: flex; justify-content: space-between; padding: clamp(15px, 4vw, 30px); text-shadow: 0 0 10px rgba(0,0,0,0.8); }
        .hud-left, .hud-center, .hud-right { flex: 1; }
        
        .hud-left { font-size: clamp(20px, 5vw, 32px); font-weight: 900; font-style: italic; }
        .combo-text { color: var(--perfect-color); font-size: 1.2em; }
        
        .hud-center { text-align: center; }
        .timer-text { font-family: monospace; font-size: clamp(24px, 6vw, 40px); font-weight: bold; color: #fff; }
        .frenzy-warn { color: var(--frenzy-color); font-size: 14px; font-weight: bold; letter-spacing: 2px; opacity: 0; transition: 0.3s; }
        body.frenzy-mode .frenzy-warn { opacity: 1; animation: flashWarn 0.5s infinite; }
        @keyframes flashWarn { 0%, 100% { opacity: 1; } 50% { opacity: 0; } }

        /* 教育強化：學習提示區 */
        .edu-hint-box {
            margin-top: 10px; background: rgba(0, 0, 0, 0.6); border: 1px solid var(--perfect-color);
            border-radius: 8px; padding: 5px 15px; display: inline-block; opacity: 0; transition: opacity 0.3s;
        }
        .edu-hint-box.show { opacity: 1; }
        .edu-word { color: var(--perfect-color); font-weight: bold; font-size: 20px; }
        .edu-meaning { color: #fff; font-size: 16px; margin-left: 10px; }

        .hud-right { display: flex; flex-direction: column; align-items: flex-end; }
        .opponent-box {
            background: rgba(0,0,0,0.7); border: 1px solid rgba(255,255,255,0.2); border-radius: 10px;
            padding: 10px 15px; display: flex; align-items: center; gap: 10px;
        }
        .opp-avatar { width: 40px; height: 40px; background: #333; border-radius: 50%; border: 2px solid #555; display: flex; justify-content: center; align-items: center; font-size: 20px;}
        .opp-info { text-align: right; }
        .opp-name { font-size: 14px; color: #aaa; font-weight: bold; }
        .opp-score { font-family: monospace; font-size: 20px; font-weight: bold; color: #ef4444; }

        /* 手機模式專屬打擊按鈕 */
        #touch-overlay { 
            position: absolute; bottom: 0; left: 0; width: 100%; height: 20%; 
            display: none; pointer-events: auto; z-index: 10; gap: 5px; padding: 10px;
        }
        #touch-overlay.mobile-mode { display: flex; }
        
        .touch-pad {
            flex: 1; height: 100%; background: rgba(255,255,255,0.05); border: 2px solid rgba(255,255,255,0.2);
            border-radius: 15px; display: flex; justify-content: center; align-items: center;
            font-size: clamp(24px, 6vw, 36px); font-weight: bold; color: rgba(255,255,255,0.5);
            transition: all 0.1s;
        }
        .touch-pad.active {
            background: rgba(0, 255, 204, 0.2); border-color: var(--perfect-color);
            color: #fff; transform: scale(0.95); box-shadow: 0 0 20px rgba(0,255,204,0.5);
        }

        #cert-preview { width: 100%; max-width: 400px; border-radius: 10px; margin-bottom: 20px; box-shadow: 0 0 20px rgba(0,0,0,0.8); }
        .btn-group { display: flex; gap: 15px; }

    </style>
</head>
<body>

    <!-- 0. 裝置選擇 (新增) -->
    <div id="screen-device" class="screen active">
        <div class="glass-panel ui-box">
            <h1>SELECT DEVICE</h1>
            <p style="color: #aaa;">選擇您的遊玩裝置</p>
            <div class="device-group">
                <button class="device-btn" id="btn-pc">💻 電腦模式 (鍵盤 D,F,J,K)</button>
                <button class="device-btn" id="btn-mobile">📱 手機模式 (觸控大按鈕)</button>
            </div>
        </div>
    </div>

    <!-- 1. 登入畫面 -->
    <div id="screen-login" class="screen">
        <div class="glass-panel ui-box">
            <h1>EDU-STRIKE</h1>
            <p style="color: #aaa;">Vocabulary Rhythm Engine v4.0</p>
            <input type="text" id="player-name" placeholder="ENTER YOUR NAME" maxlength="12" autocomplete="off">
            <button id="btn-login" class="btn">NEXT</button>
            <div class="enter-hint">👉 按 Enter 繼續</div>
        </div>
    </div>

    <!-- 2. 教學與難度選擇 -->
    <div id="screen-instructions" class="screen">
        <div class="glass-panel ui-box" style="max-width: 600px;">
            <h2>MISSION BRIEFING</h2>
            <ul class="instruction-list">
                <li>🧠 <b>教育核心：</b> 音符上會顯示「英文單字」。</li>
                <li>👁 <b>學習回饋：</b> 擊中或錯過時，會顯示該單字的「中文意思」。</li>
                <li>🎯 <b>計分：</b> <span style="color:var(--perfect-color)">普通音符</span> | <span style="color:var(--gold-color)">金色音符 (3倍)</span> | <span style="color:var(--hold-color)">長音符 (按住)</span></li>
                <li>⏱ <b>倒數 60 秒！最後 10 秒分數 DOUBLE！</b></li>
            </ul>
            
            <div class="diff-group">
                <button class="diff-btn" data-diff="easy">🟢 初階 (無雙押)</button>
                <button class="diff-btn active" data-diff="normal">🟡 中階</button>
                <button class="diff-btn" data-diff="hard">🔴 高階</button>
            </div>

            <button id="btn-start" class="btn">START LEARNING</button>
            <div class="enter-hint">👉 按 Enter 開始遊戲</div>
        </div>
    </div>

    <!-- 3. 遊戲畫面 -->
    <div id="screen-game" class="screen">
        <canvas id="game-canvas"></canvas>
        <div id="game-hud">
            <div class="top-hud">
                <div class="hud-left">
                    <div id="hud-score">0000000</div>
                    <div id="hud-combo" class="combo-text">0x</div>
                </div>
                <div class="hud-center">
                    <div id="hud-timer" class="timer-text">01:00</div>
                    <div class="frenzy-warn">🔥 2X SCORE FRENZY 🔥</div>
                    <!-- 單字學習提示區 -->
                    <div id="edu-hint" class="edu-hint-box">
                        <span id="edu-word" class="edu-word">APPLE</span>
                        <span id="edu-meaning" class="edu-meaning">蘋果</span>
                    </div>
                </div>
                <div class="hud-right">
                    <div id="opponent-box" class="opponent-box">
                        <div class="opp-info">
                            <div id="opp-name" class="opp-name">TARGET</div>
                            <div id="opp-score" class="opp-score">0000000</div>
                        </div>
                        <div class="opp-avatar" id="opp-avatar">🤖</div>
                    </div>
                </div>
            </div>
            
            <!-- 手機模式觸控區塊 -->
            <div id="touch-overlay">
                <div class="touch-pad" id="pad-0"></div>
                <div class="touch-pad" id="pad-1"></div>
                <div class="touch-pad" id="pad-2"></div>
                <div class="touch-pad" id="pad-3"></div>
            </div>
        </div>
    </div>

    <!-- 4. 結果畫面 -->
    <div id="screen-result" class="screen">
        <div class="glass-panel ui-box" style="max-width: 600px;">
            <h2 style="color: var(--perfect-color);">LESSON CLEARED</h2>
            <img id="cert-preview" alt="Certificate">
            <div class="btn-group">
                <button id="btn-download" class="btn">SAVE PNG</button>
                <button id="btn-restart" class="btn" style="border-color: #888; color: #ccc; box-shadow: none;">RETRY</button>
            </div>
            <div class="enter-hint">👉 按 Enter 重新開始</div>
        </div>
    </div>

    <canvas id="cert-canvas" width="1600" height="900" style="display:none;"></canvas>

<script>
/**
 * 國中英文單字庫 (25個)
 */
const VocabBank = [
    { word: "apple", meaning: "蘋果" }, { word: "banana", meaning: "香蕉" },
    { word: "cat", meaning: "貓" }, { word: "dog", meaning: "狗" },
    { word: "elephant", meaning: "大象" }, { word: "fish", meaning: "魚" },
    { word: "grape", meaning: "葡萄" }, { word: "house", meaning: "房子" },
    { word: "island", meaning: "島嶼" }, { word: "juice", meaning: "果汁" },
    { word: "kite", meaning: "風箏" }, { word: "lion", meaning: "獅子" },
    { word: "monkey", meaning: "猴子" }, { word: "night", meaning: "夜晚" },
    { word: "orange", meaning: "橘子" }, { word: "pig", meaning: "豬" },
    { word: "queen", meaning: "女王" }, { word: "rabbit", meaning: "兔子" },
    { word: "sun", meaning: "太陽" }, { word: "tree", meaning: "樹木" },
    { word: "umbrella", meaning: "雨傘" }, { word: "violin", meaning: "小提琴" },
    { word: "water", meaning: "水" }, { word: "yellow", meaning: "黃色" },
    { word: "zebra", meaning: "斑馬" }
];

const Config = {
    lanes: 4, keys: ['d', 'f', 'j', 'k'],
    hitWindows: { perfect: 50, good: 100, miss: 150 },
    scores: { perfect: 1000, good: 500, miss: 0 },
    gameDuration: 60000, frenzyTime: 10000, hitLineOffset: 0.80 // 稍微往上移，留空間給觸控區
};

const DifficultySettings = {
    easy:   { fallDur: 2500, gapMin: 600, gapMax: 900, doubleChance: 0.0, label: '🟢 初階' },
    normal: { fallDur: 2000, gapMin: 400, gapMax: 700, doubleChance: 0.1, label: '🟡 中階' },
    hard:   { fallDur: 1500, gapMin: 250, gapMax: 500, doubleChance: 0.2, label: '🔴 高階' }
};

const AudioEngine = {
    ctx: null,
    init() { window.AudioContext = window.AudioContext || window.webkitAudioContext; this.ctx = new AudioContext(); },
    playHit(type = 'normal') {
        if (!this.ctx) return;
        if (this.ctx.state === 'suspended') this.ctx.resume();
        const osc = this.ctx.createOscillator(); const gain = this.ctx.createGain();
        osc.connect(gain); gain.connect(this.ctx.destination);
        if (type === 'gold') { osc.type = 'triangle'; osc.frequency.setValueAtTime(1200, this.ctx.currentTime); osc.frequency.exponentialRampToValueAtTime(300, this.ctx.currentTime + 0.1); } 
        else if (type === 'hold_tick') { osc.type = 'sine'; osc.frequency.setValueAtTime(600, this.ctx.currentTime); gain.gain.setValueAtTime(0.05, this.ctx.currentTime); gain.gain.exponentialRampToValueAtTime(0.01, this.ctx.currentTime + 0.05); osc.start(); osc.stop(this.ctx.currentTime + 0.05); return; } 
        else { osc.type = 'square'; osc.frequency.setValueAtTime(800, this.ctx.currentTime); osc.frequency.exponentialRampToValueAtTime(100, this.ctx.currentTime + 0.05); }
        gain.gain.setValueAtTime(0.2, this.ctx.currentTime); gain.gain.exponentialRampToValueAtTime(0.01, this.ctx.currentTime + 0.1);
        osc.start(); osc.stop(this.ctx.currentTime + 0.1);
    }
};

const Engine = {
    state: 'device', isMobile: false, playerName: '', difficulty: 'normal',
    ctx: null, gameTime: 0, startTime: 0, animFrameId: null,
    score: 0, combo: 0, maxCombo: 0, isFrenzy: false,
    
    chart: [], particles: [], judgements: [], laneGlow: [0,0,0,0],
    activeHolds: [null,null,null,null], keysDown: [false,false,false,false],
    
    opponents: [], currentOppIdx: 0, displayOppScore: 0, theoMaxScore: 0,
    wordsEncountered: 0, // 學習統計

    ui: {
        screens: { 
            device: document.getElementById('screen-device'),
            login: document.getElementById('screen-login'), 
            instructions: document.getElementById('screen-instructions'), 
            game: document.getElementById('screen-game'), 
            result: document.getElementById('screen-result') 
        },
        inputName: document.getElementById('player-name'), canvas: document.getElementById('game-canvas'),
        hudScore: document.getElementById('hud-score'), hudCombo: document.getElementById('hud-combo'), hudTimer: document.getElementById('hud-timer'),
        oppBox: document.getElementById('opponent-box'), oppName: document.getElementById('opp-name'), oppScore: document.getElementById('opp-score'), oppAvatar: document.getElementById('opp-avatar'),
        touchOverlay: document.getElementById('touch-overlay'), pads: [document.getElementById('pad-0'), document.getElementById('pad-1'), document.getElementById('pad-2'), document.getElementById('pad-3')],
        eduHintBox: document.getElementById('edu-hint'), eduWord: document.getElementById('edu-word'), eduMeaning: document.getElementById('edu-meaning')
    },

    init() {
        this.ctx = this.ui.canvas.getContext('2d', { alpha: false });
        this.bindEvents();
        this.resize(); window.addEventListener('resize', () => this.resize());
    },

    switchScreen(name) {
        Object.values(this.ui.screens).forEach(s => s.classList.remove('active'));
        this.ui.screens[name].classList.add('active');
        this.state = name;
        if(name === 'login') { this.ui.inputName.value = ''; this.ui.inputName.focus(); }
    },

    bindEvents() {
        // 裝置選擇
        document.getElementById('btn-pc').addEventListener('click', () => { this.isMobile = false; this.setupDevice(); });
        document.getElementById('btn-mobile').addEventListener('click', () => { this.isMobile = true; this.setupDevice(); });

        document.getElementById('btn-login').addEventListener('click', () => this.handleLogin());
        document.getElementById('btn-start').addEventListener('click', () => this.startGame());
        document.getElementById('btn-download').addEventListener('click', () => this.downloadCert());
        document.getElementById('btn-restart').addEventListener('click', () => this.switchScreen('device'));

        document.querySelectorAll('.diff-btn').forEach(btn => {
            btn.addEventListener('click', (e) => {
                document.querySelectorAll('.diff-btn').forEach(b => b.classList.remove('active'));
                e.target.classList.add('active');
                this.difficulty = e.target.dataset.diff;
            });
        });

        window.addEventListener('keydown', (e) => {
            if (e.key === 'Enter') {
                if (this.state === 'login') this.handleLogin();
                else if (this.state === 'instructions') this.startGame();
                else if (this.state === 'result') this.switchScreen('device');
            }
            if (this.state !== 'playing' || e.repeat || this.isMobile) return; // 手機模式忽略鍵盤
            const lane = Config.keys.indexOf(e.key.toLowerCase());
            if (lane !== -1) { this.keysDown[lane] = true; this.handleInputDown(lane); }
        });
        window.addEventListener('keyup', (e) => {
            if (this.state !== 'playing' || this.isMobile) return;
            const lane = Config.keys.indexOf(e.key.toLowerCase());
            if (lane !== -1) { this.keysDown[lane] = false; this.handleInputUp(lane); }
        });

        // 綁定觸控按鈕
        for (let i = 0; i < Config.lanes; i++) {
            const pad = this.ui.pads[i];
            const ts = (e) => { if(e.cancelable) e.preventDefault(); if (!this.keysDown[i]) { this.keysDown[i] = true; pad.classList.add('active'); this.handleInputDown(i); } };
            const te = (e) => { if(e.cancelable) e.preventDefault(); this.keysDown[i] = false; pad.classList.remove('active'); this.handleInputUp(i); };
            pad.addEventListener('touchstart', ts, { passive: false }); pad.addEventListener('mousedown', ts);
            pad.addEventListener('touchend', te); pad.addEventListener('touchcancel', te); pad.addEventListener('mouseup', te); pad.addEventListener('mouseleave', te);
        }
    },

    setupDevice() {
        if (this.isMobile) {
            this.ui.touchOverlay.classList.add('mobile-mode');
            // 設定觸控按鈕提示文字為空，靠位置辨識
            this.ui.pads.forEach((pad, i) => pad.innerText = "");
        } else {
            this.ui.touchOverlay.classList.remove('mobile-mode');
        }
        this.switchScreen('login');
    },

    handleLogin() {
        const name = this.ui.inputName.value.trim().toUpperCase();
        if (!name) {
            this.ui.inputName.style.borderColor = '#ef4444';
            setTimeout(() => this.ui.inputName.style.borderColor = 'rgba(255,255,255,0.2)', 300);
            return;
        }
        this.playerName = name;
        this.switchScreen('instructions');
    },

    resize() {
        const dpr = window.devicePixelRatio || 1;
        this.ui.canvas.width = window.innerWidth * dpr; this.ui.canvas.height = window.innerHeight * dpr;
        this.ctx.scale(dpr, dpr);
    },

    generateChart() {
        this.chart = [];
        const diffConfig = DifficultySettings[this.difficulty];
        const fallDur = diffConfig.fallDur;
        
        let time = 3000; let laneFreeTime = [0, 0, 0, 0]; 
        this.theoMaxScore = 0; let theoCombo = 0;

        while (time < Config.gameDuration) {
            const isDouble = Math.random() < diffConfig.doubleChance;
            const numNotes = isDouble ? 2 : 1;
            
            let availableLanes = [0, 1, 2, 3].filter(l => laneFreeTime[l] < time);
            if (availableLanes.length < numNotes) { time += 100; continue; }

            availableLanes.sort(() => Math.random() - 0.5);
            let selectedLanes = availableLanes.slice(0, numNotes);

            selectedLanes.forEach(lane => {
                let r = Math.random() * 100;
                let type = 'normal'; let duration = 0; let baseScore = Config.scores.perfect;

                if (r < 15 && this.difficulty !== 'easy') { // 初階不給長音符
                    type = 'hold'; duration = Math.floor(Math.random() * 800) + 400;
                    laneFreeTime[lane] = time + duration + 100;
                    baseScore += Math.floor(duration/100) * 20; 
                } else if (r < 30) {
                    type = 'gold'; baseScore *= 3;
                }

                // 分配隨機單字給這個音符
                const vocab = VocabBank[Math.floor(Math.random() * VocabBank.length)];

                this.chart.push({
                    lane: lane, type: type, hitTime: time, spawnTime: time - fallDur,
                    duration: duration, active: true, isBeingHeld: false, holdTicks: 0,
                    vocab: vocab // 教育核心資料
                });

                theoCombo++;
                this.theoMaxScore += baseScore + (theoCombo * 10);
            });

            time += Math.random() * (diffConfig.gapMax - diffConfig.gapMin) + diffConfig.gapMin;
        }
        this.wordsEncountered = this.chart.length;
    },

    generateOpponents() {
        const names = ["NOVICE_BOT", "ENGLISH_GEEK", "PRO_STUDENT", "MASTER_AI"];
        const avatars = ["👶", "👓", "🔥", "👑"];
        const percentages = [0.2, 0.45, 0.7, 0.95]; 
        this.opponents = percentages.map((p, i) => {
            return { name: names[i], avatar: avatars[i], targetScore: Math.floor(this.theoMaxScore * p) + Math.floor(Math.random()*1000) };
        });
        this.currentOppIdx = 0; this.displayOppScore = 0;
        this.updateOpponentUI(true);
    },

    updateOpponentUI(instant = false) {
        if(this.currentOppIdx >= this.opponents.length) return;
        const opp = this.opponents[this.currentOppIdx];
        this.ui.oppName.innerText = opp.name;
        this.ui.oppAvatar.innerText = opp.avatar;
    },

    startGame() {
        if (!AudioEngine.ctx) AudioEngine.init();
        this.switchScreen('game');
        this.generateChart();
        this.generateOpponents();
        
        this.score = 0; this.combo = 0; this.maxCombo = 0;
        this.isFrenzy = false; document.body.classList.remove('frenzy-mode');
        this.particles = []; this.judgements = []; this.laneGlow.fill(0);
        this.activeHolds.fill(null); this.keysDown.fill(false);
        this.ui.eduHintBox.classList.remove('show');
        
        this.updateHUD();
        this.state = 'playing';
        this.startTime = performance.now();
        this.loop();
    },

    loop() {
        if (this.state !== 'playing') return;
        this.gameTime = performance.now() - this.startTime;
        let timeLeft = Math.max(0, Config.gameDuration - this.gameTime);
        
        if (timeLeft <= Config.frenzyTime && !this.isFrenzy) {
            this.isFrenzy = true; document.body.classList.add('frenzy-mode');
        }

        this.update(); this.draw();
        
        let sec = Math.ceil(timeLeft / 1000);
        this.ui.hudTimer.innerText = `${String(Math.floor(sec / 60)).padStart(2, '0')}:${String(sec % 60).padStart(2, '0')}`;
        if (timeLeft <= 10000) this.ui.hudTimer.style.color = '#ff0055'; 

        if (timeLeft <= 0) { this.endGame(); return; }
        this.animFrameId = requestAnimationFrame(() => this.loop());
    },

    update() {
        for(let i=0; i<Config.lanes; i++) if(this.laneGlow[i] > 0) this.laneGlow[i] -= 0.05;

        for (let i = 0; i < this.chart.length; i++) {
            const note = this.chart[i];
            if (!note.active) continue;

            if (note.type === 'hold' && note.isBeingHeld) {
                if (this.gameTime >= note.hitTime + note.duration) {
                    note.active = false; note.isBeingHeld = false; this.activeHolds[note.lane] = null;
                    this.registerJudgement('PERFECT', note.lane, note); 
                } else {
                    if (this.gameTime - note.hitTime > note.holdTicks * 100) {
                        this.score += 20 * (this.isFrenzy ? 2 : 1); note.holdTicks++; this.updateHUD();
                        this.createExplosion(note.lane, 'var(--hold-color)', 2); 
                        if(note.holdTicks % 2 === 0) AudioEngine.playHit('hold_tick');
                    }
                }
            } else {
                if (this.gameTime - note.hitTime > Config.hitWindows.miss) {
                    note.active = false; this.registerJudgement('MISS', note.lane, note);
                }
            }
        }

        if (this.currentOppIdx < this.opponents.length) {
            const opp = this.opponents[this.currentOppIdx];
            const progress = Math.min(1, this.gameTime / Config.gameDuration);
            this.displayOppScore = Math.floor(opp.targetScore * Math.pow(progress, 0.8));
            this.ui.oppScore.innerText = this.displayOppScore.toString().padStart(7, '0');

            if (this.score > this.displayOppScore && this.score > 0) {
                this.currentOppIdx++;
                if (this.currentOppIdx < this.opponents.length) this.updateOpponentUI();
                else { this.ui.oppName.innerText = "VOCAB MASTER"; this.ui.oppAvatar.innerText = "🎓"; this.ui.oppScore.innerText = "CLEARED"; this.ui.oppScore.style.color = "var(--perfect-color)"; }
            }
        }

        this.particles = this.particles.filter(p => p.life > 0);
        this.particles.forEach(p => { p.x += p.vx; p.y += p.vy; p.life -= 0.03; });
        this.judgements = this.judgements.filter(j => j.life > 0);
        this.judgements.forEach(j => { j.y -= 1; j.life -= 0.02; });
    },

    handleInputDown(lane) {
        this.laneGlow[lane] = 1;
        let targetNote = null; let minDelta = Infinity;

        for (let i = 0; i < this.chart.length; i++) {
            const note = this.chart[i];
            if (note.active && note.lane === lane && !note.isBeingHeld) {
                const delta = Math.abs(this.gameTime - note.hitTime);
                if (this.gameTime - note.hitTime > -200 && delta < minDelta) { minDelta = delta; targetNote = note; }
            }
        }

        if (targetNote) {
            let hitType = 'MISS';
            if (minDelta <= Config.hitWindows.perfect) hitType = 'PERFECT';
            else if (minDelta <= Config.hitWindows.good) hitType = 'GOOD';

            if (hitType !== 'MISS') {
                if (targetNote.type === 'normal') { targetNote.active = false; AudioEngine.playHit('normal'); this.registerJudgement(hitType, lane, targetNote); } 
                else if (targetNote.type === 'gold') { targetNote.active = false; AudioEngine.playHit('gold'); this.registerJudgement(hitType, lane, targetNote, 3); } 
                else if (targetNote.type === 'hold') { targetNote.isBeingHeld = true; this.activeHolds[lane] = targetNote; AudioEngine.playHit('normal'); this.registerJudgement(hitType, lane, targetNote, 1, true); }
            } else {
                targetNote.active = false; this.registerJudgement('MISS', lane, targetNote);
            }
        }
    },

    handleInputUp(lane) {
        const holdNote = this.activeHolds[lane];
        if (holdNote && holdNote.active) {
            if ((holdNote.hitTime + holdNote.duration) - this.gameTime > 150) { 
                holdNote.active = false; holdNote.isBeingHeld = false; this.registerJudgement('MISS', lane, holdNote);
            }
        }
        this.activeHolds[lane] = null;
    },

    registerJudgement(type, lane, noteData, multiplier = 1, isHoldStart = false) {
        let text, color, scoreAdd = 0;
        const globalMult = this.isFrenzy ? 2 : 1;
        let vocabText = "";
        
        // 核心教育回饋邏輯
        if (noteData && noteData.vocab) {
            this.showEduHint(noteData.vocab.word, noteData.vocab.meaning, type === 'MISS');
            if (type === 'MISS') {
                vocabText = `正確: ${noteData.vocab.word}\n(${noteData.vocab.meaning})`;
            } else {
                vocabText = `${noteData.vocab.meaning}`;
            }
        }

        if (type === 'PERFECT') {
            text = 'PERFECT'; color = 'var(--perfect-color)'; scoreAdd = Config.scores.perfect * multiplier * globalMult;
            if(!isHoldStart) this.combo++; this.createExplosion(lane, color, 15);
        } else if (type === 'GOOD') {
            text = 'GOOD'; color = 'var(--good-color)'; scoreAdd = Config.scores.good * multiplier * globalMult;
            if(!isHoldStart) this.combo++; this.createExplosion(lane, color, 10);
        } else {
            text = 'MISS'; color = 'var(--miss-color)'; this.combo = 0;
        }

        if (this.combo > this.maxCombo) this.maxCombo = this.combo;
        this.score += scoreAdd + (this.combo * 10 * globalMult);
        this.updateHUD();

        const w = window.innerWidth; const laneWidth = w / Config.lanes;
        this.judgements.push({ 
            text: text, vocabText: vocabText, color: color, 
            x: laneWidth * lane + laneWidth / 2, y: window.innerHeight * Config.hitLineOffset - 60, 
            life: 1.0, scale: (type === 'PERFECT' && !isHoldStart) ? 1.2 : 1.0 
        });
    },

    showEduHint(word, meaning, isMiss) {
        this.ui.eduWord.innerText = word.toUpperCase();
        this.ui.eduMeaning.innerText = meaning;
        this.ui.eduWord.style.color = isMiss ? 'var(--miss-color)' : 'var(--perfect-color)';
        this.ui.eduHintBox.style.borderColor = isMiss ? 'var(--miss-color)' : 'var(--perfect-color)';
        
        this.ui.eduHintBox.classList.remove('show');
        void this.ui.eduHintBox.offsetWidth; // trigger reflow
        this.ui.eduHintBox.classList.add('show');
    },

    createExplosion(lane, color, count) {
        const w = window.innerWidth; const laneWidth = w / Config.lanes;
        const x = laneWidth * lane + laneWidth / 2; const y = window.innerHeight * Config.hitLineOffset;
        for (let i = 0; i < count; i++) {
            const angle = Math.random() * Math.PI * 2; const speed = Math.random() * 8 + 2;
            this.particles.push({ x: x, y: y, vx: Math.cos(angle) * speed, vy: Math.sin(angle) * speed, color: color, life: 1.0, size: Math.random() * 4 + 2 });
        }
    },

    updateHUD() {
        this.ui.hudScore.innerText = this.score.toString().padStart(7, '0');
        this.ui.hudCombo.innerText = this.combo > 3 ? `${this.combo}x` : '';
        this.ui.hudCombo.style.textShadow = this.combo > 50 ? '0 0 20px var(--perfect-color)' : 'none';
        if(this.isFrenzy) this.ui.hudScore.style.color = 'var(--frenzy-color)';
    },

    draw() {
        const ctx = this.ctx; const w = window.innerWidth; const h = window.innerHeight;
        const hitY = h * Config.hitLineOffset; const laneWidth = w / Config.lanes; 
        const noteHeight = Math.max(30, h * 0.04); // 為了塞單字，音符加厚
        const fallDur = DifficultySettings[this.difficulty].fallDur;

        ctx.clearRect(0, 0, w, h);
        ctx.globalCompositeOperation = 'lighter';

        for (let i = 0; i < Config.lanes; i++) {
            const x = i * laneWidth;
            ctx.beginPath(); ctx.moveTo(x, 0); ctx.lineTo(x, h);
            ctx.strokeStyle = 'rgba(255,255,255,0.05)'; ctx.lineWidth = 1; ctx.stroke();

            let glow = this.laneGlow[i]; if (this.activeHolds[i]) glow = 1; 
            if (glow > 0) {
                const gradient = ctx.createLinearGradient(0, hitY, 0, h);
                gradient.addColorStop(0, this.isFrenzy ? `rgba(255, 0, 85, ${glow * 0.4})` : `rgba(0, 255, 204, ${glow * 0.3})`);
                gradient.addColorStop(1, 'rgba(0, 0, 0, 0)');
                ctx.fillStyle = gradient; ctx.fillRect(x, hitY, laneWidth, h - hitY);
            }
        }

        ctx.beginPath(); ctx.moveTo(0, hitY); ctx.lineTo(w, hitY);
        ctx.strokeStyle = this.isFrenzy ? 'rgba(255, 0, 85, 0.6)' : 'rgba(255, 255, 255, 0.4)'; ctx.lineWidth = 2;
        ctx.shadowBlur = 15; ctx.shadowColor = this.isFrenzy ? '#ff0055' : '#00ffcc'; ctx.stroke(); ctx.shadowBlur = 0;

        ctx.globalCompositeOperation = 'source-over';

        // 繪製 Hold 尾巴
        for (let i = 0; i < this.chart.length; i++) {
            const note = this.chart[i];
            if (!note.active || note.type !== 'hold') continue;
            
            const velocity = hitY / fallDur;
            let headY = (this.gameTime - note.spawnTime) * velocity;
            let tailY = (this.gameTime - (note.spawnTime + note.duration)) * velocity;
            if (note.isBeingHeld) headY = hitY;
            if (tailY > hitY + 100 || headY < 0) continue; 

            const x = note.lane * laneWidth + laneWidth * 0.1; const width = laneWidth * 0.8;
            const drawTailY = Math.max(0, tailY); const drawHeadY = Math.min(h, headY);
            const length = Math.max(0, drawHeadY - drawTailY);

            if (length > 0) {
                const grad = ctx.createLinearGradient(0, drawTailY, 0, drawHeadY);
                grad.addColorStop(0, 'rgba(168, 85, 247, 0.2)'); grad.addColorStop(1, 'rgba(168, 85, 247, 0.8)');
                ctx.fillStyle = grad; ctx.fillRect(x, drawTailY, width, length);
                ctx.fillStyle = '#d8b4fe'; ctx.fillRect(x, drawTailY, 4, length); ctx.fillRect(x + width - 4, drawTailY, 4, length);
            }
        }

        // 繪製音符頭部與「英文單字」
        for (let i = 0; i < this.chart.length; i++) {
            const note = this.chart[i];
            if (!note.active || (note.type === 'hold' && note.isBeingHeld)) continue; 

            const progress = (this.gameTime - note.spawnTime) / fallDur;
            const y = progress * hitY;
            if (y < -50 || y > h + 50) continue;

            const x = note.lane * laneWidth;
            const noteWidth = laneWidth - 10;
            const nX = x + 5; const nY = y - noteHeight/2;
            
            ctx.beginPath(); ctx.roundRect(nX, nY, noteWidth, noteHeight, 8);
            
            let noteColor = this.isFrenzy ? '#ffffff' : 'var(--perfect-color)';
            let textColor = '#000000'; // 深色字體保證對比度
            
            if (note.type === 'gold') { noteColor = 'var(--gold-color)'; } 
            else if (note.type === 'hold') { noteColor = '#ffffff'; textColor = 'var(--hold-color)'; } 
            
            ctx.fillStyle = noteColor; 
            ctx.shadowBlur = this.isFrenzy || note.type === 'gold' ? 15 : 0; 
            ctx.shadowColor = noteColor;
            ctx.fill(); ctx.shadowBlur = 0; 

            // 繪製單字
            ctx.fillStyle = textColor;
            ctx.textAlign = 'center'; ctx.textBaseline = 'middle';
            
            // 動態調整字體大小防溢出
            let fontSize = Math.max(14, w * 0.025);
            ctx.font = `bold ${fontSize}px sans-serif`;
            let textWidth = ctx.measureText(note.vocab.word).width;
            if (textWidth > noteWidth - 10) {
                fontSize = fontSize * ((noteWidth - 10) / textWidth);
                ctx.font = `bold ${fontSize}px sans-serif`;
            }
            ctx.fillText(note.vocab.word, nX + noteWidth/2, nY + noteHeight/2);
        }

        ctx.globalCompositeOperation = 'lighter';
        this.particles.forEach(p => {
            ctx.globalAlpha = p.life; ctx.fillStyle = p.color;
            ctx.beginPath(); ctx.arc(p.x, p.y, p.size, 0, Math.PI * 2); ctx.fill();
        });
        
        ctx.globalCompositeOperation = 'source-over'; ctx.textAlign = 'center'; ctx.textBaseline = 'middle';
        this.judgements.forEach(j => {
            ctx.globalAlpha = j.life;
            ctx.fillStyle = getComputedStyle(document.documentElement).getPropertyValue(j.color.replace('var(','').replace(')','')).trim() || j.color;
            const baseSize = Math.max(20, w * 0.03) * j.scale; ctx.font = `900 italic ${baseSize}px 'Segoe UI', sans-serif`;
            ctx.shadowBlur = 10; ctx.shadowColor = ctx.fillStyle; 
            ctx.fillText(j.text, j.x, j.y); 
            
            // 繪製教育回饋文字 (如：正確：apple)
            if (j.vocabText) {
                ctx.font = `bold ${Math.max(14, w * 0.02)}px sans-serif`;
                let lines = j.vocabText.split('\n');
                lines.forEach((line, idx) => {
                    ctx.fillText(line, j.x, j.y + baseSize + (idx * 20));
                });
            }
            ctx.shadowBlur = 0;
        });
        ctx.globalAlpha = 1.0;
    },

    endGame() {
        this.state = 'result';
        cancelAnimationFrame(this.animFrameId);
        document.body.classList.remove('frenzy-mode');
        this.ui.hudScore.style.color = '#fff';
        this.ui.eduHintBox.classList.remove('show');
        this.generateCertificate();
        this.switchScreen('result');
    },

    generateCertificate() {
        const cvs = document.getElementById('cert-canvas'); const ctx = cvs.getContext('2d');
        const w = cvs.width; const h = cvs.height;

        ctx.fillStyle = '#09090b'; ctx.fillRect(0, 0, w, h);
        ctx.strokeStyle = 'rgba(0, 255, 204, 0.1)'; ctx.lineWidth = 2;
        for(let i=0; i<w; i+=50) { ctx.beginPath(); ctx.moveTo(i,0); ctx.lineTo(i,h); ctx.stroke(); }
        for(let i=0; i<h; i+=50) { ctx.beginPath(); ctx.moveTo(0,i); ctx.lineTo(w,i); ctx.stroke(); }

        ctx.strokeStyle = '#00ffcc'; ctx.lineWidth = 10; ctx.strokeRect(40, 40, w - 80, h - 80);
        ctx.strokeStyle = '#3b82f6'; ctx.lineWidth = 4; ctx.strokeRect(55, 55, w - 110, h - 110);

        ctx.textAlign = 'center'; ctx.textBaseline = 'middle';
        ctx.fillStyle = '#00ffcc'; ctx.font = '900 italic 70px sans-serif';
        ctx.fillText('EDU-STRIKE LEARNING REPORT', w/2, 130);
        
        ctx.fillStyle = '#eab308'; ctx.font = 'bold 35px sans-serif';
        ctx.fillText(`MODE: ${this.isMobile ? 'MOBILE' : 'PC'} | DIFF: ${DifficultySettings[this.difficulty].label.replace(/[^A-Z]/g,'')}`, w/2, 210);

        ctx.fillStyle = '#fff'; let nameFont = 90; ctx.font = `bold ${nameFont}px sans-serif`;
        const nameText = `STUDENT: ${this.playerName}`;
        while (ctx.measureText(nameText).width > w - 200 && nameFont > 40) { nameFont -= 5; ctx.font = `bold ${nameFont}px sans-serif`; }
        ctx.fillText(nameText, w/2, 350);

        ctx.fillStyle = '#fbbf24'; ctx.font = 'bold 60px monospace';
        ctx.fillText(`SCORE: ${this.score.toString().padStart(7, '0')}`, w/2, 480);
        
        ctx.fillStyle = '#a855f7'; ctx.font = 'bold 45px monospace';
        ctx.fillText(`MAX COMBO: ${this.maxCombo}`, w/2, 570);
        
        // 教育數據專屬
        ctx.fillStyle = '#22c55e'; ctx.font = 'bold 40px sans-serif';
        ctx.fillText(`WORDS ENCOUNTERED: ${this.wordsEncountered}`, w/2, 650);

        const d = new Date(); const dateStr = `${d.getFullYear()}/${String(d.getMonth()+1).padStart(2,'0')}/${String(d.getDate()).padStart(2,'0')} ${String(d.getHours()).padStart(2,'0')}:${String(d.getMinutes()).padStart(2,'0')}`;
        ctx.fillStyle = '#888'; ctx.font = '30px sans-serif'; ctx.fillText(`LEARNING SESSION: ${dateStr}`, w/2, 780);

        document.getElementById('cert-preview').src = cvs.toDataURL('image/png');
    },

    downloadCert() {
        const link = document.createElement('a'); link.download = `EduStrike_${this.playerName}.png`;
        link.href = document.getElementById('cert-canvas').toDataURL('image/png'); link.click();
    }
};

window.onload = () => Engine.init();

</script>
</body>
</html>
