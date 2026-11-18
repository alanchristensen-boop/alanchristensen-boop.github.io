<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>DEAD SHOT — 3D Zombie Survival Arena</title>
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

    <!-- Pre-Game Survival Lobby Overlay -->
    <div id="main-lobby-overlay" class="fixed inset-0 z-50 flex items-center justify-center bg-gradient-to-br from-stone-950/95 via-emerald-950/90 to-stone-950/95 backdrop-blur-md p-4">
        <div class="glass-panel max-w-2xl w-full p-6 sm:p-8 rounded-3xl shadow-2xl border border-lime-400/35">
            <div class="flex flex-col sm:flex-row sm:items-center justify-between gap-3 border-b border-stone-700/80 pb-5">
                <div>
                    <div class="inline-flex items-center gap-2 px-2.5 py-0.5 rounded-full bg-red-500/20 border border-red-400/40 text-red-300 text-[10px] font-black uppercase tracking-widest mb-1.5">
                        <span class="w-2 h-2 rounded-full bg-red-500 animate-ping"></span>
                        <span id="lobby-top-badge">OUTBREAK ZONE • 19 INFECTED</span>
                    </div>
                    <h1 class="text-4xl sm:text-5xl font-black tracking-tighter text-white">DEAD <span class="text-lime-400">SHOT</span></h1>
                    <p id="lobby-subtitle" class="text-xs text-stone-400 mt-1">Zombie Survival • 100 HP • 25 Headshot / 15 Body</p>
                </div>
                <div class="bg-stone-900/90 px-4 py-2.5 rounded-2xl border border-stone-700/80 text-right">
                    <div class="text-[10px] uppercase tracking-wider text-stone-400 font-bold">Lobby Status</div>
                    <div class="text-sm font-black text-lime-400 mono">READY TO DEPLOY</div>
                </div>
            </div>

            <div class="grid grid-cols-1 md:grid-cols-2 gap-5 mt-6">
                <!-- Survivor Setup -->
                <div class="bg-stone-900/80 p-4 rounded-2xl border border-stone-700/70 space-y-4">
                    <div>
                        <label class="block text-[11px] font-extrabold uppercase tracking-wider text-lime-300 mb-1.5">Survivor Callsign</label>
                        <input id="lobby-player-name" type="text" maxlength="18" value="Survivor-01"
                               class="w-full px-3.5 py-2 rounded-xl bg-stone-950 border border-stone-700 focus:border-lime-400 focus:outline-none text-sm font-bold text-white" />
                    </div>
                    <div>
                        <label class="block text-[11px] font-extrabold uppercase tracking-wider text-purple-300 mb-1.5">Game Mode</label>
                        <div class="grid grid-cols-2 gap-2">
                            <button type="button" onclick="setLobbyGameMode('zombies')" id="mode-zombies"
                                    class="py-2 rounded-xl bg-lime-500 text-stone-950 font-black text-xs uppercase transition">
                                Zombie Survival
                            </button>
                            <button type="button" onclick="setLobbyGameMode('classic')" id="mode-classic"
                                    class="py-2 rounded-xl bg-stone-800 text-stone-300 hover:bg-stone-700 font-extrabold text-xs uppercase transition">
                                Classic Bots BR
                            </button>
                        </div>
                    </div>
                    <div>
                        <label class="block text-[11px] font-extrabold uppercase tracking-wider text-sky-300 mb-1.5">Deployment Map</label>
                        <div class="grid grid-cols-3 gap-2">
                            <button type="button" onclick="setLobbyMap('woodland')" id="map-woodland"
                                    class="py-2 rounded-xl bg-lime-500 text-stone-950 font-black text-xs uppercase transition">
                                Woodland
                            </button>
                            <button type="button" onclick="setLobbyMap('city')" id="map-city"
                                    class="py-2 rounded-xl bg-stone-800 text-stone-300 hover:bg-stone-700 font-extrabold text-xs uppercase transition">
                                City
                            </button>
                            <button type="button" onclick="setLobbyMap('desert')" id="map-desert"
                                    class="py-2 rounded-xl bg-stone-800 text-stone-300 hover:bg-stone-700 font-extrabold text-xs uppercase transition">
                                Desert
                            </button>
                        </div>
                    </div>
                    <div>
                        <label class="block text-[11px] font-extrabold uppercase tracking-wider text-amber-300 mb-1.5">Outbreak Threat Level</label>
                        <div class="grid grid-cols-3 gap-2">
                            <button type="button" onclick="setLobbyDifficulty('normal')" id="diff-normal"
                                    class="py-2 rounded-xl bg-lime-500 text-stone-950 font-black text-xs uppercase transition">
                                Normal
                            </button>
                            <button type="button" onclick="setLobbyDifficulty('hard')" id="diff-hard"
                                    class="py-2 rounded-xl bg-stone-800 text-stone-300 hover:bg-stone-700 font-extrabold text-xs uppercase transition">
                                Frenzied
                            </button>
                            <button type="button" onclick="setLobbyDifficulty('nightmare')" id="diff-nightmare"
                                    class="py-2 rounded-xl bg-stone-800 text-stone-300 hover:bg-stone-700 font-extrabold text-xs uppercase transition">
                                Nightmare
                            </button>
                        </div>
                    </div>
                    <div class="pt-1 text-[11px] text-stone-400 space-y-1">
                        <div>• Spawn with <span class="text-white font-bold">M9 Pistol</span> & <span class="text-white font-bold">Combat Knife</span></div>
                        <div>• Loot glowing <span class="text-amber-300 font-bold">Supply Chests [E]</span> for M4A1, Shotgun & Medkits</div>
                    </div>
                </div>

                <!-- Lobby Staging Roster -->
                <div class="bg-stone-900/80 p-4 rounded-2xl border border-stone-700/70 flex flex-col justify-between">
                    <div>
                        <div class="flex items-center justify-between mb-2.5">
                            <span class="text-[11px] font-extrabold uppercase tracking-wider text-stone-300">Sector Staging</span>
                            <span id="lobby-enemy-count" class="mono text-[11px] font-bold text-red-400">19 ZOMBIES DETECTED</span>
                        </div>
                        <div class="space-y-2 text-xs">
                            <div class="flex items-center justify-between px-3 py-2 rounded-xl bg-lime-500/15 border border-lime-400/30">
                                <span id="lobby-preview-name" class="font-extrabold text-lime-300">Survivor-01 (You)</span>
                                <span class="mono text-[10px] font-black px-2 py-0.5 rounded bg-lime-400 text-stone-950">READY</span>
                            </div>
                            <div class="flex items-center justify-between px-3 py-2 rounded-xl bg-stone-950/70 border border-stone-800">
                                <span id="lobby-enemy-label" class="font-bold text-red-300">Infected Horde (x19)</span>
                                <span class="mono text-[10px] font-bold text-red-400">100 HP EACH</span>
                            </div>
                            <div class="flex items-center justify-between px-3 py-2 rounded-xl bg-stone-950/70 border border-stone-800">
                                <span id="lobby-map-label" class="font-bold text-amber-300">Woodland Supply Chests</span>
                                <span class="mono text-[10px] font-bold text-amber-400">23 ACTIVE</span>
                            </div>
                        </div>
                    </div>

                    <button onclick="deployFromLobby()"
                            class="mt-5 w-full py-3.5 rounded-2xl bg-gradient-to-r from-lime-500 to-emerald-500 hover:from-lime-400 hover:to-emerald-400 text-stone-950 font-black text-sm uppercase tracking-wider shadow-lg transition">
                        Deploy Into Dead Shot
                    </button>
                </div>
            </div>
        </div>
    </div>

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
                    <span id="hud-mode-badge" class="text-xs font-extrabold uppercase tracking-wider text-lime-300">DEAD SHOT (19 Zombies)</span>
                </div>
                <span class="mono text-[11px] text-red-400 font-bold">ZOMBIE SURVIVAL</span>
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
                <span class="flex items-center gap-1"><span class="w-2 h-2 rounded-full bg-red-500 inline-block"></span>Zombies</span>
                <span class="flex items-center gap-1"><span class="w-2 h-2 rounded-full bg-amber-400 inline-block"></span>Loot</span>
            </div>
        </div>
    </div>

    <!-- Bottom-Left Health & Eliminations HUD -->
    <div class="fixed bottom-4 left-4 z-20 pointer-events-none">
        <div class="glass-panel p-3.5 rounded-2xl w-64 sm:w-72 shadow-xl">
            <div class="flex justify-between items-baseline mb-1.5">
                <div class="flex items-baseline gap-1.5">
                    <span id="hud-health-val" class="text-3xl font-black text-lime-400 mono">100</span>
                    <span class="text-xs font-bold text-stone-400">/ 100 HP</span>
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
                <span>[Q] Build • [H] Medkit</span>
            </div>
        </div>
    </div>

    <!-- Bottom-Right Weapon Loadout, Build Blueprints & Ammo HUD -->
    <div class="fixed bottom-4 right-4 z-20 flex flex-col items-end gap-2 pointer-events-none">
        <div class="flex items-center gap-2">
            <!-- Wood Materials Counter -->
            <div class="glass-panel px-3 py-2.5 rounded-2xl text-right shadow-xl border border-amber-500/40">
                <div class="text-[10px] font-extrabold uppercase tracking-wider text-amber-300">WOOD MATS</div>
                <div class="flex items-baseline justify-end gap-1 mt-0.5">
                    <span id="hud-wood-count" class="text-2xl font-black text-amber-400 mono">150</span>
                    <span class="text-[10px] font-bold text-stone-400 mono">/ 999</span>
                </div>
            </div>
            <!-- Active Weapon / Blueprint Readout -->
            <div class="glass-panel px-4 py-2.5 rounded-2xl text-right shadow-xl">
                <div id="hud-weapon-name" class="text-xs font-extrabold uppercase tracking-wider text-lime-300">M9 TACTICAL PISTOL</div>
                <div class="flex items-baseline justify-end gap-1.5 mt-0.5">
                    <span id="hud-ammo-mag" class="text-3xl font-black text-white mono">15</span>
                    <span id="hud-ammo-res" class="text-sm font-bold text-stone-400 mono">/ 90</span>
                </div>
            </div>
        </div>

        <!-- Fortnite Build Piece Selector Bar (Shown in Build Mode [Q]) -->
        <div id="build-bar" class="hidden items-center gap-1.5">
            <div id="bpiece-wall" class="glass-panel px-3 py-1.5 rounded-xl border-2 border-sky-400 bg-sky-950/60 text-center min-w-[66px]">
                <div class="text-[10px] font-bold text-sky-300 mono">[Z]</div>
                <div class="text-[11px] font-extrabold text-white">WALL</div>
            </div>
            <div id="bpiece-floor" class="glass-panel px-3 py-1.5 rounded-xl border border-stone-700 bg-sky-950/40 text-center min-w-[66px]">
                <div class="text-[10px] font-bold text-sky-300 mono">[X]</div>
                <div class="text-[11px] font-extrabold text-stone-200">FLOOR</div>
            </div>
            <div id="bpiece-ramp" class="glass-panel px-3 py-1.5 rounded-xl border border-stone-700 bg-sky-950/40 text-center min-w-[66px]">
                <div class="text-[10px] font-bold text-sky-300 mono">[C]</div>
                <div class="text-[11px] font-extrabold text-stone-200">RAMP</div>
            </div>
            <div class="glass-panel px-3 py-1.5 rounded-xl border border-lime-400/50 bg-lime-950/50 text-center">
                <div class="text-[10px] font-bold text-lime-300 mono">[Q]</div>
                <div class="text-[11px] font-extrabold text-lime-200">COMBAT</div>
            </div>
        </div>

        <!-- 5-Slot Inventory Bar -->
        <div id="weapon-bar" class="flex items-center gap-1.5">
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

    <!-- Victory / Elimination Match End Banner -->
    <div id="match-end-overlay" class="fixed inset-0 z-40 hidden items-center justify-center bg-black/75 backdrop-blur-md p-4">
        <div class="glass-panel max-w-md w-full p-7 rounded-3xl text-center shadow-2xl border border-lime-400/40">
            <div id="end-banner-tag" class="inline-block px-3 py-1 rounded-full text-xs font-black uppercase tracking-widest bg-lime-400/20 text-lime-300 mb-3">
                OUTBREAK CONTAINED
            </div>
            <h2 id="end-banner-title" class="text-3xl font-black text-white tracking-tight">SECTOR CLEARED!</h2>
            <p id="end-banner-sub" class="text-sm text-stone-300 mt-2">You eliminated all 19 zombies in Dead Shot.</p>
            <div class="mt-5 grid grid-cols-2 gap-3 bg-stone-900/80 p-3.5 rounded-2xl border border-stone-700/70">
                <div>
                    <div class="text-[11px] text-stone-400 uppercase font-bold">Zombies Slain</div>
                    <div id="end-stat-kills" class="text-2xl font-black text-amber-400 mono mt-0.5">0</div>
                </div>
                <div>
                    <div class="text-[11px] text-stone-400 uppercase font-bold">Final Status</div>
                    <div id="end-stat-place" class="text-2xl font-black text-lime-400 mono mt-0.5">SURVIVED</div>
                </div>
            </div>
            <div class="mt-6 flex flex-col sm:flex-row gap-2.5">
                <button onclick="resetSoloMatch()"
                        class="flex-1 py-3.5 rounded-2xl bg-gradient-to-r from-lime-500 to-emerald-500 hover:from-lime-400 hover:to-emerald-400 text-stone-950 font-black text-xs uppercase tracking-wider shadow-lg transition">
                    Play Again
                </button>
                <button onclick="returnToMainLobby()"
                        class="py-3.5 px-4 rounded-2xl bg-stone-800 hover:bg-stone-700 text-stone-200 font-extrabold text-xs uppercase tracking-wider border border-stone-600 transition">
                    Lobby
                </button>
            </div>
        </div>
    </div>

    <!-- Pause & Roster Overlay ([P]) -->
    <div id="pause-menu-overlay" class="fixed inset-0 z-30 hidden items-center justify-center bg-black/70 backdrop-blur-md p-4">
        <div class="glass-panel max-w-2xl w-full p-6 rounded-3xl shadow-2xl border border-lime-400/30 max-h-[90vh] overflow-y-auto">
            <div class="flex items-center justify-between border-b border-stone-700/80 pb-4">
                <div>
                    <h1 class="text-2xl font-black tracking-tight text-white">DEAD <span class="text-lime-400">SHOT</span></h1>
                    <p class="text-xs text-stone-400 mt-0.5">Woodland Zombie Outbreak • 100 HP • 25 Head / 15 Body</p>
                </div>
                <button onclick="resumeGameplay()" class="px-4 py-2 rounded-xl bg-lime-500 hover:bg-lime-400 text-stone-950 font-black text-xs uppercase tracking-wider">
                    Resume
                </button>
            </div>

            <div class="grid grid-cols-1 md:grid-cols-2 gap-5 mt-5">
                <div class="bg-stone-900/75 p-4 rounded-2xl border border-stone-700/70 flex flex-col justify-between">
                    <div>
                        <h3 class="text-xs font-extrabold uppercase tracking-wider text-lime-300 mb-2">Survival Controls</h3>
                        <ul class="space-y-1.5 text-xs text-stone-300">
                            <li><span class="mono text-lime-300 font-bold">W A S D</span> — Move • <span class="mono text-lime-300 font-bold">Shift</span> — Sprint • <span class="mono text-lime-300 font-bold">Space</span> — Jump</li>
                            <li><span class="mono text-lime-300 font-bold">Left Click / F</span> — Fire • <span class="mono text-lime-300 font-bold">R</span> — Reload</li>
                            <li><span class="mono text-amber-300 font-bold">E</span> — Open Supply Chests (M4A1, M870, Medkits)</li>
                            <li><span class="mono text-red-300 font-bold">5 / H</span> — Apply Fortnite Medkit (3.0s -> 100 HP)</li>
                            <li><span class="mono text-lime-300 font-bold">M</span> — Toggle Live Auto-Survivor Simulation</li>
                        </ul>
                    </div>
                    <div class="mt-4 flex gap-2">
                        <button onclick="resetSoloMatch()" class="flex-1 py-2.5 rounded-xl bg-emerald-600 hover:bg-emerald-500 text-white font-extrabold text-xs uppercase tracking-wider transition">
                            Restart Wave
                        </button>
                        <button onclick="returnToMainLobby()" class="px-3.5 py-2.5 rounded-xl bg-stone-800 hover:bg-stone-700 text-stone-200 font-extrabold text-xs uppercase tracking-wider border border-stone-600 transition">
                            Lobby
                        </button>
                    </div>
                </div>

                <div class="bg-stone-900/75 p-4 rounded-2xl border border-stone-700/70">
                    <div class="flex items-center justify-between mb-2">
                        <h3 class="text-xs font-extrabold uppercase tracking-wider text-red-400">Infected Horde (19)</h3>
                        <span id="menu-alive-count" class="mono text-[11px] text-lime-400 font-bold">19 REMAINING</span>
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
        const PLAYER_MAX_HEALTH = 100;
        const BOT_MAX_HEALTH = 100;
        const HEADSHOT_DAMAGE = 25;
        const BODYSHOT_DAMAGE = 15;
        const ZOMBIE_CLAW_DAMAGE = 22;
        const BOT_COUNT = 19; // 19 zombies (no respawns)
        const MEDKIT_DURATION = 3.0; // 3.0s channeling timer

        const ZOMBIE_NAMES = [
            "Walker-01", "Rotter-02", "Stalker-03", "Shambler-04", "Crawler-05",
            "Biter-06", "Ghoul-07", "Lurker-08", "Infected-09", "Husk-10",
            "Brute-11", "Fester-12", "Ravager-13", "Plague-14", "Gnasher-15",
            "Screecher-16", "Hollow-17", "Wretch-18", "Blight-19"
        ];

        const BOT_CALLSIGNS = [
            "Viper-01", "Ghost-02", "Reaper-03", "Falcon-04", "Shadow-05",
            "Cobra-06", "Nomad-07", "Titan-08", "Specter-09", "Rogue-10",
            "Raven-11", "Wolf-12", "Jackal-13", "Hawk-14", "Blaze-15",
            "Storm-16", "Apex-17", "Hunter-18", "Phantom-19"
        ];

        let gameMode = "zombies"; // "zombies" or "classic"
        let lobbyDifficulty = "normal";
        let selectedMap = "woodland";
        let inMainLobby = true;

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
        let blueprintViewModel = null;
        let buildPreviewGroup = null;
        let builtStructures = [];
        let arenaGroup = null;
        let colliders = [];
        let bots = [];
        let chests = [];
        let tracers = [];

        const BUILD_GRID = 4.0;
        const BUILD_HEIGHT = 3.2;

        const player = {
            name: "Survivor-01",
            pos: new THREE.Vector3(0, 1.75, 0),
            vel: new THREE.Vector3(0, 0, 0),
            yaw: 0,
            pitch: 0,
            health: PLAYER_MAX_HEALTH,
            alive: true,
            kills: 0,
            onGround: true,
            activeSlot: 2, // Start with M9 Pistol equipped
            buildMode: false,
            buildPiece: "wall", // "wall", "floor", "ramp"
            wood: 150,
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

            arenaGroup = new THREE.Group();
            scene.add(arenaGroup);

            buildFirstPersonViewModels();
            buildSelectedMap();
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
        // 4. FIRST-PERSON 3D VIEW-MODELS (M4A1, M870, M9 PISTOL, KNIFE, MEDKIT)
        // =========================================================================
        function buildFirstPersonViewModels() {
            viewModels = [];
            const darkMetal = new THREE.MeshStandardMaterial({ color: 0x1f2428, metalness: 0.75, roughness: 0.28 });
            const gunMetal = new THREE.MeshStandardMaterial({ color: 0x3b4248, metalness: 0.65, roughness: 0.35 });
            const woodMat = new THREE.MeshStandardMaterial({ color: 0x6b3e26, roughness: 0.75 });
            const bladeMat = new THREE.MeshStandardMaterial({ color: 0xd1d5db, metalness: 0.9, roughness: 0.15 });
            const redCaseMat = new THREE.MeshStandardMaterial({ color: 0xdc2626, roughness: 0.32, metalness: 0.1 });
            const whiteCrossMat = new THREE.MeshBasicMaterial({ color: 0xffffff });
            const darkTrimMat = new THREE.MeshStandardMaterial({ color: 0x18181b, roughness: 0.6 });

            // [0] M4A1 Carbine
            const m4Group = new THREE.Group();
            const m4Body = new THREE.Mesh(new THREE.BoxGeometry(0.07, 0.1, 0.42), darkMetal);
            const m4Barrel = new THREE.Mesh(new THREE.CylinderGeometry(0.014, 0.014, 0.34, 8), gunMetal);
            m4Barrel.rotation.x = Math.PI / 2;
            m4Barrel.position.set(0, 0.02, -0.34);
            const m4Mag = new THREE.Mesh(new THREE.BoxGeometry(0.045, 0.14, 0.07), gunMetal);
            m4Mag.position.set(0, -0.1, -0.04);
            m4Mag.rotation.x = 0.2;
            const m4Sight = new THREE.Mesh(new THREE.BoxGeometry(0.04, 0.045, 0.09), darkMetal);
            m4Sight.position.set(0, 0.07, 0.02);
            m4Group.add(m4Body, m4Barrel, m4Mag, m4Sight);
            m4Group.position.set(0.24, -0.23, -0.44);
            m4Group.visible = false;
            camera.add(m4Group);
            viewModels.push(m4Group);

            // [1] M870 Breacher Shotgun
            const sgGroup = new THREE.Group();
            const sgReceiver = new THREE.Mesh(new THREE.BoxGeometry(0.075, 0.1, 0.38), darkMetal);
            const sgBarrel = new THREE.Mesh(new THREE.CylinderGeometry(0.02, 0.02, 0.44, 8), gunMetal);
            sgBarrel.rotation.x = Math.PI / 2;
            sgBarrel.position.set(0, 0.025, -0.36);
            const sgPump = new THREE.Mesh(new THREE.BoxGeometry(0.065, 0.065, 0.18), woodMat);
            sgPump.position.set(0, -0.02, -0.26);
            sgGroup.add(sgReceiver, sgBarrel, sgPump);
            sgGroup.position.set(0.24, -0.23, -0.44);
            sgGroup.visible = false;
            camera.add(sgGroup);
            viewModels.push(sgGroup);

            // [2] M9 Tactical Pistol (Default Equipped)
            const pistolGroup = new THREE.Group();
            const slide = new THREE.Mesh(new THREE.BoxGeometry(0.052, 0.065, 0.22), darkMetal);
            const grip = new THREE.Mesh(new THREE.BoxGeometry(0.048, 0.12, 0.06), gunMetal);
            grip.position.set(0, -0.075, 0.06);
            grip.rotation.x = -0.22;
            pistolGroup.add(slide, grip);
            pistolGroup.position.set(0.23, -0.22, -0.42);
            pistolGroup.visible = true;
            camera.add(pistolGroup);
            viewModels.push(pistolGroup);

            // [3] Combat Knife
            const knifeGroup = new THREE.Group();
            const handle = new THREE.Mesh(new THREE.CylinderGeometry(0.02, 0.022, 0.13, 8), darkTrimMat);
            handle.rotation.x = Math.PI / 2.4;
            const blade = new THREE.Mesh(new THREE.BoxGeometry(0.01, 0.032, 0.19), bladeMat);
            blade.position.set(0, 0.035, -0.13);
            blade.rotation.x = -0.25;
            knifeGroup.add(handle, blade);
            knifeGroup.position.set(0.24, -0.22, -0.38);
            knifeGroup.visible = false;
            camera.add(knifeGroup);
            viewModels.push(knifeGroup);

            // [4] Fortnite-Style Hard-Shell Med Kit
            const medGroup = new THREE.Group();
            const medShell = new THREE.Mesh(new THREE.BoxGeometry(0.24, 0.18, 0.1), redCaseMat);
            const medHandle = new THREE.Mesh(new THREE.BoxGeometry(0.11, 0.035, 0.04), darkTrimMat);
            medHandle.position.set(0, 0.105, 0);
            const latchL = new THREE.Mesh(new THREE.BoxGeometry(0.025, 0.03, 0.11), darkTrimMat);
            latchL.position.set(-0.07, 0.075, 0);
            const latchR = new THREE.Mesh(new THREE.BoxGeometry(0.025, 0.03, 0.11), darkTrimMat);
            latchR.position.set(0.07, 0.075, 0);
            const crossH = new THREE.Mesh(new THREE.BoxGeometry(0.09, 0.028, 0.104), whiteCrossMat);
            const crossV = new THREE.Mesh(new THREE.BoxGeometry(0.028, 0.09, 0.104), whiteCrossMat);
            medGroup.add(medShell, medHandle, latchL, latchR, crossH, crossV);
            medGroup.position.set(0.22, -0.22, -0.42);
            medGroup.rotation.y = -0.25;
            medGroup.visible = false;
            camera.add(medGroup);
            viewModels.push(medGroup);

            // [Build Mode] Blueprint Paper & Pencil View-Model
            blueprintViewModel = new THREE.Group();
            const bpBoard = new THREE.Mesh(
                new THREE.BoxGeometry(0.24, 0.014, 0.18),
                new THREE.MeshStandardMaterial({ color: 0x0284c7, roughness: 0.4 })
            );
            const bpGrid = new THREE.Mesh(
                new THREE.BoxGeometry(0.21, 0.016, 0.15),
                new THREE.MeshBasicMaterial({ color: 0x38bdf8, wireframe: true })
            );
            const pencil = new THREE.Mesh(
                new THREE.CylinderGeometry(0.008, 0.008, 0.16, 6),
                new THREE.MeshStandardMaterial({ color: 0xfacc15 })
            );
            pencil.rotation.z = Math.PI / 3;
            pencil.position.set(0.06, 0.04, 0.02);
            blueprintViewModel.add(bpBoard, bpGrid, pencil);
            blueprintViewModel.position.set(0.2, -0.22, -0.38);
            blueprintViewModel.rotation.x = 0.45;
            blueprintViewModel.rotation.y = -0.2;
            blueprintViewModel.visible = false;
            camera.add(blueprintViewModel);

            // World-Space Hologram Preview Group for Fortnite Snap-Building
            buildPreviewGroup = new THREE.Group();
            buildPreviewGroup.visible = false;
            scene.add(buildPreviewGroup);
        }

        // =========================================================================
        // 4B. FORTNITE GRID BUILDING SYSTEM (WALLS, FLOORS & RAMPS)
        // =========================================================================
        function createStructureGeometry(type) {
            if (type === "wall") {
                return new THREE.BoxGeometry(BUILD_GRID, BUILD_HEIGHT, 0.28);
            } else if (type === "floor") {
                return new THREE.BoxGeometry(BUILD_GRID, 0.24, BUILD_GRID);
            } else {
                // Angled Ramp
                const rampLen = Math.hypot(BUILD_GRID, BUILD_HEIGHT);
                const geo = new THREE.BoxGeometry(BUILD_GRID, 0.24, rampLen);
                geo.rotateX(Math.atan2(BUILD_HEIGHT, BUILD_GRID));
                return geo;
            }
        }

        function updateBlueprintHologramMesh() {
            if (!buildPreviewGroup) return;
            while (buildPreviewGroup.children.length > 0) {
                buildPreviewGroup.remove(buildPreviewGroup.children[0]);
            }
            const geo = createStructureGeometry(player.buildPiece);
            const holoMat = new THREE.MeshBasicMaterial({
                color: player.wood >= 10 ? 0x38bdf8 : 0xef4444,
                transparent: true,
                opacity: 0.42,
                depthWrite: false
            });
            const wireMat = new THREE.MeshBasicMaterial({
                color: player.wood >= 10 ? 0x7dd3fc : 0xfca5a5,
                wireframe: true
            });
            buildPreviewGroup.add(new THREE.Mesh(geo, holoMat), new THREE.Mesh(geo, wireMat));
        }

        function computeBuildSnapTransform() {
            const forward = new THREE.Vector3(-Math.sin(player.yaw), 0, -Math.cos(player.yaw));
            const targetPoint = player.pos.clone().addScaledVector(forward, 3.6);

            const gx = Math.round(targetPoint.x / BUILD_GRID) * BUILD_GRID;
            const gz = Math.round(targetPoint.z / BUILD_GRID) * BUILD_GRID;
            const footY = Math.max(0, player.pos.y - 1.75);
            const levelY = Math.floor((footY + 0.4) / BUILD_HEIGHT) * BUILD_HEIGHT;

            // Snap rotation to nearest 90-degree cardinal direction
            const cardinalAngle = Math.round(player.yaw / (Math.PI / 2)) * (Math.PI / 2);

            let yPos = levelY;
            if (player.buildPiece === "wall") yPos = levelY + BUILD_HEIGHT / 2;
            else if (player.buildPiece === "floor") yPos = levelY + 0.12;
            else if (player.buildPiece === "ramp") yPos = levelY + BUILD_HEIGHT / 2;

            return {
                x: gx,
                y: yPos,
                z: gz,
                rotY: cardinalAngle,
                levelY
            };
        }

        function toggleBuildMode(forceState) {
            player.buildMode = typeof forceState === "boolean" ? forceState : !player.buildMode;
            if (player.buildMode) {
                player.isHealing = false;
                player.reloading = false;
                document.getElementById("medkit-progress-wrap").classList.add("hidden");
                viewModels.forEach(vm => { vm.visible = false; });
                if (blueprintViewModel) blueprintViewModel.visible = true;
                updateBlueprintHologramMesh();
                if (buildPreviewGroup) buildPreviewGroup.visible = true;
            } else {
                if (blueprintViewModel) blueprintViewModel.visible = false;
                if (buildPreviewGroup) buildPreviewGroup.visible = false;
                switchWeapon(player.activeSlot);
            }
            updateWeaponHUD();
        }

        function selectBuildPiece(piece) {
            player.buildPiece = piece;
            if (!player.buildMode) {
                toggleBuildMode(true);
            } else {
                updateBlueprintHologramMesh();
                updateWeaponHUD();
            }
        }

        function placeFortniteStructure() {
            if (!player.buildMode || player.wood < 10 || !player.alive) return;
            const snap = computeBuildSnapTransform();

            // Prevent duplicate overlapping piece at the exact same grid coordinate & type
            const duplicate = builtStructures.some(s =>
                s.type === player.buildPiece &&
                Math.abs(s.x - snap.x) < 0.2 &&
                Math.abs(s.y - snap.y) < 0.2 &&
                Math.abs(s.z - snap.z) < 0.2
            );
            if (duplicate) return;

            player.wood -= 10;
            updateWoodHUD();
            playSound("loot");

            const geo = createStructureGeometry(player.buildPiece);
            const plankMat = new THREE.MeshStandardMaterial({
                color: 0xa16207,
                roughness: 0.78
            });
            const frameMat = new THREE.MeshStandardMaterial({
                color: 0x713f12,
                roughness: 0.85
            });

            const mesh = new THREE.Mesh(geo, plankMat);
            mesh.position.set(snap.x, snap.y, snap.z);
            mesh.rotation.y = snap.rotY;
            mesh.castShadow = true;
            mesh.receiveShadow = true;

            const wireOverlay = new THREE.Mesh(geo, frameMat);
            wireOverlay.scale.set(1.005, 1.005, 1.005);
            wireOverlay.material.wireframe = true;
            mesh.add(wireOverlay);

            scene.add(mesh);

            // Add solid wall collision if placing a vertical wall
            if (player.buildPiece === "wall") {
                const isZAligned = Math.abs(Math.cos(snap.rotY)) > 0.5;
                addBoxCollider(
                    snap.x,
                    snap.z,
                    isZAligned ? BUILD_GRID / 2 : 0.2,
                    isZAligned ? 0.2 : BUILD_GRID / 2
                );
            }

            builtStructures.push({
                type: player.buildPiece,
                x: snap.x,
                y: snap.y,
                z: snap.z,
                levelY: snap.levelY,
                rotY: snap.rotY,
                mesh
            });

            updateBlueprintHologramMesh();
        }

        function getStructureSupportHeight(x, z, currentFootY) {
            let supportY = 0;
            for (let i = 0; i < builtStructures.length; i++) {
                const s = builtStructures[i];
                if (s.type === "wall") continue;
                const half = BUILD_GRID / 2 + 0.25;
                if (Math.abs(x - s.x) <= half && Math.abs(z - s.z) <= half) {
                    if (s.type === "floor") {
                        const topY = s.levelY + 0.24;
                        if (topY <= currentFootY + 0.65 && topY > supportY) {
                            supportY = topY;
                        }
                    } else if (s.type === "ramp") {
                        // Compute local forward coordinate along the ramp slope
                        const dx = x - s.x;
                        const dz = z - s.z;
                        const localForward = -dx * Math.sin(s.rotY) - dz * Math.cos(s.rotY);
                        const t = Math.max(0, Math.min(1, (localForward + BUILD_GRID / 2) / BUILD_GRID));
                        const rampTopY = s.levelY + t * BUILD_HEIGHT + 0.15;
                        if (rampTopY <= currentFootY + 0.85 && rampTopY > supportY) {
                            supportY = rampTopY;
                        }
                    }
                }
            }
            return supportY;
        }

        function updateWoodHUD() {
            const el = document.getElementById("hud-wood-count");
            if (el) el.textContent = player.wood;
        }

        // =========================================================================
        // 5. MULTI-MAP ARENA BUILDER (WOODLAND, CITY, DESERT — 390 x 390)
        // =========================================================================
        function addBoxCollider(x, z, halfW, halfD) {
            colliders.push({
                minX: x - halfW - 0.65,
                maxX: x + halfW + 0.65,
                minZ: z - halfD - 0.65,
                maxZ: z + halfD + 0.65
            });
        }

        function resolveArenaCollisions(pos) {
            const limit = ARENA_HALF_SIZE - 2.2;
            if (pos.x < -limit) pos.x = -limit;
            if (pos.x > limit) pos.x = limit;
            if (pos.z < -limit) pos.z = -limit;
            if (pos.z > limit) pos.z = limit;

            for (let i = 0; i < colliders.length; i++) {
                const c = colliders[i];
                if (pos.x > c.minX && pos.x < c.maxX && pos.z > c.minZ && pos.z < c.maxZ) {
                    const penLeft = pos.x - c.minX;
                    const penRight = c.maxX - pos.x;
                    const penTop = pos.z - c.minZ;
                    const penBottom = c.maxZ - pos.z;
                    const minPen = Math.min(penLeft, penRight, penTop, penBottom);

                    if (minPen === penLeft) pos.x = c.minX;
                    else if (minPen === penRight) pos.x = c.maxX;
                    else if (minPen === penTop) pos.z = c.minZ;
                    else pos.z = c.maxZ;
                }
            }
        }

        function buildSelectedMap() {
            colliders = [];
            while (arenaGroup.children.length > 0) {
                arenaGroup.remove(arenaGroup.children[0]);
            }

            let seed = selectedMap === "city" ? 4040 : (selectedMap === "desert" ? 9001 : 1337);
            const rand = () => {
                seed = (seed * 16807) % 2147483647;
                return (seed - 1) / 2147483646;
            };

            const wallLen = ARENA_HALF_SIZE * 2;
            const addPerimeterWalls = (colorHex) => {
                const wallMat = new THREE.MeshStandardMaterial({ color: colorHex, roughness: 0.85 });
                const northWall = new THREE.Mesh(new THREE.BoxGeometry(wallLen, 10, 3), wallMat);
                northWall.position.set(0, 5, -ARENA_HALF_SIZE);
                const southWall = new THREE.Mesh(new THREE.BoxGeometry(wallLen, 10, 3), wallMat);
                southWall.position.set(0, 5, ARENA_HALF_SIZE);
                const eastWall = new THREE.Mesh(new THREE.BoxGeometry(3, 10, wallLen), wallMat);
                eastWall.position.set(ARENA_HALF_SIZE, 5, 0);
                const westWall = new THREE.Mesh(new THREE.BoxGeometry(3, 10, wallLen), wallMat);
                westWall.position.set(-ARENA_HALF_SIZE, 5, 0);
                arenaGroup.add(northWall, southWall, eastWall, westWall);
            };

            if (selectedMap === "city") {
                // Urban Ruined City Atmosphere
                scene.background = new THREE.Color(0x64748b);
                scene.fog = new THREE.FogExp2(0x64748b, 0.0046);

                const asphaltMat = new THREE.MeshStandardMaterial({ color: 0x27272a, roughness: 0.9 });
                const ground = new THREE.Mesh(new THREE.PlaneGeometry(wallLen + 20, wallLen + 20), asphaltMat);
                ground.rotation.x = -Math.PI / 2;
                ground.receiveShadow = true;
                arenaGroup.add(ground);

                addPerimeterWalls(0x3f3f46);

                // Cross-city avenue road markings
                const roadStripeMat = new THREE.MeshBasicMaterial({ color: 0xfacc15 });
                for (let s = -170; s <= 170; s += 16) {
                    if (Math.abs(s) < 12) continue;
                    const stripeZ = new THREE.Mesh(new THREE.BoxGeometry(0.45, 0.04, 7), roadStripeMat);
                    stripeZ.position.set(0, 0.02, s);
                    const stripeX = new THREE.Mesh(new THREE.BoxGeometry(7, 0.04, 0.45), roadStripeMat);
                    stripeX.position.set(s, 0.02, 0);
                    arenaGroup.add(stripeZ, stripeX);
                }

                // City Skyscrapers & Apartment Blocks
                const bldgColors = [0x475569, 0x334155, 0x52525b, 0x3f3f46, 0x64748b];
                const windowMat = new THREE.MeshBasicMaterial({ color: 0xfef08a });
                for (let gx = -150; gx <= 150; gx += 50) {
                    for (let gz = -150; gz <= 150; gz += 50) {
                        if (Math.abs(gx) < 25 || Math.abs(gz) < 25) continue;
                        const w = 16 + rand() * 8;
                        const d = 16 + rand() * 8;
                        const h = 16 + rand() * 26;
                        const bMat = new THREE.MeshStandardMaterial({
                            color: bldgColors[Math.floor(rand() * bldgColors.length)],
                            roughness: 0.78
                        });
                        const bldg = new THREE.Mesh(new THREE.BoxGeometry(w, h, d), bMat);
                        bldg.position.set(gx, h / 2, gz);
                        bldg.castShadow = true;
                        bldg.receiveShadow = true;
                        arenaGroup.add(bldg);
                        addBoxCollider(gx, gz, w / 2, d / 2);

                        // Lit apartment windows
                        const winBand = new THREE.Mesh(new THREE.BoxGeometry(w + 0.25, 0.65, d + 0.25), windowMat);
                        winBand.position.set(gx, Math.min(h - 3, 7 + rand() * 8), gz);
                        arenaGroup.add(winBand);
                    }
                }

                // Abandoned Cars & Concrete Barricades
                const carColors = [0xb91c1c, 0x1d4ed8, 0xd97706, 0xe4e4e7, 0x18181b];
                for (let i = 0; i < 110; i++) {
                    const cx = (rand() - 0.5) * (ARENA_HALF_SIZE * 1.84);
                    const cz = (rand() - 0.5) * (ARENA_HALF_SIZE * 1.84);
                    if (Math.hypot(cx, cz) < 16) continue;
                    // Skip inside buildings
                    if (Math.abs(cx % 50) < 15 && Math.abs(cz % 50) < 15 && Math.abs(cx) > 25 && Math.abs(cz) > 25) continue;

                    const carMat = new THREE.MeshStandardMaterial({
                        color: carColors[Math.floor(rand() * carColors.length)],
                        roughness: 0.45,
                        metalness: 0.4
                    });
                    const carBody = new THREE.Mesh(new THREE.BoxGeometry(2.3, 1.1, 4.4), carMat);
                    carBody.position.set(cx, 0.65, cz);
                    carBody.castShadow = true;
                    const cabin = new THREE.Mesh(new THREE.BoxGeometry(2.0, 0.85, 2.2), carMat);
                    cabin.position.set(cx, 1.55, cz - 0.2);
                    arenaGroup.add(carBody, cabin);
                    addBoxCollider(cx, cz, 1.25, 2.25);
                }

                // Concrete Jersey Barriers, Dumpsters & Streetlights
                const barrierMat = new THREE.MeshStandardMaterial({ color: 0x9ca3af, roughness: 0.88 });
                const dumpsterMat = new THREE.MeshStandardMaterial({ color: 0x15803d, roughness: 0.75 });
                const poleMat = new THREE.MeshStandardMaterial({ color: 0x27272a, metalness: 0.7, roughness: 0.35 });
                const lampBulbMat = new THREE.MeshBasicMaterial({ color: 0xfef9c3 });

                for (let i = 0; i < 120; i++) {
                    const bx = (rand() - 0.5) * (ARENA_HALF_SIZE * 1.85);
                    const bz = (rand() - 0.5) * (ARENA_HALF_SIZE * 1.85);
                    if (Math.hypot(bx, bz) < 14) continue;
                    if (Math.abs(bx % 50) < 15 && Math.abs(bz % 50) < 15 && Math.abs(bx) > 25 && Math.abs(bz) > 25) continue;

                    const kind = i % 3;
                    if (kind === 0) {
                        // Concrete Road Barrier
                        const alongX = rand() > 0.5;
                        const w = alongX ? 4.2 : 0.9;
                        const d = alongX ? 0.9 : 4.2;
                        const barrier = new THREE.Mesh(new THREE.BoxGeometry(w, 1.25, d), barrierMat);
                        barrier.position.set(bx, 0.625, bz);
                        barrier.castShadow = true;
                        arenaGroup.add(barrier);
                        addBoxCollider(bx, bz, w / 2, d / 2);
                    } else if (kind === 1) {
                        // Industrial Dumpster
                        const dumpster = new THREE.Mesh(new THREE.BoxGeometry(2.4, 1.6, 1.6), dumpsterMat);
                        dumpster.position.set(bx, 0.8, bz);
                        dumpster.castShadow = true;
                        arenaGroup.add(dumpster);
                        addBoxCollider(bx, bz, 1.2, 0.8);
                    } else {
                        // Streetlight Pole
                        const pole = new THREE.Mesh(new THREE.CylinderGeometry(0.14, 0.2, 7.5, 6), poleMat);
                        pole.position.set(bx, 3.75, bz);
                        const arm = new THREE.Mesh(new THREE.BoxGeometry(1.6, 0.16, 0.16), poleMat);
                        arm.position.set(bx + 0.7, 7.4, bz);
                        const lamp = new THREE.Mesh(new THREE.BoxGeometry(0.5, 0.18, 0.3), lampBulbMat);
                        lamp.position.set(bx + 1.3, 7.28, bz);
                        arenaGroup.add(pole, arm, lamp);
                        addBoxCollider(bx, bz, 0.35, 0.35);
                    }
                }
            } else if (selectedMap === "desert") {
                // Scorched Desert Outpost Atmosphere
                scene.background = new THREE.Color(0xe6be8a);
                scene.fog = new THREE.FogExp2(0xe6be8a, 0.0038);

                const sandMat = new THREE.MeshStandardMaterial({ color: 0xd9a05b, roughness: 0.95 });
                const ground = new THREE.Mesh(new THREE.PlaneGeometry(wallLen + 20, wallLen + 20), sandMat);
                ground.rotation.x = -Math.PI / 2;
                ground.receiveShadow = true;
                arenaGroup.add(ground);

                addPerimeterWalls(0x9a6b3b);

                // Adobe Compound Ruins
                const adobeMat = new THREE.MeshStandardMaterial({ color: 0xc28549, roughness: 0.88 });
                const roofTrimMat = new THREE.MeshStandardMaterial({ color: 0x784b26, roughness: 0.8 });
                const compoundCoords = [
                    [-45, -40], [50, -45], [-55, 55], [48, 52],
                    [-115, -105], [115, -105], [-115, 105], [115, 105],
                    [0, -115], [0, 115], [-135, 0], [135, 0]
                ];
                compoundCoords.forEach(([cx, cz]) => {
                    const house = new THREE.Mesh(new THREE.BoxGeometry(12, 4.8, 10), adobeMat);
                    house.position.set(cx, 2.4, cz);
                    house.castShadow = true;
                    house.receiveShadow = true;
                    const parapet = new THREE.Mesh(new THREE.BoxGeometry(12.6, 0.6, 10.6), roofTrimMat);
                    parapet.position.set(cx, 5.0, cz);
                    arenaGroup.add(house, parapet);
                    addBoxCollider(cx, cz, 6.0, 5.0);
                });

                // Sandstone Mesas, Ruined Walls, Supply Crates & Cacti
                const mesaMat = new THREE.MeshStandardMaterial({ color: 0xb45309, roughness: 0.9 });
                const cactusMat = new THREE.MeshStandardMaterial({ color: 0x2e6f40, roughness: 0.8 });
                const crateMat = new THREE.MeshStandardMaterial({ color: 0x854d0e, roughness: 0.78 });

                for (let i = 0; i < 120; i++) {
                    const mx = (rand() - 0.5) * (ARENA_HALF_SIZE * 1.84);
                    const mz = (rand() - 0.5) * (ARENA_HALF_SIZE * 1.84);
                    if (Math.hypot(mx, mz) < 16) continue;
                    const rTop = 1.8 + rand() * 2.4;
                    const rBot = rTop + 1.5;
                    const h = 2.4 + rand() * 4.8;
                    const mesa = new THREE.Mesh(new THREE.CylinderGeometry(rTop, rBot, h, 7), mesaMat);
                    mesa.position.set(mx, h / 2, mz);
                    mesa.castShadow = true;
                    mesa.receiveShadow = true;
                    arenaGroup.add(mesa);
                    addBoxCollider(mx, mz, rBot * 0.85, rBot * 0.85);
                }

                // Desert Ruin Walls, Wooden Crate Clusters & Sandstone Pillars
                for (let i = 0; i < 115; i++) {
                    const ox = (rand() - 0.5) * (ARENA_HALF_SIZE * 1.84);
                    const oz = (rand() - 0.5) * (ARENA_HALF_SIZE * 1.84);
                    if (Math.hypot(ox, oz) < 14) continue;
                    const kind = i % 3;
                    if (kind === 0) {
                        // Crumbled Adobe Cover Wall
                        const alongX = rand() > 0.5;
                        const w = alongX ? 6.5 : 1.1;
                        const d = alongX ? 1.1 : 6.5;
                        const wall = new THREE.Mesh(new THREE.BoxGeometry(w, 2.2, d), adobeMat);
                        wall.position.set(ox, 1.1, oz);
                        wall.castShadow = true;
                        arenaGroup.add(wall);
                        addBoxCollider(ox, oz, w / 2, d / 2);
                    } else if (kind === 1) {
                        // Stacked Wooden Supply Crates
                        const crate1 = new THREE.Mesh(new THREE.BoxGeometry(1.6, 1.6, 1.6), crateMat);
                        crate1.position.set(ox, 0.8, oz);
                        crate1.castShadow = true;
                        const crate2 = new THREE.Mesh(new THREE.BoxGeometry(1.3, 1.3, 1.3), crateMat);
                        crate2.position.set(ox + 0.15, 2.25, oz - 0.1);
                        crate2.castShadow = true;
                        arenaGroup.add(crate1, crate2);
                        addBoxCollider(ox, oz, 0.9, 0.9);
                    } else {
                        // Ancient Sandstone Pillar
                        const pillar = new THREE.Mesh(new THREE.CylinderGeometry(0.75, 0.85, 5.2, 8), mesaMat);
                        pillar.position.set(ox, 2.6, oz);
                        pillar.castShadow = true;
                        arenaGroup.add(pillar);
                        addBoxCollider(ox, oz, 0.85, 0.85);
                    }
                }

                for (let i = 0; i < 210; i++) {
                    const cx = (rand() - 0.5) * (ARENA_HALF_SIZE * 1.86);
                    const cz = (rand() - 0.5) * (ARENA_HALF_SIZE * 1.86);
                    if (Math.hypot(cx, cz) < 14) continue;
                    const trunk = new THREE.Mesh(new THREE.CylinderGeometry(0.28, 0.34, 3.6, 7), cactusMat);
                    trunk.position.set(cx, 1.8, cz);
                    trunk.castShadow = true;
                    const arm = new THREE.Mesh(new THREE.BoxGeometry(1.3, 0.28, 0.28), cactusMat);
                    arm.position.set(cx, 2.2, cz);
                    arenaGroup.add(trunk, arm);
                    addBoxCollider(cx, cz, 0.5, 0.5);
                }
            } else {
                // Default Woodland Forest Atmosphere
                scene.background = new THREE.Color(0x88b396);
                scene.fog = new THREE.FogExp2(0x88b396, 0.0042);

                const groundMat = new THREE.MeshStandardMaterial({ color: 0x3a5436, roughness: 0.92 });
                const ground = new THREE.Mesh(new THREE.PlaneGeometry(wallLen + 20, wallLen + 20), groundMat);
                ground.rotation.x = -Math.PI / 2;
                ground.receiveShadow = true;
                arenaGroup.add(ground);

                addPerimeterWalls(0x57534e);

                // Outpost Cabins
                const cabinWallMat = new THREE.MeshStandardMaterial({ color: 0x6b4423, roughness: 0.8 });
                const roofMat = new THREE.MeshStandardMaterial({ color: 0x292524, roughness: 0.7 });
                const cabinCoords = [
                    [-45, -40], [50, -45], [-55, 55], [48, 52],
                    [-115, -105], [115, -105], [-115, 105], [115, 105],
                    [0, -120], [0, 120]
                ];
                cabinCoords.forEach(([cx, cz]) => {
                    const body = new THREE.Mesh(new THREE.BoxGeometry(11, 5.2, 9), cabinWallMat);
                    body.position.set(cx, 2.6, cz);
                    body.castShadow = true;
                    body.receiveShadow = true;
                    const roof = new THREE.Mesh(new THREE.ConeGeometry(8.5, 3.2, 4), roofMat);
                    roof.position.set(cx, 6.7, cz);
                    roof.rotation.y = Math.PI / 4;
                    arenaGroup.add(body, roof);
                    addBoxCollider(cx, cz, 5.5, 4.5);
                });

                // Pine Trees, Fallen Logs, Sandbag Bunkers & Cover Boulders
                const trunkMat = new THREE.MeshStandardMaterial({ color: 0x4a321f, roughness: 0.9 });
                const foliageMat = new THREE.MeshStandardMaterial({ color: 0x1e3f20, roughness: 0.85 });
                const rockMat = new THREE.MeshStandardMaterial({ color: 0x6b7280, roughness: 0.88 });
                const sandbagMat = new THREE.MeshStandardMaterial({ color: 0xa1885f, roughness: 0.92 });

                for (let i = 0; i < 290; i++) {
                    const tx = (rand() - 0.5) * (ARENA_HALF_SIZE * 1.86);
                    const tz = (rand() - 0.5) * (ARENA_HALF_SIZE * 1.86);
                    if (Math.hypot(tx, tz) < 16) continue;

                    const trunk = new THREE.Mesh(new THREE.CylinderGeometry(0.45, 0.65, 5.5, 7), trunkMat);
                    trunk.position.set(tx, 2.75, tz);
                    trunk.castShadow = true;
                    const crown1 = new THREE.Mesh(new THREE.ConeGeometry(3.1, 6.5, 7), foliageMat);
                    crown1.position.set(tx, 7.2, tz);
                    crown1.castShadow = true;
                    const crown2 = new THREE.Mesh(new THREE.ConeGeometry(2.3, 5.2, 7), foliageMat);
                    crown2.position.set(tx, 10.4, tz);
                    arenaGroup.add(trunk, crown1, crown2);
                    addBoxCollider(tx, tz, 0.7, 0.7);
                }

                for (let i = 0; i < 135; i++) {
                    const rx = (rand() - 0.5) * (ARENA_HALF_SIZE * 1.84);
                    const rz = (rand() - 0.5) * (ARENA_HALF_SIZE * 1.84);
                    if (Math.hypot(rx, rz) < 14) continue;
                    const scale = 1.4 + rand() * 1.9;
                    const rock = new THREE.Mesh(new THREE.DodecahedronGeometry(scale, 0), rockMat);
                    rock.position.set(rx, scale * 0.65, rz);
                    rock.castShadow = true;
                    rock.receiveShadow = true;
                    arenaGroup.add(rock);
                    addBoxCollider(rx, rz, scale * 0.8, scale * 0.8);
                }

                // Fallen Timber Logs & Tactical Sandbag Barricades
                for (let i = 0; i < 95; i++) {
                    const lx = (rand() - 0.5) * (ARENA_HALF_SIZE * 1.84);
                    const lz = (rand() - 0.5) * (ARENA_HALF_SIZE * 1.84);
                    if (Math.hypot(lx, lz) < 14) continue;
                    const alongX = rand() > 0.5;
                    if (i % 2 === 0) {
                        // Fallen Tree Trunk
                        const log = new THREE.Mesh(new THREE.CylinderGeometry(0.55, 0.65, 5.8, 7), trunkMat);
                        log.position.set(lx, 0.55, lz);
                        if (alongX) log.rotation.z = Math.PI / 2;
                        else log.rotation.x = Math.PI / 2;
                        log.castShadow = true;
                        arenaGroup.add(log);
                        addBoxCollider(lx, lz, alongX ? 2.9 : 0.65, alongX ? 0.65 : 2.9);
                    } else {
                        // Sandbag Cover Wall
                        const w = alongX ? 4.8 : 1.0;
                        const d = alongX ? 1.0 : 4.8;
                        const bags = new THREE.Mesh(new THREE.BoxGeometry(w, 1.35, d), sandbagMat);
                        bags.position.set(lx, 0.675, lz);
                        bags.castShadow = true;
                        arenaGroup.add(bags);
                        addBoxCollider(lx, lz, w / 2, d / 2);
                    }
                }
            }
        }

        // =========================================================================
        // 6. GLOWING SUPPLY CHESTS (M4A1, M870 SHOTGUN, FORTNITE MEDKITS & AMMO)
        // =========================================================================
        function spawnSupplyChests() {
            chests.forEach(c => scene.remove(c.group));
            chests = [];

            const chestLocations = [
                [0, -14], [14, 0], [-14, 0], [0, 14],
                [-38, -32], [42, -36], [-46, 46], [40, 44],
                [-85, 0], [85, 0], [0, -85], [0, 85],
                [-105, -95], [105, -95], [-105, 95], [105, 95],
                [-145, -40], [145, -40], [-145, 45], [145, 45],
                [-55, -145], [55, -145], [0, 150]
            ];

            const woodChestMat = new THREE.MeshStandardMaterial({ color: 0x78350f, roughness: 0.6 });
            const goldTrimMat = new THREE.MeshStandardMaterial({ color: 0xfbbf24, metalness: 0.7, roughness: 0.25 });
            const beaconMat = new THREE.MeshBasicMaterial({ color: 0xfde047, transparent: true, opacity: 0.25 });

            chestLocations.forEach(([cx, cz]) => {
                const group = new THREE.Group();
                group.position.set(cx, 0, cz);

                const base = new THREE.Mesh(new THREE.BoxGeometry(1.5, 0.75, 0.95), woodChestMat);
                base.position.y = 0.375;
                base.castShadow = true;

                const trim = new THREE.Mesh(new THREE.BoxGeometry(1.56, 0.14, 1.01), goldTrimMat);
                trim.position.y = 0.72;

                const lid = new THREE.Mesh(new THREE.BoxGeometry(1.52, 0.34, 0.97), woodChestMat);
                lid.position.set(0, 0.92, 0);

                const beacon = new THREE.Mesh(new THREE.CylinderGeometry(0.18, 0.45, 7.5, 10), beaconMat);
                beacon.position.y = 4.2;

                group.add(base, trim, lid, beacon);
                scene.add(group);

                chests.push({
                    x: cx,
                    z: cz,
                    group,
                    lid,
                    beacon,
                    opened: false,
                    respawnTimer: 0
                });
            });
        }

        function openChest(chest, silent = false) {
            if (chest.opened) return;
            chest.opened = true;
            chest.respawnTimer = 18.0;
            chest.lid.rotation.x = -0.85;
            chest.lid.position.z = -0.35;
            chest.beacon.visible = false;

            if (!silent) playSound("loot");

            const roll = Math.random();
            let lootMsg = "";
            if (!player.weapons[0].unlocked || roll < 0.38) {
                player.weapons[0].unlocked = true;
                player.weapons[0].reserve += 60;
                lootMsg = "LOOTED: M4A1 CARBINE + 60 AMMO";
                if (player.activeSlot === 2) switchWeapon(0);
            } else if (!player.weapons[1].unlocked || roll < 0.68) {
                player.weapons[1].unlocked = true;
                player.weapons[1].reserve += 18;
                lootMsg = "LOOTED: M870 BREACHER SHOTGUN + SHELLS";
            } else {
                player.weapons[4].unlocked = true;
                player.weapons[4].mag = Math.min(3, player.weapons[4].mag + 1);
                player.weapons[2].reserve += 30;
                lootMsg = "LOOTED: FORTNITE MED KIT + 60 WOOD";
            }

            player.wood = Math.min(999, player.wood + 60);
            updateWoodHUD();
            updateWeaponHUD();
            if (!silent) addKillFeedEntry(lootMsg, "text-amber-300");
        }

        // =========================================================================
        // 7. 19 ZOMBIES (100 HP, NO RESPAWNS, OUTSTRETCHED ARMS & CLAW ATTACKS)
        // =========================================================================
        function createZombieMesh() {
            const group = new THREE.Group();
            const tatteredClothMat = new THREE.MeshStandardMaterial({ color: 0x2c332d, roughness: 0.9 });
            const pantsMat = new THREE.MeshStandardMaterial({ color: 0x1e242b, roughness: 0.9 });
            const zombieSkinMat = new THREE.MeshStandardMaterial({ color: 0x5c7a52, roughness: 0.75 });
            const darkWoundMat = new THREE.MeshStandardMaterial({ color: 0x591c1c, roughness: 0.6 });
            const eyeGlowMat = new THREE.MeshBasicMaterial({ color: 0xff2a2a });

            // Hunched Tattered Torso
            const torso = new THREE.Mesh(new THREE.BoxGeometry(0.62, 0.85, 0.36), tatteredClothMat);
            torso.position.set(0, 1.16, 0.04);
            torso.rotation.x = 0.14;
            torso.castShadow = true;

            const wound = new THREE.Mesh(new THREE.BoxGeometry(0.32, 0.28, 0.38), darkWoundMat);
            wound.position.set(0.08, 1.24, 0.05);
            wound.rotation.x = 0.14;

            // Rotting Zombie Head & Glowing Red Eyes
            const head = new THREE.Mesh(new THREE.BoxGeometry(0.36, 0.38, 0.36), zombieSkinMat);
            head.position.set(0, 1.78, 0.12);
            head.rotation.x = 0.12;
            head.castShadow = true;

            const eyeL = new THREE.Mesh(new THREE.BoxGeometry(0.06, 0.04, 0.04), eyeGlowMat);
            eyeL.position.set(-0.08, 1.81, 0.30);
            const eyeR = new THREE.Mesh(new THREE.BoxGeometry(0.06, 0.04, 0.04), eyeGlowMat);
            eyeR.position.set(0.08, 1.81, 0.30);

            // Shambling Legs
            const legL = new THREE.Mesh(new THREE.BoxGeometry(0.24, 0.76, 0.26), pantsMat);
            legL.position.set(-0.17, 0.38, 0);
            const legR = new THREE.Mesh(new THREE.BoxGeometry(0.24, 0.76, 0.26), pantsMat);
            legR.position.set(0.17, 0.38, 0);

            // Outstretched Zombie Arms (Reaching forward)
            const armL = new THREE.Mesh(new THREE.BoxGeometry(0.16, 0.16, 0.74), zombieSkinMat);
            armL.position.set(-0.39, 1.38, 0.36);
            armL.rotation.x = -0.15;
            armL.castShadow = true;

            const armR = new THREE.Mesh(new THREE.BoxGeometry(0.16, 0.16, 0.74), zombieSkinMat);
            armR.position.set(0.39, 1.38, 0.36);
            armR.rotation.x = -0.15;
            armR.castShadow = true;

            group.add(torso, wound, head, eyeL, eyeR, legL, legR, armL, armR);
            return { group, head, torso, legL, legR, armL, armR };
        }

        function createSoldierBotMesh() {
            const group = new THREE.Group();
            // Full Tactical Military Uniform (Olive Drab Tunic, Tactical Vest, Cargo Pants, Helmet & Boots)
            const oliveTunicMat = new THREE.MeshStandardMaterial({ color: 0x3f4f36, roughness: 0.78 });
            const tacVestMat = new THREE.MeshStandardMaterial({ color: 0x262e22, roughness: 0.65 });
            const cargoPantsMat = new THREE.MeshStandardMaterial({ color: 0x4b533b, roughness: 0.85 });
            const bootMat = new THREE.MeshStandardMaterial({ color: 0x18181b, roughness: 0.6 });
            const skinMat = new THREE.MeshStandardMaterial({ color: 0xd6a883, roughness: 0.7 });
            const helmetMat = new THREE.MeshStandardMaterial({ color: 0x2f3b28, roughness: 0.55 });
            const visorMat = new THREE.MeshStandardMaterial({ color: 0x111827, metalness: 0.8, roughness: 0.2 });
            const gunMat = new THREE.MeshStandardMaterial({ color: 0x1f2937, metalness: 0.75, roughness: 0.3 });

            // Military Torso & Armored Tactical Rig
            const torso = new THREE.Mesh(new THREE.BoxGeometry(0.62, 0.85, 0.34), oliveTunicMat);
            torso.position.set(0, 1.18, 0);
            torso.castShadow = true;

            const vest = new THREE.Mesh(new THREE.BoxGeometry(0.66, 0.58, 0.40), tacVestMat);
            vest.position.set(0, 1.24, 0);
            vest.castShadow = true;

            // Soldier Head, Combat Helmet & Tactical Visor
            const head = new THREE.Mesh(new THREE.BoxGeometry(0.34, 0.36, 0.34), skinMat);
            head.position.set(0, 1.78, 0);
            head.castShadow = true;

            const helmet = new THREE.Mesh(new THREE.BoxGeometry(0.40, 0.19, 0.40), helmetMat);
            helmet.position.set(0, 1.91, -0.01);

            const visor = new THREE.Mesh(new THREE.BoxGeometry(0.32, 0.08, 0.06), visorMat);
            visor.position.set(0, 1.82, 0.18);

            // Cargo Pants & Combat Boots
            const legL = new THREE.Mesh(new THREE.BoxGeometry(0.24, 0.76, 0.26), cargoPantsMat);
            legL.position.set(-0.17, 0.38, 0);
            const bootL = new THREE.Mesh(new THREE.BoxGeometry(0.26, 0.20, 0.32), bootMat);
            bootL.position.set(0, -0.28, 0.03);
            legL.add(bootL);

            const legR = new THREE.Mesh(new THREE.BoxGeometry(0.24, 0.76, 0.26), cargoPantsMat);
            legR.position.set(0.17, 0.38, 0);
            const bootR = new THREE.Mesh(new THREE.BoxGeometry(0.26, 0.20, 0.32), bootMat);
            bootR.position.set(0, -0.28, 0.03);
            legR.add(bootR);

            // Military Sleeves & M4 Carbine
            const armL = new THREE.Mesh(new THREE.BoxGeometry(0.17, 0.52, 0.17), oliveTunicMat);
            armL.position.set(-0.39, 1.25, 0.12);
            armL.rotation.x = -0.6;

            const armR = new THREE.Mesh(new THREE.BoxGeometry(0.17, 0.52, 0.17), oliveTunicMat);
            armR.position.set(0.39, 1.25, 0.18);
            armR.rotation.x = -0.9;

            const rifle = new THREE.Mesh(new THREE.BoxGeometry(0.08, 0.1, 0.68), gunMat);
            rifle.position.set(0.26, 1.25, 0.42);

            group.add(torso, vest, head, helmet, visor, legL, legR, armL, armR, rifle);
            return { group, head, torso, legL, legR, armL, armR };
        }

        function spawnAllBots() {
            bots.forEach(b => scene.remove(b.mesh));
            bots = [];

            const isZombieMode = gameMode === "zombies";
            const speedMult = lobbyDifficulty === "nightmare" ? 1.35 : (lobbyDifficulty === "hard" ? 1.18 : 1.0);

            for (let i = 0; i < BOT_COUNT; i++) {
                const angle = (i / BOT_COUNT) * Math.PI * 2;
                const radius = 44 + (i % 4) * 34;
                const bx = Math.cos(angle) * radius;
                const bz = Math.sin(angle) * radius;

                const rig = isZombieMode ? createZombieMesh() : createSoldierBotMesh();
                rig.group.position.set(bx, 0, bz);
                scene.add(rig.group);

                bots.push({
                    id: i,
                    name: isZombieMode ? (ZOMBIE_NAMES[i] || `Zombie-${i + 1}`) : (BOT_CALLSIGNS[i] || `Bot-${i + 1}`),
                    pos: new THREE.Vector3(bx, 0, bz),
                    yaw: Math.random() * Math.PI * 2,
                    health: BOT_MAX_HEALTH,
                    alive: true,
                    kills: 0,
                    moveSpeed: isZombieMode
                        ? (6.4 + (i % 4) * 0.65) * speedMult
                        : (7.2 + (i % 3) * 0.85) * speedMult,
                    attackCooldown: isZombieMode ? (0.4 + Math.random() * 0.5) : (0.9 + Math.random() * 1.1),
                    wanderTarget: new THREE.Vector3(bx, 0, bz),
                    wanderTimer: 0,
                    stride: Math.random() * 6,
                    mesh: rig.group,
                    headMesh: rig.head,
                    torsoMesh: rig.torso,
                    legL: rig.legL,
                    legR: rig.legR,
                    armL: rig.armL,
                    armR: rig.armR
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

            if (player.buildMode) {
                placeFortniteStructure();
                return;
            }

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
                const verb = gameMode === "zombies" ? "killed" : "eliminated";
                addKillFeedEntry(`${attackerName} ${verb} ${bot.name}${isHeadshot ? " [HEADSHOT]" : ""}`, "text-lime-300");
                updateAliveAndRosterHUD();
                checkMatchVictory();
            }
        }

        function applyDamageToPlayer(amount, attackerName, isHeadshot) {
            if (!player.alive || isMatchOver || inMainLobby) return;
            player.health = Math.max(0, player.health - amount);
            updateHealthHUD();

            const flash = document.getElementById("damage-flash");
            flash.style.opacity = "0.85";
            setTimeout(() => { flash.style.opacity = "0"; }, 140);

            if (player.health <= 0) {
                player.alive = false;
                const aliveEnemies = bots.filter(b => b.alive).length;
                const verb = gameMode === "zombies" ? "infected" : "eliminated";
                addKillFeedEntry(`${attackerName} ${verb} ${player.name}${isHeadshot ? " [HEADSHOT]" : ""}`, "text-red-400");
                updateAliveAndRosterHUD();
                showMatchEnd(false, aliveEnemies);
            }
        }

        function checkMatchVictory() {
            if (isMatchOver) return;
            const aliveEnemies = bots.filter(b => b.alive).length;
            if (player.alive && aliveEnemies === 0) {
                showMatchEnd(true, 0);
            }
        }

        function showMatchEnd(isVictory, remainingEnemies) {
            isMatchOver = true;
            exitPointerLockSafely();
            const overlay = document.getElementById("match-end-overlay");
            overlay.classList.remove("hidden");
            overlay.classList.add("flex");

            const isZombie = gameMode === "zombies";
            document.getElementById("end-banner-tag").textContent = isVictory
                ? (isZombie ? "OUTBREAK CONTAINED" : "VICTORY ROYALE")
                : (isZombie ? "YOU WERE INFECTED" : "ELIMINATED");
            document.getElementById("end-banner-title").textContent = isVictory
                ? "DEAD SHOT VICTORY!"
                : (isZombie ? "OVERRUN BY ZOMBIES" : "BETTER LUCK NEXT DROP");
            document.getElementById("end-banner-sub").textContent = isVictory
                ? (isZombie ? "You eliminated all 19 zombies and survived the outbreak." : "You outlasted all 19 rival soldiers to win the match.")
                : (isZombie ? `${remainingEnemies} zombies were still roaming the sector.` : `Placed #${remainingEnemies + 1} out of 20 combatants.`);
            document.getElementById("end-stat-kills").textContent = player.kills;
            document.getElementById("end-stat-place").textContent = isVictory ? "#1 / 20" : `#${remainingEnemies + 1}`;
        }

        function setLobbyGameMode(mode) {
            gameMode = mode;
            ["zombies", "classic"].forEach(m => {
                const btn = document.getElementById(`mode-${m}`);
                if (!btn) return;
                btn.className = m === mode
                    ? "py-2 rounded-xl bg-lime-500 text-stone-950 font-black text-xs uppercase transition"
                    : "py-2 rounded-xl bg-stone-800 text-stone-300 hover:bg-stone-700 font-extrabold text-xs uppercase transition";
            });
            const isZ = mode === "zombies";
            document.getElementById("lobby-top-badge").textContent = isZ ? "OUTBREAK ZONE • 19 INFECTED" : "BATTLE ROYALE • 20 SOLDIERS";
            document.getElementById("lobby-subtitle").textContent = isZ
                ? "Zombie Survival • 100 HP • 25 Headshot / 15 Body"
                : "Classic Bots BR • 100 HP • Last Survivor Wins";
            document.getElementById("lobby-enemy-count").textContent = isZ ? "19 ZOMBIES DETECTED" : "19 RIVAL BOTS";
            document.getElementById("lobby-enemy-label").textContent = isZ ? "Infected Horde (x19)" : "Tactical Bots (x19)";
        }

        function setLobbyDifficulty(diff) {
            lobbyDifficulty = diff;
            ["normal", "hard", "nightmare"].forEach(d => {
                const btn = document.getElementById(`diff-${d}`);
                if (!btn) return;
                btn.className = d === diff
                    ? "py-2 rounded-xl bg-lime-500 text-stone-950 font-black text-xs uppercase transition"
                    : "py-2 rounded-xl bg-stone-800 text-stone-300 hover:bg-stone-700 font-extrabold text-xs uppercase transition";
            });
        }

        function setLobbyMap(mapName) {
            selectedMap = mapName;
            ["woodland", "city", "desert"].forEach(m => {
                const btn = document.getElementById(`map-${m}`);
                if (!btn) return;
                btn.className = m === mapName
                    ? "py-2 rounded-xl bg-lime-500 text-stone-950 font-black text-xs uppercase transition"
                    : "py-2 rounded-xl bg-stone-800 text-stone-300 hover:bg-stone-700 font-extrabold text-xs uppercase transition";
            });
            const labelEl = document.getElementById("lobby-map-label");
            if (labelEl) {
                const title = mapName.charAt(0).toUpperCase() + mapName.slice(1);
                labelEl.textContent = `${title} Supply Chests`;
            }
            if (arenaGroup) {
                buildSelectedMap();
            }
        }

        function deployFromLobby() {
            initAudio();
            const inputEl = document.getElementById("lobby-player-name");
            const customName = inputEl && inputEl.value.trim() ? inputEl.value.trim() : "Survivor-01";
            player.name = customName;
            inMainLobby = false;
            document.getElementById("main-lobby-overlay").classList.add("hidden");
            resetSoloMatch();
        }

        function returnToMainLobby() {
            inMainLobby = true;
            isPaused = false;
            isMatchOver = false;
            exitPointerLockSafely();
            document.getElementById("pause-menu-overlay").classList.add("hidden");
            document.getElementById("pause-menu-overlay").classList.remove("flex");
            document.getElementById("match-end-overlay").classList.add("hidden");
            document.getElementById("match-end-overlay").classList.remove("flex");
            document.getElementById("main-lobby-overlay").classList.remove("hidden");
            document.getElementById("lobby-preview-name").textContent = `${player.name} (You)`;
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
            player.buildMode = false;
            player.buildPiece = "wall";
            player.wood = 150;
            if (blueprintViewModel) blueprintViewModel.visible = false;
            if (buildPreviewGroup) buildPreviewGroup.visible = false;
            builtStructures.forEach(s => scene.remove(s.mesh));
            builtStructures = [];
            updateWoodHUD();
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
            buildSelectedMap();
            spawnSupplyChests();
            spawnAllBots();
            requestCanvasLockSafely();
        }

        // =========================================================================
        // 10. ENEMY AI UPDATE (ZOMBIE HORDE OR CLASSIC SOLDIER BOTS)
        // =========================================================================
        function updateBots(dt) {
            if (isPaused || isMatchOver || inMainLobby) return;

            const isZombieMode = gameMode === "zombies";
            const dmgMult = lobbyDifficulty === "nightmare" ? 1.35 : (lobbyDifficulty === "hard" ? 1.15 : 1.0);

            for (let i = 0; i < bots.length; i++) {
                const enemy = bots[i];
                if (!enemy.alive) continue;

                if (enemy.attackCooldown > 0) enemy.attackCooldown -= dt;

                if (isZombieMode) {
                    // ZOMBIE SURVIVAL AI: Hunt player and attack with melee claws
                    let chasingPlayer = false;
                    let distToPlayer = Infinity;

                    if (player.alive) {
                        distToPlayer = Math.hypot(player.pos.x - enemy.pos.x, player.pos.z - enemy.pos.z);
                        if (distToPlayer < 135) chasingPlayer = true;
                    }

                    if (chasingPlayer) {
                        const dx = player.pos.x - enemy.pos.x;
                        const dz = player.pos.z - enemy.pos.z;
                        enemy.yaw = Math.atan2(dx, dz);

                        if (distToPlayer > 1.85) {
                            enemy.pos.x += (dx / distToPlayer) * enemy.moveSpeed * dt;
                            enemy.pos.z += (dz / distToPlayer) * enemy.moveSpeed * dt;
                            enemy.stride += dt * 10;
                        }

                        if (distToPlayer <= 2.35 && enemy.attackCooldown <= 0) {
                            enemy.attackCooldown = 1.05;
                            enemy.armL.rotation.x = -0.85;
                            enemy.armR.rotation.x = -0.85;
                            applyDamageToPlayer(Math.round(ZOMBIE_CLAW_DAMAGE * dmgMult), enemy.name, false);
                        } else {
                            enemy.armL.rotation.x = -0.15 + Math.sin(enemy.stride * 0.7) * 0.14;
                            enemy.armR.rotation.x = -0.15 - Math.sin(enemy.stride * 0.7) * 0.14;
                        }
                    } else {
                        enemy.wanderTimer -= dt;
                        if (enemy.wanderTimer <= 0) {
                            enemy.wanderTimer = 3.0 + Math.random() * 4.0;
                            enemy.wanderTarget.set(
                                (Math.random() - 0.5) * (ARENA_HALF_SIZE * 1.7),
                                0,
                                (Math.random() - 0.5) * (ARENA_HALF_SIZE * 1.7)
                            );
                        }
                        const wx = enemy.wanderTarget.x - enemy.pos.x;
                        const wz = enemy.wanderTarget.z - enemy.pos.z;
                        const wd = Math.hypot(wx, wz);
                        if (wd > 2) {
                            enemy.yaw = Math.atan2(wx, wz);
                            enemy.pos.x += (wx / wd) * (enemy.moveSpeed * 0.55) * dt;
                            enemy.pos.z += (wz / wd) * (enemy.moveSpeed * 0.55) * dt;
                            enemy.stride += dt * 6.5;
                        }
                    }
                } else {
                    // CLASSIC BATTLE ROYALE BOTS AI: Shoot each other & the player with rifles
                    let nearestTarget = null;
                    let nearestDist = 115;

                    if (player.alive) {
                        const dp = Math.hypot(player.pos.x - enemy.pos.x, player.pos.z - enemy.pos.z);
                        if (dp < nearestDist) {
                            nearestDist = dp;
                            nearestTarget = { isPlayer: true, pos: player.pos };
                        }
                    }

                    for (let j = 0; j < bots.length; j++) {
                        if (i === j || !bots[j].alive) continue;
                        const db = Math.hypot(bots[j].pos.x - enemy.pos.x, bots[j].pos.z - enemy.pos.z);
                        if (db < nearestDist) {
                            nearestDist = db;
                            nearestTarget = { isPlayer: false, bot: bots[j], pos: bots[j].pos };
                        }
                    }

                    if (nearestTarget) {
                        const dx = nearestTarget.pos.x - enemy.pos.x;
                        const dz = nearestTarget.pos.z - enemy.pos.z;
                        enemy.yaw = Math.atan2(dx, dz);

                        if (nearestDist > 16) {
                            enemy.pos.x += (dx / nearestDist) * enemy.moveSpeed * dt;
                            enemy.pos.z += (dz / nearestDist) * enemy.moveSpeed * dt;
                            enemy.stride += dt * 9;
                        }

                        if (enemy.attackCooldown <= 0) {
                            enemy.attackCooldown = 0.8 + Math.random() * 1.15;
                            const muzzle = new THREE.Vector3(enemy.pos.x, 1.45, enemy.pos.z);
                            const aimTarget = new THREE.Vector3(
                                nearestTarget.pos.x + (Math.random() - 0.5) * 1.8,
                                1.35 + (Math.random() - 0.5) * 0.7,
                                nearestTarget.pos.z + (Math.random() - 0.5) * 1.8
                            );
                            spawnBulletTracer(muzzle, aimTarget, 0xf87171);

                            // Original balanced accuracy (12% - 32%)
                            const hitChance = Math.max(0.12, Math.min(0.32, 0.34 - (nearestDist / 320)));
                            if (Math.random() < hitChance) {
                                const isHead = Math.random() < 0.2;
                                const dmg = isHead ? 30 : 20;
                                if (nearestTarget.isPlayer) {
                                    applyDamageToPlayer(dmg, enemy.name, isHead);
                                } else {
                                    applyDamageToBot(nearestTarget.bot, dmg, enemy.name, isHead);
                                    if (!nearestTarget.bot.alive) enemy.kills++;
                                }
                            }
                        }
                    } else {
                        enemy.wanderTimer -= dt;
                        if (enemy.wanderTimer <= 0) {
                            enemy.wanderTimer = 2.5 + Math.random() * 3.5;
                            enemy.wanderTarget.set(
                                (Math.random() - 0.5) * (ARENA_HALF_SIZE * 1.7),
                                0,
                                (Math.random() - 0.5) * (ARENA_HALF_SIZE * 1.7)
                            );
                        }
                        const wx = enemy.wanderTarget.x - enemy.pos.x;
                        const wz = enemy.wanderTarget.z - enemy.pos.z;
                        const wd = Math.hypot(wx, wz);
                        if (wd > 2) {
                            enemy.yaw = Math.atan2(wx, wz);
                            enemy.pos.x += (wx / wd) * enemy.moveSpeed * dt;
                            enemy.pos.z += (wz / wd) * enemy.moveSpeed * dt;
                            enemy.stride += dt * 8;
                        }
                    }
                }

                resolveArenaCollisions(enemy.pos);
                enemy.mesh.position.copy(enemy.pos);
                enemy.mesh.rotation.y = enemy.yaw;
                enemy.legL.rotation.x = Math.sin(enemy.stride) * 0.5;
                enemy.legR.rotation.x = -Math.sin(enemy.stride) * 0.5;
            }
        }

        function updatePlayer(dt) {
            if (!player.alive || isPaused || isMatchOver || inMainLobby) return;

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
                    addKillFeedEntry("MED KIT APPLIED: +100 FULL HEALTH", "text-emerald-300");
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
            // Gravity, Jump & Walkable Ramps / Floors Support
            player.vel.y -= 19.5 * dt;
            player.pos.y += player.vel.y * dt;
            const footY = player.pos.y - 1.75;
            const groundSupportY = getStructureSupportHeight(player.pos.x, player.pos.z, footY);
            const targetEyeY = 1.75 + groundSupportY;

            if (player.pos.y <= targetEyeY) {
                player.pos.y = targetEyeY;
                player.vel.y = 0;
                player.onGround = true;
            } else {
                player.onGround = false;
            }

            resolveArenaCollisions(player.pos);

            // Update Fortnite Hologram Blueprint Position in Build Mode
            if (player.buildMode && buildPreviewGroup) {
                const snap = computeBuildSnapTransform();
                buildPreviewGroup.position.set(snap.x, snap.y, snap.z);
                buildPreviewGroup.rotation.y = snap.rotY;
            }

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
            if (player.health < 55 && player.weapons[4].mag > 0 && !player.isHealing) {
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
            const buildBar = document.getElementById("build-bar");
            const weaponBar = document.getElementById("weapon-bar");

            if (player.buildMode) {
                if (buildBar) { buildBar.classList.remove("hidden"); buildBar.classList.add("flex"); }
                if (weaponBar) { weaponBar.classList.add("hidden"); }
                document.getElementById("hud-weapon-name").textContent = `BUILD: ${player.buildPiece.toUpperCase()} (-10 WOOD)`;
                document.getElementById("hud-ammo-mag").textContent = Math.floor(player.wood / 10);
                document.getElementById("hud-ammo-res").textContent = "PIECES";

                ["wall", "floor", "ramp"].forEach(p => {
                    const el = document.getElementById(`bpiece-${p}`);
                    if (!el) return;
                    if (p === player.buildPiece) {
                        el.classList.remove("border", "border-stone-700");
                        el.classList.add("border-2", "border-sky-400");
                    } else {
                        el.classList.remove("border-2", "border-sky-400");
                        el.classList.add("border", "border-stone-700");
                    }
                });
                return;
            }

            if (buildBar) { buildBar.classList.add("hidden"); buildBar.classList.remove("flex"); }
            if (weaponBar) { weaponBar.classList.remove("hidden"); }

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
            const aliveEnemies = bots.filter(b => b.alive).length;
            const isZombieMode = gameMode === "zombies";
            document.getElementById("hud-mode-badge").textContent = isZombieMode
                ? `DEAD SHOT (${aliveEnemies} Zombies)`
                : `DEAD SHOT BR (${aliveEnemies + (player.alive ? 1 : 0)} Alive)`;
            document.getElementById("menu-alive-count").textContent = `${aliveEnemies} REMAINING`;

            const list = document.getElementById("roster-list");
            if (!list) return;
            const allCombatants = [
                { name: player.name, alive: player.alive, kills: player.kills, isPlayer: true },
                ...bots.map(b => ({ name: b.name, alive: b.alive, kills: b.kills, isPlayer: false }))
            ].sort((a, b) => (b.alive - a.alive) || (b.kills - a.kills));

            list.innerHTML = allCombatants.map(c => `
                <div class="flex items-center justify-between px-2.5 py-1.5 rounded-lg ${c.alive ? "bg-stone-800/80" : "bg-stone-950/60 opacity-50"}">
                    <span class="font-bold ${c.isPlayer ? "text-lime-300" : "text-red-300"}">${c.name}</span>
                    <div class="flex items-center gap-2 mono text-[11px]">
                        ${(c.isPlayer || !isZombieMode) ? `<span class="text-amber-400">${c.kills} K</span>` : ""}
                        <span class="px-1.5 py-0.5 rounded text-[10px] font-bold ${c.alive ? (c.isPlayer ? "bg-emerald-500/20 text-emerald-300" : "bg-red-500/20 text-red-300") : "bg-stone-800 text-stone-400"}">
                            ${c.alive ? (c.isPlayer ? "ALIVE" : (isZombieMode ? "INFECTED" : "ACTIVE")) : "OUT"}
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
                if (inMainLobby) return;
                initAudio();
                keys[e.code] = true;

                if (e.code === "KeyQ") toggleBuildMode();
                else if (e.code === "KeyZ") selectBuildPiece("wall");
                else if (e.code === "KeyX") selectBuildPiece("floor");
                else if (e.code === "KeyC") selectBuildPiece("ramp");
                else if (e.code === "Digit1") { if (player.buildMode) toggleBuildMode(false); switchWeapon(0); }
                else if (e.code === "Digit2") { if (player.buildMode) toggleBuildMode(false); switchWeapon(1); }
                else if (e.code === "Digit3") { if (player.buildMode) toggleBuildMode(false); switchWeapon(2); }
                else if (e.code === "Digit4") { if (player.buildMode) toggleBuildMode(false); switchWeapon(3); }
                else if (e.code === "Digit5") { if (player.buildMode) toggleBuildMode(false); switchWeapon(4); }
                else if (e.code === "KeyH") { if (player.buildMode) toggleBuildMode(false); startMedkitHeal(); }
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
