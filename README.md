<!DOCTYPE html>
<html lang="en" class="dark">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>Cyber Runner 2077 | Arcade Action Game</title>
  
  <!-- Tailwind CSS CDN -->
  <script src="https://cdn.tailwindcss.com"></script>
  <script>
    tailwind.config = {
      darkMode: 'class',
      theme: {
        extend: {
          colors: {
            neon: {
              cyan: '#06b6d4',
              pink: '#ec4899',
              yellow: '#eab308',
              purple: '#a855f7'
            }
          },
          fontFamily: {
            mono: ['Fira Code', 'Courier New', 'monospace'],
          }
        }
      }
    }
  </script>
  
  <!-- Lucide Icons -->
  <script src="https://unpkg.com/lucide@latest"></script>

  <style>
    @keyframes pulse-glow {
      0%, 100% { box-shadow: 0 0 15px rgba(6, 182, 212, 0.3); }
      50% { box-shadow: 0 0 30px rgba(236, 72, 153, 0.5); }
    }
    .glow-effect {
      animation: pulse-glow 3s infinite;
    }
    canvas {
      touch-action: none;
    }
  </style>
</head>
<body class="bg-slate-950 text-slate-100 min-h-screen font-sans antialiased flex flex-col items-center justify-between p-4 selection:bg-cyan-500 selection:text-slate-950">

  <!-- HEADER BAR -->
  <header class="w-full max-w-4xl flex items-center justify-between py-4 border-b border-slate-800">
    <div class="flex items-center gap-3">
      <div class="w-10 h-10 rounded-xl bg-gradient-to-tr from-cyan-500 to-pink-500 p-[2px] shadow-lg shadow-cyan-500/20">
        <div class="w-full h-full bg-slate-950 rounded-[10px] flex items-center justify-center">
          <i data-lucide="gamepad-2" class="w-5 h-5 text-cyan-400"></i>
        </div>
      </div>
      <div>
        <h1 class="text-xl font-black font-mono tracking-wider text-white">
          CYBER<span class="text-transparent bg-clip-text bg-gradient-to-r from-cyan-400 to-pink-500">RUNNER</span>
        </h1>
        <p class="text-[10px] text-slate-400 font-mono">Endless Arcade Action</p>
      </div>
    </div>

    <!-- HUD STATS -->
    <div class="flex items-center gap-6 font-mono text-xs">
      <div class="bg-slate-900 border border-slate-800 px-3 py-1.5 rounded-lg">
        <span class="text-slate-500 uppercase">High Score:</span>
        <span id="highScoreText" class="text-yellow-400 font-bold ml-1">0</span>
      </div>
      <div class="bg-slate-900 border border-slate-800 px-3 py-1.5 rounded-lg">
        <span class="text-slate-500 uppercase">Score:</span>
        <span id="scoreText" class="text-cyan-400 font-bold ml-1">0</span>
      </div>
    </div>
  </header>

  <!-- GAME CANVAS CONTAINER -->
  <main class="relative w-full max-w-4xl my-auto flex flex-col items-center">
    
    <div class="relative w-full aspect-[16/9] max-h-[500px] rounded-2xl overflow-hidden border border-slate-800 shadow-2xl bg-slate-950 glow-effect flex items-center justify-center">
      
      <!-- HTML5 CANVAS -->
      <canvas id="gameCanvas" class="w-full h-full block"></canvas>

      <!-- START / GAME OVER OVERLAY -->
      <div id="uiOverlay" class="absolute inset-0 bg-slate-950/90 backdrop-blur-md flex flex-col items-center justify-center p-6 text-center z-20 transition-all duration-300">
        <div class="space-y-4 max-w-md">
          <div class="inline-flex p-3 rounded-2xl bg-slate-900 border border-slate-800 text-cyan-400">
            <i data-lucide="zap" class="w-8 h-8 text-yellow-400"></i>
          </div>
          
          <h2 id="overlayTitle" class="text-3xl font-black tracking-tight text-white font-mono">
            READY TO RUN?
          </h2>
          
          <p id="overlayMessage" class="text-xs text-slate-400 font-sans">
            Jump over corrupted firewalls, slide under malware lasers, and collect neon data cubes!
          </p>

          <!-- FINAL SCORE STATS (HIDDEN INITIALLY) -->
          <div id="finalStats" class="hidden grid-cols-2 gap-3 bg-slate-900/80 border border-slate-800 p-3 rounded-xl font-mono text-xs text-left">
            <div>
              <span class="text-slate-500 block text-[10px]">DISTANCE RUN</span>
              <span id="finalScoreVal" class="text-cyan-400 font-bold text-base">0 m</span>
            </div>
            <div>
              <span class="text-slate-500 block text-[10px]">CUBES COLLECTED</span>
              <span id="finalCubesVal" class="text-pink-400 font-bold text-base">0</span>
            </div>
          </div>

          <button id="startBtn" onclick="startGame()" class="w-full py-4 rounded-xl bg-gradient-to-r from-cyan-400 via-blue-500 to-pink-500 text-slate-950 font-black text-sm font-mono uppercase tracking-wider hover:opacity-95 transition-all shadow-lg shadow-cyan-500/20 cursor-pointer">
            Start Neural Run
          </button>
        </div>
      </div>

    </div>

    <!-- TOUCH / ON-SCREEN CONTROLS FOR MOBILE -->
    <div class="w-full grid grid-cols-2 gap-4 mt-4 max-w-4xl sm:hidden font-mono text-xs">
      <button onpointerdown="triggerJump()" class="py-4 bg-slate-900 border border-slate-800 active:bg-cyan-950 rounded-xl text-cyan-400 font-bold flex items-center justify-center gap-2">
        <i data-lucide="arrow-up" class="w-4 h-4"></i> JUMP (UP / SPACE)
      </button>
      <button onpointerdown="triggerDuck()" onpointerup="releaseDuck()" class="py-4 bg-slate-900 border border-slate-800 active:bg-pink-950 rounded-xl text-pink-400 font-bold flex items-center justify-center gap-2">
        <i data-lucide="arrow-down" class="w-4 h-4"></i> SLIDE (DOWN)
      </button>
    </div>

    <!-- CONTROLS HINT -->
    <div class="hidden sm:flex items-center justify-center gap-8 mt-4 text-xs font-mono text-slate-500">
      <span class="flex items-center gap-1.5">
        <kbd class="px-2 py-1 bg-slate-900 border border-slate-800 rounded text-slate-300">Space</kbd> or <kbd class="px-2 py-1 bg-slate-900 border border-slate-800 rounded text-slate-300">W</kbd> / <kbd class="px-2 py-1 bg-slate-900 border border-slate-800 rounded text-slate-300">↑</kbd> to Jump
      </span>
      <span class="flex items-center gap-1.5">
        <kbd class="px-2 py-1 bg-slate-900 border border-slate-800 rounded text-slate-300">S</kbd> / <kbd class="px-2 py-1 bg-slate-900 border border-slate-800 rounded text-slate-300">↓</kbd> to Duck/Slide
      </span>
    </div>

  </main>

  <!-- FOOTER -->
  <footer class="w-full max-w-4xl py-4 border-t border-slate-800 text-center text-xs font-mono text-slate-600 flex items-center justify-between">
    <span>CYBER RUNNER 2077 v1.0</span>
    <span>Built with HTML5 Canvas & Web Audio API</span>
  </footer>

  <!-- GAME ENGINE SCRIPT -->
  <script>
    lucide.createIcons();

    // CANVAS SETUP
    const canvas = document.getElementById('gameCanvas');
    const ctx = canvas.getContext('2d');
    
    // Internal canvas resolution
    canvas.width = 960;
    canvas.height = 540;

    // WEB AUDIO API SYNTHESIZER
    let audioCtx = null;

    function initAudio() {
      if (!audioCtx) {
        audioCtx = new (window.AudioContext || window.webkitAudioContext)();
      }
    }

    function playSound(type) {
      if (!audioCtx) return;
      try {
        const osc = audioCtx.createOscillator();
        const gain = audioCtx.createGain();
        osc.connect(gain);
        gain.connect(audioCtx.destination);

        const now = audioCtx.currentTime;

        if (type === 'jump') {
          osc.type = 'sine';
          osc.frequency.setValueAtTime(180, now);
          osc.frequency.exponentialRampToValueAtTime(450, now + 0.15);
          gain.gain.setValueAtTime(0.15, now);
          gain.gain.exponentialRampToValueAtTime(0.01, now + 0.15);
          osc.start(now);
          osc.stop(now + 0.15);
        } else if (type === 'coin') {
          osc.type = 'triangle';
          osc.frequency.setValueAtTime(523.25, now); // C5
          osc.frequency.setValueAtTime(659.25, now + 0.08); // E5
          gain.gain.setValueAtTime(0.2, now);
          gain.gain.exponentialRampToValueAtTime(0.01, now + 0.2);
          osc.start(now);
          osc.stop(now + 0.2);
        } else if (type === 'hit') {
          osc.type = 'sawtooth';
          osc.frequency.setValueAtTime(120, now);
          osc.frequency.exponentialRampToValueAtTime(30, now + 0.3);
          gain.gain.setValueAtTime(0.3, now);
          gain.gain.exponentialRampToValueAtTime(0.01, now + 0.3);
          osc.start(now);
          osc.stop(now + 0.3);
        }
      } catch (e) {
        // Audio fallback
      }
    }

    // GAME STATE VARIABLES
    let isGameRunning = false;
    let score = 0;
    let cubesCollected = 0;
    let highScore = localStorage.getItem('cyber_runner_highscore') || 0;
    document.getElementById('highScoreText').innerText = highScore;

    let gameSpeed = 6;
    let frameCount = 0;

    // GROUND LEVEL
    const groundY = 440;

    // PLAYER OBJECT
    const player = {
      x: 100,
      y: groundY - 60,
      width: 36,
      height: 60,
      vy: 0,
      gravity: 0.8,
      jumpForce: -15,
      isGrounded: false,
      isDucking: false,
      normalHeight: 60,
      duckHeight: 30,
      color: '#06b6d4',
      trail: []
    };

    // GAME ARRAYS
    let obstacles = [];
    let collectibles = [];
    let particles = [];
    let backgroundBuildings = [];

    // INITIALIZE BACKGROUND BUILDINGS
    for (let i = 0; i < 15; i++) {
      backgroundBuildings.push({
        x: i * 80,
        width: 60 + Math.random() * 30,
        height: 120 + Math.random() * 200,
        color: Math.random() > 0.5 ? '#0f172a' : '#1e1b4b'
      });
    }

    // INPUT LISTENERS
    const keys = {};

    window.addEventListener('keydown', (e) => {
      if (['Space', 'KeyW', 'ArrowUp', 'KeyS', 'ArrowDown'].includes(e.code)) {
        e.preventDefault();
      }
      keys[e.code] = true;

      if ((e.code === 'Space' || e.code === 'KeyW' || e.code === 'ArrowUp')) {
        triggerJump();
      }
      if (e.code === 'KeyS' || e.code === 'ArrowDown') {
        triggerDuck();
      }
    });

    window.addEventListener('keyup', (e) => {
      keys[e.code] = false;
      if (e.code === 'KeyS' || e.code === 'ArrowDown') {
        releaseDuck();
      }
    });

    function triggerJump() {
      if (!isGameRunning) return;
      initAudio();
      if (player.isGrounded) {
        player.vy = player.jumpForce;
        player.isGrounded = false;
        playSound('jump');
        createParticles(player.x + player.width / 2, player.y + player.height, '#06b6d4', 8);
      }
    }

    function triggerDuck() {
      if (!isGameRunning) return;
      if (player.isGrounded) {
        player.isDucking = true;
      }
    }

    function releaseDuck() {
      player.isDucking = false;
    }

    // PARTICLE SYSTEM
    function createParticles(x, y, color, count = 5) {
      for (let i = 0; i < count; i++) {
        particles.push({
          x: x,
          y: y,
          vx: (Math.random() - 0.5) * 4,
          vy: (Math.random() - 0.5) * 4,
          size: Math.random() * 4 + 2,
          color: color,
          life: 1.0
        });
      }
    }

    // SPAWN OBSTACLES & COLLECTIBLES
    function spawnEntities() {
      // Spawn Obstacles
      if (frameCount % Math.max(40, Math.floor(100 - gameSpeed * 3)) === 0) {
        const type = Math.random() > 0.4 ? 'GROUND' : 'LASER';
        if (type === 'GROUND') {
          obstacles.push({
            x: canvas.width + 50,
            y: groundY - 50,
            width: 30,
            height: 50,
            type: 'GROUND',
            color: '#ec4899'
          });
        } else {
          // Overhead laser obstacle (requires ducking)
          obstacles.push({
            x: canvas.width + 50,
            y: groundY - 80,
            width: 45,
            height: 25,
            type: 'LASER',
            color: '#eab308'
          });
        }
      }

      // Spawn Data Cubes
      if (frameCount % 130 === 0) {
        collectibles.push({
          x: canvas.width + 50,
          y: groundY - (Math.random() > 0.5 ? 90 : 40),
          size: 16,
          rotation: 0,
          collected: false
        });
      }
    }

    // MAIN GAME LOOP
    function update() {
      if (!isGameRunning) return;

      frameCount++;
      gameSpeed += 0.001; // Gradually speed up

      // Update Player Height (Ducking)
      if (player.isDucking && player.isGrounded) {
        player.height = player.duckHeight;
        player.y = groundY - player.duckHeight;
      } else {
        player.height = player.normalHeight;
      }

      // Apply Physics
      player.vy += player.gravity;
      player.y += player.vy;

      // Ground Collision
      if (player.y + player.height >= groundY) {
        player.y = groundY - player.height;
        player.vy = 0;
        player.isGrounded = true;
      }

      // Player Motion Trail
      if (frameCount % 3 === 0) {
        player.trail.push({ x: player.x, y: player.y, width: player.width, height: player.height, alpha: 0.5 });
        if (player.trail.length > 5) player.trail.shift();
      }

      // Move Background
      backgroundBuildings.forEach(b => {
        b.x -= gameSpeed * 0.2;
        if (b.x + b.width < 0) {
          b.x = canvas.width + Math.random() * 40;
        }
      });

      // Update Obstacles
      for (let i = obstacles.length - 1; i >= 0; i--) {
        const obs = obstacles[i];
        obs.x -= gameSpeed;

        // Collision Check
        if (
          player.x < obs.x + obs.width &&
          player.x + player.width > obs.x &&
          player.y < obs.y + obs.height &&
          player.y + player.height > obs.y
        ) {
          gameOver();
          return;
        }

        if (obs.x + obs.width < 0) {
          obstacles.splice(i, 1);
          score += 10;
          document.getElementById('scoreText').innerText = score;
        }
      }

      // Update Collectibles
      for (let i = collectibles.length - 1; i >= 0; i--) {
        const c = collectibles[i];
        c.x -= gameSpeed;
        c.rotation += 0.05;

        // Collect Check
        if (
          !c.collected &&
          player.x < c.x + c.size &&
          player.x + player.width > c.x &&
          player.y < c.y + c.size &&
          player.y + player.height > c.y
        ) {
          c.collected = true;
          cubesCollected++;
          score += 50;
          playSound('coin');
          createParticles(c.x, c.y, '#eab308', 12);
          document.getElementById('scoreText').innerText = score;
          collectibles.splice(i, 1);
        } else if (c.x + c.size < 0) {
          collectibles.splice(i, 1
