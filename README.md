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
            <div class="absolute top-0 w-0.5 h-2 bg-lime-400 shadow"></div>
            <div class="absolute bottom-0 w-0.5 h-2 bg-lime-400 shadow"></div>
            <div class="absolute left-0 h-0.5 w-2 bg-lime-400 shadow"></div>
            <div class="absolute right-0 h-0.5 w-2 bg-lime-400 shadow"></div>
            <div class="w-1 h-1 rounded-full bg-white"></div>
            <div id="hitmarker" class="absolute inset-0 opacity-0 transition-opacity duration-75 flex items-center justify-center">
                <div class="w-5 h-0.5 bg-red-500 rotate-45 absolute"></div>
                <div class="w-5 h-0.5 bg-red-500 -rotate-45 absolute"></div>
            </div>
        </div>
    </div>

    <!-- Chest Loot Prompt -->
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

    <!-- Top-Left Match Status, Aim Mode & Simulation Controls -->
    <div class="fixed top-3 left-3 z-20 flex flex-col gap-2 max-w-xs sm:max-w-sm">
        <div class="glass-panel p-3 rounded-2xl shadow-lg">
            <div class="flex items-center justify-between gap-3">
                <div class="flex items-center gap-2">
                    <span class="w-2.5 h-2.5 rounded-full bg-lime-400 animate-pulse"></span>
                    <span id="hud-mode-badge" class="text-xs font-extrabold uppercase tracking-wider text-lime-300">Woodland BR (20 Alive)</span>
                </div>
                <span class="mono text-[11px] text-stone-400">SOLO BR</span>
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

    <!-- Pause & Roster Overlay ([P]) -->
    <div id="pause-menu-overlay" class="fixed inset-0 z-30 hidden items-center justify-center bg-black/70 backdrop-blur-md p-4">
        <div class="glass-panel max-w-2xl w-full p-6 rounded-3xl shadow-2xl border border-lime-400/30 max-h-[90vh] overflow-y-auto">
            <div class="flex items-center justify-between border-b border-stone-700/80 pb-4">
                <div>
                    <h1 class="text-2xl font-black tracking-tight text-white">VANGUARD ARENA</h1>
                    <p class="text-xs text-stone-400 mt-0.5">20-Player Woodland Battle Royale • 200 Player HP / 100 Bot HP • 25 Head / 15 Body</p>
                </div>
                <button onclick="resumeGameplay()" class="px-4 py-2 rounded-xl bg-lime-500 hover:bg-lime-400 text-stone-950 font-black text-xs uppercase tracking-wider">
                    Resume Match
                </button>
            </div>

            <div class="grid grid-cols-1 md:grid-cols-2 gap-5 mt-5">
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
        const BOT_MAX_HEALTH = 100;
        const HEADSHOT_DAMAGE = 25;
        const BODYSHOT_DAMAGE = 15;
        const BOT_COUNT = 19; // 19 bots + 1 player = 20 combatants (no respawns)
        const MEDKIT_DURATION = 3.0; // 3.0s channeling timer

        const BOT_CALLSIGNS = [
            "Alpha-1", "Bravo-6", "Viper-2", "Ghost-9", "Reaper-4",
            "Kodiak-7", "Falcon-3", "Nomad-8", "Specter-5", "Titan-11",
            "Jackal-12", "Cobra-14", "Raven-15", "Wolf-16", "Outlaw-17",
            "Striker-18", "Shadow-19", "Hawk-20", "Phantom-21"
        ];

        // =========================================================================
        // 2. AUDIO ENGINE (TONE.JS PROCEDURAL SYNTHS)
        // =========================================================================
        let audioReady = false;
        let rifleSynth, shotgunSynth, pistolSynth, knifeSynth, healSynth, lootSynth;

        function initAudio() {
            if (audioReady || typeof Tone === "undefined") return;
            try {
                Tone.start();
                rifleSynth = new Tone.NoiseSynth({
                    noise: { type: "white" },
                    envelope: { attack: 0.002, decay: 0.08, sustain: 0 }
                }).toDestination();
                rifleSynth.volume.value = -10;

                shotgunSynth = new Tone.NoiseSynth({
                    noise: { type: "pink" },
                    envelope: { attack: 0.005, decay: 0.22, sustain: 0 }
                }).toDestination();
                shotgunSynth.volume.value = -6;

                pistolSynth = new Tone.MembraneSynth({
                    pitchDecay: 0.02,
                    octaves: 3,
                    envelope: { attack: 0.001, decay: 0.09, sustain: 0 }
                }).toDestination();
                pistolSynth.volume.value = -8;

                knifeSynth = new Tone.NoiseSynth({
                    noise: { type: "brown" },
                    envelope: { attack: 0.01, decay: 0.07, sustain: 0 }
                }).toDestination();
                knifeSynth.volume.value = -12;

                healSynth = new Tone.Synth({
                    oscillator: { type: "sine" },
                    envelope: { attack: 0.05, decay: 0.35, sustain: 0.1, release: 0.4 }
                }).toDestination();
                healSynth.volume.value = -10;

                lootSynth = new Tone.PolySynth(Tone.Synth).toDestination();
                lootSynth.volume.value = -10;

                audioReady = true;
            } catch (e) {
                console.warn("Audio init skipped:", e);
            }
        }

        function playSound(type) {
            if (!audioReady) return;
            try {
                const now = Tone.now();
                if (type === "rifle") rifleSynth.triggerAttackRelease("16n", now);
                else if (type === "shotgun") shotgunSynth.triggerAttackRelease("8n", now);
                else if (type === "pistol") pistolSynth.triggerAttackRelease("G2", "16n", now);
                else if (type === "knife") knifeSynth.triggerAttackRelease("32n", now);
                else if (type === "heal") healSynth.triggerAttackRelease("C5", "4n", now);
                else if (type === "loot") lootSynth.triggerAttackRelease(["E5", "G5", "B5"], "16n", now);
            } catch (e) {}
        }

        // =========================================================================
        // 3. THREE.JS SCENE, CAMERA, LIGHTING & MAP STATE
        // =========================================================================
        let scene, camera, renderer, weaponGroup;
        let viewModels = [];
        let colliders = [];
        let bots = [];
        let chests = [];
        let tracers = [];

        const player = {
            name: "You (Vanguard)",
            pos: new THREE.Vector3(0, 1.75, 0),
            vel: new THREE.Vector3(0, 0, 0),
            yaw: 0,
            pitch: 0,
            health: PLAYER_MAX_HEALTH,
            alive: true,
            kills: 0,
            onGround: true,
            activeSlot: 2, // Start with M9 Pistol equipped
            reloading: false,
            reloadTimer: 0,
            fireTimer: 0,
            medkitTimer: 0,
            isHealing: false,
            recoilOffset: 0,
            bobPhase: 0,
            weapons: [
                { id: "m4a1", name: "M4A1 CARBINE", unlocked: false, mag: 30, maxMag: 30, reserve: 120, fireRate: 0.11, auto: true, range: 160, pellets: 1, spread: 0.015 },
                { id: "m870", name: "M870 BREACHER", unlocked: false, mag: 6, maxMag: 6, reserve: 36, fireRate: 0.72, auto: false, range: 42, pellets: 6, spread: 0.085 },
                { id: "m9", name: "M9 TACTICAL PISTOL", unlocked: true, mag: 15, maxMag: 15, reserve: 90, fireRate: 0.22, auto: false, range: 110, pellets: 1, spread: 0.02 },
                { id: "knife", name: "COMBAT KNIFE", unlocked: true, mag: 1, maxMag: 1, reserve: 0, fireRate: 0.42, auto: false, range: 4.2, pellets: 1, spread: 0 },
                { id: "medkit", name: "FORTNITE MED KIT", unlocked: false, mag: 0, maxMag: 3, reserve: 0, fireRate: 0.5, auto: false, range: 0, pellets: 0, spread: 0 }
            ]
        };

        const keys = {};
        let isMouseDown = false;
        let isRightDragging = false;
        let lastMouseX = null;
        let lastMouseY = null;
        let isPaused = false;
        let isMatchOver = false;
        let isAutoSim = false;
        let simSpeed = 1;
        let pointerLocked = false;
        let freeAimFallback = false;

        function initThree() {
            const container = document.getElementById("game-canvas-container");
            scene = new THREE.Scene();
            scene.background = new THREE.Color(0x88b396);
            scene.fog = new THREE.FogExp2(0x88b396, 0.0042);

            camera = new THREE.PerspectiveCamera(75, window.innerWidth / window.innerHeight, 0.1, 550);
            camera.rotation.order = "YXZ";
            scene.add(camera);

            renderer = new THREE.WebGLRenderer({ antialias: true });
            renderer.setSize(window.innerWidth, window.innerHeight);
            renderer.setPixelRatio(Math.min(window.devicePixelRatio, 2));
            renderer.shadowMap.enabled = true;
            renderer.shadowMap.type = THREE.PCFSoftShadowMap;
            container.appendChild(renderer.domElement);

            // Lighting
            const hemiLight = new THREE.HemisphereLight(0xdff5e3, 0x3b4d36, 0.85);
            scene.add(hemiLight);

            const dirLight = new THREE.DirectionalLight(0xfff7e6, 1.15);
            dirLight.position.set(110, 180, 90);
            dirLight.castShadow = true;
            dirLight.shadow.mapSize.width = 2048;
            dirLight.shadow.mapSize.height = 2048;
            dirLight.shadow.camera.near = 10;
            dirLight.shadow.camera.far = 480;
            const d = 210;
            dirLight.shadow.camera.left = -d;
            dirLight.shadow.camera.right = d;
            dirLight.shadow.camera.top = d;
            dirLight.shadow.camera.bottom = -d;
            scene.add(dirLight);

            buildFirstPersonViewModels();
            buildWoodlandArena();
            spawnSupplyChests();
            spawnAllBots();
            setupControls();

            window.addEventListener("resize", onWindowResize);
        }

        function onWindowResize() {
            if (!camera || !renderer) return;
            camera.aspect = window.innerWidth / window.innerHeight;
            camera.updateProjectionMatrix();
            renderer.setSize(window.innerWidth, window.innerHeight);
        }

        // =========================================================================
        // 4. FIRST-PERSON VIEW MODELS (M4A1, SHOTGUN, PISTOL, KNIFE, FORTNITE MEDKIT)
        // =========================================================================
        function buildFirstPersonViewModels() {
            weaponGroup = new THREE.Group();
            camera.add(weaponGroup);

            const darkMetal = new THREE.MeshStandardMaterial({ color: 0x26292e, metalness: 0.7, roughness: 0.35 });
            const olivePoly = new THREE.MeshStandardMaterial({ color: 0x4b5d44, roughness: 0.65 });
            const woodMat = new THREE.MeshStandardMaterial({ color: 0x6b4423, roughness: 0.75 });
            const bladeMat = new THREE.MeshStandardMaterial({ color: 0xd4d8de, metalness: 0.9, roughness: 0.2 });

            // [0] M4A1 Assault Rifle
            const m4 = new THREE.Group();
            const m4Body = new THREE.Mesh(new THREE.BoxGeometry(0.08, 0.11, 0.52), darkMetal);
            const m4Barrel = new THREE.Mesh(new THREE.CylinderGeometry(0.015, 0.015, 0.34, 8), darkMetal);
            m4Barrel.rotation.x = Math.PI / 2;
            m4Barrel.position.set(0, 0.02, -0.38);
            const m4Handguard = new THREE.Mesh(new THREE.BoxGeometry(0.075, 0.08, 0.28), olivePoly);
            m4Handguard.position.set(0, 0.01, -0.24);
            const m4Mag = new THREE.Mesh(new THREE.BoxGeometry(0.055, 0.18, 0.08), darkMetal);
            m4Mag.position.set(0, -0.11, -0.05);
            m4Mag.rotation.x = 0.2;
            m4.add(m4Body, m4Barrel, m4Handguard, m4Mag);
            m4.position.set(0.26, -0.23, -0.45);
            weaponGroup.add(m4);

            // [1] M870 Shotgun
            const sg = new THREE.Group();
            const sgBody = new THREE.Mesh(new THREE.BoxGeometry(0.085, 0.1, 0.48), darkMetal);
            const sgBarrel = new THREE.Mesh(new THREE.CylinderGeometry(0.022, 0.022, 0.46, 8), darkMetal);
            sgBarrel.rotation.x = Math.PI / 2;
            sgBarrel.position.set(0, 0.03, -0.36);
            const sgPump = new THREE.Mesh(new THREE.BoxGeometry(0.09, 0.08, 0.22), woodMat);
            sgPump.position.set(0, -0.01, -0.32);
            sg.add(sgBody, sgBarrel, sgPump);
            sg.position.set(0.26, -0.24, -0.45);
            sg.visible = false;
            weaponGroup.add(sg);

            // [2] M9 Pistol
            const pistol = new THREE.Group();
            const pSlide = new THREE.Mesh(new THREE.BoxGeometry(0.055, 0.075, 0.26), darkMetal);
            const pGrip = new THREE.Mesh(new THREE.BoxGeometry(0.05, 0.14, 0.07), olivePoly);
            pGrip.position.set(0, -0.08, 0.07);
            pGrip.rotation.x = -0.25;
            pistol.add(pSlide, pGrip);
            pistol.position.set(0.24, -0.22, -0.42);
            pistol.visible = true;
            weaponGroup.add(pistol);

            // [3] Combat Knife
            const knife = new THREE.Group();
            const kHandle = new THREE.Mesh(new THREE.CylinderGeometry(0.022, 0.025, 0.15, 8), olivePoly);
            kHandle.rotation.x = Math.PI / 2.5;
            const kBlade = new THREE.Mesh(new THREE.BoxGeometry(0.012, 0.04, 0.22), bladeMat);
            kBlade.position.set(0, 0.04, -0.15);
            kBlade.rotation.x = -0.3;
            knife.add(kHandle, kBlade);
            knife.position.set(0.25, -0.24, -0.38);
            knife.visible = false;
            weaponGroup.add(knife);

            // [4] Fortnite-Style Red & White Hard-Shell Med Kit
            const medkit = new THREE.Group();
            const redShellMat = new THREE.MeshStandardMaterial({ color: 0xdc2626, roughness: 0.35, metalness: 0.1 });
            const whiteCrossMat = new THREE.MeshStandardMaterial({ color: 0xffffff, roughness: 0.25 });
            const latchMat = new THREE.MeshStandardMaterial({ color: 0x1f2937, roughness: 0.5 });

            const box = new THREE.Mesh(new THREE.BoxGeometry(0.32, 0.24, 0.14), redShellMat);
            // White medical cross on front & back
            const crossV = new THREE.Mesh(new THREE.BoxGeometry(0.045, 0.13, 0.148), whiteCrossMat);
            const crossH = new THREE.Mesh(new THREE.BoxGeometry(0.13, 0.045, 0.148), whiteCrossMat);
            // Top carry handle
            const handleTop = new THREE.Mesh(new THREE.BoxGeometry(0.14, 0.025, 0.04), latchMat);
            handleTop.position.set(0, 0.145, 0);
            const handleL = new THREE.Mesh(new THREE.BoxGeometry(0.025, 0.035, 0.04), latchMat);
            handleL.position.set(-0.058, 0.13, 0);
            const handleR = new THREE.Mesh(new THREE.BoxGeometry(0.025, 0.035, 0.04), latchMat);
            handleR.position.set(0.058, 0.13, 0);
            // Front latches
            const latch1 = new THREE.Mesh(new THREE.BoxGeometry(0.03, 0.045, 0.15), latchMat);
            latch1.position.set(-0.09, 0.09, 0);
            const latch2 = new THREE.Mesh(new THREE.BoxGeometry(0.03, 0.045, 0.15), latchMat);
            latch2.position.set(0.09, 0.09, 0);

            medkit.add(box, crossV, crossH, handleTop, handleL, handleR, latch1, latch2);
            medkit.position.set(0.24, -0.22, -0.45);
            medkit.rotation.y = -0.35;
            medkit.visible = false;
            weaponGroup.add(medkit);

            viewModels = [m4, sg, pistol, knife, medkit];
        }

        // =========================================================================
        // 5. 390x390 WOODLAND BATTLE ROYALE ARENA & COLLIDERS
        // =========================================================================
        function addStaticBox(w, h, d, x, y, z, mat, addCollider = true) {
            const mesh = new THREE.Mesh(new THREE.BoxGeometry(w, h, d), mat);
            mesh.position.set(x, y, z);
            mesh.castShadow = true;
            mesh.receiveShadow = true;
            scene.add(mesh);
            if (addCollider) {
                colliders.push({
                    minX: x - w / 2 - 0.55,
                    maxX: x + w / 2 + 0.55,
                    minZ: z - d / 2 - 0.55,
                    maxZ: z + d / 2 + 0.55
                });
            }
            return mesh;
        }

        function buildWoodlandArena() {
            colliders = [];
            const groundMat = new THREE.MeshStandardMaterial({ color: 0x3b5932, roughness: 0.92 });
            const ground = new THREE.Mesh(new THREE.PlaneGeometry(ARENA_HALF_SIZE * 2 + 20, ARENA_HALF_SIZE * 2 + 20), groundMat);
            ground.rotation.x = -Math.PI / 2;
            ground.receiveShadow = true;
            scene.add(ground);

            // Perimeter concrete walls
            const wallMat = new THREE.MeshStandardMaterial({ color: 0x4b5548, roughness: 0.85 });
            const S = ARENA_HALF_SIZE;
            addStaticBox(S * 2 + 4, 12, 3, 0, 6, -S, wallMat);
            addStaticBox(S * 2 + 4, 12, 3, 0, 6, S, wallMat);
            addStaticBox(3, 12, S * 2 + 4, -S, 6, 0, wallMat);
            addStaticBox(3, 12, S * 2 + 4, S, 6, 0, wallMat);

            // Central Command Outpost & 4 Quadrant Cabins
            const cabinWood = new THREE.MeshStandardMaterial({ color: 0x6b472b, roughness: 0.8 });
            const roofMat = new THREE.MeshStandardMaterial({ color: 0x2f3b2e, roughness: 0.7 });
            const outpostLocations = [
                { x: 0, z: -24 },
                { x: -85, z: -85 },
                { x: 85, z: -85 },
                { x: -85, z: 85 },
                { x: 85, z: 85 }
            ];
            outpostLocations.forEach(loc => {
                addStaticBox(14, 4.5, 0.8, loc.x, 2.25, loc.z - 6, cabinWood);
                addStaticBox(4.5, 4.5, 0.8, loc.x - 4.75, 2.25, loc.z + 6, cabinWood);
                addStaticBox(4.5, 4.5, 0.8, loc.x + 4.75, 2.25, loc.z + 6, cabinWood);
                addStaticBox(0.8, 4.5, 12.8, loc.x - 7, 2.25, loc.z, cabinWood);
                addStaticBox(0.8, 4.5, 12.8, loc.x + 7, 2.25, loc.z, cabinWood);
                addStaticBox(15.6, 0.6, 13.6, loc.x, 4.8, loc.z, roofMat, false);
            });

            // Pine Trees across the 390x390 map
            const trunkMat = new THREE.MeshStandardMaterial({ color: 0x4a321f, roughness: 0.9 });
            const foliageMat = new THREE.MeshStandardMaterial({ color: 0x1f4726, roughness: 0.85 });
            let seed = 42;
            function rand() {
                seed = (seed * 1664525 + 1013904223) % 4294967296;
                return seed / 4294967296;
            }

            for (let i = 0; i < 240; i++) {
                const tx = (rand() - 0.5) * (ARENA_HALF_SIZE * 1.88);
                const tz = (rand() - 0.5) * (ARENA_HALF_SIZE * 1.88);
                if (Math.hypot(tx, tz) < 14) continue; // Keep center spawn clear

                const scale = 0.85 + rand() * 0.75;
                const trunk = new THREE.Mesh(new THREE.CylinderGeometry(0.45 * scale, 0.65 * scale, 5 * scale, 7), trunkMat);
                trunk.position.set(tx, 2.5 * scale, tz);
                trunk.castShadow = true;
                scene.add(trunk);

                const c1 = new THREE.Mesh(new THREE.ConeGeometry(3.2 * scale, 6.5 * scale, 7), foliageMat);
                c1.position.set(tx, 6.5 * scale, tz);
                c1.castShadow = true;
                scene.add(c1);

                const c2 = new THREE.Mesh(new THREE.ConeGeometry(2.3 * scale, 5.2 * scale, 7), foliageMat);
                c2.position.set(tx, 9.8 * scale, tz);
                c2.castShadow = true;
                scene.add(c2);

                colliders.push({
                    minX: tx - 0.95 * scale,
                    maxX: tx + 0.95 * scale,
                    minZ: tz - 0.95 * scale,
                    maxZ: tz + 0.95 * scale
                });
            }

            // Cover Boulders & Sandbag Barricades
            const rockMat = new THREE.MeshStandardMaterial({ color: 0x646b66, roughness: 0.9 });
            for (let i = 0; i < 95; i++) {
                const rx = (rand() - 0.5) * (ARENA_HALF_SIZE * 1.85);
                const rz = (rand() - 0.5) * (ARENA_HALF_SIZE * 1.85);
                if (Math.hypot(rx, rz) < 12) continue;
                const w = 2.2 + rand() * 2.5;
                const h = 1.4 + rand() * 1.4;
                const d = 2.0 + rand() * 2.4;
                addStaticBox(w, h, d, rx, h / 2, rz, rockMat, true);
            }
        }

        // =========================================================================
        // 6. TACTICAL SUPPLY CHESTS (M4A1, SHOTGUN, MEDKITS & AMMO)
        // =========================================================================
        function spawnSupplyChests() {
            chests.forEach(c => scene.remove(c.group));
            chests = [];

            const chestCoords = [
                { x: 0, z: -24 }, { x: 8, z: 6 }, { x: -12, z: 10 },
                { x: -85, z: -85 }, { x: 85, z: -85 }, { x: -85, z: 85 }, { x: 85, z: 85 },
                { x: -45, z: 0 }, { x: 45, z: 0 }, { x: 0, z: -65 }, { x: 0, z: 65 },
                { x: -135, z: -40 }, { x: 135, z: -40 }, { x: -135, z: 45 }, { x: 135, z: 45 },
                { x: -50, z: -135 }, { x: 50, z: -135 }, { x: -50, z: 135 }, { x: 50, z: 135 },
                { x: -145, z: -145 }, { x: 145, z: -145 }, { x: -145, z: 145 }, { x: 145, z: 145 }
            ];

            const baseMat = new THREE.MeshStandardMaterial({ color: 0x78350f, roughness: 0.6 });
            const trimMat = new THREE.MeshStandardMaterial({ color: 0xf59e0b, metalness: 0.7, roughness: 0.25, emissive: 0x78350f, emissiveIntensity: 0.35 });
            const beaconMat = new THREE.MeshBasicMaterial({ color: 0xfbbf24, transparent: true, opacity: 0.22 });

            chestCoords.forEach((pt, idx) => {
                const group = new THREE.Group();
                group.position.set(pt.x, 0, pt.z);

                const body = new THREE.Mesh(new THREE.BoxGeometry(1.5, 0.75, 0.95), baseMat);
                body.position.y = 0.375;
                body.castShadow = true;

                const lid = new THREE.Mesh(new THREE.BoxGeometry(1.56, 0.28, 1.0), trimMat);
                lid.position.y = 0.88;
                lid.castShadow = true;

                const beacon = new THREE.Mesh(new THREE.CylinderGeometry(0.18, 0.45, 7.5, 10), beaconMat);
                beacon.position.y = 4.0;

                group.add(body, lid, beacon);
                scene.add(group);

                chests.push({
                    id: idx,
                    x: pt.x,
                    z: pt.z,
                    opened: false,
                    respawnTimer: 0,
                    group,
                    lid,
                    beacon
                });
            });
        }

        function openChest(chest, byBot = false) {
            if (chest.opened) return;
            chest.opened = true;
            chest.respawnTimer = 20;
            chest.lid.rotation.x = -0.9;
            chest.lid.position.z = -0.35;
            chest.beacon.visible = false;

            if (byBot) return;

            playSound("loot");
            const roll = Math.random();
            let rewardMsg = "";

            if (!player.weapons[0].unlocked) {
                player.weapons[0].unlocked = true;
                player.weapons[0].mag = 30;
                player.weapons[0].reserve += 90;
                switchWeapon(0);
                rewardMsg = "LOOTED: M4A1 CARBINE + 90 AMMO";
            } else if (!player.weapons[1].unlocked && roll < 0.55) {
                player.weapons[1].unlocked = true;
                player.weapons[1].mag = 6;
                player.weapons[1].reserve += 24;
                switchWeapon(1);
                rewardMsg = "LOOTED: M870 BREACHER SHOTGUN";
            } else {
                player.weapons[4].unlocked = true;
                player.weapons[4].mag = Math.min(3, player.weapons[4].mag + 1);
                player.weapons[0].reserve += 60;
                player.weapons[1].reserve += 12;
                player.weapons[2].reserve += 30;
                rewardMsg = "LOOTED: FORTNITE MED KIT + AMMO PACK";
            }

            addKillFeedEntry(rewardMsg, "text-amber-300");
            updateWeaponHUD();
        }

        // =========================================================================
        // 7. 19 ENEMY SOLDIERS (100 HP, NO RESPAWNS, REALISTIC ACCURACY)
        // =========================================================================
        function createSoldierMesh() {
            const group = new THREE.Group();
            const uniformMat = new THREE.MeshStandardMaterial({ color: 0x3f4e37, roughness: 0.75 });
            const vestMat = new THREE.MeshStandardMaterial({ color: 0x262925, roughness: 0.65 });
            const skinMat = new THREE.MeshStandardMaterial({ color: 0xd6a582, roughness: 0.6 });
            const helmetMat = new THREE.MeshStandardMaterial({ color: 0x2c3627, roughness: 0.55 });
            const gunMat = new THREE.MeshStandardMaterial({ color: 0x18181b, metalness: 0.7, roughness: 0.3 });

            const torso = new THREE.Mesh(new THREE.BoxGeometry(0.62, 0.85, 0.36), uniformMat);
            torso.position.y = 1.18;
            torso.castShadow = true;

            const vest = new THREE.Mesh(new THREE.BoxGeometry(0.66, 0.62, 0.4), vestMat);
            vest.position.y = 1.22;

            const head = new THREE.Mesh(new THREE.BoxGeometry(0.36, 0.38, 0.36), skinMat);
            head.position.y = 1.82;
            head.castShadow = true;

            const helmet = new THREE.Mesh(new THREE.BoxGeometry(0.41, 0.22, 0.41), helmetMat);
            helmet.position.y = 1.94;

            const legL = new THREE.Mesh(new THREE.BoxGeometry(0.24, 0.76, 0.26), uniformMat);
            legL.position.set(-0.17, 0.38, 0);
            const legR = new THREE.Mesh(new THREE.BoxGeometry(0.24, 0.76, 0.26), uniformMat);
            legR.position.set(0.17, 0.38, 0);

            const gun = new THREE.Mesh(new THREE.BoxGeometry(0.08, 0.12, 0.68), gunMat);
            gun.position.set(0.34, 1.25, 0.28);

            group.add(torso, vest, head, helmet, legL, legR, gun);
            return { group, head, torso, legL, legR };
        }

        function spawnAllBots() {
            bots.forEach(b => scene.remove(b.mesh));
            bots = [];

            for (let i = 0; i < BOT_COUNT; i++) {
                const angle = (i / BOT_COUNT) * Math.PI * 2;
                const radius = 48 + (i % 4) * 34;
                const bx = Math.cos(angle) * radius;
                const bz = Math.sin(angle) * radius;

                const rig = createSoldierMesh();
                rig.group.position.set(bx, 0, bz);
                scene.add(rig.group);

                bots.push({
                    id: i,
                    name: BOT_CALLSIGNS[i] || `Bot-${i + 1}`,
                    pos: new THREE.Vector3(bx, 0, bz),
                    yaw: Math.random() * Math.PI * 2,
                    health: BOT_MAX_HEALTH,
                    alive: true,
                    kills: 0,
                    hasRifle: i % 2 === 0,
                    fireCooldown: 1.0 + Math.random() * 1.5,
                    wanderTarget: new THREE.Vector3(bx, 0, bz),
                    wanderTimer: 0,
                    stride: Math.random() * 6,
                    mesh: rig.group,
                    headMesh: rig.head,
                    torsoMesh: rig.torso,
                    legL: rig.legL,
                    legR: rig.legR
                });
            }
            updateAliveAndRosterHUD();
        }

        // =========================================================================
        // 8. WEAPONS, SHOOTING, MEDKIT (3.0s) & DAMAGE LOGIC
        // =========================================================================
        function switchWeapon(slotIdx) {
            if (slotIdx < 0 || slotIdx > 4) return;
            const w = player.weapons[slotIdx];
            if (!w.unlocked && !(slotIdx === 4 && w.mag > 0)) return;

            player.activeSlot = slotIdx;
            player.reloading = false;
            player.isHealing = false;
            player.medkitTimer = 0;
            document.getElementById("medkit-progress-wrap").classList.add("hidden");

            viewModels.forEach((vm, idx) => {
                vm.visible = idx === slotIdx;
            });
            updateWeaponHUD();
        }

        function startMedkitHeal() {
            const med = player.weapons[4];
            if (!player.alive || player.isHealing || med.mag <= 0 || player.health >= PLAYER_MAX_HEALTH) return;
            if (player.activeSlot !== 4) {
                switchWeapon(4);
            }
            player.isHealing = true;
            player.medkitTimer = MEDKIT_DURATION;
            document.getElementById("medkit-progress-wrap").classList.remove("hidden");
            playSound("heal");
        }

        function spawnBulletTracer(startPos, endPos, colorHex = 0xfef08a) {
            const dist = startPos.distanceTo(endPos);
            if (dist < 0.2) return;
            const geo = new THREE.CylinderGeometry(0.022, 0.022, dist, 5);
            geo.rotateX(Math.PI / 2);
            const mat = new THREE.MeshBasicMaterial({ color: colorHex, transparent: true, opacity: 0.85 });
            const line = new THREE.Mesh(geo, mat);
            line.position.copy(startPos).lerp(endPos, 0.5);
            line.lookAt(endPos);
            scene.add(line);
            tracers.push({ mesh: line, life: 0.09 });
        }

        function triggerHitmarker(isHeadshot) {
            const hm = document.getElementById("hitmarker");
            hm.style.opacity = "1";
            hm.querySelectorAll("div").forEach(bar => {
                bar.style.backgroundColor = isHeadshot ? "#facc15" : "#ef4444";
            });
            setTimeout(() => { hm.style.opacity = "0"; }, 110);
        }

        function fireActiveWeapon() {
            if (!player.alive || isPaused || isMatchOver) return;
            const w = player.weapons[player.activeSlot];

            // Slot 4 is Fortnite Medkit
            if (player.activeSlot === 4) {
                startMedkitHeal();
                return;
            }

            if (player.reloading || player.fireTimer > 0) return;

            if (w.id !== "knife" && w.mag <= 0) {
                reloadWeapon();
                return;
            }

            player.fireTimer = w.fireRate;
            player.recoilOffset = w.id === "m870" ? 0.14 : (w.id === "knife" ? 0.18 : 0.05);

            if (w.id === "m4a1") playSound("rifle");
            else if (w.id === "m870") playSound("shotgun");
            else if (w.id === "m9") playSound("pistol");
            else playSound("knife");

            if (w.id !== "knife") {
                w.mag--;
                updateWeaponHUD();
            }

            const forward = new THREE.Vector3(0, 0, -1).applyQuaternion(camera.quaternion);
            const muzzleStart = camera.position.clone()
                .addScaledVector(forward, 0.65)
                .add(new THREE.Vector3(0, -0.14, 0));

            const targetMeshes = [];
            bots.forEach(b => {
                if (b.alive) {
                    targetMeshes.push(b.headMesh, b.torsoMesh);
                }
            });

            for (let p = 0; p < w.pellets; p++) {
                const dir = forward.clone();
                if (w.spread > 0) {
                    dir.x += (Math.random() - 0.5) * w.spread;
                    dir.y += (Math.random() - 0.5) * w.spread;
                    dir.z += (Math.random() - 0.5) * w.spread;
                    dir.normalize();
                }

                const raycaster = new THREE.Raycaster(camera.position, dir, 0.2, w.range);
                const hits = raycaster.intersectObjects(targetMeshes, false);

                if (hits.length > 0) {
                    const hit = hits[0];
                    if (w.id !== "knife") {
                        spawnBulletTracer(muzzleStart, hit.point, 0xfef08a);
                    }
                    const hitBot = bots.find(b => b.alive && (b.headMesh === hit.object || b.torsoMesh === hit.object));
                    if (hitBot) {
                        const isHead = hit.object === hitBot.headMesh;
                        const dmg = isHead ? HEADSHOT_DAMAGE : BODYSHOT_DAMAGE;
                        triggerHitmarker(isHead);
                        applyDamageToBot(hitBot, dmg, player.name, isHead);
                        if (!hitBot.alive) {
                            player.kills++;
                            document.getElementById("hud-kills-val").textContent = player.kills;
                        }
                    }
                } else if (w.id !== "knife") {
                    const missEnd = camera.position.clone().addScaledVector(dir, Math.min(w.range, 85));
                    spawnBulletTracer(muzzleStart, missEnd, 0xfef08a);
                }
            }
        }

        function reloadWeapon() {
            const w = player.weapons[player.activeSlot];
            if (player.reloading || player.activeSlot >= 3 || w.mag >= w.maxMag || w.reserve <= 0) return;
            player.reloading = true;
            player.reloadTimer = 1.35;
            document.getElementById("hud-weapon-name").textContent = `${w.name} (RELOADING...)`;
        }

        function applyDamageToBot(bot, amount, attackerName, isHeadshot) {
            if (!bot.alive) return;
            bot.health = Math.max(0, bot.health - amount);
            if (bot.health <= 0) {
                bot.alive = false;
                bot.mesh.visible = false;
                addKillFeedEntry(`${attackerName} eliminated ${bot.name}${isHeadshot ? " [HEADSHOT]" : ""}`, "text-lime-300");
                updateAliveAndRosterHUD();
                checkMatchVictory();
            }
        }

        function applyDamageToPlayer(amount, attackerName, isHeadshot) {
            if (!player.alive || isMatchOver) return;
            player.health = Math.max(0, player.health - amount);
            updateHealthHUD();

            const flash = document.getElementById("damage-flash");
            flash.style.opacity = "0.85";
            setTimeout(() => { flash.style.opacity = "0"; }, 140);

            if (player.health <= 0) {
                player.alive = false;
                const aliveBots = bots.filter(b => b.alive).length;
                addKillFeedEntry(`${attackerName} eliminated ${player.name}${isHeadshot ? " [HEADSHOT]" : ""}`, "text-red-400");
                updateAliveAndRosterHUD();
                showMatchEnd(false, aliveBots + 1);
            }
        }

        function checkMatchVictory() {
            if (isMatchOver) return;
            const aliveBots = bots.filter(b => b.alive).length;
            if (player.alive && aliveBots === 0) {
                showMatchEnd(true, 1);
            }
        }

        function showMatchEnd(isVictory, placement) {
            isMatchOver = true;
            exitPointerLockSafely();
            const overlay = document.getElementById("match-end-overlay");
            overlay.classList.remove("hidden");
            overlay.classList.add("flex");

            document.getElementById("end-banner-tag").textContent = isVictory ? "VICTORY ROYALE" : "ELIMINATED";
            document.getElementById("end-banner-title").textContent = isVictory ? "LAST SOLDIER STANDING!" : "YOU WERE ELIMINATED";
            document.getElementById("end-banner-sub").textContent = isVictory
                ? "You survived the 20-player woodland Battle Royale."
                : `You placed #${placement} out of 20 combatants.`;
            document.getElementById("end-stat-kills").textContent = player.kills;
            document.getElementById("end-stat-place").textContent = `#${placement} / 20`;
        }

        function resetSoloMatch() {
            isMatchOver = false;
            isPaused = false;
            document.getElementById("match-end-overlay").classList.add("hidden");
            document.getElementById("match-end-overlay").classList.remove("flex");
            document.getElementById("pause-menu-overlay").classList.add("hidden");
            document.getElementById("pause-menu-overlay").classList.remove("flex");
            document.getElementById("kill-feed").innerHTML = "";

            player.pos.set(0, 1.75, 0);
            player.vel.set(0, 0, 0);
            player.yaw = 0;
            player.pitch = 0;
            player.health = PLAYER_MAX_HEALTH;
            player.alive = true;
            player.kills = 0;
            player.isHealing = false;
            player.medkitTimer = 0;
            document.getElementById("medkit-progress-wrap").classList.add("hidden");

            player.weapons[0].unlocked = false; player.weapons[0].mag = 30; player.weapons[0].reserve = 120;
            player.weapons[1].unlocked = false; player.weapons[1].mag = 6; player.weapons[1].reserve = 36;
            player.weapons[2].unlocked = true;  player.weapons[2].mag = 15; player.weapons[2].reserve = 90;
            player.weapons[3].unlocked = true;
            player.weapons[4].unlocked = false; player.weapons[4].mag = 0;
            switchWeapon(2);

            document.getElementById("hud-kills-val").textContent = "0";
            updateHealthHUD();
            spawnSupplyChests();
            spawnAllBots();
            requestCanvasLockSafely();
        }

        // =========================================================================
        // 9. PLAYER MOVEMENT, COLLISION & AUTO-BOT SIMULATION AI ([M])
        // =========================================================================
        function resolveArenaCollisions(pos) {
            const limit = ARENA_HALF_SIZE - 2.2;
            pos.x = Math.max(-limit, Math.min(limit, pos.x));
            pos.z = Math.max(-limit, Math.min(limit, pos.z));

            for (let i = 0; i < colliders.length; i++) {
                const c = colliders[i];
                if (pos.x > c.minX && pos.x < c.maxX && pos.z > c.minZ && pos.z < c.maxZ) {
                    const dLeft = Math.abs(pos.x - c.minX);
                    const dRight = Math.abs(c.maxX - pos.x);
                    const dTop = Math.abs(pos.z - c.minZ);
                    const dBottom = Math.abs(c.maxZ - pos.z);
                    const m = Math.min(dLeft, dRight, dTop, dBottom);
                    if (m === dLeft) pos.x = c.minX;
                    else if (m === dRight) pos.x = c.maxX;
                    else if (m === dTop) pos.z = c.minZ;
                    else pos.z = c.maxZ;
                }
            }
        }

        function updatePlayer(dt) {
            if (!player.alive || isPaused || isMatchOver) return;

            if (player.fireTimer > 0) player.fireTimer -= dt;
            if (player.recoilOffset > 0) player.recoilOffset = Math.max(0, player.recoilOffset - dt * 0.75);

            // Reload timer
            if (player.reloading) {
                player.reloadTimer -= dt;
                if (player.reloadTimer <= 0) {
                    const w = player.weapons[player.activeSlot];
                    const needed = w.maxMag - w.mag;
                    const take = Math.min(needed, w.reserve);
                    w.mag += take;
                    w.reserve -= take;
                    player.reloading = false;
                    updateWeaponHUD();
                }
            }

            // Medkit 3.0s channeling timer
            if (player.isHealing) {
                player.medkitTimer -= dt;
                const pct = Math.min(100, ((MEDKIT_DURATION - player.medkitTimer) / MEDKIT_DURATION) * 100);
                document.getElementById("medkit-progress-bar").style.width = `${pct}%`;
                document.getElementById("medkit-timer-text").textContent = `${Math.max(0, player.medkitTimer).toFixed(1)}s`;

                if (player.medkitTimer <= 0) {
                    player.isHealing = false;
                    player.health = PLAYER_MAX_HEALTH;
                    player.weapons[4].mag = Math.max(0, player.weapons[4].mag - 1);
                    if (player.weapons[4].mag === 0) player.weapons[4].unlocked = false;
                    document.getElementById("medkit-progress-wrap").classList.add("hidden");
                    updateHealthHUD();
                    switchWeapon(player.weapons[0].unlocked ? 0 : 2);
                    addKillFeedEntry("MED KIT APPLIED: +200 FULL HEALTH", "text-emerald-300");
                }
            }

            // Keyboard arrow camera look support
            const turnSpeed = 2.1 * dt;
            if (keys["ArrowLeft"]) player.yaw += turnSpeed;
            if (keys["ArrowRight"]) player.yaw -= turnSpeed;
            if (keys["ArrowUp"]) player.pitch = Math.min(1.4, player.pitch + turnSpeed * 0.75);
            if (keys["ArrowDown"]) player.pitch = Math.max(-1.4, player.pitch - turnSpeed * 0.75);

            // Movement vector (Manual or Auto-Sim AI)
            let moveForward = 0;
            let moveRight = 0;
            let sprint = keys["ShiftLeft"] || keys["ShiftRight"];

            if (isAutoSim) {
                runAutoSimPilot(dt);
            } else {
                if (keys["KeyW"]) moveForward += 1;
                if (keys["KeyS"]) moveForward -= 1;
                if (keys["KeyD"]) moveRight += 1;
                if (keys["KeyA"]) moveRight -= 1;

                const speed = sprint ? 14.5 : 9.8;
                const sinY = Math.sin(player.yaw);
                const cosY = Math.cos(player.yaw);
                const fwdX = -sinY;
                const fwdZ = -cosY;
                const rightX = cosY;
                const rightZ = -sinY;

                let vx = fwdX * moveForward + rightX * moveRight;
                let vz = fwdZ * moveForward + rightZ * moveRight;
                const len = Math.hypot(vx, vz);
                if (len > 0) {
                    vx = (vx / len) * speed;
                    vz = (vz / len) * speed;
                    player.bobPhase += dt * (sprint ? 13 : 9);
                }
                player.pos.x += vx * dt;
                player.pos.z += vz * dt;

                if (keys["Space"] && player.onGround) {
                    player.vel.y = 7.4;
                    player.onGround = false;
                }
            }

            // Gravity & Jump
            player.vel.y -= 19.5 * dt;
            player.pos.y += player.vel.y * dt;
            if (player.pos.y <= 1.75) {
                player.pos.y = 1.75;
                player.vel.y = 0;
                player.onGround = true;
            }

            resolveArenaCollisions(player.pos);

            // Update camera & weapon sway
            camera.position.copy(player.pos);
            camera.rotation.y = player.yaw;
            camera.rotation.x = player.pitch + player.recoilOffset * 0.35;

            const activeVM = viewModels[player.activeSlot];
            if (activeVM) {
                activeVM.position.z = -0.44 + player.recoilOffset;
                activeVM.position.y = -0.23 + Math.sin(player.bobPhase) * 0.008;
            }

            // Automatic fire if holding Left Click or F
            if ((isMouseDown || keys["KeyF"]) && player.weapons[player.activeSlot].auto) {
                fireActiveWeapon();
            }

            // Nearby Chest Detection
            let nearestChest = null;
            let nearestDist = 4.2;
            for (let i = 0; i < chests.length; i++) {
                const c = chests[i];
                if (!c.opened) {
                    const d = Math.hypot(player.pos.x - c.x, player.pos.z - c.z);
                    if (d < nearestDist) {
                        nearestDist = d;
                        nearestChest = c;
                    }
                }
            }

            const promptEl = document.getElementById("loot-prompt");
            if (nearestChest) {
                promptEl.classList.remove("hidden");
                if (keys["KeyE"] || isAutoSim) {
                    openChest(nearestChest, false);
                }
            } else {
                promptEl.classList.add("hidden");
            }
        }

        function runAutoSimPilot(dt) {
            // Heal if injured and holding Medkit
            if (player.health < 110 && player.weapons[4].mag > 0 && !player.isHealing) {
                startMedkitHeal();
                return;
            }

            // Find closest alive bot
            let targetBot = null;
            let bestDist = Infinity;
            bots.forEach(b => {
                if (b.alive) {
                    const d = Math.hypot(b.pos.x - player.pos.x, b.pos.z - player.pos.z);
                    if (d < bestDist) {
                        bestDist = d;
                        targetBot = b;
                    }
                }
            });

            if (targetBot) {
                const dx = targetBot.pos.x - player.pos.x;
                const dz = targetBot.pos.z - player.pos.z;
                const desiredYaw = Math.atan2(-dx, -dz);
                player.yaw = desiredYaw;
                player.pitch = 0;

                if (bestDist > 22) {
                    player.pos.x += (dx / bestDist) * 11.5 * dt;
                    player.pos.z += (dz / bestDist) * 11.5 * dt;
                    player.bobPhase += dt * 11;
                }
                if (bestDist < 75 && !player.isHealing) {
                    fireActiveWeapon();
                }
            }
        }

        // =========================================================================
        // 10. BOT AI UPDATE (100 HP, NO RESPAWNS, REALISTIC ACCURACY)
        // =========================================================================
        function updateBots(dt) {
            if (isPaused || isMatchOver) return;

            for (let i = 0; i < bots.length; i++) {
                const bot = bots[i];
                if (!bot.alive) continue;

                // Find nearest target (Player or another Bot)
                let targetPos = null;
                let targetIsPlayer = false;
                let targetBot = null;
                let bestDist = 85;

                if (player.alive) {
                    const dp = Math.hypot(player.pos.x - bot.pos.x, player.pos.z - bot.pos.z);
                    if (dp < bestDist) {
                        bestDist = dp;
                        targetPos = player.pos;
                        targetIsPlayer = true;
                    }
                }

                for (let j = 0; j < bots.length; j++) {
                    if (i === j || !bots[j].alive) continue;
                    const db = Math.hypot(bots[j].pos.x - bot.pos.x, bots[j].pos.z - bot.pos.z);
                    if (db < bestDist) {
                        bestDist = db;
                        targetPos = bots[j].pos;
                        targetIsPlayer = false;
                        targetBot = bots[j];
                    }
                }

                if (targetPos) {
                    const dx = targetPos.x - bot.pos.x;
                    const dz = targetPos.z - bot.pos.z;
                    bot.yaw = Math.atan2(dx, dz);

                    // Advance until within combat range
                    if (bestDist > 20) {
                        bot.pos.x += (dx / bestDist) * 6.8 * dt;
                        bot.pos.z += (dz / bestDist) * 6.8 * dt;
                        bot.stride += dt * 9;
                    }

                    // Bot shooting (realistic cooldowns & distance-scaled accuracy so bots miss often)
                    bot.fireCooldown -= dt;
                    if (bot.fireCooldown <= 0 && bestDist < 65) {
                        bot.fireCooldown = bot.hasRifle ? (0.85 + Math.random() * 0.75) : (1.25 + Math.random() * 0.85);
                        const start = new THREE.Vector3(bot.pos.x, 1.4, bot.pos.z);
                        const missSpread = Math.min(2.6, 0.45 + bestDist * 0.032);
                        const end = new THREE.Vector3(
                            targetPos.x + (Math.random() - 0.5) * missSpread,
                            1.35 + (Math.random() - 0.5) * (missSpread * 0.6),
                            targetPos.z + (Math.random() - 0.5) * missSpread
                        );
                        spawnBulletTracer(start, end, 0xf87171);

                        // Distance-scaled accuracy roll (18% close range down to 5% long range)
                        const hitChance = Math.max(0.05, 0.20 - (bestDist / 360));
                        const hitRoll = Math.random();
                        if (hitRoll < hitChance) {
                            const isHead = hitRoll < (hitChance * 0.15);
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
                    // Patrol woodland sector
                    bot.wanderTimer -= dt;
                    if (bot.wanderTimer <= 0) {
                        bot.wanderTimer = 3.0 + Math.random() * 4.0;
                        bot.wanderTarget.set(
                            (Math.random() - 0.5) * (ARENA_HALF_SIZE * 1.7),
                            0,
                            (Math.random() - 0.5) * (ARENA_HALF_SIZE * 1.7)
                        );
                    }
                    const wx = bot.wanderTarget.x - bot.pos.x;
                    const wz = bot.wanderTarget.z - bot.pos.z;
                    const wd = Math.hypot(wx, wz);
                    if (wd > 2) {
                        bot.yaw = Math.atan2(wx, wz);
                        bot.pos.x += (wx / wd) * 5.2 * dt;
                        bot.pos.z += (wz / wd) * 5.2 * dt;
                        bot.stride += dt * 7;
                    }
                }

                resolveArenaCollisions(bot.pos);
                bot.mesh.position.copy(bot.pos);
                bot.mesh.rotation.y = bot.yaw;
                bot.legL.rotation.x = Math.sin(bot.stride) * 0.45;
                bot.legR.rotation.x = -Math.sin(bot.stride) * 0.45;
            }
        }

        function updateChestsAndTracers(dt) {
            for (let i = 0; i < chests.length; i++) {
                const c = chests[i];
                if (c.opened) {
                    c.respawnTimer -= dt;
                    if (c.respawnTimer <= 0) {
                        c.opened = false;
                        c.lid.rotation.x = 0;
                        c.lid.position.z = 0;
                        c.beacon.visible = true;
                    }
                }
            }

            for (let i = tracers.length - 1; i >= 0; i--) {
                tracers[i].life -= dt;
                if (tracers[i].life <= 0) {
                    scene.remove(tracers[i].mesh);
                    tracers.splice(i, 1);
                }
            }
        }

        // =========================================================================
        // 11. TACTICAL RADAR MINIMAP & HUD UPDATES
        // =========================================================================
        function drawMinimap() {
            const canvas = document.getElementById("minimap-canvas");
            if (!canvas) return;
            const ctx = canvas.getContext("2d");
            const w = canvas.width;
            const h = canvas.height;
            const cx = w / 2;
            const cy = h / 2;
            const scale = (w / 2 - 8) / ARENA_HALF_SIZE;

            ctx.clearRect(0, 0, w, h);

            // Grid crosshairs
            ctx.strokeStyle = "rgba(163, 230, 53, 0.14)";
            ctx.lineWidth = 1;
            ctx.beginPath();
            ctx.moveTo(cx, 0); ctx.lineTo(cx, h);
            ctx.moveTo(0, cy); ctx.lineTo(w, cy);
            ctx.stroke();

            // Unopened Loot Chests (Gold dots)
            ctx.fillStyle = "#fbbf24";
            chests.forEach(c => {
                if (!c.opened) {
                    ctx.fillRect(cx + c.x * scale - 1.5, cy + c.z * scale - 1.5, 3, 3);
                }
            });

            // Alive Bots (Red dots)
            ctx.fillStyle = "#ef4444";
            bots.forEach(b => {
                if (b.alive) {
                    ctx.beginPath();
                    ctx.arc(cx + b.pos.x * scale, cy + b.pos.z * scale, 2.6, 0, Math.PI * 2);
                    ctx.fill();
                }
            });

            // Player Position & Look Direction (Lime green)
            if (player.alive) {
                const px = cx + player.pos.x * scale;
                const pz = cy + player.pos.z * scale;
                ctx.strokeStyle = "rgba(163, 230, 53, 0.55)";
                ctx.lineWidth = 1.5;
                ctx.beginPath();
                ctx.moveTo(px, pz);
                ctx.lineTo(px - Math.sin(player.yaw) * 14, pz - Math.cos(player.yaw) * 14);
                ctx.stroke();

                ctx.fillStyle = "#a3e635";
                ctx.beginPath();
                ctx.arc(px, pz, 3.5, 0, Math.PI * 2);
                ctx.fill();
            }
        }

        function updateHealthHUD() {
            document.getElementById("hud-health-val").textContent = Math.ceil(player.health);
            const pct = Math.max(0, (player.health / PLAYER_MAX_HEALTH) * 100);
            const bar = document.getElementById("hud-health-bar");
            bar.style.width = `${pct}%`;
            bar.className = pct > 40
                ? "h-full bg-gradient-to-r from-emerald-500 to-lime-400 transition-all duration-150"
                : "h-full bg-gradient-to-r from-red-600 to-amber-500 transition-all duration-150";
        }

        function updateWeaponHUD() {
            const w = player.weapons[player.activeSlot];
            document.getElementById("hud-weapon-name").textContent = w.name;
            if (w.id === "knife") {
                document.getElementById("hud-ammo-mag").textContent = "∞";
                document.getElementById("hud-ammo-res").textContent = "";
            } else if (w.id === "medkit") {
                document.getElementById("hud-ammo-mag").textContent = w.mag;
                document.getElementById("hud-ammo-res").textContent = "KITS";
            } else {
                document.getElementById("hud-ammo-mag").textContent = w.mag;
                document.getElementById("hud-ammo-res").textContent = `/ ${w.reserve}`;
            }

            document.getElementById("slot-4-label").textContent = `MED (${player.weapons[4].mag})`;

            for (let i = 0; i < 5; i++) {
                const el = document.getElementById(`slot-${i}`);
                const hasItem = player.weapons[i].unlocked || (i === 4 && player.weapons[4].mag > 0);
                el.style.opacity = hasItem ? "1" : "0.4";
                if (i === player.activeSlot) {
                    el.classList.remove("border", "border-stone-700");
                    el.classList.add("border-2", "border-lime-400");
                } else {
                    el.classList.remove("border-2", "border-lime-400");
                    el.classList.add("border", "border-stone-700");
                }
            }
        }

        function updateAliveAndRosterHUD() {
            const aliveBots = bots.filter(b => b.alive).length;
            const totalAlive = aliveBots + (player.alive ? 1 : 0);
            document.getElementById("hud-mode-badge").textContent = `Woodland BR (${totalAlive} Alive)`;
            document.getElementById("menu-alive-count").textContent = `${totalAlive} ALIVE`;

            const list = document.getElementById("roster-list");
            if (!list) return;
            const allCombatants = [
                { name: player.name, alive: player.alive, kills: player.kills, isPlayer: true },
                ...bots.map(b => ({ name: b.name, alive: b.alive, kills: b.kills, isPlayer: false }))
            ].sort((a, b) => (b.alive - a.alive) || (b.kills - a.kills));

            list.innerHTML = allCombatants.map(c => `
                <div class="flex items-center justify-between px-2.5 py-1.5 rounded-lg ${c.alive ? "bg-stone-800/80" : "bg-stone-950/60 opacity-50"}">
                    <span class="font-bold ${c.isPlayer ? "text-lime-300" : "text-stone-200"}">${c.name}</span>
                    <div class="flex items-center gap-2 mono text-[11px]">
                        <span class="text-amber-400">${c.kills} K</span>
                        <span class="px-1.5 py-0.5 rounded text-[10px] font-bold ${c.alive ? "bg-emerald-500/20 text-emerald-300" : "bg-red-500/20 text-red-400"}">
                            ${c.alive ? "ALIVE" : "OUT"}
                        </span>
                    </div>
                </div>
            `).join("");
        }

        function addKillFeedEntry(text, colorClass = "text-stone-200") {
            const feed = document.getElementById("kill-feed");
            const item = document.createElement("div");
            item.className = `glass-panel px-2.5 py-1 rounded-lg text-[11px] font-bold ${colorClass} shadow`;
            item.textContent = text;
            feed.prepend(item);
            while (feed.children.length > 5) {
                feed.removeChild(feed.lastChild);
            }
            setTimeout(() => {
                if (item.parentNode) item.parentNode.removeChild(item);
            }, 4500);
        }

        // =========================================================================
        // 12. INPUT, POINTER LOCK & SANDBOX FREE-AIM FALLBACK
        // =========================================================================
        function requestCanvasLockSafely() {
            const canvas = renderer && renderer.domElement;
            if (!canvas) return;
            try {
                if (document.pointerLockElement !== canvas && canvas.requestPointerLock) {
                    const promise = canvas.requestPointerLock();
                    if (promise && typeof promise.catch === "function") {
                        promise.catch(() => enableSandboxFreeAim());
                    }
                }
            } catch (e) {
                enableSandboxFreeAim();
            }
        }

        function exitPointerLockSafely() {
            try {
                if (document.pointerLockElement && document.exitPointerLock) {
                    document.exitPointerLock();
                }
            } catch (e) {}
        }

        function enableSandboxFreeAim() {
            freeAimFallback = true;
            document.getElementById("hud-aim-status").textContent = "Sandbox Free-Aim Active (Move mouse or drag to aim)";
        }

        function toggleSimMode() {
            isAutoSim = !isAutoSim;
            const btn = document.getElementById("btn-sim-toggle");
            btn.textContent = isAutoSim ? "[M] Mode: Auto-Sim ON" : "[M] Mode: Manual Play";
            btn.className = isAutoSim
                ? "px-2.5 py-1 rounded-lg bg-emerald-600 text-stone-950 font-extrabold text-[11px] transition"
                : "px-2.5 py-1 rounded-lg bg-stone-800 hover:bg-stone-700 border border-stone-600 text-[11px] font-bold text-emerald-300 transition";
        }

        function setSimSpeed(spd) {
            simSpeed = spd;
            [1, 2, 4].forEach(s => {
                const el = document.getElementById(`spd-${s}`);
                el.className = s === spd
                    ? "px-2 py-0.5 rounded bg-lime-600 text-stone-950 font-bold text-[10px]"
                    : "px-2 py-0.5 rounded bg-stone-800 text-stone-300 font-bold text-[10px]";
            });
        }

        function openPauseMenu() {
            isPaused = true;
            exitPointerLockSafely();
            updateAliveAndRosterHUD();
            const overlay = document.getElementById("pause-menu-overlay");
            overlay.classList.remove("hidden");
            overlay.classList.add("flex");
        }

        function resumeGameplay() {
            isPaused = false;
            const overlay = document.getElementById("pause-menu-overlay");
            overlay.classList.add("hidden");
            overlay.classList.remove("flex");
            requestCanvasLockSafely();
        }

        function setupControls() {
            const canvas = renderer.domElement;

            document.addEventListener("pointerlockchange", () => {
                pointerLocked = document.pointerLockElement === canvas;
                if (pointerLocked) {
                    freeAimFallback = false;
                    document.getElementById("hud-aim-status").textContent = "Cursor Locked • Move Mouse to Aim (ESC to unlock)";
                } else if (!freeAimFallback) {
                    document.getElementById("hud-aim-status").textContent = "Click screen to lock cursor (or move mouse / drag)";
                }
            });

            document.addEventListener("pointerlockerror", () => {
                enableSandboxFreeAim();
            });

            canvas.addEventListener("mousedown", (e) => {
                initAudio();
                if (isPaused || isMatchOver) return;
                if (!pointerLocked) {
                    requestCanvasLockSafely();
                }
                if (e.button === 0) {
                    isMouseDown = true;
                    fireActiveWeapon();
                } else if (e.button === 2) {
                    isRightDragging = true;
                }
            });

            window.addEventListener("mouseup", (e) => {
                if (e.button === 0) isMouseDown = false;
                if (e.button === 2) isRightDragging = false;
            });

            canvas.addEventListener("contextmenu", (e) => e.preventDefault());

            window.addEventListener("mousemove", (e) => {
                if (isPaused || isMatchOver || isAutoSim) return;

                if (pointerLocked) {
                    player.yaw -= e.movementX * 0.0022;
                    player.pitch = Math.max(-1.42, Math.min(1.42, player.pitch - e.movementY * 0.0022));
                } else if (freeAimFallback || isRightDragging) {
                    if (lastMouseX !== null && lastMouseY !== null) {
                        const dx = e.clientX - lastMouseX;
                        const dy = e.clientY - lastMouseY;
                        player.yaw -= dx * 0.0032;
                        player.pitch = Math.max(-1.42, Math.min(1.42, player.pitch - dy * 0.0032));
                    }
                    lastMouseX = e.clientX;
                    lastMouseY = e.clientY;
                }
            });

            window.addEventListener("keydown", (e) => {
                initAudio();
                keys[e.code] = true;

                if (e.code === "Digit1") switchWeapon(0);
                else if (e.code === "Digit2") switchWeapon(1);
                else if (e.code === "Digit3") switchWeapon(2);
                else if (e.code === "Digit4") switchWeapon(3);
                else if (e.code === "Digit5") switchWeapon(4);
                else if (e.code === "KeyH") startMedkitHeal();
                else if (e.code === "KeyR") reloadWeapon();
                else if (e.code === "KeyF") fireActiveWeapon();
                else if (e.code === "KeyM") toggleSimMode();
                else if (e.code === "KeyP") {
                    if (isPaused) resumeGameplay();
                    else openPauseMenu();
                }
            });

            window.addEventListener("keyup", (e) => {
                keys[e.code] = false;
            });

            window.addEventListener("wheel", (e) => {
                if (isPaused || isMatchOver) return;
                const dir = e.deltaY > 0 ? 1 : -1;
                for (let step = 1; step <= 5; step++) {
                    const next = (player.activeSlot + dir * step + 5) % 5;
                    const w = player.weapons[next];
                    if (w.unlocked || (next === 4 && w.mag > 0)) {
                        switchWeapon(next);
                        break;
                    }
                }
            });
        }

        // =========================================================================
        // 13. MAIN ANIMATION LOOP
        // =========================================================================
        let lastTime = performance.now();

        function animate(now) {
            requestAnimationFrame(animate);
            const rawDt = Math.min(0.05, (now - lastTime) / 1000);
            lastTime = now;

            const steps = isAutoSim ? simSpeed : 1;
            for (let s = 0; s < steps; s++) {
                updatePlayer(rawDt);
                updateBots(rawDt);
                updateChestsAndTracers(rawDt);
            }

            drawMinimap();
            if (renderer && scene && camera) {
                renderer.render(scene, camera);
            }
        }

        window.onload = function () {
            initThree();
            updateHealthHUD();
            updateWeaponHUD();
            requestAnimationFrame(animate);
        };
    </script>
</body>
</html>
