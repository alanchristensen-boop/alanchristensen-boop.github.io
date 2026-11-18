<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>Vanguard Arena — 20-Player Woodland Battle Royale</title>
    <!-- Tailwind CSS for Tactical HUD & Menus -->
    <script src="https://cdn.tailwindcss.com"></script>
    <!-- Three.js for 3D Rendering -->
    <script src="https://cdnjs.cloudflare.com/ajax/libs/three.js/r128/three.min.js"></script>
    <!-- Tone.js for Procedural Weapon, Loot, Heal & Footstep Audio -->
    <script src="https://cdnjs.cloudflare.com/ajax/libs/tone/14.8.49/Tone.js"></script>
    <!-- PeerJS for Optional Cross-Device WebRTC Multiplayer -->
    <script src="https://unpkg.com/peerjs@1.5.2/dist/peerjs.min.js"></script>
    <link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;600;700;800;900&family=JetBrains+Mono:wght@500;700&display=swap" rel="stylesheet">
    <style>
        * { box-sizing: border-box; user-select: none; -webkit-user-select: none; }
        html, body {
            margin: 0;
            padding: 0;
            width: 100%;
            height: 100%;
            overflow: hidden;
            background-color: #0c140e;
            font-family: 'Inter', sans-serif;
            color: #f4f4f5;
        }
        #game-canvas-container {
            position: fixed;
            inset: 0;
            width: 100%;
            height: 100%;
            z-index: 1;
        }
        canvas {
            display: block;
            width: 100%;
            height: 100%;
            cursor: crosshair;
        }
        .mono { font-family: 'JetBrains Mono', monospace; }
        .glass-panel {
            background: rgba(15, 23, 18, 0.84);
            backdrop-filter: blur(10px);
            -webkit-backdrop-filter: blur(10px);
            border: 1px solid rgba(163, 230, 53, 0.25);
        }
        @keyframes pulseFast {
            0%, 100% { opacity: 1; transform: scale(1); }
            50% { opacity: 0.65; transform: scale(0.97); }
        }
        .animate-pulse-fast { animation: pulseFast 1.2s infinite ease-in-out; }
    </style>
