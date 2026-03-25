<!DOCTYPE html>
<html lang="zh-TW">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>Edu-Strike Pro v5.0 | Adaptive Learning</title>
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
            --fam-high: #22c55e; /* 熟悉 (綠) */
            --fam-mid: #eab308;  /* 普通 (黃) */
            --fam-low: #ef4444;  /* 不熟 (紅) */
        }

        * { box-sizing: border-box; margin: 0; padding: 0; user-select: none; -webkit-user-select: none; touch-action: none; }

        body {
            font-family: 'Segoe UI', system-ui, -apple-system, sans-serif;
            background-color: var(--bg-dark); color: #fff; overflow: hidden;
            width: 100vw; height: 100vh; transition: background-color 0.5s, box-shadow 0.5s;
        }

        body.frenzy-mode {
            background-color: #1a0005; box-shadow: inset 0 0 150px rgba(255, 0, 85, 0.2);
        }

        .glass-panel {
            background: var(--glass-bg); backdrop-filter: blur(20px); -webkit-backdrop-filter: blur(20px);
            border: 1px solid rgba(255, 255, 255, 0.1); border-radius: 24px;
            box-shadow: 0 0 50px rgba(0, 255, 204, 0.1); padding: clamp(20px, 5vw, 40px);
            max-height: 90vh; overflow-y: auto;
        }
        /* 隱藏卷軸 */
        .glass-panel::-webkit-scrollbar { width: 0px; }

        .screen {
            position: absolute; top: 0; left: 0; width: 100%; height: 100%;
            display: flex; flex-direction: column; justify-content: center; align-items: center;
            opacity: 0; pointer-events: none; transition: opacity 0.4s ease; z-index: 10;
        }
        .screen.active { opacity: 1; pointer-events: auto; }

        .ui-box { width: 95%; max-width: 650px; text-align: center; }

        h1, h2, h3 { color: transparent; -webkit-text-stroke: 1px var(--perfect-color); text-shadow: 0 0 20px rgba(0, 255, 204, 0.4); }
        h1 { font-size: clamp(32px, 8vw, 54px); margin-bottom: 5px; letter-spacing: 2px; }
        h2 { font-size: clamp(24px, 6vw, 36px); margin-bottom: 15px; }

        .subtitle { color: #aaa; font-size: clamp(14px, 4vw, 18px); margin-bottom: 30px; letter-spacing: 1px; }

        /* 按鈕與表單 */
        .device-group { display: flex; flex-direction: column; gap: 15px; margin-top: 20px; }
        .device-btn {
            background: rgba(0,0,0,0.6); color: #fff; border: 2px solid #555;
            padding: 20px; font-size: clamp(18px, 5vw, 24px); border-radius: 16px; cursor: pointer;
            transition: all 0.2s; display: flex; align-items: center; justify-content: center; gap: 15px;
        }
        .device-btn:hover, .device-btn:active { border-color: var(--perfect-color); box-shadow: 0 0 20px rgba(0,255,204,0.3); transform: translateY(-2px); }

        input[type="text"] {
            width: 100%; padding: 18px; border-radius: 14px; border: 2px solid rgba(255,255,255,0.2);
            background: rgba(0,0,0,0.5); color: #fff; font-size: 22px; text-align: center;
            margin: 15px 0 25px 0; outline: none; transition: 0.3s;
        }
        input[type="text"]:focus { border-color: var(--perfect-color); box-shadow: 0 0 15px rgba(0, 255, 204, 0.3); }

        .btn {
            background: rgba(0, 255, 204, 0.1); color: #fff; border: 2px solid var(--perfect-color);
            padding: 16px 32px; font-size: clamp(18px, 5vw, 22px); border-radius: 14px; cursor: pointer; width: 100%;
            font-weight: bold; text-transform: uppercase; letter-spacing: 2px; transition: all 0.2s;
            box-shadow: inset 0 0 10px rgba(0, 255, 204, 0.1), 0 0 20px rgba(0, 255, 204, 0.2);
        }
        .btn:active { transform: scale(0.96); background: var(--perfect-color); color: #000; }
        
        .enter-hint { margin-top: 15px; font-size: 14px; color: rgba(255,255,255,0.4); animation: pulse 2s infinite; font-weight: bold; }
        @keyframes pulse { 0%, 100% { opacity: 0.4; } 50% { opacity: 1; } }

        .diff-group { display: flex; gap: 10px; margin: 25px 0; }
        .diff-btn { flex: 1; padding: 12px 5px; border-radius: 10px; border: 2px solid #444; background: transparent; color: #aaa; cursor: pointer; font-weight: bold; transition: 0.3s; font-size: clamp(14px, 4vw, 16px);}
        .diff-btn.active[data-diff="easy"] { border-color: #22c55e; color: #22c55e; background: rgba(34,197,94,0.1); box-shadow: 0 0 15px rgba(34,197,94,0.3); }
        .diff-btn.active[data-diff="normal"] { border-color: #eab308; color: #eab308; background: rgba(234,179,8,0.1); box-shadow: 0 0 15px rgba(234,179,8,0.3); }
        .diff-btn.active[data-diff="hard"] { border-color: #ef4444; color: #ef4444; background: rgba(239,68,68,0.1); box-shadow: 0 0 15px rgba(239,68,68,0.3); }

        /* 遊戲畫面 HUD */
        #game-canvas { position: absolute; top: 0; left: 0; width: 100%; height: 100%; z-index: 1; }
        #game-hud { position: absolute; top: 0; left: 0; width: 100%; height: 100%; pointer-events: none; z-index: 5; display: flex; flex-direction: column;}

        .top-hud { display: flex; justify-content: space-between; padding: clamp(10px, 3vw, 25px); text-shadow: 0 2px 10px rgba(0,0,0,0.9); }
        .hud-left, .hud-center, .hud-right { flex: 1; }
        
        .hud-left { font-size: clamp(20px, 5vw, 32px); font-weight: 900; font-style: italic; }
        .combo-text { color: var(--perfect-color); font-size: 1.1em; display: inline-block; transition: transform 0.1s; }
        
        .hud-center { text-align: center; display: flex; flex-direction: column; align-items: center; }
        .timer-text { font-family: monospace; font-size: clamp(28px, 7vw, 42px); font-weight: bold; color: #fff; letter-spacing: 2px;}
        .frenzy-warn { color: var(--frenzy-color); font-size: clamp(12px, 3vw, 16px); font-weight: bold; letter-spacing: 2px; opacity: 0; margin-top: 5px;}
        body.frenzy-mode .frenzy-warn { animation: flashWarn 0.5s infinite; }
        @keyframes flashWarn { 0%, 100% { opacity: 1; } 50% { opacity: 0; } }

        /* 頂部中文提示 (全域字義) */
        .global-hint {
            margin-top: 15px; padding: 8px 20px; background: rgba(0, 0, 0, 0.7);
            border: 1px solid rgba(255,255,255,0.2); border-radius: 20px;
            font-size: clamp(16px, 4vw, 22px); font-weight: bold; color: #ddd;
            display: flex; gap: 15px; align-items: center;
        }
        .global-hint .word { color: var(--perfect-color); text-transform: uppercase; }

        .hud-right { display: flex; flex-direction: column; align-items: flex-end; }
        .opponent-box { background: rgba(0,0,0,0.8); border: 1px solid rgba(255,255,255,0.15); border-radius: 12px; padding: 8px 15px; display: flex; align-items: center; gap: 10px; }
        .opp-avatar { width: 36px; height: 36px; background: #222; border-radius: 50%; border: 2px solid #666; display: flex; justify-content: center; align-items: center; font-size: 20px;}
        .opp-info { text-align: right; }
        .opp-name { font-size: clamp(10px, 3vw, 12px); color: #aaa; font-weight: bold; letter-spacing: 1px;}
        .opp-score { font-family: monospace; font-size: clamp(16px, 4vw, 20px); font-weight: bold; color: #ef4444; }

        /* 手機觸控區 */
        #touch-overlay { 
            position: absolute; bottom: 0; left: 0; width: 100%; height: 22%; 
            display: none; pointer-events: auto; z-index: 10; gap: 8px; padding: 10px;
            background: linear-gradient(to top, rgba(0,0,0,0.8), transparent);
        }
        #touch-overlay.mobile-mode { display: flex; }
        .touch-pad {
            flex: 1; height: 100%; background: rgba(255,255,255,0.03); border: 2px solid rgba(255,255,255,0.1);
            border-radius: 16px; display: flex; justify-content: center; align-items: flex-end; padding-bottom: 15px;
            font-size: clamp(20px, 5vw, 28px); font-weight: bold; color: rgba(255,255,255,0.2); transition: all 0.05s;
        }
        .touch-pad.active { background: rgba(0, 255, 204, 0.2); border-color: var(--perfect-color); color: #fff; transform: scale(0.95); box-shadow: inset 0 0 20px rgba(0,255,204,0.3); }

        /* 學習分析報告 UI */
        .report-grid { display: grid; grid-template-columns: 1fr 1fr; gap: 15px; margin: 20px 0; text-align: left; }
        @media (max-width: 500px) { .report-grid { grid-template-columns: 1fr; } }
        
        .report-card { background: rgba(0,0,0,0.5); border: 1px solid rgba(255,255,255,0.1); border-radius: 12px; padding: 15px; }
        .report-card h3 { font-size: 16px; color: #fff; -webkit-text-stroke: 0; text-shadow: none; margin-bottom: 10px; display: flex; align-items: center; justify-content: space-between; }
        
        .stat-row { display: flex; justify-content: space-between; margin-bottom: 8px; font-size: 14px; align-items: center; padding: 5px; border-radius: 6px; background: rgba(255,255,255,0.03);}
        .stat-word { font-weight: bold; text-transform: uppercase; }
        .fam-bar-container { width: 60px; height: 6px; background: #333; border-radius: 3px; overflow: hidden; }
        .fam-bar { height: 100%; border-radius: 3px; transition: width 1s ease-out; }
        
        .fam-high { color: var(--fam-high); } .fam-bar.fam-high { background: var(--fam-high); box-shadow: 0 0 8px var(--fam-high); }
        .fam-mid { color: var(--fam-mid); } .fam-bar.fam-mid { background: var(--fam-mid); box-shadow: 0 0 8px var(--fam-mid); }
        .fam-low { color: var(--fam-low); } .fam-bar.fam-low { background: var(--fam-low); box-shadow: 0 0 8px var(--fam-low); }

        .acc-display { font-size: 36px; font-weight: 900; color: var(--perfect-color); text-align: center; margin-bottom: 15px; text-shadow: 0 0 15px rgba(0,255,204,0.4); }
        .btn-group { display: flex; gap: 10px; margin-top: 20px; }
    </style>
</head>
<body>

    <!-- 0. 裝置選擇 -->
    <div id="screen-device" class="screen active">
        <div class="glass-panel ui-box">
            <h1>EDU-STRIKE PRO</h1>
            <div class="subtitle">v5.0 Adaptive Learning System</div>
            <div class="device-group">
                <button class="device-btn" id="btn-pc">💻 電腦鍵盤 (D, F, J, K)</button>
                <button class="device-btn" id="btn-mobile">📱 手機觸控 (底部感應區)</button>
            </div>
        </div>
    </div>

    <!-- 1. 登入 -->
    <div id="screen-login" class="screen">
        <div class="glass-panel ui-box">
            <h2>STUDENT ID</h2>
            <div class="subtitle">請輸入您的學習代號以追蹤學習歷程</div>
            <input type="text" id="player-name" placeholder="ENTER YOUR NAME" maxlength="12" autocomplete="off">
            <button id="btn-login" class="btn">INITIALIZE</button>
            <div class="enter-hint hidden-mobile">👉 按 Enter 繼續</div>
        </div>
    </div>

    <!-- 2. 難度與教學 -->
    <div id="screen-instructions" class="screen">
        <div class="glass-panel ui-box">
            <h2>LEARNING PROTOCOL</h2>
            <div style="text-align: left; font-size: clamp(14px, 3.5vw, 16px); color: #ccc; line-height: 1.6; background: rgba(0,0,0,0.4); padding: 15px; border-radius: 12px; margin-bottom: 15px;">
                <p>🧠 <b>自適應系統啟動：</b> 系統將分析您的記憶弱點，您<b>不熟悉的單字將會提高出現頻率</b>。</p>
                <p>👁 <b>雙重回饋：</b> 畫面上方提示字義，漏打時強制顯示正確翻譯加深印象。</p>
                <p>🎵 <b>操作：</b> 擊中音符底線。包含 <span style="color:var(--gold-color)">金色加分</span> 與 <span style="color:var(--hold-color)">紫色長按</span> 音符。</p>
            </div>
            
            <div class="diff-group">
                <button class="diff-btn" data-diff="easy">🟢 初階</button>
                <button class="diff-btn active" data-diff="normal">🟡 中階</button>
                <button class="diff-btn" data-diff="hard">🔴 高階</button>
            </div>

            <button id="btn-start" class="btn">START SESSION</button>
            <div class="enter-hint hidden-mobile">👉 按 Enter 開始</div>
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
                    <div class="frenzy-warn">🔥 2X NEURAL FRENZY 🔥</div>
                    <!-- 頂部中文提示 -->
                    <div class="global-hint" id="global-hint" style="opacity: 0;">
                        <span class="word" id="hint-word">---</span>
                        <span class="meaning" id="hint-meaning">---</span>
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

    <!-- 4. 學習報告 (結果) -->
    <div id="screen-result" class="screen">
        <div class="glass-panel ui-box">
            <h2>LEARNING REPORT</h2>
            
            <div class="acc-display">
                <span id="res-acc">0</span>% ACCURACY
            </div>
            <div style="font-size: 14px; color: #aaa; margin-bottom: 10px;">
                SCORE: <span id="res-score" style="color: #fff; font-weight:bold;">0</span> | 
                MAX COMBO: <span id="res-combo" style="color: #fff; font-weight:bold;">0</span>
            </div>

            <div class="report-grid">
                <div class="report-card">
                    <h3>🔴 記憶弱點 <span style="font-size: 12px; color:#aaa; font-weight:normal;">(最需加強)</span></h3>
                    <div id="list-weak"></div>
                </div>
                <div class="report-card">
                    <h3>🟢 已掌握 <span style="font-size: 12px; color:#aaa; font-weight:normal;">(熟悉度高)</span></h3>
                    <div id="list-strong"></div>
                </div>
            </div>

            <div class="btn-group">
                <button id="btn-download" class="btn" style="flex: 2; font-size: 16px;">📥 匯出證書 PNG</button>
                <button id="btn-restart" class="btn" style="flex: 1; background: transparent; border-color: #555; color: #ccc; box-shadow: none; font-size: 16px;">↻ 重試</button>
            </div>
        </div>
    </div>

    <!-- 隱藏的 Canvas 用於生成證書 -->
    <canvas id="cert-canvas" width="1080" height="1080" style="display:none;"></canvas>

<script>
/**
 * V5 核心：教育科技模組 (30字題庫 + 難度分級)
 */
const VocabBank = [
    { word: "apple", meaning: "蘋果", level: "easy" }, { word: "banana", meaning: "香蕉", level: "easy" },
    { word: "cat", meaning: "貓", level: "easy" }, { word: "dog", meaning: "狗", level: "easy" },
    { word: "fish", meaning: "魚", level: "easy" }, { word: "sun", meaning: "太陽", level: "easy" },
    { word: "tree", meaning: "樹木", level: "easy" }, { word: "water", meaning: "水", level: "easy" },
    { word: "book", meaning: "書本", level: "easy" }, { word: "milk", meaning: "牛奶", level: "easy" },
    
    { word: "elephant", meaning: "大象", level: "normal" }, { word: "grape", meaning: "葡萄", level: "normal" },
    { word: "house", meaning: "房子", level: "normal" }, { word: "juice", meaning: "果汁", level: "normal" },
    { word: "kite", meaning: "風箏", level: "normal" }, { word: "lion", meaning: "獅子", level: "normal" },
    { word: "monkey", meaning: "猴子", level: "normal" }, { word: "night", meaning: "夜晚", level: "normal" },
    { word: "orange", meaning: "橘子", level: "normal" }, { word: "rabbit", meaning: "兔子", level: "normal" },
    { word: "yellow", meaning: "黃色", level: "normal" }, { word: "zebra", meaning: "斑馬", level: "normal" },
    { word: "bird", meaning: "鳥", level: "normal" }, { word: "desk", meaning: "書桌", level: "normal" },
    { word: "clock", meaning: "時鐘", level: "normal" },
    
    { word: "island", meaning: "島嶼", level: "hard" }, { word: "queen", meaning: "女王", level: "hard" },
    { word: "umbrella", meaning: "雨傘", level: "hard" }, { word: "violin", meaning: "小提琴", level: "hard" },
    { word: "xylophone", meaning: "木琴", level: "hard" }
];

/**
 * 自適應學習引擎 (Adaptive Learning & Spaced Repetition)
 */
const LearningSystem = {
    data: {}, // { "apple": { familiarity: 0.5, correct: 0, wrong: 0, lastSeen: 0 } }
    sessionStats: { totalNotes: 0, hitNotes: 0 }, // 單局統計
    
    init(playerName) {
        this.storageKey = `EduStrike_v5_${playerName}`;
        this.loadData();
    },
    
    loadData() {
        const stored = localStorage.getItem(this.storageKey);
        if (stored) {
            this.data = JSON.parse(stored);
        } else {
            // 初始化
            VocabBank.forEach(v => {
                this.data[v.word] = { familiarity: 0.3, correct: 0, wrong: 0, lastSeen: 0 };
            });
        }
        // 確保新增加的單字也被初始化
        VocabBank.forEach(v => {
            if (!this.data[v.word]) this.data[v.word] = { familiarity: 0.3, correct: 0, wrong: 0, lastSeen: 0 };
        });
    },

    saveData() { localStorage.setItem(this.storageKey, JSON.stringify(this.data)); },

    // 主動回憶更新：打對增加，打錯扣除
    updateFamiliarity(word, isHit) {
        let stat = this.data[word];
        this.sessionStats.totalNotes++;
        if (isHit) {
            stat.familiarity = Math.min(1.0, stat.familiarity + 0.1);
            stat.correct++;
            this.sessionStats.hitNotes++;
        } else {
            stat.familiarity = Math.max(0.0, stat.familiarity - 0.15);
            stat.wrong++;
        }
        this.saveData();
    },

    // 核心 AI：根據權重與冷卻時間挑選單字
    selectWord(currentTimeSim, tempStatsTracker) {
        let totalWeight = 0;
        let weights = [];

        for (let i = 0; i < VocabBank.length; i++) {
            let w = VocabBank[i].word;
            let stat = this.data[w];
            
            // 使用模擬追蹤器（防止同一局連續出同一個字）
            let lastSeen = tempStatsTracker[w] || stat.lastSeen; 
            
            // 間隔重複邏輯 (Spaced Repetition)
            let timeDiff = currentTimeSim - lastSeen;
            let cooldownPenalty = 1.0;
            if (timeDiff < 4000) cooldownPenalty = 0.05; // 剛出過，機率極低
            else if (timeDiff < 8000) cooldownPenalty = 0.3;
            
            // 越不熟權重越高
            let baseWeight = 1.0 - stat.familiarity;
            // 保底機率，避免熟字完全不出現
            let finalWeight = Math.max(0.05, baseWeight) * cooldownPenalty;
            
            weights.push(finalWeight);
            totalWeight += finalWeight;
        }

        let rand = Math.random() * totalWeight;
        let sum = 0;
        for (let i = 0; i < VocabBank.length; i++) {
            sum += weights[i];
            if (rand <= sum) {
                tempStatsTracker[VocabBank[i].word] = currentTimeSim; // 更新模擬時間
                return VocabBank[i];
            }
        }
        return VocabBank[0]; // Fallback
    },

    // 分析報告用
    getSortedWords(type) {
        let arr = VocabBank.map(v => ({ word: v.word, meaning: v.meaning, ...this.data[v.word] }));
        if (type === 'weak') {
            // 最弱的：familiarity 低，錯的多
            return arr.sort((a, b) => a.familiarity - b.familiarity || b.wrong - a.wrong).slice(0, 5);
        } else {
            // 最強的：familiarity 高，對的多
            return arr.sort((a, b) => b.familiarity - a.familiarity || b.correct - a.correct).slice(0, 5);
        }
    },
    
    getFamClass(val) {
        if(val >= 0.7) return 'fam-high';
        if(val >= 0.4) return 'fam-mid';
        return 'fam-low';
    }
};

/**
 * 遊戲設定與引擎
 */
const Config = {
    lanes: 4, keys: ['d', 'f', 'j', 'k'],
    hitWindows: { perfect: 60, good: 120, miss: 200 }, // 放寬一點容錯率以便閱讀單字
    scores: { perfect: 1000, good: 500, miss: 0 },
    gameDuration: 60000, frenzyTime: 10000, hitLineOffset: 0.75 
};

const DifficultySettings = {
    easy:   { fallDur: 2800, gapMin: 700, gapMax: 1000, doubleChance: 0.0, label: '🟢 初階' },
    normal: { fallDur: 2200, gapMin: 500, gapMax: 800,  doubleChance: 0.1, label: '🟡 中階' },
    hard:   { fallDur: 1600, gapMin: 350, gapMax: 600,  doubleChance: 0.2, label: '🔴 高階' }
};

const AudioEngine = {
    ctx: null,
    init() { window.AudioContext = window.AudioContext || window.webkitAudioContext; this.ctx = new AudioContext(); },
    playHit(type = 'normal') {
        if (!this.ctx || this.ctx.state !== 'running') return;
        const osc = this.ctx.createOscillator(); const gain = this.ctx.createGain();
        osc.connect(gain); gain.connect(this.ctx.destination);
        if (type === 'gold') { osc.type = 'triangle'; osc.frequency.setValueAtTime(1200, this.ctx.currentTime); osc.frequency.exponentialRampToValueAtTime(300, this.ctx.currentTime + 0.1); } 
        else if (type === 'hold_tick') { osc.type = 'sine'; osc.frequency.setValueAtTime(600, this.ctx.currentTime); gain.gain.setValueAtTime(0.05, this.ctx.currentTime); gain.gain.exponentialRampToValueAtTime(0.01, this.ctx.currentTime + 0.05); osc.start(); osc.stop(this.ctx.currentTime + 0.05); return; } 
        else { osc.type = 'square'; osc.frequency.setValueAtTime(800, this.ctx.currentTime); osc.frequency.exponentialRampToValueAtTime(100, this.ctx.currentTime + 0.05); }
        gain.gain.setValueAtTime(0.15, this.ctx.currentTime); gain.gain.exponentialRampToValueAtTime(0.01, this.ctx.currentTime + 0.1);
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

    ui: {
        screens: { 
            device: document.getElementById('screen-device'), login: document.getElementById('screen-login'), 
            instructions: document.getElementById('screen-instructions'), game: document.getElementById('screen-game'), 
            result: document.getElementById('screen-result') 
        },
        inputName: document.getElementById('player-name'), canvas: document.getElementById('game-canvas'),
        hudScore: document.getElementById('hud-score'), hudCombo: document.getElementById('hud-combo'), hudTimer: document.getElementById('hud-timer'),
        oppName: document.getElementById('opp-name'), oppScore: document.getElementById('opp-score'), oppAvatar: document.getElementById('opp-avatar'),
        touchOverlay: document.getElementById('touch-overlay'), pads: [document.getElementById('pad-0'), document.getElementById('pad-1'), document.getElementById('pad-2'), document.getElementById('pad-3')],
        globalHint: document.getElementById('global-hint'), hintWord: document.getElementById('hint-word'), hintMeaning: document.getElementById('hint-meaning')
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
        document.getElementById('btn-pc').addEventListener('click', () => { this.isMobile = false; this.setupDevice(); });
        document.getElementById('btn-mobile').addEventListener('click', () => { this.isMobile = true; this.setupDevice(); });
        document.getElementById('btn-login').addEventListener('click', () => this.handleLogin());
        document.getElementById('btn-start').addEventListener('click', () => this.startGame());
        document.getElementById('btn-download').addEventListener('click', () => this.downloadCert());
        document.getElementById('btn-restart').addEventListener('click', () => this.switchScreen('instructions'));

        document.querySelectorAll('.diff-btn').forEach(btn => {
            btn.addEventListener('click', (e) => {
                document.querySelectorAll('.diff-btn').forEach(b => b.classList.remove('active'));
                e.target.classList.add('active'); this.difficulty = e.target.dataset.diff;
            });
        });

        window.addEventListener('keydown', (e) => {
            if (e.key === 'Enter') {
                if (this.state === 'login') this.handleLogin();
                else if (this.state === 'instructions') this.startGame();
            }
            if (this.state !== 'playing' || e.repeat || this.isMobile) return;
            const lane = Config.keys.indexOf(e.key.toLowerCase());
            if (lane !== -1) { this.keysDown[lane] = true; this.handleInputDown(lane); }
        });
        window.addEventListener('keyup', (e) => {
            if (this.state !== 'playing' || this.isMobile) return;
            const lane = Config.keys.indexOf(e.key.toLowerCase());
            if (lane !== -1) { this.keysDown[lane] = false; this.handleInputUp(lane); }
        });

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
            document.querySelectorAll('.hidden-mobile').forEach(el => el.style.display = 'none');
        } else {
            this.ui.touchOverlay.classList.remove('mobile-mode');
            const kw = ['D', 'F', 'J', 'K'];
            this.ui.pads.forEach((pad, i) => pad.innerText = kw[i]);
        }
        this.switchScreen('login');
    },

    handleLogin() {
        const name = this.ui.inputName.value.trim().toUpperCase() || 'STUDENT_' + Math.floor(Math.random()*1000);
        this.playerName = name;
        LearningSystem.init(name);
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
        let tempStatsTracker = {}; // 用於生成期間模擬時間推進

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

                if (r < 15 && this.difficulty !== 'easy') {
                    type = 'hold'; duration = Math.floor(Math.random() * 600) + 400;
                    laneFreeTime[lane] = time + duration + 100;
                    baseScore += Math.floor(duration/100) * 20; 
                } else if (r < 30) {
                    type = 'gold'; baseScore *= 3;
                }

                // AI 適性化選字
                const vocab = LearningSystem.selectWord(time, tempStatsTracker);

                this.chart.push({
                    lane: lane, type: type, hitTime: time, spawnTime: time - fallDur,
                    duration: duration, active: true, isBeingHeld: false, holdTicks: 0,
                    vocab: vocab 
                });

                theoCombo++; this.theoMaxScore += baseScore + (theoCombo * 10);
            });

            time += Math.random() * (diffConfig.gapMax - diffConfig.gapMin) + diffConfig.gapMin;
        }
    },

    generateOpponents() {
        const tiers = [
            {n:"NOVICE", a:"🐣", p:0.15}, {n:"STUDENT", a:"👦", p:0.35}, 
            {n:"GEEK", a:"👓", p:0.60}, {n:"MASTER", a:"🔥", p:0.85}, {n:"GRANDMASTER", a:"👑", p:0.98}
        ];
        this.opponents = tiers.map(t => ({ name: t.n, avatar: t.a, targetScore: Math.floor(this.theoMaxScore * t.p) }));
        this.currentOppIdx = 0; this.displayOppScore = 0;
        this.updateOpponentUI();
    },

    updateOpponentUI() {
        if(this.currentOppIdx >= this.opponents.length) return;
        const opp = this.opponents[this.currentOppIdx];
        this.ui.oppName.innerText = opp.name; this.ui.oppAvatar.innerText = opp.avatar;
    },

    startGame() {
        if (!AudioEngine.ctx) AudioEngine.init();
        if (AudioEngine.ctx.state === 'suspended') AudioEngine.ctx.resume();

        this.switchScreen('game');
        LearningSystem.sessionStats = { totalNotes: 0, hitNotes: 0 };
        this.generateChart();
        this.generateOpponents();
        
        this.score = 0; this.combo = 0; this.maxCombo = 0;
        this.isFrenzy = false; document.body.classList.remove('frenzy-mode');
        this.particles = []; this.judgements = []; this.laneGlow.fill(0);
        this.activeHolds.fill(null); this.keysDown.fill(false);
        this.ui.globalHint.style.opacity = '0';
        
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

        let lowestY = -1000; let lowestVocab = null;

        for (let i = 0; i < this.chart.length; i++) {
            const note = this.chart[i];
            if (!note.active) continue;

            // 尋找最接近底線的音符，更新上方提示
            const progress = (this.gameTime - note.spawnTime) / DifficultySettings[this.difficulty].fallDur;
            const y = progress * (window.innerHeight * Config.hitLineOffset);
            if (y > lowestY && y < window.innerHeight && !note.isBeingHeld) {
                lowestY = y; lowestVocab = note.vocab;
            }

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

        // 更新頂部中文提示
        if (lowestVocab && lowestY > 0) {
            this.ui.hintWord.innerText = lowestVocab.word;
            this.ui.hintMeaning.innerText = lowestVocab.meaning;
            this.ui.globalHint.style.opacity = '1';
        } else {
            this.ui.globalHint.style.opacity = '0';
        }

        // 虛擬對手邏輯
        if (this.currentOppIdx < this.opponents.length) {
            const opp = this.opponents[this.currentOppIdx];
            const progress = Math.min(1, this.gameTime / Config.gameDuration);
            this.displayOppScore = Math.floor(opp.targetScore * Math.pow(progress, 0.9));
            this.ui.oppScore.innerText = this.displayOppScore.toString().padStart(7, '0');

            if (this.score > this.displayOppScore && this.score > 0) {
                this.currentOppIdx++;
                if (this.currentOppIdx < this.opponents.length) this.updateOpponentUI();
                else { this.ui.oppName.innerText = "VOCAB GOD"; this.ui.oppAvatar.innerText = "🌟"; this.ui.oppScore.innerText = "CLEARED"; this.ui.oppScore.style.color = "var(--perfect-color)"; }
            }
        }

        this.particles = this.particles.filter(p => p.life > 0);
        this.particles.forEach(p => { p.x += p.vx; p.y += p.vy; p.life -= 0.03; p.size *= 0.95; });
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
                if (this.gameTime - note.hitTime > -250 && delta < minDelta) { minDelta = delta; targetNote = note; }
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
        let vocabFeedback = null;
        
        if (noteData && noteData.vocab && !isHoldStart) {
            const isHit = type !== 'MISS';
            LearningSystem.updateFamiliarity(noteData.vocab.word, isHit);
            
            // 教育回饋：如果打錯，在軌道上浮現紅色的翻譯
            if (!isHit) vocabFeedback = `${noteData.vocab.word} = ${noteData.vocab.meaning}`;
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
            text: text, vocabText: vocabFeedback, color: color, 
            x: laneWidth * lane + laneWidth / 2, y: window.innerHeight * Config.hitLineOffset - 60, 
            life: 1.0, scale: (type === 'PERFECT' && !isHoldStart) ? 1.2 : 1.0 
        });
    },

    createExplosion(lane, color, count) {
        const w = window.innerWidth; const laneWidth = w / Config.lanes;
        const x = laneWidth * lane + laneWidth / 2; const y = window.innerHeight * Config.hitLineOffset;
        for (let i = 0; i < count; i++) {
            const angle = Math.random() * Math.PI * 2; const speed = Math.random() * 8 + 2;
            this.particles.push({ x: x, y: y, vx: Math.cos(angle) * speed, vy: Math.sin(angle) * speed, color: color, life: 1.0, size: Math.random() * 5 + 3 });
        }
    },

    updateHUD() {
        this.ui.hudScore.innerText = this.score.toString().padStart(7, '0');
        this.ui.hudCombo.innerText = this.combo > 3 ? `${this.combo}x` : '';
        this.ui.hudCombo.style.transform = `scale(${1 + Math.min(this.combo/100, 0.5)})`;
        setTimeout(()=> this.ui.hudCombo.style.transform = 'scale(1)', 100);
        this.ui.hudCombo.style.textShadow = this.combo > 20 ? '0 0 20px var(--perfect-color)' : 'none';
        if(this.isFrenzy) this.ui.hudScore.style.color = 'var(--frenzy-color)';
    },

    draw() {
        const ctx = this.ctx; const w = window.innerWidth; const h = window.innerHeight;
        const hitY = h * Config.hitLineOffset; const laneWidth = w / Config.lanes; 
        const noteHeight = Math.max(30, h * 0.04); 
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

        // Draw Holds
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

        // Draw Notes & Words
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
            let textColor = '#000000'; 
            
            if (note.type === 'gold') { noteColor = 'var(--gold-color)'; } 
            else if (note.type === 'hold') { noteColor = '#ffffff'; textColor = 'var(--hold-color)'; } 
            
            ctx.fillStyle = noteColor; ctx.shadowBlur = this.isFrenzy || note.type === 'gold' ? 15 : 0; 
            ctx.shadowColor = noteColor; ctx.fill(); ctx.shadowBlur = 0; 

            // Text fitting
            ctx.fillStyle = textColor; ctx.textAlign = 'center'; ctx.textBaseline = 'middle';
            let fontSize = Math.max(14, w * 0.03); ctx.font = `bold ${fontSize}px sans-serif`;
            let textWidth = ctx.measureText(note.vocab.word).width;
            if (textWidth > noteWidth - 10) {
                fontSize = fontSize * ((noteWidth - 10) / textWidth); ctx.font = `bold ${fontSize}px sans-serif`;
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
            const baseSize = Math.max(20, w * 0.03) * j.scale; ctx.font = `900 italic ${baseSize}px sans-serif`;
            ctx.shadowBlur = 10; ctx.shadowColor = ctx.fillStyle; 
            ctx.fillText(j.text, j.x, j.y); 
            
            // 教育回饋文字 (Miss 時)
            if (j.vocabText) {
                ctx.fillStyle = 'var(--miss-color)'; ctx.shadowColor = 'var(--miss-color)';
                ctx.font = `bold ${Math.max(16, w * 0.025)}px sans-serif`;
                ctx.fillText(j.vocabText, j.x, j.y + baseSize + 10);
            }
            ctx.shadowBlur = 0;
        });
        ctx.globalAlpha = 1.0;
    },

    endGame() {
        this.state = 'result';
        cancelAnimationFrame(this.animFrameId);
        document.body.classList.remove('frenzy-mode');
        this.ui.hudTimer.style.color = '#fff';
        this.ui.hudScore.style.color = '#fff';
        
        this.buildLearningReport();
        this.switchScreen('result');
    },

    buildLearningReport() {
        // 更新 UI 數據
        document.getElementById('res-score').innerText = this.score;
        document.getElementById('res-combo').innerText = this.maxCombo;
        
        let acc = LearningSystem.sessionStats.totalNotes > 0 
            ? Math.round((LearningSystem.sessionStats.hitNotes / LearningSystem.sessionStats.totalNotes) * 100) 
            : 0;
        document.getElementById('res-acc').innerText = acc;

        // 生成兩欄清單
        const weakList = LearningSystem.getSortedWords('weak');
        const strongList = LearningSystem.getSortedWords('strong');

        const createHTML = (item) => {
            let famPercent = Math.round(item.familiarity * 100);
            let famClass = LearningSystem.getFamClass(item.familiarity);
            return `
            <div class="stat-row">
                <div>
                    <span class="stat-word">${item.word}</span> <span style="color:#aaa; font-size:12px;">(${item.meaning})</span>
                </div>
                <div style="display:flex; align-items:center; gap:8px;">
                    <span class="${famClass}" style="font-size:12px; font-weight:bold;">${famPercent}%</span>
                    <div class="fam-bar-container"><div class="fam-bar ${famClass}" style="width: ${famPercent}%;"></div></div>
                </div>
            </div>`;
        };

        document.getElementById('list-weak').innerHTML = weakList.map(createHTML).join('');
        document.getElementById('list-strong').innerHTML = strongList.map(createHTML).join('');
    },

    downloadCert() {
        const cvs = document.getElementById('cert-canvas'); const ctx = cvs.getContext('2d');
        const w = cvs.width; const h = cvs.height;

        // 背景
        ctx.fillStyle = '#09090b'; ctx.fillRect(0, 0, w, h);
        const grad = ctx.createLinearGradient(0,0,w,h);
        grad.addColorStop(0, 'rgba(0, 255, 204, 0.05)'); grad.addColorStop(1, 'rgba(168, 85, 247, 0.05)');
        ctx.fillStyle = grad; ctx.fillRect(0,0,w,h);

        // 邊框
        ctx.strokeStyle = '#00ffcc'; ctx.lineWidth = 15; ctx.strokeRect(40, 40, w - 80, h - 80);
        ctx.strokeStyle = '#3b82f6'; ctx.lineWidth = 4; ctx.strokeRect(65, 65, w - 130, h - 130);

        // 標題
        ctx.textAlign = 'center'; ctx.textBaseline = 'middle';
        ctx.fillStyle = '#00ffcc'; ctx.font = '900 italic 70px sans-serif';
        ctx.fillText('EDU-STRIKE PRO: LEARNING ANALYTICS', w/2, 140);
        
        // 學生資料
        ctx.fillStyle = '#fff'; ctx.font = 'bold 50px sans-serif';
        ctx.fillText(`STUDENT: ${this.playerName}`, w/2, 230);
        
        ctx.fillStyle = '#aaa'; ctx.font = '30px sans-serif';
        const d = new Date(); const dateStr = `${d.getFullYear()}/${String(d.getMonth()+1).padStart(2,'0')}/${String(d.getDate()).padStart(2,'0')} ${String(d.getHours()).padStart(2,'0')}:${String(d.getMinutes()).padStart(2,'0')}`;
        ctx.fillText(`SESSION TIME: ${dateStr} | MODE: ${this.isMobile?'MOBILE':'PC'}`, w/2, 290);

        // 核心數據區
        ctx.fillStyle = 'rgba(255,255,255,0.05)'; ctx.fillRect(100, 340, w-200, 200);
        ctx.fillStyle = '#fbbf24'; ctx.font = 'bold 55px monospace';
        ctx.fillText(`SCORE: ${this.score.toString().padStart(7,'0')}`, w/2, 400);
        
        let acc = document.getElementById('res-acc').innerText;
        ctx.fillStyle = '#22c55e'; ctx.font = 'bold 45px sans-serif';
        ctx.fillText(`ACCURACY: ${acc}%  |  MAX COMBO: ${this.maxCombo}`, w/2, 480);

        // 分析欄位標題
        ctx.textAlign = 'left';
        ctx.fillStyle = '#ef4444'; ctx.font = 'bold 40px sans-serif'; ctx.fillText('🔴 CRITICAL WEAKNESS', 120, 620);
        ctx.fillStyle = '#22c55e'; ctx.font = 'bold 40px sans-serif'; ctx.fillText('🟢 MASTERED WORDS', w/2 + 20, 620);

        // 繪製單字列表
        ctx.font = '35px sans-serif';
        const weakList = LearningSystem.getSortedWords('weak');
        const strongList = LearningSystem.getSortedWords('strong');

        for(let i=0; i<5; i++) {
            let y = 690 + i * 55;
            if(weakList[i]) {
                ctx.fillStyle = '#fff'; ctx.fillText(`${weakList[i].word}`, 120, y);
                ctx.fillStyle = '#888'; ctx.fillText(`(${weakList[i].meaning})`, 320, y);
            }
            if(strongList[i]) {
                ctx.fillStyle = '#fff'; ctx.fillText(`${strongList[i].word}`, w/2 + 20, y);
                ctx.fillStyle = '#888'; ctx.fillText(`(${strongList[i].meaning})`, w/2 + 220, y);
            }
        }

        ctx.textAlign = 'center'; ctx.fillStyle = '#555'; ctx.font = 'bold 24px sans-serif';
        ctx.fillText('POWERED BY ADAPTIVE LEARNING ALGORITHM', w/2, h - 90);

        // 下載
        const link = document.createElement('a'); link.download = `EduStrike_Pro_${this.playerName}.png`;
        link.href = cvs.toDataURL('image/png'); link.click();
    }
};

window.onload = () => Engine.init();

</script>
</body>
</html>
