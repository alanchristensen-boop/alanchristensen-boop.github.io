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
            <!-- Fortnite-Style Top Navigation Bar -->
            <div class="flex flex-wrap items-center justify-between gap-3 border-b border-stone-700/80 pb-4">
                <div class="flex items-center gap-3">
                    <h1 class="text-2xl sm:text-3xl font-black tracking-tighter text-white">DEAD <span class="text-lime-400">SHOT</span></h1>
                    <div class="flex items-center gap-1.5 bg-stone-950/90 p-1 rounded-2xl border border-stone-700/80">
                        <button type="button" onclick="switchLobbyTab('gameplay')" id="tab-btn-gameplay"
                                class="px-4 py-1.5 rounded-xl bg-lime-400 text-stone-950 font-black text-xs uppercase tracking-wider shadow transition">
                            Gameplay
                        </button>
                        <button type="button" onclick="switchLobbyTab('locker')" id="tab-btn-locker"
                                class="px-4 py-1.5 rounded-xl text-stone-300 hover:text-white font-extrabold text-xs uppercase tracking-wider transition">
                            Locker
                        </button>
                    </div>
                </div>
                <div class="bg-stone-900/90 px-3.5 py-1.5 rounded-2xl border border-stone-700/80 text-right">
                    <div class="text-[9px] uppercase tracking-wider text-stone-400 font-bold">Lobby Status</div>
                    <div class="text-xs font-black text-lime-400 mono">READY TO DEPLOY</div>
                </div>
            </div>

            <!-- TAB 1: GAMEPLAY (MODE, MAP, MULTIPLAYER & DEPLOY) -->
            <div id="lobby-tab-gameplay">
                <div class="flex flex-col sm:flex-row sm:items-center justify-between gap-2 mt-4">
                    <div>
                        <div class="inline-flex items-center gap-2 px-2.5 py-0.5 rounded-full bg-red-500/20 border border-red-400/40 text-red-300 text-[10px] font-black uppercase tracking-widest">
                            <span class="w-2 h-2 rounded-full bg-red-500 animate-ping"></span>
                            <span id="lobby-top-badge">OUTBREAK ZONE • 19 INFECTED</span>
                        </div>
                        <p id="lobby-subtitle" class="text-xs text-stone-400 mt-1">Zombie Survival • 100 HP • 25 Headshot / 15 Body</p>
                    </div>
                    <button type="button" onclick="switchLobbyTab('locker')" class="self-start sm:self-center px-3 py-1.5 rounded-xl bg-stone-900 hover:bg-stone-800 border border-stone-700 text-[11px] font-bold text-amber-300 transition">
                        Outfit: <span id="gameplay-skin-summary">RANGER OLIVE</span> →
                    </button>
                </div>

                <!-- Live Online Multiplayer Room Bar -->
                <div class="mt-4 bg-stone-900/90 p-3.5 rounded-2xl border border-sky-400/35">
                    <div class="flex flex-col sm:flex-row sm:items-center justify-between gap-2">
                        <div>
                            <div class="flex items-center gap-2">
                                <span id="mp-status-dot" class="w-2 h-2 rounded-full bg-amber-400"></span>
                                <span class="text-[11px] font-extrabold uppercase tracking-wider text-sky-300">Live Online Multiplayer</span>
                                <span id="mp-conn-mode" class="mono text-[9px] font-black px-2 py-0.5 rounded bg-amber-500/20 text-amber-300 border border-amber-400/30">CHECKING...</span>
                                <span id="mp-room-badge" class="mono text-[10px] font-black px-2 py-0.5 rounded bg-stone-800 text-stone-300">SOLO / OFFLINE</span>
                            </div>
                            <div class="text-[10px] text-stone-400 mt-1 mono break-all">
                                Your ID: <span id="mp-user-id" class="text-stone-200 select-all">Connecting...</span>
                            </div>
                        </div>
                        <div class="flex flex-wrap items-center gap-1.5">
                            <button type="button" onclick="createMultiplayerRoom()"
                                    class="px-3 py-1.5 rounded-xl bg-sky-500 hover:bg-sky-400 text-stone-950 font-black text-[11px] uppercase transition">
                                Host Room
                            </button>
                            <input id="mp-room-input" type="text" maxlength="6" placeholder="CODE"
                                   class="w-20 px-2.5 py-1.5 rounded-xl bg-stone-950 border border-stone-700 focus:border-sky-400 focus:outline-none text-xs font-black uppercase text-center text-white mono" />
                            <button type="button" onclick="joinMultiplayerRoom()"
                                    class="px-3 py-1.5 rounded-xl bg-emerald-500 hover:bg-emerald-400 text-stone-950 font-black text-[11px] uppercase transition">
                                Join
                            </button>
                            <button type="button" id="mp-leave-btn" onclick="leaveMultiplayerRoom()"
                                    class="hidden px-2.5 py-1.5 rounded-xl bg-red-500/20 hover:bg-red-500/30 border border-red-400/40 text-red-300 font-bold text-[11px] uppercase transition">
                                Leave
                            </button>
                        </div>
                    </div>
                    <div id="mp-feedback-msg" class="hidden mt-2 text-[11px] font-bold text-amber-300"></div>
                    <div id="mp-party-header" class="mt-2.5 pt-2 border-t border-stone-800 hidden items-center justify-between">
                        <span class="text-[10px] font-extrabold uppercase tracking-wider text-sky-300">Active Party Squad</span>
                        <span id="mp-party-count" class="mono text-[10px] font-bold text-lime-400">1 PLAYER IN PARTY</span>
                    </div>
                    <div id="mp-players-list" class="mt-1.5 hidden flex-wrap items-center gap-1.5 text-[11px]"></div>
                </div>

                <div class="grid grid-cols-1 md:grid-cols-2 gap-5 mt-4">
                    <div class="bg-stone-900/75 p-4 rounded-2xl border border-stone-700/70 flex flex-col justify-between">
                        <div>
                            <label class="block text-[11px] font-extrabold uppercase tracking-wider text-purple-300 mb-1">Game Mode</label>
                            <div class="grid grid-cols-3 gap-1.5">
                                <button type="button" onclick="setLobbyGameMode('zombies')" id="mode-zombies"
                                        class="py-2 px-1 rounded-xl bg-lime-500 text-stone-950 font-black text-[11px] uppercase transition">
                                    Zombies
                                </button>
                                <button type="button" onclick="setLobbyGameMode('classic')" id="mode-classic"
                                        class="py-2 px-1 rounded-xl bg-stone-800 text-stone-300 hover:bg-stone-700 font-extrabold text-[11px] uppercase transition">
                                    Classic BR
                                </button>
                                <button type="button" onclick="setLobbyGameMode('pvp')" id="mode-pvp"
                                        class="py-2 px-1 rounded-xl bg-stone-800 text-stone-300 hover:bg-stone-700 font-extrabold text-[11px] uppercase transition">
                                    PvP Arena
                                </button>
                            </div>

                            <label class="block text-[11px] font-extrabold uppercase tracking-wider text-sky-300 mt-3 mb-1">Building Mode</label>
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

                            <label class="block text-[11px] font-extrabold uppercase tracking-wider text-amber-300 mt-3 mb-1">Select Map</label>
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

                            <label class="block text-[11px] font-extrabold uppercase tracking-wider text-stone-300 mt-3 mb-1">Difficulty</label>
                            <div class="grid grid-cols-3 gap-2">
                                <button type="button" onclick="setLobbyDifficulty('normal')" id="diff-normal"
                                        class="py-2 rounded-xl bg-lime-500 text-stone-950 font-black text-xs uppercase transition">
                                    Normal
                                </button>
                                <button type="button" onclick="setLobbyDifficulty('hard')" id="diff-hard"
                                        class="py-2 rounded-xl bg-stone-800 text-stone-300 hover:bg-stone-700 font-extrabold text-xs uppercase transition">
                                    Hard
                                </button>
                                <button type="button" onclick="setLobbyDifficulty('nightmare')" id="diff-nightmare"
                                        class="py-2 rounded-xl bg-stone-800 text-stone-300 hover:bg-stone-700 font-extrabold text-xs uppercase transition">
                                    Nightmare
                                </button>
                            </div>
                        </div>
                    </div>

                    <div class="bg-stone-900/75 p-4 rounded-2xl border border-stone-700/70 flex flex-col justify-between">
                        <div>
                            <div class="text-[11px] font-extrabold uppercase tracking-wider text-lime-300 mb-2">Match Briefing</div>
                            <div class="text-xs space-y-2 text-stone-300">
                                <div class="flex justify-between">
                                    <span id="lobby-enemy-label" class="text-stone-400">Infected Horde (x19)</span>
                                    <span id="lobby-enemy-count" class="font-bold text-red-400 mono">19 ZOMBIES DETECTED</span>
                                </div>
                                <div class="flex justify-between">
                                    <span class="text-stone-400">Sector Loot</span>
                                    <span id="lobby-map-label" class="font-bold text-amber-300">Woodland Supply Chests</span>
                                </div>
                                <div class="flex justify-between">
                                    <span class="text-stone-400">Active Survivor</span>
                                    <span id="lobby-preview-name" class="font-bold text-lime-300">Survivor-01 (You)</span>
                                </div>
                                <div class="flex justify-between">
                                    <span class="text-stone-400">Loadout Finish</span>
                                    <span id="lobby-briefing-skin" class="font-bold text-sky-300">Ranger Olive</span>
                                </div>
                            </div>
                        </div>

                        <button type="button" onclick="deployFromLobby()"
                                class="mt-5 w-full py-3.5 rounded-2xl bg-gradient-to-r from-lime-400 to-emerald-500 hover:from-lime-300 hover:to-emerald-400 text-stone-950 font-black text-sm uppercase tracking-wider shadow-lg transition transform active:scale-98">
                            Play / Deploy Into Match
                        </button>
                    </div>
                </div>
            </div>

            <!-- TAB 2: LOCKER (CHARACTER SKINS, WEAPON WRAPS & CALLSIGN) -->
            <div id="lobby-tab-locker" class="hidden mt-5">
                <div class="grid grid-cols-1 md:grid-cols-2 gap-5">
                    <div class="bg-stone-900/75 p-5 rounded-2xl border border-stone-700/70">
                        <div class="flex items-center justify-between mb-3">
                            <span class="text-xs font-black uppercase tracking-wider text-amber-300">Character & Weapon Wraps</span>
                            <span id="lobby-skin-badge" class="px-2.5 py-0.5 rounded text-[10px] font-black text-stone-950 bg-lime-400">RANGER OLIVE</span>
                        </div>

                        <div class="grid grid-cols-2 gap-2.5">
                            <button type="button" onclick="setLobbySkin('ranger')" id="skin-ranger"
                                    class="p-3 rounded-2xl bg-lime-500 text-stone-950 font-black text-xs uppercase text-left transition flex flex-col gap-1">
                                <span class="w-5 h-5 rounded-full bg-emerald-800 border-2 border-stone-950 inline-block"></span>
                                <span>Ranger Olive</span>
                                <span class="text-[10px] opacity-80 font-bold">Standard Issue</span>
                            </button>
                            <button type="button" onclick="setLobbySkin('crimson')" id="skin-crimson"
                                    class="p-3 rounded-2xl bg-stone-800 text-stone-300 hover:bg-stone-700 font-extrabold text-xs uppercase text-left transition flex flex-col gap-1">
                                <span class="w-5 h-5 rounded-full bg-red-600 border-2 border-stone-950 inline-block"></span>
                                <span>Crimson Ops</span>
                                <span class="text-[10px] opacity-75 font-semibold">Spec-Ops Red</span>
                            </button>
                            <button type="button" onclick="setLobbySkin('arctic')" id="skin-arctic"
                                    class="p-3 rounded-2xl bg-stone-800 text-stone-300 hover:bg-stone-700 font-extrabold text-xs uppercase text-left transition flex flex-col gap-1">
                                <span class="w-5 h-5 rounded-full bg-sky-300 border-2 border-stone-950 inline-block"></span>
                                <span>Arctic Ghost</span>
                                <span class="text-[10px] opacity-75 font-semibold">Frost White</span>
                            </button>
                            <button type="button" onclick="setLobbySkin('gold')" id="skin-gold"
                                    class="p-3 rounded-2xl bg-stone-800 text-stone-300 hover:bg-stone-700 font-extrabold text-xs uppercase text-left transition flex flex-col gap-1">
                                <span class="w-5 h-5 rounded-full bg-amber-400 border-2 border-stone-950 inline-block"></span>
                                <span>Gold Champion</span>
                                <span class="text-[10px] opacity-75 font-semibold">Legendary Gold</span>
                            </button>
                        </div>

                        <div class="mt-4">
                            <label class="block text-xs font-black uppercase tracking-wider text-lime-300 mb-1.5">Survivor Callsign</label>
                            <input id="lobby-player-name" type="text" value="Survivor-01" maxlength="16"
                                   oninput="syncLobbyCallsignPreview()"
                                   class="w-full px-3.5 py-2 rounded-xl bg-stone-950 border border-stone-700 focus:border-lime-400 focus:outline-none text-sm font-bold text-white" />
                        </div>
                    </div>

                    <div class="bg-stone-900/75 p-5 rounded-2xl border border-stone-700/70 flex flex-col justify-between">
                        <div>
                            <div class="flex items-center justify-between mb-2">
                                <span class="text-xs font-black uppercase tracking-wider text-sky-300">Live 3D Outfit Preview</span>
                                <span class="text-[10px] font-bold text-stone-400">Drag to rotate</span>
                            </div>
                            <div class="relative w-full h-52 rounded-2xl bg-gradient-to-b from-stone-950 via-emerald-950/50 to-stone-950 border border-stone-700/80 overflow-hidden">
                                <canvas id="locker-preview-canvas" class="w-full h-full block cursor-grab active:cursor-grabbing"></canvas>
                                <div class="absolute bottom-2 left-2.5 right-2.5 flex items-center justify-between text-[10px] font-bold pointer-events-none">
                                    <span id="locker-detail-outfit" class="px-2 py-0.5 rounded bg-stone-950/85 text-lime-300 border border-stone-700/70">Ranger Olive Tactical</span>
                                    <span id="locker-detail-tag" class="px-2 py-0.5 rounded bg-stone-950/85 text-sky-300 border border-stone-700/70">Survivor-01</span>
                                </div>
                            </div>
                        </div>

                        <button type="button" onclick="switchLobbyTab('gameplay')"
                                class="mt-4 w-full py-3 rounded-2xl bg-lime-400 hover:bg-lime-300 text-stone-950 font-black text-xs uppercase tracking-wider shadow-lg transition">
                            Equip & Return To Gameplay Tab
                        </button>
                    </div>
                </div>
            </div>
        </div>
    </div>

    <!-- Fullscreen 3D Canvas -->
    <div id="game-canvas-container"></div>

    <!-- Damage Flash & Hitmarker -->
    <div id="damage-flash" class="fixed inset-0 pointer-events-none z-10 opacity-0 transition-opacity duration-150"
         style="background: radial-gradient(circle, transparent 55%, rgba(220, 38, 38, 0.55) 100%);"></div>

    <div id="crosshair-wrap" class="fixed inset-0 pointer-events-none z-20 flex items-center justify-center">
        <div class="relative w-6 h-6 flex items-center justify-center">
            <div class="w-1 h-1 rounded-full bg-lime-300 shadow"></div>
            <div class="absolute top-0 w-0.5 h-1.5 bg-lime-300/80"></div>
            <div class="absolute bottom-0 w-0.5 h-1.5 bg-lime-300/80"></div>
            <div class="absolute left-0 h-0.5 w-1.5 bg-lime-300/80"></div>
            <div class="absolute right-0 h-0.5 w-1.5 bg-lime-300/80"></div>
            <div id="hitmarker" class="absolute inset-0 opacity-0 transition-opacity duration-75 flex items-center justify-center">
                <div class="w-4 h-0.5 bg-red-500 rotate-45 absolute"></div>
                <div class="w-4 h-0.5 bg-red-500 -rotate-45 absolute"></div>
            </div>
        </div>
    </div>

    <!-- Top-Left Status & Live Spectator / Speed Controls -->
    <div class="fixed top-4 left-4 z-20 flex flex-col gap-2 max-w-xs sm:max-w-sm">
        <div class="glass-panel px-3.5 py-2.5 rounded-2xl shadow-lg">
            <div class="flex items-center justify-between gap-3">
                <span id="hud-mode-badge" class="text-xs font-extrabold tracking-wide text-lime-300">DEAD SHOT (19 Zombies)</span>
                <span id="hud-submode-badge" class="px-2 py-0.5 rounded-full bg-lime-500/20 text-lime-300 text-[10px] font-bold uppercase">ZOMBIE SURVIVAL</span>
            </div>
            <div class="mt-1 text-[11px] text-stone-300">
                <span id="hud-aim-status" class="text-amber-300 font-semibold">Click screen to activate aim (or drag mouse)</span>
            </div>
            <div id="hud-party-live" class="hidden mt-1.5 pt-1.5 border-t border-stone-700/70 text-[10px] font-bold text-sky-300"></div>
            <div class="mt-2 pt-2 border-t border-stone-700/70 flex flex-wrap items-center justify-between gap-1.5">
                <button id="btn-sim-toggle" onclick="toggleSimMode()"
                        class="px-2.5 py-1 rounded-lg bg-stone-800 hover:bg-stone-700 text-emerald-300 font-bold text-[10px] transition">
                    [M] Mode: Manual Play
                </button>
                <div class="flex items-center gap-1">
                    <button id="spd-1" onclick="setSimSpeed(1)" class="px-2 py-0.5 rounded bg-lime-600 text-stone-950 font-bold text-[10px]">1x</button>
                    <button id="spd-2" onclick="setSimSpeed(2)" class="px-2 py-0.5 rounded bg-stone-800 text-stone-300 font-bold text-[10px]">2x</button>
                    <button id="spd-4" onclick="setSimSpeed(4)" class="px-2 py-0.5 rounded bg-stone-800 text-stone-300 font-bold text-[10px]">4x</button>
                </div>
            </div>
        </div>

        <div id="kill-feed" class="flex flex-col gap-1 pointer-events-none"></div>
    </div>

    <!-- Top-Right Minimap -->
    <div class="fixed top-4 right-4 z-20 flex flex-col items-end gap-2">
        <div class="glass-panel p-2.5 rounded-2xl shadow-lg">
            <canvas id="minimap-canvas" width="144" height="144" class="rounded-xl bg-stone-950/90 border border-stone-700/80"></canvas>
            <div class="mt-1.5 flex items-center justify-between text-[10px] font-semibold text-stone-300 px-1">
                <span class="flex items-center gap-1"><span class="w-2 h-2 rounded-full bg-lime-400 inline-block"></span>You</span>
                <span class="flex items-center gap-1"><span class="w-2 h-2 rounded-full bg-sky-400 inline-block"></span>Squad/PvP</span>
                <span class="flex items-center gap-1"><span class="w-2 h-2 rounded-full bg-red-500 inline-block"></span>Enemies</span>
                <span class="flex items-center gap-1"><span class="w-2 h-2 rounded-full bg-amber-400 inline-block"></span>Loot</span>
            </div>
        </div>
        <button onclick="openPauseMenu()" class="glass-panel px-3 py-1.5 rounded-xl text-xs font-bold text-stone-200 hover:border-lime-400 transition">
            [P] Tactical Menu
        </button>
    </div>

    <!-- Center Prompts (Loot & Medkit Progress) -->
    <div id="loot-prompt" class="hidden fixed bottom-36 left-1/2 -translate-x-1/2 z-20 glass-panel px-4 py-2 rounded-full text-xs font-extrabold text-amber-300 shadow-lg">
        Press <span class="px-1.5 py-0.5 bg-amber-400 text-stone-950 rounded mx-1">E</span> to Open Supply Chest
    </div>

    <div id="medkit-progress-wrap" class="hidden fixed bottom-44 left-1/2 -translate-x-1/2 z-20 glass-panel px-5 py-3 rounded-2xl w-64 text-center shadow-xl">
        <div class="flex justify-between text-xs font-bold text-emerald-300 mb-1.5">
            <span>Applying Med Kit...</span>
            <span id="medkit-timer-text" class="mono">3.0s</span>
        </div>
        <div class="w-full h-2.5 bg-stone-800 rounded-full overflow-hidden">
            <div id="medkit-progress-bar" class="h-full bg-gradient-to-r from-emerald-400 to-lime-400 w-0 transition-all duration-75"></div>
        </div>
    </div>

    <!-- Bottom HUD: Health, Kills, Wood & Weapons / Build Hotbar -->
    <div class="fixed bottom-4 left-4 right-4 z-20 flex flex-col sm:flex-row items-center justify-between gap-3 pointer-events-none">
        <div class="flex items-center gap-2 pointer-events-auto">
            <div class="glass-panel px-4 py-2.5 rounded-2xl min-w-[180px]">
                <div class="flex items-baseline justify-between">
                    <span class="text-[10px] font-extrabold uppercase tracking-wider text-stone-400">Health</span>
                    <span class="mono text-lg font-black text-lime-400"><span id="hud-health-val">100</span> <span class="text-xs text-stone-400">/ 100</span></span>
                </div>
                <div class="w-full h-2.5 bg-stone-800 rounded-full overflow-hidden mt-1">
                    <div id="hud-health-bar" class="h-full bg-gradient-to-r from-lime-400 to-emerald-400 w-full transition-all duration-150"></div>
                </div>
            </div>

            <div class="glass-panel px-3.5 py-2.5 rounded-2xl text-center">
                <div class="text-[10px] font-extrabold uppercase tracking-wider text-stone-400">Elims</div>
                <div id="hud-kills-val" class="mono text-lg font-black text-red-400">0</div>
            </div>

            <div id="hud-wood-panel" class="glass-panel px-3.5 py-2.5 rounded-2xl text-center">
                <div class="text-[10px] font-extrabold uppercase tracking-wider text-amber-300">Wood</div>
                <div id="hud-wood-count" class="mono text-lg font-black text-amber-400">150</div>
            </div>
        </div>

        <div class="flex flex-col items-center gap-1.5 pointer-events-auto">
            <div id="hud-build-bar" class="hidden items-center gap-1.5 glass-panel px-3 py-1.5 rounded-xl text-[11px] font-extrabold">
                <span class="text-sky-300 mr-1">BUILD:</span>
                <button id="bpiece-wall" onclick="selectBuildPiece('wall')" class="px-2 py-0.5 rounded bg-stone-800 text-white border-2 border-sky-400">[Z] Wall</button>
                <button id="bpiece-floor" onclick="selectBuildPiece('floor')" class="px-2 py-0.5 rounded bg-stone-800 text-white border border-stone-700">[X] Floor</button>
                <button id="bpiece-ramp" onclick="selectBuildPiece('ramp')" class="px-2 py-0.5 rounded bg-stone-800 text-white border border-stone-700">[C] Ramp</button>
            </div>

            <div id="hud-weapon-bar" class="glass-panel px-4 py-1.5 rounded-xl flex items-center gap-3 text-xs">
                <span id="hud-weapon-name" class="font-black text-white">M9 TACTICAL PISTOL</span>
                <span class="mono font-bold text-lime-400"><span id="hud-ammo-mag">15</span> <span id="hud-ammo-res" class="text-stone-400">/ 90</span></span>
                <span id="hud-controls-hint" class="text-[10px] text-stone-400 hidden md:inline">[Q] Build • [H] Medkit</span>
            </div>

            <div class="flex items-center gap-1.5">
                <button id="slot-0" onclick="switchWeaponSlot(0)" class="glass-panel px-2.5 py-1.5 rounded-xl text-[11px] font-bold border border-stone-700">1: M4A1</button>
                <button id="slot-1" onclick="switchWeaponSlot(1)" class="glass-panel px-2.5 py-1.5 rounded-xl text-[11px] font-bold border border-stone-700">2: M870</button>
                <button id="slot-2" onclick="switchWeaponSlot(2)" class="glass-panel px-2.5 py-1.5 rounded-xl text-[11px] font-bold border-2 border-lime-400">3: M9</button>
                <button id="slot-3" onclick="switchWeaponSlot(3)" class="glass-panel px-2.5 py-1.5 rounded-xl text-[11px] font-bold border border-stone-700">4: KNIFE</button>
                <button id="slot-4" onclick="startMedkitHeal()" class="glass-panel px-2.5 py-1.5 rounded-xl text-[11px] font-bold border border-stone-700 text-emerald-300">5: <span id="slot-4-label">MED (0)</span></button>
            </div>
        </div>
    </div>

    <!-- Tactical Pause Menu Overlay -->
    <div id="pause-menu-overlay" class="hidden fixed inset-0 z-40 items-center justify-center bg-stone-950/80 backdrop-blur-sm p-4">
        <div class="glass-panel max-w-lg w-full p-6 rounded-3xl shadow-2xl">
            <div class="flex items-center justify-between border-b border-stone-700 pb-4">
                <div>
                    <h2 class="text-2xl font-black text-white">DEAD SHOT — PAUSED</h2>
                    <p class="text-xs text-stone-400">WASD Move • Mouse Aim • Click Shoot • Q Build • H Medkit</p>
                </div>
                <button onclick="resumeGameplay()" class="px-4 py-2 rounded-xl bg-lime-400 hover:bg-lime-300 text-stone-950 font-black text-xs uppercase">Resume</button>
            </div>

            <div class="grid grid-cols-1 sm:grid-cols-2 gap-4 mt-4">
                <div class="bg-stone-900/75 p-4 rounded-2xl border border-stone-700/70 flex flex-col justify-between">
                    <div class="text-xs text-stone-300 space-y-1.5">
                        <div class="font-bold text-lime-300">Combat Rules</div>
                        <div>• Player & Enemy Health: 100 HP</div>
                        <div>• Headshot: 25 DMG | Body: 15 DMG</div>
                        <div>• Med Kit: 3.0s channel to 100 HP</div>
                    </div>
                    <div class="mt-4 flex flex-col gap-2">
                        <button onclick="resetSoloMatch()" class="w-full py-2.5 rounded-xl bg-emerald-600 hover:bg-emerald-500 text-white font-extrabold text-xs uppercase tracking-wider transition">
                            Restart Match
                        </button>
                        <button onclick="returnToMainLobby()" class="w-full py-2.5 rounded-xl bg-stone-800 hover:bg-stone-700 text-stone-200 font-extrabold text-xs uppercase tracking-wider transition">
                            Return To Lobby
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

    <!-- Victory / Elimination Banner Overlay -->
    <div id="match-end-overlay" class="hidden fixed inset-0 z-40 items-center justify-center bg-stone-950/85 backdrop-blur-md p-4">
        <div class="glass-panel max-w-md w-full p-7 rounded-3xl text-center shadow-2xl border border-lime-400/40">
            <div id="end-banner-tag" class="inline-block px-3 py-1 rounded-full bg-lime-400/20 text-lime-300 text-xs font-black uppercase tracking-widest mb-2">
                VICTORY ROYALE
            </div>
            <h2 id="end-banner-title" class="text-3xl sm:text-4xl font-black text-white">VICTORY ROYALE!</h2>
            <p id="end-banner-sub" class="text-xs text-stone-300 mt-2">You outlasted every threat in Dead Shot.</p>

            <div class="grid grid-cols-2 gap-3 my-5">
                <div class="bg-stone-900/90 p-3 rounded-2xl border border-stone-700">
                    <div class="text-[10px] font-bold uppercase text-stone-400">Eliminations</div>
                    <div id="end-stat-kills" class="mono text-2xl font-black text-red-400 mt-0.5">0</div>
                </div>
                <div class="bg-stone-900/90 p-3 rounded-2xl border border-stone-700">
                    <div class="text-[10px] font-bold uppercase text-stone-400">Placement</div>
                    <div id="end-stat-place" class="mono text-2xl font-black text-lime-400 mt-0.5">#1 / 20</div>
                </div>
            </div>

            <div class="flex gap-2.5">
                <button onclick="resetSoloMatch()" class="flex-1 py-3 rounded-2xl bg-lime-400 hover:bg-lime-300 text-stone-950 font-black text-xs uppercase tracking-wider transition">
                    Play Again
                </button>
                <button onclick="returnToMainLobby()" class="flex-1 py-3 rounded-2xl bg-stone-800 hover:bg-stone-700 text-white font-black text-xs uppercase tracking-wider transition">
                    Main Lobby
                </button>
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
        const BOT_HEADSHOT_DAMAGE = 18;
        const BOT_BODYSHOT_DAMAGE = 10;
        const ZOMBIE_CLAW_DAMAGE = 12;
        const BOT_COUNT = 19;
        const MEDKIT_DURATION = 3.0;

        const ZOMBIE_NAMES = [
            "Walker-01", "Rotter-02", "Stalker-03", "Shambler-04", "Crawler-05",
            "Biter-06", "Ghoul-07", "Lurker-08", "Infected-09", "Husk-10",
            "Brute-11", "Gnasher-12", "Revenant-13", "Spitter-14", "Decay-15",
            "Howler-16", "Fiend-17", "Wretch-18", "Apex-19"
        ];

        const BOT_CALLSIGNS = [
            "Sgt. Vance", "Cpl. Mercer", "Pvt. Kestrel", "Lt. Thorne", "Spec. Rook",
            "Capt. Strider", "Sgt. Nomad", "Cpl. Vector", "Pvt. Havoc", "Lt. Frost",
            "Spec.Cipher", "Sgt. Talon", "Cpl. Orion", "Pvt. Cobalt", "Lt. Reaper",
            "Spec. Warden", "Sgt. Apex", "Cpl. Lynx", "Maj.Titan"
        ];

        let gameMode = "zombies";   // "zombies", "classic", or "pvp"
        let buildEnabled = true;    // true = Build Mode, false = Zero Build
        let lobbyDifficulty = "normal";
        let selectedMap = "woodland";
        let selectedSkin = "ranger";
        let inMainLobby = true;

        // Multiplayer State
        let mpActiveRoom = null;
        let mpRemotePlayers = {};   // uid -> { data, mesh, targetPos }
        let mpRenderedBuilds = new Set();
        let mpSyncAccumulator = 0;
        let mpUnsubRoom = null;
        let mpUnsubPlayers = null;
        let mpUnsubEvents = null;
        let mpUnsubBuilds = null;
        let mpProcessedEvents = new Set();

        const PLAYER_SKINS = {
            ranger:  { label: "Ranger Olive",  sleeve: 0x3b4d34, glove: 0x1f2421, gunPrimary: 0x1e2226, gunAccent: 0x3a4736, metalness: 0.75, hudHex: "#a3e635" },
            crimson: { label: "Crimson Ops",   sleeve: 0x991b1b, glove: 0x18181b, gunPrimary: 0x27272a, gunAccent: 0xdc2626, metalness: 0.80, hudHex: "#f87171" },
            arctic:  { label: "Arctic Ghost",  sleeve: 0xe2e8f0, glove: 0x334155, gunPrimary: 0xcbd5e1, gunAccent: 0x38bdf8, metalness: 0.65, hudHex: "#38bdf8" },
            gold:    { label: "Gold Champion", sleeve: 0x1c1917, glove: 0xd97706, gunPrimary: 0xfacc15, gunAccent: 0xb45309, metalness: 0.92, hudHex: "#facc15" }
        };

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
        let firstPersonArmGroup, sleeveMesh, gloveMesh;
        let skinPrimaryMat, skinAccentMat;
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
            const skinCfg = PLAYER_SKINS[selectedSkin] || PLAYER_SKINS.ranger;
            skinPrimaryMat = new THREE.MeshStandardMaterial({ color: skinCfg.gunPrimary, roughness: 0.35, metalness: skinCfg.metalness });
            skinAccentMat = new THREE.MeshStandardMaterial({ color: skinCfg.gunAccent, roughness: 0.5, metalness: 0.4 });
            const woodMat = new THREE.MeshStandardMaterial({ color: 0x6b4226, roughness: 0.8 });
            const silverBlade = new THREE.MeshStandardMaterial({ color: 0xd4d8dd, roughness: 0.25, metalness: 0.9 });

            // First-Person Right Forearm & Tactical Glove (changes color with equipped skin)
            firstPersonArmGroup = new THREE.Group();
            const sleeveMat = new THREE.MeshStandardMaterial({ color: skinCfg.sleeve, roughness: 0.8 });
            const gloveMat = new THREE.MeshStandardMaterial({ color: skinCfg.glove, roughness: 0.6 });
            sleeveMesh = new THREE.Mesh(new THREE.BoxGeometry(0.085, 0.085, 0.34), sleeveMat);
            sleeveMesh.position.set(0.02, -0.09, 0.22);
            sleeveMesh.rotation.x = 0.12;
            gloveMesh = new THREE.Mesh(new THREE.BoxGeometry(0.078, 0.078, 0.12), gloveMat);
            gloveMesh.position.set(0.01, -0.075, 0.04);
            firstPersonArmGroup.add(sleeveMesh, gloveMesh);
            weaponHolder.add(firstPersonArmGroup);

            // Slot 0: M4A1 Carbine
            const m4 = new THREE.Group();
            const m4Receiver = new THREE.Mesh(new THREE.BoxGeometry(0.07, 0.09, 0.42), skinPrimaryMat);
            const m4Barrel = new THREE.Mesh(new THREE.CylinderGeometry(0.014, 0.014, 0.34, 8), skinPrimaryMat);
            m4Barrel.rotation.x = Math.PI / 2;
            m4Barrel.position.set(0, 0.015, -0.34);
            const m4Handguard = new THREE.Mesh(new THREE.BoxGeometry(0.06, 0.065, 0.24), skinAccentMat);
            m4Handguard.position.set(0, 0.01, -0.22);
            const m4Mag = new THREE.Mesh(new THREE.BoxGeometry(0.045, 0.16, 0.07), skinAccentMat);
            m4Mag.position.set(0, -0.09, -0.06);
            m4Mag.rotation.x = 0.2;
            m4.add(m4Receiver, m4Barrel, m4Handguard, m4Mag);
            weaponHolder.add(m4);
            weaponMeshes[0] = m4;

            // Slot 1: M870 Breacher Shotgun
            const m870 = new THREE.Group();
            const sgBody = new THREE.Mesh(new THREE.BoxGeometry(0.075, 0.085, 0.46), skinPrimaryMat);
            const sgBarrel = new THREE.Mesh(new THREE.CylinderGeometry(0.02, 0.02, 0.42, 10), skinPrimaryMat);
            sgBarrel.rotation.x = Math.PI / 2;
            sgBarrel.position.set(0, 0.02, -0.36);
            const sgPump = new THREE.Mesh(new THREE.BoxGeometry(0.082, 0.07, 0.18), skinAccentMat);
            sgPump.position.set(0, -0.01, -0.28);
            m870.add(sgBody, sgBarrel, sgPump);
            weaponHolder.add(m870);
            weaponMeshes[1] = m870;

            // Slot 2: M9 Tactical Pistol
            const m9 = new THREE.Group();
            const pSlide = new THREE.Mesh(new THREE.BoxGeometry(0.055, 0.065, 0.24), skinPrimaryMat);
            const pGrip = new THREE.Mesh(new THREE.BoxGeometry(0.05, 0.13, 0.065), skinAccentMat);
            pGrip.position.set(0, -0.07, 0.06);
            pGrip.rotation.x = -0.22;
            m9.add(pSlide, pGrip);
            weaponHolder.add(m9);
            weaponMeshes[2] = m9;

            // Slot 3: Combat Knife
            const knife = new THREE.Group();
            const kHandle = new THREE.Mesh(new THREE.CylinderGeometry(0.022, 0.025, 0.14, 8), skinAccentMat);
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
            const handleBar = new THREE.Mesh(new THREE.BoxGeometry(0.12, 0.025, 0.03), woodMat);
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

            if (mpActiveRoom && window.mpPublishBuild) {
                window.mpPublishBuild({
                    type: player.buildPiece,
                    x: snap.gx,
                    z: snap.gz,
                    baseY: snap.gy,
                    rotY: snap.cardRot
                });
            }
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
                // Blockpost 2.0 Voxel Tactical Soldier Style for Classic Battle Royale Bots
                const oliveTunicMat = new THREE.MeshStandardMaterial({ color: 0x3b4d34, roughness: 0.72 });
                const tacticalVestMat = new THREE.MeshStandardMaterial({ color: 0x222920, roughness: 0.6 });
                const pouchMat = new THREE.MeshStandardMaterial({ color: 0x4b5542, roughness: 0.65 });
                const cargoPantsMat = new THREE.MeshStandardMaterial({ color: 0x4b533c, roughness: 0.78 });
                const kneePadMat = new THREE.MeshStandardMaterial({ color: 0x18181b, roughness: 0.5 });
                const helmetMat = new THREE.MeshStandardMaterial({ color: 0x2d3a27, roughness: 0.45, metalness: 0.25 });
                const skinFaceMat = new THREE.MeshStandardMaterial({ color: 0xd6a57c, roughness: 0.65 });
                const goggleFrameMat = new THREE.MeshStandardMaterial({ color: 0x18181b, roughness: 0.4 });
                const goggleLensMat = new THREE.MeshBasicMaterial({ color: 0x38bdf8 });
                const bootMat = new THREE.MeshStandardMaterial({ color: 0x18181b, roughness: 0.7 });
                const gunMat = new THREE.MeshStandardMaterial({ color: 0x18181b, metalness: 0.8, roughness: 0.3 });

                // Chunky Blockpost Legs + Knee Pads + Boots
                const leftLeg = new THREE.Mesh(new THREE.BoxGeometry(0.28, 0.58, 0.28), cargoPantsMat);
                leftLeg.position.set(-0.17, 0.47, 0);
                leftLeg.castShadow = true;

                const rightLeg = new THREE.Mesh(new THREE.BoxGeometry(0.28, 0.58, 0.28), cargoPantsMat);
                rightLeg.position.set(0.17, 0.47, 0);
                rightLeg.castShadow = true;

                const leftKnee = new THREE.Mesh(new THREE.BoxGeometry(0.3, 0.14, 0.31), kneePadMat);
                leftKnee.position.set(-0.17, 0.48, 0.02);
                const rightKnee = new THREE.Mesh(new THREE.BoxGeometry(0.3, 0.14, 0.31), kneePadMat);
                rightKnee.position.set(0.17, 0.48, 0.02);

                const leftBoot = new THREE.Mesh(new THREE.BoxGeometry(0.3, 0.2, 0.35), bootMat);
                leftBoot.position.set(-0.17, 0.1, 0.03);
                const rightBoot = new THREE.Mesh(new THREE.BoxGeometry(0.3, 0.2, 0.35), bootMat);
                rightBoot.position.set(0.17, 0.1, 0.03);

                // Blocky Tactical Torso + Chest Rig + Ammo Pouches + Backpack
                const torso = new THREE.Mesh(new THREE.BoxGeometry(0.68, 0.72, 0.38), oliveTunicMat);
                torso.position.set(0, 1.14, 0);
                torso.castShadow = true;

                const vest = new THREE.Mesh(new THREE.BoxGeometry(0.72, 0.56, 0.44), tacticalVestMat);
                vest.position.set(0, 1.16, 0);

                const pouchL = new THREE.Mesh(new THREE.BoxGeometry(0.16, 0.22, 0.1), pouchMat);
                pouchL.position.set(-0.16, 1.08, 0.24);
                const pouchR = new THREE.Mesh(new THREE.BoxGeometry(0.16, 0.22, 0.1), pouchMat);
                pouchR.position.set(0.16, 1.08, 0.24);

                const backpack = new THREE.Mesh(new THREE.BoxGeometry(0.5, 0.54, 0.22), pouchMat);
                backpack.position.set(0, 1.18, -0.28);

                // Blocky Shoulder Pads & Arms
                const leftArm = new THREE.Mesh(new THREE.BoxGeometry(0.24, 0.64, 0.24), oliveTunicMat);
                leftArm.position.set(-0.48, 1.16, 0.04);
                const rightArm = new THREE.Mesh(new THREE.BoxGeometry(0.24, 0.64, 0.24), oliveTunicMat);
                rightArm.position.set(0.48, 1.16, 0.12);
                rightArm.rotation.x = -0.45;

                const leftShoulder = new THREE.Mesh(new THREE.BoxGeometry(0.28, 0.22, 0.28), tacticalVestMat);
                leftShoulder.position.set(-0.49, 1.38, 0.02);
                const rightShoulder = new THREE.Mesh(new THREE.BoxGeometry(0.28, 0.22, 0.28), tacticalVestMat);
                rightShoulder.position.set(0.49, 1.38, 0.02);

                // Signature Oversized Blockpost Cubic Head + Tactical Helmet + Visor Goggles
                const head = new THREE.Mesh(new THREE.BoxGeometry(0.56, 0.56, 0.56), skinFaceMat);
                head.position.set(0, 1.8, 0);
                head.castShadow = true;

                const helmetTop = new THREE.Mesh(new THREE.BoxGeometry(0.62, 0.28, 0.62), helmetMat);
                helmetTop.position.set(0, 1.98, -0.01);
                const helmetRim = new THREE.Mesh(new THREE.BoxGeometry(0.64, 0.08, 0.65), helmetMat);
                helmetRim.position.set(0, 1.88, 0.02);

                const goggleFrame = new THREE.Mesh(new THREE.BoxGeometry(0.58, 0.2, 0.1), goggleFrameMat);
                goggleFrame.position.set(0, 1.81, 0.27);
                const goggleGlass = new THREE.Mesh(new THREE.BoxGeometry(0.48, 0.13, 0.04), goggleLensMat);
                goggleGlass.position.set(0, 1.81, 0.31);

                const gun = new THREE.Mesh(new THREE.BoxGeometry(0.11, 0.14, 0.72), gunMat);
                gun.position.set(0.46, 1.16, 0.36);

                root.add(
                    leftLeg, rightLeg, leftKnee, rightKnee, leftBoot, rightBoot,
                    torso, vest, pouchL, pouchR, backpack,
                    leftArm, rightArm, leftShoulder, rightShoulder,
                    head, helmetTop, helmetRim, goggleFrame, goggleGlass, gun
                );
                return { root, head, torso, leftLeg, rightLeg, leftArm, rightArm };
            }
        }

        function spawnAllBots() {
            bots.forEach(b => scene.remove(b.mesh.root));
            bots = [];

            if (gameMode === "pvp") return;

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
                    fireCooldown: 0.6 + Math.random() * 1.2,
                    strafeDir: Math.random() > 0.5 ? 1 : -1,
                    strafeTimer: 1.5 + Math.random() * 2,
                    walkPhase: Math.random() * 6,
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
            const remoteHeadTargets = [];
            const remoteBodyTargets = [];

            bots.forEach(b => {
                if (!b.alive) return;
                b.mesh.head.userData = { bot: b, isHead: true };
                b.mesh.torso.userData = { bot: b, isHead: false };
                b.mesh.leftLeg.userData = { bot: b, isHead: false };
                b.mesh.rightLeg.userData = { bot: b, isHead: false };
                headTargets.push(b.mesh.head);
                bodyTargets.push(b.mesh.torso, b.mesh.leftLeg, b.mesh.rightLeg);
            });

            Object.keys(mpRemotePlayers).forEach(uid => {
                const rp = mpRemotePlayers[uid];
                if (!rp || !rp.data || !rp.data.alive) return;
                rp.mesh.head.userData = { remoteUid: uid, remoteName: rp.data.name, isHead: true };
                rp.mesh.torso.userData = { remoteUid: uid, remoteName: rp.data.name, isHead: false };
                rp.mesh.leftLeg.userData = { remoteUid: uid, remoteName: rp.data.name, isHead: false };
                rp.mesh.rightLeg.userData = { remoteUid: uid, remoteName: rp.data.name, isHead: false };
                remoteHeadTargets.push(rp.mesh.head);
                remoteBodyTargets.push(rp.mesh.torso, rp.mesh.leftLeg, rp.mesh.rightLeg);
            });

            const allTargets = [...headTargets, ...bodyTargets, ...remoteHeadTargets, ...remoteBodyTargets];

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
                    const uData = hit.object.userData;
                    if (w.id !== "knife") spawnTracer(muzzlePos, hit.point, 0xfef08a);
                    spawnImpactSparks(hit.point, uData.isHead ? 0xef4444 : 0xfacc15);
                    flashHitmarker();
                    if (uData.bot) {
                        applyDamageToBot(uData.bot, uData.isHead ? HEADSHOT_DAMAGE : BODYSHOT_DAMAGE, player.name, true, uData.isHead);
                    } else if (uData.remoteUid && window.mpSendHitEvent) {
                        const dmg = uData.isHead ? HEADSHOT_DAMAGE : BODYSHOT_DAMAGE;
                        window.mpSendHitEvent(uData.remoteUid, uData.remoteName, dmg, uData.isHead);
                    }
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
            if (gameMode === "pvp") {
                const alivePeers = Object.values(mpRemotePlayers).filter(rp => rp.data && rp.data.alive).length;
                if (Object.keys(mpRemotePlayers).length > 0 && alivePeers === 0 && player.alive) {
                    showMatchEndBanner(true, "PVP CHAMPION!", "You defeated all online players in the arena.", "#1");
                }
                return;
            }
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
            const onlinePeers = Object.values(mpRemotePlayers).filter(rp => rp.data && rp.data.alive).length;
            if (gameMode === "pvp") {
                document.getElementById("hud-mode-badge").textContent = `PVP ARENA (${onlinePeers + 1} Live Players)`;
                document.getElementById("hud-submode-badge").textContent = mpActiveRoom ? `ROOM: ${mpActiveRoom}` : "ONLINE PVP";
            } else {
                const enemyNoun = gameMode === "zombies" ? "Zombies" : "Enemies";
                const roomTag = mpActiveRoom ? ` • RM ${mpActiveRoom}` : "";
                document.getElementById("hud-mode-badge").textContent = `DEAD SHOT (${aliveEnemies} ${enemyNoun}${roomTag})`;
                document.getElementById("hud-submode-badge").textContent = gameMode === "zombies" ? "ZOMBIE SURVIVAL" : "CLASSIC BR";
            }

            const buildBar = document.getElementById("hud-build-bar");
            const weaponBar = document.getElementById("hud-weapon-bar");

            if (player.buildMode) {
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

            isPaused = false;
            cancelMedkitHeal();

            builtStructures.forEach(s => { if (s.mesh) scene.remove(s.mesh); });
            builtStructures = [];
            buildSelectedMap(selectedMap);

            player.pos.set(0, 0, 0);
            player.vel.set(0, 0, 0);
            player.yaw = 0;
            player.pitch = 0;
            if (mpActiveRoom && window.mpGetSpawnCoordinates) {
                const sp = window.mpGetSpawnCoordinates();
                player.pos.set(sp.x, 0, sp.z);
                player.yaw = sp.yaw;
            }
            player.health = PLAYER_MAX_HEALTH;
            player.alive = true;
            player.kills = 0;
            player.grounded = true;
            player.reloading = false;
            player.medkits = 0;
            player.wood = buildEnabled ? 150 : 0;
            player.buildMode = false;
            player.buildPiece = "wall";
            if (blueprintViewModel) blueprintViewModel.visible = false;
            if (buildPreviewGroup) buildPreviewGroup.visible = false;
            updateWoodHUD();

            player.weapons[0].owned = false; player.weapons[0].mag = 30; player.weapons[0].reserve = 150;
            player.weapons[1].owned = false; player.weapons[1].mag = 6;  player.weapons[1].reserve = 36;
            player.weapons[2].owned = true;  player.weapons[2].mag = 15; player.weapons[2].reserve = 90;
            player.weapons[3].owned = true;
            player.weapons[4].owned = false; player.weapons[4].mag = 0;

            switchWeaponSlot(2);
            spawnSupplyChests();
            spawnAllBots();
            updateHUD();
            updateRosterUI();

            const modeLabel = gameMode === "zombies" ? "Zombie Survival" : (gameMode === "classic" ? "Classic BR" : "PvP Arena");
            addKillFeedEntry(`Deployed into ${selectedMap.toUpperCase()} (${modeLabel}) — 100 HP`, "text-lime-300");
        }

        function switchLobbyTab(tabName) {
            const gpTab = document.getElementById("lobby-tab-gameplay");
            const lkTab = document.getElementById("lobby-tab-locker");
            const gpBtn = document.getElementById("tab-btn-gameplay");
            const lkBtn = document.getElementById("tab-btn-locker");

            if (tabName === "locker") {
                if (gpTab) gpTab.classList.add("hidden");
                if (lkTab) lkTab.classList.remove("hidden");
                if (lkBtn) lkBtn.className = "px-4 py-1.5 rounded-xl bg-lime-400 text-stone-950 font-black text-xs uppercase tracking-wider shadow transition";
                if (gpBtn) gpBtn.className = "px-4 py-1.5 rounded-xl text-stone-300 hover:text-white font-extrabold text-xs uppercase tracking-wider transition";
                initLockerPreviewScene();
                resizeLockerPreview();
            } else {
                if (lkTab) lkTab.classList.add("hidden");
                if (gpTab) gpTab.classList.remove("hidden");
                if (gpBtn) gpBtn.className = "px-4 py-1.5 rounded-xl bg-lime-400 text-stone-950 font-black text-xs uppercase tracking-wider shadow transition";
                if (lkBtn) lkBtn.className = "px-4 py-1.5 rounded-xl text-stone-300 hover:text-white font-extrabold text-xs uppercase tracking-wider transition";
            }
        }

        // =========================================================================
        // 11B. LOCKER 3D CHARACTER & WEAPON SKIN PREVIEW RENDERER
        // =========================================================================
        let lockerScene, lockerCamera, lockerRenderer, lockerCharGroup;
        let lkBodyMat, lkVestMat, lkHeadMat, lkGunPrimaryMat, lkGunAccentMat;
        let lkDragActive = false, lkLastX = 0, lkManualYaw = 0.45;

        function initLockerPreviewScene() {
            const canvas = document.getElementById("locker-preview-canvas");
            if (!canvas || lockerRenderer) return;

            lockerScene = new THREE.Scene();
            lockerCamera = new THREE.PerspectiveCamera(40, 1.6, 0.1, 30);
            lockerCamera.position.set(0, 1.32, 3.35);
            lockerCamera.lookAt(0, 1.05, 0);

            lockerRenderer = new THREE.WebGLRenderer({ canvas, antialias: true, alpha: true });
            lockerRenderer.setPixelRatio(Math.min(window.devicePixelRatio, 2));

            const hemi = new THREE.HemisphereLight(0xffffff, 0x1c1917, 1.15);
            const keyLight = new THREE.DirectionalLight(0xfffbeb, 1.25);
            keyLight.position.set(2.5, 4, 3.5);
            const rimLight = new THREE.DirectionalLight(0x38bdf8, 0.65);
            rimLight.position.set(-2.5, 2, -2.5);
            lockerScene.add(hemi, keyLight, rimLight);

            const cfg = PLAYER_SKINS[selectedSkin] || PLAYER_SKINS.ranger;
            lkBodyMat = new THREE.MeshStandardMaterial({ color: cfg.sleeve, roughness: 0.65 });
            lkVestMat = new THREE.MeshStandardMaterial({ color: cfg.gunAccent, roughness: 0.5 });
            lkHeadMat = new THREE.MeshStandardMaterial({ color: cfg.glove, roughness: 0.45 });
            lkGunPrimaryMat = new THREE.MeshStandardMaterial({ color: cfg.gunPrimary, roughness: 0.3, metalness: cfg.metalness });
            lkGunAccentMat = new THREE.MeshStandardMaterial({ color: cfg.gunAccent, roughness: 0.45 });
            const lkSkinFaceMat = new THREE.MeshStandardMaterial({ color: 0xd6a57c, roughness: 0.65 });
            const lkDarkGearMat = new THREE.MeshStandardMaterial({ color: 0x18181b, roughness: 0.5 });

            lockerCharGroup = new THREE.Group();

            // Pedestal ring
            const pedestal = new THREE.Mesh(
                new THREE.CylinderGeometry(0.78, 0.88, 0.1, 24),
                new THREE.MeshStandardMaterial({ color: 0x1e293b, metalness: 0.7, roughness: 0.3 })
            );
            pedestal.position.y = -0.05;

            // Blockpost 2.0 Chunky Voxel Legs + Knee Pads + Boots
            const leftLeg = new THREE.Mesh(new THREE.BoxGeometry(0.28, 0.58, 0.28), lkHeadMat);
            leftLeg.position.set(-0.17, 0.47, 0);
            const rightLeg = new THREE.Mesh(new THREE.BoxGeometry(0.28, 0.58, 0.28), lkHeadMat);
            rightLeg.position.set(0.17, 0.47, 0);

            const leftKnee = new THREE.Mesh(new THREE.BoxGeometry(0.3, 0.14, 0.31), lkDarkGearMat);
            leftKnee.position.set(-0.17, 0.48, 0.02);
            const rightKnee = new THREE.Mesh(new THREE.BoxGeometry(0.3, 0.14, 0.31), lkDarkGearMat);
            rightKnee.position.set(0.17, 0.48, 0.02);

            const leftBoot = new THREE.Mesh(new THREE.BoxGeometry(0.3, 0.2, 0.35), lkDarkGearMat);
            leftBoot.position.set(-0.17, 0.1, 0.03);
            const rightBoot = new THREE.Mesh(new THREE.BoxGeometry(0.3, 0.2, 0.35), lkDarkGearMat);
            rightBoot.position.set(0.17, 0.1, 0.03);

            // Blockpost Voxel Torso + Tactical Vest + Chest Pouches + Backpack
            const torso = new THREE.Mesh(new THREE.BoxGeometry(0.68, 0.72, 0.38), lkBodyMat);
            torso.position.set(0, 1.14, 0);
            const vest = new THREE.Mesh(new THREE.BoxGeometry(0.72, 0.56, 0.44), lkVestMat);
            vest.position.set(0, 1.16, 0);

            const pouchL = new THREE.Mesh(new THREE.BoxGeometry(0.16, 0.22, 0.1), lkHeadMat);
            pouchL.position.set(-0.16, 1.08, 0.24);
            const pouchR = new THREE.Mesh(new THREE.BoxGeometry(0.16, 0.22, 0.1), lkHeadMat);
            pouchR.position.set(0.16, 1.08, 0.24);

            const backpack = new THREE.Mesh(new THREE.BoxGeometry(0.5, 0.54, 0.22), lkHeadMat);
            backpack.position.set(0, 1.18, -0.28);

            // Blocky Shoulder Pads & Arms
            const leftArm = new THREE.Mesh(new THREE.BoxGeometry(0.24, 0.64, 0.24), lkBodyMat);
            leftArm.position.set(-0.48, 1.15, 0.08);
            leftArm.rotation.x = -0.35;
            const rightArm = new THREE.Mesh(new THREE.BoxGeometry(0.24, 0.64, 0.24), lkBodyMat);
            rightArm.position.set(0.48, 1.15, 0.16);
            rightArm.rotation.x = -0.65;

            const leftShoulder = new THREE.Mesh(new THREE.BoxGeometry(0.28, 0.22, 0.28), lkVestMat);
            leftShoulder.position.set(-0.49, 1.38, 0.02);
            const rightShoulder = new THREE.Mesh(new THREE.BoxGeometry(0.28, 0.22, 0.28), lkVestMat);
            rightShoulder.position.set(0.49, 1.38, 0.02);

            // Oversized Blockpost Cubic Head + Combat Helmet + Glowing Goggles
            const headFace = new THREE.Mesh(new THREE.BoxGeometry(0.56, 0.56, 0.56), lkSkinFaceMat);
            headFace.position.set(0, 1.8, 0);

            const helmetTop = new THREE.Mesh(new THREE.BoxGeometry(0.62, 0.28, 0.62), lkHeadMat);
            helmetTop.position.set(0, 1.98, -0.01);
            const helmetRim = new THREE.Mesh(new THREE.BoxGeometry(0.64, 0.08, 0.65), lkVestMat);
            helmetRim.position.set(0, 1.88, 0.02);

            const goggleFrame = new THREE.Mesh(new THREE.BoxGeometry(0.58, 0.2, 0.1), lkDarkGearMat);
            goggleFrame.position.set(0, 1.81, 0.27);
            const visor = new THREE.Mesh(
                new THREE.BoxGeometry(0.48, 0.13, 0.04),
                new THREE.MeshBasicMaterial({ color: 0x38bdf8 })
            );
            visor.position.set(0, 1.81, 0.31);

            // Blocky Voxel Carbine Rifle in Hands
            const rifleBody = new THREE.Mesh(new THREE.BoxGeometry(0.1, 0.14, 0.74), lkGunPrimaryMat);
            rifleBody.position.set(0.26, 1.14, 0.38);
            rifleBody.rotation.y = -0.38;
            const rifleMag = new THREE.Mesh(new THREE.BoxGeometry(0.07, 0.2, 0.11), lkGunAccentMat);
            rifleMag.position.set(0.24, 1.02, 0.42);
            rifleMag.rotation.y = -0.38;

            lockerCharGroup.add(
                pedestal, leftLeg, rightLeg, leftKnee, rightKnee, leftBoot, rightBoot,
                torso, vest, pouchL, pouchR, backpack,
                leftArm, rightArm, leftShoulder, rightShoulder,
                headFace, helmetTop, helmetRim, goggleFrame, visor, rifleBody, rifleMag
            );
            lockerCharGroup.rotation.y = lkManualYaw;
            lockerScene.add(lockerCharGroup);

            canvas.addEventListener("mousedown", (e) => { lkDragActive = true; lkLastX = e.clientX; });
            window.addEventListener("mouseup", () => { lkDragActive = false; });
            window.addEventListener("mousemove", (e) => {
                if (!lkDragActive) return;
                lkManualYaw += (e.clientX - lkLastX) * 0.015;
                lkLastX = e.clientX;
            });
            canvas.addEventListener("touchstart", (e) => {
                lkDragActive = true;
                lkLastX = e.changedTouches[0].clientX;
            }, { passive: true });
            canvas.addEventListener("touchend", () => { lkDragActive = false; }, { passive: true });
            canvas.addEventListener("touchmove", (e) => {
                if (!lkDragActive) return;
                const x = e.changedTouches[0].clientX;
                lkManualYaw += (x - lkLastX) * 0.015;
                lkLastX = x;
            }, { passive: true });

            updateLockerPreviewMaterials();
        }

        function resizeLockerPreview() {
            const canvas = document.getElementById("locker-preview-canvas");
            if (!canvas || !lockerRenderer || !lockerCamera) return;
            const rect = canvas.getBoundingClientRect();
            if (rect.width > 0 && rect.height > 0) {
                lockerRenderer.setSize(rect.width, rect.height, false);
                lockerCamera.aspect = rect.width / rect.height;
                lockerCamera.updateProjectionMatrix();
            }
        }

        function updateLockerPreviewMaterials() {
            const cfg = PLAYER_SKINS[selectedSkin] || PLAYER_SKINS.ranger;
            if (lkBodyMat) lkBodyMat.color.setHex(cfg.sleeve);
            if (lkVestMat) lkVestMat.color.setHex(cfg.gunAccent);
            if (lkHeadMat) lkHeadMat.color.setHex(cfg.glove);
            if (lkGunPrimaryMat) {
                lkGunPrimaryMat.color.setHex(cfg.gunPrimary);
                lkGunPrimaryMat.metalness = cfg.metalness;
            }
            if (lkGunAccentMat) lkGunAccentMat.color.setHex(cfg.gunAccent);
        }

        function syncLobbyCallsignPreview() {
            const nameInput = document.getElementById("lobby-player-name");
            const cleanName = (nameInput && nameInput.value.trim()) ? nameInput.value.trim() : "Survivor-01";
            player.name = cleanName;
            const previewEl = document.getElementById("lobby-preview-name");
            const tagEl = document.getElementById("locker-detail-tag");
            if (previewEl) previewEl.textContent = `${cleanName} (You)`;
            if (tagEl) tagEl.textContent = cleanName;
        }

        function setLobbySkin(skinKey) {
            if (!PLAYER_SKINS[skinKey]) return;
            selectedSkin = skinKey;
            const cfg = PLAYER_SKINS[skinKey];

            const colorSwatches = {
                ranger: "bg-emerald-800",
                crimson: "bg-red-600",
                arctic: "bg-sky-300",
                gold: "bg-amber-400"
            };
            const subLabels = {
                ranger: "Standard Issue",
                crimson: "Spec-Ops Red",
                arctic: "Frost White",
                gold: "Legendary Gold"
            };

            Object.keys(PLAYER_SKINS).forEach(k => {
                const btn = document.getElementById("skin-" + k);
                if (!btn) return;
                const active = (k === skinKey);
                btn.className = active
                    ? "p-3 rounded-2xl bg-lime-500 text-stone-950 font-black text-xs uppercase text-left transition flex flex-col gap-1"
                    : "p-3 rounded-2xl bg-stone-800 text-stone-300 hover:bg-stone-700 font-extrabold text-xs uppercase text-left transition flex flex-col gap-1";
                btn.innerHTML = `
                    <span class="w-5 h-5 rounded-full ${colorSwatches[k]} border-2 border-stone-950 inline-block"></span>
                    <span>${PLAYER_SKINS[k].label}</span>
                    <span class="text-[10px] ${active ? "opacity-85 font-bold" : "opacity-75 font-semibold"}">${subLabels[k]}</span>
                `;
            });

            const badge = document.getElementById("lobby-skin-badge");
            if (badge) {
                badge.textContent = cfg.label.toUpperCase();
                badge.style.backgroundColor = cfg.hudHex;
            }

            const gpSummary = document.getElementById("gameplay-skin-summary");
            const brSkin = document.getElementById("lobby-briefing-skin");
            const lkOutfit = document.getElementById("locker-detail-outfit");
            const lkWrap = document.getElementById("locker-detail-wrap");
            if (gpSummary) gpSummary.textContent = cfg.label.toUpperCase();
            if (brSkin) brSkin.textContent = cfg.label;
            if (lkOutfit) lkOutfit.textContent = `${cfg.label} Tactical`;
            if (lkWrap) lkWrap.textContent = `${cfg.label} Finish`;

            if (sleeveMesh) sleeveMesh.material.color.setHex(cfg.sleeve);
            if (gloveMesh) gloveMesh.material.color.setHex(cfg.glove);
            if (skinPrimaryMat) {
                skinPrimaryMat.color.setHex(cfg.gunPrimary);
                skinPrimaryMat.metalness = cfg.metalness;
            }
            if (skinAccentMat) {
                skinAccentMat.color.setHex(cfg.gunAccent);
            }
            updateLockerPreviewMaterials();
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
            ["zombies", "classic", "pvp"].forEach(m => {
                const btn = document.getElementById("mode-" + m);
                if (!btn) return;
                btn.className = mode === m
                    ? "py-2 px-1 rounded-xl bg-lime-500 text-stone-950 font-black text-[11px] uppercase transition"
                    : "py-2 px-1 rounded-xl bg-stone-800 text-stone-300 hover:bg-stone-700 font-extrabold text-[11px] uppercase transition";
            });

            const topBadge = document.getElementById("lobby-top-badge");
            const subEl = document.getElementById("lobby-subtitle");
            const countEl = document.getElementById("lobby-enemy-count");
            const labelEl = document.getElementById("lobby-enemy-label");

            if (mode === "zombies") {
                if (topBadge) topBadge.textContent = "OUTBREAK ZONE • 19 INFECTED";
                if (subEl) subEl.textContent = "Zombie Survival • 100 HP • 25 Headshot / 15 Body";
                if (countEl) countEl.textContent = "19 ZOMBIES DETECTED";
                if (labelEl) labelEl.textContent = "Infected Horde (x19)";
            } else if (mode === "classic") {
                if (topBadge) topBadge.textContent = "20-PLAYER SOLO • MILITARY BOTS";
                if (subEl) subEl.textContent = "Classic Battle Royale • 100 HP • Bots Fight Each Other";
                if (countEl) countEl.textContent = "19 MILITARY BOTS";
                if (labelEl) labelEl.textContent = "Rival Military Squads (x19)";
            } else {
                if (topBadge) topBadge.textContent = "ONLINE MULTIPLAYER • LIVE PVP";
                if (subEl) subEl.textContent = "Player vs Player Arena • 100 HP • Build & Battle Friends";
                if (countEl) countEl.textContent = "LIVE PLAYERS ONLY";
                if (labelEl) labelEl.textContent = "Online Opponents (Room)";
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
            if (mpActiveRoom && window.mpHostStartMatch) {
                window.mpHostStartMatch();
            }
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

            Object.values(mpRemotePlayers).forEach(rp => {
                if (!rp || !rp.data || !rp.data.alive) return;
                const mx = cx + rp.data.x * scale;
                const my = cy + rp.data.z * scale;
                ctx.fillStyle = "#38bdf8";
                ctx.beginPath();
                ctx.arc(mx, my, 3.3, 0, Math.PI * 2);
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

            if (window.updateRemotePlayersVisuals) {
                window.updateRemotePlayersVisuals(rawDt);
            }

            if (mpActiveRoom && window.mpSyncLocalState) {
                mpSyncAccumulator += rawDt;
                if (mpSyncAccumulator >= 0.07) {
                    mpSyncAccumulator = 0;
                    window.mpSyncLocalState();
                }
            }

            if (camera && renderer && scene) {
                renderer.render(scene, camera);
            }
            if (inMainLobby && lockerRenderer && lockerScene && lockerCamera && lockerCharGroup) {
                const lkTab = document.getElementById("lobby-tab-locker");
                if (lkTab && !lkTab.classList.contains("hidden")) {
                    if (!lkDragActive) lkManualYaw += rawDt * 0.85;
                    lockerCharGroup.rotation.y = lkManualYaw;
                    lockerRenderer.render(lockerScene, lockerCamera);
                }
            }
            drawMinimap();
        }

        window.onload = function () {
            initThreeScene();
            buildSelectedMap(selectedMap);
            spawnSupplyChests();
            spawnAllBots();
            setLobbySkin(selectedSkin);
            setLobbyGameMode(gameMode);
            setLobbyBuildMode(buildEnabled);
            setLobbyMap(selectedMap);
            setLobbyDifficulty(lobbyDifficulty);
            updateHUD();
            updateRosterUI();
            animateLoop();
        };
    </script>

    <!-- Real-Time Cloud Multiplayer Sync Module -->
    <script type="module">
        import { initializeApp } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-app.js";
        import { getAuth, signInAnonymously, signInWithCustomToken, onAuthStateChanged } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-auth.js";
        import { getFirestore, doc, setDoc, getDoc, addDoc, updateDoc, deleteDoc, onSnapshot, collection } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-firestore.js";

        const appId = typeof __app_id !== "undefined" ? __app_id : "default-app-id";
        let db = null;
        let auth = null;
        let currentUser = null;
        let isRoomHost = false;
        let lastKnownMatchNonce = 0;
        let mpReadyPromise = null;
        let mpChannel = null;
        let mpHeartbeatTimer = null;

        function setMpFeedback(msg, isError = false) {
            const el = document.getElementById("mp-feedback-msg");
            if (!el) return;
            if (!msg) {
                el.classList.add("hidden");
                el.textContent = "";
                return;
            }
            el.classList.remove("hidden");
            el.className = `mt-2 text-[11px] font-bold ${isError ? "text-red-400" : "text-sky-300"}`;
            el.textContent = msg;
        }

        function ensureLocalIdentity() {
            if (currentUser && currentUser.uid) return currentUser;
            const fallbackId = "player_" + Math.random().toString(36).substring(2, 10).toUpperCase();
            currentUser = { uid: fallbackId, isOfflineFallback: true };
            const uidEl = document.getElementById("mp-user-id");
            const dotEl = document.getElementById("mp-status-dot");
            const modeEl = document.getElementById("mp-conn-mode");
            if (uidEl) uidEl.textContent = fallbackId;
            if (dotEl) dotEl.className = "w-2 h-2 rounded-full bg-sky-400";
            if (modeEl) {
                modeEl.textContent = "LOCAL TAB ONLY";
                modeEl.className = "mono text-[9px] font-black px-2 py-0.5 rounded bg-sky-500/20 text-sky-300 border border-sky-400/30";
            }
            return currentUser;
        }

        function spawnRemoteBuildMesh(b) {
            const plankMat = new THREE.MeshStandardMaterial({ color: 0xa16207, roughness: 0.78 });
            const group = new THREE.Group();
            group.rotation.y = b.rotY || 0;

            if (b.type === "wall") {
                const wall = new THREE.Mesh(new THREE.BoxGeometry(BUILD_GRID_SIZE, BUILD_WALL_HEIGHT, 0.26), plankMat);
                group.add(wall);
                group.position.set(b.x, b.baseY + BUILD_WALL_HEIGHT / 2, b.z);
                const isZAligned = Math.abs(Math.sin(b.rotY || 0)) > 0.5;
                addBoxCollider(b.x, b.z, isZAligned ? 0.35 : BUILD_GRID_SIZE / 2, isZAligned ? BUILD_GRID_SIZE / 2 : 0.35, b.baseY, b.baseY + BUILD_WALL_HEIGHT);
                builtStructures.push({ type: "wall", x: b.x, z: b.z, baseY: b.baseY, y: b.baseY + BUILD_WALL_HEIGHT, rotY: b.rotY || 0, mesh: group });
            } else if (b.type === "floor") {
                const floor = new THREE.Mesh(new THREE.BoxGeometry(BUILD_GRID_SIZE, 0.26, BUILD_GRID_SIZE), plankMat);
                group.add(floor);
                group.position.set(b.x, b.baseY + 0.13, b.z);
                builtStructures.push({ type: "floor", x: b.x, z: b.z, baseY: b.baseY, y: b.baseY + 0.26, rotY: b.rotY || 0, mesh: group });
            } else if (b.type === "ramp") {
                const slopeLen = Math.hypot(BUILD_GRID_SIZE, BUILD_WALL_HEIGHT);
                const ramp = new THREE.Mesh(new THREE.BoxGeometry(BUILD_GRID_SIZE, 0.26, slopeLen), plankMat);
                ramp.rotation.x = Math.atan2(BUILD_WALL_HEIGHT, BUILD_GRID_SIZE);
                group.add(ramp);
                group.position.set(b.x, b.baseY + BUILD_WALL_HEIGHT / 2, b.z);
                builtStructures.push({ type: "ramp", x: b.x, z: b.z, baseY: b.baseY, y: b.baseY + BUILD_WALL_HEIGHT, rotY: b.rotY || 0, mesh: group });
            }
            scene.add(group);
        }

        function initBroadcastBridge() {
            if (typeof BroadcastChannel === "undefined") return;
            mpChannel = new BroadcastChannel("deadshot_mp_bridge_" + appId);
            mpChannel.onmessage = (e) => {
                const msg = e.data;
                if (!msg || !mpActiveRoom || msg.roomCode !== mpActiveRoom) return;
                if (!currentUser || msg.senderUid === currentUser.uid) return;

                if (msg.type === "player_state" && msg.player) {
                    upsertRemotePlayer(msg.player);
                } else if (msg.type === "build_piece" && msg.piece) {
                    const key = `${msg.piece.type}_${msg.piece.x}_${msg.piece.baseY}_${msg.piece.z}`;
                    if (!mpRenderedBuilds.has(key)) {
                        mpRenderedBuilds.add(key);
                        spawnRemoteBuildMesh(msg.piece);
                    }
                } else if (msg.type === "room_start" && !isRoomHost) {
                    if (msg.map) setLobbyMap(msg.map);
                    if (msg.gameMode) setLobbyGameMode(msg.gameMode);
                    if (typeof msg.buildEnabled === "boolean") setLobbyBuildMode(msg.buildEnabled);
                    if (msg.difficulty) setLobbyDifficulty(msg.difficulty);
                    if (inMainLobby) {
                        const nameInput = document.getElementById("lobby-player-name");
                        player.name = (nameInput && nameInput.value.trim()) ? nameInput.value.trim() : "Survivor-01";
                        inMainLobby = false;
                        const lobbyEl = document.getElementById("main-lobby-overlay");
                        if (lobbyEl) {
                            lobbyEl.classList.add("hidden");
                            lobbyEl.classList.remove("flex");
                        }
                        resetSoloMatch();
                        engageLookControl();
                    }
                } else if (msg.type === "hit" && msg.targetUid === currentUser.uid && player.alive) {
                    applyDamageToPlayer(msg.damage || 15, msg.attackerName || "Online Player");
                    if (!player.alive && window.mpSendKillConfirm) {
                        window.mpSendKillConfirm(msg.attackerUid, msg.attackerName);
                    }
                } else if (msg.type === "elim") {
                    if (msg.attackerUid === currentUser.uid) {
                        player.kills += 1;
                        updateHUD();
                    }
                    addKillFeedEntry(`${msg.attackerName} eliminated ${msg.victimName}`, "text-sky-300");
                    checkMatchVictory();
                } else if (msg.type === "player_leave") {
                    if (mpRemotePlayers[msg.senderUid]) {
                        scene.remove(mpRemotePlayers[msg.senderUid].mesh.root);
                        delete mpRemotePlayers[msg.senderUid];
                        refreshPartyUI();
                    }
                }
            };
        }

        function initMultiplayerService() {
            initBroadcastBridge();
            mpReadyPromise = (async () => {
                try {
                    if (typeof __firebase_config === "undefined" || !__firebase_config) {
                        ensureLocalIdentity();
                        return;
                    }
                    const firebaseConfig = JSON.parse(__firebase_config);
                    const app = initializeApp(firebaseConfig);
                    auth = getAuth(app);
                    db = getFirestore(app);

                    if (typeof __initial_auth_token !== "undefined" && __initial_auth_token) {
                        await signInWithCustomToken(auth, __initial_auth_token);
                    } else {
                        await signInAnonymously(auth);
                    }

                    if (auth.currentUser) {
                        currentUser = auth.currentUser;
                        const uidEl = document.getElementById("mp-user-id");
                        const dotEl = document.getElementById("mp-status-dot");
                        const modeEl = document.getElementById("mp-conn-mode");
                        if (uidEl) uidEl.textContent = currentUser.uid;
                        if (dotEl) dotEl.className = "w-2 h-2 rounded-full bg-lime-400";
                        if (modeEl) {
                            modeEl.textContent = "CLOUD ONLINE";
                            modeEl.className = "mono text-[9px] font-black px-2 py-0.5 rounded bg-lime-500/20 text-lime-300 border border-lime-400/30";
                        }
                    } else {
                        ensureLocalIdentity();
                    }

                    onAuthStateChanged(auth, (user) => {
                        if (user) {
                            currentUser = user;
                            const uidEl = document.getElementById("mp-user-id");
                            const dotEl = document.getElementById("mp-status-dot");
                            const modeEl = document.getElementById("mp-conn-mode");
                            if (uidEl) uidEl.textContent = user.uid;
                            if (dotEl) dotEl.className = "w-2 h-2 rounded-full bg-lime-400";
                            if (modeEl) {
                                modeEl.textContent = "CLOUD ONLINE";
                                modeEl.className = "mono text-[9px] font-black px-2 py-0.5 rounded bg-lime-500/20 text-lime-300 border border-lime-400/30";
                            }
                        }
                    });
                } catch (err) {
                    console.warn("Cloud auth fallback:", err);
                    ensureLocalIdentity();
                }
            })();
            return mpReadyPromise;
        }

        function createRemoteCharacterMesh(skinKey, labelText) {
            const cfg = PLAYER_SKINS[skinKey] || PLAYER_SKINS.ranger;
            const root = new THREE.Group();

            const bodyMat = new THREE.MeshStandardMaterial({ color: cfg.sleeve, roughness: 0.7 });
            const vestMat = new THREE.MeshStandardMaterial({ color: cfg.gunAccent, roughness: 0.55 });
            const headMat = new THREE.MeshStandardMaterial({ color: cfg.glove, roughness: 0.45 });
            const skinFaceMat = new THREE.MeshStandardMaterial({ color: 0xd6a57c, roughness: 0.65 });
            const darkGearMat = new THREE.MeshStandardMaterial({ color: 0x18181b, roughness: 0.5 });
            const visorMat = new THREE.MeshBasicMaterial({ color: 0x38bdf8 });

            // Blockpost 2.0 Voxel Soldier Proportions
            const leftLeg = new THREE.Mesh(new THREE.BoxGeometry(0.28, 0.58, 0.28), headMat);
            leftLeg.position.set(-0.17, 0.47, 0);
            const rightLeg = new THREE.Mesh(new THREE.BoxGeometry(0.28, 0.58, 0.28), headMat);
            rightLeg.position.set(0.17, 0.47, 0);

            const leftBoot = new THREE.Mesh(new THREE.BoxGeometry(0.3, 0.2, 0.35), darkGearMat);
            leftBoot.position.set(-0.17, 0.1, 0.03);
            const rightBoot = new THREE.Mesh(new THREE.BoxGeometry(0.3, 0.2, 0.35), darkGearMat);
            rightBoot.position.set(0.17, 0.1, 0.03);

            const torso = new THREE.Mesh(new THREE.BoxGeometry(0.68, 0.72, 0.38), bodyMat);
            torso.position.set(0, 1.14, 0);
            const vest = new THREE.Mesh(new THREE.BoxGeometry(0.72, 0.56, 0.44), vestMat);
            vest.position.set(0, 1.16, 0);

            const pouchL = new THREE.Mesh(new THREE.BoxGeometry(0.16, 0.22, 0.1), headMat);
            pouchL.position.set(-0.16, 1.08, 0.24);
            const pouchR = new THREE.Mesh(new THREE.BoxGeometry(0.16, 0.22, 0.1), headMat);
            pouchR.position.set(0.16, 1.08, 0.24);
            const backpack = new THREE.Mesh(new THREE.BoxGeometry(0.5, 0.54, 0.22), headMat);
            backpack.position.set(0, 1.18, -0.28);

            const leftArm = new THREE.Mesh(new THREE.BoxGeometry(0.24, 0.64, 0.24), bodyMat);
            leftArm.position.set(-0.48, 1.16, 0.04);
            const rightArm = new THREE.Mesh(new THREE.BoxGeometry(0.24, 0.64, 0.24), bodyMat);
            rightArm.position.set(0.48, 1.16, 0.12);
            rightArm.rotation.x = -0.45;

            const leftShoulder = new THREE.Mesh(new THREE.BoxGeometry(0.28, 0.22, 0.28), vestMat);
            leftShoulder.position.set(-0.49, 1.38, 0.02);
            const rightShoulder = new THREE.Mesh(new THREE.BoxGeometry(0.28, 0.22, 0.28), vestMat);
            rightShoulder.position.set(0.49, 1.38, 0.02);

            const head = new THREE.Mesh(new THREE.BoxGeometry(0.56, 0.56, 0.56), skinFaceMat);
            head.position.set(0, 1.8, 0);
            const helmetTop = new THREE.Mesh(new THREE.BoxGeometry(0.62, 0.28, 0.62), headMat);
            helmetTop.position.set(0, 1.98, -0.01);
            const goggleFrame = new THREE.Mesh(new THREE.BoxGeometry(0.58, 0.2, 0.1), darkGearMat);
            goggleFrame.position.set(0, 1.81, 0.27);
            const visor = new THREE.Mesh(new THREE.BoxGeometry(0.48, 0.13, 0.04), visorMat);
            visor.position.set(0, 1.81, 0.31);

            // Floating Nameplate Sprite
            const canvas = document.createElement("canvas");
            canvas.width = 256;
            canvas.height = 64;
            const ctx = canvas.getContext("2d");
            ctx.fillStyle = "rgba(15, 23, 42, 0.82)";
            ctx.fillRect(16, 10, 224, 44);
            ctx.strokeStyle = "#38bdf8";
            ctx.lineWidth = 3;
            ctx.strokeRect(16, 10, 224, 44);
            ctx.fillStyle = "#f8fafc";
            ctx.font = "bold 24px sans-serif";
            ctx.textAlign = "center";
            ctx.textBaseline = "middle";
            ctx.fillText(labelText || "Player", 128, 32);

            const tex = new THREE.CanvasTexture(canvas);
            const sprite = new THREE.Sprite(new THREE.SpriteMaterial({ map: tex, transparent: true }));
            sprite.position.set(0, 2.5, 0);
            sprite.scale.set(2.2, 0.55, 1);

            const beacon = new THREE.Mesh(
                new THREE.CylinderGeometry(0.08, 0.25, 14, 8, 1, true),
                new THREE.MeshBasicMaterial({ color: 0x38bdf8, transparent: true, opacity: 0.25, side: THREE.DoubleSide })
            );
            beacon.position.set(0, 7.2, 0);

            root.add(
                leftLeg, rightLeg, leftBoot, rightBoot,
                torso, vest, pouchL, pouchR, backpack,
                leftArm, rightArm, leftShoulder, rightShoulder,
                head, helmetTop, goggleFrame, visor, sprite, beacon
            );
            scene.add(root);
            return { root, head, torso, leftLeg, rightLeg, skinKey };
        }

        function clearRemotePlayers() {
            Object.values(mpRemotePlayers).forEach(rp => {
                if (rp && rp.mesh && rp.mesh.root) scene.remove(rp.mesh.root);
            });
            mpRemotePlayers = {};
            mpRenderedBuilds.clear();
            mpProcessedEvents.clear();
        }

        function upsertRemotePlayer(p) {
            if (!p || !p.uid || !currentUser || p.uid === currentUser.uid) return;
            const isNewPeer = !mpRemotePlayers[p.uid];
            if (isNewPeer || mpRemotePlayers[p.uid].mesh.skinKey !== p.skin) {
                if (mpRemotePlayers[p.uid]) scene.remove(mpRemotePlayers[p.uid].mesh.root);
                mpRemotePlayers[p.uid] = {
                    data: p,
                    targetPos: new THREE.Vector3(p.x || 0, p.y || 0, p.z || 0),
                    mesh: createRemoteCharacterMesh(p.skin || "ranger", p.name || "Player")
                };
                mpRemotePlayers[p.uid].mesh.root.position.set(p.x || 0, p.y || 0, p.z || 0);
                if (isNewPeer) {
                    setMpFeedback(`${p.name || "Player"} joined your party!`, false);
                    addKillFeedEntry(`${p.name || "Player"} joined the party!`, "text-sky-300");
                }
            } else {
                mpRemotePlayers[p.uid].data = p;
                mpRemotePlayers[p.uid].targetPos.set(p.x || 0, p.y || 0, p.z || 0);
            }
            mpRemotePlayers[p.uid].mesh.root.visible = Boolean(p.alive);
            refreshPartyUI();
        }

        function refreshPartyUI() {
            const listForUi = [];
            if (currentUser) {
                listForUi.push({
                    uid: currentUser.uid,
                    name: player.name,
                    skin: selectedSkin,
                    health: player.health,
                    inMatch: !inMainLobby
                });
            }
            Object.values(mpRemotePlayers).forEach(rp => {
                if (rp && rp.data) listForUi.push(rp.data);
            });
            renderMpPlayerChips(listForUi);
            updateHUD();
        }

        function renderMpPlayerChips(playersArr) {
            const header = document.getElementById("mp-party-header");
            const countEl = document.getElementById("mp-party-count");
            const wrap = document.getElementById("mp-players-list");
            const liveHud = document.getElementById("hud-party-live");
            if (!wrap) return;
            if (!mpActiveRoom) {
                if (header) {
                    header.classList.add("hidden");
                    header.classList.remove("flex");
                }
                wrap.classList.add("hidden");
                wrap.classList.remove("flex");
                if (liveHud) liveHud.classList.add("hidden");
                return;
            }
            if (header) {
                header.classList.remove("hidden");
                header.classList.add("flex");
            }
            if (countEl) {
                const n = playersArr.length;
                countEl.textContent = `${n} ${n === 1 ? "PLAYER" : "PLAYERS"} IN PARTY`;
            }
            wrap.classList.remove("hidden");
            wrap.classList.add("flex");
            wrap.innerHTML = playersArr.map(p => {
                const isMe = currentUser && p.uid === currentUser.uid;
                const statusTag = p.inMatch ? "IN MATCH" : "IN LOBBY";
                const skinLabel = (PLAYER_SKINS[p.skin] && PLAYER_SKINS[p.skin].label) ? PLAYER_SKINS[p.skin].label : "Ranger";
                return `<span class="px-2.5 py-1 rounded-xl font-bold flex items-center gap-1.5 ${isMe ? "bg-lime-500/20 text-lime-300 border border-lime-400/40" : "bg-sky-500/20 text-sky-300 border border-sky-400/40"}">
                    <span class="w-2 h-2 rounded-full ${p.inMatch ? "bg-amber-400" : "bg-lime-400"}"></span>
                    <span>${p.name}${isMe ? " (You)" : ""}</span>
                    <span class="text-[9px] uppercase opacity-75">• ${skinLabel} • ${statusTag}</span>
                </span>`;
            }).join("");

            if (liveHud) {
                liveHud.classList.remove("hidden");
                const names = playersArr.map(p => `${p.name}${currentUser && p.uid === currentUser.uid ? " (You)" : ""}`).join(" • ");
                liveHud.textContent = `PARTY (${playersArr.length}): ${names}`;
            }
        }

        function subscribeToRoom(roomCode) {
            if (!db || !currentUser) return;
            if (mpUnsubRoom) mpUnsubRoom();
            if (mpUnsubPlayers) mpUnsubPlayers();
            if (mpUnsubEvents) mpUnsubEvents();
            if (mpUnsubBuilds) mpUnsubBuilds();

            const roomDocRef = doc(db, "artifacts", appId, "public", "data", "deadshot_rooms", roomCode);
            const playersColRef = collection(db, "artifacts", appId, "public", "data", `deadshot_players_${roomCode}`);
            const eventsColRef = collection(db, "artifacts", appId, "public", "data", `deadshot_events_${roomCode}`);
            const buildsColRef = collection(db, "artifacts", appId, "public", "data", `deadshot_builds_${roomCode}`);

            mpUnsubRoom = onSnapshot(roomDocRef, (snap) => {
                if (!snap.exists()) return;
                const data = snap.data();
                if (!isRoomHost) {
                    if (data.map && data.map !== selectedMap) setLobbyMap(data.map);
                    if (data.gameMode && data.gameMode !== gameMode) setLobbyGameMode(data.gameMode);
                    if (typeof data.buildEnabled === "boolean" && data.buildEnabled !== buildEnabled) setLobbyBuildMode(data.buildEnabled);
                    if (data.difficulty && data.difficulty !== lobbyDifficulty) setLobbyDifficulty(data.difficulty);

                    if (data.matchNonce && data.matchNonce > lastKnownMatchNonce) {
                        lastKnownMatchNonce = data.matchNonce;
                        if (inMainLobby) {
                            const nameInput = document.getElementById("lobby-player-name");
                            player.name = (nameInput && nameInput.value.trim()) ? nameInput.value.trim() : "Survivor-01";
                            inMainLobby = false;
                            const lobbyEl = document.getElementById("main-lobby-overlay");
                            if (lobbyEl) {
                                lobbyEl.classList.add("hidden");
                                lobbyEl.classList.remove("flex");
                            }
                            resetSoloMatch();
                            engageLookControl();
                        }
                    }
                }
            }, (err) => console.warn("Room snapshot error:", err));

            mpUnsubPlayers = onSnapshot(playersColRef, (snap) => {
                if (!currentUser) return;
                const activeUids = new Set();

                snap.forEach((docSnap) => {
                    const p = docSnap.data();
                    if (!p || !p.uid) return;
                    activeUids.add(p.uid);
                    if (p.uid === currentUser.uid) return;
                    upsertRemotePlayer(p);
                });

                Object.keys(mpRemotePlayers).forEach((uid) => {
                    if (!activeUids.has(uid)) {
                        scene.remove(mpRemotePlayers[uid].mesh.root);
                        delete mpRemotePlayers[uid];
                    }
                });

                refreshPartyUI();
            }, (err) => console.warn("Players snapshot error:", err));

            mpUnsubEvents = onSnapshot(eventsColRef, (snap) => {
                if (!currentUser) return;
                snap.forEach((docSnap) => {
                    if (mpProcessedEvents.has(docSnap.id)) return;
                    mpProcessedEvents.add(docSnap.id);
                    const ev = docSnap.data();
                    if (!ev || ev.createdAt < Date.now() - 12000) return;

                    if (ev.targetUid === currentUser.uid && ev.type === "hit" && player.alive) {
                        applyDamageToPlayer(ev.damage || 15, ev.attackerName || "Online Player");
                        if (!player.alive && window.mpSendKillConfirm) {
                            window.mpSendKillConfirm(ev.attackerUid, ev.attackerName);
                        }
                    } else if (ev.type === "elim") {
                        if (ev.attackerUid === currentUser.uid) {
                            player.kills += 1;
                            updateHUD();
                        }
                        addKillFeedEntry(`${ev.attackerName} eliminated ${ev.victimName}`, "text-sky-300");
                        checkMatchVictory();
                    }
                });
            }, (err) => console.warn("Events snapshot error:", err));

            mpUnsubBuilds = onSnapshot(buildsColRef, (snap) => {
                if (!currentUser) return;
                snap.forEach((docSnap) => {
                    if (mpRenderedBuilds.has(docSnap.id)) return;
                    mpRenderedBuilds.add(docSnap.id);
                    const b = docSnap.data();
                    if (!b || b.ownerUid === currentUser.uid) return;
                    spawnRemoteBuildMesh(b);
                });
            }, (err) => console.warn("Builds snapshot error:", err));
        }

        window.createMultiplayerRoom = async function () {
            setMpFeedback("Creating room...", false);
            if (mpReadyPromise) await mpReadyPromise;
            const user = ensureLocalIdentity();

            const code = Math.random().toString(36).substring(2, 6).toUpperCase();
            isRoomHost = true;
            mpActiveRoom = code;
            clearRemotePlayers();

            const nameInput = document.getElementById("lobby-player-name");
            player.name = (nameInput && nameInput.value.trim()) ? nameInput.value.trim() : "Survivor-01";

            if (db && !user.isOfflineFallback) {
                try {
                    const roomDocRef = doc(db, "artifacts", appId, "public", "data", "deadshot_rooms", code);
                    await setDoc(roomDocRef, {
                        code,
                        hostUid: user.uid,
                        map: selectedMap,
                        gameMode,
                        buildEnabled,
                        difficulty: lobbyDifficulty,
                        matchNonce: 0,
                        updatedAt: Date.now()
                    });
                    subscribeToRoom(code);
                } catch (err) {
                    console.warn("Cloud room create fallback:", err);
                }
            }

            if (mpHeartbeatTimer) clearInterval(mpHeartbeatTimer);
            mpHeartbeatTimer = setInterval(() => {
                if (mpActiveRoom && window.mpSyncLocalState) window.mpSyncLocalState();
            }, 600);

            document.getElementById("mp-room-badge").textContent = `HOSTING ROOM: ${code}`;
            document.getElementById("mp-room-badge").className = "mono text-[10px] font-black px-2 py-0.5 rounded bg-sky-400 text-stone-950";
            document.getElementById("mp-room-input").value = code;
            document.getElementById("mp-leave-btn").classList.remove("hidden");
            setMpFeedback(`Room ${code} active! Share code "${code}" with your friend to join.`, false);

            refreshPartyUI();
            await window.mpSyncLocalState();
        };

        window.joinMultiplayerRoom = async function () {
            const raw = document.getElementById("mp-room-input").value.trim().toUpperCase();
            if (!raw) {
                setMpFeedback("Enter a 4-letter room code first.", true);
                return;
            }

            setMpFeedback(`Joining room ${raw}...`, false);
            if (mpReadyPromise) await mpReadyPromise;
            const user = ensureLocalIdentity();

            let roomData = null;
            if (db && !user.isOfflineFallback) {
                try {
                    const roomDocRef = doc(db, "artifacts", appId, "public", "data", "deadshot_rooms", raw);
                    const snap = await getDoc(roomDocRef);
                    if (snap.exists()) {
                        roomData = snap.data();
                    } else if (!mpChannel) {
                        setMpFeedback(`Room "${raw}" not found.`, true);
                        return;
                    }
                } catch (err) {
                    console.warn("Cloud room lookup fallback:", err);
                }
            }

            isRoomHost = false;
            mpActiveRoom = raw;
            clearRemotePlayers();

            if (roomData) {
                if (roomData.map) setLobbyMap(roomData.map);
                if (roomData.gameMode) setLobbyGameMode(roomData.gameMode);
                if (typeof roomData.buildEnabled === "boolean") setLobbyBuildMode(roomData.buildEnabled);
                if (roomData.difficulty) setLobbyDifficulty(roomData.difficulty);
            }

            const nameInput = document.getElementById("lobby-player-name");
            player.name = (nameInput && nameInput.value.trim()) ? nameInput.value.trim() : "Survivor-01";

            document.getElementById("mp-room-badge").textContent = `JOINED ROOM: ${raw}`;
            document.getElementById("mp-room-badge").className = "mono text-[10px] font-black px-2 py-0.5 rounded bg-emerald-400 text-stone-950";
            document.getElementById("mp-leave-btn").classList.remove("hidden");
            setMpFeedback(`Connected to party room ${raw}!`, false);

            if (db && !user.isOfflineFallback) {
                subscribeToRoom(raw);
            }

            if (mpHeartbeatTimer) clearInterval(mpHeartbeatTimer);
            mpHeartbeatTimer = setInterval(() => {
                if (mpActiveRoom && window.mpSyncLocalState) window.mpSyncLocalState();
            }, 600);

            refreshPartyUI();
            await window.mpSyncLocalState();

            if (roomData && roomData.matchNonce && roomData.matchNonce > 0 && inMainLobby) {
                lastKnownMatchNonce = roomData.matchNonce;
                inMainLobby = false;
                const lobbyEl = document.getElementById("main-lobby-overlay");
                if (lobbyEl) {
                    lobbyEl.classList.add("hidden");
                    lobbyEl.classList.remove("flex");
                }
                resetSoloMatch();
                engageLookControl();
            }
        };

        window.leaveMultiplayerRoom = async function () {
            if (mpHeartbeatTimer) clearInterval(mpHeartbeatTimer);
            if (mpUnsubRoom) mpUnsubRoom();
            if (mpUnsubPlayers) mpUnsubPlayers();
            if (mpUnsubEvents) mpUnsubEvents();
            if (mpUnsubBuilds) mpUnsubBuilds();

            if (mpChannel && mpActiveRoom && currentUser) {
                mpChannel.postMessage({ type: "player_leave", roomCode: mpActiveRoom, senderUid: currentUser.uid });
            }

            if (db && currentUser && !currentUser.isOfflineFallback && mpActiveRoom) {
                try {
                    const myDoc = doc(db, "artifacts", appId, "public", "data", `deadshot_players_${mpActiveRoom}`, currentUser.uid);
                    await deleteDoc(myDoc);
                } catch (e) {}
            }

            mpActiveRoom = null;
            isRoomHost = false;
            clearRemotePlayers();
            renderMpPlayerChips([]);
            setMpFeedback("", false);

            document.getElementById("mp-room-badge").textContent = "SOLO / OFFLINE";
            document.getElementById("mp-room-badge").className = "mono text-[10px] font-black px-2 py-0.5 rounded bg-stone-800 text-stone-300";
            document.getElementById("mp-leave-btn").classList.add("hidden");
            updateHUD();
        };

        window.mpHostStartMatch = async function () {
            if (!currentUser || !mpActiveRoom || !isRoomHost) return;
            if (mpChannel) {
                mpChannel.postMessage({
                    type: "room_start",
                    roomCode: mpActiveRoom,
                    senderUid: currentUser.uid,
                    map: selectedMap,
                    gameMode,
                    buildEnabled,
                    difficulty: lobbyDifficulty
                });
            }
            if (!db || currentUser.isOfflineFallback) return;
            try {
                const roomDocRef = doc(db, "artifacts", appId, "public", "data", "deadshot_rooms", mpActiveRoom);
                await updateDoc(roomDocRef, {
                    map: selectedMap,
                    gameMode,
                    buildEnabled,
                    difficulty: lobbyDifficulty,
                    matchNonce: Date.now(),
                    updatedAt: Date.now()
                });
            } catch (e) {}
        };

        window.mpGetSpawnCoordinates = function () {
            const x = isRoomHost ? -5.5 : 5.5;
            const z = 0;
            const yaw = isRoomHost ? -Math.PI / 2 : Math.PI / 2;
            return { x, z, yaw };
        };

        window.mpSyncLocalState = async function () {
            if (!currentUser || !mpActiveRoom) return;
            const payload = {
                uid: currentUser.uid,
                name: player.name,
                skin: selectedSkin,
                x: Number(player.pos.x.toFixed(2)),
                y: Number(player.pos.y.toFixed(2)),
                z: Number(player.pos.z.toFixed(2)),
                yaw: Number(player.yaw.toFixed(3)),
                health: player.health,
                alive: player.alive,
                inMatch: !inMainLobby,
                kills: player.kills,
                updatedAt: Date.now()
            };

            if (mpChannel) {
                mpChannel.postMessage({
                    type: "player_state",
                    roomCode: mpActiveRoom,
                    senderUid: currentUser.uid,
                    player: payload
                });
            }

            if (!db || currentUser.isOfflineFallback) return;
            try {
                const myDoc = doc(db, "artifacts", appId, "public", "data", `deadshot_players_${mpActiveRoom}`, currentUser.uid);
                await setDoc(myDoc, payload);
            } catch (e) {}
        };

        window.mpSendHitEvent = async function (targetUid, targetName, damage, isHead) {
            if (!currentUser || !mpActiveRoom) return;
            if (mpChannel) {
                mpChannel.postMessage({
                    type: "hit",
                    roomCode: mpActiveRoom,
                    senderUid: currentUser.uid,
                    attackerUid: currentUser.uid,
                    attackerName: player.name,
                    targetUid,
                    targetName,
                    damage,
                    isHead
                });
            }
            if (!db || currentUser.isOfflineFallback) return;
            try {
                const eventsCol = collection(db, "artifacts", appId, "public", "data", `deadshot_events_${mpActiveRoom}`);
                await addDoc(eventsCol, {
                    type: "hit",
                    attackerUid: currentUser.uid,
                    attackerName: player.name,
                    targetUid,
                    targetName,
                    damage,
                    isHead,
                    createdAt: Date.now()
                });
            } catch (e) {}
        };

        window.mpSendKillConfirm = async function (attackerUid, attackerName) {
            if (!currentUser || !mpActiveRoom) return;
            if (mpChannel) {
                mpChannel.postMessage({
                    type: "elim",
                    roomCode: mpActiveRoom,
                    senderUid: currentUser.uid,
                    attackerUid,
                    attackerName: attackerName || "Online Player",
                    victimUid: currentUser.uid,
                    victimName: player.name
                });
            }
            if (!db || currentUser.isOfflineFallback) return;
            try {
                const eventsCol = collection(db, "artifacts", appId, "public", "data", `deadshot_events_${mpActiveRoom}`);
                await addDoc(eventsCol, {
                    type: "elim",
                    attackerUid,
                    attackerName: attackerName || "Online Player",
                    victimUid: currentUser.uid,
                    victimName: player.name,
                    createdAt: Date.now()
                });
            } catch (e) {}
        };

        window.mpPublishBuild = async function (pieceData) {
            if (!currentUser || !mpActiveRoom) return;
            if (mpChannel) {
                mpChannel.postMessage({
                    type: "build_piece",
                    roomCode: mpActiveRoom,
                    senderUid: currentUser.uid,
                    piece: pieceData
                });
            }
            if (!db || currentUser.isOfflineFallback) return;
            try {
                const buildsCol = collection(db, "artifacts", appId, "public", "data", `deadshot_builds_${mpActiveRoom}`);
                await addDoc(buildsCol, {
                    ownerUid: currentUser.uid,
                    ...pieceData,
                    createdAt: Date.now()
                });
            } catch (e) {}
        };

        window.updateRemotePlayersVisuals = function (dt) {
            Object.values(mpRemotePlayers).forEach((rp) => {
                if (!rp || !rp.mesh || !rp.data) return;
                if (rp.mesh.root.position.distanceTo(rp.targetPos) > 25) {
                    rp.mesh.root.position.copy(rp.targetPos);
                } else {
                    rp.mesh.root.position.lerp(rp.targetPos, Math.min(1, dt * 16));
                }
                rp.mesh.root.rotation.y = (rp.data.yaw || 0) + Math.PI;
            });
        };

        initMultiplayerService();
    </script>
</body>
</html>