</head>
<body>

    <!-- 3D WebGL Viewport -->
    <div id="game-canvas-container"></div>

    <!-- Damage Vignette Flash -->
    <div id="damage-flash" class="pointer-events-none fixed inset-0 z-10 opacity-0 transition-opacity duration-150"
         style="background: radial-gradient(circle, rgba(220,38,38,0) 45%, rgba(220,38,38,0.65) 100%);"></div>

    <!-- Center Crosshair & Hitmarker -->
    <div id="crosshair-container" class="pointer-events-none fixed inset-0 z-20 flex items-center justify-center">
        <div class="relative w-8 h-8 flex items-center justify-center">
            <div id="ch-top" class="absolute top-0 w-0.5 h-2 bg-lime-400 shadow"></div>
            <div id="ch-bottom" class="absolute bottom-0 w-0.5 h-2 bg-lime-400 shadow"></div>
            <div id="ch-left" class="absolute left-0 h-0.5 w-2 bg-lime-400 shadow"></div>
            <div id="ch-right" class="absolute right-0 h-0.5 w-2 bg-lime-400 shadow"></div>
            <div class="w-1 h-1 rounded-full bg-white"></div>
            <!-- Hitmarker X -->
            <div id="hitmarker" class="absolute inset-0 opacity-0 transition-opacity duration-75 flex items-center justify-center">
                <div class="w-5 h-0.5 bg-red-500 rotate-45 absolute"></div>
                <div class="w-5 h-0.5 bg-red-500 -rotate-45 absolute"></div>
            </div>
        </div>
    </div>

    <!-- Chest Loot / Interaction Prompt -->
    <div id="loot-prompt" class="pointer-events-none fixed bottom-36 left-1/2 -translate-x-1/2 z-20 hidden">
        <div class="glass-panel px-4 py-2 rounded-xl flex items-center gap-2.5 shadow-xl border border-amber-400/50">
            <span class="px-2 py-0.5 rounded bg-amber-400 text-stone-950 font-black text-xs mono">E</span>
            <span id="loot-prompt-text" class="text-xs font-bold tracking-wide text-amber-200">OPEN SUPPLY CHEST</span>
        </div>
    </div>

    <!-- Medkit 3.0s Progress Bar -->
    <div id="medkit-progress-wrap" class="pointer-events-none fixed bottom-44 left-1/2 -translate-x-1/2 z-20 hidden w-64">
        <div class="glass-panel p-3 rounded-xl border border-red-400/50 shadow-xl">
            <div class="flex justify-between items-center text-xs font-extrabold text-red-300 mb-1.5">
                <span>APPLYING MEDKIT...</span>
                <span id="medkit-timer-text" class="mono">3.0s</span>
            </div>
            <div class="w-full h-2.5 bg-stone-800 rounded-full overflow-hidden">
                <div id="medkit-progress-bar" class="h-full bg-gradient-to-r from-red-500 to-emerald-400 w-0 transition-all duration-75"></div>
            </div>
        </div>
    </div>

    <!-- Top-Left Match Status, Aim Mode & Auto-Sim Controls -->
    <div class="fixed top-3 left-3 z-20 flex flex-col gap-2 max-w-xs sm:max-w-sm">
        <div class="glass-panel p-3 rounded-2xl shadow-lg">
            <div class="flex items-center justify-between gap-3">
                <div class="flex items-center gap-2">
                    <span class="w-2.5 h-2.5 rounded-full bg-lime-400 animate-pulse"></span>
                    <span id="hud-mode-badge" class="text-xs font-extrabold uppercase tracking-wider text-lime-300">Woodland BR (20 Alive)</span>
                </div>
                <span id="hud-lobby-tag" class="mono text-[11px] text-stone-400">SOLO BR</span>
            </div>
            <div class="mt-1.5 flex items-center justify-between text-[11px]">
                <span id="hud-aim-status" class="text-amber-300 font-semibold">Click screen to lock cursor (or move mouse / drag)</span>
            </div>
            <div class="mt-2 pt-2 border-t border-stone-700/70 flex flex-wrap items-center justify-between gap-1.5">
                <button id="btn-sim-toggle" onclick="toggleSimMode()"
                        class="px-2.5 py-1 rounded-lg bg-stone-800 hover:bg-stone-700 border border-stone-600 text-[11px] font-bold text-emerald-300 transition">
                    [M] Mode: Manual Play
                </button>
                <div class="flex items-center gap-1">
                    <button onclick="setSimSpeed(1)" id="spd-1" class="px-2 py-0.5 rounded bg-lime-600 text-stone-950 font-bold text-[10px]">1x</button>
                    <button onclick="setSimSpeed(2)" id="spd-2" class="px-2 py-0.5 rounded bg-stone-800 text-stone-300 font-bold text-[10px]">2x</button>
                    <button onclick="setSimSpeed(4)" id="spd-4" class="px-2 py-0.5 rounded bg-stone-800 text-stone-300 font-bold text-[10px]">4x</button>
                    <button onclick="openPauseMenu()" class="ml-1 px-2 py-0.5 rounded bg-stone-800 hover:bg-stone-700 text-stone-200 font-bold text-[10px]">[P] Menu</button>
                </div>
            </div>
        </div>

        <!-- Live Kill Feed -->
        <div id="kill-feed" class="flex flex-col gap-1 pointer-events-none"></div>
    </div>

    <!-- Top-Right Tactical Radar Minimap -->
    <div class="fixed top-3 right-3 z-20 flex flex-col items-end gap-2">
        <div class="glass-panel p-2 rounded-2xl shadow-lg">
            <canvas id="minimap-canvas" width="148" height="148" class="rounded-xl bg-stone-950/90 border border-stone-700/80"></canvas>
            <div class="mt-1.5 flex items-center justify-between text-[10px] font-semibold text-stone-300 px-1">
                <span class="flex items-center gap-1"><span class="w-2 h-2 rounded-full bg-lime-400 inline-block"></span>You</span>
                <span class="flex items-center gap-1"><span class="w-2 h-2 rounded-full bg-red-500 inline-block"></span>Bots</span>
                <span class="flex items-center gap-1"><span class="w-2 h-2 rounded-full bg-amber-400 inline-block"></span>Loot</span>
            </div>
        </div>
    </div>

    <!-- Bottom-Left Health & Eliminations HUD -->
    <div class="fixed bottom-4 left-4 z-20 pointer-events-none">
        <div class="glass-panel p-3.5 rounded-2xl w-64 sm:w-72 shadow-xl">
            <div class="flex justify-between items-baseline mb-1.5">
                <div class="flex items-baseline gap-1.5">
                    <span id="hud-health-val" class="text-3xl font-black text-lime-400 mono">200</span>
                    <span class="text-xs font-bold text-stone-400">/ 200 HP</span>
                </div>
                <div class="text-right">
                    <span class="text-[10px] uppercase tracking-wider text-stone-400 block">Eliminations</span>
                    <span id="hud-kills-val" class="text-base font-black text-amber-400 mono">0</span>
                </div>
            </div>
            <div class="w-full h-3 bg-stone-900 rounded-full overflow-hidden border border-stone-700/70">
                <div id="hud-health-bar" class="h-full bg-gradient-to-r from-emerald-500 to-lime-400 transition-all duration-150" style="width: 100%;"></div>
            </div>
            <div class="mt-2 flex justify-between text-[10px] text-stone-400 font-medium">
                <span>WASD Move • Space Jump</span>
                <span>[H] Quick Medkit</span>
            </div>
        </div>
    </div>

    <!-- Bottom-Right Weapon Loadout & Ammo HUD -->
    <div class="fixed bottom-4 right-4 z-20 flex flex-col items-end gap-2 pointer-events-none">
        <div class="glass-panel px-4 py-2.5 rounded-2xl text-right shadow-xl">
            <div id="hud-weapon-name" class="text-xs font-extrabold uppercase tracking-wider text-lime-300">M9 TACTICAL PISTOL</div>
            <div class="flex items-baseline justify-end gap-1.5 mt-0.5">
                <span id="hud-ammo-mag" class="text-3xl font-black text-white mono">15</span>
                <span id="hud-ammo-res" class="text-sm font-bold text-stone-400 mono">/ 90</span>
            </div>
        </div>
        <!-- 5-Slot Inventory Bar -->
        <div class="flex items-center gap-1.5">
            <div id="slot-0" class="glass-panel px-2.5 py-1.5 rounded-xl border border-stone-700 opacity-45 text-center min-w-[54px]">
                <div class="text-[10px] font-bold text-stone-400 mono">[1]</div>
                <div class="text-[11px] font-extrabold text-stone-200">M4A1</div>
            </div>
            <div id="slot-1" class="glass-panel px-2.5 py-1.5 rounded-xl border border-stone-700 opacity-45 text-center min-w-[54px]">
                <div class="text-[10px] font-bold text-stone-400 mono">[2]</div>
                <div class="text-[11px] font-extrabold text-stone-200">M870</div>
            </div>
            <div id="slot-2" class="glass-panel px-2.5 py-1.5 rounded-xl border-2 border-lime-400 text-center min-w-[54px]">
                <div class="text-[10px] font-bold text-lime-300 mono">[3]</div>
                <div class="text-[11px] font-extrabold text-white">M9</div>
            </div>
            <div id="slot-3" class="glass-panel px-2.5 py-1.5 rounded-xl border border-stone-700 text-center min-w-[54px]">
                <div class="text-[10px] font-bold text-stone-400 mono">[4]</div>
                <div class="text-[11px] font-extrabold text-stone-200">KNIFE</div>
            </div>
            <div id="slot-4" class="glass-panel px-2.5 py-1.5 rounded-xl border border-stone-700 opacity-45 text-center min-w-[58px]">
                <div class="text-[10px] font-bold text-red-400 mono">[5]</div>
                <div id="slot-4-label" class="text-[11px] font-extrabold text-red-200">MED (0)</div>
            </div>
        </div>
    </div>

    <!-- Victory / Elimination End-of-Match Overlay -->
    <div id="match-end-overlay" class="fixed inset-0 z-40 hidden items-center justify-center bg-black/75 backdrop-blur-sm p-4">
        <div class="glass-panel max-w-md w-full p-6 rounded-3xl text-center border border-lime-400/40 shadow-2xl">
            <div id="end-banner-tag" class="inline-block px-3 py-1 rounded-full bg-lime-500/20 text-lime-300 text-xs font-extrabold uppercase tracking-widest mb-2">
                MATCH COMPLETE
            </div>
            <h2 id="end-banner-title" class="text-3xl font-black text-white tracking-tight">VICTORY ROYALE!</h2>
            <p id="end-banner-sub" class="text-sm text-stone-300 mt-2">You outlasted all 19 enemy soldiers in the woodland arena.</p>
            <div class="mt-5 grid grid-cols-2 gap-3 bg-stone-900/80 p-3.5 rounded-2xl border border-stone-700/70">
                <div>
                    <div class="text-[11px] text-stone-400 uppercase font-bold">Your Eliminations</div>
                    <div id="end-stat-kills" class="text-2xl font-black text-amber-400 mono mt-0.5">0</div>
                </div>
                <div>
                    <div class="text-[11px] text-stone-400 uppercase font-bold">Final Placement</div>
                    <div id="end-stat-place" class="text-2xl font-black text-lime-400 mono mt-0.5">#1 / 20</div>
                </div>
            </div>
            <button onclick="resetSoloMatch()"
                    class="mt-6 w-full py-3.5 rounded-2xl bg-gradient-to-r from-lime-500 to-emerald-500 hover:from-lime-400 hover:to-emerald-400 text-stone-950 font-black text-sm uppercase tracking-wider shadow-lg transition">
                Play Again (New 20-Player Match)
            </button>
        </div>
    </div>

    <!-- Pause / Multiplayer & Leaderboard Overlay ([P]) -->
    <div id="pause-menu-overlay" class="fixed inset-0 z-30 hidden items-center justify-center bg-black/70 backdrop-blur-md p-4">
        <div class="glass-panel max-w-2xl w-full p-6 rounded-3xl shadow-2xl border border-lime-400/30 max-h-[90vh] overflow-y-auto">
            <div class="flex items-center justify-between border-b border-stone-700/80 pb-4">
                <div>
                    <h1 class="text-2xl font-black tracking-tight text-white">VANGUARD ARENA</h1>
                    <p class="text-xs text-stone-400 mt-0.5">20-Player Woodland Battle Royale • 200 HP • 25 Head / 15 Body</p>
                </div>
                <button onclick="resumeGameplay()" class="px-4 py-2 rounded-xl bg-lime-500 hover:bg-lime-400 text-stone-950 font-black text-xs uppercase tracking-wider">
                    Resume Match
                </button>
            </div>

            <div class="grid grid-cols-1 md:grid-cols-2 gap-5 mt-5">
                <!-- Left Column: Controls & Solo Reset -->
                <div class="bg-stone-900/75 p-4 rounded-2xl border border-stone-700/70 flex flex-col justify-between">
                    <div>
                        <h3 class="text-xs font-extrabold uppercase tracking-wider text-lime-300 mb-2.5">Field Controls</h3>
                        <ul class="text-xs text-stone-300 space-y-1.5">
                            <li><span class="mono text-lime-300 font-bold">W A S D</span> — Tactical Movement (<span class="mono">Shift</span> Sprint)</li>
                            <li><span class="mono text-lime-300 font-bold">Mouse / Arrows</span> — Aim Camera (Click to lock)</li>
                            <li><span class="mono text-lime-300 font-bold">Left Click / F</span> — Fire Weapon / Use Medkit</li>
                            <li><span class="mono text-lime-300 font-bold">E</span> — Open Supply Chests (M4A1, Shotgun, Medkits)</li>
                            <li><span class="mono text-lime-300 font-bold">1–5 / H</span> — Switch Gear / 3.0s Fortnite Medkit</li>
                            <li><span class="mono text-lime-300 font-bold">M</span> — Toggle Live Auto-Bot Simulation</li>
                        </ul>
                    </div>
                    <button onclick="resetSoloMatch()" class="mt-4 w-full py-2.5 rounded-xl bg-emerald-600 hover:bg-emerald-500 text-white font-extrabold text-xs uppercase tracking-wider transition">
                        Restart Fresh 20-Bot Match
                    </button>
                </div>

                <!-- Right Column: Live Match Standings -->
                <div class="bg-stone-900/75 p-4 rounded-2xl border border-stone-700/70">
                    <div class="flex items-center justify-between mb-2">
                        <h3 class="text-xs font-extrabold uppercase tracking-wider text-amber-300">Combatant Roster (20)</h3>
                        <span id="menu-alive-count" class="mono text-[11px] text-lime-400 font-bold">20 ALIVE</span>
                    </div>
                    <div id="roster-list" class="max-h-52 overflow-y-auto space-y-1 pr-1 text-xs"></div>
                </div>
            </div>
        </div>
    </div>

    <script>
        // =========================================================================
        // 1. GAME CONSTANTS & COMBAT CONFIGURATION
        // =========================================================================
        const ARENA_HALF_SIZE = 195; // 390 x 390 woodland map (3x original size)
        const PLAYER_MAX_HEALTH = 200;
        const BOT_MAX_HEALTH = 200;
        const HEADSHOT_DAMAGE = 25;
        const BODYSHOT_DAMAGE = 15;
        const BOT_COUNT = 19; // 19 bots + 1 player = 20 combatants (no respawns)
        const MEDKIT_DURATION = 3.0; // 3.0s channeling timer

        const BOT_CALLSIGNS = [
            'Viper-01', 'Ghost-02', 'Reaper-03', 'Nomad-04', 'Kestrel-05',
            'Talon-06', 'Shadow-07', 'Cobra-08', 'Falcon-09', 'Wraith-10',
            'Ranger-11', 'Striker-12', 'Havoc-13', 'Blaze-14', 'Atlas-15',
            'Valkyrie-16', 'Titan-17', 'Lynx-18', 'Specter-19'
        ];

        const WEAPON_DEFS = [
            { id: 'm4a1', name: 'M4A1 CARBINE', magSize: 30, reserve: 120, fireRate: 0.11, auto: true, pellets: 1, spread: 0.015, range: 160, unlocked: false },
            { id: 'm870', name: 'M870 SHOTGUN', magSize: 6, reserve: 36, fireRate: 0.72, auto: false, pellets: 5, spread: 0.065, range: 45, unlocked: false },
            { id: 'm9',   name: 'M9 9MM PISTOL', magSize: 15, reserve: 90, fireRate: 0.22, auto: false, pellets: 1, spread: 0.02, range: 110, unlocked: true },
            { id: 'knife',name: 'COMBAT KNIFE', magSize: 1, reserve: 0, fireRate: 0.42, auto: false, pellets: 1, spread: 0.0, range: 4.2, unlocked: true },
            { id: 'medkit',name: 'FORTNITE MEDKIT', magSize: 1, reserve: 0, fireRate: 0.5, auto: false, pellets: 0, spread: 0, range: 0, unlocked: false }
        ];

        // =========================================================================
        // 2. CORE ENGINE STATE
        // =========================================================================
        let scene, camera, renderer, clock;
        let gameActive = true;
        let isMenuOpen = false;
        let simModeEnabled = false;
        let simSpeedMultiplier = 1;

        // Player state
        let playerHealth = PLAYER_MAX_HEALTH;
        let playerKills = 0;
        let playerDead = false;
        let playerPos = new THREE.Vector3(0, 1.75, 0);
        let playerVel = new THREE.Vector3();
        let playerYaw = 0;
        let playerPitch = 0;
        let onGround = true;

        // Inventory & Medkit state
        let weapons = [];
        let currentWeaponIdx = 2; // Spawn with M9 Pistol equipped
        let medkitCount = 0;
        let isApplyingMedkit = false;
        let medkitTimer = 0;
        let isReloading = false;
        let lastFireTime = 0;
        let isFiring = false;
        let recoilOffset = 0;

        // Input & Pointer Lock / Sandbox Free-Aim Fallback
        let lastMouseX = null;
        let lastMouseY = null;
        let isRightMouseDown = false;
        let pointerLockUnsupported = false;
        const keys = {};

        // World Entities
        const obstacles = []; // { minX, maxX, minZ, maxZ, height }
        const bots = [];
        const chests = [];
        const tracers = [];
        let viewModelGroup = null;
        let viewModels = {};
        let audioReady = false;

        // =========================================================================
        // 3. AUDIO SYNTHESIS (TONE.JS)
        // =========================================================================
        let gunNoise, hitSynth, lootSynth, healSynth;
        function initAudio() {
            if (audioReady || typeof Tone === 'undefined') return;
            try {
                Tone.start();
                gunNoise = new Tone.NoiseSynth({
                    noise: { type: 'white' },
                    envelope: { attack: 0.002, decay: 0.09, sustain: 0 }
                }).toDestination();
                gunNoise.volume.value = -10;

                hitSynth = new Tone.Synth({
                    oscillator: { type: 'triangle' },
                    envelope: { attack: 0.001, decay: 0.06, sustain: 0 }
                }).toDestination();
                hitSynth.volume.value = -8;

                lootSynth = new Tone.PolySynth(Tone.Synth).toDestination();
                lootSynth.volume.value = -10;

                healSynth = new Tone.Synth({
                    oscillator: { type: 'sine' },
                    envelope: { attack: 0.05, decay: 0.35, sustain: 0.1, release: 0.3 }
                }).toDestination();
                healSynth.volume.value = -8;
                audioReady = true;
            } catch (e) {}
        }

        function playShotSound(weaponId) {
            if (!audioReady || !gunNoise) return;
            try {
                gunNoise.envelope.decay = weaponId === 'm870' ? 0.18 : (weaponId === 'knife' ? 0.04 : 0.08);
                gunNoise.triggerAttackRelease('16n');
            } catch (e) {}
        }

        function playHitSound(isHeadshot) {
            if (!audioReady || !hitSynth) return;
            try {
                hitSynth.triggerAttackRelease(isHeadshot ? 'C6' : 'G5', '32n');
            } catch (e) {}
        }

        function playLootSound() {
            if (!audioReady || !lootSynth) return;
            try {
                lootSynth.triggerAttackRelease(['E5', 'G5', 'B5'], '16n');
            } catch (e) {}
        }

        function playHealCompleteSound() {
            if (!audioReady || !healSynth) return;
            try {
                healSynth.triggerAttackRelease('A5', '8n');
            } catch (e) {}
        }

        // =========================================================================
        // 4. THREE.JS SCENE & 390x390 WOODLAND BATTLE ROYALE MAP
        // =========================================================================
        function initThree() {
            const container = document.getElementById('game-canvas-container');
            scene = new THREE.Scene();
            scene.background = new THREE.Color(0x8bb8d8);
            scene.fog = new THREE.FogExp2(0x8bb8d8, 0.0042);

            camera = new THREE.PerspectiveCamera(75, window.innerWidth / window.innerHeight, 0.1, 460);
            camera.rotation.order = 'YXZ';

            renderer = new THREE.WebGLRenderer({ antialias: true, powerPreference: 'high-performance' });
            renderer.setSize(window.innerWidth, window.innerHeight);
            renderer.setPixelRatio(Math.min(window.devicePixelRatio, 2));
            renderer.shadowMap.enabled = true;
            renderer.shadowMap.type = THREE.PCFSoftShadowMap;
            container.innerHTML = '';
            container.appendChild(renderer.domElement);

            // Lighting
            const hemiLight = new THREE.HemisphereLight(0xdff2ff, 0x2d4a22, 0.85);
            scene.add(hemiLight);

            const sunLight = new THREE.DirectionalLight(0xfff5db, 1.15);
            sunLight.position.set(110, 180, 90);
            sunLight.castShadow = true;
            sunLight.shadow.mapSize.width = 2048;
            sunLight.shadow.mapSize.height = 2048;
            sunLight.shadow.camera.near = 10;
            sunLight.shadow.camera.far = 480;
            const d = 210;
            sunLight.shadow.camera.left = -d;
            sunLight.shadow.camera.right = d;
            sunLight.shadow.camera.top = d;
            sunLight.shadow.camera.bottom = -d;
            scene.add(sunLight);

            scene.add(camera);
            buildFirstPersonViewModels();
            buildWoodlandArena();
        }

        function addBoxObstacle(x, y, z, w, h, d, color, castShadow = true) {
            const geo = new THREE.BoxGeometry(w, h, d);
            const mat = new THREE.MeshStandardMaterial({ color, roughness: 0.85, metalness: 0.05 });
            const mesh = new THREE.Mesh(geo, mat);
            mesh.position.set(x, y, z);
            mesh.castShadow = castShadow;
            mesh.receiveShadow = true;
            scene.add(mesh);

            obstacles.push({
                minX: x - w / 2,
                maxX: x + w / 2,
                minZ: z - d / 2,
                maxZ: z + d / 2,
                height: y + h / 2
            });
            return mesh;
        }

        function buildWoodlandArena() {
            obstacles.length = 0;

            // Procedural mossy grass ground texture
            const groundCanvas = document.createElement('canvas');
            groundCanvas.width = 256;
            groundCanvas.height = 256;
            const gctx = groundCanvas.getContext('2d');
            gctx.fillStyle = '#365928';
            gctx.fillRect(0, 0, 256, 256);
            for (let i = 0; i < 1800; i++) {
                gctx.fillStyle = i % 2 === 0 ? '#2c4a20' : '#446e33';
                gctx.fillRect(Math.random() * 256, Math.random() * 256, 3, 3);
            }
            const groundTex = new THREE.CanvasTexture(groundCanvas);
            groundTex.wrapS = THREE.RepeatWrapping;
            groundTex.wrapT = THREE.RepeatWrapping;
            groundTex.repeat.set(60, 60);

            const ground = new THREE.Mesh(
                new THREE.PlaneGeometry(ARENA_HALF_SIZE * 2 + 20, ARENA_HALF_SIZE * 2 + 20),
                new THREE.MeshStandardMaterial({ map: groundTex, roughness: 0.95 })
            );
            ground.rotation.x = -Math.PI / 2;
            ground.receiveShadow = true;
            scene.add(ground);

            // Perimeter Fortress Walls
            const wallH = 12;
            const span = ARENA_HALF_SIZE * 2;
            addBoxObstacle(0, wallH / 2, -ARENA_HALF_SIZE, span, wallH, 4, 0x4b5563);
            addBoxObstacle(0, wallH / 2, ARENA_HALF_SIZE, span, wallH, 4, 0x4b5563);
            addBoxObstacle(-ARENA_HALF_SIZE, wallH / 2, 0, 4, wallH, span, 0x4b5563);
            addBoxObstacle(ARENA_HALF_SIZE, wallH / 2, 0, 4, wallH, span, 0x4b5563);

            // Central Tactical Outpost & 4 Quadrant Compounds
            const compoundCoords = [
                [0, 0], [-85, -85], [85, -85], [-85, 85], [85, 85],
                [0, -120], [0, 120], [-120, 0], [120, 0]
            ];
            compoundCoords.forEach(([cx, cz]) => {
                // Cabin walls with doorway openings
                addBoxObstacle(cx - 6, 2.2, cz - 6, 12, 4.4, 0.8, 0x6b4423);
                addBoxObstacle(cx - 6, 2.2, cz + 6, 12, 4.4, 0.8, 0x6b4423);
                addBoxObstacle(cx - 11.6, 2.2, cz, 0.8, 4.4, 12, 0x5c3a1d);
                // Roof
                const roof = new THREE.Mesh(
                    new THREE.BoxGeometry(13.5, 0.5, 13.5),
                    new THREE.MeshStandardMaterial({ color: 0x374151, roughness: 0.8 })
                );
                roof.position.set(cx - 6, 4.65, cz);
                roof.castShadow = true;
                scene.add(roof);

                // Low concrete sandbag cover nearby
                addBoxObstacle(cx + 8, 0.65, cz + 4, 4.5, 1.3, 1.1, 0x78716c);
            });

            // Scatter 220 Pine Trees across the 390x390 forest
            const trunkGeo = new THREE.CylinderGeometry(0.42, 0.58, 4.5, 7);
            const trunkMat = new THREE.MeshStandardMaterial({ color: 0x4a2e16, roughness: 0.9 });
            const foliageGeo = new THREE.ConeGeometry(2.7, 7.5, 7);
            const foliageMat = new THREE.MeshStandardMaterial({ color: 0x1e3f1c, roughness: 0.85 });

            let seed = 42;
            const rand = () => {
                seed = (seed * 16807) % 2147483647;
                return (seed - 1) / 2147483646;
            };

            for (let i = 0; i < 220; i++) {
                const tx = (rand() * 2 - 1) * (ARENA_HALF_SIZE - 12);
                const tz = (rand() * 2 - 1) * (ARENA_HALF_SIZE - 12);
                if (Math.hypot(tx, tz) < 14) continue; // Keep center spawn clear

                const trunk = new THREE.Mesh(trunkGeo, trunkMat);
                trunk.position.set(tx, 2.25, tz);
                trunk.castShadow = true;
                trunk.receiveShadow = true;
                scene.add(trunk);

                const foliage = new THREE.Mesh(foliageGeo, foliageMat);
                foliage.position.set(tx, 7.2, tz);
                foliage.castShadow = true;
                scene.add(foliage);

                obstacles.push({
                    minX: tx - 0.65,
                    maxX: tx + 0.65,
                    minZ: tz - 0.65,
                    maxZ: tz + 0.65,
                    height: 5.0
                });
            }

            // Scatter 95 Cover Boulders
            for (let i = 0; i < 95; i++) {
                const bx = (rand() * 2 - 1) * (ARENA_HALF_SIZE - 14);
                const bz = (rand() * 2 - 1) * (ARENA_HALF_SIZE - 14);
                if (Math.hypot(bx, bz) < 12) continue;
                const bw = 2.2 + rand() * 2.4;
                const bh = 1.4 + rand() * 1.4;
                const bd = 2.2 + rand() * 2.4;
                addBoxObstacle(bx, bh / 2, bz, bw, bh, bd, 0x6b7280);
            }
        }

        // =========================================================================
        // 5. FIRST-PERSON WEAPON & FORTNITE MEDKIT VIEW-MODELS
        // =========================================================================
        function buildFirstPersonViewModels() {
            viewModelGroup = new THREE.Group();
            viewModelGroup.position.set(0.28, -0.25, -0.45);
            camera.add(viewModelGroup);

            const darkMetal = new THREE.MeshStandardMaterial({ color: 0x1f2937, metalness: 0.7, roughness: 0.3 });
            const polymer = new THREE.MeshStandardMaterial({ color: 0x111827, roughness: 0.6 });
            const woodMat = new THREE.MeshStandardMaterial({ color: 0x78350f, roughness: 0.7 });
            const steelBlade = new THREE.MeshStandardMaterial({ color: 0xd1d5db, metalness: 0.9, roughness: 0.2 });

            // [0] M4A1 Carbine
            const m4 = new THREE.Group();
            const m4Body = new THREE.Mesh(new THREE.BoxGeometry(0.07, 0.09, 0.44), darkMetal);
            const m4Barrel = new THREE.Mesh(new THREE.CylinderGeometry(0.014, 0.014, 0.28, 8), polymer);
            m4Barrel.rotation.x = Math.PI / 2;
            m4Barrel.position.set(0, 0.015, -0.32);
            const m4Mag = new THREE.Mesh(new THREE.BoxGeometry(0.045, 0.14, 0.07), polymer);
            m4Mag.position.set(0, -0.09, -0.05);
            m4Mag.rotation.x = 0.2;
            m4.add(m4Body, m4Barrel, m4Mag);
            viewModels['m4a1'] = m4;

            // [1] M870 Shotgun
            const m870 = new THREE.Group();
            const sgBody = new THREE.Mesh(new THREE.BoxGeometry(0.075, 0.085, 0.48), darkMetal);
            const sgPump = new THREE.Mesh(new THREE.BoxGeometry(0.085, 0.075, 0.18), woodMat);
            sgPump.position.set(0, -0.015, -0.22);
            m870.add(sgBody, sgPump);
            viewModels['m870'] = m870;

            // [2] M9 9mm Pistol
            const m9 = new THREE.Group();
            const pSlide = new THREE.Mesh(new THREE.BoxGeometry(0.05, 0.06, 0.22), darkMetal);
            const pGrip = new THREE.Mesh(new THREE.BoxGeometry(0.045, 0.12, 0.06), polymer);
            pGrip.position.set(0, -0.07, 0.06);
            pGrip.rotation.x = -0.22;
            m9.add(pSlide, pGrip);
            viewModels['m9'] = m9;

            // [3] Combat Knife
            const knife = new THREE.Group();
            const kHandle = new THREE.Mesh(new THREE.BoxGeometry(0.035, 0.04, 0.12), polymer);
            const kBlade = new THREE.Mesh(new THREE.BoxGeometry(0.012, 0.045, 0.18), steelBlade);
            kBlade.position.set(0, 0.005, -0.14);
            knife.add(kHandle, kBlade);
            viewModels['knife'] = knife;

            // [4] Iconic Red-and-White Hard-Shell Fortnite Medkit
            const medkit = new THREE.Group();
            const redShellMat = new THREE.MeshStandardMaterial({ color: 0xdc2626, roughness: 0.35, metalness: 0.15 });
            const whiteCrossMat = new THREE.MeshStandardMaterial({ color: 0xffffff, roughness: 0.25 });
            const latchMat = new THREE.MeshStandardMaterial({ color: 0x1f2937, roughness: 0.5 });

            const caseBody = new THREE.Mesh(new THREE.BoxGeometry(0.24, 0.18, 0.10), redShellMat);
            // White medical cross on front & back
            const crossV = new THREE.Mesh(new THREE.BoxGeometry(0.036, 0.11, 0.106), whiteCrossMat);
            const crossH = new THREE.Mesh(new THREE.BoxGeometry(0.11, 0.036, 0.106), whiteCrossMat);
            // Top carry handle & latches
            const handleTop = new THREE.Mesh(new THREE.BoxGeometry(0.10, 0.025, 0.03), latchMat);
            handleTop.position.set(0, 0.105, 0);
            const latchL = new THREE.Mesh(new THREE.BoxGeometry(0.028, 0.035, 0.108), latchMat);
            latchL.position.set(-0.07, 0.075, 0);
            const latchR = latchL.clone();
            latchR.position.x = 0.07;

            medkit.add(caseBody, crossV, crossH, handleTop, latchL, latchR);
            medkit.rotation.y = -0.35;
            viewModels['medkit'] = medkit;

            Object.values(viewModels).forEach(vm => {
                vm.visible = false;
                viewModelGroup.add(vm);
            });
        }

        function syncActiveViewModel() {
            const activeId = weapons[currentWeaponIdx]?.id;
            Object.entries(viewModels).forEach(([id, mesh]) => {
                mesh.visible = (id === activeId);
            });
        }

        // =========================================================================
        // 6. LOOT CHESTS (M4A1, M870 SHOTGUN, FORTNITE MEDKITS & AMMO)
        // =========================================================================
        function spawnLootChests() {
            chests.forEach(c => scene.remove(c.group));
            chests.length = 0;

            const chestSpots = [
                [8, 6], [-14, -12], [22, -18], [-28, 25], [42, 35],
                [-48, -42], [65, -55], [-68, 62], [85, 76], [-85, -76],
                [85, -76], [-85, 76], [0, -110], [0, 110], [-110, 0],
                [110, 0], [140, 130], [-140, -130], [140, -130], [-140, 130],
                [35, -95], [-40, 98], [98, -32]
            ];

            const woodMat = new THREE.MeshStandardMaterial({ color: 0x92400e, roughness: 0.7 });
            const goldMat = new THREE.MeshStandardMaterial({ color: 0xfbbf24, metalness: 0.75, roughness: 0.25, emissive: 0x78350f, emissiveIntensity: 0.35 });

            chestSpots.forEach(([x, z], idx) => {
                const group = new THREE.Group();
                group.position.set(x, 0.45, z);

                const base = new THREE.Mesh(new THREE.BoxGeometry(1.3, 0.65, 0.85), woodMat);
                base.castShadow = true;
                base.receiveShadow = true;

                const lid = new THREE.Mesh(new THREE.BoxGeometry(1.34, 0.28, 0.88), goldMat);
                lid.position.set(0, 0.44, 0);
                lid.castShadow = true;

                const beacon = new THREE.Mesh(
                    new THREE.OctahedronGeometry(0.24, 0),
                    new THREE.MeshBasicMaterial({ color: 0xfde047 })
                );
                beacon.position.set(0, 1.25, 0);

                group.add(base, lid, beacon);
                scene.add(group);

                chests.push({
                    id: idx,
                    group,
                    lid,
                    beacon,
                    x,
                    z,
                    opened: false,
                    restockTimer: 0
                });
            });
        }

        function openChest(chest, isPlayer = true) {
            if (chest.opened) return;
            chest.opened = true;
            chest.restockTimer = 18.0;
            chest.lid.rotation.x = -0.85;
            chest.lid.position.y = 0.62;
            chest.lid.position.z = -0.25;
            chest.beacon.visible = false;

            if (isPlayer) {
                playLootSound();
                const roll = Math.random();
                let lootMsg = '';
                if (!weapons[0].unlocked && roll < 0.45) {
                    weapons[0].unlocked = true;
                    weapons[0].mag = weapons[0].magSize;
                    weapons[0].reserve = 120;
                    switchWeapon(0);
                    lootMsg = 'Looted M4A1 Carbine + 120 Ammo!';
                } else if (!weapons[1].unlocked && roll < 0.8) {
                    weapons[1].unlocked = true;
                    weapons[1].mag = weapons[1].magSize;
                    weapons[1].reserve = 36;
                    switchWeapon(1);
                    lootMsg = 'Looted M870 Shotgun + 36 Shells!';
                } else {
                    medkitCount = Math.min(3, medkitCount + 1);
                    weapons[4].unlocked = true;
                    weapons[0].reserve += 30;
                    weapons[1].reserve += 8;
                    weapons[2].reserve += 15;
                    lootMsg = 'Looted Fortnite Medkit + Extra Ammo!';
                }
                addKillFeedEntry(`SUPPLY CHEST: ${lootMsg}`, 'text-amber-300');
                updateWeaponHUD();
            }
        }

        // =========================================================================
        // 7. 19 ARTICULATED 3D ENEMY BOTS (200 HP, NO RESPAWN)
        // =========================================================================
        function createSoldierMesh(shirtColor) {
            const root = new THREE.Group();
            const skinMat = new THREE.MeshStandardMaterial({ color: 0xd9a077, roughness: 0.7 });
            const uniformMat = new THREE.MeshStandardMaterial({ color: shirtColor, roughness: 0.8 });
            const pantsMat = new THREE.MeshStandardMaterial({ color: 0x1f2937, roughness: 0.85 });
            const helmetMat = new THREE.MeshStandardMaterial({ color: 0x365314, roughness: 0.6 });

            const torso = new THREE.Mesh(new THREE.BoxGeometry(0.62, 0.78, 0.34), uniformMat);
            torso.position.y = 1.12;
            torso.castShadow = true;

            const head = new THREE.Mesh(new THREE.BoxGeometry(0.36, 0.36, 0.36), skinMat);
            head.position.y = 1.72;
            head.castShadow = true;

            const helmet = new THREE.Mesh(new THREE.BoxGeometry(0.40, 0.18, 0.40), helmetMat);
            helmet.position.y = 1.86;

            const leftLeg = new THREE.Mesh(new THREE.BoxGeometry(0.24, 0.72, 0.26), pantsMat);
            leftLeg.position.set(-0.17, 0.36, 0);
            leftLeg.castShadow = true;

            const rightLeg = new THREE.Mesh(new THREE.BoxGeometry(0.24, 0.72, 0.26), pantsMat);
            rightLeg.position.set(0.17, 0.36, 0);
            rightLeg.castShadow = true;

            const rifle = new THREE.Mesh(
                new THREE.BoxGeometry(0.08, 0.10, 0.65),
                new THREE.MeshStandardMaterial({ color: 0x111827, metalness: 0.6 })
            );
            rifle.position.set(0.34, 1.18, 0.25);

            // Health bar above bot
            const hpBg = new THREE.Mesh(
                new THREE.PlaneGeometry(0.9, 0.11),
                new THREE.MeshBasicMaterial({ color: 0x111827, side: THREE.DoubleSide })
            );
            hpBg.position.y = 2.15;

            const hpFill = new THREE.Mesh(
                new THREE.PlaneGeometry(0.86, 0.08),
                new THREE.MeshBasicMaterial({ color: 0xef4444, side: THREE.DoubleSide })
            );
            hpFill.position.set(0, 2.15, 0.01);

            root.add(torso, head, helmet, leftLeg, rightLeg, rifle, hpBg, hpFill);
            return { root, head, torso, leftLeg, rightLeg, hpBg, hpFill };
        }

        function spawnAllBots() {
            bots.forEach(b => scene.remove(b.mesh.root));
            bots.length = 0;

            const colors = [0x991b1b, 0x7c2d12, 0x3f6212, 0x1e3a8a, 0x581c87, 0x854d0e];

            for (let i = 0; i < BOT_COUNT; i++) {
                const angle = (i / BOT_COUNT) * Math.PI * 2;
                const radius = 38 + (i % 4) * 36;
                const bx = Math.cos(angle) * radius;
                const bz = Math.sin(angle) * radius;

                const mesh = createSoldierMesh(colors[i % colors.length]);
                mesh.root.position.set(bx, 0, bz);
                scene.add(mesh.root);

                bots.push({
                    id: i,
                    name: BOT_CALLSIGNS[i] || `Bot-${i + 1}`,
                    alive: true,
                    health: BOT_MAX_HEALTH,
                    kills: 0,
                    pos: new THREE.Vector3(bx, 0, bz),
                    yaw: Math.random() * Math.PI * 2,
                    moveTimer: Math.random() * 2,
                    fireCooldown: 0.4 + Math.random() * 0.8,
                    hasRifle: false,
                    walkPhase: Math.random() * 6,
                    mesh
                });
            }
        }

        // =========================================================================
        // 8. WEAPONS, 3.0s FORTNITE MEDKIT & COMBAT DAMAGE (25 HEAD / 15 BODY)
        // =========================================================================
        function initPlayerLoadout() {
            weapons = WEAPON_DEFS.map(def => ({
                ...def,
                mag: def.magSize,
                reserve: def.reserve
            }));
            medkitCount = 0;
            isApplyingMedkit = false;
            medkitTimer = 0;
            isReloading = false;
            currentWeaponIdx = 2; // Start with M9 Pistol
            syncActiveViewModel();
            updateWeaponHUD();
        }

        function switchWeapon(idx) {
            if (idx < 0 || idx >= weapons.length) return;
            if (idx === 4 && medkitCount <= 0) return;
            if (!weapons[idx].unlocked && idx !== 4) return;

            if (isApplyingMedkit) {
                cancelMedkit();
            }
            isReloading = false;
            currentWeaponIdx = idx;
            syncActiveViewModel();
            updateWeaponHUD();
        }

        function startMedkitChannel() {
            if (playerDead || isApplyingMedkit) return;
            if (medkitCount <= 0) {
                addKillFeedEntry('No Medkits! Open glowing Supply Chests [E] to find one.', 'text-stone-400');
                return;
            }
            if (playerHealth >= PLAYER_MAX_HEALTH) {
                addKillFeedEntry('Already at full 200 HP!', 'text-lime-300');
                return;
            }
            switchWeapon(4);
            isApplyingMedkit = true;
            medkitTimer = MEDKIT_DURATION;
            document.getElementById('medkit-progress-wrap').classList.remove('hidden');
        }

        function cancelMedkit() {
            isApplyingMedkit = false;
            medkitTimer = 0;
            document.getElementById('medkit-progress-wrap').classList.add('hidden');
        }

        function completeMedkitHeal() {
            isApplyingMedkit = false;
            medkitTimer = 0;
            document.getElementById('medkit-progress-wrap').classList.add('hidden');
            if (medkitCount > 0) {
                medkitCount--;
                playerHealth = PLAYER_MAX_HEALTH;
                playHealCompleteSound();
                addKillFeedEntry('MEDKIT APPLIED: Restored to 200 HP!', 'text-emerald-300');
                if (medkitCount === 0) {
                    weapons[4].unlocked = false;
                    const fallbackIdx = weapons[0].unlocked ? 0 : (weapons[1].unlocked ? 1 : 2);
                    switchWeapon(fallbackIdx);
                }
                updateHealthHUD();
                updateWeaponHUD();
            }
        }

        function reloadCurrentWeapon() {
            const w = weapons[currentWeaponIdx];
            if (!w || w.id === 'knife' || w.id === 'medkit' || isReloading) return;
            if (w.mag >= w.magSize || w.reserve <= 0) return;

            isReloading = true;
            setTimeout(() => {
                if (!isReloading) return;
                const needed = w.magSize - w.mag;
                const load = Math.min(needed, w.reserve);
                w.mag += load;
                w.reserve -= load;
                isReloading = false;
                updateWeaponHUD();
            }, 950);
        }

        function spawnBulletTracer(startPos, endPos, color = 0xfef08a) {
            const geo = new THREE.BufferGeometry().setFromPoints([startPos, endPos]);
            const mat = new THREE.LineBasicMaterial({ color, transparent: true, opacity: 0.85 });
            const line = new THREE.Line(geo, mat);
            scene.add(line);
            tracers.push({ line, life: 0.08 });
        }

        function showHitmarker(isHeadshot) {
            const hm = document.getElementById('hitmarker');
            hm.style.opacity = '1';
            hm.querySelectorAll('div').forEach(d => {
                d.style.backgroundColor = isHeadshot ? '#facc15' : '#ef4444';
            });
            setTimeout(() => { hm.style.opacity = '0'; }, 110);
        }

        function fireCurrentWeapon() {
            if (playerDead || isMenuOpen) return;
            const w = weapons[currentWeaponIdx];
            if (!w) return;

            if (w.id === 'medkit') {
                startMedkitChannel();
                return;
            }

            const now = performance.now() / 1000;
            if (now - lastFireTime < w.fireRate || isReloading) return;

            if (w.id !== 'knife') {
                if (w.mag <= 0) {
                    reloadCurrentWeapon();
                    return;
                }
                w.mag--;
                updateWeaponHUD();
            }

            lastFireTime = now;
            recoilOffset = Math.min(0.08, recoilOffset + 0.032);
            playShotSound(w.id);

            const forward = new THREE.Vector3(0, 0, -1).applyEuler(camera.rotation);
            const muzzlePos = camera.position.clone().addScaledVector(forward, 0.6);

            for (let p = 0; p < w.pellets; p++) {
                const dir = forward.clone();
                dir.x += (Math.random() - 0.5) * w.spread;
                dir.y += (Math.random() - 0.5) * w.spread;
                dir.z += (Math.random() - 0.5) * w.spread;
                dir.normalize();

                let closestDist = w.range;
                let hitBot = null;
                let isHeadshot = false;

                // Check obstacle occlusion first
                obstacles.forEach(obs => {
                    const cx = (obs.minX + obs.maxX) / 2;
                    const cz = (obs.minZ + obs.maxZ) / 2;
                    const toObs = new THREE.Vector3(cx - camera.position.x, 0, cz - camera.position.z);
                    const proj = toObs.dot(dir);
                    if (proj > 0 && proj < closestDist) {
                        const hitPt = camera.position.clone().addScaledVector(dir, proj);
                        if (hitPt.x >= obs.minX && hitPt.x <= obs.maxX && hitPt.z >= obs.minZ && hitPt.z <= obs.maxZ && hitPt.y <= obs.height) {
                            closestDist = proj;
                        }
                    }
                });

                // Check bot head & body hitboxes
                bots.forEach(bot => {
                    if (!bot.alive) return;
                    const headCenter = new THREE.Vector3(bot.pos.x, 1.74, bot.pos.z);
                    const bodyCenter = new THREE.Vector3(bot.pos.x, 1.05, bot.pos.z);

                    const toHead = headCenter.clone().sub(camera.position);
                    const headProj = toHead.dot(dir);
                    if (headProj > 0 && headProj < closestDist) {
                        const perpHead = headCenter.distanceTo(camera.position.clone().addScaledVector(dir, headProj));
                        if (perpHead < 0.28) {
                            closestDist = headProj;
                            hitBot = bot;
                            isHeadshot = true;
                            return;
                        }
                    }

                    const toBody = bodyCenter.clone().sub(camera.position);
                    const bodyProj = toBody.dot(dir);
                    if (bodyProj > 0 && bodyProj < closestDist) {
                        const perpBody = bodyCenter.distanceTo(camera.position.clone().addScaledVector(dir, bodyProj));
                        if (perpBody < 0.44) {
                            closestDist = bodyProj;
                            hitBot = bot;
                            isHeadshot = false;
                        }
                    }
                });

                const endPoint = camera.position.clone().addScaledVector(dir, closestDist);
                if (w.id !== 'knife') {
                    spawnBulletTracer(muzzlePos, endPoint, isHeadshot ? 0xfacc15 : 0xfef08a);
                }

                if (hitBot) {
                    const dmg = isHeadshot ? HEADSHOT_DAMAGE : BODYSHOT_DAMAGE;
                    applyDamageToBot(hitBot, dmg, 'YOU', isHeadshot);
                    playHitSound(isHeadshot);
                    showHitmarker(isHeadshot);
                }
            }
        }

        function applyDamageToBot(bot, amount, attackerName, isHeadshot) {
            if (!bot.alive) return;
            bot.health = Math.max(0, bot.health - amount);
            const ratio = bot.health / BOT_MAX_HEALTH;
            bot.mesh.hpFill.scale.x = Math.max(0.001, ratio);
            bot.mesh.hpFill.position.x = -(1 - ratio) * 0.43;

            if (bot.health <= 0) {
                bot.alive = false;
                bot.mesh.root.visible = false;
                if (attackerName === 'YOU') {
                    playerKills++;
                    document.getElementById('hud-kills-val').textContent = playerKills;
                }
                addKillFeedEntry(
                    `${attackerName} eliminated ${bot.name} ${isHeadshot ? '(HEADSHOT)' : ''}`,
                    attackerName === 'YOU' ? 'text-lime-300' : 'text-stone-300'
                );
                updateAliveRosterHUD();
                checkMatchEndCondition();
            }
        }

        function applyDamageToPlayer(amount, attackerName, isHeadshot) {
            if (playerDead) return;
            if (isApplyingMedkit) cancelMedkit();

            playerHealth = Math.max(0, playerHealth - amount);
            updateHealthHUD();

            const flash = document.getElementById('damage-flash');
            flash.style.opacity = '1';
            setTimeout(() => { flash.style.opacity = '0'; }, 140);

            if (playerHealth <= 0) {
                playerDead = true;
                const aliveBots = bots.filter(b => b.alive).length;
                addKillFeedEntry(`${attackerName} eliminated YOU!`, 'text-red-400');
                updateAliveRosterHUD();
                showMatchEndBanner(false, aliveBots + 1);
            }
        }

        // =========================================================================
        // 9. BOT AI & AUTO-SIMULATION PILOT ([M])
        // =========================================================================
        function updateBotsAI(dt) {
            const aliveBots = bots.filter(b => b.alive);

            aliveBots.forEach(bot => {
                // Find nearest target (player or another alive bot)
                let targetPos = null;
                let targetBot = null;
                let targetIsPlayer = false;
                let bestDist = 110;

                if (!playerDead) {
                    const dPlayer = bot.pos.distanceTo(playerPos);
                    if (dPlayer < bestDist) {
                        bestDist = dPlayer;
                        targetPos = playerPos;
                        targetIsPlayer = true;
                    }
                }

                aliveBots.forEach(other => {
                    if (other.id === bot.id) return;
                    const dOther = bot.pos.distanceTo(other.pos);
                    if (dOther < bestDist) {
                        bestDist = dOther;
                        targetPos = other.pos;
                        targetBot = other;
                        targetIsPlayer = false;
                    }
                });

                // Check nearby unopened chest
                chests.forEach(c => {
                    if (!c.opened && Math.hypot(bot.pos.x - c.x, bot.pos.z - c.z) < 2.6) {
                        openChest(c, false);
                        bot.hasRifle = true;
                    }
                });

                if (targetPos) {
                    const dx = targetPos.x - bot.pos.x;
                    const dz = targetPos.z - bot.pos.z;
                    bot.yaw = Math.atan2(dx, dz);

                    // Strafe / advance toward combat range
                    const desiredRange = bot.hasRifle ? 22 : 14;
                    if (bestDist > desiredRange) {
                        const step = 6.2 * dt;
                        const nx = bot.pos.x + Math.sin(bot.yaw) * step;
                        const nz = bot.pos.z + Math.cos(bot.yaw) * step;
                        if (!checkCircleCollision(nx, nz, 0.5)) {
                            bot.pos.x = nx;
                            bot.pos.z = nz;
                        }
                    }

                    // Bot shooting
                    bot.fireCooldown -= dt;
                    if (bot.fireCooldown <= 0 && bestDist < 75) {
                        bot.fireCooldown = bot.hasRifle ? (0.45 + Math.random() * 0.4) : (0.75 + Math.random() * 0.5);
                        const start = new THREE.Vector3(bot.pos.x, 1.4, bot.pos.z);
                        const end = new THREE.Vector3(targetPos.x, 1.35, targetPos.z);
                        spawnBulletTracer(start, end, 0xf87171);

                        // Accuracy roll (25 headshot / 15 body)
                        const hitRoll = Math.random();
                        if (hitRoll < 0.34) {
                            const isHead = hitRoll < 0.08;
                            const dmg = isHead ? HEADSHOT_DAMAGE : BODYSHOT_DAMAGE;
                            if (targetIsPlayer) {
                                applyDamageToPlayer(dmg, bot.name, isHead);
                            } else if (targetBot) {
                                applyDamageToBot(targetBot, dmg, bot.name, isHead);
                                if (!targetBot.alive) bot.kills++;
                            }
                        }
                    }
                } else {
                    // Patrol roam
                    bot.moveTimer -= dt;
                    if (bot.moveTimer <= 0) {
                        bot.moveTimer = 2 + Math.random() * 3;
                        bot.yaw += (Math.random() - 0.5) * 2.4;
                    }
                    const step = 4.5 * dt;
                    const nx = Math.max(-ARENA_HALF_SIZE + 5, Math.min(ARENA_HALF_SIZE - 5, bot.pos.x + Math.sin(bot.yaw) * step));
                    const nz = Math.max(-ARENA_HALF_SIZE + 5, Math.min(ARENA_HALF_SIZE - 5, bot.pos.z + Math.cos(bot.yaw) * step));
                    if (!checkCircleCollision(nx, nz, 0.5)) {
                        bot.pos.x = nx;
                        bot.pos.z = nz;
                    } else {
                        bot.yaw += Math.PI * 0.6;
                    }
                }

                // Animate bot legs & billboard health bar
                bot.walkPhase += dt * 9;
                bot.mesh.leftLeg.rotation.x = Math.sin(bot.walkPhase) * 0.45;
                bot.mesh.rightLeg.rotation.x = -Math.sin(bot.walkPhase) * 0.45;
                bot.mesh.root.position.set(bot.pos.x, 0, bot.pos.z);
                bot.mesh.root.rotation.y = bot.yaw;
                bot.mesh.hpBg.lookAt(camera.position);
                bot.mesh.hpFill.lookAt(camera.position);
            });
        }

        function updateAutoSimPilot(dt) {
            if (!simModeEnabled || playerDead) return;

            // Heal if damaged and carrying a Medkit
            if (playerHealth < 120 && medkitCount > 0 && !isApplyingMedkit) {
                startMedkitChannel();
                return;
            }
            if (isApplyingMedkit) return;

            // Find nearest alive bot
            let nearestBot = null;
            let nearestDist = Infinity;
            bots.forEach(b => {
                if (!b.alive) return;
                const d = playerPos.distanceTo(b.pos);
                if (d < nearestDist) {
                    nearestDist = d;
                    nearestBot = b;
                }
            });

            if (nearestBot) {
                const dx = nearestBot.pos.x - playerPos.x;
                const dz = nearestBot.pos.z - playerPos.z;
                const targetYaw = Math.atan2(-dx, -dz);
                playerYaw = targetYaw;
                playerPitch = -0.02;

                if (nearestDist > 18) {
                    const step = 8.5 * dt;
                    const nx = playerPos.x - Math.sin(playerYaw) * step;
                    const nz = playerPos.z - Math.cos(playerYaw) * step;
                    if (!checkCircleCollision(nx, nz, 0.55)) {
                        playerPos.x = nx;
                        playerPos.z = nz;
                    }
                }
                if (nearestDist < 70) {
                    fireCurrentWeapon();
                }
            }

            // Auto-loot nearby chests
            chests.forEach(c => {
                if (!c.opened && Math.hypot(playerPos.x - c.x, playerPos.z - c.z) < 3.2) {
                    openChest(c, true);
                }
            });
        }

        // =========================================================================
        // 10. PLAYER MOVEMENT, COLLISIONS & MINIMAP RADAR
        // =========================================================================
        function checkCircleCollision(x, z, radius) {
            if (Math.abs(x) > ARENA_HALF_SIZE - 2.5 || Math.abs(z) > ARENA_HALF_SIZE - 2.5) return true;
            for (let i = 0; i < obstacles.length; i++) {
                const o = obstacles[i];
                if (x + radius > o.minX && x - radius < o.maxX && z + radius > o.minZ && z - radius < o.maxZ) {
                    return true;
                }
            }
            return false;
        }

        function updatePlayerMovement(dt) {
            if (playerDead || isMenuOpen) return;

            // Arrow-key camera turn support
            const turnSpeed = 2.1 * dt;
            if (keys['ArrowLeft']) playerYaw += turnSpeed;
            if (keys['ArrowRight']) playerYaw -= turnSpeed;
            if (keys['ArrowUp']) playerPitch = Math.min(Math.PI / 2.2, playerPitch + turnSpeed * 0.7);
            if (keys['ArrowDown']) playerPitch = Math.max(-Math.PI / 2.2, playerPitch - turnSpeed * 0.7);

            camera.rotation.y = playerYaw;
            camera.rotation.x = playerPitch + recoilOffset;
            recoilOffset = Math.max(0, recoilOffset - dt * 0.35);

            if (!simModeEnabled) {
                let forwardMove = 0;
                let rightMove = 0;
                if (keys['KeyW']) forwardMove += 1;
                if (keys['KeyS']) forwardMove -= 1;
                if (keys['KeyD']) rightMove += 1;
                if (keys['KeyA']) rightMove -= 1;

                const speed = (keys['ShiftLeft'] || keys['ShiftRight']) ? 11.5 : 7.8;
                const sinY = Math.sin(playerYaw);
                const cosY = Math.cos(playerYaw);

                const moveX = (-sinY * forwardMove + cosY * rightMove);
                const moveZ = (-cosY * forwardMove - sinY * rightMove);
                const len = Math.hypot(moveX, moveZ);

                if (len > 0) {
                    const nx = playerPos.x + (moveX / len) * speed * dt;
                    const nz = playerPos.z + (moveZ / len) * speed * dt;
                    if (!checkCircleCollision(nx, playerPos.z, 0.55)) playerPos.x = nx;
                    if (!checkCircleCollision(playerPos.x, nz, 0.55)) playerPos.z = nz;
                }

                // Jump & gravity
                if (keys['Space'] && onGround) {
                    playerVel.y = 7.6;
                    onGround = false;
                }
                playerVel.y -= 20.0 * dt;
                playerPos.y += playerVel.y * dt;
                if (playerPos.y <= 1.75) {
                    playerPos.y = 1.75;
                    playerVel.y = 0;
                    onGround = true;
                }
            }

            camera.position.copy(playerPos);

            // Auto-fire if holding M4A1
            if (isFiring && weapons[currentWeaponIdx]?.auto) {
                fireCurrentWeapon();
            }

            // Medkit 3.0s progress timer
            if (isApplyingMedkit) {
                medkitTimer -= dt;
                const pct = Math.min(100, Math.max(0, ((MEDKIT_DURATION - medkitTimer) / MEDKIT_DURATION) * 100));
                document.getElementById('medkit-progress-bar').style.width = `${pct}%`;
                document.getElementById('medkit-timer-text').textContent = `${Math.max(0, medkitTimer).toFixed(1)}s`;
                if (medkitTimer <= 0) {
                    completeMedkitHeal();
                }
            }

            // Check nearby chest prompt
            let nearChest = null;
            for (const c of chests) {
                if (!c.opened && Math.hypot(playerPos.x - c.x, playerPos.z - c.z) < 3.4) {
                    nearChest = c;
                    break;
                }
            }
            const promptEl = document.getElementById('loot-prompt');
            if (nearChest) {
                promptEl.classList.remove('hidden');
            } else {
                promptEl.classList.add('hidden');
            }
        }

        function drawMinimap() {
            const canvas = document.getElementById('minimap-canvas');
            if (!canvas) return;
            const ctx = canvas.getContext('2d');
            const w = canvas.width;
            const h = canvas.height;
            ctx.clearRect(0, 0, w, h);

            const scale = w / (ARENA_HALF_SIZE * 2);
            const toMapX = (wx) => (wx + ARENA_HALF_SIZE) * scale;
            const toMapY = (wz) => (wz + ARENA_HALF_SIZE) * scale;

            // Grid lines
            ctx.strokeStyle = 'rgba(163, 230, 53, 0.12)';
            ctx.lineWidth = 1;
            for (let g = 1; g < 4; g++) {
                ctx.beginPath();
                ctx.moveTo((w / 4) * g, 0);
                ctx.lineTo((w / 4) * g, h);
                ctx.stroke();
                ctx.beginPath();
                ctx.moveTo(0, (h / 4) * g);
                ctx.lineTo(w, (h / 4) * g);
                ctx.stroke();
            }

            // Unopened Loot Chests
            chests.forEach(c => {
                if (c.opened) return;
                ctx.fillStyle = '#fbbf24';
                ctx.fillRect(toMapX(c.x) - 2, toMapY(c.z) - 2, 4, 4);
            });

            // Alive Bots
            bots.forEach(b => {
                if (!b.alive) return;
                ctx.fillStyle = '#ef4444';
                ctx.beginPath();
                ctx.arc(toMapX(b.pos.x), toMapY(b.pos.z), 2.8, 0, Math.PI * 2);
                ctx.fill();
            });

            // Player dot + look cone
            if (!playerDead) {
                const px = toMapX(playerPos.x);
                const py = toMapY(playerPos.z);

                ctx.strokeStyle = 'rgba(163, 230, 53, 0.6)';
                ctx.lineWidth = 1.5;
                ctx.beginPath();
                ctx.moveTo(px, py);
                ctx.lineTo(px - Math.sin(playerYaw) * 14, py - Math.cos(playerYaw) * 14);
                ctx.stroke();

                ctx.fillStyle = '#a3e635';
                ctx.beginPath();
                ctx.arc(px, py, 3.6, 0, Math.PI * 2);
                ctx.fill();
            }
        }

        // =========================================================================
        // 11. HUD UPDATES, ROSTER & MATCH FLOW
        // =========================================================================
        function updateHealthHUD() {
            document.getElementById('hud-health-val').textContent = playerHealth;
            const pct = (playerHealth / PLAYER_MAX_HEALTH) * 100;
            const bar = document.getElementById('hud-health-bar');
            bar.style.width = `${pct}%`;
            bar.className = playerHealth > 75
                ? 'h-full bg-gradient-to-r from-emerald-500 to-lime-400 transition-all duration-150'
                : 'h-full bg-gradient-to-r from-red-600 to-amber-500 transition-all duration-150';
        }

        function updateWeaponHUD() {
            const w = weapons[currentWeaponIdx];
            if (!w) return;
            document.getElementById('hud-weapon-name').textContent = w.name;
            if (w.id === 'knife') {
                document.getElementById('hud-ammo-mag').textContent = 'INF';
                document.getElementById('hud-ammo-res').textContent = '';
            } else if (w.id === 'medkit') {
                document.getElementById('hud-ammo-mag').textContent = medkitCount;
                document.getElementById('hud-ammo-res').textContent = 'KIT';
            } else {
                document.getElementById('hud-ammo-mag').textContent = w.mag;
                document.getElementById('hud-ammo-res').textContent = `/ ${w.reserve}`;
            }

            document.getElementById('slot-4-label').textContent = `MED (${medkitCount})`;

            for (let i = 0; i < 5; i++) {
                const el = document.getElementById(`slot-${i}`);
                const unlocked = (i === 4) ? (medkitCount > 0) : weapons[i].unlocked;
                el.style.opacity = unlocked ? '1' : '0.4';
                if (i === currentWeaponIdx) {
                    el.className = 'glass-panel px-2.5 py-1.5 rounded-xl border-2 border-lime-400 text-center min-w-[54px]';
                } else {
                    el.className = 'glass-panel px-2.5 py-1.5 rounded-xl border border-stone-700 text-center min-w-[54px]';
                }
            }
        }

        function addKillFeedEntry(text, colorClass = 'text-stone-200') {
            const feed = document.getElementById('kill-feed');
            const item = document.createElement('div');
            item.className = `glass-panel px-2.5 py-1 rounded-lg text-[11px] font-bold ${colorClass} shadow`;
            item.textContent = text;
            feed.prepend(item);
            while (feed.children.length > 4) {
                feed.removeChild(feed.lastChild);
            }
            setTimeout(() => { if (item.parentNode) item.remove(); }, 5500);
        }

        function updateAliveRosterHUD() {
            const aliveBots = bots.filter(b => b.alive).length;
            const totalAlive = aliveBots + (playerDead ? 0 : 1);
            document.getElementById('hud-mode-badge').textContent = `Woodland BR (${totalAlive} Alive)`;
            document.getElementById('menu-alive-count').textContent = `${totalAlive} ALIVE`;

            const roster = document.getElementById('roster-list');
            if (!roster) return;
            let html = `
                <div class="flex justify-between items-center px-2.5 py-1.5 rounded-lg ${playerDead ? 'bg-red-950/40 text-stone-500' : 'bg-lime-950/50 text-lime-300 font-bold'}">
                    <span>YOU (Commander)</span>
                    <span class="mono">${playerDead ? 'OUT' : `${playerHealth} HP • ${playerKills} K`}</span>
                </div>
            `;
            bots.forEach(b => {
                html += `
                    <div class="flex justify-between items-center px-2.5 py-1 rounded-lg ${b.alive ? 'bg-stone-800/60 text-stone-200' : 'bg-stone-900/40 text-stone-600 line-through'}">
                        <span>${b.name}</span>
                        <span class="mono">${b.alive ? `${b.health} HP` : 'OUT'}</span>
                    </div>
                `;
            });
            roster.innerHTML = html;
        }

        function checkMatchEndCondition() {
            const aliveBots = bots.filter(b => b.alive).length;
            if (!playerDead && aliveBots === 0) {
                showMatchEndBanner(true, 1);
            }
        }

        function showMatchEndBanner(isVictory, placement) {
            const overlay = document.getElementById('match-end-overlay');
            overlay.classList.remove('hidden');
            overlay.classList.add('flex');
            document.getElementById('end-banner-tag').textContent = isVictory ? 'VICTORY ROYALE' : 'ELIMINATED';
            document.getElementById('end-banner-title').textContent = isVictory ? 'LAST SOLDIER STANDING!' : 'YOU WERE ELIMINATED';
            document.getElementById('end-banner-sub').textContent = isVictory
                ? 'You outlasted all 19 enemy soldiers across the 390×390 woodland arena.'
                : `You finished #${placement} out of 20 combatants.`;
            document.getElementById('end-stat-kills').textContent = playerKills;
            document.getElementById('end-stat-place').textContent = `#${placement} / 20`;
        }

        function resetSoloMatch() {
            document.getElementById('match-end-overlay').classList.add('hidden');
            document.getElementById('match-end-overlay').classList.remove('flex');
            playerHealth = PLAYER_MAX_HEALTH;
            playerKills = 0;
            playerDead = false;
            playerPos.set(0, 1.75, 0);
            playerVel.set(0, 0, 0);
            playerYaw = 0;
            playerPitch = 0;
            document.getElementById('hud-kills-val').textContent = '0';
            initPlayerLoadout();
            spawnLootChests();
            spawnAllBots();
            updateHealthHUD();
            updateAliveRosterHUD();
            resumeGameplay();
            addKillFeedEntry('New 20-Player Battle Royale Started! Loot chests [E] for weapons & Medkits.', 'text-lime-300');
        }

        function toggleSimMode() {
            simModeEnabled = !simModeEnabled;
            const btn = document.getElementById('btn-sim-toggle');
            btn.textContent = simModeEnabled ? '[M] Mode: Auto-Bot Sim' : '[M] Mode: Manual Play';
            btn.className = simModeEnabled
                ? 'px-2.5 py-1 rounded-lg bg-amber-500 text-stone-950 font-extrabold text-[11px] transition'
                : 'px-2.5 py-1 rounded-lg bg-stone-800 hover:bg-stone-700 border border-stone-600 text-[11px] font-bold text-emerald-300 transition';
        }

        function setSimSpeed(mult) {
            simSpeedMultiplier = mult;
            [1, 2, 4].forEach(s => {
                const el = document.getElementById(`spd-${s}`);
                if (el) {
                    el.className = (s === mult)
                        ? 'px-2 py-0.5 rounded bg-lime-600 text-stone-950 font-bold text-[10px]'
                        : 'px-2 py-0.5 rounded bg-stone-800 text-stone-300 font-bold text-[10px]';
                }
            });
        }

        // =========================================================================
        // 12. POINTER LOCK & SANDBOX FREE-AIM CONTROLS
        // =========================================================================
        function requestCanvasLockSafely() {
            const canvas = renderer && renderer.domElement;
            if (!canvas || pointerLockUnsupported) return;
            try {
                if (document.pointerLockElement !== canvas && canvas.requestPointerLock) {
                    const promise = canvas.requestPointerLock();
                    if (promise && typeof promise.catch === 'function') {
                        promise.catch(() => {
                            pointerLockUnsupported = true;
                            updateAimStatusHUD();
                        });
                    }
                }
            } catch (err) {
                pointerLockUnsupported = true;
                updateAimStatusHUD();
            }
        }

        function updateAimStatusHUD() {
            const statusEl = document.getElementById('hud-aim-status');
            if (!statusEl) return;
            const locked = renderer && document.pointerLockElement === renderer.domElement;
            if (locked) {
                statusEl.textContent = 'Cursor Locked • Move mouse to aim • [ESC] unlock';
                statusEl.className = 'text-lime-400 font-semibold';
            } else if (pointerLockUnsupported) {
                statusEl.textContent = 'Sandbox Free-Aim Active • Move mouse or Drag to turn';
                statusEl.className = 'text-sky-300 font-semibold';
            } else {
                statusEl.textContent = 'Click screen to lock cursor (or move mouse / drag)';
                statusEl.className = 'text-amber-300 font-semibold';
            }
        }

        function resumeGameplay() {
            isMenuOpen = false;
            const menu = document.getElementById('pause-menu-overlay');
            menu.classList.add('hidden');
            menu.classList.remove('flex');
            requestCanvasLockSafely();
        }

        function openPauseMenu() {
            isMenuOpen = true;
            isFiring = false;
            updateAliveRosterHUD();
            const menu = document.getElementById('pause-menu-overlay');
            menu.classList.remove('hidden');
            menu.classList.add('flex');
            if (document.pointerLockElement) {
                try { document.exitPointerLock(); } catch (e) {}
            }
        }

        function setupEventListeners() {
            window.addEventListener('resize', () => {
                if (!camera || !renderer) return;
                camera.aspect = window.innerWidth / window.innerHeight;
                camera.updateProjectionMatrix();
                renderer.setSize(window.innerWidth, window.innerHeight);
            });

            document.addEventListener('pointerlockchange', updateAimStatusHUD);
            document.addEventListener('pointerlockerror', () => {
                pointerLockUnsupported = true;
                updateAimStatusHUD();
            });

            renderer.domElement.addEventListener('contextmenu', e => e.preventDefault());

            renderer.domElement.addEventListener('mousedown', (e) => {
                initAudio();
                if (!gameActive || isMenuOpen) return;
                window.focus();
                if (document.pointerLockElement !== renderer.domElement) {
                    requestCanvasLockSafely();
                }
                if (e.button === 0) {
                    isFiring = true;
                    fireCurrentWeapon();
                } else if (e.button === 2) {
                    isRightMouseDown = true;
                    lastMouseX = e.clientX;
                    lastMouseY = e.clientY;
                }
            });

            window.addEventListener('mouseup', (e) => {
                if (e.button === 0) isFiring = false;
                if (e.button === 2) isRightMouseDown = false;
            });

            window.addEventListener('mousemove', (e) => {
                if (!gameActive || isMenuOpen || playerDead || simModeEnabled) return;

                if (document.pointerLockElement === renderer.domElement) {
                    playerYaw -= e.movementX * 0.0024;
                    playerPitch = Math.max(-Math.PI / 2.2, Math.min(Math.PI / 2.2, playerPitch - e.movementY * 0.0024));
                    return;
                }

                // Fallback smooth aim when Pointer Lock is blocked by sandboxed iframe
                if (lastMouseX !== null && lastMouseY !== null) {
                    const dx = e.clientX - lastMouseX;
                    const dy = e.clientY - lastMouseY;
                    const sensitivity = isRightMouseDown ? 0.0045 : 0.0032;
                    playerYaw -= dx * sensitivity;
                    playerPitch = Math.max(-Math.PI / 2.2, Math.min(Math.PI / 2.2, playerPitch - dy * sensitivity));
                }
                lastMouseX = e.clientX;
                lastMouseY = e.clientY;
            });

            window.addEventListener('keydown', (e) => {
                keys[e.code] = true;
                if (e.code === 'KeyP') {
                    isMenuOpen ? resumeGameplay() : openPauseMenu();
                } else if (e.code === 'KeyM') {
                    toggleSimMode();
                } else if (e.code === 'KeyE') {
                    for (const c of chests) {
                        if (!c.opened && Math.hypot(playerPos.x - c.x, playerPos.z - c.z) < 3.4) {
                            openChest(c, true);
                            break;
                        }
                    }
                } else if (e.code === 'KeyH') {
                    startMedkitChannel();
                } else if (e.code === 'KeyR') {
                    reloadCurrentWeapon();
                } else if (e.code === 'KeyF') {
                    fireCurrentWeapon();
                } else if (e.code === 'Digit1') {
                    switchWeapon(0);
                } else if (e.code === 'Digit2') {
                    switchWeapon(1);
                } else if (e.code === 'Digit3') {
                    switchWeapon(2);
                } else if (e.code === 'Digit4') {
                    switchWeapon(3);
                } else if (e.code === 'Digit5') {
                    switchWeapon(4);
                }
            });

            window.addEventListener('keyup', (e) => {
                keys[e.code] = false;
            });
        }

        // =========================================================================
        // 13. MAIN GAME LOOP
        // =========================================================================
        function animateLoop() {
            requestAnimationFrame(animateLoop);
            const rawDt = Math.min(0.05, clock.getDelta());
            const steps = simModeEnabled ? simSpeedMultiplier : 1;

            for (let s = 0; s < steps; s++) {
                if (!isMenuOpen) {
                    updatePlayerMovement(rawDt);
                    updateAutoSimPilot(rawDt);
                    updateBotsAI(rawDt);
                }

                // Spin chest beacons & handle chest restocks
                chests.forEach(c => {
                    if (!c.opened) {
                        c.beacon.rotation.y += rawDt * 2.5;
                    } else {
                        c.restockTimer -= rawDt;
                        if (c.restockTimer <= 0) {
                            c.opened = false;
                            c.lid.rotation.x = 0;
                            c.lid.position.set(0, 0.44, 0);
                            c.beacon.visible = true;
                        }
                    }
                });

                // Fade bullet tracers
                for (let i = tracers.length - 1; i >= 0; i--) {
                    tracers[i].life -= rawDt;
                    if (tracers[i].life <= 0) {
                        scene.remove(tracers[i].line);
                        tracers.splice(i, 1);
                    }
                }
            }

            drawMinimap();
            renderer.render(scene, camera);
        }

        window.onload = function () {
            clock = new THREE.Clock();
            initThree();
            initPlayerLoadout();
            spawnLootChests();
            spawnAllBots();
            setupEventListeners();
            updateHealthHUD();
            updateAliveRosterHUD();
            addKillFeedEntry('Deployed into Woodland BR! Click screen to aim, [E] loot chests, [H] Medkit.', 'text-lime-300');
            animateLoop();
        };
    </script>
</body>
</html>
