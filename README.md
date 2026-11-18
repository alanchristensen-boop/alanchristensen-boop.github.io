<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>DEAD SHOT — 3D Survival & Battle Royale</title>
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
        <div class="glass-panel max-w-2xl w-full p-6 sm:p-8 rounded-3xl shadow-2xl border border-lime-400/35 max-h-[94vh] overflow-y-auto">
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

            <div class="grid grid-cols-1 md:grid-cols-2 gap-5 mt-5">
                <!-- Survivor Setup -->
                <div class="bg-stone-900/80 p-4 rounded-2xl border border-stone-700/70 space-y-3.5">
                    <div>
                        <label class="block text-[11px] font-extrabold uppercase tracking-wider text-lime-300 mb-1">Survivor Callsign</label>
                        <input id="lobby-player-name" type="text" maxlength="18" value="Survivor-01"
                               class="w-full px-3.5 py-2 rounded-xl bg-stone-950 border border-stone-700 focus:border-lime-400 focus:outline-none text-sm font-bold text-white" />
                    </div>
                    <div>
                        <label class="block text-[11px] font-extrabold uppercase tracking-wider text-purple-300 mb-1">Game Mode</label>
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
                        <label class="block text-[11px] font-extrabold uppercase tracking-wider text-cyan-300 mb-1">Building Mode</label>
                        <div class="grid grid-cols-2 gap-2">
                            <button type="button" onclick="setLobbyBuildMode(true)" id="build-opt-on"
                                    class="py-2 rounded-xl bg-lime-500 text-stone-950 font-black text-xs uppercase transition">
                                Build Mode
                            </button>
                            <button type="button" onclick="setLobbyBuildMode(false)" id="build-opt-off"
                                    class="py-2 rounded-xl bg-stone-800 text-stone-300 hover:bg-stone-700 font-extrabold text-xs uppercase transition">
                                Zero Build
                            </button>
                        </div>
                    </div>
                    <div>
                        <label class="block text-[11px] font-extrabold uppercase tracking-wider text-sky-300 mb-1">Deployment Map</label>
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
                        <label class="block text-[11px] font-extrabold uppercase tracking-wider text-amber-300 mb-1">Threat Level</label>
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
                        <div class="mt-3 pt-3 border-t border-stone-800 text-[11px] text-stone-400 space-y-1">
                            <div>• Spawn with <span class="text-white font-bold">M9 Pistol</span> & <span class="text-white font-bold">Combat Knife</span></div>
                            <div>• Loot <span class="text-amber-300 font-bold">Supply Chests [E]</span> for M4A1, Shotgun & Medkits</div>
                            <div>• Press <span class="text-sky-300 font-bold">[Q]</span> in Build Mode for Walls, Floors & Ramps</div>
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
                <span id="hud-submode-badge" class="mono text-[11px] text-red-400 font-bold">ZOMBIE SURVIVAL</span>
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
                <span class="flex items-center gap-1"><span class="w-2 h-2 rounded-full bg-red-500 inline-block"></span>Enemies</span>
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
                <span id="hud-controls-hint">[Q] Build • [H] Medkit</span>
            </div>
        </div>
    </div>

    <!-- Bottom-Right Weapon Loadout, Build Blueprints & Ammo HUD -->
    <div class="fixed bottom-4 right-4 z-20 flex flex-col items-end gap-2 pointer-events-none">
        <div class="flex items-center gap-2">
            <!-- Wood Materials Counter -->
            <div id="hud-wood-panel" class="glass-panel px-3 py-2.5 rounded-2xl text-right shadow-xl border border-amber-500/40">
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
                VICTORY ROYALE
            </div>
            <h2 id="end-banner-title" class="text-3xl font-black text-white tracking-tight">SECTOR CLEARED!</h2>
            <p id="end-banner-sub" class="text-sm text-stone-300 mt-2">You eliminated all enemies in Dead Shot.</p>
            <div class="mt-5 grid grid-cols-2 gap-3 bg-stone-900/80 p-3.5 rounded-2xl border border-stone-700/70">
                <div>
                    <div class="text-[11px] text-stone-400 uppercase font-bold">Eliminations</div>
                    <div id="end-stat-kills" class="text-2xl font-black text-amber-400 mono mt-0.5">0</div>
                </div>
                <div>
                    <div class="text-[11px] text-stone-400 uppercase font-bold">Placement</div>
                    <div id="end-stat-place" class="text-2xl font-black text-lime-400 mono mt-0.5">#1 / 20</div>
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
                    <p class="text-xs text-stone-400 mt-0.5">100 HP • 25 Headshot / 15 Body • Supply Chests & Building</p>
                </div>
                <button onclick="resumeGameplay()" class="px-4 py-2 rounded-xl bg-lime-500 hover:bg-lime-400 text-stone-950 font-black text-xs uppercase tracking-wider">
                    Resume
                </button>
            </div>

            <div class="grid grid-cols-1 md:grid-cols-2 gap-5 mt-5">
                <div class="bg-stone-900/75 p-4 rounded-2xl border border-stone-700/70 flex flex-col justify-between">
                    <div>
                        <h3 class="text-xs font-extrabold uppercase tracking-wider text-lime-300 mb-2">Controls</h3>
                        <ul class="space-y-1.5 text-xs text-stone-300">
                            <li><span class="mono text-lime-300 font-bold">W A S D</span> — Move • <span class="mono text-lime-300 font-bold">Shift</span> — Sprint • <span class="mono text-lime-300 font-bold">Space</span> — Jump</li>
                            <li><span class="mono text-lime-300 font-bold">Left Click / F</span> — Fire / Place Build • <span class="mono text-lime-300 font-bold">R</span> — Reload</li>
                            <li><span class="mono text-sky-300 font-bold">Q</span> — Toggle Build Mode (<span class="mono">Z</span> Wall, <span class="mono">X</span> Floor, <span class="mono">C</span> Ramp)</li>
                            <li><span class="mono text-amber-300 font-bold">E</span> — Open Supply Chests (Weapons, Medkits, Wood)</li>
                            <li><span class="mono text-red-300 font-bold">5 / H</span> — Apply Medkit (3.0s -> 100 HP)</li>
                            <li><span class="mono text-lime-300 font-bold">M</span> — Toggle Live Auto-Simulation</li>
                        </ul>
                    </div>
                    <div class="mt-4 flex gap-2">
                        <button onclick="resetSoloMatch()" class="flex-1 py-2.5 rounded-xl bg-emerald-600 hover:bg-emerald-500 text-white font-extrabold text-xs uppercase tracking-wider transition">
                            Restart Match
                        </button>
                        <button onclick="returnToMainLobby()" class="px-3.5 py-2.5 rounded-xl bg-stone-800 hover:bg-stone-700 text-stone-200 font-extrabold text-xs uppercase tracking-wider border border-stone-600 transition">
                            Lobby
                        </button>
                    </div>
                </div>

                <div class="bg-stone-900/75 p-4 rounded-2xl border border-stone-700/70">
                    <div class="flex items-center justify-between mb-2">
                        <h3 class="text-xs font-extrabold uppercase tracking-wider text-red-400">Enemy Roster (19)</h3>
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
        const ARENA_HALF_SIZE = 195;
        const PLAYER_MAX_HEALTH = 100;
        const BOT_MAX_HEALTH = 100;
        const HEADSHOT_DAMAGE = 25;
        const BODYSHOT_DAMAGE = 15;
        const BOT_HEADSHOT_DAMAGE = 30;
        const BOT_BODYSHOT_DAMAGE = 20;
        const ZOMBIE_CLAW_DAMAGE = 22;
        const BOT_COUNT = 19;
        const MEDKIT_DURATION = 3.0;

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

        let gameMode = "zombies";   // "zombies" or "classic"
        let buildEnabled = true;    // true = Build Mode, false = Zero Build
        let lobbyDifficulty = "normal";
        let selectedMap = "woodland";
        let inMainLobby = true;

        // =========================================================================
        // 2. AUDIO ENGINE (TONE.JS PROCEDURAL SYNTHS)
        // =========================================================================
        let audioReady = false;
        let rifleSynth, shotgunSynth, pistolSynth, knifeSynth, lootSynth, healSynth, hitSynth, buildSynth;

        function initAudio() {
            if (audioReady || typeof Tone === "undefined") return;
            try {
                Tone.start();
                rifleSynth = new Tone.NoiseSynth({
                    noise: { type: "white" },
                    envelope: { attack: 0.002, decay: 0.09, sustain: 0 }
                }).toDestination();
                rifleSynth.volume.value = -10;

                shotgunSynth = new Tone.NoiseSynth({
                    noise: { type: "brown" },
                    envelope: { attack: 0.005, decay: 0.24, sustain: 0 }
                }).toDestination();
                shotgunSynth.volume.value = -6;

                pistolSynth = new Tone.MembraneSynth({
                    pitchDecay: 0.02,
                    octaves: 3,
                    envelope: { attack: 0.001, decay: 0.08, sustain: 0 }
                }).toDestination();
                pistolSynth.volume.value = -10;

                knifeSynth = new Tone.NoiseSynth({
                    noise: { type: "pink" },
                    envelope: { attack: 0.01, decay: 0.07, sustain: 0 }
                }).toDestination();
                knifeSynth.volume.value = -14;

                lootSynth = new Tone.PolySynth(Tone.Synth).toDestination();
                lootSynth.volume.value = -12;

                healSynth = new Tone.Synth({
                    oscillator: { type: "sine" },
                    envelope: { attack: 0.05, decay: 0.3, sustain: 0.1, release: 0.4 }
                }).toDestination();
                healSynth.volume.value = -10;

                hitSynth = new Tone.Synth({
                    oscillator: { type: "triangle" },
                    envelope: { attack: 0.001, decay: 0.05, sustain: 0 }
                }).toDestination();
                hitSynth.volume.value = -8;

                buildSynth = new Tone.MembraneSynth({
                    pitchDecay: 0.03,
                    octaves: 2,
                    envelope: { attack: 0.002, decay: 0.09, sustain: 0 }
                }).toDestination();
                buildSynth.volume.value = -9;

                audioReady = true;
            } catch (e) {
                console.warn("Audio initialization skipped:", e);
            }
        }

        function playSound(type) {
            if (!audioReady) return;
            try {
                if (type === "rifle") rifleSynth.triggerAttackRelease("16n");
                else if (type === "shotgun") shotgunSynth.triggerAttackRelease("8n");
                else if (type === "pistol") pistolSynth.triggerAttackRelease("G2", "16n");
                else if (type === "knife") knifeSynth.triggerAttackRelease("32n");
                else if (type === "loot") lootSynth.triggerAttackRelease(["C5", "E5", "G5", "B5"], "16n");
                else if (type === "heal") healSynth.triggerAttackRelease("A5", "8n");
                else if (type === "hit") hitSynth.triggerAttackRelease("C6", "32n");
                else if (type === "build") buildSynth.triggerAttackRelease("D2", "16n");
            } catch (e) {}
        }

        // =========================================================================
        // 3. THREE.JS SCENE, LIGHTING, FIRST-PERSON VIEWMODELS & MAPS
        // =========================================================================
        let scene, camera, renderer, clock;
        let worldGroup;
        let weaponHolder, weaponMeshes = [];
        let blueprintViewModel, buildPreviewGroup;
        let colliders = [];
        let builtStructures = [];
        let chests = [];
        let bots = [];
        let tracers = [];
        let particles = [];

        const BUILD_GRID_SIZE = 4.0;
        const BUILD_WALL_HEIGHT = 3.6;

        function initThreeScene() {
            scene = new THREE.Scene();
            scene.background = new THREE.Color(0x88a898);
            scene.fog = new THREE.FogExp2(0x88a898, 0.0042);

            camera = new THREE.PerspectiveCamera(75, window.innerWidth / window.innerHeight, 0.1, 600);
            camera.rotation.order = "YXZ";

            renderer = new THREE.WebGLRenderer({ antialias: true, powerPreference: "high-performance" });
            renderer.setSize(window.innerWidth, window.innerHeight);
            renderer.setPixelRatio(Math.min(window.devicePixelRatio, 2));
            renderer.shadowMap.enabled = true;
            renderer.shadowMap.type = THREE.PCFSoftShadowMap;
            document.getElementById("game-canvas-container").appendChild(renderer.domElement);

            clock = new THREE.Clock();

            const hemiLight = new THREE.HemisphereLight(0xdff5e8, 0x3b4d36, 0.85);
            scene.add(hemiLight);

            const dirLight = new THREE.DirectionalLight(0xfff4d6, 1.05);
            dirLight.position.set(90, 150, 75);
            dirLight.castShadow = true;
            dirLight.shadow.mapSize.width = 2048;
            dirLight.shadow.mapSize.height = 2048;
            dirLight.shadow.camera.near = 10;
            dirLight.shadow.camera.far = 420;
            const d = 180;
            dirLight.shadow.camera.left = -d;
            dirLight.shadow.camera.right = d;
            dirLight.shadow.camera.top = d;
            dirLight.shadow.camera.bottom = -d;
            scene.add(dirLight);

            worldGroup = new THREE.Group();
            scene.add(worldGroup);

            weaponHolder = new THREE.Group();
            weaponHolder.position.set(0.28, -0.24, -0.45);
            camera.add(weaponHolder);
            scene.add(camera);

            buildFirstPersonViewModels();
            initBuildPreviewHologram();

            window.addEventListener("resize", () => {
                camera.aspect = window.innerWidth / window.innerHeight;
                camera.updateProjectionMatrix();
                renderer.setSize(window.innerWidth, window.innerHeight);
            });
        }

        function buildFirstPersonViewModels() {
            const darkMetal = new THREE.MeshStandardMaterial({ color: 0x1e2226, roughness: 0.4, metalness: 0.8 });
            const oliveBody = new THREE.MeshStandardMaterial({ color: 0x3a4736, roughness: 0.6, metalness: 0.3 });
            const woodMat = new THREE.MeshStandardMaterial({ color: 0x6b4226, roughness: 0.8 });
            const silverBlade = new THREE.MeshStandardMaterial({ color: 0xd4d8dd, roughness: 0.25, metalness: 0.9 });

            // Slot 0: M4A1 Carbine
            const m4 = new THREE.Group();
            const m4Receiver = new THREE.Mesh(new THREE.BoxGeometry(0.07, 0.09, 0.42), darkMetal);
            const m4Barrel = new THREE.Mesh(new THREE.CylinderGeometry(0.014, 0.014, 0.34, 8), darkMetal);
            m4Barrel.rotation.x = Math.PI / 2;
            m4Barrel.position.set(0, 0.015, -0.34);
            const m4Handguard = new THREE.Mesh(new THREE.BoxGeometry(0.06, 0.065, 0.24), oliveBody);
            m4Handguard.position.set(0, 0.01, -0.22);
            const m4Mag = new THREE.Mesh(new THREE.BoxGeometry(0.045, 0.16, 0.07), darkMetal);
            m4Mag.position.set(0, -0.09, -0.06);
            m4Mag.rotation.x = 0.2;
            m4.add(m4Receiver, m4Barrel, m4Handguard, m4Mag);
            weaponHolder.add(m4);
            weaponMeshes[0] = m4;

            // Slot 1: M870 Breacher Shotgun
            const m870 = new THREE.Group();
            const sgBody = new THREE.Mesh(new THREE.BoxGeometry(0.075, 0.085, 0.46), darkMetal);
            const sgBarrel = new THREE.Mesh(new THREE.CylinderGeometry(0.02, 0.02, 0.42, 10), darkMetal);
            sgBarrel.rotation.x = Math.PI / 2;
            sgBarrel.position.set(0, 0.02, -0.36);
            const sgPump = new THREE.Mesh(new THREE.BoxGeometry(0.082, 0.07, 0.18), woodMat);
            sgPump.position.set(0, -0.01, -0.28);
            m870.add(sgBody, sgBarrel, sgPump);
            weaponHolder.add(m870);
            weaponMeshes[1] = m870;

            // Slot 2: M9 Tactical Pistol
            const m9 = new THREE.Group();
            const pSlide = new THREE.Mesh(new THREE.BoxGeometry(0.055, 0.065, 0.24), darkMetal);
            const pGrip = new THREE.Mesh(new THREE.BoxGeometry(0.05, 0.13, 0.065), oliveBody);
            pGrip.position.set(0, -0.07, 0.06);
            pGrip.rotation.x = -0.22;
            m9.add(pSlide, pGrip);
            weaponHolder.add(m9);
            weaponMeshes[2] = m9;

            // Slot 3: Combat Knife
            const knife = new THREE.Group();
            const kHandle = new THREE.Mesh(new THREE.CylinderGeometry(0.022, 0.025, 0.14, 8), darkMetal);
            kHandle.rotation.x = Math.PI / 3;
            const kBlade = new THREE.Mesh(new THREE.BoxGeometry(0.012, 0.035, 0.22), silverBlade);
            kBlade.position.set(0, 0.06, -0.12);
            kBlade.rotation.x = -0.35;
            knife.add(kHandle, kBlade);
            weaponHolder.add(knife);
            weaponMeshes[3] = knife;

            // Slot 4: Red-and-White Fortnite Med Kit
            const medkit = new THREE.Group();
            const redBox = new THREE.Mesh(
                new THREE.BoxGeometry(0.24, 0.18, 0.16),
                new THREE.MeshStandardMaterial({ color: 0xdc2626, roughness: 0.45 })
            );
            const whiteCrossH = new THREE.Mesh(
                new THREE.BoxGeometry(0.11, 0.035, 0.17),
                new THREE.MeshStandardMaterial({ color: 0xffffff, roughness: 0.3 })
            );
            const whiteCrossV = new THREE.Mesh(
                new THREE.BoxGeometry(0.035, 0.11, 0.17),
                new THREE.MeshStandardMaterial({ color: 0xffffff, roughness: 0.3 })
            );
            const handleBar = new THREE.Mesh(new THREE.BoxGeometry(0.12, 0.025, 0.03), darkMetal);
            handleBar.position.set(0, 0.105, 0);
            medkit.add(redBox, whiteCrossH, whiteCrossV, handleBar);
            weaponHolder.add(medkit);
            weaponMeshes[4] = medkit;

            // Build Mode Blueprint & Pencil
            blueprintViewModel = new THREE.Group();
            const bpPaper = new THREE.Mesh(
                new THREE.BoxGeometry(0.24, 0.012, 0.19),
                new THREE.MeshStandardMaterial({ color: 0x0284c7, roughness: 0.5 })
            );
            bpPaper.rotation.set(0.35, -0.2, 0.1);
            const pencil = new THREE.Mesh(
                new THREE.CylinderGeometry(0.008, 0.008, 0.16, 6),
                new THREE.MeshStandardMaterial({ color: 0xfacc15, roughness: 0.4 })
            );
            pencil.position.set(0.08, 0.05, -0.02);
            pencil.rotation.set(0.4, 0, 0.5);
            blueprintViewModel.add(bpPaper, pencil);
            blueprintViewModel.visible = false;
            weaponHolder.add(blueprintViewModel);
        }

        function initBuildPreviewHologram() {
            buildPreviewGroup = new THREE.Group();
            const holoMat = new THREE.MeshBasicMaterial({
                color: 0x38bdf8,
                transparent: true,
                opacity: 0.38,
                side: THREE.DoubleSide
            });
            const holoMesh = new THREE.Mesh(new THREE.BoxGeometry(BUILD_GRID_SIZE, BUILD_WALL_HEIGHT, 0.25), holoMat);
            buildPreviewGroup.add(holoMesh);
            buildPreviewGroup.visible = false;
            scene.add(buildPreviewGroup);
        }

        function addBoxCollider(cx, cz, halfW, halfD, minY = 0, maxY = 50) {
            colliders.push({ cx, cz, halfW, halfD, minY, maxY });
        }

        function resolveArenaCollisions(pos) {
            const margin = 1.1;
            const limit = ARENA_HALF_SIZE - margin;
            if (pos.x < -limit) pos.x = -limit;
            if (pos.x > limit) pos.x = limit;
            if (pos.z < -limit) pos.z = -limit;
            if (pos.z > limit) pos.z = limit;

            const radius = 0.75;
            for (let i = 0; i < colliders.length; i++) {
                const c = colliders[i];
                if (pos.y > c.maxY - 0.15 || pos.y + 1.8 < c.minY) continue;
                const dx = pos.x - c.cx;
                const dz = pos.z - c.cz;
                const overlapX = (c.halfW + radius) - Math.abs(dx);
                const overlapZ = (c.halfD + radius) - Math.abs(dz);
                if (overlapX > 0 && overlapZ > 0) {
                    if (overlapX < overlapZ) {
                        pos.x += dx >= 0 ? overlapX : -overlapX;
                    } else {
                        pos.z += dz >= 0 ? overlapZ : -overlapZ;
                    }
                }
            }
        }

        function getSupportHeightAt(x, z, currentY) {
            let supportY = 0;
            for (let i = 0; i < builtStructures.length; i++) {
                const s = builtStructures[i];
                if (s.type === "floor") {
                    if (Math.abs(x - s.x) <= BUILD_GRID_SIZE * 0.52 && Math.abs(z - s.z) <= BUILD_GRID_SIZE * 0.52) {
                        if (currentY >= s.y - 0.5 && s.y > supportY) {
                            supportY = s.y;
                        }
                    }
                } else if (s.type === "ramp") {
                    const dx = x - s.x;
                    const dz = z - s.z;
                    const cos = Math.cos(-s.rotY);
                    const sin = Math.sin(-s.rotY);
                    const localX = dx * cos - dz * sin;
                    const localZ = dx * sin + dz * cos;
                    if (Math.abs(localX) <= BUILD_GRID_SIZE * 0.52 && Math.abs(localZ) <= BUILD_GRID_SIZE * 0.52) {
                        const t = THREE.MathUtils.clamp((localZ + BUILD_GRID_SIZE * 0.5) / BUILD_GRID_SIZE, 0, 1);
                        const rampH = s.baseY + t * BUILD_WALL_HEIGHT;
                        if (currentY >= rampH - 0.8 && rampH > supportY) {
                            supportY = rampH;
                        }
                    }
                }
            }
            return supportY;
        }

        function clearWorldMap() {
            while (worldGroup.children.length > 0) {
                worldGroup.remove(worldGroup.children[0]);
            }
            colliders = [];
        }

        function buildPerimeterWalls(wallColor) {
            const wallMat = new THREE.MeshStandardMaterial({ color: wallColor, roughness: 0.9 });
            const wallThick = 4;
            const wallHeight = 14;
            const span = ARENA_HALF_SIZE * 2 + wallThick;

            const north = new THREE.Mesh(new THREE.BoxGeometry(span, wallHeight, wallThick), wallMat);
            north.position.set(0, wallHeight / 2, -ARENA_HALF_SIZE);
            const south = new THREE.Mesh(new THREE.BoxGeometry(span, wallHeight, wallThick), wallMat);
            south.position.set(0, wallHeight / 2, ARENA_HALF_SIZE);
            const east = new THREE.Mesh(new THREE.BoxGeometry(wallThick, wallHeight, span), wallMat);
            east.position.set(ARENA_HALF_SIZE, wallHeight / 2, 0);
            const west = new THREE.Mesh(new THREE.BoxGeometry(wallThick, wallHeight, span), wallMat);
            west.position.set(-ARENA_HALF_SIZE, wallHeight / 2, 0);
            worldGroup.add(north, south, east, west);
        }

        function buildWoodlandArena() {
            clearWorldMap();
            scene.background = new THREE.Color(0x88a898);
            scene.fog = new THREE.FogExp2(0x88a898, 0.0042);

            const groundMat = new THREE.MeshStandardMaterial({ color: 0x3b5932, roughness: 0.95 });
            const ground = new THREE.Mesh(new THREE.PlaneGeometry(ARENA_HALF_SIZE * 2, ARENA_HALF_SIZE * 2), groundMat);
            ground.rotation.x = -Math.PI / 2;
            ground.receiveShadow = true;
            worldGroup.add(ground);

            buildPerimeterWalls(0x4b524d);

            const trunkMat = new THREE.MeshStandardMaterial({ color: 0x4d3423, roughness: 0.9 });
            const foliageMat = new THREE.MeshStandardMaterial({ color: 0x224221, roughness: 0.85 });
            const rockMat = new THREE.MeshStandardMaterial({ color: 0x696e6b, roughness: 0.9 });
            const cabinWallMat = new THREE.MeshStandardMaterial({ color: 0x6e4b30, roughness: 0.85 });
            const roofMat = new THREE.MeshStandardMaterial({ color: 0x2d3330, roughness: 0.8 });
            const sandbagMat = new THREE.MeshStandardMaterial({ color: 0x8f8263, roughness: 0.9 });

            let seed = 4219;
            const rand = () => {
                seed = (seed * 16807) % 2147483647;
                return (seed - 1) / 2147483646;
            };

            const cabinCoords = [
                [-65, -60], [72, -55], [-80, 68], [85, 78], [0, -110],
                [-125, -15], [130, 18], [15, 125], [-40, -145], [115, -120]
            ];
            cabinCoords.forEach(([cx, cz]) => {
                const w = 12, d = 10, h = 5.5;
                const cabin = new THREE.Mesh(new THREE.BoxGeometry(w, h, d), cabinWallMat);
                cabin.position.set(cx, h / 2, cz);
                cabin.castShadow = true;
                cabin.receiveShadow = true;
                const roof = new THREE.Mesh(new THREE.ConeGeometry(9.2, 3.8, 4), roofMat);
                roof.position.set(cx, h + 1.9, cz);
                roof.rotation.y = Math.PI / 4;
                roof.castShadow = true;
                worldGroup.add(cabin, roof);
                addBoxCollider(cx, cz, w / 2 + 0.4, d / 2 + 0.4);
            });

            for (let i = 0; i < 290; i++) {
                const tx = (rand() - 0.5) * (ARENA_HALF_SIZE * 1.88);
                const tz = (rand() - 0.5) * (ARENA_HALF_SIZE * 1.88);
                if (Math.hypot(tx, tz) < 16) continue;
                const scale = 0.85 + rand() * 0.95;
                const tree = new THREE.Group();
                const trunk = new THREE.Mesh(new THREE.CylinderGeometry(0.45 * scale, 0.7 * scale, 5.5 * scale, 7), trunkMat);
                trunk.position.y = 2.75 * scale;
                trunk.castShadow = true;
                const cone1 = new THREE.Mesh(new THREE.ConeGeometry(3.2 * scale, 6.5 * scale, 7), foliageMat);
                cone1.position.y = 7.2 * scale;
                cone1.castShadow = true;
                const cone2 = new THREE.Mesh(new THREE.ConeGeometry(2.4 * scale, 5.2 * scale, 7), foliageMat);
                cone2.position.y = 10.6 * scale;
                tree.add(trunk, cone1, cone2);
                tree.position.set(tx, 0, tz);
                worldGroup.add(tree);
                addBoxCollider(tx, tz, 0.85 * scale, 0.85 * scale);
            }

            for (let i = 0; i < 135; i++) {
                const rx = (rand() - 0.5) * (ARENA_HALF_SIZE * 1.85);
                const rz = (rand() - 0.5) * (ARENA_HALF_SIZE * 1.85);
                if (Math.hypot(rx, rz) < 15) continue;
                const rw = 2.2 + rand() * 3.4;
                const rh = 1.6 + rand() * 2.5;
                const rd = 2.0 + rand() * 3.2;
                const rock = new THREE.Mesh(new THREE.DodecahedronGeometry(Math.max(rw, rd) * 0.55, 0), rockMat);
                rock.scale.set(1, rh / Math.max(rw, rd), 1);
                rock.position.set(rx, rh * 0.4, rz);
                rock.castShadow = true;
                rock.receiveShadow = true;
                worldGroup.add(rock);
                addBoxCollider(rx, rz, rw * 0.46, rd * 0.46);
            }

            for (let i = 0; i < 95; i++) {
                const bx = (rand() - 0.5) * (ARENA_HALF_SIZE * 1.85);
                const bz = (rand() - 0.5) * (ARENA_HALF_SIZE * 1.85);
                if (Math.hypot(bx, bz) < 14) continue;
                if (i % 2 === 0) {
                    const rot = rand() > 0.5 ? 0 : Math.PI / 2;
                    const log = new THREE.Mesh(new THREE.CylinderGeometry(0.65, 0.75, 6.2, 8), trunkMat);
                    log.rotation.z = Math.PI / 2;
                    log.rotation.y = rot;
                    log.position.set(bx, 0.65, bz);
                    log.castShadow = true;
                    worldGroup.add(log);
                    addBoxCollider(bx, bz, rot === 0 ? 3.1 : 0.8, rot === 0 ? 0.8 : 3.1);
                } else {
                    const bunker = new THREE.Mesh(new THREE.BoxGeometry(4.4, 1.35, 1.2), sandbagMat);
                    bunker.position.set(bx, 0.67, bz);
                    bunker.castShadow = true;
                    worldGroup.add(bunker);
                    addBoxCollider(bx, bz, 2.2, 0.65);
                }
            }
        }

        function buildCityArena() {
            clearWorldMap();
            scene.background = new THREE.Color(0x707e8c);
            scene.fog = new THREE.FogExp2(0x707e8c, 0.0044);

            const asphaltMat = new THREE.MeshStandardMaterial({ color: 0x282b30, roughness: 0.92 });
            const ground = new THREE.Mesh(new THREE.PlaneGeometry(ARENA_HALF_SIZE * 2, ARENA_HALF_SIZE * 2), asphaltMat);
            ground.rotation.x = -Math.PI / 2;
            ground.receiveShadow = true;
            worldGroup.add(ground);

            buildPerimeterWalls(0x373c42);

            const stripeMat = new THREE.MeshBasicMaterial({ color: 0xeab308 });
            for (let offset = -120; offset <= 120; offset += 60) {
                const roadH = new THREE.Mesh(new THREE.PlaneGeometry(ARENA_HALF_SIZE * 1.9, 0.45), stripeMat);
                roadH.rotation.x = -Math.PI / 2;
                roadH.position.set(0, 0.03, offset);
                const roadV = new THREE.Mesh(new THREE.PlaneGeometry(0.45, ARENA_HALF_SIZE * 1.9), stripeMat);
                roadV.rotation.x = -Math.PI / 2;
                roadV.position.set(offset, 0.03, 0);
                worldGroup.add(roadH, roadV);
            }

            const bldgColors = [0x475569, 0x334155, 0x52525b, 0x3f3f46, 0x64748b];
            const windowMat = new THREE.MeshBasicMaterial({ color: 0xfef08a });
            const carColors = [0xdc2626, 0x2563eb, 0xe5e7eb, 0x16a34a, 0xd97706];
            const barrierMat = new THREE.MeshStandardMaterial({ color: 0x9ca3af, roughness: 0.85 });
            const dumpsterMat = new THREE.MeshStandardMaterial({ color: 0x15803d, roughness: 0.7 });
            const poleMat = new THREE.MeshStandardMaterial({ color: 0x1f2937, metalness: 0.7, roughness: 0.4 });

            let seed = 7831;
            const rand = () => {
                seed = (seed * 16807) % 2147483647;
                return (seed - 1) / 2147483646;
            };

            for (let gx = -150; gx <= 150; gx += 60) {
                for (let gz = -150; gz <= 150; gz += 60) {
                    if (Math.abs(gx) < 35 && Math.abs(gz) < 35) continue;
                    const bw = 18 + rand() * 14;
                    const bd = 18 + rand() * 14;
                    const bh = 22 + rand() * 34;
                    const bMat = new THREE.MeshStandardMaterial({
                        color: bldgColors[Math.floor(rand() * bldgColors.length)],
                        roughness: 0.75
                    });
                    const bldg = new THREE.Mesh(new THREE.BoxGeometry(bw, bh, bd), bMat);
                    bldg.position.set(gx, bh / 2, gz);
                    bldg.castShadow = true;
                    bldg.receiveShadow = true;
                    worldGroup.add(bldg);

                    const winBand = new THREE.Mesh(new THREE.BoxGeometry(bw + 0.3, 1.4, bd + 0.3), windowMat);
                    winBand.position.set(gx, 6.5, gz);
                    worldGroup.add(winBand);

                    addBoxCollider(gx, gz, bw / 2 + 0.4, bd / 2 + 0.4);
                }
            }

            for (let i = 0; i < 110; i++) {
                const vx = (rand() - 0.5) * (ARENA_HALF_SIZE * 1.85);
                const vz = (rand() - 0.5) * (ARENA_HALF_SIZE * 1.85);
                if (Math.hypot(vx, vz) < 16) continue;
                const isHorizontal = rand() > 0.5;
                const cw = isHorizontal ? 4.8 : 2.3;
                const cd = isHorizontal ? 2.3 : 4.8;
                const carMat = new THREE.MeshStandardMaterial({
                    color: carColors[Math.floor(rand() * carColors.length)],
                    roughness: 0.45,
                    metalness: 0.4
                });
                const chassis = new THREE.Mesh(new THREE.BoxGeometry(cw, 1.15, cd), carMat);
                chassis.position.set(vx, 0.7, vz);
                chassis.castShadow = true;
                const cabin = new THREE.Mesh(new THREE.BoxGeometry(cw * 0.6, 0.85, cd * 0.82), carMat);
                cabin.position.set(vx, 1.55, vz);
                worldGroup.add(chassis, cabin);
                addBoxCollider(vx, vz, cw / 2 + 0.2, cd / 2 + 0.2);
            }

            for (let i = 0; i < 120; i++) {
                const ox = (rand() - 0.5) * (ARENA_HALF_SIZE * 1.85);
                const oz = (rand() - 0.5) * (ARENA_HALF_SIZE * 1.85);
                if (Math.hypot(ox, oz) < 15) continue;
                if (i % 3 === 0) {
                    const barrier = new THREE.Mesh(new THREE.BoxGeometry(3.8, 1.3, 0.85), barrierMat);
                    barrier.position.set(ox, 0.65, oz);
                    barrier.castShadow = true;
                    worldGroup.add(barrier);
                    addBoxCollider(ox, oz, 1.95, 0.5);
                } else if (i % 3 === 1) {
                    const dumpster = new THREE.Mesh(new THREE.BoxGeometry(2.6, 1.75, 1.8), dumpsterMat);
                    dumpster.position.set(ox, 0.88, oz);
                    dumpster.castShadow = true;
                    worldGroup.add(dumpster);
                    addBoxCollider(ox, oz, 1.35, 0.95);
                } else {
                    const pole = new THREE.Mesh(new THREE.CylinderGeometry(0.16, 0.22, 8.5, 6), poleMat);
                    pole.position.set(ox, 4.25, oz);
                    const lamp = new THREE.Mesh(new THREE.BoxGeometry(1.1, 0.25, 0.45), windowMat);
                    lamp.position.set(ox + 0.45, 8.4, oz);
                    worldGroup.add(pole, lamp);
                    addBoxCollider(ox, oz, 0.35, 0.35);
                }
            }
        }

        function buildDesertArena() {
            clearWorldMap();
            scene.background = new THREE.Color(0xd9a76a);
            scene.fog = new THREE.FogExp2(0xd9a76a, 0.0038);

            const sandMat = new THREE.MeshStandardMaterial({ color: 0xc99655, roughness: 0.96 });
            const ground = new THREE.Mesh(new THREE.PlaneGeometry(ARENA_HALF_SIZE * 2, ARENA_HALF_SIZE * 2), sandMat);
            ground.rotation.x = -Math.PI / 2;
            ground.receiveShadow = true;
            worldGroup.add(ground);

            buildPerimeterWalls(0x8c5a32);

            const mesaMat = new THREE.MeshStandardMaterial({ color: 0xa65c32, roughness: 0.9 });
            const adobeMat = new THREE.MeshStandardMaterial({ color: 0xd6b07c, roughness: 0.88 });
            const cactusMat = new THREE.MeshStandardMaterial({ color: 0x2f6b32, roughness: 0.8 });
            const crateMat = new THREE.MeshStandardMaterial({ color: 0x855834, roughness: 0.82 });

            let seed = 9103;
            const rand = () => {
                seed = (seed * 16807) % 2147483647;
                return (seed - 1) / 2147483646;
            };

            const compounds = [
                [-68, -62], [75, -58], [-82, 72], [86, 80], [0, -112],
                [-128, -12], [128, 22], [12, 128], [-115, 118], [118, -115],
                [-45, 32], [48, -36]
            ];
            compounds.forEach(([cx, cz]) => {
                const w = 13, d = 11, h = 5.2;
                const house = new THREE.Mesh(new THREE.BoxGeometry(w, h, d), adobeMat);
                house.position.set(cx, h / 2, cz);
                house.castShadow = true;
                house.receiveShadow = true;
                const parapet = new THREE.Mesh(new THREE.BoxGeometry(w + 0.6, 0.9, d + 0.6), mesaMat);
                parapet.position.set(cx, h + 0.45, cz);
                worldGroup.add(house, parapet);
                addBoxCollider(cx, cz, w / 2 + 0.4, d / 2 + 0.4);
            });

            for (let i = 0; i < 120; i++) {
                const mx = (rand() - 0.5) * (ARENA_HALF_SIZE * 1.85);
                const mz = (rand() - 0.5) * (ARENA_HALF_SIZE * 1.85);
                if (Math.hypot(mx, mz) < 17) continue;
                const mw = 4 + rand() * 8;
                const md = 4 + rand() * 8;
                const mh = 3 + rand() * 9;
                const mesa = new THREE.Mesh(new THREE.BoxGeometry(mw, mh, md), mesaMat);
                mesa.position.set(mx, mh / 2, mz);
                mesa.castShadow = true;
                mesa.receiveShadow = true;
                worldGroup.add(mesa);
                addBoxCollider(mx, mz, mw / 2 + 0.3, md / 2 + 0.3);
            }

            for (let i = 0; i < 210; i++) {
                const cx = (rand() - 0.5) * (ARENA_HALF_SIZE * 1.88);
                const cz = (rand() - 0.5) * (ARENA_HALF_SIZE * 1.88);
                if (Math.hypot(cx, cz) < 14) continue;
                const h = 2.6 + rand() * 2.2;
                const cactus = new THREE.Group();
                const stem = new THREE.Mesh(new THREE.CylinderGeometry(0.32, 0.38, h, 7), cactusMat);
                stem.position.y = h / 2;
                stem.castShadow = true;
                const arm = new THREE.Mesh(new THREE.CylinderGeometry(0.22, 0.22, 1.2, 6), cactusMat);
                arm.position.set(0.45, h * 0.6, 0);
                arm.rotation.z = -Math.PI / 3;
                cactus.add(stem, arm);
                cactus.position.set(cx, 0, cz);
                worldGroup.add(cactus);
                addBoxCollider(cx, cz, 0.55, 0.55);
            }

            for (let i = 0; i < 115; i++) {
                const rx = (rand() - 0.5) * (ARENA_HALF_SIZE * 1.85);
                const rz = (rand() - 0.5) * (ARENA_HALF_SIZE * 1.85);
                if (Math.hypot(rx, rz) < 15) continue;
                if (i % 3 === 0) {
                    const ruin = new THREE.Mesh(new THREE.BoxGeometry(5.2, 2.2, 1.1), adobeMat);
                    ruin.position.set(rx, 1.1, rz);
                    ruin.castShadow = true;
                    worldGroup.add(ruin);
                    addBoxCollider(rx, rz, 2.65, 0.6);
                } else if (i % 3 === 1) {
                    const crate1 = new THREE.Mesh(new THREE.BoxGeometry(1.7, 1.7, 1.7), crateMat);
                    crate1.position.set(rx, 0.85, rz);
                    crate1.castShadow = true;
                    const crate2 = new THREE.Mesh(new THREE.BoxGeometry(1.4, 1.4, 1.4), crateMat);
                    crate2.position.set(rx + 0.15, 2.4, rz - 0.1);
                    worldGroup.add(crate1, crate2);
                    addBoxCollider(rx, rz, 0.95, 0.95);
                } else {
                    const pillar = new THREE.Mesh(new THREE.CylinderGeometry(0.85, 1.0, 4.5, 8), mesaMat);
                    pillar.position.set(rx, 2.25, rz);
                    pillar.castShadow = true;
                    worldGroup.add(pillar);
                    addBoxCollider(rx, rz, 1.0, 1.0);
                }
            }
        }

        function buildSelectedMap(mapName) {
            if (mapName === "city") buildCityArena();
            else if (mapName === "desert") buildDesertArena();
            else buildWoodlandArena();
        }

        // =========================================================================
        // 4. PLAYER STATE, INVENTORY, FORTNITE BUILDING & MEDKIT SYSTEM
        // =========================================================================
        const player = {
            name: "Survivor-01",
            pos: new THREE.Vector3(0, 0, 0),
            vel: new THREE.Vector3(0, 0, 0),
            yaw: 0,
            pitch: 0,
            health: PLAYER_MAX_HEALTH,
            alive: true,
            kills: 0,
            grounded: true,
            activeSlot: 2,
            fireCooldown: 0,
            reloading: false,
            reloadTimer: 0,
            medkits: 0,
            isHealing: false,
            healTimer: 0,
            wood: 150,
            buildMode: false,
            buildPiece: "wall",
            weapons: [
                { id: "m4", name: "M4A1 CARBINE", owned: false, mag: 30, magSize: 30, reserve: 150, fireRate: 0.11, auto: true, pellets: 1, spread: 0.014, range: 260, sound: "rifle" },
                { id: "shotgun", name: "M870 BREACHER", owned: false, mag: 6, magSize: 6, reserve: 36, fireRate: 0.78, auto: false, pellets: 6, spread: 0.065, range: 65, sound: "shotgun" },
                { id: "pistol", name: "M9 TACTICAL PISTOL", owned: true, mag: 15, magSize: 15, reserve: 90, fireRate: 0.22, auto: false, pellets: 1, spread: 0.018, range: 170, sound: "pistol" },
                { id: "knife", name: "COMBAT KNIFE", owned: true, mag: 1, magSize: 1, reserve: 0, fireRate: 0.42, auto: false, pellets: 1, spread: 0.02, range: 3.8, sound: "knife" },
                { id: "medkit", name: "FORTNITE MED KIT", owned: false, mag: 0, magSize: 1, reserve: 0, fireRate: 0.5, auto: false, pellets: 0, spread: 0, range: 0, sound: "heal" }
            ]
        };

        function switchWeaponSlot(slotIndex) {
            if (slotIndex < 0 || slotIndex > 4) return;
            const w = player.weapons[slotIndex];
            if (!w.owned && !(slotIndex === 4 && player.medkits > 0)) return;
            if (player.buildMode) toggleBuildMode(false);
            if (player.isHealing && slotIndex !== 4) cancelMedkitHeal();
            player.reloading = false;
            player.activeSlot = slotIndex;
            weaponMeshes.forEach((mesh, idx) => {
                if (mesh) mesh.visible = (idx === slotIndex);
            });
            updateHUD();
        }

        function toggleBuildMode(forceState) {
            if (!buildEnabled) {
                player.buildMode = false;
                if (blueprintViewModel) blueprintViewModel.visible = false;
                if (buildPreviewGroup) buildPreviewGroup.visible = false;
                return;
            }
            player.buildMode = typeof forceState === "boolean" ? forceState : !player.buildMode;
            if (player.buildMode) {
                if (player.isHealing) cancelMedkitHeal();
                player.reloading = false;
                weaponMeshes.forEach(m => { if (m) m.visible = false; });
                if (blueprintViewModel) blueprintViewModel.visible = true;
                if (buildPreviewGroup) buildPreviewGroup.visible = true;
                updateBuildPreviewMesh();
            } else {
                if (blueprintViewModel) blueprintViewModel.visible = false;
                if (buildPreviewGroup) buildPreviewGroup.visible = false;
                weaponMeshes.forEach((m, idx) => { if (m) m.visible = (idx === player.activeSlot); });
            }
            updateHUD();
        }

        function selectBuildPiece(piece) {
            if (!buildEnabled) return;
            player.buildPiece = piece;
            if (!player.buildMode) {
                toggleBuildMode(true);
            } else {
                updateBuildPreviewMesh();
                updateHUD();
            }
        }

        function computeBuildSnapTransform() {
            const forward = new THREE.Vector3(-Math.sin(player.yaw), 0, -Math.cos(player.yaw));
            const target = player.pos.clone().addScaledVector(forward, 3.6);
            const gx = Math.round(target.x / BUILD_GRID_SIZE) * BUILD_GRID_SIZE;
            const gz = Math.round(target.z / BUILD_GRID_SIZE) * BUILD_GRID_SIZE;
            const gy = Math.floor((player.pos.y + 0.5) / BUILD_WALL_HEIGHT) * BUILD_WALL_HEIGHT;
            const cardRot = Math.round(player.yaw / (Math.PI / 2)) * (Math.PI / 2);
            return { gx, gy, gz, cardRot };
        }

        function updateBuildPreviewMesh() {
            if (!buildPreviewGroup) return;
            while (buildPreviewGroup.children.length > 0) {
                buildPreviewGroup.remove(buildPreviewGroup.children[0]);
            }
            const holoMat = new THREE.MeshBasicMaterial({
                color: player.wood >= 10 ? 0x38bdf8 : 0xef4444,
                transparent: true,
                opacity: 0.42,
                side: THREE.DoubleSide
            });
            let geo;
            if (player.buildPiece === "wall") {
                geo = new THREE.BoxGeometry(BUILD_GRID_SIZE, BUILD_WALL_HEIGHT, 0.24);
            } else if (player.buildPiece === "floor") {
                geo = new THREE.BoxGeometry(BUILD_GRID_SIZE, 0.24, BUILD_GRID_SIZE);
            } else {
                const slopeLen = Math.hypot(BUILD_GRID_SIZE, BUILD_WALL_HEIGHT);
                geo = new THREE.BoxGeometry(BUILD_GRID_SIZE, 0.24, slopeLen);
            }
            const mesh = new THREE.Mesh(geo, holoMat);
            if (player.buildPiece === "ramp") {
                mesh.rotation.x = Math.atan2(BUILD_WALL_HEIGHT, BUILD_GRID_SIZE);
            }
            buildPreviewGroup.add(mesh);
        }

        function updateBuildPreviewPosition() {
            if (!player.buildMode || !buildPreviewGroup) return;
            const snap = computeBuildSnapTransform();
            buildPreviewGroup.rotation.set(0, snap.cardRot, 0);
            if (player.buildPiece === "wall") {
                buildPreviewGroup.position.set(snap.gx, snap.gy + BUILD_WALL_HEIGHT / 2, snap.gz);
            } else if (player.buildPiece === "floor") {
                buildPreviewGroup.position.set(snap.gx, snap.gy + 0.12, snap.gz);
            } else {
                buildPreviewGroup.position.set(snap.gx, snap.gy + BUILD_WALL_HEIGHT / 2, snap.gz);
            }
        }

        function placeFortniteStructure() {
            if (!buildEnabled || !player.buildMode || player.wood < 10 || !player.alive) return;
            const snap = computeBuildSnapTransform();
            const dup = builtStructures.some(s =>
                s.type === player.buildPiece &&
                Math.abs(s.x - snap.gx) < 0.2 &&
                Math.abs(s.z - snap.gz) < 0.2 &&
                Math.abs(s.baseY - snap.gy) < 0.2
            );
            if (dup) return;

            player.wood -= 10;
            playSound("build");
            updateWoodHUD();
            updateBuildPreviewMesh();

            const plankMat = new THREE.MeshStandardMaterial({ color: 0xa16207, roughness: 0.78 });
            const group = new THREE.Group();
            group.rotation.y = snap.cardRot;

            if (player.buildPiece === "wall") {
                const wall = new THREE.Mesh(new THREE.BoxGeometry(BUILD_GRID_SIZE, BUILD_WALL_HEIGHT, 0.26), plankMat);
                wall.castShadow = true;
                wall.receiveShadow = true;
                group.add(wall);
                group.position.set(snap.gx, snap.gy + BUILD_WALL_HEIGHT / 2, snap.gz);
                const isZAligned = Math.abs(Math.sin(snap.cardRot)) > 0.5;
                addBoxCollider(
                    snap.gx,
                    snap.gz,
                    isZAligned ? 0.35 : BUILD_GRID_SIZE / 2,
                    isZAligned ? BUILD_GRID_SIZE / 2 : 0.35,
                    snap.gy,
                    snap.gy + BUILD_WALL_HEIGHT
                );
                builtStructures.push({ type: "wall", x: snap.gx, z: snap.gz, baseY: snap.gy, y: snap.gy + BUILD_WALL_HEIGHT, rotY: snap.cardRot, mesh: group });
            } else if (player.buildPiece === "floor") {
                const floor = new THREE.Mesh(new THREE.BoxGeometry(BUILD_GRID_SIZE, 0.26, BUILD_GRID_SIZE), plankMat);
                floor.castShadow = true;
                floor.receiveShadow = true;
                group.add(floor);
                const topY = snap.gy + 0.26;
                group.position.set(snap.gx, snap.gy + 0.13, snap.gz);
                builtStructures.push({ type: "floor", x: snap.gx, z: snap.gz, baseY: snap.gy, y: topY, rotY: snap.cardRot, mesh: group });
            } else {
                const slopeLen = Math.hypot(BUILD_GRID_SIZE, BUILD_WALL_HEIGHT);
                const ramp = new THREE.Mesh(new THREE.BoxGeometry(BUILD_GRID_SIZE, 0.26, slopeLen), plankMat);
                ramp.rotation.x = Math.atan2(BUILD_WALL_HEIGHT, BUILD_GRID_SIZE);
                ramp.castShadow = true;
                ramp.receiveShadow = true;
                group.add(ramp);
                group.position.set(snap.gx, snap.gy + BUILD_WALL_HEIGHT / 2, snap.gz);
                builtStructures.push({ type: "ramp", x: snap.gx, z: snap.gz, baseY: snap.gy, y: snap.gy + BUILD_WALL_HEIGHT, rotY: snap.cardRot, mesh: group });
            }

            scene.add(group);
            weaponHolder.position.y += 0.04;
        }

        function updateWoodHUD() {
            const el = document.getElementById("hud-wood-count");
            if (el) el.textContent = player.wood;
        }

        function startMedkitHeal() {
            if (!player.alive || player.isHealing || player.medkits <= 0 || player.health >= PLAYER_MAX_HEALTH) return;
            switchWeaponSlot(4);
            player.isHealing = true;
            player.healTimer = MEDKIT_DURATION;
            document.getElementById("medkit-progress-wrap").classList.remove("hidden");
        }

        function cancelMedkitHeal() {
            player.isHealing = false;
            player.healTimer = 0;
            document.getElementById("medkit-progress-wrap").classList.add("hidden");
            document.getElementById("medkit-progress-bar").style.width = "0%";
        }

        function completeMedkitHeal() {
            player.isHealing = false;
            player.medkits = Math.max(0, player.medkits - 1);
            player.weapons[4].mag = player.medkits;
            if (player.medkits === 0) player.weapons[4].owned = false;
            player.health = PLAYER_MAX_HEALTH;
            document.getElementById("medkit-progress-wrap").classList.add("hidden");
            playSound("heal");
            addKillFeedEntry(`${player.name} applied a Med Kit (+100 HP)`, "text-emerald-300");
            if (player.medkits === 0) switchWeaponSlot(player.weapons[0].owned ? 0 : 2);
            updateHUD();
        }

        // =========================================================================
        // 5. LOOT CHEST SYSTEM
        // =========================================================================
        function spawnSupplyChests() {
            chests.forEach(c => scene.remove(c.group));
            chests = [];

            const chestCoords = [
                [0, -14], [-35, -35], [38, -32], [-42, 40], [45, 42],
                [-65, -53], [72, -48], [-80, 75], [85, 85], [0, -103],
                [-115, -85], [115, -90], [-125, 105], [120, 110], [0, 135],
                [-150, 0], [150, 0], [-70, 145], [75, -145], [-145, -140],
                [145, 140], [-20, 78], [24, -75]
            ];

            const baseMat = new THREE.MeshStandardMaterial({ color: 0xb45309, roughness: 0.45, metalness: 0.35 });
            const trimMat = new THREE.MeshStandardMaterial({ color: 0xfacc15, roughness: 0.3, metalness: 0.8, emissive: 0x6b4f00 });

            chestCoords.forEach(([x, z], idx) => {
                const group = new THREE.Group();
                const box = new THREE.Mesh(new THREE.BoxGeometry(1.4, 0.85, 0.95), baseMat);
                box.position.y = 0.425;
                box.castShadow = true;

                const lid = new THREE.Mesh(new THREE.BoxGeometry(1.46, 0.28, 1.0), trimMat);
                lid.position.y = 0.96;

                const beacon = new THREE.Mesh(
                    new THREE.CylinderGeometry(0.12, 0.35, 5.5, 8, 1, true),
                    new THREE.MeshBasicMaterial({ color: 0xfde047, transparent: true, opacity: 0.22, side: THREE.DoubleSide })
                );
                beacon.position.y = 3.2;

                group.add(box, lid, beacon);
                group.position.set(x, 0, z);
                scene.add(group);

                chests.push({
                    id: idx,
                    pos: new THREE.Vector3(x, 0, z),
                    group,
                    lid,
                    beacon,
                    opened: false
                });
            });
        }

        function openChest(chest) {
            if (chest.opened) return;
            chest.opened = true;
            chest.lid.rotation.x = -0.65;
            chest.lid.position.y = 1.12;
            chest.beacon.visible = false;
            playSound("loot");

            const roll = chest.id % 3;
            let lootMsg = "";
            if (roll === 0) {
                player.weapons[0].owned = true;
                player.weapons[0].mag = 30;
                player.weapons[0].reserve = Math.min(240, player.weapons[0].reserve + 90);
                lootMsg = "M4A1 Carbine + 90 Ammo";
                if (player.activeSlot === 2 || player.activeSlot === 3) switchWeaponSlot(0);
            } else if (roll === 1) {
                player.weapons[1].owned = true;
                player.weapons[1].mag = 6;
                player.weapons[1].reserve = Math.min(60, player.weapons[1].reserve + 18);
                lootMsg = "M870 Breacher Shotgun + 18 Shells";
                if (!player.weapons[0].owned && (player.activeSlot === 2 || player.activeSlot === 3)) switchWeaponSlot(1);
            } else {
                player.medkits += 1;
                player.weapons[4].owned = true;
                player.weapons[4].mag = player.medkits;
                player.weapons[0].reserve += 30;
                player.weapons[2].reserve += 30;
                lootMsg = "Fortnite Med Kit (+1) & Ammo";
            }

            if (buildEnabled) {
                player.wood = Math.min(999, player.wood + 60);
                updateWoodHUD();
                lootMsg += " + 60 Wood";
            }

            addKillFeedEntry(`Opened Chest: ${lootMsg}`, "text-amber-300");
            updateHUD();
        }

        // =========================================================================
        // 6. ENEMY ENTITIES: 3D ZOMBIES & MILITARY BOTS (100 HP, NO RESPAWNS)
        // =========================================================================
        function createEnemyMesh(isZombie) {
            const root = new THREE.Group();

            if (isZombie) {
                const skinMat = new THREE.MeshStandardMaterial({ color: 0x4d7c47, roughness: 0.85 });
                const darkSkinMat = new THREE.MeshStandardMaterial({ color: 0x365932, roughness: 0.9 });
                const tatteredShirtMat = new THREE.MeshStandardMaterial({ color: 0x3b3532, roughness: 0.95 });
                const tatteredPantsMat = new THREE.MeshStandardMaterial({ color: 0x262930, roughness: 0.95 });
                const eyeGlowMat = new THREE.MeshBasicMaterial({ color: 0xff2a2a });

                const leftLeg = new THREE.Mesh(new THREE.BoxGeometry(0.24, 0.78, 0.24), tatteredPantsMat);
                leftLeg.position.set(-0.16, 0.39, 0);
                leftLeg.castShadow = true;

                const rightLeg = new THREE.Mesh(new THREE.BoxGeometry(0.24, 0.78, 0.24), tatteredPantsMat);
                rightLeg.position.set(0.16, 0.39, 0);
                rightLeg.castShadow = true;

                const torso = new THREE.Mesh(new THREE.BoxGeometry(0.62, 0.76, 0.34), tatteredShirtMat);
                torso.position.set(0, 1.14, 0.04);
                torso.rotation.x = 0.14;
                torso.castShadow = true;

                const head = new THREE.Mesh(new THREE.BoxGeometry(0.42, 0.44, 0.42), skinMat);
                head.position.set(0, 1.72, 0.1);
                head.rotation.x = 0.12;
                head.castShadow = true;

                const leftEye = new THREE.Mesh(new THREE.BoxGeometry(0.08, 0.05, 0.04), eyeGlowMat);
                leftEye.position.set(-0.1, 1.75, 0.31);
                const rightEye = new THREE.Mesh(new THREE.BoxGeometry(0.08, 0.05, 0.04), eyeGlowMat);
                rightEye.position.set(0.1, 1.75, 0.31);

                const leftArm = new THREE.Mesh(new THREE.BoxGeometry(0.18, 0.18, 0.78), darkSkinMat);
                leftArm.position.set(-0.38, 1.32, 0.36);
                leftArm.rotation.x = -0.15;
                leftArm.castShadow = true;

                const rightArm = new THREE.Mesh(new THREE.BoxGeometry(0.18, 0.18, 0.78), darkSkinMat);
                rightArm.position.set(0.38, 1.32, 0.36);
                rightArm.rotation.x = -0.15;
                rightArm.castShadow = true;

                root.add(leftLeg, rightLeg, torso, head, leftEye, rightEye, leftArm, rightArm);
                return { root, head, torso, leftLeg, rightLeg, leftArm, rightArm };
            } else {
                // Full Military Uniform for Classic Battle Royale Bots
                const oliveTunicMat = new THREE.MeshStandardMaterial({ color: 0x3b4d34, roughness: 0.75 });
                const tacticalVestMat = new THREE.MeshStandardMaterial({ color: 0x262e23, roughness: 0.65 });
                const cargoPantsMat = new THREE.MeshStandardMaterial({ color: 0x4b533c, roughness: 0.8 });
                const helmetMat = new THREE.MeshStandardMaterial({ color: 0x2d3a27, roughness: 0.5, metalness: 0.3 });
                const visorMat = new THREE.MeshStandardMaterial({ color: 0x111827, roughness: 0.2, metalness: 0.85 });
                const bootMat = new THREE.MeshStandardMaterial({ color: 0x18181b, roughness: 0.7 });
                const gunMat = new THREE.MeshStandardMaterial({ color: 0x18181b, metalness: 0.8, roughness: 0.3 });

                const leftLeg = new THREE.Mesh(new THREE.BoxGeometry(0.24, 0.62, 0.24), cargoPantsMat);
                leftLeg.position.set(-0.16, 0.47, 0);
                leftLeg.castShadow = true;

                const rightLeg = new THREE.Mesh(new THREE.BoxGeometry(0.24, 0.62, 0.24), cargoPantsMat);
                rightLeg.position.set(0.16, 0.47, 0);
                rightLeg.castShadow = true;

                const leftBoot = new THREE.Mesh(new THREE.BoxGeometry(0.26, 0.18, 0.3), bootMat);
                leftBoot.position.set(-0.16, 0.09, 0.02);
                const rightBoot = new THREE.Mesh(new THREE.BoxGeometry(0.26, 0.18, 0.3), bootMat);
                rightBoot.position.set(0.16, 0.09, 0.02);

                const torso = new THREE.Mesh(new THREE.BoxGeometry(0.62, 0.76, 0.34), oliveTunicMat);
                torso.position.set(0, 1.16, 0);
                torso.castShadow = true;

                const vest = new THREE.Mesh(new THREE.BoxGeometry(0.66, 0.56, 0.39), tacticalVestMat);
                vest.position.set(0, 1.2, 0);

                const head = new THREE.Mesh(new THREE.BoxGeometry(0.42, 0.42, 0.42), helmetMat);
                head.position.set(0, 1.76, 0);
                head.castShadow = true;

                const visor = new THREE.Mesh(new THREE.BoxGeometry(0.36, 0.14, 0.05), visorMat);
                visor.position.set(0, 1.78, 0.21);

                const gun = new THREE.Mesh(new THREE.BoxGeometry(0.1, 0.12, 0.65), gunMat);
                gun.position.set(0.36, 1.22, 0.28);

                root.add(leftLeg, rightLeg, leftBoot, rightBoot, torso, vest, head, visor, gun);
                return { root, head, torso, leftLeg, rightLeg, leftArm: null, rightArm: null };
            }
        }

        function spawnAllBots() {
            bots.forEach(b => scene.remove(b.mesh.root));
            bots = [];

            const isZombieMode = gameMode === "zombies";
            const names = isZombieMode ? ZOMBIE_NAMES : BOT_CALLSIGNS;

            for (let i = 0; i < BOT_COUNT; i++) {
                const angle = (i / BOT_COUNT) * Math.PI * 2 + 0.25;
                const radius = 58 + (i % 4) * 32;
                const bx = Math.cos(angle) * radius;
                const bz = Math.sin(angle) * radius;

                const mesh = createEnemyMesh(isZombieMode);
                mesh.root.position.set(bx, 0, bz);
                scene.add(mesh.root);

                bots.push({
                    id: i,
                    name: names[i],
                    isZombie: isZombieMode,
                    pos: new THREE.Vector3(bx, 0, bz),
                    yaw: Math.random() * Math.PI * 2,
                    health: BOT_MAX_HEALTH,
                    alive: true,
                    kills: 0,
                    fireCooldown: 0.7 + Math.random() * 1.1,
                    strafeDir: Math.random() > 0.5 ? 1 : -1,
                    strafeTimer: 1.5 + Math.random() * 2,
                    walkPhase: Math.random() * Math.PI * 2,
                    mesh
                });
            }
        }

        // =========================================================================
        // 7. INPUT HANDLING & SANDBOX-SAFE POINTER LOCK / FREE-AIM MOUSE LOOK
        // =========================================================================
        const keys = {};
        let isPointerLocked = false;
        let freeAimActive = false;
        let isMouseDown = false;
        let lastMouseX = null;
        let lastMouseY = null;
        let isPaused = false;
        let simModeActive = false;
        let simSpeed = 1;

        function updateAimBadge() {
            const el = document.getElementById("hud-aim-status");
            if (!el) return;
            if (isPointerLocked) {
                el.textContent = "Cursor Locked • Move mouse to aim • ESC to release";
                el.className = "text-lime-400 font-semibold";
            } else if (freeAimActive) {
                el.textContent = "Free-Aim Active • Move mouse or drag to turn";
                el.className = "text-emerald-300 font-semibold";
            } else {
                el.textContent = "Click screen to activate aim (or drag mouse)";
                el.className = "text-amber-300 font-semibold";
            }
        }

        function engageLookControl() {
            if (isPaused || inMainLobby) return;
            freeAimActive = true;
            updateAimBadge();
            const canvas = renderer.domElement;
            if (canvas && canvas.requestPointerLock) {
                try {
                    const promise = canvas.requestPointerLock();
                    if (promise && typeof promise.catch === "function") {
                        promise.catch(() => {
                            isPointerLocked = false;
                            updateAimBadge();
                        });
                    }
                } catch (e) {
                    isPointerLocked = false;
                    updateAimBadge();
                }
            }
        }

        function unlockLookControl() {
            freeAimActive = false;
            isPointerLocked = false;
            if (document.pointerLockElement && document.exitPointerLock) {
                try { document.exitPointerLock(); } catch (e) {}
            }
            updateAimBadge();
        }

        document.addEventListener("pointerlockchange", () => {
            isPointerLocked = (document.pointerLockElement === renderer.domElement);
            if (isPointerLocked) freeAimActive = true;
            updateAimBadge();
        });

        document.addEventListener("pointerlockerror", () => {
            isPointerLocked = false;
            freeAimActive = true;
            updateAimBadge();
        });

        window.addEventListener("mousedown", (e) => {
            if (inMainLobby) return;
            initAudio();
            if (isPaused) return;
            if (e.target.closest("button")) return;

            if (!isPointerLocked && !freeAimActive) {
                engageLookControl();
            }

            if (e.button === 0) {
                isMouseDown = true;
                if (!simModeActive) {
                    if (player.buildMode) placeFortniteStructure();
                    else attemptPlayerAttack();
                }
            } else if (e.button === 2) {
                freeAimActive = true;
                updateAimBadge();
            }
        });

        window.addEventListener("mouseup", (e) => {
            if (e.button === 0) isMouseDown = false;
        });

        window.addEventListener("contextmenu", (e) => e.preventDefault());

        window.addEventListener("mousemove", (e) => {
            if (isPaused || simModeActive || inMainLobby) return;
            const sensitivity = 0.0024;
            if (isPointerLocked) {
                player.yaw -= e.movementX * sensitivity;
                player.pitch = THREE.MathUtils.clamp(player.pitch - e.movementY * sensitivity, -1.42, 1.42);
            } else if (freeAimActive || isMouseDown) {
                if (lastMouseX !== null && lastMouseY !== null) {
                    const dx = e.clientX - lastMouseX;
                    const dy = e.clientY - lastMouseY;
                    player.yaw -= dx * sensitivity * 1.15;
                    player.pitch = THREE.MathUtils.clamp(player.pitch - dy * sensitivity * 1.15, -1.42, 1.42);
                }
            }
            lastMouseX = e.clientX;
            lastMouseY = e.clientY;
        });

        let touchLookId = null;
        let lastTouchX = 0;
        let lastTouchY = 0;
        window.addEventListener("touchstart", (e) => {
            if (inMainLobby) return;
            initAudio();
            if (isPaused || e.target.closest("button")) return;
            const t = e.changedTouches[0];
            touchLookId = t.identifier;
            lastTouchX = t.clientX;
            lastTouchY = t.clientY;
            if (!simModeActive) {
                if (player.buildMode) placeFortniteStructure();
                else attemptPlayerAttack();
            }
        }, { passive: true });

        window.addEventListener("touchmove", (e) => {
            if (isPaused || simModeActive || inMainLobby) return;
            for (let i = 0; i < e.changedTouches.length; i++) {
                const t = e.changedTouches[i];
                if (t.identifier === touchLookId) {
                    const dx = t.clientX - lastTouchX;
                    const dy = t.clientY - lastTouchY;
                    player.yaw -= dx * 0.0045;
                    player.pitch = THREE.MathUtils.clamp(player.pitch - dy * 0.0045, -1.42, 1.42);
                    lastTouchX = t.clientX;
                    lastTouchY = t.clientY;
                }
            }
        }, { passive: true });

        window.addEventListener("keydown", (e) => {
            if (inMainLobby) return;
            const code = e.code;
            keys[code] = true;

            if (code === "KeyP" || code === "Escape") {
                if (isPaused) resumeGameplay();
                else openPauseMenu();
                return;
            }
            if (isPaused) return;

            if (code === "Digit1") switchWeaponSlot(0);
            else if (code === "Digit2") switchWeaponSlot(1);
            else if (code === "Digit3") switchWeaponSlot(2);
            else if (code === "Digit4") switchWeaponSlot(3);
            else if (code === "Digit5" || code === "KeyH") startMedkitHeal();
            else if (code === "KeyQ") toggleBuildMode();
            else if (code === "KeyZ") selectBuildPiece("wall");
            else if (code === "KeyX") selectBuildPiece("floor");
            else if (code === "KeyC") selectBuildPiece("ramp");
            else if (code === "KeyR") triggerReload();
            else if (code === "KeyE") tryInteractChest();
            else if (code === "KeyF") {
                if (player.buildMode) placeFortniteStructure();
                else attemptPlayerAttack();
            }
            else if (code === "KeyM") toggleSimMode();
        });

        window.addEventListener("keyup", (e) => {
            keys[e.code] = false;
        });

        // =========================================================================
        // 8. BALLISTICS, HITSCAN, TRACERS & COMBAT RESOLUTION
        // =========================================================================
        function triggerReload() {
            if (player.buildMode) return;
            const w = player.weapons[player.activeSlot];
            if (!w || w.id === "knife" || w.id === "medkit") return;
            if (player.reloading || w.mag >= w.magSize || w.reserve <= 0) return;
            player.reloading = true;
            player.reloadTimer = 1.15;
            updateHUD();
        }

        function spawnTracer(start, end, colorHex) {
            const points = [start.clone(), end.clone()];
            const geo = new THREE.BufferGeometry().setFromPoints(points);
            const mat = new THREE.LineBasicMaterial({ color: colorHex, transparent: true, opacity: 0.85 });
            const line = new THREE.Line(geo, mat);
            scene.add(line);
            tracers.push({ line, life: 0.12 });
        }

        function spawnImpactSparks(pos, colorHex) {
            for (let i = 0; i < 6; i++) {
                const spark = new THREE.Mesh(
                    new THREE.BoxGeometry(0.08, 0.08, 0.08),
                    new THREE.MeshBasicMaterial({ color: colorHex })
                );
                spark.position.copy(pos);
                const vel = new THREE.Vector3(
                    (Math.random() - 0.5) * 7,
                    Math.random() * 5 + 1,
                    (Math.random() - 0.5) * 7
                );
                scene.add(spark);
                particles.push({ mesh: spark, vel, life: 0.28 });
            }
        }

        function flashHitmarker() {
            const hm = document.getElementById("hitmarker");
            hm.style.opacity = "1";
            playSound("hit");
            setTimeout(() => { hm.style.opacity = "0"; }, 95);
        }

        function attemptPlayerAttack() {
            if (!player.alive || player.reloading || player.fireCooldown > 0) return;
            if (player.buildMode) {
                placeFortniteStructure();
                return;
            }
            const w = player.weapons[player.activeSlot];
            if (!w || !w.owned) return;

            if (w.id === "medkit") {
                startMedkitHeal();
                return;
            }

            if (player.isHealing) cancelMedkitHeal();

            if (w.id !== "knife") {
                if (w.mag <= 0) {
                    triggerReload();
                    return;
                }
                w.mag -= 1;
                if (w.mag === 0 && w.reserve > 0) {
                    setTimeout(() => triggerReload(), 120);
                }
            }

            player.fireCooldown = w.fireRate;
            playSound(w.sound);
            weaponHolder.position.z = -0.36;
            weaponHolder.rotation.x = 0.14;

            const origin = camera.position.clone();
            const forward = new THREE.Vector3(0, 0, -1).applyQuaternion(camera.quaternion);
            const muzzlePos = origin.clone().addScaledVector(forward, 0.75).add(new THREE.Vector3(0, -0.12, 0));

            const headTargets = [];
            const bodyTargets = [];
            bots.forEach(b => {
                if (!b.alive) return;
                b.mesh.head.userData = { bot: b, isHead: true };
                b.mesh.torso.userData = { bot: b, isHead: false };
                b.mesh.leftLeg.userData = { bot: b, isHead: false };
                b.mesh.rightLeg.userData = { bot: b, isHead: false };
                headTargets.push(b.mesh.head);
                bodyTargets.push(b.mesh.torso, b.mesh.leftLeg, b.mesh.rightLeg);
            });
            const allTargets = [...headTargets, ...bodyTargets];

            for (let p = 0; p < w.pellets; p++) {
                const dir = forward.clone();
                dir.x += (Math.random() - 0.5) * w.spread;
                dir.y += (Math.random() - 0.5) * w.spread;
                dir.z += (Math.random() - 0.5) * w.spread;
                dir.normalize();

                const raycaster = new THREE.Raycaster(origin, dir, 0.2, w.range);
                const hits = raycaster.intersectObjects(allTargets, false);

                if (hits.length > 0) {
                    const hit = hits[0];
                    const { bot, isHead } = hit.object.userData;
                    if (w.id !== "knife") spawnTracer(muzzlePos, hit.point, 0xfef08a);
                    spawnImpactSparks(hit.point, isHead ? 0xef4444 : 0xfacc15);
                    flashHitmarker();
                    applyDamageToBot(bot, isHead ? HEADSHOT_DAMAGE : BODYSHOT_DAMAGE, player.name, true, isHead);
                } else {
                    const endPt = origin.clone().addScaledVector(dir, Math.min(w.range, 120));
                    if (w.id !== "knife") spawnTracer(muzzlePos, endPt, 0xfde047);
                }
            }

            updateHUD();
        }

        function applyDamageToBot(bot, amount, attackerName, byPlayer, isHead) {
            if (!bot.alive) return;
            bot.health = Math.max(0, bot.health - amount);
            if (bot.health === 0) {
                bot.alive = false;
                bot.mesh.root.visible = false;
                if (byPlayer) {
                    player.kills += 1;
                }
                const tag = isHead ? " [HEADSHOT]" : "";
                addKillFeedEntry(`${attackerName} eliminated ${bot.name}${tag}`, byPlayer ? "text-lime-300" : "text-stone-300");
                updateHUD();
                updateRosterUI();
                checkMatchVictory();
            }
        }

        function applyDamageToPlayer(amount, attackerName) {
            if (!player.alive) return;
            if (player.isHealing) cancelMedkitHeal();
            player.health = Math.max(0, player.health - amount);

            const flash = document.getElementById("damage-flash");
            flash.style.opacity = "1";
            setTimeout(() => { flash.style.opacity = "0"; }, 160);

            updateHUD();
            if (player.health === 0) {
                player.alive = false;
                const aliveCount = bots.filter(b => b.alive).length + 1;
                showMatchEndBanner(
                    false,
                    "YOU WERE ELIMINATED",
                    `Taken down by ${attackerName}.`,
                    `#${aliveCount} / 20`
                );
            }
        }

        function checkMatchVictory() {
            const aliveBots = bots.filter(b => b.alive).length;
            if (aliveBots === 0 && player.alive) {
                const subText = gameMode === "zombies"
                    ? "You eliminated all 19 zombies in Dead Shot."
                    : "You outlasted all 19 military opponents in Dead Shot.";
                showMatchEndBanner(
                    true,
                    "VICTORY ROYALE!",
                    subText,
                    "#1 / 20"
                );
            }
        }

        // =========================================================================
        // 9. PLAYER MOVEMENT, AI SURVIVOR SIMULATION & ENEMY AI
        // =========================================================================
        function updatePlayer(dt) {
            if (!player.alive) return;

            if (player.fireCooldown > 0) player.fireCooldown = Math.max(0, player.fireCooldown - dt);

            weaponHolder.position.z = THREE.MathUtils.lerp(weaponHolder.position.z, -0.45, dt * 14);
            weaponHolder.position.y = THREE.MathUtils.lerp(weaponHolder.position.y, -0.24, dt * 14);
            weaponHolder.rotation.x = THREE.MathUtils.lerp(weaponHolder.rotation.x, 0, dt * 14);

            if (player.reloading) {
                player.reloadTimer -= dt;
                if (player.reloadTimer <= 0) {
                    player.reloading = false;
                    const w = player.weapons[player.activeSlot];
                    const needed = w.magSize - w.mag;
                    const transfer = Math.min(needed, w.reserve);
                    w.mag += transfer;
                    w.reserve -= transfer;
                    updateHUD();
                }
            }

            if (player.isHealing) {
                player.healTimer -= dt;
                const pct = Math.min(100, Math.max(0, ((MEDKIT_DURATION - player.healTimer) / MEDKIT_DURATION) * 100));
                document.getElementById("medkit-progress-bar").style.width = `${pct}%`;
                document.getElementById("medkit-timer-text").textContent = `${Math.max(0, player.healTimer).toFixed(1)}s`;
                if (player.healTimer <= 0) {
                    completeMedkitHeal();
                }
            }

            if (simModeActive) {
                runPlayerAutoSimulator(dt);
                return;
            }

            if (keys["ArrowLeft"]) player.yaw += 2.1 * dt;
            if (keys["ArrowRight"]) player.yaw -= 2.1 * dt;
            if (keys["ArrowUp"]) player.pitch = THREE.MathUtils.clamp(player.pitch + 1.5 * dt, -1.42, 1.42);
            if (keys["ArrowDown"]) player.pitch = THREE.MathUtils.clamp(player.pitch - 1.5 * dt, -1.42, 1.42);

            const forward = new THREE.Vector3(-Math.sin(player.yaw), 0, -Math.cos(player.yaw));
            const right = new THREE.Vector3(Math.cos(player.yaw), 0, -Math.sin(player.yaw));
            const moveDir = new THREE.Vector3();

            if (keys["KeyW"]) moveDir.add(forward);
            if (keys["KeyS"]) moveDir.sub(forward);
            if (keys["KeyD"]) moveDir.add(right);
            if (keys["KeyA"]) moveDir.sub(right);

            if (moveDir.lengthSq() > 0) {
                moveDir.normalize();
                const isSprinting = (keys["ShiftLeft"] || keys["ShiftRight"]) && !player.isHealing;
                const speed = player.isHealing ? 4.2 : (isSprinting ? 14.5 : 9.8);
                player.pos.addScaledVector(moveDir, speed * dt);
            }

            if (keys["Space"] && player.grounded) {
                player.vel.y = 8.4;
                player.grounded = false;
            }
            player.vel.y -= 21.0 * dt;
            player.pos.y += player.vel.y * dt;

            const floorHeight = getSupportHeightAt(player.pos.x, player.pos.z, player.pos.y);
            if (player.pos.y <= floorHeight) {
                player.pos.y = floorHeight;
                player.vel.y = 0;
                player.grounded = true;
            } else {
                player.grounded = false;
            }

            resolveArenaCollisions(player.pos);

            camera.position.set(player.pos.x, player.pos.y + 1.72, player.pos.z);
            camera.rotation.y = player.yaw;
            camera.rotation.x = player.pitch;

            if (player.buildMode) {
                updateBuildPreviewPosition();
            }

            if (isMouseDown && !player.buildMode && player.weapons[player.activeSlot].auto) {
                attemptPlayerAttack();
            }

            updateNearbyChestPrompt();
        }

        function runPlayerAutoSimulator(dt) {
            let nearestBot = null;
            let bestDist = Infinity;
            bots.forEach(b => {
                if (!b.alive) return;
                const d = player.pos.distanceTo(b.pos);
                if (d < bestDist) {
                    bestDist = d;
                    nearestBot = b;
                }
            });

            if (player.health < 55 && player.medkits > 0 && !player.isHealing && bestDist > 16) {
                startMedkitHeal();
            }

            if (nearestBot) {
                const dx = nearestBot.pos.x - player.pos.x;
                const dz = nearestBot.pos.z - player.pos.z;
                const targetYaw = Math.atan2(-dx, -dz);
                player.yaw = targetYaw;
                player.pitch = 0;

                const dir = new THREE.Vector3(dx, 0, dz).normalize();
                if (bestDist > 22) {
                    player.pos.addScaledVector(dir, 10.5 * dt);
                } else {
                    const strafe = new THREE.Vector3(-dir.z, 0, dir.x);
                    player.pos.addScaledVector(strafe, 6.5 * dt);
                }
                resolveArenaCollisions(player.pos);
                camera.position.set(player.pos.x, player.pos.y + 1.72, player.pos.z);
                camera.rotation.y = player.yaw;
                camera.rotation.x = 0;

                if (bestDist < 90 && !player.isHealing) {
                    attemptPlayerAttack();
                }
            }
        }

        function tryInteractChest() {
            for (let i = 0; i < chests.length; i++) {
                const c = chests[i];
                if (!c.opened && player.pos.distanceTo(c.pos) <= 3.6) {
                    openChest(c);
                    break;
                }
            }
        }

        function updateNearbyChestPrompt() {
            const promptEl = document.getElementById("loot-prompt");
            let found = false;
            for (let i = 0; i < chests.length; i++) {
                const c = chests[i];
                if (!c.opened && player.pos.distanceTo(c.pos) <= 3.6) {
                    found = true;
                    break;
                }
            }
            if (found) promptEl.classList.remove("hidden");
            else promptEl.classList.add("hidden");
        }

        function updateBots(dt) {
            const speedMult = lobbyDifficulty === "nightmare" ? 1.32 : (lobbyDifficulty === "hard" ? 1.15 : 1.0);
            const isZombieMode = gameMode === "zombies";

            for (let i = 0; i < bots.length; i++) {
                const bot = bots[i];
                if (!bot.alive) continue;

                bot.fireCooldown -= dt;
                bot.strafeTimer -= dt;
                if (bot.strafeTimer <= 0) {
                    bot.strafeDir *= -1;
                    bot.strafeTimer = 1.4 + Math.random() * 2.2;
                }

                if (isZombieMode) {
                    // Zombie Horde Mode: Shambling pursuit + melee claw attack
                    if (!player.alive) continue;
                    const dx = player.pos.x - bot.pos.x;
                    const dz = player.pos.z - bot.pos.z;
                    const dist = Math.hypot(dx, dz);

                    bot.yaw = Math.atan2(dx, dz);
                    bot.mesh.root.rotation.y = bot.yaw;

                    const moveSpeed = (dist < 24 ? 7.6 : 6.1) * speedMult;
                    if (dist > 1.75) {
                        const nx = dx / dist;
                        const nz = dz / dist;
                        bot.pos.x += nx * moveSpeed * dt;
                        bot.pos.z += nz * moveSpeed * dt;
                        resolveArenaCollisions(bot.pos);
                        bot.mesh.root.position.copy(bot.pos);
                    }

                    bot.walkPhase += dt * 9.5;
                    bot.mesh.leftLeg.rotation.x = Math.sin(bot.walkPhase) * 0.55;
                    bot.mesh.rightLeg.rotation.x = -Math.sin(bot.walkPhase) * 0.55;
                    if (bot.mesh.leftArm && bot.mesh.rightArm) {
                        bot.mesh.leftArm.rotation.x = -0.15 + Math.sin(bot.walkPhase * 0.5) * 0.14;
                        bot.mesh.rightArm.rotation.x = -0.15 - Math.sin(bot.walkPhase * 0.5) * 0.14;
                    }

                    if (dist <= 2.35 && bot.fireCooldown <= 0) {
                        bot.fireCooldown = 1.05 / speedMult;
                        if (bot.mesh.leftArm && bot.mesh.rightArm) {
                            bot.mesh.leftArm.rotation.x = -0.85;
                            bot.mesh.rightArm.rotation.x = -0.85;
                        }
                        applyDamageToPlayer(ZOMBIE_CLAW_DAMAGE, bot.name);
                    }
                } else {
                    // Classic Battle Royale Mode: Military bots engage both player and rival bots
                    let targetPos = null;
                    let targetIsPlayer = false;
                    let targetBot = null;
                    let bestDist = Infinity;

                    if (player.alive) {
                        const dp = bot.pos.distanceTo(player.pos);
                        if (dp < bestDist) {
                            bestDist = dp;
                            targetPos = player.pos;
                            targetIsPlayer = true;
                        }
                    }

                    for (let j = 0; j < bots.length; j++) {
                        if (i === j) continue;
                        const other = bots[j];
                        if (!other.alive) continue;
                        const db = bot.pos.distanceTo(other.pos);
                        if (db < bestDist) {
                            bestDist = db;
                            targetPos = other.pos;
                            targetIsPlayer = false;
                            targetBot = other;
                        }
                    }

                    if (!targetPos) continue;

                    const dx = targetPos.x - bot.pos.x;
                    const dz = targetPos.z - bot.pos.z;
                    bot.yaw = Math.atan2(dx, dz);
                    bot.mesh.root.rotation.y = bot.yaw;

                    const forward = new THREE.Vector3(dx, 0, dz).normalize();
                    const strafe = new THREE.Vector3(-forward.z, 0, forward.x).multiplyScalar(bot.strafeDir);

                    if (bestDist > 26) {
                        bot.pos.addScaledVector(forward, 6.8 * speedMult * dt);
                    } else if (bestDist < 12) {
                        bot.pos.addScaledVector(forward, -4.5 * dt);
                    }
                    bot.pos.addScaledVector(strafe, 4.2 * dt);
                    resolveArenaCollisions(bot.pos);
                    bot.mesh.root.position.copy(bot.pos);

                    bot.walkPhase += dt * 9;
                    bot.mesh.leftLeg.rotation.x = Math.sin(bot.walkPhase) * 0.45;
                    bot.mesh.rightLeg.rotation.x = -Math.sin(bot.walkPhase) * 0.45;

                    if (bestDist < 85 && bot.fireCooldown <= 0) {
                        bot.fireCooldown = (0.78 + Math.random() * 1.1) / speedMult;
                        const muzzle = bot.pos.clone().add(new THREE.Vector3(0, 1.35, 0));

                        const hitChance = Math.max(0.12, 0.32 - bestDist * 0.0024);
                        if (Math.random() < hitChance) {
                            const isHead = Math.random() < 0.22;
                            const dmg = isHead ? BOT_HEADSHOT_DAMAGE : BOT_BODYSHOT_DAMAGE;
                            const hitPt = targetPos.clone().add(new THREE.Vector3(0, isHead ? 1.7 : 1.1, 0));
                            spawnTracer(muzzle, hitPt, 0xff6b6b);

                            if (targetIsPlayer) {
                                applyDamageToPlayer(dmg, bot.name);
                            } else if (targetBot) {
                                applyDamageToBot(targetBot, dmg, bot.name, false, isHead);
                            }
                        } else {
                            const missPt = targetPos.clone().add(new THREE.Vector3(
                                (Math.random() - 0.5) * 4.5,
                                0.5 + Math.random() * 2.2,
                                (Math.random() - 0.5) * 4.5
                            ));
                            spawnTracer(muzzle, missPt, 0xfca5a5);
                        }
                    }
                }
            }
        }

        function updateVisualEffects(dt) {
            for (let i = tracers.length - 1; i >= 0; i--) {
                tracers[i].life -= dt;
                if (tracers[i].life <= 0) {
                    scene.remove(tracers[i].line);
                    tracers.splice(i, 1);
                }
            }
            for (let i = particles.length - 1; i >= 0; i--) {
                const p = particles[i];
                p.life -= dt;
                p.vel.y -= 14 * dt;
                p.mesh.position.addScaledVector(p.vel, dt);
                if (p.life <= 0) {
                    scene.remove(p.mesh);
                    particles.splice(i, 1);
                }
            }
        }

        // =========================================================================
        // 10. HUD, KILL FEED, ROSTER & SIMULATION MODE CONTROLS
        // =========================================================================
        function updateHUD() {
            const hpPct = (player.health / PLAYER_MAX_HEALTH) * 100;
            document.getElementById("hud-health-val").textContent = player.health;
            document.getElementById("hud-health-bar").style.width = `${hpPct}%`;
            document.getElementById("hud-kills-val").textContent = player.kills;

            const aliveEnemies = bots.filter(b => b.alive).length;
            const enemyNoun = gameMode === "zombies" ? "Zombies" : "Enemies";
            document.getElementById("hud-mode-badge").textContent = `DEAD SHOT (${aliveEnemies} ${enemyNoun})`;
            document.getElementById("hud-submode-badge").textContent = gameMode === "zombies" ? "ZOMBIE SURVIVAL" : "CLASSIC BR";

            const buildBar = document.getElementById("build-bar");
            const weaponBar = document.getElementById("weapon-bar");

            if (player.buildMode && buildEnabled) {
                document.getElementById("hud-weapon-name").textContent = `BLUEPRINT • ${player.buildPiece.toUpperCase()}`;
                document.getElementById("hud-ammo-mag").textContent = "10";
                document.getElementById("hud-ammo-res").textContent = "WOOD / PIECE";
                if (buildBar) {
                    buildBar.classList.remove("hidden");
                    buildBar.classList.add("flex");
                }
                if (weaponBar) weaponBar.classList.add("hidden");

                ["wall", "floor", "ramp"].forEach(piece => {
                    const el = document.getElementById("bpiece-" + piece);
                    if (!el) return;
                    if (player.buildPiece === piece) {
                        el.classList.add("border-2", "border-sky-400");
                        el.classList.remove("border", "border-stone-700");
                    } else {
                        el.classList.remove("border-2", "border-sky-400");
                        el.classList.add("border", "border-stone-700");
                    }
                });
            } else {
                if (buildBar) {
                    buildBar.classList.add("hidden");
                    buildBar.classList.remove("flex");
                }
                if (weaponBar) weaponBar.classList.remove("hidden");

                const w = player.weapons[player.activeSlot];
                document.getElementById("hud-weapon-name").textContent = player.reloading ? "RELOADING..." : w.name;
                if (w.id === "knife") {
                    document.getElementById("hud-ammo-mag").textContent = "∞";
                    document.getElementById("hud-ammo-res").textContent = "MELEE";
                } else if (w.id === "medkit") {
                    document.getElementById("hud-ammo-mag").textContent = player.medkits;
                    document.getElementById("hud-ammo-res").textContent = "KITS";
                } else {
                    document.getElementById("hud-ammo-mag").textContent = w.mag;
                    document.getElementById("hud-ammo-res").textContent = `/ ${w.reserve}`;
                }
            }

            document.getElementById("slot-4-label").textContent = `MED (${player.medkits})`;
            for (let i = 0; i < 5; i++) {
                const slotEl = document.getElementById(`slot-${i}`);
                const owned = player.weapons[i].owned || (i === 4 && player.medkits > 0);
                slotEl.style.opacity = owned ? "1" : "0.38";
                if (!player.buildMode && i === player.activeSlot) {
                    slotEl.classList.add("border-2", "border-lime-400");
                    slotEl.classList.remove("border-stone-700");
                } else {
                    slotEl.classList.remove("border-2", "border-lime-400");
                    slotEl.classList.add("border-stone-700");
                }
            }
        }

        function addKillFeedEntry(text, colorClass = "text-stone-200") {
            const feed = document.getElementById("kill-feed");
            const item = document.createElement("div");
            item.className = `glass-panel px-3 py-1.5 rounded-xl text-[11px] font-bold shadow ${colorClass}`;
            item.textContent = text;
            feed.prepend(item);
            while (feed.children.length > 5) {
                feed.removeChild(feed.lastChild);
            }
            setTimeout(() => {
                if (item.parentNode) item.remove();
            }, 5500);
        }

        function updateRosterUI() {
            const list = document.getElementById("roster-list");
            const aliveBots = bots.filter(b => b.alive).length;
            document.getElementById("menu-alive-count").textContent = `${aliveBots} REMAINING`;

            let html = `
                <div class="flex items-center justify-between px-2.5 py-1.5 rounded-lg bg-lime-500/20 border border-lime-400/40 font-bold text-lime-300">
                    <span>${player.name} (You)</span>
                    <span class="mono">${player.alive ? player.health + " HP" : "ELIMINATED"}</span>
                </div>
            `;
            bots.forEach(b => {
                html += `
                    <div class="flex items-center justify-between px-2.5 py-1 rounded-lg ${b.alive ? "bg-stone-800/70 text-stone-200" : "bg-stone-900/40 text-stone-500 line-through"}">
                        <span>${b.name}</span>
                        <span class="mono text-[11px]">${b.alive ? b.health + " HP" : "DEAD"}</span>
                    </div>
                `;
            });
            list.innerHTML = html;
        }

        function toggleSimMode() {
            simModeActive = !simModeActive;
            const btn = document.getElementById("btn-sim-toggle");
            if (simModeActive) {
                btn.textContent = "[M] Mode: Auto-Survivor";
                btn.classList.replace("text-emerald-300", "text-amber-300");
            } else {
                btn.textContent = "[M] Mode: Manual Play";
                btn.classList.replace("text-amber-300", "text-emerald-300");
            }
        }

        function setSimSpeed(speed) {
            simSpeed = speed;
            [1, 2, 4].forEach(s => {
                const b = document.getElementById(`spd-${s}`);
                if (s === speed) {
                    b.className = "px-2 py-0.5 rounded bg-lime-600 text-stone-950 font-bold text-[10px]";
                } else {
                    b.className = "px-2 py-0.5 rounded bg-stone-800 text-stone-300 font-bold text-[10px]";
                }
            });
        }

        function openPauseMenu() {
            isPaused = true;
            unlockLookControl();
            updateRosterUI();
            document.getElementById("pause-menu-overlay").classList.remove("hidden");
            document.getElementById("pause-menu-overlay").classList.add("flex");
        }

        function resumeGameplay() {
            isPaused = false;
            document.getElementById("pause-menu-overlay").classList.add("hidden");
            document.getElementById("pause-menu-overlay").classList.remove("flex");
            engageLookControl();
        }

        function showMatchEndBanner(isWin, title, sub, placement) {
            unlockLookControl();
            document.getElementById("end-banner-tag").textContent = isWin ? "VICTORY ROYALE" : "MATCH OVER";
            document.getElementById("end-banner-title").textContent = title;
            document.getElementById("end-banner-sub").textContent = sub;
            document.getElementById("end-stat-kills").textContent = player.kills;
            document.getElementById("end-stat-place").textContent = placement;
            document.getElementById("match-end-overlay").classList.remove("hidden");
            document.getElementById("match-end-overlay").classList.add("flex");
        }

        // =========================================================================
        // 11. LOBBY CONFIGURATION & MATCH RESET
        // =========================================================================
        function resetSoloMatch() {
            document.getElementById("match-end-overlay").classList.add("hidden");
            document.getElementById("match-end-overlay").classList.remove("flex");
            document.getElementById("pause-menu-overlay").classList.add("hidden");
            document.getElementById("pause-menu-overlay").classList.remove("flex");
            document.getElementById("kill-feed").innerHTML = "";
            isPaused = false;

            player.pos.set(0, 0, 0);
            player.vel.set(0, 0, 0);
            player.yaw = 0;
            player.pitch = 0;
            player.health = PLAYER_MAX_HEALTH;
            player.alive = true;
            player.kills = 0;
            player.medkits = 0;
            player.reloading = false;
            cancelMedkitHeal();

            player.weapons[0].owned = false; player.weapons[0].mag = 30; player.weapons[0].reserve = 150;
            player.weapons[1].owned = false; player.weapons[1].mag = 6; player.weapons[1].reserve = 36;
            player.weapons[2].owned = true;  player.weapons[2].mag = 15; player.weapons[2].reserve = 90;
            player.weapons[3].owned = true;
            player.weapons[4].owned = false; player.weapons[4].mag = 0;

            player.buildMode = false;
            player.buildPiece = "wall";
            player.wood = buildEnabled ? 150 : 0;
            setLobbyBuildMode(buildEnabled);
            if (blueprintViewModel) blueprintViewModel.visible = false;
            if (buildPreviewGroup) buildPreviewGroup.visible = false;
            builtStructures.forEach(s => scene.remove(s.mesh));
            builtStructures = [];
            updateWoodHUD();

            buildSelectedMap(selectedMap);
            switchWeaponSlot(2);
            spawnSupplyChests();
            spawnAllBots();
            updateHUD();
            updateRosterUI();
        }

        function setLobbyBuildMode(enabled) {
            buildEnabled = enabled;
            const onBtn = document.getElementById("build-opt-on");
            const offBtn = document.getElementById("build-opt-off");
            if (onBtn && offBtn) {
                onBtn.className = enabled
                    ? "py-2 rounded-xl bg-lime-500 text-stone-950 font-black text-xs uppercase transition"
                    : "py-2 rounded-xl bg-stone-800 text-stone-300 hover:bg-stone-700 font-extrabold text-xs uppercase transition";
                offBtn.className = !enabled
                    ? "py-2 rounded-xl bg-lime-500 text-stone-950 font-black text-xs uppercase transition"
                    : "py-2 rounded-xl bg-stone-800 text-stone-300 hover:bg-stone-700 font-extrabold text-xs uppercase transition";
            }
            const woodPanel = document.getElementById("hud-wood-panel");
            const hintEl = document.getElementById("hud-controls-hint");
            if (woodPanel) woodPanel.style.display = enabled ? "" : "none";
            if (hintEl) hintEl.textContent = enabled ? "[Q] Build • [H] Medkit" : "[R] Reload • [H] Medkit";
        }

        function setLobbyGameMode(mode) {
            gameMode = mode;
            const zBtn = document.getElementById("mode-zombies");
            const cBtn = document.getElementById("mode-classic");
            if (zBtn && cBtn) {
                zBtn.className = mode === "zombies"
                    ? "py-2 rounded-xl bg-lime-500 text-stone-950 font-black text-xs uppercase transition"
                    : "py-2 rounded-xl bg-stone-800 text-stone-300 hover:bg-stone-700 font-extrabold text-xs uppercase transition";
                cBtn.className = mode === "classic"
                    ? "py-2 rounded-xl bg-lime-500 text-stone-950 font-black text-xs uppercase transition"
                    : "py-2 rounded-xl bg-stone-800 text-stone-300 hover:bg-stone-700 font-extrabold text-xs uppercase transition";
            }

            const topBadge = document.getElementById("lobby-top-badge");
            const subEl = document.getElementById("lobby-subtitle");
            const countEl = document.getElementById("lobby-enemy-count");
            const labelEl = document.getElementById("lobby-enemy-label");

            if (mode === "zombies") {
                if (topBadge) topBadge.textContent = "OUTBREAK ZONE • 19 INFECTED";
                if (subEl) subEl.textContent = "Zombie Survival • 100 HP • 25 Headshot / 15 Body";
                if (countEl) countEl.textContent = "19 ZOMBIES DETECTED";
                if (labelEl) labelEl.textContent = "Infected Horde (x19)";
            } else {
                if (topBadge) topBadge.textContent = "20-PLAYER SOLO • MILITARY BOTS";
                if (subEl) subEl.textContent = "Classic Battle Royale • 100 HP • Bots Fight Each Other";
                if (countEl) countEl.textContent = "19 MILITARY BOTS";
                if (labelEl) labelEl.textContent = "Rival Military Squads (x19)";
            }
        }

        function setLobbyMap(mapName) {
            selectedMap = mapName;
            ["woodland", "city", "desert"].forEach(m => {
                const btn = document.getElementById("map-" + m);
                if (!btn) return;
                btn.className = (m === mapName)
                    ? "py-2 rounded-xl bg-lime-500 text-stone-950 font-black text-xs uppercase transition"
                    : "py-2 rounded-xl bg-stone-800 text-stone-300 hover:bg-stone-700 font-extrabold text-xs uppercase transition";
            });
            const mapTitle = mapName.charAt(0).toUpperCase() + mapName.slice(1);
            const mapLabel = document.getElementById("lobby-map-label");
            if (mapLabel) mapLabel.textContent = mapTitle + " Supply Chests";
        }

        function setLobbyDifficulty(diff) {
            lobbyDifficulty = diff;
            ["normal", "hard", "nightmare"].forEach(d => {
                const btn = document.getElementById("diff-" + d);
                if (!btn) return;
                btn.className = (d === diff)
                    ? "py-2 rounded-xl bg-lime-500 text-stone-950 font-black text-xs uppercase transition"
                    : "py-2 rounded-xl bg-stone-800 text-stone-300 hover:bg-stone-700 font-extrabold text-xs uppercase transition";
            });
        }

        function deployFromLobby() {
            initAudio();
            const nameInput = document.getElementById("lobby-player-name");
            const cleanName = (nameInput && nameInput.value.trim()) ? nameInput.value.trim() : "Survivor-01";
            player.name = cleanName;
            const previewEl = document.getElementById("lobby-preview-name");
            if (previewEl) previewEl.textContent = `${cleanName} (You)`;

            inMainLobby = false;
            const lobbyEl = document.getElementById("main-lobby-overlay");
            if (lobbyEl) {
                lobbyEl.classList.add("hidden");
                lobbyEl.classList.remove("flex");
            }
            resetSoloMatch();
            engageLookControl();
        }

        function returnToMainLobby() {
            inMainLobby = true;
            isPaused = false;
            unlockLookControl();
            document.getElementById("pause-menu-overlay").classList.add("hidden");
            document.getElementById("pause-menu-overlay").classList.remove("flex");
            document.getElementById("match-end-overlay").classList.add("hidden");
            document.getElementById("match-end-overlay").classList.remove("flex");
            const lobbyEl = document.getElementById("main-lobby-overlay");
            if (lobbyEl) {
                lobbyEl.classList.remove("hidden");
                lobbyEl.classList.add("flex");
            }
        }

        // =========================================================================
        // 12. MINIMAP & MAIN RENDER LOOP
        // =========================================================================
        function drawMinimap() {
            const canvas = document.getElementById("minimap-canvas");
            if (!canvas) return;
            const ctx = canvas.getContext("2d");
            const w = canvas.width;
            const h = canvas.height;
            ctx.clearRect(0, 0, w, h);

            const scale = (w / 2) / ARENA_HALF_SIZE;
            const cx = w / 2;
            const cy = h / 2;

            ctx.strokeStyle = "rgba(163, 230, 53, 0.14)";
            ctx.lineWidth = 1;
            ctx.beginPath();
            ctx.arc(cx, cy, w * 0.25, 0, Math.PI * 2);
            ctx.arc(cx, cy, w * 0.45, 0, Math.PI * 2);
            ctx.stroke();

            chests.forEach(c => {
                if (c.opened) return;
                const mx = cx + c.pos.x * scale;
                const my = cy + c.pos.z * scale;
                ctx.fillStyle = "#fbbf24";
                ctx.fillRect(mx - 2, my - 2, 4, 4);
            });

            bots.forEach(b => {
                if (!b.alive) return;
                const mx = cx + b.pos.x * scale;
                const my = cy + b.pos.z * scale;
                ctx.fillStyle = "#ef4444";
                ctx.beginPath();
                ctx.arc(mx, my, 2.7, 0, Math.PI * 2);
                ctx.fill();
            });

            const px = cx + player.pos.x * scale;
            const py = cy + player.pos.z * scale;
            ctx.save();
            ctx.translate(px, py);
            ctx.rotate(-player.yaw);
            ctx.fillStyle = "#a3e635";
            ctx.beginPath();
            ctx.moveTo(0, -6);
            ctx.lineTo(4.5, 5);
            ctx.lineTo(0, 2.5);
            ctx.lineTo(-4.5, 5);
            ctx.closePath();
            ctx.fill();
            ctx.restore();
        }

        function animateLoop() {
            requestAnimationFrame(animateLoop);
            const rawDt = Math.min(clock.getDelta(), 0.05);
            const steps = simModeActive ? simSpeed : 1;

            if (!isPaused && !inMainLobby) {
                for (let s = 0; s < steps; s++) {
                    updatePlayer(rawDt);
                    updateBots(rawDt);
                    updateVisualEffects(rawDt);
                }
            }

            if (camera && renderer && scene) {
                renderer.render(scene, camera);
            }
            drawMinimap();
        }

        window.onload = function () {
            initThreeScene();
            buildSelectedMap(selectedMap);
            spawnSupplyChests();
            spawnAllBots();
            setLobbyGameMode(gameMode);
            setLobbyBuildMode(buildEnabled);
            setLobbyMap(selectedMap);
            setLobbyDifficulty(lobbyDifficulty);
            updateHUD();
            updateRosterUI();
            animateLoop();
        };
    </script>
</body>
</html>
