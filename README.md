<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<title>FATAL ERROR: v9.4 AUDIO ENGINEER</title>
<style>
  @import url('https://fonts.googleapis.com/css2?family=VT323&display=swap');

  body {
    margin: 0; background: #050505; color: #00ffff;
    font-family: 'VT323', monospace; text-align: center;
    overflow: hidden; user-select: none;
    display: flex; justify-content: center; align-items: center; height: 100vh;
  }

  /* --- INTERFACE --- */
  #interface-container {
    position: relative; display: flex; flex-direction: row;
    width: 1100px; height: 450px;
    border: 4px solid #333; box-shadow: 0 0 20px #00ffff; background: #000;
  }
  #game-wrapper {
    position: relative; width: 800px; height: 100%;
    border-right: 4px solid #333; overflow: hidden;
  }
  canvas { display: block; background: #111; width: 100%; height: 100%; }

  /* --- SIDEBAR --- */
  #console-sidebar {
    width: 300px; height: 100%; background: #000;
    display: flex; flex-direction: column; padding: 10px;
    box-sizing: border-box; text-align: left; z-index: 1;
  }
  #console-area {
    width: 100%; flex-grow: 1; overflow-y: auto;
    border-bottom: 2px solid #00ffff; margin-bottom: 10px; font-size: 16px;
  }
  #console-area::-webkit-scrollbar { width: 8px; }
  #console-area::-webkit-scrollbar-track { background: #111; }
  #console-area::-webkit-scrollbar-thumb { background: #00ffff; }
  #console-input {
    background: transparent; border: none; color: white;
    font-family: 'VT323', monospace; font-size: 18px; width: 100%; outline: none;
  }

  /* --- OVERLAYS --- */
  .overlay-screen {
    position: absolute; top: 0; left: 0; width: 100%; height: 100%;
    background: rgba(5, 5, 5, 0.98);
    display: flex; flex-direction: column; justify-content: center; align-items: center;
    z-index: 100;
  }
  .hidden { display: none !important; }
  
  h1 { font-size: 60px; margin: 0; text-shadow: 4px 4px #ff0055; animation: glitchText 2s infinite; }
  h2 { color: #ffff00; margin-bottom: 15px; }

  .btn {
    background: transparent; border: 2px solid #00ffff; color: #00ffff;
    font-family: 'VT323', monospace; font-size: 24px; padding: 5px 30px; margin: 5px;
    cursor: pointer; transition: 0.2s; width: 250px; pointer-events: auto;
  }
  .btn:hover { background: #00ffff; color: #000; box-shadow: 0 0 15px #00ffff; }

  /* --- AUDIO SLIDERS --- */
  .slider-container {
      display: flex; align-items: center; justify-content: space-between;
      width: 300px; margin: 10px 0; font-size: 22px; color: #fff;
  }
  input[type=range] {
      -webkit-appearance: none; width: 180px; background: transparent;
  }
  input[type=range]::-webkit-slider-thumb {
      -webkit-appearance: none; height: 20px; width: 10px; background: #00ffff;
      cursor: pointer; margin-top: -8px; box-shadow: 0 0 5px #fff;
  }
  input[type=range]::-webkit-slider-runnable-track {
      width: 100%; height: 4px; background: #333; border: 1px solid #555;
  }

  /* --- GAME UI ELEMENTS --- */
  .stats-grid {
      display: grid; grid-template-columns: 1fr 1fr; gap: 10px; text-align: left;
      font-size: 18px; color: #fff; margin-bottom: 20px; border: 1px solid #333;
      padding: 15px; background: #111; width: 500px;
  }
  .stat-label { color: #888; }
  .stat-val { color: #00ff41; font-weight: bold; text-align: right; }

  .tutorial-box {
      width: 750px; text-align: left; font-size: 16px; color: #ddd; line-height: 1.4;
      border: 1px solid #00ffff; padding: 20px; background: rgba(0, 20, 0, 0.8);
      margin-bottom: 15px; height: 320px; overflow-y: auto;
      display: grid; grid-template-columns: 1fr 1fr; gap: 20px;
  }
  .tut-column { display: flex; flex-direction: column; }
  .tut-header { color: #ffff00; font-weight: bold; margin-top: 10px; display:block; border-bottom: 1px solid #333; margin-bottom: 5px; font-size: 18px; }
  .tut-item { margin-bottom: 8px; }
  .key-bind { color: #00ffff; font-weight: bold; }
  
  #stats-display { position: absolute; top: 10px; right: 10px; color: #00ff41; font-size: 20px; z-index: 25; }
  #combo-display { position: absolute; top: 40px; right: 10px; color: #ffff00; font-size: 24px; z-index: 25; font-weight: bold; display: none; }
  #combo-charge { position: absolute; top: 70px; right: 10px; color: #00ffff; font-size: 16px; z-index: 25; }
  #combo-bar { position: absolute; top: 90px; right: 10px; width: 100px; height: 8px; background:#333; border:1px solid #00ffff; z-index: 25; }
  #combo-fill { height: 100%; background: linear-gradient(90deg, #00ffff, #ff00ff); transition: width 0.1s; }
  #admin-indicator { position: absolute; top: 10px; left: 10px; color: #ff0000; font-weight: bold; font-size: 16px; display: none; z-index: 30; animation: blink 1s infinite; }

  .log-msg { margin: 2px 0; opacity: 0.9; border-bottom: 1px solid #111; min-height: 20px; }
  .sys { color: #ffff00; } .err { color: #ff0055; } .root { color: #00ff41; }
  .buff { color: #00ffff; font-weight: bold; } .admin { color: #ff0000; font-weight: bold; text-shadow: 0 0 5px red; }
  .player-cmd { color: #0088ff; font-weight: bold; font-style: italic; }

  /* ANIMATIONS */
  @keyframes glitchText {
    0% { transform: translate(0); } 20% { transform: translate(-2px, 2px); }
    40% { transform: translate(-2px, -2px); } 60% { transform: translate(2px, 2px); }
    80% { transform: translate(2px, -2px); } 100% { transform: translate(0); }
  }
  @keyframes blink { 50% { opacity: 0; } }
  .shake { animation: shake 0.3s cubic-bezier(.36,.07,.19,.97) both; }
  @keyframes shake { 10%, 90% { transform: translate3d(-2px, -1px, 0); } 20%, 80% { transform: translate3d(2px, 2px, 0); } 30%, 50%, 70% { transform: translate3d(-4px, -4px, 0); } 40%, 60% { transform: translate3d(4px, 4px, 0); } }
  
  /* Character glow effects */
  .player-glow { box-shadow: 0 0 20px #00ffff, inset 0 0 10px rgba(0, 255, 255, 0.3); }
  .enemy-glow { box-shadow: 0 0 20px #ff0055, inset 0 0 10px rgba(255, 0, 85, 0.3); }
  .god-glow { box-shadow: 0 0 30px #ffffff, 0 0 60px #ffff00; animation: godPulse 0.5s infinite; }
  
  @keyframes godPulse {
    0%, 100% { box-shadow: 0 0 20px #ffffff, 0 0 40px #ffff00; }
    50% { box-shadow: 0 0 40px #ffffff, 0 0 80px #ffff00; }
  }
  
  /* Attack slash effect */
  .slash-effect {
    animation: slashAnim 0.2s ease-out;
  }
  @keyframes slashAnim {
    0% { transform: scale(0.5); opacity: 1; }
    100% { transform: scale(1.5); opacity: 0; }
  }
  
  /* Laser beam animation */
  .laser-beam {
    animation: laserFlicker 0.1s infinite;
  }
  @keyframes laserFlicker {
    0%, 100% { opacity: 1; }
    50% { opacity: 0.8; }
  }
  
  .scanlines {
    position: absolute; top: 0; left: 0; width: 100%; height: 100%;
    background: linear-gradient(to bottom, rgba(255,255,255,0), rgba(255,255,255,0) 50%, rgba(0,0,0,0.2) 50%, rgba(0,0,0,0.2));
    background-size: 100% 4px; pointer-events: none; z-index: 10;
  }
  
  /* LOADING SPECIFIC */
  #cutscene-overlay { background: #000; z-index: 200; padding: 0; text-align: center; }
  .boot-line { color: #00ff41; font-size: 24px; text-shadow: 0 0 5px #00ff41; margin-bottom: 10px; font-family: 'VT323', monospace; }
  .error-window { width: 400px; height: 200px; background: #000; border: 2px solid #ff0055; box-shadow: 15px 15px 0px #220000; display: flex; flex-direction: column; animation: popIn 0.1s; z-index: 300; }
  .error-header { background: #ff0055; color: #fff; padding: 5px; font-weight: bold; text-align: left; padding-left: 10px; }
  .error-content { flex-grow: 1; display: flex; flex-direction: column; justify-content: center; align-items: center; color: #ff0055; }
  .error-title { font-size: 40px; margin: 0; text-shadow: 2px 2px #fff; }
  .studio-logo { font-size: 50px; color: #00ffff; text-shadow: 0 0 20px #00ffff; animation: fadeIn 2s; }
  .presents-text { font-size: 24px; color: #fff; margin-top: 10px; letter-spacing: 5px; animation: fadeIn 3s; }
  @keyframes fadeIn { 0% { opacity: 0; } 100% { opacity: 1; } }
  @keyframes popIn { 0% { transform: scale(0); } 100% { transform: scale(1); } }
</style>
</head>

<body>

<div id="interface-container">
  <div id="game-wrapper">
    <div id="stats-display">
      <span id="streak-hud">0</span> | GLITCH: <span id="glitch-hud">0%</span>
      <span id="combo-charge"></span>
      <div id="combo-bar"><div id="combo-fill" style="width: 0%"></div></div>
      <button id="pause-btn" onclick="togglePause()" style="margin-left:15px; background:#111; border:1px solid #00ff41; color:#00ff41; padding:4px 10px; cursor:pointer; font-family:'VT323',monospace; font-size:16px;">⏸ PAUSE</button>
    </div>
    <div id="combo-display"><span id="combo-count">0</span> HITS</div>
    <div id="admin-indicator">[ ROOT ACCESS ]</div>

    <div id="cutscene-overlay" class="overlay-screen">
      <div id="boot-log" class="boot-line"></div>
      <button id="skip-boot-btn" onclick="skipBootSequence()" style="position:absolute; bottom:20px; right:20px; background:#111; border:1px solid #00ff41; color:#00ff41; padding:8px 16px; cursor:pointer; font-family:'VT323',monospace; font-size:18px;">[ SKIP BOOT ]</button>
    </div>

    <div id="main-menu" class="overlay-screen hidden">
      <h1>FATAL ERROR</h1>
      <div style="color:#ff0055; letter-spacing: 4px; margin-bottom: 20px;">v9.4 AUDIO ENGINEER</div>
      <button class="btn" onmouseenter="sfx.hover()" onclick="initGameAndAudio()">INITIALIZE (PLAY)</button>
      <button class="btn" onmouseenter="sfx.hover()" onclick="switchScreen('tutorial-menu')">DATABASE (FULL)</button>
      <button class="btn" onmouseenter="sfx.hover()" onclick="showStats()">PLAYER STATS</button>
      <button class="btn" onmouseenter="sfx.hover()" onclick="switchScreen('options-menu')">SYSTEM CONFIG</button>
    </div>

    <div id="options-menu" class="overlay-screen hidden">
      <h2>SYSTEM CONFIGURATION</h2>
      
      <div class="slider-container">
          <span>MASTER VOL</span>
          <input type="range" min="0" max="100" value="50" oninput="sfx.setVolume('master', this.value)">
      </div>
      <div class="slider-container">
          <span>MUSIC VOL</span>
          <input type="range" min="0" max="100" value="60" oninput="sfx.setVolume('music', this.value)">
      </div>
      <div class="slider-container">
          <span>SFX VOL</span>
          <input type="range" min="0" max="100" value="80" oninput="sfx.setVolume('sfx', this.value)">
      </div>

      <div style="margin-top: 20px; font-size: 14px; color: #888;">RENDERER: CANVAS 2D | AUDIO: WEB AUDIO API</div>
      
      <button class="btn" style="margin-top: 20px;" onmouseenter="sfx.hover()" onclick="switchScreen('main-menu')">BACK</button>
      <button class="btn" style="border-color:red; color:red;" onclick="clearSave()">WIPE DATA</button>
    </div>

    <div id="tutorial-menu" class="overlay-screen hidden">
      <h2 style="color:#ffff00">SYSTEM DATABASE</h2>
      <div class="tutorial-box">
          <div class="tut-column">
              <span class="tut-header">/// MOVEMENT & CORE</span>
              <div class="tut-item"><span class="key-bind">WASD:</span> Standard Movement.</div>
              <div class="tut-item"><span class="key-bind">W or SPACE:</span> Jump / Double Jump.</div>
              <div class="tut-item"><span class="key-bind">SHIFT:</span> Phase Dash. Cooldown: 0.75s.</div>
              
              <span class="tut-header">/// OFFENSIVE PROTOCOLS</span>
              <div class="tut-item"><span class="key-bind">L-CLICK:</span> Light Attack. Fast, builds combo chain.</div>
              <div class="tut-item"><span class="key-bind">R-CLICK:</span> Heavy Attack. High damage, Guard Break.</div>
              <div class="tut-item"><span class="key-bind">S + R-CLICK:</span> <span style="color:#00ffff; font-weight:bold;">KERNEL PANIC</span>. Expanding Ring AOE.</div>
          </div>
          <div class="tut-column">
              <span class="tut-header">/// WEAPONRY (Q to Swap)</span>
              <div class="tut-item"><span class="key-bind">KATANA:</span> High melee damage, wide arc. Good for parrying.</div>
              <div class="tut-item"><span class="key-bind">PISTOL:</span> Ranged chip damage. Light = Single Shot. Heavy = Burst Fire.</div>
              
              <span class="tut-header">/// ADVANCED COMBAT</span>
              <div class="tut-item"><span class="key-bind">HOLD BACK (A/D):</span> Block. Reduces Incoming Dmg by 80%.</div>
              <div class="tut-item"><span class="key-bind">E (EX MOVE):</span> Costs 25 Meter. Cooldown 1s.</div>
              <div class="tut-item"><span class="key-bind">Z (ULTIMATE):</span> Overclock. Costs 100 Meter. Restores HP, Boosts Speed/Dmg.</div>
              
              <span class="tut-header">/// PASSIVES</span>
              <div class="tut-item"><span style="color:#00ff41">LIFESTEAL:</span> Heal 5% of all damage dealt.</div>
              <div class="tut-item"><span style="color:#ffff00">COMBO:</span> +2% Damage per hit (Stacks infinitely).</div>
          </div>
      </div>
      <button class="btn" onmouseenter="sfx.hover()" onclick="switchScreen('main-menu')">BACK</button>
    </div>

    <div id="stats-menu" class="overlay-screen hidden">
      <h2>PLAYER'S STATS</h2>
      <div class="stats-grid">
          <span class="stat-label">Total Wins:</span> <span class="stat-val" id="stat-wins">0</span>
          <span class="stat-label">Win/Loss Ratio:</span> <span class="stat-val" id="stat-ratio">0.0</span>
          <span class="stat-label">Consecutive Streak:</span> <span class="stat-val" id="stat-streak">0</span>
          <span class="stat-label">Total Losses:</span> <span class="stat-val" style="color:#ff0055" id="stat-losses">0</span>
          <span class="stat-label">No Hit Wins:</span> <span class="stat-val" style="color:#ffff00" id="stat-nohit">0</span>
          <span class="stat-label">Terminal Wins:</span> <span class="stat-val" id="stat-term">0</span>
          <span class="stat-label">Avg Dmg Dealt:</span> <span class="stat-val" id="stat-avg-dealt">0</span>
          <span class="stat-label">Highest Dmg Dealt:</span> <span class="stat-val" id="stat-high-dealt">0</span>
          <span class="stat-label">Highest Dmg Taken:</span> <span class="stat-val" style="color:#ff0055" id="stat-high-taken">0</span>
      </div>
      <button class="btn" onmouseenter="sfx.hover()" onclick="switchScreen('main-menu')">BACK</button>
    </div>

    <div id="pause-menu" class="overlay-screen hidden">
      <h2 style="color:#ffff00">/// SYSTEM PAUSED ///</h2>
      <button class="btn" onmouseenter="sfx.hover()" onclick="resumeGame()">RESUME</button>
      <button class="btn" onmouseenter="sfx.hover()" onclick="switchScreen('options-menu'); gameState='paused'">SYSTEM CONFIG</button>
      <button class="btn" onmouseenter="sfx.hover()" onclick="restartGame()">RESTART</button>
      <button class="btn" onmouseenter="sfx.hover()" onclick="switchScreen('main-menu'); gameState='menu'">MAIN MENU</button>
    </div>

    <canvas id="game" width="800" height="450"></canvas>
    <div class="scanlines"></div>
  </div>

  <div id="console-sidebar">
    <div id="console-area" onclick="document.getElementById('console-input').focus()">
      <div id="logs">
        <div class="log-msg sys">> BIOS DATE 01/01/2099</div>
        <div class="log-msg root">> SYSTEM CHECK... OK.</div>
        <div class="log-msg root">> WAITING FOR INPUT...</div>
      </div>
    </div>
    <div style="display: flex; padding: 5px;">
      <span style="color:#00ffff;">>_&nbsp;</span>
      <input type="text" id="console-input" autocomplete="off">
    </div>
  </div>
</div>

<script>
// ================== AUDIO MIXER ENGINE (UPDATED v9.4) ==================
const sfx = {
    ctx: null,
    masterGain: null,
    musicGain: null,
    sfxGain: null,
    
    // Sequencer State
    isPlaying: false,
    currentTrack: null,
    tempo: 120,
    beat: 0,
    nextNoteTime: 0,
    timerID: null,
    
    // Volume Settings (0-1)
    vols: { master: 0.5, music: 0.6, sfx: 0.8 },

    init: function() {
        if (!this.ctx) {
            window.AudioContext = window.AudioContext || window.webkitAudioContext;
            this.ctx = new AudioContext();
            
            // Create Mixer Nodes
            this.masterGain = this.ctx.createGain();
            this.musicGain = this.ctx.createGain();
            this.sfxGain = this.ctx.createGain();

            // Connect Mixer: Source -> TypeGain -> MasterGain -> Destination
            this.musicGain.connect(this.masterGain);
            this.sfxGain.connect(this.masterGain);
            this.masterGain.connect(this.ctx.destination);

            this.applyVolumes();
        }
        if (this.ctx.state === 'suspended') {
            this.ctx.resume();
        }
    },

    setVolume: function(type, val) {
        this.vols[type] = val / 100;
        if(this.ctx) this.applyVolumes();
    },

    applyVolumes: function() {
        this.masterGain.gain.setTargetAtTime(this.vols.master, this.ctx.currentTime, 0.1);
        this.musicGain.gain.setTargetAtTime(this.vols.music, this.ctx.currentTime, 0.1);
        this.sfxGain.gain.setTargetAtTime(this.vols.sfx, this.ctx.currentTime, 0.1);
    },

    // --- SYNTHESIZERS ---
    // Standard Oscillator (Beeps, Tones)
    osc: function(freq, type, dur, vol = 0.5, slideTo = null, delay = 0, isMusic = false) {
        if(!this.ctx) return;
        const start = this.ctx.currentTime + delay;
        const o = this.ctx.createOscillator();
        const g = this.ctx.createGain();
        o.type = type;
        o.frequency.setValueAtTime(freq, start);
        if(slideTo) o.frequency.exponentialRampToValueAtTime(slideTo, start + dur);
        
        g.gain.setValueAtTime(vol, start);
        g.gain.exponentialRampToValueAtTime(0.001, start + dur);
        
        o.connect(g);
        g.connect(isMusic ? this.musicGain : this.sfxGain);
        o.start(start); o.stop(start + dur);
    },

    // Noise Generator (Explosions, Snares)
    noise: function(dur, vol, filterType = null, freq = 1000, isMusic = false) {
        if(!this.ctx) return;
        const bufSize = this.ctx.sampleRate * dur;
        const buf = this.ctx.createBuffer(1, bufSize, this.ctx.sampleRate);
        const data = buf.getChannelData(0);
        for(let i=0; i<bufSize; i++) data[i] = Math.random() * 2 - 1;

        const src = this.ctx.createBufferSource();
        src.buffer = buf;
        const g = this.ctx.createGain();
        g.gain.setValueAtTime(vol, this.ctx.currentTime);
        g.gain.exponentialRampToValueAtTime(0.01, this.ctx.currentTime + dur);

        if(filterType) {
            const f = this.ctx.createBiquadFilter();
            f.type = filterType;
            f.frequency.setValueAtTime(freq, this.ctx.currentTime);
            if(filterType === 'lowpass') f.frequency.exponentialRampToValueAtTime(100, this.ctx.currentTime + dur);
            src.connect(f); f.connect(g);
        } else {
            src.connect(g);
        }
        g.connect(isMusic ? this.musicGain : this.sfxGain);
        src.start();
    },

    // Kick Drum Synth (Punchy)
    kick: function(vol = 1.0) {
        if(!this.ctx) return;
        const t = this.ctx.currentTime;
        const osc = this.ctx.createOscillator();
        const gain = this.ctx.createGain();
        osc.frequency.setValueAtTime(150, t);
        osc.frequency.exponentialRampToValueAtTime(0.01, t + 0.5);
        gain.gain.setValueAtTime(vol, t);
        gain.gain.exponentialRampToValueAtTime(0.01, t + 0.5);
        osc.connect(gain); gain.connect(this.musicGain);
        osc.start(t); osc.stop(t + 0.5);
    },

    // --- MUSIC SEQUENCER ---
    playMusic: function(trackName) {
        if (this.currentTrack === trackName) return;
        if (this.timerID) clearTimeout(this.timerID);
        this.currentTrack = trackName;
        this.beat = 0;
        this.nextNoteTime = this.ctx.currentTime;
        this.isPlaying = true;
        this.scheduler();
    },

    stopMusic: function() {
        this.isPlaying = false;
        this.currentTrack = null;
        if (this.timerID) clearTimeout(this.timerID);
    },

    scheduler: function() {
        if(!this.isPlaying) return;
        while (this.nextNoteTime < this.ctx.currentTime + 0.1) {
            this.playBeat(this.currentTrack, this.beat);
            // Advance beat
            const secondsPerBeat = 60.0 / this.tempo; 
            // 16th notes = divide by 4
            this.nextNoteTime += 0.25 * secondsPerBeat; 
            this.beat = (this.beat + 1) % 32; // 32 step loop
        }
        this.timerID = setTimeout(() => this.scheduler(), 25);
    },

    playBeat: function(track, step) {
        // === LOBBY: "ETHEREAL NETWORK" (Requested: God Music as Full Song) ===
        if (track === 'lobby') {
            this.tempo = 100; // Moderate pace
            
            // Rapid Arpeggio (F Major 7 - God Theme)
            // Pattern: Root - min3 - 5 - 8
            const arp = [349, 440, 523, 659, 698, 659, 523, 440];
            let note = arp[step % 8];
            if(step % 16 > 8) note *= 0.5; // Octave shift for variation
            this.osc(note, 'sine', 0.3, 0.15, null, 0, true);

            // Deep Bass Drone (Instrumental feeling)
            if (step % 16 === 0) this.osc(87.31, 'triangle', 4.0, 0.2, null, 0, true); // F2
            
            // Shimmering Highs
            if (step % 4 === 0) this.noise(0.2, 0.05, 'highpass', 8000, true);
        }

        // === BOSS / COMBAT TRACKS ===
        else if (track === 'battle_boss' || track === 'battle_god') {
            this.tempo = (track === 'battle_god') ? 140 : 125; // Faster for God

            // IMPACTFUL KICK DRUM (Four on the floor)
            if (step % 4 === 0) this.kick(0.9);

            // Industrial Bass
            if (step % 4 === 2) {
                let bassFreq = (track === 'battle_god') ? 43 : 55; // Lower for God
                this.osc(bassFreq, 'sawtooth', 0.15, 0.4, null, 0, true);
            }

            // Hi-Hats
            if (step % 2 === 0) this.noise(0.05, 0.1, 'highpass', 4000, true);

            // Chaotic melody for Boss
            if (step % 8 === 7) this.osc(880, 'square', 0.1, 0.1, 440, 0, true);
        }

        // === STANDARD COMBAT ===
        else if (track === 'battle_1') {
            this.tempo = 110;
            if (step % 4 === 0) this.osc(110, 'square', 0.1, 0.2, null, 0, true); // Bass
            if (step % 8 === 4) this.noise(0.1, 0.2, 'bandpass', 1500, true); // Snare
            if (step % 16 === 0) this.osc(440, 'triangle', 0.5, 0.1, null, 0, true); // Lead
        }
    },

    // --- SFX LIBRARY ---
    bootHum: function() { 
        // HDD Spin up sound
        this.osc(50, 'sawtooth', 3.0, 0.2, 150); 
        this.noise(3.0, 0.1, 'lowpass', 200); 
    },
    bootBeep: function() { this.osc(800, 'square', 0.1, 0.2); },
    
    death: function() {
        this.stopMusic();
        this.osc(300, 'sawtooth', 2.0, 0.4, 10);
        this.noise(2.0, 0.5, 'lowpass', 2000);
    },

    keystroke: function() {
        if(!this.ctx) return;
        let detune = (Math.random() - 0.5) * 200;
        this.noise(0.03, 0.1, 'highpass', 2000);
        this.osc(800 + detune, 'sine', 0.03, 0.05);
    },
    
    type: function(cat) {
        if(!this.ctx) return;
        if (cat === 'err') this.osc(150, 'sawtooth', 0.05, 0.1); 
        else if (cat === 'admin' || cat === 'root' || cat === 'buff') {
            this.osc(1200, 'sine', 0.05, 0.1);
            setTimeout(() => this.osc(1800, 'sine', 0.05, 0.08), 30);
        } else if (cat === 'player-cmd') this.noise(0.02, 0.1, 'bandpass', 1000);
        else this.osc(800, 'square', 0.02, 0.05); 
    },

    hover: function() { this.osc(1200, 'sine', 0.05, 0.05); },
    menuBlip: function() { 
        this.osc(800, 'sine', 0.1, 0.3); 
        setTimeout(() => this.osc(1200, 'sine', 0.1, 0.2), 50);
    },
    jump: function() { this.osc(150, 'square', 0.2, 0.3, 400); },
    doubleJump: function() { this.osc(600, 'sine', 0.25, 0.3, 1200); this.osc(800, 'sawtooth', 0.1, 0.1, 1500); },
    dash: function() { this.noise(0.3, 0.4, 'bandpass', 600); },
    attackLight: function() { this.noise(0.15, 0.4, 'bandpass', 1200); },
    attackHeavy: function() { this.osc(100, 'sawtooth', 0.3, 0.4, 50); this.noise(0.3, 0.3, 'lowpass', 800); },
    shot: function() { this.osc(800, 'square', 0.15, 0.25, 100); this.osc(1200, 'sawtooth', 0.1, 0.2, 50); },
    hit: function() { this.noise(0.1, 0.5); this.osc(100, 'sawtooth', 0.1, 0.3, 50); },
    block: function() { this.osc(300, 'square', 0.1, 0.4); this.osc(305, 'square', 0.1, 0.4); },
    explosion: function() { this.osc(60, 'sawtooth', 0.8, 0.6, 10); this.noise(0.6, 0.6, 'lowpass', 1000); },
    laser: function() { this.osc(2000, 'sawtooth', 0.3, 0.5, 100); this.noise(0.3, 0.5, 'highpass', 5000); },
    slam: function() { this.osc(50, 'square', 0.5, 0.6, 20); this.noise(0.4, 0.7, 'lowpass', 300); },
    rapidAttack: function() { this.osc(800, 'square', 0.08, 0.3); setTimeout(() => this.osc(1000, 'square', 0.08, 0.3), 50); },
    warp: function() { this.noise(0.2, 0.5, 'bandpass', 2000); this.osc(400, 'sine', 0.2, 0.3, 800); },
    win: function() {
        let notes = [440, 554, 659, 880];
        notes.forEach((n, i) => setTimeout(() => this.osc(n, 'square', 0.3, 0.3), i*100));
        setTimeout(() => this.osc(440, 'triangle', 1.0, 0.3), 400);
    }
};

// ================== CORE SETUP ==================
const canvas = document.getElementById("game");
const ctx = canvas.getContext("2d");
const consoleLogs = document.getElementById("logs");
const consoleArea = document.getElementById("console-area");
const consoleInput = document.getElementById("console-input");
const streakHud = document.getElementById("streak-hud");
const glitchHud = document.getElementById("glitch-hud");
const comboHud = document.getElementById("combo-display");
const comboCount = document.getElementById("combo-count");
const gameWrapper = document.getElementById("game-wrapper");

let gameState = 'boot'; 
let keys = {};
let adminMode = false;
let testerMode = false;
let awaitingPassword = false;
let drawHitboxes = false;
let showUI = true;
let glitchFrozen = false; 

// ================== PROGRESSION FLAGS ==================
const BOSS_THRESHOLD = 3;
const GOD_THRESHOLD = 10;
let systemOverwrite = false; 
let lagPulseActive = false; 

// ================== SPRITE ANIMATION SYSTEM ==================
const spriteSystem = {
    // Sprite configurations - map animation states to sprite folders
    sprites: {},
    loaded: false,
    
    // Define which sprite images exist for each state
    spriteMap: {
        'idle': ['Idle.png/Screenshot 2026-02-03 145521.png', 'Idle.png/Screenshot 2026-02-03 145529.png', 'Idle.png/Screenshot 2026-02-03 145539.png', 'Idle.png/Screenshot 2026-02-03 145545.png', 'Idle.png/Screenshot 2026-02-03 145552.png', 'Idle.png/Screenshot 2026-02-03 145601.png'],
        'run': ['Run.png/'],  // Will use Run folder if available
        'attack': ['Attack.png/Screenshot 2026-02-03 145614.png', 'Attack.png/Screenshot 2026-02-03 145621.png', 'Attack.png/Screenshot 2026-02-03 145631.png', 'Attack.png/Screenshot 2026-02-03 145640.png', 'Attack.png/Screenshot 2026-02-03 145650.png'],
        'dash': ['Dash.png/Screenshot 2026-02-03 145735.png'],
        'hurt': ['Hurt.png/Screenshot 2026-02-03 150233.png', 'Hurt.png/Screenshot 2026-02-03 150316.png'],
        'jump': ['Jump.png/'],  // Will use Jump folder if available
        'overclock': ['Overclock.png/Screenshot 2026-02-03 150322.png', 'Overclock.png/Screenshot 2026-02-03 150329.png']
    },
    
    // Initialize - preload sprite images
    init: function() {
        console.log("Initializing sprite system...");
        
        // Create Image objects for each sprite
        for (let state in this.spriteMap) {
            this.sprites[state] = [];
            let frames = this.spriteMap[state];
            
            // If folder path without specific files, try to load folder contents
            if (frames.length === 1 && frames[0].endsWith('/')) {
                // This is a folder reference - we'll handle it dynamically
                console.log(`Sprite folder for ${state}: ${frames[0]}`);
            } else {
                // Load specific images
                frames.forEach((src, index) => {
                    let img = new Image();
                    img.src = `Sprites/${src}`;
                    img.onload = () => { 
                        console.log(`Loaded sprite: ${state} frame ${index}`); 
                        this.loaded = true;
                    };
                    img.onerror = () => { 
                        console.log(`Failed to load sprite: ${state} frame ${index}`); 
                    };
                    this.sprites[state].push(img);
                });
            }
        }
        
        // Set a fallback timeout - assume loaded if no errors after 2 seconds
        setTimeout(() => { this.loaded = true; console.log("Sprite system initialization complete"); }, 2000);
    },
    
    // Get sprite frame for a given state
    getFrame: function(state, frameIndex) {
        if (!this.sprites[state] || this.sprites[state].length === 0) {
            return null;
        }
        let frames = this.sprites[state];
        return frames[frameIndex % frames.length];
    },
    
    // Check if sprite system is ready
    isReady: function() {
        return this.loaded && Object.keys(this.sprites).length > 0;
    }
};

// ================== SMART LOGGING ==================
let logQueue = [];
let isLogProcessing = false;
let recentLogs = new Set();

function log(msg, type = "") {
    if (recentLogs.has(msg)) return;
    recentLogs.add(msg);
    setTimeout(() => recentLogs.delete(msg), 1500); 

    logQueue.push({msg, type});
    if (!isLogProcessing) processLogs();
}

function processLogs() {
    if (logQueue.length === 0) { isLogProcessing = false; return; }
    isLogProcessing = true;
    const item = logQueue.shift();
    const div = document.createElement("div"); 
    div.className = "log-msg " + item.type; 
    consoleLogs.appendChild(div); 
    
    let fullText = "> " + item.msg;
    let i = 0;
    let speed = 15; 

    function typeChar() {
        if (i < fullText.length) {
            div.textContent += fullText.charAt(i);
            consoleArea.scrollTop = consoleArea.scrollHeight;
            if (i % 2 === 0) sfx.type(item.type);
            i++;
            setTimeout(typeChar, speed);
        } else { setTimeout(processLogs, 500); }
    }
    typeChar();
}

// ================== SAVE SYSTEM ==================
const saveSystem = {
    data: { color: "#00ffff", wins: 0, losses: 0, winStreak: 0, noHitWins: 0, terminalWins: 0, totalRounds: 0, totalDmgDealt: 0, maxDmgDealt: 0, maxDmgTaken: 0, totalDmgTaken: 0 },
    load: function() {
        try {
            const stored = localStorage.getItem("fatalErrorData_v9");
            if (stored) {
                this.data = JSON.parse(stored);
                player.customColor = this.data.color;
                player.color = this.data.color;
                streakHud.innerText = this.data.winStreak;
                if(!this.data.maxDmgDealt) this.data.maxDmgDealt = 0;
                if(!this.data.maxDmgTaken) this.data.maxDmgTaken = 0;
            }
        } catch(e) {}
    },
    save: function() {
        this.data.color = player.customColor;
        localStorage.setItem("fatalErrorData_v9", JSON.stringify(this.data));
        streakHud.innerText = this.data.winStreak;
    },
    recordGame: function(isWin) {
        if (adminMode || testerMode) return; 
        this.data.totalRounds++;
        this.data.totalDmgDealt += roundStats.dmgDealt;
        this.data.totalDmgTaken += roundStats.dmgTaken;
        if (roundStats.dmgDealt > this.data.maxDmgDealt) this.data.maxDmgDealt = roundStats.dmgDealt;
        if (roundStats.dmgTaken > this.data.maxDmgTaken) this.data.maxDmgTaken = roundStats.dmgTaken;
        if (isWin) {
            this.data.wins++; this.data.winStreak++;
            if (roundStats.dmgTaken === 0) this.data.noHitWins++;
            if (roundStats.usedTerminal) this.data.terminalWins++;
        } else { this.data.losses++; this.data.winStreak = 0; }
        this.save();
    },
    wipe: function() { localStorage.removeItem("fatalErrorData_v9"); location.reload(); }
};

let roundStats = { dmgDealt: 0, dmgTaken: 0, usedTerminal: false };

// ================== INPUT & JUMP LOGIC ==================
consoleInput.addEventListener("input", () => sfx.keystroke());

window.addEventListener("keydown", e => {
    if(document.activeElement !== consoleInput) keys[e.key.toLowerCase()] = true;
    
    // Pause menu toggle
    if (e.key === 'Escape' && (gameState === 'playing' || gameState === 'paused')) {
        if (gameState === 'playing') {
            gameState = 'paused';
            document.getElementById('pause-menu').classList.remove('hidden');
            sfx.menuBlip();
        } else {
            resumeGame();
        }
    }
    
    if ((e.key.toLowerCase() === 'w' || e.key === ' ') && gameState === 'playing' && document.activeElement !== consoleInput) {
        if (player.onGround) {
            player.vy = player.jumpForce; player.onGround = false; player.jumpCount = 1; sfx.jump();
            spawnParticles(player.x, player.y+player.h, "#fff", 5);
        } else if (player.jumpCount < 2) {
            player.vy = player.jumpForce * 0.9; player.jumpCount = 2; sfx.doubleJump();
            spawnParticles(player.x, player.y+player.h, "#00ffff", 8);
        }
    }

    if (gameState === 'win' || gameState === 'lose') {
        if (e.key.toLowerCase() === 'r') switchScreen('main-menu'); 
        if (e.key === ' ' && gameState === 'win') nextRound(); 
    }
    if (e.key === "Enter" && document.activeElement === consoleInput) {
        processCommand(consoleInput.value); consoleInput.value = "";
    }
});
window.addEventListener("keyup", e => keys[e.key.toLowerCase()] = false);
canvas.addEventListener("mousedown", e => {
    if (gameState === 'playing') {
        if (e.button === 0) handleInput(player, "light");
        if (e.button === 2) handleInput(player, "heavy");
    }
});
canvas.addEventListener("contextmenu", e => e.preventDefault());

// ================== VARIABLES ==================
let gravity = 0.6;
const groundY = 410;
let glitch = 0;

// ========== PLATFORMS FOR BOSS ARENA ==========
const platforms = [
    { x: 0, y: 410, w: 800, h: 40 },    // Main ground
    { x: 100, y: 320, w: 150, h: 15 },  // Left platform
    { x: 550, y: 320, w: 150, h: 15 },  // Right platform  
    { x: 325, y: 250, w: 150, h: 15 },  // Top center platform
    { x: 150, y: 180, w: 100, h: 15 },  // Upper left
    { x: 550, y: 180, w: 100, h: 15 },  // Upper right
];
let frameCount = 0;
let difficultyLevel = 1;
let projectiles = [];
let particles = [];
let effects = [];

let fpsInterval = 1000 / 60;
let now, then = Date.now(), elapsed;

const createChar = (x, color, name, isBoss = false) => ({
    x: x, y: 300, 
    w: isBoss ? 70 : 50,  // Boss is bigger
    h: isBoss ? 100 : 75,  // Boss is taller
    vx: 0, vy: 0,
    hp: isBoss ? 25000 : 15000, 
    maxHp: isBoss ? 25000 : 15000, 
    onGround: false, facing: 1, name: name,
    jumpCount: 0, 
    attackCooldown: 0, windup: 0, hitstun: 0, invuln: 0,
    comboStep: 0, comboTimer: 0,
    comboCharge: 0,  // Charge meter for weapon switch
    weapon: 'katana', switchCooldown: 0,
    overclockTimer: 0, instinctActive: false,
    lagTimer: 0, silenceTimer: 0, dashTimer: 0, dashCooldown: 0, exCooldown: 0, 
    dmgBuff: 1, godMode: false, noclip: false, aiPaused: false,
    baseSpeed: isBoss ? 3 : 5, jumpForce: isBoss ? -10 : -12, ghosts: [], attackBox: null,
    color: color, customColor: color, isBlocking: false,
    isBoss: isBoss, isGod: false,
    hits: 0, lastHitTime: 0,
    // Animation System
    animState: 'idle', // idle, run, jump, fall, attack, heavyAttack, shoot, block, knockback, dash
    animFrame: 0,
    animTimer: 0,
    effectColor: null,
    effectTimer: 0,
    // Boss phases
    bossPhase: 1,
    bossMaxPhase: 3,
    introTimer: 120,
    introComplete: false
});

const player = createChar(100, "#00ffff", "Catalyst", false);
const enemy = createChar(600, "#ff0055", "Glitch Proxy", true);

// ================== SKIP BOOT SEQUENCE ==================
function skipBootSequence() {
    const overlay = document.getElementById("cutscene-overlay");
    const skipBtn = document.getElementById("skip-boot-btn");
    
    // Hide skip button
    if (skipBtn) skipBtn.style.display = 'none';
    
    // Clear the boot log and hide overlay
    overlay.innerHTML = "";
    overlay.classList.add('hidden');
    
    // Go to main menu instead of starting game directly
    gameState = 'menu';
    document.getElementById('main-menu').classList.remove('hidden');
    log("SYSTEM ONLINE. FIGHT STARTED.", "sys");
}

// ================== CINEMATIC BOOT ==================
const bootLines = ["LOADING KERNEL...", "MOUNTING AUDIO DRIVERS...", "GENERATING WAVEFORMS...", "INITIALIZING COMBAT UI...", "WELCOME USER."];
let bootIndex = 0;

function runBootSequence() {
    const logDiv = document.getElementById("boot-log");
    const overlay = document.getElementById("cutscene-overlay");
    sfx.init(); // Init Audio Context
    
    if (bootIndex === 0) sfx.bootHum(); // Start Engine Sound

    if (bootIndex < bootLines.length) {
        logDiv.innerHTML += bootLines[bootIndex] + "<br>";
        sfx.type('sys'); 
        bootIndex++; 
        setTimeout(runBootSequence, 400);
    } 
    else if (bootIndex === bootLines.length) {
        setTimeout(() => {
            bootIndex++;
            logDiv.innerHTML = "";
            let studio = document.createElement("div"); studio.className = "studio-logo"; studio.innerText = "CATALYST STUDIOS";
            let presents = document.createElement("div"); presents.className = "presents-text"; presents.innerText = "PRESENTS";
            overlay.appendChild(studio); overlay.appendChild(presents);
            sfx.explosion(); 
            runBootSequence();
        }, 1000);
    }
    else if (bootIndex === bootLines.length + 1) {
        setTimeout(() => {
            bootIndex++;
            overlay.innerHTML = ""; 
            let win = document.createElement("div"); win.className = "error-window";
            let header = document.createElement("div"); header.className = "error-header"; header.innerText = "X SYSTEM CRASH";
            let content = document.createElement("div"); content.className = "error-content";
            let title = document.createElement("h2"); title.className = "error-title"; title.innerText = "FATAL ERROR";
            let sub = document.createElement("div"); sub.innerText = "v9.4 Audio Engineer";
            content.appendChild(title); content.appendChild(sub);
            win.appendChild(header); win.appendChild(content);
            overlay.appendChild(win);
            shakeScreen(); sfx.hit();
            runBootSequence();
        }, 2000);
    }
    else {
        setTimeout(() => {
            overlay.classList.add("hidden");
            document.getElementById("main-menu").classList.remove("hidden");
            gameState = 'menu';
            sfx.playMusic('lobby'); // START LOBBY MUSIC
        }, 1500);
    }
}

// ================== GAME LOGIC ==================
function switchScreen(screenId) {
    sfx.menuBlip();
    document.querySelectorAll('.overlay-screen').forEach(el => el.classList.add('hidden'));
    if(screenId) {
        document.getElementById(screenId).classList.remove('hidden');
        gameState = (screenId === 'main-menu') ? 'menu' : 'paused';
        if (screenId === 'main-menu') sfx.playMusic('lobby'); 
    }
}

function initGameAndAudio() {
    sfx.init(); 
    startGame();
}

function startGame() {
    switchScreen(null); 
    roundStats = { dmgDealt: 0, dmgTaken: 0, usedTerminal: false };
    player.hp = 15000; player.maxHp = 15000;
    player.x = 100; player.y = 300; player.vx = 0; player.vy = 0;
    player.w = 35; player.h = 55; player.color = player.customColor;
    player.dashTimer = 0; player.dashCooldown = 0; player.exCooldown = 0;
    player.lagTimer = 0; player.silenceTimer = 0;
    player.weapon = 'katana'; player.overclockTimer = 0; player.instinctActive = false;
    player.dmgBuff = 1; player.ghosts = []; player.hits = 0;
    enemy.hp = 25000; enemy.maxHp = 25000; 
    enemy.x = 600; enemy.y = 300; enemy.vx = 0; enemy.vy = 0;
    enemy.w = 40; enemy.h = 60; enemy.ghosts = [];
    glitch = 0; glitchFrozen = false; projectiles = []; particles = []; effects = [];
    systemOverwrite = false; lagPulseActive = false;
    keys = {};
    setDifficulty();
    gameState = 'playing';
    
    // Initialize cinematic intro for God boss only (spiral attacks during intro)
    if (enemy.isGod) {
        enemy.introTimer = 180; // 3 seconds cinematic intro with spiral
        enemy.introComplete = false;
        enemy.x = 400 - enemy.w/2; // Position boss at middle
        enemy.y = 200; // Mid-air position
        enemy.aiPaused = true; // Pause AI during intro
        enemy.vx = 0;
        enemy.vy = 0;
        log(">>> OMEGA ENTITY MANIFESTING <<<", "admin");
    } else if (enemy.isBoss) {
        // Mini-boss gets brief pause then fight starts
        enemy.introTimer = 60;
        enemy.introComplete = false;
        enemy.x = 600;
        enemy.y = 300;
        enemy.aiPaused = true;
        log(">>> ANOMALY DETECTED <<<", "err");
    }
    
    log("SYSTEM ONLINE. FIGHT STARTED.", "sys");
}

function resumeGame() {
    gameState = 'playing';
    document.getElementById('pause-menu').classList.add('hidden');
    sfx.menuBlip();
    log("RESUMING...", "sys");
}

// ================== PAUSE BUTTON ==================
function togglePause() {
    const pauseBtn = document.getElementById('pause-btn');
    
    if (gameState === 'playing') {
        gameState = 'paused';
        document.getElementById('pause-menu').classList.remove('hidden');
        if (pauseBtn) pauseBtn.innerHTML = '▶ PLAY';
    } else if (gameState === 'paused') {
        gameState = 'playing';
        document.getElementById('pause-menu').classList.add('hidden');
        if (pauseBtn) pauseBtn.innerHTML = '⏸ PAUSE';
        log("RESUMING...", "sys");
    }
}

function restartGame() {
    document.getElementById('pause-menu').classList.add('hidden');
    startGame();
}

function nextRound() {
    player.hp = Math.min(player.hp + 5000, player.maxHp);
    enemy.x = 600; enemy.y = 300;
    player.x = 100; player.y = 300;
    glitch = 0; systemOverwrite = false; gameState = 'playing'; keys = {}; 
    setDifficulty(); 
    log("NEXT ROUND. ENEMY BUFFED.", "sys");
}

function setDifficulty() {
    let streak = saveSystem.data.winStreak;
    
    // Reset enemy properties
    enemy.isBoss = false;
    enemy.isGod = false;
    enemy.bossPhase = 1;
    enemy.bossMaxPhase = 3;
    enemy.lastSpiralPhase = -1; // Track spiral phase for God boss
    enemy.effectColor = null; // For mini-boss attack telegraphing
    
    if (streak >= GOD_THRESHOLD) {
        // ===== FINAL BOSS - OMEGA (Souls-like) =====
        enemy.name = "FATAL EXCEPTION // OMEGA";
        enemy.maxHp = 100000 + ((streak - 10) * 10000); 
        enemy.w = 70; enemy.h = 100;
        enemy.isBoss = true; enemy.isGod = true;
        enemy.color = "#ffffff"; // White - godlike
        enemy.baseSpeed = 3; // Slow but powerful
        enemy.jumpForce = -10;
        difficultyLevel = 5;
        log(">>> CRITICAL THREAT: OMEGA MODE <<<", "admin");
        sfx.playMusic('battle_god');
    }
    else if (streak >= BOSS_THRESHOLD) {
        // ===== SECOND BOSS - SYSTEM ERROR (Mini-boss - Big Brute) =====
        enemy.name = "SYSTEM ERROR // GLITCH";
        enemy.maxHp = 45000 + ((streak - 3) * 5000); 
        enemy.w = 70; enemy.h = 100; // Big brute size
        enemy.isBoss = true; enemy.isGod = false;
        enemy.color = "#cc4400"; // Dark orange - heavy, dangerous
        enemy.baseSpeed = 3; // Heavy but powerful
        enemy.jumpForce = -11;
        difficultyLevel = 3;
        log(">>> WARNING: ANOMALY DETECTED <<<", "err");
        sfx.playMusic('battle_boss');
    }
    else {
        // ===== FIRST ENEMY - GLITCH PROXY (Practice) =====
        enemy.name = "Glitch Proxy v" + (1 + streak * 0.1).toFixed(1);
        enemy.maxHp = 15000 + (streak * 2000); 
        enemy.w = 30; enemy.h = 50; // Smaller than player
        enemy.isBoss = false; enemy.isGod = false;
        enemy.color = "#ff6699"; // Lighter pink - cute, non-threatening
        enemy.baseSpeed = 5; // Slower, practice dummy
        enemy.jumpForce = -12;
        difficultyLevel = 1;
        sfx.playMusic('battle_1');
    }
    enemy.hp = enemy.maxHp;
}

function showStats() {
    saveSystem.load(); switchScreen('stats-menu');
    let d = saveSystem.data;
    let avgDealt = d.totalRounds > 0 ? (d.totalDmgDealt / d.totalRounds).toFixed(0) : 0;
    let ratio = d.losses > 0 ? (d.wins / d.losses).toFixed(2) : d.wins;
    document.getElementById("stat-wins").innerText = d.wins;
    document.getElementById("stat-ratio").innerText = ratio;
    document.getElementById("stat-streak").innerText = d.winStreak;
    document.getElementById("stat-losses").innerText = d.losses;
    document.getElementById("stat-nohit").innerText = d.noHitWins;
    document.getElementById("stat-term").innerText = d.terminalWins;
    document.getElementById("stat-avg-dealt").innerText = avgDealt;
    document.getElementById("stat-high-dealt").innerText = d.maxDmgDealt || 0;
    document.getElementById("stat-high-taken").innerText = d.maxDmgTaken || 0;
}

function clearSave() { if(confirm("Delete all data?")) saveSystem.wipe(); }
function shakeScreen() { gameWrapper.classList.add("shake"); setTimeout(() => gameWrapper.classList.remove("shake"), 300); }

// ================== COMMANDS ==================
function processCommand(cmd) {
    cmd = cmd.toLowerCase().trim();
    
    if (cmd === "/reset") { startGame(); log("ROUND RESET.", "sys"); return; }
    if (cmd === "/restart") { location.reload(); return; }
    if (cmd.startsWith("/streak ")) { let n = parseInt(cmd.split(" ")[1]); saveSystem.data.winStreak = n; streakHud.innerText = n; saveSystem.save(); log(`STREAK SET: ${n}`, "sys"); return; }
    if (cmd === "/test over") { if (testerMode) { testerMode = false; log("TESTER SESSION ENDED.", "sys"); } else { log("NOT IN TESTER MODE.", "err"); } return; }
    if (cmd === "/admin") { if (adminMode) { adminMode = false; testerMode = false; document.getElementById("admin-indicator").style.display = "none"; log("LOGGED OUT.", "sys"); } else { awaitingPassword = true; log("ENTER ROOT KEY:", "sys"); } return; }
    if (awaitingPassword) { if (cmd === "159372486") { adminMode = true; testerMode = false; awaitingPassword = false; document.getElementById("admin-indicator").style.display = "block"; log("ACCESS GRANTED.", "admin"); } else { log("DENIED.", "err"); awaitingPassword = false; } return; }
    if (cmd.startsWith("sudo system_tester;")) { let pass = cmd.split(";")[1]; if (pass && pass.trim() === "000000") { testerMode = true; log("TESTER MODE ACTIVE.", "sys"); } else if (pass && pass.trim() === "159372486") { adminMode = true; testerMode = false; document.getElementById("admin-indicator").style.display = "block"; log("ADMIN OVERRIDE GRANTED.", "admin"); } else { log("INVALID TESTER KEY.", "err"); } return; }
    if (cmd === "sudo system_override; admin") { if (adminMode) { log("ALREADY ROOT.", "sys"); } else { log("OVERRIDE INITIATED...", "sys"); log("ENTER ROOT KEY:", "admin"); awaitingPassword = true; } return; }

    log(cmd.toUpperCase(), "player-cmd"); 

    if (cmd === "/clear") { document.getElementById("logs").innerHTML = ""; logQueue=[]; log("CONSOLE CLEARED.", "sys"); return; }
    if (cmd === "/glitch fix") { glitchFrozen = !glitchFrozen; log(`GLITCH FROZEN: ${glitchFrozen}`, "sys"); return; }

    if (adminMode || testerMode) {
        if (cmd.startsWith("/cmd")) {
            const sub = cmd.split(" ")[1]; 
            const playerList = "PLAYER: boost, shield, purge, overclock, color [hex]";
            const testerList = "TESTER: /fps, /box, /log, /lag, /clean, /ui, /shake_test, /dummy, /self_hit, /part_max, /color_test, /difficulty, /reset_cd, /test over, /hp, /atk, /spd, /grav, /ehp, /espd, /eatk, /estun, /escale, /eai, /etp, /edash, /eheal, /ekill, /clear, /glitch fix, /vamp, /moon, /heavy, /sprint, /tank";
            const adminList  = "ADMIN: /god, /heal, /kill, /onehit, /noclip, /smite, /fill, /drain, /flash, /size, /ghost, /tp, /reset, /restart, /streak [n]";

            if (!sub) { log("--- ALL SYSTEM COMMANDS ---", "sys"); log(playerList, "player-cmd"); log(testerList, "sys"); if (adminMode) log(adminList, "admin"); }
            else if (sub === "player") log(playerList, "player-cmd");
            else if (sub === "tester") log(testerList, "sys");
            else if (sub === "admin") { if (adminMode) log(adminList, "admin"); else log("ACCESS DENIED.", "err"); }
            return;
        }
        if (cmd === "/vamp") { player.lifesteal = !player.lifesteal; log(`LIFESTEAL: ${player.lifesteal}`, "sys"); return; }
        if (cmd === "/moon") { gravity = 0.3; log("MOON GRAVITY.", "sys"); return; }
        if (cmd === "/heavy") { gravity = 1.2; log("HEAVY GRAVITY.", "sys"); return; }
        if (cmd === "/sprint") { player.baseSpeed = 10; log("SPRINT MODE.", "sys"); return; }
        if (cmd === "/tank") { player.maxHp = 30000; player.hp = 30000; log("TANK MODE.", "sys"); return; }
        if (cmd === "/box") { drawHitboxes = !drawHitboxes; log(`HITBOXES: ${drawHitboxes}`, "sys"); return; }
        if (cmd === "/log") { console.log(player, enemy); log("DUMPED TO CONSOLE.", "sys"); return; }
        if (cmd === "/lag") { let start = Date.now(); while(Date.now() < start + 50); log("LAG SIMULATED.", "sys"); return; }
        if (cmd === "/clean") { particles = []; projectiles = []; log("CLEANUP DONE.", "sys"); return; }
        if (cmd === "/ui") { showUI = !showUI; log(`UI HIDDEN: ${!showUI}`, "sys"); return; }
        if (cmd === "/shake_test") { shakeScreen(); log("SHAKE TRIGGERED.", "sys"); return; }
        if (cmd === "/dummy") { enemy.aiPaused = !enemy.aiPaused; log(`AI PAUSED: ${enemy.aiPaused}`, "sys"); return; }
        if (cmd === "/self_hit") { player.hp -= 1000; log("OUCH.", "sys"); return; }
        if (cmd === "/part_max") { spawnParticles(400, 200, "#fff", 100); log("PARTICLE STRESS.", "sys"); return; }
        if (cmd === "/color_test") { player.color = `hsl(${Math.random()*360},100%,50%)`; log("COLOR RANDO.", "sys"); return; }
        if (cmd.startsWith("/fps ")) { fpsInterval = 1000 / parseInt(cmd.split(" ")[1]); log("FPS CAP UPDATED.", "sys"); return; }
        if (cmd.startsWith("/difficulty ")) { difficultyLevel = parseInt(cmd.split(" ")[1]); log("DIFF SET.", "sys"); return; }
        if (cmd === "/reset_cd") { player.attackCooldown=0; player.dashCooldown=0; player.exCooldown=0; log("CD RESET.", "sys"); return; }
        if (cmd.startsWith("/hp ")) { player.hp = parseInt(cmd.split(" ")[1]); log("HP SET.", "sys"); return; }
        if (cmd.startsWith("/atk ")) { player.dmgBuff = parseFloat(cmd.split(" ")[1]); log("DMG MULT SET.", "sys"); return; }
        if (cmd.startsWith("/spd ")) { player.baseSpeed = parseInt(cmd.split(" ")[1]); log("SPEED SET.", "sys"); return; }
        if (cmd.startsWith("/grav ")) { gravity = parseFloat(cmd.split(" ")[1]); log("GRAVITY SET.", "sys"); return; }
        if (cmd.startsWith("/ehp ")) { enemy.hp = parseInt(cmd.split(" ")[1]); enemy.maxHp = enemy.hp; log("ENEMY HP SET.", "sys"); return; }
        if (cmd.startsWith("/espd ")) { enemy.baseSpeed = parseInt(cmd.split(" ")[1]); log("ENEMY SPD SET.", "sys"); return; }
        if (cmd.startsWith("/eatk ")) { enemy.dmgBuff = parseFloat(cmd.split(" ")[1]); log("ENEMY ATK MULT SET.", "sys"); return; }
        if (cmd.startsWith("/estun ")) { enemy.hitstun = parseInt(cmd.split(" ")[1]); log("ENEMY STUNNED.", "sys"); return; }
        if (cmd.startsWith("/escale ")) { let s = parseFloat(cmd.split(" ")[1]); enemy.w = 40 * s; enemy.h = 60 * s; log("ENEMY SIZE SET.", "sys"); return; }
        if (cmd.startsWith("/eai ")) { enemy.aiPaused = (cmd.split(" ")[1] === "0"); log(`ENEMY AI: ${!enemy.aiPaused}`, "sys"); return; }
        if (cmd === "/etp") { enemy.x = 400; enemy.y = 300; log("ENEMY RESET.", "sys"); return; }
        if (cmd === "/edash") { enemy.vx = enemy.facing * 20; enemy.dashTimer = 10; log("ENEMY DASH FORCED.", "sys"); return; }
        if (cmd === "/eheal") { enemy.hp = enemy.maxHp; log("ENEMY HEALED.", "sys"); return; }
        if (cmd === "/ekill") { enemy.hp = 1; log("ENEMY CRITICAL.", "sys"); return; }
        if (adminMode) {
            if (cmd.startsWith("/size ")) { let s = parseFloat(cmd.split(" ")[1]); player.w = 40 * s; player.h = 60 * s; log("SIZE MODIFIED.", "admin"); return; }
            if (cmd === "/ghost") { player.customColor = (player.customColor === player.color) ? "rgba(0,255,255,0.3)" : "#00ffff"; player.color = player.customColor; log("STEALTH TOGGLED.", "admin"); return; }
            if (cmd === "/tp enemy") { player.x = enemy.x - (enemy.facing * 50); log("TELEPORTED.", "admin"); return; }
            if (cmd === "/tp center") { player.x = 400; player.y = 300; log("TELEPORTED.", "admin"); return; }
            if (cmd === "/smite") { enemy.hp -= 10000; spawnParticles(enemy.x, enemy.y, "#fff", 50); shakeScreen(); log("SMITE CAST.", "admin"); return; }
            if (cmd === "/fill") { glitch = 100; log("METER FILLED.", "admin"); return; }
            if (cmd === "/drain") { enemy.hp = Math.min(enemy.hp, enemy.maxHp * 0.1); glitch = 0; log("ENEMY DRAINED.", "admin"); return; }
            if (cmd === "/flash") { ctx.fillStyle = "white"; ctx.fillRect(0,0,800,450); log("FLASHBANG.", "admin"); return; }
            if (cmd === "/onehit") { player.dmgBuff = 1000; log("ONE PUNCH MODE.", "admin"); return; }
            if (cmd === "/god") { player.godMode = !player.godMode; log(`GOD MODE: ${player.godMode}`, "admin"); return; }
            if (cmd === "/heal") { player.hp = player.maxHp; log("HP RESTORED.", "admin"); return; }
            if (cmd === "/kill") { enemy.hp = 1; log("ENEMY HP CRITICAL.", "admin"); return; }
            if (cmd === "/noclip") { player.noclip = !player.noclip; log(`NOCLIP: ${player.noclip}`, "admin"); return; }
        }
    }

    if (gameState === 'menu') { if (cmd.startsWith("color ")) { let c = cmd.split(" ")[1]; player.customColor = c; player.color = c; saveSystem.save(); log("COLOR SAVED.", "player-cmd"); } return; }
    if (gameState === 'playing') {
        if (player.silenceTimer > 0) { log("ERROR: ABILITIES SILENCED", "err"); return; }
        roundStats.usedTerminal = true;
        if (cmd === "boost") { player.dmgBuff = 1.15; log("DAMAGE BOOST (+15%)", "buff"); } 
        else if (cmd === "shield") { player.hp = Math.min(player.hp + 2000, player.maxHp); log("SHIELD RESTORED", "buff"); } 
        else if (cmd === "purge") { player.lagTimer = 0; player.silenceTimer = 0; glitch = 0; log("SYSTEM PURGED.", "buff"); } 
        else if (cmd === "overclock") { triggerOverclock(); }
    }
}

function triggerOverclock() {
    if (glitch >= 100) {
        player.overclockTimer = 300; player.dmgBuff = 1.25; player.hp = player.maxHp;
        glitch = 0; log("/// CATALYST OVERCLOCK ENGAGED ///", "buff");
        sfx.osc(600, 'square', 0.5, 0.4, 1200); 
    } else log("NEED 100 INSTABILITY", "err");
}

// ========== CINEMATIC BOSS INTRO & SPIRAL ATTACK ==========
let spiralAngle = 0;

function spawnSpiralProjectile() {
    // Spawn a projectile in a spiral pattern around the boss
    let angle = spiralAngle;
    let radius = 50;
    let speed = 8;
    
    projectiles.push({
        x: enemy.x + enemy.w/2 + Math.cos(angle) * radius,
        y: enemy.y + enemy.h/2 + Math.sin(angle) * radius,
        vx: Math.cos(angle) * speed,
        vy: Math.sin(angle) * speed,
        owner: enemy,
        damage: 150,
        life: 60,
        type: 'spiral',
        w: 12,
        h: 12,
        color: "#ff00ff"
    });
    
    spiralAngle += 0.5; // Increment spiral angle for next projectile
}

function handleCinematicIntro() {
    // Handle mini-boss intro (brief pause, no spiral)
    if (enemy.isBoss && !enemy.isGod && !enemy.introComplete && enemy.introTimer > 0) {
        enemy.introTimer--;
        if (enemy.introTimer === 0) {
            enemy.introComplete = true;
            enemy.aiPaused = false;
            log(">>> ANOMALY ENGAGED <<<", "err");
        }
        return;
    }
    
    // Only God boss gets cinematic intro with spiral attacks
    if (!enemy.isGod || enemy.introComplete) return;
    
    if (enemy.introTimer > 0) {
        enemy.introTimer--;
        
        // Boss dramatic float in center
        enemy.y = 180 + Math.sin(enemy.introTimer * 0.1) * 10;
        
        // INTENSIFIED Spiral attack pattern during intro - for God boss only
        if (enemy.introTimer % 6 === 0 && enemy.introTimer < 150) {
            // Fire multiple projectiles in spiral pattern
            for (let i = 0; i < 6; i++) {
                let angle = (Math.PI * 2 / 6) * i + spiralAngle;
                let speed = 7;
                projectiles.push({
                    x: enemy.x + enemy.w/2 + Math.cos(angle) * 40,
                    y: enemy.y + enemy.h/2 + Math.sin(angle) * 40,
                    vx: Math.cos(angle) * speed,
                    vy: Math.sin(angle) * speed,
                    owner: enemy,
                    damage: 250,
                    life: 60,
                    type: 'spiral',
                    w: 14,
                    h: 14,
                    color: "#ff00ff"
                });
            }
            spiralAngle += 0.4;
            sfx.shot();
            spawnParticles(enemy.x + enemy.w/2, enemy.y + enemy.h/2, "#ff00ff", 10);
        }
        
        // Warning at certain points
        if (enemy.introTimer === 150) {
            log(">>> WARNING: OMEGA SPIRAL INCOMING <<<", "admin");
            shakeScreen();
        }
        
        // Screen flash effect at the end of intro
        if (enemy.introTimer === 30) {
            log(">>> DIVINE FORM MANIFEST <<<", "admin");
            enemy.effectColor = "#ffffff";
            enemy.effectTimer = 30;
            shakeScreen();
            sfx.explosion();
        }
        
        // Intro complete
        if (enemy.introTimer === 0) {
            enemy.introComplete = true;
            enemy.aiPaused = false;
            enemy.x = 600; // Move boss to normal position
            enemy.y = 300;
            log(">>> OMEGA BATTLE INITIATED <<<", "admin");
            spiralAngle = 0; // Reset spiral angle
        }
    }
}

function spawnParticles(x, y, color, count) {
    for(let i=0; i<count; i++) {
        particles.push({
            x: x, y: y,
            vx: (Math.random()-0.5) * 10,
            vy: (Math.random()-0.5) * 10,
            life: 20 + Math.random() * 20,
            color: color,
            size: 2 + Math.random() * 3
        });
    }
}

// ================== ANIMATION SYSTEM ==================

// Linear interpolation for smooth animation transitions
function lerp(start, end, t) {
    return start + (end - start) * t;
}

// Store previous animation values for interpolation
let prevAnimState = { player: null, enemy: null };
let animTransition = { player: 0, enemy: 0 };

function updateAnimations(char) {
    let isPlayer = (char === player);
    
    // Determine animation state based on current actions - priority ordered
    let newState = 'idle';
    
    // Priority 1: Knocback (highest priority - can't do anything else)
    if (char.hitstun > 0) {
        newState = 'knockback';
    }
    // Priority 2: Dash (invulnerable state)
    else if (char.dashTimer > 0) {
        newState = 'dash';
    }
    // Priority 3: Weapon switch animation
    else if (char.switchCooldown > 45) {
        newState = 'switch';
    }
    // Priority 4: Heavy attack windup
    else if (char.windup > 0) {
        newState = 'heavyAttack';
    }
    // Priority 5: Active attack (melee or ranged)
    else if (char.attackBox && char.attackBox.time > 0) {
        if (char === player && char.weapon === 'pistol') {
            newState = 'shoot';
        } else {
            newState = 'attack';
        }
    }
    // Priority 6: Blocking
    else if (char.isBlocking) {
        newState = 'block';
    }
    // Priority 7: Airborne states
    else if (!char.onGround) {
        // Going up = jump, falling = fall
        if (char.vy < -1) {
            newState = 'jump';
        } else if (char.vy > 1) {
            newState = 'fall';
        } else {
            // Small vertical velocity - still in air
            newState = char.vy < 0 ? 'jump' : 'fall';
        }
    }
    // Priority 8: Running (when moving on ground)
    else if (Math.abs(char.vx) > 0.5) {
        newState = 'run';
    }
    // Default: idle
    
    // Only update state and reset frame if state actually changed
    if (newState !== char.animState) {
        // Start new transition
        let isPlayer = (char === player);
        prevAnimState[isPlayer ? 'player' : 'enemy'] = char.animState;
        animTransition[isPlayer ? 'player' : 'enemy'] = 0;
        
        char.animState = newState;
        char.animFrame = 0;
        char.animTimer = 0;
    } else {
        // Update transition progress for smooth blending
        let isPlayer = (char === player);
        animTransition[isPlayer ? 'player' : 'enemy'] = Math.min(1, animTransition[isPlayer ? 'player' : 'enemy'] + 0.15);
    }
    
    // Advance animation frame based on state (faster for smoother animation)
    char.animTimer++;
    let frameSpeed = 1;  // Lower values = smoother, faster animation
    switch(char.animState) {
        case 'idle': frameSpeed = 4; break;  // Reduced from 6 for smoother breathing
        case 'run': frameSpeed = 2; break;   // Reduced from 3 for smoother running
        case 'jump': frameSpeed = 3; break;
        case 'fall': frameSpeed = 3; break;
        case 'attack': frameSpeed = 1; break;  // Faster attack animation
        case 'heavyAttack': frameSpeed = 1; break;
        case 'shoot': frameSpeed = 1; break;
        case 'block': frameSpeed = 3; break;
        case 'knockback': frameSpeed = 1; break;
        case 'dash': frameSpeed = 1; break;
        case 'switch': frameSpeed = 2; break;
        default: frameSpeed = 3;
    }
    
    if (char.animTimer >= frameSpeed) {
        char.animTimer = 0;
        char.animFrame++;
    }
    
    // Update effect color for special effects
    if (char.effectTimer > 0) {
        char.effectTimer--;
        if (char.effectTimer === 0) char.effectColor = null;
    }
}

// ========== CHARACTER TIER SYSTEM - DISTINCT VISUALS FOR EACH TYPE ==========
function drawCharacterAnimation(ctx, char, isPlayer) {
    let facing = char.facing;
    let baseColor = char.color;
    
    // Determine character tier
    let isPlayerChar = (char === player);
    let isBossChar = char.isBoss;
    let isGodChar = char.isGod;
    let isFirstEnemy = !isBossChar && !isPlayerChar; // Practice enemy
    
    // Effect color override
    if (char.effectColor) baseColor = char.effectColor;
    
    ctx.fillStyle = baseColor;
    ctx.strokeStyle = baseColor;
    ctx.lineWidth = 2;
    
    // ===== PLAYER CHARACTER - Fast, Nimble, Clean =====
    if (isPlayerChar) {
        // Sleek thin body - emphasizes speed
        ctx.fillRect(char.x + 5, char.y, char.w - 10, char.h);
        
        // Small compact head
        ctx.fillRect(char.x + 10, char.y - 12, char.w - 20, 15);
        
        // Sharp white eyes - focused
        ctx.fillStyle = "#ffffff";
        ctx.fillRect(char.x + 12, char.y - 7, 4, 3);
        ctx.fillRect(char.x + char.w - 16, char.y - 7, 4, 3);
        
        // Thin weapon - katana or pistol
        if (char.weapon === 'katana') {
            let weaponX = char.x + (facing > 0 ? char.w - 3 : 3);
            ctx.fillStyle = "#00ffff";
            ctx.shadowBlur = 15;
            ctx.shadowColor = "#00ffff";
            ctx.fillRect(weaponX - 1, char.y + 8, 2, 30);
        } else {
            let gunX = char.x + (facing > 0 ? char.w - 2 : -10);
            ctx.fillStyle = "#888";
            ctx.fillRect(gunX, char.y + 12, 12, 6);
        }
        
        // Overclock mode - blue neon glow
        if (char.overclockTimer > 0) {
            ctx.shadowBlur = 25;
            ctx.shadowColor = "#00aaff"; // Blue neon instead of gold
            ctx.fillStyle = "#aaddff";
            ctx.fillRect(char.x + 5, char.y, char.w - 10, char.h);
        }
        
        // Instinct mode - red aura
        if (char.instinctActive) {
            ctx.shadowBlur = 20;
            ctx.shadowColor = "#ff0000";
        }
        
        // Dash trail effect
        if (char.dashTimer > 0) {
            ctx.globalAlpha = 0.3;
            ctx.fillStyle = "#00ffff";
            ctx.fillRect(char.x - facing * 15, char.y, char.w, char.h);
            ctx.fillRect(char.x - facing * 30, char.y, char.w, char.h);
            ctx.globalAlpha = 1.0;
        }
    }
    
    // ===== FIRST ENEMY (GLITCH PROXY) - Practice, Less Threatening =====
    else if (isFirstEnemy && !isBossChar) {
        // Small, neat body - practice target
        ctx.fillRect(char.x + 3, char.y, char.w - 6, char.h);
        
        // Small compact head
        ctx.fillRect(char.x + 6, char.y - 12, char.w - 12, 14);
        
        // Cute pink eyes - non-threatening
        ctx.fillStyle = "#ff88aa";
        ctx.fillRect(char.x + 6, char.y - 6, 4, 3);
        ctx.fillRect(char.x + char.w - 10, char.y - 6, 4, 3);
        
        // No weapon - practice dummy
    }
    
    // ===== SECOND BOSS (SYSTEM ERROR) - Mini-boss, Big Brute =====
    else if (isBossChar && !isGodChar) {
        // Wide, bulky body - brute look
        ctx.fillRect(char.x - 5, char.y, char.w + 10, char.h);
        
        // Big shoulders wider than head
        ctx.fillRect(char.x - 10, char.y + 10, 15, 30);
        ctx.fillRect(char.x + char.w - 5, char.y + 10, 15, 30);
        
        // Large angular head
        ctx.beginPath();
        ctx.moveTo(char.x + char.w/2, char.y - 25);
        ctx.lineTo(char.x + char.w + 5, char.y - 5);
        ctx.lineTo(char.x + char.w + 5, char.y + 10);
        ctx.lineTo(char.x + char.w/2, char.y + 18);
        ctx.lineTo(char.x - 3, char.y + 15);
        ctx.lineTo(char.x - 3, char.y);
        ctx.closePath();
        ctx.fill();
        
        // Glowing red eyes - menacing
        ctx.fillStyle = "#ff0000";
        ctx.shadowBlur = 10;
        ctx.shadowColor = "#ff0000";
        ctx.fillRect(char.x + 12, char.y - 6, 10, 6);
        ctx.fillRect(char.x + char.w - 22, char.y - 6, 10, 6);
        
        // Phase indicator - color shift in phase 2
        if (char.bossPhase >= 2) {
            ctx.strokeStyle = "#ff4400";
            ctx.lineWidth = 3;
            ctx.strokeRect(char.x - 5, char.y - 5, char.w + 10, char.h + 10);
        }
    }
    
    // ===== FINAL BOSS (OMEGA/GOD) - Souls-like, Most Dangerous =====
    else if (isGodChar) {
        // Largest, most imposing
        ctx.fillRect(char.x, char.y, char.w, char.h);
        
        // Octagonal head - alien, godlike
        ctx.beginPath();
        for (let i = 0; i < 8; i++) {
            let angle = (i * Math.PI * 2 / 8) - Math.PI/2;
            let radius = (i % 2 === 0) ? 18 : 12;
            let hx = char.x + char.w/2 + Math.cos(angle) * radius;
            let hy = char.y - 10 + Math.sin(angle) * radius;
            if (i === 0) ctx.moveTo(hx, hy);
            else ctx.lineTo(hx, hy);
        }
        ctx.closePath();
        ctx.fill();
        
        // Intense glowing red eyes - divine threat
        ctx.fillStyle = "#ff0000";
        ctx.shadowBlur = 20;
        ctx.shadowColor = "#ff0000";
        ctx.beginPath();
        ctx.arc(char.x + char.w/2 - 8, char.y - 8, 5, 0, Math.PI*2);
        ctx.arc(char.x + char.w/2 + 8, char.y - 8, 5, 0, Math.PI*2);
        ctx.fill();
        
        // Golden aura for god mode
        ctx.strokeStyle = "#ffff00";
        ctx.lineWidth = 2;
        ctx.shadowBlur = 30;
        ctx.shadowColor = "#ffffff";
        ctx.strokeRect(char.x - 8, char.y - 8, char.w + 16, char.h + 16);
        
        // Phase 3 warning glow
        if (char.bossPhase >= 3) {
            ctx.fillStyle = "rgba(255, 255, 255, 0.3)";
            ctx.fillRect(char.x, char.y, char.w, char.h);
        }
    }
    
    ctx.shadowBlur = 0;
    
    // ===== WINDUP INDICATOR - Souls-like telegraph =====
    if (char.windup > 0) {
        // Pulsing warning indicator
        let pulseSize = 8 + Math.sin(char.windup * 0.5) * 3;
        ctx.fillStyle = "#ffff00";
        ctx.font = "bold 28px Arial";
        ctx.fillText("!", char.x + char.w/2 - 6, char.y - 25);
        
        // Warning triangle
        ctx.beginPath();
        ctx.moveTo(char.x + char.w/2, char.y - 40);
        ctx.lineTo(char.x + char.w/2 - 10, char.y - 55);
        ctx.lineTo(char.x + char.w/2 + 10, char.y - 55);
        ctx.closePath();
        ctx.fillStyle = "rgba(255, 255, 0, 0.7)";
        ctx.fill();
    }
    
    return; // Skip sprite and geometric drawing
    /*
    // ========== LEGACY CODE BELOW (UNREACHABLE) ==========
    let state = char.animState;
    
    // Try to use sprite-based rendering first
    if (spriteSystem.isReady()) {
        let spriteState = state;
        
        // Map animation states to sprite states
        if (state === 'knockback') spriteState = 'hurt';
        if (state === 'heavyAttack') spriteState = 'attack';
        
        let spriteFrame = spriteSystem.getFrame(spriteState, char.animFrame);
        
        if (spriteFrame && spriteFrame.complete && spriteFrame.naturalHeight > 0) {
            // Draw the sprite
            let x = char.x;
            let y = char.y;
            let w = char.w;
            let h = char.h;
            
            // Handle flipping for facing direction
            ctx.save();
            if (char.facing < 0) {
                ctx.translate(x + w, 0);
                ctx.scale(-1, 1);
                x = 0;
            }
            
            // Draw sprite scaled to character size
            try {
                ctx.drawImage(spriteFrame, x, y, w, h);
            } catch(e) {
                // If sprite drawing fails, fall back to stickman
            }
            
            ctx.restore();
            return;  // Successfully drew sprite, exit function
        }
    }
    
    // Fall back to stickman/geometric drawing if no sprites available
    let frame = char.animFrame;
    let facing = char.facing;
    let x = char.x;
    let y = char.y;
    let w = char.w;
    let h = char.h;
    
    // Calculate center and proportions for stickman
    let centerX = x + w / 2;
    let bodyY = y + 15;  // Start body closer to head
    let bodyH = h - 35;  // Adjusted body height to prevent exceeding character bounds
    let headY = y + 5;
    let headR = 8;
    let legY = bodyY + bodyH;
    let legH = 18;  // Shorter legs that fit within character height
    let armY = bodyY + 2;
    
    // Base color
    let baseColor = char.isBlocking ? "#aaf" : char.color;
    if (char === player && char.instinctActive) {
        ctx.shadowBlur = 10;
        ctx.shadowColor = "red";
    }
    if (char === player && char.overclockTimer > 0) {
        ctx.shadowBlur = 20;
        ctx.shadowColor = "#00aaff"; // Blue neon to match
        baseColor = "#aaddff";
    }
    if (char.windup > 0) baseColor = "#ffffff";
    
    // Apply effect color override
    if (char.effectColor) baseColor = char.effectColor;
    
    // ========== SIMPLE BOX CHARACTER DRAWING (BOX VERSION) ==========
    ctx.fillStyle = baseColor;
    ctx.strokeStyle = baseColor;
    ctx.lineWidth = 2;
    
    // Draw main body as rectangle
    let bodyWidth = char.w;
    let bodyHeight = char.h;
    
    // Body glow for boss
    if (char.isBoss) {
        ctx.shadowBlur = 15;
        ctx.shadowColor = char.color;
    }
    
    // Draw body rectangle
    ctx.fillRect(char.x, char.y, bodyWidth, bodyHeight);
    
    // Draw head (top part of box)
    ctx.fillRect(char.x + 5, char.y - 15, bodyWidth - 10, 20);
    
    // Draw eyes (for enemy/boss)
    if (char === enemy) {
        ctx.fillStyle = "#ff0000";
        ctx.fillRect(char.x + 15, char.y - 8, 8, 6);
        ctx.fillRect(char.x + bodyWidth - 23, char.y - 8, 8, 6);
    } else {
        // Player eyes
        ctx.fillStyle = "#ffffff";
        ctx.fillRect(char.x + 15, char.y - 8, 6, 4);
        ctx.fillRect(char.x + bodyWidth - 21, char.y - 8, 6, 4);
    }
    
    // Draw weapon indicator
    if (char === player) {
        ctx.fillStyle = char.weapon === 'katana' ? "#00ffff" : "#ffff00";
        if (char.weapon === 'katana') {
            // Draw katana blade
            let weaponX = char.x + (facing > 0 ? bodyWidth - 5 : 5);
            ctx.fillRect(weaponX - 2, char.y + 10, 4, 35);
            // Blade glow
            ctx.shadowBlur = 10;
            ctx.shadowColor = "#00ffff";
            ctx.fillRect(weaponX - 1, char.y + 5, 2, 40);
        } else {
            // Draw pistol
            let gunX = char.x + (facing > 0 ? bodyWidth : -15);
            ctx.fillStyle = "#666";
            ctx.fillRect(gunX, char.y + 15, 15, 8);
        }
    }
    
    ctx.shadowBlur = 0;
    
    // Draw attack windup indicator
    if (char.windup > 0) {
        ctx.fillStyle = "#ffffff";
        ctx.font = "30px Arial";
        ctx.fillText("!", char.x + bodyWidth/2 - 5, char.y - 20);
    }
    
    return; // Skip the rest of the geometric drawing
    
    // ========== GEOMETRIC CHARACTER DRAWING FUNCTION (DISABLED) ==========
    function drawGeometricChar(headX, headY, bodyX, bodyY, bodyH, armAngleL, armAngleR, legAngleL, legAngleR, hasWeapon, weaponType) {
        let isEnemy = (char === enemy);
        let isBossChar = char.isBoss;
        
        // Draw head as different shape based on character type
        if (isEnemy) {
            // BOSS: Menacing octagon with spikes
            let headR_boss = headR * 1.8;
            ctx.beginPath();
            for (let i = 0; i < 8; i++) {
                let angle = (i * Math.PI * 2 / 8) - Math.PI / 2;
                let radius = (i % 2 === 0) ? headR_boss : headR_boss * 0.7;
                let hx = headX + Math.cos(angle) * radius;
                let hy = headY + Math.sin(angle) * radius;
                if (i === 0) ctx.moveTo(hx, hy);
                else ctx.lineTo(hx, hy);
            }
            ctx.closePath();
            ctx.fill();
            
            // Boss eye glow
            ctx.fillStyle = "#ff0000";
            ctx.beginPath();
            ctx.arc(headX - 4, headY, 3, 0, Math.PI * 2);
            ctx.arc(headX + 4, headY, 3, 0, Math.PI * 2);
            ctx.fill();
        } else {
            // PLAYER: Sleek hexagon
            let headR_geo = headR * 1.3;
            ctx.beginPath();
            for (let i = 0; i < 6; i++) {
                let angle = (i * Math.PI * 2 / 6) - Math.PI / 2;
                let hx = headX + Math.cos(angle) * headR_geo;
                let hy = headY + Math.sin(angle) * headR_geo;
                if (i === 0) ctx.moveTo(hx, hy);
            else ctx.lineTo(hx, hy);
            }
            ctx.closePath();
            ctx.fill();
        }
        
        // Draw body - different style for boss vs player
        let torsoW = isBossChar ? 28 : 18;
        let torsoH = bodyH;
        
        if (isEnemy) {
            // BOSS: Menacing armor with angular spikes
            ctx.beginPath();
            ctx.moveTo(bodyX - torsoW/2, bodyY);
            ctx.lineTo(bodyX + torsoW/2, bodyY);
            ctx.lineTo(bodyX + torsoW/2 + 5, bodyY + torsoH * 0.3);
            ctx.lineTo(bodyX + torsoW/3, bodyY + torsoH);
            ctx.lineTo(bodyX - torsoW/3, bodyY + torsoH);
            ctx.lineTo(bodyX - torsoW/2 - 5, bodyY + torsoH * 0.3);
            ctx.closePath();
            ctx.stroke();
            ctx.fill();
            
            // Boss chest core glow
            ctx.fillStyle = "#ff0000";
            ctx.beginPath();
            ctx.arc(bodyX, bodyY + torsoH * 0.4, 6, 0, Math.PI * 2);
            ctx.fill();
        } else {
            // PLAYER: Sleek parallelogram torso
            ctx.beginPath();
            ctx.moveTo(bodyX - torsoW/2, bodyY);
            ctx.lineTo(bodyX + torsoW/2, bodyY);
            ctx.lineTo(bodyX + torsoW/3, bodyY + torsoH);
            ctx.lineTo(bodyX - torsoW/3, bodyY + torsoH);
            ctx.closePath();
            ctx.stroke();
            ctx.fill();
        }
        
        // Draw arms as geometric rectangles with joints
        let armLength = 16;
        let armWidth = 6;
        
        // Left arm (rotated rectangle)
        ctx.save();
        ctx.translate(bodyX, armY);
        ctx.rotate(armAngleL);
        ctx.fillRect(-armWidth/2, 0, armWidth, armLength);
        // Left elbow joint
        ctx.beginPath();
        ctx.arc(0, armLength, 3, 0, Math.PI * 2);
        ctx.fill();
        ctx.restore();
        
        // Right arm (rotated rectangle)
        ctx.save();
        ctx.translate(bodyX, armY);
        ctx.rotate(armAngleR);
        ctx.fillRect(-armWidth/2, 0, armWidth, armLength);
        // Right elbow joint
        ctx.beginPath();
        ctx.arc(0, armLength, 3, 0, Math.PI * 2);
        ctx.fill();
        ctx.restore();
        
        // Draw legs as geometric shapes
        let legLength = 16;
        let legWidth = 8;
        
        // Left leg
        ctx.save();
        ctx.translate(bodyX, legY);
        ctx.rotate(legAngleL);
        ctx.fillRect(-legWidth/2, 0, legWidth, legLength);
        // Left knee joint
        ctx.beginPath();
        ctx.arc(0, legLength * 0.5, 4, 0, Math.PI * 2);
        ctx.fill();
        ctx.restore();
        
        // Right leg
        ctx.save();
        ctx.translate(bodyX, legY);
        ctx.rotate(legAngleR);
        ctx.fillRect(-legWidth/2, 0, legWidth, legLength);
        // Right knee joint
        ctx.beginPath();
        ctx.arc(0, legLength * 0.5, 4, 0, Math.PI * 2);
        ctx.fill();
        ctx.restore();
        
        // Draw geometric weapons
        if (hasWeapon && char === player) {
            let weaponAngle = armAngleR;
            
            // Calculate right arm end position in world coordinates
            let rightArmEndX = bodyX + Math.cos(weaponAngle) * armLength;
            let rightArmEndY = armY + Math.sin(weaponAngle) * armLength;
            
            // Adjust for character facing direction
            if (facing < 0) {
                weaponAngle = Math.PI - weaponAngle;
            }
            
            if (weaponType === 'katana') {
                // Geometric katana - angular blade (drawn at arm end position)
                ctx.strokeStyle = "#00ffff";
                ctx.lineWidth = 4;  // Thicker for visibility
                
                // Handle
                let handleLen = 12;
                ctx.beginPath();
                ctx.moveTo(rightArmEndX, rightArmEndY);
                ctx.lineTo(rightArmEndX + Math.cos(weaponAngle) * handleLen, rightArmEndY + Math.sin(weaponAngle) * handleLen);
                ctx.stroke();
                
                // Blade with glow
                ctx.shadowBlur = 20;
                ctx.shadowColor = "#00ffff";
                let swordLen = 40;
                let bladeEndX = rightArmEndX + Math.cos(weaponAngle) * (handleLen + swordLen);
                let bladeEndY = rightArmEndY + Math.sin(weaponAngle) * (handleLen + swordLen);
                ctx.beginPath();
                ctx.moveTo(rightArmEndX + Math.cos(weaponAngle) * handleLen, rightArmEndY + Math.sin(weaponAngle) * handleLen);
                ctx.lineTo(bladeEndX, bladeEndY);
                ctx.stroke();
                
                // Blade tips
                ctx.beginPath();
                ctx.moveTo(bladeEndX, bladeEndY);
                ctx.lineTo(bladeEndX + Math.cos(weaponAngle + 0.3) * 8, bladeEndY + Math.sin(weaponAngle + 0.3) * 8);
                ctx.moveTo(bladeEndX, bladeEndY);
                ctx.lineTo(bladeEndX + Math.cos(weaponAngle - 0.3) * 8, bladeEndY + Math.sin(weaponAngle - 0.3) * 8);
                ctx.stroke();
                ctx.shadowBlur = 0;
            } else if (weaponType === 'pistol') {
                // Geometric gun at arm position
                let gunX = rightArmEndX;
                let gunY = rightArmEndY;
                
                ctx.fillStyle = "#444";
                // Grip
                ctx.beginPath();
                ctx.moveTo(gunX - 4, gunY);
                ctx.lineTo(gunX + 4, gunY);
                ctx.lineTo(gunX + 2, gunY + 12);
                ctx.lineTo(gunX - 2, gunY + 12);
                ctx.closePath();
                ctx.fill();
                
                // Main body
                ctx.fillStyle = "#666";
                ctx.beginPath();
                ctx.moveTo(gunX - 6, gunY - 5);
                ctx.lineTo(gunX + 14, gunY - 2);
                ctx.lineTo(gunX + 14, gunY + 3);
                ctx.lineTo(gunX - 6, gunY + 5);
                ctx.closePath();
                ctx.fill();
                
                // Slide
                ctx.fillStyle = "#888";
                ctx.fillRect(gunX - 4, gunY - 3, 14, 4);
                
                // Muzzle flash
                if (state === 'shoot' && frame < 2) {
                    ctx.fillStyle = "#ffff00";
                    ctx.shadowBlur = 30;
                    ctx.shadowColor = "yellow";
                    ctx.beginPath();
                    ctx.arc(gunX + 16, gunY, 8, 0, Math.PI * 2);
                    ctx.fill();
                    ctx.shadowBlur = 0;
                }
            }
        }
        
        // Block shield (geometric hexagon)
        if (state === 'block') {
            ctx.strokeStyle = "#0088ff";
            ctx.lineWidth = 3;
            let shieldAngle = facing > 0 ? 0 : Math.PI;
            let shieldDist = 25;
            let shieldX = bodyX + Math.cos(shieldAngle) * shieldDist;
            let shieldY = armY;
            // Draw hexagon shield
            ctx.beginPath();
            for (let i = 0; i < 6; i++) {
                let angle = (i * Math.PI * 2 / 6) + (frame * 0.05);
                let sx = shieldX + Math.cos(angle) * 18;
                let sy = shieldY + Math.sin(angle) * 18;
                if (i === 0) ctx.moveTo(sx, sy);
                else ctx.lineTo(sx, sy);
            }
            ctx.closePath();
            ctx.stroke();
        }
    }
    
    // Calculate animation values based on state
    let headX = centerX;
    let bodyX = centerX;
    let armAngleL = 0, armAngleR = 0;
    let legAngleL = 0, legAngleR = 0;
    
    // Determine weapon type
    let weaponType = (char === player) ? char.weapon : 'none';
    let hasWeapon = (char === player);
    
    // ========== STATE-SPECIFIC ANIMATIONS ==========
    switch(state) {
        case 'idle':
            // Idle breathing - smooth bob and arm sway with interpolation
            let bob = Math.sin(frame * 0.4) * 2;
            let breathe = Math.sin(frame * 0.2) * 1;  // Subtle chest expansion
            headX = centerX;
            bodyX = centerX;
            armAngleL = -Math.PI / 4 + Math.sin(frame * 0.25) * 0.1 + breathe * 0.05;
            armAngleR = Math.PI / 4 + Math.sin(frame * 0.25 + 0.5) * 0.1 - breathe * 0.05;
            legAngleL = -0.1 + Math.sin(frame * 0.3) * 0.05;
            legAngleR = 0.1 - Math.sin(frame * 0.3) * 0.05;
            drawGeometricChar(headX, headY + bob, bodyX, bodyY + bob, bodyH, armAngleL, armAngleR, legAngleL, legAngleR, hasWeapon, weaponType);
            break;
            
        case 'run':
            // Running - smooth arm and leg alternation with lean
            let runPhase = frame * 1.0;  // Increased speed for smoother run cycle
            let runLean = facing * 0.15;
            headX = centerX + runLean * 5;
            bodyX = centerX + runLean * 3;
            // Arms swing opposite to legs for natural running motion
            armAngleL = Math.sin(runPhase) * 0.7;
            armAngleR = Math.sin(runPhase + Math.PI) * 0.7;
            // Legs have larger range of motion
            legAngleL = Math.sin(runPhase) * 0.7;
            legAngleR = Math.sin(runPhase + Math.PI) * 0.7;
            drawGeometricChar(headX, headY, bodyX, bodyY + 2, bodyH, armAngleL, armAngleR, legAngleL, legAngleR, hasWeapon, weaponType);
            break;
            
        case 'jump':
            // Jumping - tucked pose, arms up
            headX = centerX;
            bodyX = centerX;
            armAngleL = -Math.PI / 2 - 0.3;  // Both arms up
            armAngleR = -Math.PI / 2 + 0.3;
            legAngleL = 0.3;  // Legs tucked
            legAngleR = -0.3;
            drawGeometricChar(headX, headY + 5, bodyX, bodyY + 3, bodyH - 5, armAngleL, armAngleR, legAngleL, legAngleR, hasWeapon, weaponType);
            break;
            
        case 'fall':
            // Falling - spread limbs
            headX = centerX;
            bodyX = centerX;
            armAngleL = -Math.PI / 2 - 0.5;  // Arms out wide
            armAngleR = -Math.PI / 2 + 0.5;
            legAngleL = 0.4;
            legAngleR = -0.4;
            drawGeometricChar(headX, headY, bodyX, bodyY + 3, bodyH, armAngleL, armAngleR, legAngleL, legAngleR, hasWeapon, weaponType);
            break;
            
        case 'attack':
            // Melee attack - sword swing
            let swingPhase = (frame % 4);
            headX = centerX + facing * 3;
            bodyX = centerX + facing * 2;
            armAngleL = -Math.PI / 4;
            armAngleR = facing > 0 ? 
                (swingPhase < 2 ? Math.PI / 2 : Math.PI / 4) : 
                (swingPhase < 2 ? -Math.PI / 2 : -Math.PI / 4);  // Weapon arm swings
            legAngleL = -0.2;
            legAngleR = 0.2;
            drawGeometricChar(headX, headY, bodyX, bodyY, bodyH, armAngleL, armAngleR, legAngleL, legAngleR, hasWeapon, weaponType);
            
            // Attack swing effect
            if (frame < 3) {
                ctx.strokeStyle = "#00ffff";
                ctx.lineWidth = 2;
                ctx.beginPath();
                let arcStart = facing > 0 ? -Math.PI / 2 : Math.PI / 2;
                ctx.arc(centerX + facing * 15, bodyY + 10, 25, arcStart, arcStart + (facing > 0 ? Math.PI : -Math.PI));
                ctx.stroke();
                spawnParticles(centerX + facing * 25, bodyY + 10, "#00ffff", 3);
            }
            break;
            
        case 'heavyAttack':
            // Heavy attack - wind up then swing
            let heavyFrame = char.windup > 5 ? Math.floor(char.windup / 5) : frame;
            let heavyOffset = (4 - heavyFrame) * 0.3 * facing;
            headX = centerX + heavyOffset * 5;
            bodyX = centerX + heavyOffset * 3;
            armAngleL = -Math.PI / 4;
            armAngleR = heavyFrame < 2 ? Math.PI / 4 : (facing > 0 ? -Math.PI / 4 : Math.PI / 4);  // Wind up then strike
            legAngleL = -0.3 * heavyFrame;
            legAngleR = 0.3 * heavyFrame;
            drawGeometricChar(headX, headY, bodyX, bodyY, bodyH, armAngleL, armAngleR, legAngleL, legAngleR, hasWeapon, weaponType);
            
            // Charging effect
            if (char.windup > 10) {
                ctx.strokeStyle = "#ff0000";
                ctx.lineWidth = 3;
                ctx.beginPath();
                ctx.arc(centerX, bodyY + bodyH / 2, 30 + Math.sin(frame * 0.5) * 10, 0, Math.PI * 2);
                ctx.stroke();
            }
            break;
            
        case 'shoot':
            // Shooting - gun raised
            let recoil = (frame < 2) ? -5 : 0;
            headX = centerX + recoil * 0.5;
            bodyX = centerX + recoil * 0.3;
            armAngleL = -Math.PI / 3;
            armAngleR = facing > 0 ? Math.PI / 6 : Math.PI - Math.PI / 6;  // Gun arm forward
            legAngleL = -0.1;
            legAngleR = 0.1;
            drawGeometricChar(headX, headY, bodyX, bodyY, bodyH, armAngleL, armAngleR, legAngleL, legAngleR, hasWeapon, weaponType);
            
            // Muzzle flash particles
            if (frame < 2) spawnParticles(centerX + facing * 30, bodyY + 5, "#ffff00", 5);
            break;
            
        case 'block':
            // Blocking - arms crossed or raised
            headX = centerX;
            bodyX = centerX;
            armAngleL = facing > 0 ? Math.PI / 3 : -Math.PI / 3;
            armAngleR = facing > 0 ? -Math.PI / 4 : Math.PI / 4;
            legAngleL = -0.2;
            legAngleR = 0.2;
            drawGeometricChar(headX, headY, bodyX, bodyY, bodyH, armAngleL, armAngleR, legAngleL, legAngleR, hasWeapon, weaponType);
            break;
            
        case 'knockback':
            // Knockback - ragdoll, limbs flailing
            let knockDir = char.vx > 0 ? 1 : -1;
            headX = centerX + knockDir * 5;
            bodyX = centerX;
            armAngleL = -Math.PI / 2 + knockDir * 0.5;
            armAngleR = -Math.PI / 2 - knockDir * 0.5;
            legAngleL = knockDir * 0.5;
            legAngleR = -knockDir * 0.5;
            drawGeometricChar(headX, headY + 3, bodyX, bodyY + 3, bodyH, armAngleL, armAngleR, legAngleL, legAngleR, hasWeapon, weaponType);
            
            // Dazed stars
            if (frame % 2 === 0) {
                ctx.fillStyle = "#ffff00";
                ctx.font = "14px Arial";
                ctx.fillText("★", headX + knockDir * -15, headY - 12);
                ctx.fillText("☆", headX + knockDir * 10, headY - 18);
            }
            spawnParticles(centerX, bodyY + bodyH / 2, "#ff0000", 2);
            break;
            
        case 'dash':
            // Dash - streaking pose, arms back
            let dashDir = facing;
            headX = centerX + dashDir * 3;
            bodyX = centerX + dashDir * 2;
            armAngleL = -Math.PI / 2;
            armAngleR = -Math.PI / 2;
            legAngleL = 0.5 * dashDir;
            legAngleR = -0.5 * dashDir;
            drawGeometricChar(headX, headY, bodyX, bodyY, bodyH, armAngleL, armAngleR, legAngleL, legAngleR, hasWeapon, weaponType);
            
            // Speed lines
            ctx.strokeStyle = "#00ffff";
            ctx.lineWidth = 1;
            for (let i = 0; i < 3; i++) {
                ctx.beginPath();
                ctx.moveTo(centerX, bodyY + 10 + i * 10);
                ctx.lineTo(centerX - dashDir * 25, bodyY + 10 + i * 10);
                ctx.stroke();
            }
            spawnParticles(centerX - dashDir * 15, bodyY + bodyH / 2, "#00ffff", 3);
            break;
            
        case 'switch':
            // Weapon switch - dramatic pose with effects
            headX = centerX;
            bodyX = centerX;
            // Arms spread wide during switch
            armAngleL = -Math.PI / 2;
            armAngleR = Math.PI / 2;
            legAngleL = -0.2;
            legAngleR = 0.2;
            drawGeometricChar(headX, headY, bodyX, bodyY, bodyH, armAngleL, armAngleR, legAngleL, legAngleR, hasWeapon, weaponType);
            
            // Dramatic switch effect - expanding rings
            ctx.strokeStyle = char.weapon === 'katana' ? "#00ffff" : "#ffff00";
            ctx.lineWidth = 3;
            ctx.shadowBlur = 20;
            ctx.shadowColor = char.weapon === 'katana' ? "#00ffff" : "#ffff00";
            
            // Concentric expanding rings
            for (let i = 0; i < 3; i++) {
                let ringRadius = (frame * 8) + (i * 15);
                ctx.beginPath();
                ctx.arc(centerX, bodyY + bodyH / 2, ringRadius, 0, Math.PI * 2);
                ctx.globalAlpha = Math.max(0, 1 - (ringRadius / 80));
                ctx.stroke();
            }
            ctx.globalAlpha = 1;
            ctx.shadowBlur = 0;
            
            // Particle burst
            spawnParticles(centerX, bodyY + bodyH / 2, char.weapon === 'katana' ? "#00ffff" : "#ffff00", 8);
            break;
            
        default:
            // Default idle pose
            drawGeometricChar(centerX, headY, centerX, bodyY, bodyH, -Math.PI / 4, Math.PI / 4, -0.1, 0.1, hasWeapon, weaponType);
    }
    
    ctx.shadowBlur = 0;
    */ // END LEGACY CODE
}

function handleInput(char, type) {
    if (char.attackCooldown > 0 || char.windup > 0 || char.hitstun > 0 || char.silenceTimer > 0) return;
    char.invuln = 0; 
    
    if (char === player) {
        if (type === "switch") {
            // Allow switch if cooldown ready OR combo is fully charged
            if (char.switchCooldown > 0 && char.comboCharge < 100) return;
            
            char.weapon = (char.weapon === 'katana') ? 'pistol' : 'katana';
            char.switchCooldown = 60;
            char.comboCharge = 0;  // Reset combo charge on switch
            char.dashTimer = 5;
            sfx.menuBlip();
            sfx.menuBlip();
            log(`WEAPON: ${char.weapon.toUpperCase()}`, "player-cmd"); return;
        }
        if (type === "ult") { triggerOverclock(); return; }
        
        if (type === "dash") {
            if (char.dashTimer > 0 || char.dashCooldown > 0) return; 
            char.dashTimer = 15; char.dashCooldown = 45; char.invuln = 20; 
            let dashSpd = Math.max(25, char.baseSpeed * 4); 
            if (keys['w']) char.vx = char.facing * dashSpd; 
            else if (keys['a']) char.vx = -dashSpd;
            else if (keys['d']) char.vx = dashSpd; 
            else char.vx = -char.facing * 20; 
            shakeScreen(); spawnParticles(char.x, char.y+20, "#00ffff", 8);
            sfx.dash();
            for(let i=0; i<3; i++) char.ghosts.push({x: char.x - (char.vx*i*0.2), y: char.y, alpha: 0.5});
            return;
        }

        if (type === "ex") {
            if (char.exCooldown > 0) { if(frameCount % 60 === 0) log("EX RECHARGING...", "err"); return; }
            if (glitch < 25) { log("NEED 25 GLITCH", "err"); return; }
            glitch -= 25; char.exCooldown = 60; 
            if (char.weapon === 'katana') {
                let startX = char.x; char.x += char.facing * 400; 
                if(char.x < 0) char.x = 0; if(char.x > 760) char.x = 760;
                spawnAttack(char, "void", startX);
                sfx.dash();
                for(let i=0; i<5; i++) char.ghosts.push({x: startX + (i * (char.x - startX)/5), y: char.y, alpha: 0.8 - (i*0.1)});
            } else spawnProjectile(char, 'spread');
            return;
        }
    }

    if (type === "heavy") {
        if (char === player && keys['s']) { spawnAttack(char, "bomb"); return; }
        if (char === player && char.weapon === 'pistol') { spawnProjectile(char, 'burst'); char.attackCooldown = 30; return; }
        char.windup = 15; 
    } else if (type === "light") {
        if (char === player && char.weapon === 'pistol') { spawnProjectile(char, 'single'); char.attackCooldown = 15; } 
        else {
            if (char.comboTimer > 0) char.comboStep = (char.comboStep + 1) % 3; else char.comboStep = 0;
            char.comboTimer = 60; spawnAttack(char, "light");
        }
    }
}

function spawnProjectile(attacker, mode) {
    let dmg = 350; if (mode === 'burst') dmg = 300;
    if (attacker === player) {
        sfx.shot();
        if (player.instinctActive) dmg *= 1.12;
        if (player.overclockTimer > 0) dmg *= 1.2;
        dmg *= player.dmgBuff;
        let comboBonus = player.hits * 0.02;
        dmg *= (1 + comboBonus);
        spawnParticles(attacker.x + (attacker.facing*40), attacker.y + 20, "#ffff00", 5);
    }
    let angles = [0]; if (mode === 'spread') angles = [-0.2, 0, 0.2]; if (mode === 'storm') { angles = [-0.4, -0.2, 0, 0.2, 0.4]; dmg = 200; }
    angles.forEach(angle => {
        projectiles.push({ x: attacker.x + (attacker.w/2), y: attacker.y + 20, vx: attacker.facing * 18, vy: angle * 10, owner: attacker, damage: dmg, life: 35 });
    });
    if (mode === 'burst') setTimeout(() => spawnProjectile(attacker, 'echo'), 100);
}

function spawnAttack(attacker, type, originX = 0) {
    let heavy = (type === "heavy"); let spike = (type === "spike"); let bomb = (type === "bomb"); let voidSlash = (type === "void");
    let cdMult = 1; if (attacker === enemy && difficultyLevel > 1) cdMult = 0.8;
    attacker.attackCooldown = (heavy ? 40 : 15) * cdMult;
    if (spike) attacker.attackCooldown = 120; if (bomb) attacker.attackCooldown = 80; if (voidSlash) attacker.attackCooldown = 40;
    let dir = attacker.facing;
    let baseDmg = heavy ? 800 : 350; 
    let stunTime = heavy ? 30 : 10;
    
    // SFX
    if (attacker === player) {
        if (heavy) sfx.attackHeavy(); else sfx.attackLight();
    }

    if (bomb) { baseDmg = 1000; stunTime = 40; } if (voidSlash) { baseDmg = 1200; stunTime = 50; }
    
    if (attacker === player) {
        if (player.instinctActive) baseDmg *= 1.12;
        if (player.overclockTimer > 0) baseDmg *= 1.2;
        baseDmg *= player.dmgBuff;
        let comboBonus = player.hits * 0.02;
        baseDmg *= (1 + comboBonus);
    }
    let atkBox = { owner: attacker, x: attacker.x + (dir === 1 ? attacker.w : -60), y: attacker.y + 10, w: heavy ? 90 : 60, h: 40, damage: baseDmg, stun: stunTime, knockback: heavy ? 15 : 5, time: 8, type: type };
    if (spike) { projectiles.push({ x: attacker.x, y: groundY - 40, vx: (attacker.x < player.x ? 1 : -1) * 10, vy: 0, owner: attacker, damage: 600, life: 16, w: 20, h: 40, type: 'ground_shock' }); return; }
    if (bomb) { atkBox.shape = 'circle'; atkBox.r = 10; atkBox.maxR = 150; atkBox.x = attacker.x + 20; atkBox.y = attacker.y + 20; spawnParticles(attacker.x, attacker.y, "#00ffff", 30); effects.push({type: 'explosion', x: attacker.x + 20, y: attacker.y + 20, r: 10, maxR: 150, color: "rgba(0,255,255,0.5)", life: 1}); shakeScreen(); log("KERNEL PANIC DETECTED!", "buff"); sfx.explosion(); }
    if (voidSlash) { let dist = Math.abs(attacker.x - originX); atkBox.w = dist + 60; atkBox.x = Math.min(attacker.x, originX); atkBox.knockback = 0; atkBox.time = 8; effects.push({ type: 'slash', x: atkBox.x, y: atkBox.y, w: atkBox.w, h: atkBox.h, life: 10 }); }
    attacker.attackBox = atkBox;
}

function checkCollisions() {
    [player, enemy].forEach(attacker => {
        if (!attacker.attackBox) return;
        let ab = attacker.attackBox;
        let defender = (attacker === player) ? enemy : player;
        if (defender.invuln > 0 || (defender === player && player.godMode)) return;
        
        let hit = false;
        if (ab.shape === 'circle') {
            ab.r += 10; if (ab.r >= ab.maxR) attacker.attackBox = null;
            let testX = ab.x; let testY = ab.y;
            if (ab.x < defender.x) testX = defender.x; else if (ab.x > defender.x + defender.w) testX = defender.x + defender.w;
            if (ab.y < defender.y) testY = defender.y; else if (ab.y > defender.y + defender.h) testY = defender.y + defender.h;
            let distX = ab.x - testX; let distY = ab.y - testY;
            if (Math.sqrt((distX*distX) + (distY*distY)) <= ab.r) hit = true;
        } else {
            if (ab.x < defender.x + defender.w && ab.x + ab.w > defender.x && ab.y < defender.y + defender.h && ab.y + ab.h > defender.y) hit = true;
        }

        if (hit) {
            let finalDmg = ab.damage || 600; let finalStun = ab.stun || 30;
            if (defender === player && player.hp < player.maxHp * 0.25) finalDmg *= 0.5;

            if (defender.isBlocking) { 
                finalDmg *= 0.1; finalStun = 0; 
                spawnParticles(defender.x + 20, defender.y + 20, "#0088ff", 15); 
                sfx.block();
                if(defender === player) log("ATTACK BLOCKED", "buff"); 
            } 
            else { 
                defender.hitstun = finalStun; 
                spawnParticles(defender.x + 20, defender.y + 20, defender.color, 10); 
                sfx.hit();
            }
            defender.hp -= finalDmg;
            
            if (attacker === player) {
                player.hp = Math.min(player.maxHp, player.hp + (finalDmg * 0.05)); // 5% Lifesteal (reduced from 15%)
                player.hits++; player.lastHitTime = 120;
                player.comboCharge = Math.min(100, player.comboCharge + 8);  // 8% per hit, ~12 hits for switch
                roundStats.dmgDealt += finalDmg; glitch += 12;
            }
            if (attacker === enemy) roundStats.dmgTaken += finalDmg;
            
            if (!defender.isBlocking) { let kn = ab.knockback || 20; let impactDir = (attacker.x < defender.x) ? 1 : -1; defender.vx = impactDir * kn; defender.vy = -4; defender.onGround = false; defender.invuln = 30; }
            if (ab.shape !== 'circle' && ab.type !== 'spike') attacker.attackBox = null;
        }
        if (ab && ab.shape !== 'circle') { ab.time--; if (ab.time <= 0) attacker.attackBox = null; }
    });
}

function updateAI() {
    if (enemy.aiPaused || enemy.hitstun > 0 || enemy.windup > 0) return;
    const dx = player.x - enemy.x; const dist = Math.abs(dx); let speed = enemy.baseSpeed || 4; 
    let aiCooldownMult = enemy.isGod ? 0.5 : 1; 
    if(enemy.isGod) speed *= 1.5;
    if (enemy.isGod && enemy.hp < enemy.maxHp) enemy.hp += 50;
    
    // ===== FIRST ENEMY (GLITCH PROXY) - Practice AI =====
    // Simple, slow, telegraphed attacks - practice before real bosses
    if (!enemy.isBoss) {
        // Very slow movement - practice dummy
        speed = 2; 
        
        // Only attack occasionally - with long windup telegraph
        if (dist < 80 && enemy.attackCooldown <= 0 && Math.random() < 0.015) {
            // Wind up telegraph - player can see it coming
            enemy.windup = 30;
            enemy.effectColor = "#ff0055";
            enemy.attackCooldown = 60;
            log("GLITCH SURGE", "err");
        }
        
        // Simple chase when far
        if (dist > 150) {
            enemy.vx = (dx > 0) ? speed : -speed;
        } else {
            enemy.vx = 0;
        }
        
        // Rare jump
        if (enemy.onGround && Math.random() < 0.005) {
            enemy.vy = enemy.jumpForce;
        }
        
        return; // Don't use boss AI for first enemy
    }
    
    // ===== SECOND BOSS (SYSTEM ERROR) - Mini-boss AI =====
    // More aggressive but fair attacks
    if (enemy.isBoss && !enemy.isGod) {
        // Mini-boss specific attacks
    } 

    let ultThreshold = enemy.isGod ? 0.5 : 0.3;
    if (!systemOverwrite && enemy.hp < enemy.maxHp * ultThreshold && enemy.isBoss) {
        systemOverwrite = true; enemy.dmgBuff = 2; enemy.hp += (enemy.isGod ? 50000 : 25000);
        log(enemy.isGod ? ">>> DIVINE INTERVENTION <<<" : "/// SYSTEM OVERWRITE INITIATED ///", "err");
        shakeScreen(); sfx.explosion();
    }

    // === BOSS PHASE 2 & 3 ATTACKS ===
    if (enemy.isBoss) {
        // Phase 2 (Boss) Special Attacks
        if (!enemy.isGod && enemy.attackCooldown <= 0) {
            // Rapid combo attack
            if (Math.random() < 0.008 && dist < 250) {
                enemy.attackCooldown = 30;
                enemy.effectColor = "#ff0000";
                enemy.effectTimer = 15;
                spawnAttack(enemy, "light");
                log("RAPID STRIKE!", "err");
                sfx.rapidAttack();
                setTimeout(() => spawnAttack(enemy, "light"), 100);
                setTimeout(() => spawnAttack(enemy, "light"), 200);
                spawnParticles(enemy.x, enemy.y, "#ff0000", 10);
                return;
            }
            // Ground slam attack
            if (Math.random() < 0.006 && dist < 300) {
                enemy.attackCooldown = 60;
                enemy.effectColor = "#ff4400";
                enemy.effectTimer = 20;
                spawnAttack(enemy, "spike");
                // Create shockwave
                effects.push({
                    type: 'shockwave',
                    x: enemy.x,
                    y: groundY,
                    r: 10,
                    maxR: 200,
                    damage: 500,
                    life: 30,
                    color: "rgba(255, 68, 0, 0.7)"
                });
                log("GROUND SLAM!", "err");
                shakeScreen();
                sfx.slam();
                spawnParticles(enemy.x, groundY, "#ff4400", 20);
                return;
            }
        }
        
        // Phase 3 (God Mode) Special Attacks
        if (enemy.isGod && enemy.attackCooldown <= 0) {
            // Laser beam attack
            if (Math.random() < 0.01 && dist < 500) {
                enemy.attackCooldown = 45;
                enemy.effectColor = "#ffffff";
                enemy.effectTimer = 30;
                log("DIVINE JUDGMENT!", "admin");
                // Create laser beam effect
                effects.push({
                    type: 'laser',
                    startX: enemy.x + enemy.w/2,
                    startY: enemy.y + enemy.h/2,
                    targetX: player.x + player.w/2,
                    targetY: player.y + player.h/2,
                    width: 40,
                    life: 20,
                    damage: 800
                });
                shakeScreen();
                sfx.laser();
                spawnParticles(enemy.x + enemy.w/2, enemy.y + enemy.h/2, "#ffffff", 30);
                return;
            }
            // Screen wipe attack
            if (Math.random() < 0.005) {
                enemy.attackCooldown = 90;
                enemy.effectColor = "#ffff00";
                enemy.effectTimer = 40;
                log("SYSTEM PURGE!", "admin");
                effects.push({
                    type: 'screenwipe',
                    x: 0,
                    y: 0,
                    w: 0,
                    maxW: 800,
                    dir: player.x < 400 ? 1 : -1,
                    life: 40,
                    damage: 600
                });
                return;
            }
            // Reality warp - teleport attack
            if (Math.random() < 0.008 && dist > 200) {
                enemy.attackCooldown = 25;
                log("REALITY WARP", "admin");
                sfx.warp();
                // Store old position for teleport effect
                let oldX = enemy.x;
                enemy.x = player.x + (Math.random() < 0.5 ? 60 : -60);
                enemy.effectColor = "#ffffff";
                enemy.effectTimer = 10;
                // Teleport particles at old and new position
                spawnParticles(oldX, enemy.y, "#ffffff", 15);
                spawnParticles(enemy.x, enemy.y, "#ffffff", 15);
                // Immediate attack after teleport
                setTimeout(() => spawnAttack(enemy, "heavy"), 50);
                return;
            }
            // Orbital strike - random damage zones
            if (Math.random() < 0.007) {
                enemy.attackCooldown = 60;
                log("ORBITAL STRIKE", "admin");
                let targetX = player.x + (Math.random() - 0.5) * 100;
                effects.push({
                    type: 'orbital',
                    x: targetX,
                    y: -50,
                    targetY: groundY - 20,
                    r: 30,
                    life: 30,
                    damage: 700
                });
                return;
            }
        }
        
        // Regular boss attacks
        if (Math.random() < 0.005 * (enemy.isGod ? 2 : 1) && dist < 400) { 
            projectiles.push({ x: enemy.x, y: enemy.y+20, vx: Math.sign(dx)*15, vy: 0, owner: enemy, damage: 100, life: 60, type: 'silence' }); 
            log("CODE SURGE DETECTED", "err"); 
            return; 
        }
        if (!lagPulseActive && Math.random() < 0.005) { lagPulseActive = true; log("LAG PULSE: SPEED DAMPENED", "err"); setTimeout(() => lagPulseActive = false, 2500); return; } 
        if (Math.random() < 0.01 && dist > 100) { spawnProjectile(enemy, 'storm'); return; }
        if (dist > 300 && Math.random() < 0.005 * (enemy.isGod ? 3 : 1)) { enemy.x = player.x + (Math.random() < 0.5 ? 60 : -60); spawnParticles(enemy.x, enemy.y, "#ff0055", 10); log("REALITY FLICKER", "err"); }
        
        // ===== MINI-BOSS ATTACKS - Charge/Slam/Shockwave (NO SPIRAL) =====
        if (enemy.isBoss && !enemy.isGod && enemy.attackCooldown <= 0) {
            // Charge attack - knockback + stun
            if (Math.random() < 0.006 && dist > 100 && dist < 400) {
                enemy.attackCooldown = 90;
                enemy.windup = 40;
                enemy.effectColor = "#ff4400";
                log("CHARGE!", "err");
                return;
            }
            // Ground slam with shockwave
            if (Math.random() < 0.008 && dist < 300) {
                enemy.attackCooldown = 80;
                enemy.windup = 30;
                enemy.effectColor = "#cc4400";
                log("GROUND SLAM!", "err");
                return;
            }
        }
        
        // ===== GOD BOSS - INTENSIFIED SPIRAL ATTACK =====
        // Only for God boss, at 75%, 50%, 25% HP phases
        if (enemy.isGod && enemy.introComplete && enemy.attackCooldown <= 0) {
            let hpPercent = enemy.hp / enemy.maxHp;
            
            // Spiral at phase transitions (75%, 50%, 25%)
            if ((hpPercent <= 0.75 && hpPercent > 0.5) || (hpPercent <= 0.50 && hpPercent > 0.25) || hpPercent <= 0.25) {
                // Check if we recently did a spiral (don't spam)
                if (enemy.lastSpiralPhase !== Math.floor(hpPercent * 4)) {
                    enemy.lastSpiralPhase = Math.floor(hpPercent * 4);
                    
                    // INTENSIFIED SPIRAL - 6 rotations, accelerating
                    enemy.attackCooldown = 60;
                    let rotations = 6; // 6 full rotations
                    let baseSpeed = 5 + (1 - hpPercent) * 3; // Gets faster as HP drops
                    
                    for (let r = 0; r < rotations; r++) {
                        setTimeout(() => {
                            for (let i = 0; i < 8; i++) {
                                let angle = (Math.PI * 2 / 8) * i + frameCount * 0.15 + r;
                                projectiles.push({
                                    x: enemy.x + enemy.w/2,
                                    y: enemy.y + enemy.h/2,
                                    vx: Math.cos(angle) * baseSpeed,
                                    vy: Math.sin(angle) * baseSpeed,
                                    owner: enemy,
                                    damage: 400 + (1 - hpPercent) * 200, // Gets stronger
                                    life: 60,
                                    type: 'spiral',
                                    w: 12,
                                    h: 12,
                                    color: "#ff00ff"
                                });
                            }
                            spawnParticles(enemy.x + enemy.w/2, enemy.y + enemy.h/2, "#ff00ff", 20);
                        }, r * 150); // Stagger each rotation
                    }
                    log("SPIRAL OMEGA!", "admin");
                    sfx.shot();
                    return;
                }
            }
        }
    }

    if (enemy.onGround && Math.random() < 0.015) enemy.vy = enemy.jumpForce;
    if (player.windup > 0 && dist < 120 && Math.random() < 0.2) { if(enemy.dashCooldown <= 0) { enemy.dashTimer = 10; enemy.dashCooldown = 60 * aiCooldownMult; enemy.invuln = 20; enemy.vx = (Math.random()<0.5 ? -1 : 1) * 20; } else enemy.vx = -Math.sign(dx) * enemy.baseSpeed; return; }
    if (dist < 100 && Math.random() < 0.05 && enemy.dashCooldown <= 0) { enemy.dashTimer = 10; enemy.dashCooldown = 60 * aiCooldownMult; enemy.invuln = 20; enemy.vx = Math.sign(dx) * 25; return; }
    if (dist > 150) { if (enemy.attackCooldown <= 0 && Math.random() < 0.02) { spawnAttack(enemy, "spike"); return; } enemy.vx = (dx > 0) ? speed : -speed; } 
    else if (dist < 60) { enemy.vx = 0; if (enemy.attackCooldown <= 0) { if (Math.random() < 0.05) enemy.windup = 40 * aiCooldownMult; else if (Math.random() < 0.1) { enemy.vx = (dx > 0) ? 20 : -20; enemy.dashTimer = 10; } } } else enemy.vx = (dx > 0) ? speed * 0.5 : -speed * 0.5;
}

function applyPhysics(char) {
    if (char.noclip) { if (keys['w']) char.y -= 10; if (keys['s']) char.y += 10; if (keys['a']) char.x -= 10; if (keys['d']) char.x += 10; return; }
    if (char.dashTimer > 0) { char.x += char.vx; char.vy = 0; char.dashTimer--; if (frameCount % 3 === 0) char.ghosts.push({x: char.x, y: char.y, alpha: 0.5}); } 
    else {
        char.vy += gravity; char.y += char.vy; 
        let spdMult = 1;
        if (char === player) { 
            if (char.hp < char.maxHp * 0.4 && !char.instinctActive) { char.instinctActive = true; log("CATALYST INSTINCT!", "buff"); } 
            if (char.instinctActive) spdMult *= 1.2; 
            if (char.overclockTimer > 0) { char.overclockTimer--; spdMult *= 1.25; char.hitstun = 0; }
            if (lagPulseActive) spdMult *= 0.4; 
        }
        if (char.lagTimer > 0) { spdMult *= 0.5; char.lagTimer--; }
        
        // Apply velocity with improved ground friction
        let targetVx = char.vx;
        if (char.onGround) {
            // Ground friction - slow down gradually
            targetVx *= 0.85;
        }
        char.x += (targetVx * spdMult);
    }
    
    // LANDING LOGIC - improved bounce handling with platforms
    let onPlatform = false;
    
    // Check platform collisions (only if falling)
    if (char.vy >= 0) {
        for (let p of platforms) {
            // Check if character is above and falling onto platform
            if (char.x + char.w > p.x && char.x < p.x + p.w) {
                // Previous position was above platform
                let prevY = char.y - char.vy;
                if (prevY + char.h <= p.y && char.y + char.h >= p.y) {
                    char.y = p.y - char.h;
                    if (char.vy > 8) {
                        char.vy = -char.vy * 0.2;
                        if (Math.abs(char.vy) < 2) char.vy = 0;
                    } else {
                        char.vy = 0;
                    }
                    onPlatform = true;
                    break;
                }
            }
        }
    }
    
    // Ground collision (only if not on platform)
    if (!onPlatform && char.y + char.h >= groundY) { 
        char.y = groundY - char.h; 
        if (char.vy > 10) {
            char.vy = -char.vy * 0.3;
            if (Math.abs(char.vy) < 2) char.vy = 0;
        } else {
            char.vy = 0;
        }
        char.onGround = (char.vy === 0); 
        char.jumpCount = 0;
    } else if (!onPlatform) {
        char.onGround = false;
    } else {
        char.onGround = true;
        char.jumpCount = 0;
    }
    
    // Boundary checks
    if (char.x < 0) { char.x = 0; char.vx = 0; }
    if (char.x > 760) { char.x = 760; char.vx = 0; }
    
    // Update timers
    if (char.hitstun > 0) char.hitstun--; 
    if (char.attackCooldown > 0) char.attackCooldown--; 
    if (char.invuln > 0) char.invuln--;
    if (char.comboTimer > 0) char.comboTimer--;
    
    // COMBO RESET LOGIC
    if (char === player && char.hits > 0) {
        char.lastHitTime--;
        if (char.lastHitTime <= 0) { char.hits = 0; }
        if (comboHud) {
            comboHud.style.display = char.hits > 1 ? "block" : "none";
            comboCount.innerText = char.hits;
        }
    }

    if (char.windup > 0) { 
        char.windup--; 
        if (char.windup === 0) {
            // Mini-boss special attacks
            if (char.isBoss && !char.isGod && char.effectColor) {
                if (char.effectColor === "#ff4400") {
                    // CHARGE ATTACK - dash at player
                    let dx = player.x - char.x;
                    char.dashTimer = 20;
                    char.dashCooldown = 60;
                    char.invuln = 25;
                    char.vx = Math.sign(dx) * 35;
                    log("CHARGE ATTACK!", "err");
                    sfx.shot();
                } else if (char.effectColor === "#cc4400") {
                    // GROUND SLAM - create shockwave projectiles
                    for (let i = 0; i < 4; i++) {
                        let angle = (Math.PI / 2) * i;
                        projectiles.push({
                            x: char.x + char.w/2,
                            y: char.y + char.h,
                            vx: Math.cos(angle) * 8,
                            vy: -3,
                            owner: char,
                            damage: 300,
                            life: 40,
                            type: 'shockwave',
                            w: 20,
                            h: 15,
                            color: "#ff6600"
                        });
                    }
                    // Also do a ground hit
                    spawnAttack(char, "heavy");
                    log("SHOCKWAVE!", "err");
                    sfx.shot();
                }
                char.effectColor = null;
            } else {
                spawnAttack(char, "heavy"); 
            }
        }
    }
    if (char.silenceTimer > 0) char.silenceTimer--; 
    if (char.switchCooldown > 0) char.switchCooldown--; 
    if (char.dashCooldown > 0) char.dashCooldown--; 
    if (char.exCooldown > 0) char.exCooldown--;
}

function updateProjectiles() {
    for (let i = projectiles.length - 1; i >= 0; i--) {
        let p = projectiles[i]; p.x += p.vx; p.y += (p.vy || 0); p.life--;
        let target = (p.owner === player) ? enemy : player;
        
        if (p.type === 'silence') {
            ctx.fillStyle = "#0000ff"; ctx.fillRect(p.x, p.y, 10, 10);
            if (p.x < target.x + target.w && p.x + 10 > target.x && p.y < target.y + target.h && p.y + 10 > target.y) {
                target.silenceTimer = 90; 
                log("WARNING: ABILITIES SILENCED", "err");
                projectiles.splice(i, 1); continue;
            }
        }
        else if(p.type === 'ground_shock') {
            spawnParticles(p.x, p.y + 20, "#ff0000", 2);
            if (p.x < target.x + target.w && p.x + 20 > target.x && Math.abs(target.y + target.h - groundY) < 10) {
                target.hp -= 400; target.vy = -15; target.hitstun = 30; 
                sfx.hit();
                projectiles.splice(i, 1); continue;
            }
        }
        else if (p.x < target.x + target.w && p.x + 8 > target.x && p.y < target.y + target.h && p.y + 4 > target.y) {
            target.hp -= p.damage; spawnParticles(p.x, p.y, "#ffff00", 3);
            sfx.hit();
            if (p.owner === player) { 
                roundStats.dmgDealt += p.damage; glitch += 4; 
                player.hits++; player.lastHitTime = 120;
            }
            projectiles.splice(i, 1); continue;
        }
        if (p.life <= 0) projectiles.splice(i, 1);
    }
}

function updateParticles() { for (let i = particles.length - 1; i >= 0; i--) { let p = particles[i]; p.x += p.vx; p.y += p.vy; p.life--; p.size *= 0.9; if(p.life <= 0) particles.splice(i, 1); } }

function animate() {
    try {
        requestAnimationFrame(animate);
        now = Date.now(); elapsed = now - then;
        if (elapsed <= fpsInterval) return;
        then = now - (elapsed % fpsInterval);

        ctx.clearRect(0,0,canvas.width, canvas.height);
        // Only update facing direction when actually moving (don't auto-face enemy)
        if (Math.abs(player.vx) > 0.5) {
            player.facing = Math.sign(player.vx);
        }
        // Enemy still faces player
        if (enemy.x < player.x) { enemy.facing = 1; } else { enemy.facing = -1; }

        if (gameState === 'menu' || gameState === 'boot') {
            ctx.fillStyle = "#050505"; ctx.fillRect(0,0,800,400); ctx.fillStyle = "#222"; ctx.fillRect(0, groundY, 800, 40);
            ctx.fillStyle = player.customColor; ctx.fillRect(player.x, player.y, player.w, player.h); return;
        }
        if (gameState === 'lose') { drawLoseScreen(); return; }
        if (gameState === 'win') { drawWinScreen(); return; }

        frameCount++;
        player.isBlocking = false;
        if (player.hitstun <= 0 && player.windup <= 0) {
            let speed = (glitch > 30 || player.overclockTimer > 0) ? 8 : player.baseSpeed;
            if (keys['a']) { player.vx = -speed; if (player.facing === 1) player.isBlocking = true; }
            else if (keys['d']) { player.vx = speed; if (player.facing === -1) player.isBlocking = true; }
            else player.vx = 0;
            if (keys['shift']) handleInput(player, "dash");
            if (keys['q']) handleInput(player, "switch");
            if (keys['z']) handleInput(player, "ult");
            if (keys['e']) handleInput(player, "ex"); 
        }

        if(!glitchFrozen) glitch = Math.min(100, glitch + 0.04);
        if(glitchHud) glitchHud.innerText = Math.floor(glitch) + "%";
        
        // Update combo charge meter
        const comboChargeEl = document.getElementById('combo-charge');
        const comboFillEl = document.getElementById('combo-fill');
        if (comboChargeEl) {
            comboChargeEl.innerText = player.comboCharge >= 100 ? "⚡ SWITCH READY!" : `COMBO: ${player.comboCharge}%`;
            comboChargeEl.style.color = player.comboCharge >= 100 ? "#ff00ff" : "#00ffff";
        }
        if (comboFillEl) {
            comboFillEl.style.width = player.comboCharge + "%";
            comboFillEl.style.background = player.comboCharge >= 100 ? "linear-gradient(90deg, #ff00ff, #ffff00)" : "linear-gradient(90deg, #00ffff, #ff00ff)";
        }
        
        if (glitch >= 85) { if (!player.godMode && frameCount % 60 === 0) player.hp -= 300; if(glitchHud) glitchHud.style.color = "#ff0055"; } 
        else if(glitchHud) glitchHud.style.color = "#00ff41";

        // Handle cinematic boss intro
        handleCinematicIntro();
        
        updateAI(); applyPhysics(player); applyPhysics(enemy); updateProjectiles(); updateParticles(); checkCollisions();
        
        // Update animations for player and enemy
        updateAnimations(player);
        updateAnimations(enemy);

        if (player.hp <= 0 && gameState !== 'lose') { 
            gameState = 'lose'; 
            saveSystem.recordGame(false); 
            sfx.death(); // PLAY DEATH SOUND
        }
        
        // Boss phase transitions
        if (enemy.isBoss && enemy.bossPhase < enemy.bossMaxPhase) {
            let hpPercent = enemy.hp / enemy.maxHp;
            if (enemy.bossPhase === 1 && hpPercent < 0.66) {
                enemy.bossPhase = 2;
                enemy.color = "#ff4400";  // Phase 2: More aggressive red
                enemy.baseSpeed = 4;
                enemy.maxHp = 30000;
                enemy.hp = 30000;
                shakeScreen();
                log("/// PHASE 2: UNLEASHED ///", "buff");
                sfx.explosion();
            } else if (enemy.bossPhase === 2 && hpPercent < 0.33) {
                enemy.bossPhase = 3;
                enemy.color = "#ff0000";  // Phase 3: Final form
                enemy.baseSpeed = 5;
                enemy.maxHp = 40000;
                enemy.hp = 40000;
                shakeScreen();
                log("/// FINAL PHASE: GLITCH OVERLOAD ///", "buff");
                sfx.explosion();
            }
        }
        
        if (enemy.hp <= 0 && gameState !== 'win') { gameState = 'win'; saveSystem.recordGame(true); sfx.win(); }
        
        glitch = Math.max(0, Math.min(100, glitch));
        drawGame();
    } catch(e) { console.error(e); }
}

function drawGame() {
    ctx.setTransform(1, 0, 0, 1, 0, 0); 
    let bg = "#050505";
    if (systemOverwrite) bg = "#330000";
    if (lagPulseActive) bg = "#333333";
    if (enemy.isGod && Math.random() < 0.05) bg = "#ffffff"; 
    
    ctx.fillStyle = bg; ctx.fillRect(0, 0, canvas.width, canvas.height);
    
    ctx.save();
    try {
        if (glitch > 70) { let s = (glitch - 70)/2; ctx.translate((Math.random()-0.5)*s, (Math.random()-0.5)*s); }
        ctx.fillStyle = "rgba(0, 255, 255, 0.05)";
        for(let i=0; i<5; i++) ctx.fillText(String.fromCharCode(0x30A0+Math.random()*96), Math.random()*800, Math.random()*400);
        
        // Draw platforms
        ctx.fillStyle = "#222";
        platforms.forEach(p => {
            // Platform glow
            ctx.fillStyle = "rgba(50, 50, 80, 0.5)";
            ctx.fillRect(p.x - 2, p.y - 2, p.w + 4, p.h + 4);
            // Platform top
            ctx.fillStyle = "#333";
            ctx.fillRect(p.x, p.y, p.w, p.h);
            // Platform highlight
            ctx.fillStyle = "#444";
            ctx.fillRect(p.x, p.y, p.w, 3);
        });

        ctx.fillStyle = enemy.color; ctx.font = "14px monospace"; ctx.fillText(enemy.name, enemy.x - 10, enemy.y - 15);

        projectiles.forEach(p => { 
            if(p.type === 'ground_shock') { ctx.fillStyle = "#ff0000"; ctx.fillRect(p.x, p.y - 20, 20, 40); } 
            else if (p.type === 'silence') { ctx.fillStyle = "#0000ff"; ctx.fillRect(p.x, p.y, 10, 10); } 
            else if (p.type === 'spiral') { 
                // Spiral projectile - glowing orb
                ctx.shadowBlur = 15;
                ctx.shadowColor = "#ff00ff";
                ctx.fillStyle = "#ff00ff";
                ctx.beginPath();
                ctx.arc(p.x, p.y, 8, 0, Math.PI * 2);
                ctx.fill();
                // Inner glow
                ctx.fillStyle = "#ffffff";
                ctx.beginPath();
                ctx.arc(p.x, p.y, 4, 0, Math.PI * 2);
                ctx.fill();
                ctx.shadowBlur = 0;
            }
            else { ctx.fillStyle = "#ffff00"; ctx.shadowBlur = 5; ctx.shadowColor = "yellow"; ctx.fillRect(p.x, p.y, 8, 4); ctx.shadowBlur = 0; }
        });
        particles.forEach(p => { ctx.fillStyle = p.color; ctx.fillRect(p.x, p.y, p.size, p.size); });
        for(let i=effects.length-1; i>=0; i--) {
            let e = effects[i];
            if(e.type === 'slash') { ctx.fillStyle = `rgba(0, 255, 255, ${e.life/10})`; ctx.fillRect(e.x, e.y, e.w, e.h); e.life--; } 
            else if (e.type === 'explosion') {
                ctx.beginPath(); ctx.arc(e.x, e.y, e.r, 0, Math.PI*2); ctx.strokeStyle = e.color; ctx.lineWidth = 5; ctx.stroke();
                e.r += 10; if(e.r > e.maxR) e.life = 0;
            }
            else if (e.type === 'shockwave') {
                // Boss ground slam shockwave
                ctx.strokeStyle = e.color;
                ctx.lineWidth = 8;
                ctx.beginPath();
                ctx.arc(e.x + 20, e.y, e.r, -Math.PI/2, Math.PI/2);
                ctx.stroke();
                ctx.strokeStyle = "rgba(255, 68, 0, 0.3)";
                ctx.lineWidth = 20;
                ctx.beginPath();
                ctx.arc(e.x + 20, e.y, e.r * 0.8, -Math.PI/2, Math.PI/2);
                ctx.stroke();
                e.r += 15;
                // Check collision with player
                if (Math.abs(player.x - e.x) < e.r && Math.abs(groundY - player.y - player.h) < 50) {
                    player.hp -= e.damage;
                    player.hitstun = 20;
                    player.vy = -8;
                }
                if(e.r > e.maxR) e.life = 0;
                e.life--;
            }
            else if (e.type === 'laser') {
                // God mode laser beam
                ctx.strokeStyle = "#ffffff";
                ctx.lineWidth = e.width;
                ctx.shadowBlur = 30;
                ctx.shadowColor = "#ffffff";
                ctx.beginPath();
                ctx.moveTo(e.startX, e.startY);
                ctx.lineTo(e.targetX, e.targetY);
                ctx.stroke();
                // Inner beam
                ctx.strokeStyle = "#ffff00";
                ctx.lineWidth = e.width / 3;
                ctx.beginPath();
                ctx.moveTo(e.startX, e.startY);
                ctx.lineTo(e.targetX, e.targetY);
                ctx.stroke();
                ctx.shadowBlur = 0;
                // Check collision
                if (Math.abs(player.x - e.targetX) < 30) {
                    player.hp -= e.damage / 10;
                    spawnParticles(player.x + player.w/2, player.y + player.h/2, "#ffffff", 5);
                }
                e.life--;
                if(e.life <= 0) {
                    // Deal full damage on last frame
                    if (Math.abs(player.x + player.w/2 - e.targetX) < 50) {
                        player.hp -= e.damage;
                        player.hitstun = 40;
                    }
                }
            }
            else if (e.type === 'screenwipe') {
                // God mode screen wipe
                ctx.fillStyle = "rgba(255, 255, 0, 0.8)";
                let w = e.maxW * (1 - e.life / 40);
                if (e.dir === 1) {
                    ctx.fillRect(0, 0, w, canvas.height);
                } else {
                    ctx.fillRect(canvas.width - w, 0, w, canvas.height);
                }
                // Damage player if hit by wipe
                if (e.life < 35) {
                    let hitZone = e.dir === 1 ? w : canvas.width - w;
                    if (player.x + player.w/2 > hitZone - 20 && player.x + player.w/2 < hitZone + 20) {
                        player.hp -= e.damage / 20;
                    }
                }
                e.life--;
                if(e.w >= e.maxW) e.life = 0;
            }
            else if (e.type === 'orbital') {
                // Orbital strike effect
                ctx.fillStyle = "#ff0000";
                ctx.shadowBlur = 20;
                ctx.shadowColor = "#ff0000";
                // Draw incoming missile
                let progress = 1 - (e.life / 30);
                let currY = e.y + (e.targetY - e.y) * progress;
                ctx.beginPath();
                ctx.arc(e.x, currY, e.r, 0, Math.PI*2);
                ctx.fill();
                // Trail
                ctx.fillStyle = "rgba(255, 0, 0, 0.5)";
                ctx.fillRect(e.x - 5, e.y, 10, currY - e.y);
                ctx.shadowBlur = 0;
                // Impact
                if (e.life <= 1) {
                    // Explosion
                    ctx.fillStyle = "#ffff00";
                    ctx.beginPath();
                    ctx.arc(e.x, e.targetY, 50, 0, Math.PI*2);
                    ctx.fill();
                    shakeScreen();
                    spawnParticles(e.x, e.targetY, "#ff0000", 30);
                    // Damage
                    if (Math.abs(player.x - e.x) < 80) {
                        player.hp -= e.damage;
                        player.hitstun = 30;
                        player.vy = -10;
                    }
                }
                e.life--;
            }
            if(e.life<=0) effects.splice(i,1);
        }

        [player, enemy].forEach(c => {
            if (c === enemy && Math.random() < (enemy.isGod ? 0.3 : 0.1) && enemy.isBoss) {
                let offX = (Math.random() - 0.5) * 20;
                ctx.fillStyle = enemy.isGod ? "rgba(255,255,255,0.7)" : "rgba(255,0,0,0.5)"; 
                ctx.fillRect(c.x + offX, c.y, c.w, c.h);
            }
            if (c.ghosts) { for(let i=c.ghosts.length-1; i>=0; i--) { let g = c.ghosts[i]; ctx.globalAlpha = g.alpha; ctx.fillStyle = c.color; ctx.fillRect(g.x, g.y, c.w, c.h); g.alpha -= 0.05; if(g.alpha <= 0) c.ghosts.splice(i,1); } }
            ctx.globalAlpha = (c.invuln > 0 && Math.floor(frameCount/4)%2===0) ? 0.5 : 1.0;
            
            // Use animated character rendering
            drawCharacterAnimation(ctx, c, c === player);
            if (c.windup > 0 && c === enemy) { ctx.fillStyle = "#ff0000"; ctx.font = "30px Arial"; ctx.fillText("!", c.x + 10, c.y - 10); }
            if (c === player) { ctx.fillStyle = "#fff"; let scale = c.w / 40; if (c.weapon === 'pistol') ctx.fillRect(c.x + (c.facing*20*scale), c.y+(20*scale), 15*scale, 5*scale); else ctx.fillRect(c.x + (c.facing*10*scale), c.y+(5*scale), 5*scale, 40*scale); }
            
            // Boss special attack indicators
            if (c === enemy && enemy.isBoss) {
                // Show warning indicator before special attacks
                if (enemy.effectTimer > 0 && enemy.effectTimer < 15) {
                    ctx.strokeStyle = enemy.isGod ? "#ffffff" : "#ff0000";
                    ctx.lineWidth = 2;
                    ctx.setLineDash([5, 5]);
                    ctx.strokeRect(c.x - 10, c.y - 10, c.w + 20, c.h + 20);
                    ctx.setLineDash([]);
                }
                // God mode aura
                if (enemy.isGod) {
                    ctx.strokeStyle = "rgba(255, 255, 255, 0.3)";
                    ctx.lineWidth = 3;
                    let auraSize = 45 + Math.sin(frameCount * 0.1) * 5;
                    ctx.beginPath();
                    ctx.arc(c.x + c.w/2, c.y + c.h/2, auraSize, 0, Math.PI*2);
                    ctx.stroke();
                }
            }
            ctx.globalAlpha = 1.0;
            if (c.attackBox && drawHitboxes) { 
                ctx.strokeStyle = (c === player) ? "#00ffff" : "#ff0055"; 
                if(c.attackBox.shape === 'circle') { ctx.beginPath(); ctx.arc(c.attackBox.x, c.attackBox.y, c.attackBox.r, 0, Math.PI*2); ctx.stroke(); }
                else ctx.strokeRect(c.attackBox.x, c.attackBox.y, c.attackBox.w, c.attackBox.h); 
            }
        });

        if(showUI) {
            drawBar(20, 20, player.hp, 15000, "#00ffff", "INTEGRITY");
            drawBar(580, 20, enemy.hp, enemy.maxHp, enemy.color, "GLITCH");
            let gCol = glitch > 85 ? "#ff0000" : (glitch > 50 ? "#ffa500" : "#ffff00");
            drawBar(250, 20, glitch, 100, gCol, "INSTABILITY");
        }
    } finally { ctx.restore(); }
}

function drawLoseScreen() {
    ctx.fillStyle = "#330000"; ctx.fillRect(0, 0, 800, 400);
    ctx.fillStyle = "#ff0000"; ctx.font = "80px VT323"; ctx.fillText("FATAL ERROR", 240, 150);
    ctx.fillStyle = "#ff4444"; ctx.font = "24px VT323"; ctx.fillText("SYSTEM INTEGRITY LOST", 290, 200);
    if (Math.floor(Date.now()/500)%2===0) { ctx.fillStyle = "#fff"; ctx.font = "20px VT323"; ctx.fillText("> PRESS 'R' TO RETURN <", 290, 300); }
}

function drawWinScreen() {
    ctx.fillStyle = "rgba(0,0,0,0.9)"; ctx.fillRect(0, 0, 800, 400);
    ctx.fillStyle = "#00ff41"; ctx.font = "60px VT323"; ctx.fillText("THREAT DELETED", 230, 160);
    ctx.fillStyle = "#fff"; ctx.font = "20px VT323"; ctx.fillText("> PRESS 'SPACE' TO CONTINUE <", 270, 280);
}

function drawBar(x, y, v, m, c, l) {
    ctx.fillStyle = "#333"; ctx.fillRect(x, y, 200, 15);
    ctx.fillStyle = c; ctx.fillRect(x, y, Math.max(0, (v/m)*200), 15);
    ctx.fillStyle = "#fff"; ctx.font = "16px VT323"; ctx.fillText(l, x, y-5);
}

// Initialize sprite system
spriteSystem.init();

saveSystem.load();
runBootSequence(); 
animate();
</script>
</body>
</html>
