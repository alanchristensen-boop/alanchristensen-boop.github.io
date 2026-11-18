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
                <button id="bpiece-rotate" onclick="rotateBuildPiece()" class="ml-1 px-2.5 py-0.5 rounded bg-sky-500/25 hover:bg-sky-500/40 text-sky-300 border border-sky-400/50 transition">[R] Rotate ↻</button>
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
        const BUILD_MAX_HEALTH = 150;

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
            const col = { cx, cz, halfW, halfD, minY, maxY };
            colliders.push(col);
            return col;
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
            const footPad = 0.75;
            for (let i = 0; i < builtStructures.length; i++) {
                const s = builtStructures[i];
                if (s.destroyed || s.health <= 0) continue;
                if (s.type === "floor") {
                    if (Math.abs(x - s.x) <= BUILD_GRID_SIZE * 0.5 + footPad && Math.abs(z - s.z) <= BUILD_GRID_SIZE * 0.5 + footPad) {
                        if (currentY >= s.y - 1.1 && s.y > supportY) {
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
                    if (Math.abs(localX) <= BUILD_GRID_SIZE * 0.5 + footPad && Math.abs(localZ) <= BUILD_GRID_SIZE * 0.5 + footPad) {
                        const t = THREE.MathUtils.clamp((BUILD_GRID_SIZE * 0.5 - localZ) / BUILD_GRID_SIZE, 0, 1);
                        const rampH = s.baseY + t * BUILD_WALL_HEIGHT;
                        if (currentY >= rampH - 2.2 && rampH > supportY) {
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
            buildRotationOffset: 0,
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

        function rotateBuildPiece() {
            if (!buildEnabled) return;
            player.buildRotationOffset = (player.buildRotationOffset + 1) % 4;
            playSound("build");
            if (!player.buildMode) {
                toggleBuildMode(true);
            } else {
                updateBuildPreviewPosition();
                updateHUD();
            }
        }

        function computeBuildSnapTransform() {
            const forward = new THREE.Vector3(-Math.sin(player.yaw), 0, -Math.cos(player.yaw));
            const target = player.pos.clone().addScaledVector(forward, 3.6);
            const gx = Math.round(target.x / BUILD_GRID_SIZE) * BUILD_GRID_SIZE;
            const gz = Math.round(target.z / BUILD_GRID_SIZE) * BUILD_GRID_SIZE;
            const gy = Math.floor((player.pos.y + 0.5) / BUILD_WALL_HEIGHT) * BUILD_WALL_HEIGHT;
            const cardRot = (Math.round(player.yaw / (Math.PI / 2)) + player.buildRotationOffset) * (Math.PI / 2);
            return { gx, gy, gz, cardRot };
        }

        function getWallEdgeOffset(rotY) {
            const half = BUILD_GRID_SIZE * 0.5;
            return {
                ox: -Math.sin(rotY) * half,
                oz: -Math.cos(rotY) * half
            };
        }

        function isBuildPlacementSupported(gx, gy, gz, pieceType = player.buildPiece, rotY = 0) {
            if (gy <= 0.25) return true;
            const edge = pieceType === "wall" ? getWallEdgeOffset(rotY) : { ox: 0, oz: 0 };
            const candidate = {
                type: pieceType,
                gridX: gx,
                gridZ: gz,
                x: gx + edge.ox,
                z: gz + edge.oz,
                baseY: gy,
                y: gy + (pieceType === "floor" ? 0.26 : BUILD_WALL_HEIGHT)
            };
            for (let i = 0; i < builtStructures.length; i++) {
                const s = builtStructures[i];
                if (!s.destroyed && s.health > 0 && areStructuresConnected(candidate, s)) {
                    return true;
                }
            }
            return false;
        }

        function updateBuildPreviewMesh(isSupported = true) {
            if (!buildPreviewGroup) return;
            while (buildPreviewGroup.children.length > 0) {
                buildPreviewGroup.remove(buildPreviewGroup.children[0]);
            }
            const canPlace = player.wood >= 10 && isSupported;
            const holoMat = new THREE.MeshBasicMaterial({
                color: canPlace ? 0x38bdf8 : 0xef4444,
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
                mesh.rotation.x = -Math.atan2(BUILD_WALL_HEIGHT, BUILD_GRID_SIZE);
            }
            buildPreviewGroup.add(mesh);
        }

        function updateBuildPreviewPosition() {
            if (!player.buildMode || !buildPreviewGroup) return;
            const snap = computeBuildSnapTransform();
            const supported = isBuildPlacementSupported(snap.gx, snap.gy, snap.gz, player.buildPiece, snap.cardRot);
            const canPlace = player.wood >= 10 && supported;
            if (buildPreviewGroup.children[0] && buildPreviewGroup.children[0].material) {
                buildPreviewGroup.children[0].material.color.setHex(canPlace ? 0x38bdf8 : 0xef4444);
            }
            buildPreviewGroup.rotation.set(0, snap.cardRot, 0);
            if (player.buildPiece === "wall") {
                const edge = getWallEdgeOffset(snap.cardRot);
                buildPreviewGroup.position.set(snap.gx + edge.ox, snap.gy + BUILD_WALL_HEIGHT / 2, snap.gz + edge.oz);
            } else if (player.buildPiece === "floor") {
                buildPreviewGroup.position.set(snap.gx, snap.gy + 0.12, snap.gz);
            } else {
                buildPreviewGroup.position.set(snap.gx, snap.gy + BUILD_WALL_HEIGHT / 2, snap.gz);
            }
        }

        function placeFortniteStructure() {
            if (!buildEnabled || !player.buildMode || player.wood < 10 || !player.alive) return;
            const snap = computeBuildSnapTransform();
            if (!isBuildPlacementSupported(snap.gx, snap.gy, snap.gz, player.buildPiece, snap.cardRot)) return;

            const edge = player.buildPiece === "wall" ? getWallEdgeOffset(snap.cardRot) : { ox: 0, oz: 0 };
            const placeX = snap.gx + edge.ox;
            const placeZ = snap.gz + edge.oz;

            const dup = builtStructures.some(s =>
                !s.destroyed &&
                s.type === player.buildPiece &&
                Math.abs(s.x - placeX) < 0.25 &&
                Math.abs(s.z - placeZ) < 0.25 &&
                Math.abs(s.baseY - snap.gy) < 0.25
            );
            if (dup) return;

            player.wood -= 10;
            playSound("build");
            updateWoodHUD();
            updateBuildPreviewMesh(true);

            const plankMat = new THREE.MeshStandardMaterial({ color: 0xa16207, roughness: 0.78 });
            const group = new THREE.Group();
            group.rotation.y = snap.cardRot;
            let structObj = null;

            if (player.buildPiece === "wall") {
                const wall = new THREE.Mesh(new THREE.BoxGeometry(BUILD_GRID_SIZE, BUILD_WALL_HEIGHT, 0.26), plankMat);
                wall.castShadow = true;
                wall.receiveShadow = true;
                group.add(wall);
                group.position.set(placeX, snap.gy + BUILD_WALL_HEIGHT / 2, placeZ);
                const isZAligned = Math.abs(Math.sin(snap.cardRot)) > 0.5;
                const col = addBoxCollider(
                    placeX, placeZ,
                    isZAligned ? 0.35 : BUILD_GRID_SIZE / 2,
                    isZAligned ? BUILD_GRID_SIZE / 2 : 0.35,
                    snap.gy,
                    snap.gy + BUILD_WALL_HEIGHT
                );
                structObj = { type: "wall", gridX: snap.gx, gridZ: snap.gz, x: placeX, z: placeZ, baseY: snap.gy, y: snap.gy + BUILD_WALL_HEIGHT, rotY: snap.cardRot, mesh: group, plankMesh: wall, collider: col, health: BUILD_MAX_HEALTH, maxHealth: BUILD_MAX_HEALTH, destroyed: false };
            } else if (player.buildPiece === "floor") {
                const floor = new THREE.Mesh(new THREE.BoxGeometry(BUILD_GRID_SIZE, 0.26, BUILD_GRID_SIZE), plankMat);
                floor.castShadow = true;
                floor.receiveShadow = true;
                group.add(floor);
                const topY = snap.gy + 0.26;
                group.position.set(placeX, snap.gy + 0.13, placeZ);
                const col = addBoxCollider(
                    placeX, placeZ,
                    BUILD_GRID_SIZE / 2,
                    BUILD_GRID_SIZE / 2,
                    snap.gy,
                    topY
                );
                structObj = { type: "floor", gridX: snap.gx, gridZ: snap.gz, x: placeX, z: placeZ, baseY: snap.gy, y: topY, rotY: snap.cardRot, mesh: group, plankMesh: floor, collider: col, health: BUILD_MAX_HEALTH, maxHealth: BUILD_MAX_HEALTH, destroyed: false };
            } else {
                const slopeLen = Math.hypot(BUILD_GRID_SIZE, BUILD_WALL_HEIGHT);
                const ramp = new THREE.Mesh(new THREE.BoxGeometry(BUILD_GRID_SIZE, 0.26, slopeLen), plankMat);
                ramp.rotation.x = -Math.atan2(BUILD_WALL_HEIGHT, BUILD_GRID_SIZE);
                ramp.castShadow = true;
                ramp.receiveShadow = true;
                group.add(ramp);
                group.position.set(placeX, snap.gy + BUILD_WALL_HEIGHT / 2, placeZ);
                structObj = { type: "ramp", gridX: snap.gx, gridZ: snap.gz, x: placeX, z: placeZ, baseY: snap.gy, y: snap.gy + BUILD_WALL_HEIGHT, rotY: snap.cardRot, mesh: group, plankMesh: ramp, collider: null, health: BUILD_MAX_HEALTH, maxHealth: BUILD_MAX_HEALTH, destroyed: false };
            }

            if (structObj) {
                structObj.plankMesh.userData.buildStruct = structObj;
                builtStructures.push(structObj);
            }

            scene.add(group);
            weaponHolder.position.y += 0.04;

            if (mpActiveRoom && window.mpPublishBuild) {
                window.mpPublishBuild({
                    type: player.buildPiece,
                    gridX: snap.gx,
                    gridZ: snap.gz,
                    x: placeX,
                    z: placeZ,
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

                const leftArm = new THREE.Mesh(new THREE.BoxGeometry(0.24, 0.64, 0.24), oliveTunicMat);
                leftArm.position.set(-0.48, 1.16, 0.04);
                const rightArm = new THREE.Mesh(new THREE.BoxGeometry(0.24, 0.64, 0.24), oliveTunicMat);
                rightArm.position.set(0.48, 1.16, 0.12);
                rightArm.rotation.x = -0.45;

                const leftShoulder = new THREE.Mesh(new THREE.BoxGeometry(0.28, 0.22, 0.28), tacticalVestMat);
                leftShoulder.position.set(-0.49, 1.38, 0.02);
                const rightShoulder = new THREE.Mesh(new THREE.BoxGeometry(0.28, 0.22, 0.28), tacticalVestMat);
                rightShoulder.position.set(0.49, 1.38, 0.02);

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
            else if (code === "KeyR") {
                if (player.buildMode) rotateBuildPiece();
                else triggerReload();
            }
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
            const buildTargets = [];

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

            builtStructures.forEach(s => {
                if (s && !s.destroyed && s.plankMesh && s.health > 0) {
                    s.plankMesh.userData.buildStruct = s;
                    buildTargets.push(s.plankMesh);
                }
            });

            const allTargets = [...headTargets, ...bodyTargets, ...remoteHeadTargets, ...remoteBodyTargets, ...buildTargets];

            for (let p = 0; p < w.pellets; p++) {
                const dir = forward.clone();
                dir.x += (Math.random() - 0.5) * w.spread;
                dir.y += (Math.random() - 0.5) * w.spread;
                dir.z += (Math.random() - 0.5) * w.spread;
                dir.normalize();

                const raycaster = new THREE.Raycaster(origin, dir, 0.2, w.range);
                const hits = raycaster.intersectObjects(allTargets, true);

                if (hits.length > 0) {
                    const hit = hits[0];
                    let uData = hit.object.userData || {};
                    if (!uData.buildStruct && hit.object.parent && hit.object.parent.userData && hit.object.parent.userData.buildStruct) {
                        uData = hit.object.parent.userData;
                    }
                    if (w.id !== "knife") spawnTracer(muzzlePos, hit.point, 0xfef08a);
                    if (uData.buildStruct) {
                        const structDmg = w.id === "knife" ? 50 : BODYSHOT_DAMAGE;
                        spawnImpactSparks(hit.point, 0xd97706);
                        flashHitmarker();
                        applyDamageToBuild(uData.buildStruct, structDmg, hit.point);
                    } else if (uData.bot) {
                        const dmg = uData.isHead ? HEADSHOT_DAMAGE : BODYSHOT_DAMAGE;
                        spawnImpactSparks(hit.point, uData.isHead ? 0xfacc15 : 0xef4444);
                        flashHitmarker();
                        applyDamageToBot(uData.bot, dmg, uData.isHead, player.name, true);
                    } else if (uData.remoteUid) {
                        const dmg = uData.isHead ? HEADSHOT_DAMAGE : BODYSHOT_DAMAGE;
                        spawnImpactSparks(hit.point, uData.isHead ? 0xfacc15 : 0x38bdf8);
                        flashHitmarker();
                        if (window.mpSendDamage) {
                            window.mpSendDamage(uData.remoteUid, uData.remoteName || "Player", dmg, uData.isHead);
                        }
                    }
                } else if (w.id !== "knife") {
                    const farPoint = origin.clone().addScaledVector(dir, Math.min(w.range, 120));
                    spawnTracer(muzzlePos, farPoint, 0xfef08a);
                }
            }
            updateHUD();
        }

        function destroySingleBuild(struct, hitPoint) {
            if (!struct || struct.destroyed) return;
            struct.destroyed = true;
            struct.health = 0;
            if (struct.plankMesh) {
                struct.plankMesh.visible = false;
                if (struct.plankMesh.parent) struct.plankMesh.parent.remove(struct.plankMesh);
            }
            if (struct.mesh) {
                struct.mesh.visible = false;
                if (struct.mesh.parent) struct.mesh.parent.remove(struct.mesh);
                scene.remove(struct.mesh);
            }
            if (struct.collider) {
                const cIdx = colliders.indexOf(struct.collider);
                if (cIdx !== -1) colliders.splice(cIdx, 1);
            }
            const sIdx = builtStructures.indexOf(struct);
            if (sIdx !== -1) builtStructures.splice(sIdx, 1);
        }

        function areStructuresConnected(a, b) {
            if (a === b) return false;
            const ax = a.gridX !== undefined ? a.gridX : Math.round(a.x / BUILD_GRID_SIZE) * BUILD_GRID_SIZE;
            const az = a.gridZ !== undefined ? a.gridZ : Math.round(a.z / BUILD_GRID_SIZE) * BUILD_GRID_SIZE;
            const bx = b.gridX !== undefined ? b.gridX : Math.round(b.x / BUILD_GRID_SIZE) * BUILD_GRID_SIZE;
            const bz = b.gridZ !== undefined ? b.gridZ : Math.round(b.z / BUILD_GRID_SIZE) * BUILD_GRID_SIZE;
            const dx = Math.abs(ax - bx);
            const dz = Math.abs(az - bz);
            const directDist = Math.hypot(a.x - b.x, a.z - b.z);
            const dy = Math.abs(a.baseY - b.baseY);

            // Same level or one story step (e.g., Floor at 3.6m resting on top of a 3.6m Ramp or Wall)
            if (dy <= BUILD_WALL_HEIGHT + 0.45) {
                if (directDist <= BUILD_GRID_SIZE + 0.55) return true;
                if ((dx <= BUILD_GRID_SIZE + 0.35 && dz <= 0.35) || (dz <= BUILD_GRID_SIZE + 0.35 && dx <= 0.35)) {
                    return true;
                }
            }
            return false;
        }

        function collapseUnsupportedStructures() {
            if (builtStructures.length === 0) return;
            const supported = new Set();
            const queue = [];

            builtStructures.forEach((s, idx) => {
                if (!s.destroyed && s.baseY <= 0.25) {
                    supported.add(idx);
                    queue.push(idx);
                }
            });

            while (queue.length > 0) {
                const currIdx = queue.shift();
                const curr = builtStructures[currIdx];
                for (let i = 0; i < builtStructures.length; i++) {
                    if (supported.has(i) || builtStructures[i].destroyed) continue;
                    if (areStructuresConnected(curr, builtStructures[i])) {
                        supported.add(i);
                        queue.push(i);
                    }
                }
            }

            const toCollapse = [];
            for (let i = 0; i < builtStructures.length; i++) {
                if (!supported.has(i) && !builtStructures[i].destroyed) {
                    toCollapse.push(builtStructures[i]);
                }
            }

            toCollapse.forEach(s => {
                destroySingleBuild(s, new THREE.Vector3(s.x, s.baseY + 1.5, s.z));
            });

            if (toCollapse.length > 0) {
                playSound("build");
            }
        }

        function applyDamageToBuild(struct, amount, hitPoint) {
            if (!struct || struct.destroyed || struct.health <= 0) return;
            struct.health = Math.max(0, struct.health - amount);
            if (struct.health > 0 && struct.plankMesh && struct.plankMesh.material) {
                const hpRatio = struct.health / struct.maxHealth;
                struct.plankMesh.material.color.setHSL(0.09, 0.85, 0.14 + hpRatio * 0.22);
            }
            if (struct.health <= 0) {
                destroySingleBuild(struct, hitPoint);
                playSound("build");
                collapseUnsupportedStructures();
            }
        }

        function applyDamageToBot(bot, amount, isHead, attackerName, isPlayerAttacker) {
            if (!bot.alive) return;
            bot.health = Math.max(0, bot.health - amount);
            if (bot.health === 0) {
                bot.alive = false;
                scene.remove(bot.mesh.root);
                if (isPlayerAttacker) {
                    player.kills += 1;
                    addKillFeedEntry(`${attackerName} eliminated ${bot.name}${isHead ? " (HEADSHOT)" : ""}`, "text-lime-300");
                } else {
                    addKillFeedEntry(`${attackerName} eliminated ${bot.name}`, "text-stone-300");
                }
                updateHUD();
                checkMatchOutcome();
            }
        }

        function applyDamageToPlayer(amount, attackerName, isHead) {
            if (!player.alive) return;
            player.health = Math.max(0, player.health - amount);

            const flash = document.getElementById("damage-flash");
            flash.style.opacity = "1";
            setTimeout(() => { flash.style.opacity = "0"; }, 180);

            updateHUD();
            if (player.health === 0) {
                player.alive = false;
                const aliveEnemies = bots.filter(b => b.alive).length + Object.values(mpRemotePlayers).filter(r => r.data && r.data.alive).length;
                addKillFeedEntry(`${attackerName} eliminated ${player.name}${isHead ? " (HEADSHOT)" : ""}`, "text-red-400");
                showMatchEndBanner(false, aliveEnemies + 1);
            }
        }

        function addKillFeedEntry(text, colorClass) {
            const feed = document.getElementById("kill-feed");
            const item = document.createElement("div");
            item.className = `glass-panel px-3 py-1 rounded-xl text-[11px] font-bold ${colorClass} shadow`;
            item.textContent = text;
            feed.prepend(item);
            while (feed.children.length > 5) {
                feed.removeChild(feed.lastChild);
            }
            setTimeout(() => {
                if (item.parentNode) item.remove();
            }, 5000);
        }

        function checkMatchOutcome() {
            if (!player.alive) return;
            const aliveBots = bots.filter(b => b.alive).length;
            const aliveRemote = Object.values(mpRemotePlayers).filter(r => r.data && r.data.alive).length;
            if (gameMode === "pvp") {
                if (mpActiveRoom && Object.keys(mpRemotePlayers).length > 0 && aliveRemote === 0) {
                    showMatchEndBanner(true, 1);
                }
                return;
            }
            if (aliveBots === 0 && aliveRemote === 0) {
                showMatchEndBanner(true, 1);
            }
        }

        // =========================================================================
        // 9. ZOMBIE HORDE AI, CLASSIC BR BOT AI & PLAYER AUTO-SIMULATION
        // =========================================================================
        function updateBots(dt) {
            const diffSpeed = lobbyDifficulty === "nightmare" ? 1.3 : (lobbyDifficulty === "hard" ? 1.12 : 1.0);
            const diffFire = lobbyDifficulty === "nightmare" ? 0.72 : (lobbyDifficulty === "hard" ? 0.85 : 1.0);

            for (let i = 0; i < bots.length; i++) {
                const b = bots[i];
                if (!b.alive) continue;

                b.fireCooldown = Math.max(0, b.fireCooldown - dt);
                b.strafeTimer -= dt;
                if (b.strafeTimer <= 0) {
                    b.strafeDir *= -1;
                    b.strafeTimer = 1.4 + Math.random() * 2.2;
                }

                let targetPos = null;
                let targetBot = null;
                let targetIsPlayer = false;
                let bestDist = Infinity;

                if (b.isZombie) {
                    if (player.alive) {
                        targetPos = player.pos;
                        targetIsPlayer = true;
                        bestDist = b.pos.distanceTo(player.pos);
                    }
                } else {
                    if (player.alive) {
                        const dPlayer = b.pos.distanceTo(player.pos);
                        if (dPlayer < 130) {
                            bestDist = dPlayer;
                            targetPos = player.pos;
                            targetIsPlayer = true;
                        }
                    }

                    for (let j = 0; j < bots.length; j++) {
                        if (i === j || !bots[j].alive) continue;
                        const dBot = b.pos.distanceTo(bots[j].pos);
                        if (dBot < bestDist && dBot < 120) {
                            bestDist = dBot;
                            targetPos = bots[j].pos;
                            targetBot = bots[j];
                            targetIsPlayer = false;
                        }
                    }
                }

                if (targetPos) {
                    const toTarget = new THREE.Vector3().subVectors(targetPos, b.pos);
                    toTarget.y = 0;
                    const dist = toTarget.length();
                    if (dist > 0.01) toTarget.normalize();

                    b.yaw = Math.atan2(toTarget.x, toTarget.z);
                    b.mesh.root.rotation.y = b.yaw;

                    if (b.isZombie) {
                        const zombieSpeed = 8.4 * diffSpeed;
                        if (dist > 1.8) {
                            b.pos.addScaledVector(toTarget, zombieSpeed * dt);
                            b.walkPhase += dt * 11;
                            b.mesh.leftLeg.rotation.x = Math.sin(b.walkPhase) * 0.65;
                            b.mesh.rightLeg.rotation.x = -Math.sin(b.walkPhase) * 0.65;
                            b.mesh.leftArm.rotation.x = -0.25 + Math.sin(b.walkPhase) * 0.18;
                            b.mesh.rightArm.rotation.x = -0.25 - Math.sin(b.walkPhase) * 0.18;
                        } else {
                            b.mesh.leftLeg.rotation.x = 0;
                            b.mesh.rightLeg.rotation.x = 0;
                            if (b.fireCooldown <= 0 && targetIsPlayer && player.alive && Math.abs(player.pos.y - b.pos.y) < 2.2) {
                                b.fireCooldown = 1.05 * diffFire;
                                applyDamageToPlayer(ZOMBIE_CLAW_DAMAGE, b.name, false);
                            }
                        }
                    } else {
                        const rightVec = new THREE.Vector3(-toTarget.z, 0, toTarget.x);
                        const moveSpeed = 7.4 * diffSpeed;
                        if (dist > 22) {
                            b.pos.addScaledVector(toTarget, moveSpeed * dt);
                        } else if (dist < 10) {
                            b.pos.addScaledVector(toTarget, -moveSpeed * 0.7 * dt);
                        }
                        b.pos.addScaledVector(rightVec, b.strafeDir * moveSpeed * 0.65 * dt);

                        b.walkPhase += dt * 9;
                        b.mesh.leftLeg.rotation.x = Math.sin(b.walkPhase) * 0.55;
                        b.mesh.rightLeg.rotation.x = -Math.sin(b.walkPhase) * 0.55;

                        if (dist < 95 && b.fireCooldown <= 0) {
                            b.fireCooldown = (0.75 + Math.random() * 0.55) * diffFire;
                            const start = b.pos.clone().add(new THREE.Vector3(0, 1.45, 0));
                            const end = targetPos.clone().add(new THREE.Vector3(
                                (Math.random() - 0.5) * 1.4,
                                1.2 + (Math.random() - 0.5) * 0.6,
                                (Math.random() - 0.5) * 1.4
                            ));
                            spawnTracer(start, end, 0xff6b6b);

                            const hitChance = Math.max(0.18, 0.55 - dist * 0.0035);
                            if (Math.random() < hitChance) {
                                const isHead = Math.random() < 0.22;
                                const dmg = isHead ? BOT_HEADSHOT_DAMAGE : BOT_BODYSHOT_DAMAGE;
                                if (targetIsPlayer) {
                                    applyDamageToPlayer(dmg, b.name, isHead);
                                } else if (targetBot) {
                                    applyDamageToBot(targetBot, dmg, isHead, b.name, false);
                                }
                            }
                        }
                    }
                } else {
                    b.pos.x += Math.sin(b.yaw) * 4.5 * dt;
                    b.pos.z += Math.cos(b.yaw) * 4.5 * dt;
                    if (Math.random() < 0.015) b.yaw += (Math.random() - 0.5) * 1.6;
                    b.mesh.root.rotation.y = b.yaw;
                }

                resolveArenaCollisions(b.pos);
                b.mesh.root.position.copy(b.pos);
            }
        }

        function updateSimulatedPlayer(dt) {
            if (!player.alive) return;

            if (player.health < 55 && player.medkits > 0 && !player.isHealing) {
                startMedkitHeal();
                return;
            }
            if (player.isHealing) return;

            for (let i = 0; i < chests.length; i++) {
                const c = chests[i];
                if (!c.opened && player.pos.distanceTo(c.pos) < 3.4) {
                    openChest(c);
                }
            }

            let targetPos = null;
            let bestDist = Infinity;

            if (!player.weapons[0].owned && !player.weapons[1].owned) {
                for (let i = 0; i < chests.length; i++) {
                    if (!chests[i].opened) {
                        const d = player.pos.distanceTo(chests[i].pos);
                        if (d < bestDist) {
                            bestDist = d;
                            targetPos = chests[i].pos;
                        }
                    }
                }
            }

            let nearestEnemy = null;
            let nearestEnemyDist = Infinity;
            for (let i = 0; i < bots.length; i++) {
                if (!bots[i].alive) continue;
                const d = player.pos.distanceTo(bots[i].pos);
                if (d < nearestEnemyDist) {
                    nearestEnemyDist = d;
                    nearestEnemy = bots[i];
                }
            }

            if (nearestEnemy && (nearestEnemyDist < 85 || !targetPos)) {
                const aimVec = new THREE.Vector3().subVectors(nearestEnemy.pos, player.pos);
                const desiredYaw = Math.atan2(-aimVec.x, -aimVec.z);
                player.yaw = desiredYaw;
                player.pitch = THREE.MathUtils.clamp((1.45 - 1.65) / Math.max(nearestEnemyDist, 4), -0.4, 0.4);

                const forward = new THREE.Vector3(-Math.sin(player.yaw), 0, -Math.cos(player.yaw));
                const strafe = new THREE.Vector3(Math.cos(player.yaw), 0, -Math.sin(player.yaw));
                if (nearestEnemyDist > 26) {
                    player.pos.addScaledVector(forward, 10.5 * dt);
                } else if (nearestEnemyDist < 12) {
                    player.pos.addScaledVector(forward, -9.0 * dt);
                }
                player.pos.addScaledVector(strafe, Math.sin(performance.now() * 0.0025) * 6.5 * dt);

                if (nearestEnemyDist < 90) {
                    attemptPlayerAttack();
                }
            } else if (targetPos) {
                const moveVec = new THREE.Vector3().subVectors(targetPos, player.pos);
                moveVec.y = 0;
                if (moveVec.length() > 0.5) {
                    moveVec.normalize();
                    player.yaw = Math.atan2(-moveVec.x, -moveVec.z);
                    player.pos.addScaledVector(moveVec, 11.0 * dt);
                }
            }
        }

        // =========================================================================
        // 10. PLAYER MOVEMENT, CHEST INTERACTION & VISUAL EFFECTS UPDATE
        // =========================================================================
        function tryInteractChest() {
            for (let i = 0; i < chests.length; i++) {
                const c = chests[i];
                if (!c.opened && player.pos.distanceTo(c.pos) < 3.6) {
                    openChest(c);
                    return;
                }
            }
        }

        function updatePlayer(dt) {
            if (!player.alive) return;

            player.fireCooldown = Math.max(0, player.fireCooldown - dt);

            if (player.reloading) {
                player.reloadTimer -= dt;
                if (player.reloadTimer <= 0) {
                    player.reloading = false;
                    const w = player.weapons[player.activeSlot];
                    const needed = w.magSize - w.mag;
                    const load = Math.min(needed, w.reserve);
                    w.mag += load;
                    w.reserve -= load;
                    updateHUD();
                }
            }

            if (player.isHealing) {
                player.healTimer -= dt;
                const pct = THREE.MathUtils.clamp(((MEDKIT_DURATION - player.healTimer) / MEDKIT_DURATION) * 100, 0, 100);
                document.getElementById("medkit-progress-bar").style.width = `${pct}%`;
                document.getElementById("medkit-timer-text").textContent = `${Math.max(0, player.healTimer).toFixed(1)}s`;
                if (player.healTimer <= 0) {
                    completeMedkitHeal();
                }
            }

            if (simModeActive) {
                updateSimulatedPlayer(dt);
            } else {
                const turnSpeed = 2.1;
                if (keys["ArrowLeft"]) player.yaw += turnSpeed * dt;
                if (keys["ArrowRight"]) player.yaw -= turnSpeed * dt;
                if (keys["ArrowUp"]) player.pitch = THREE.MathUtils.clamp(player.pitch + turnSpeed * 0.7 * dt, -1.42, 1.42);
                if (keys["ArrowDown"]) player.pitch = THREE.MathUtils.clamp(player.pitch - turnSpeed * 0.7 * dt, -1.42, 1.42);

                const forward = new THREE.Vector3(-Math.sin(player.yaw), 0, -Math.cos(player.yaw));
                const right = new THREE.Vector3(Math.cos(player.yaw), 0, -Math.sin(player.yaw));
                const moveDir = new THREE.Vector3(0, 0, 0);

                if (keys["KeyW"]) moveDir.add(forward);
                if (keys["KeyS"]) moveDir.sub(forward);
                if (keys["KeyD"]) moveDir.add(right);
                if (keys["KeyA"]) moveDir.sub(right);

                const speed = keys["ShiftLeft"] ? 14.5 : 10.2;
                if (moveDir.lengthSq() > 0) {
                    moveDir.normalize();
                    player.pos.addScaledVector(moveDir, speed * dt);
                }

                if (keys["Space"] && player.grounded) {
                    player.vel.y = 8.4;
                    player.grounded = false;
                }

                if (isMouseDown && !player.buildMode && player.weapons[player.activeSlot].auto) {
                    attemptPlayerAttack();
                }
            }

            player.vel.y -= 22.0 * dt;
            player.pos.y += player.vel.y * dt;
            const floorSupportY = getSupportHeightAt(player.pos.x, player.pos.z, player.pos.y);
            if (player.pos.y <= floorSupportY) {
                player.pos.y = floorSupportY;
                player.vel.y = 0;
                player.grounded = true;
            } else if (player.vel.y <= 0 && player.pos.y - floorSupportY < 0.85) {
                player.pos.y = floorSupportY;
                player.vel.y = 0;
                player.grounded = true;
            } else {
                player.grounded = false;
            }

            resolveArenaCollisions(player.pos);
            const postCollisionSupportY = getSupportHeightAt(player.pos.x, player.pos.z, player.pos.y);
            if (postCollisionSupportY > player.pos.y && postCollisionSupportY - player.pos.y <= 1.75) {
                player.pos.y = postCollisionSupportY;
                player.vel.y = 0;
                player.grounded = true;
            }

            resolveArenaCollisions(player.pos);

            camera.position.set(player.pos.x, player.pos.y + 1.65, player.pos.z);
            camera.rotation.y = player.yaw;
            camera.rotation.x = player.pitch;

            updateBuildPreviewPosition();

            weaponHolder.position.z = THREE.MathUtils.lerp(weaponHolder.position.z, -0.45, Math.min(1, dt * 16));
            weaponHolder.position.y = THREE.MathUtils.lerp(weaponHolder.position.y, -0.24, Math.min(1, dt * 16));
            weaponHolder.rotation.x = THREE.MathUtils.lerp(weaponHolder.rotation.x, 0, Math.min(1, dt * 16));

            let nearChest = false;
            for (let i = 0; i < chests.length; i++) {
                if (!chests[i].opened && player.pos.distanceTo(chests[i].pos) < 3.6) {
                    nearChest = true;
                    break;
                }
            }
            const promptEl = document.getElementById("loot-prompt");
            if (nearChest && !simModeActive) promptEl.classList.remove("hidden");
            else promptEl.classList.add("hidden");
        }

        function updateEffects(dt) {
            for (let i = tracers.length - 1; i >= 0; i--) {
                tracers[i].life -= dt;
                tracers[i].line.material.opacity = Math.max(0, tracers[i].life / 0.12);
                if (tracers[i].life <= 0) {
                    scene.remove(tracers[i].line);
                    tracers[i].line.geometry.dispose();
                    tracers.splice(i, 1);
                }
            }

            for (let i = particles.length - 1; i >= 0; i--) {
                const p = particles[i];
                p.life -= dt;
                p.vel.y -= 18 * dt;
                p.mesh.position.addScaledVector(p.vel, dt);
                if (p.life <= 0) {
                    scene.remove(p.mesh);
                    particles.splice(i, 1);
                }
            }
        }

        // =========================================================================
        // 11. MINIMAP, HUD UPDATES & LOCKER / GAMEPLAY TABS
        // =========================================================================
        const minimapCanvas = document.getElementById("minimap-canvas");
        const minimapCtx = minimapCanvas.getContext("2d");

        function renderMinimap() {
            const size = minimapCanvas.width;
            const half = size / 2;
            const scale = half / ARENA_HALF_SIZE;

            minimapCtx.clearRect(0, 0, size, size);
            minimapCtx.fillStyle = selectedMap === "city" ? "#1e293b" : (selectedMap === "desert" ? "#3b2c1a" : "#121c15");
            minimapCtx.fillRect(0, 0, size, size);

            minimapCtx.strokeStyle = "rgba(163, 230, 53, 0.12)";
            minimapCtx.lineWidth = 1;
            minimapCtx.beginPath();
            minimapCtx.moveTo(half, 0);
            minimapCtx.lineTo(half, size);
            minimapCtx.moveTo(0, half);
            minimapCtx.lineTo(size, half);
            minimapCtx.stroke();

            chests.forEach(c => {
                if (c.opened) return;
                minimapCtx.fillStyle = "#facc15";
                minimapCtx.fillRect(half + c.pos.x * scale - 1.5, half + c.pos.z * scale - 1.5, 3, 3);
            });

            bots.forEach(b => {
                if (!b.alive) return;
                minimapCtx.fillStyle = "#ef4444";
                minimapCtx.beginPath();
                minimapCtx.arc(half + b.pos.x * scale, half + b.pos.z * scale, 2.5, 0, Math.PI * 2);
                minimapCtx.fill();
            });

            Object.values(mpRemotePlayers).forEach(rp => {
                if (!rp.data || !rp.data.alive) return;
                minimapCtx.fillStyle = "#38bdf8";
                minimapCtx.beginPath();
                minimapCtx.arc(half + rp.data.x * scale, half + rp.data.z * scale, 3.2, 0, Math.PI * 2);
                minimapCtx.fill();
            });

            if (player.alive) {
                const px = half + player.pos.x * scale;
                const pz = half + player.pos.z * scale;
                minimapCtx.save();
                minimapCtx.translate(px, pz);
                minimapCtx.rotate(-player.yaw);
                minimapCtx.fillStyle = "#a3e635";
                minimapCtx.beginPath();
                minimapCtx.moveTo(0, -5);
                minimapCtx.lineTo(3.8, 4);
                minimapCtx.lineTo(-3.8, 4);
                minimapCtx.closePath();
                minimapCtx.fill();
                minimapCtx.restore();
            }
        }

        function updateHUD() {
            const aliveBots = bots.filter(b => b.alive).length;
            const aliveRemote = Object.values(mpRemotePlayers).filter(r => r.data && r.data.alive).length;
            const totalParticipants = (gameMode === "pvp" ? 1 : (BOT_COUNT + 1)) + Object.keys(mpRemotePlayers).length;
            const aliveTotal = (player.alive ? 1 : 0) + aliveBots + aliveRemote;

            const modeText = gameMode === "zombies" ? "ZOMBIES" : (gameMode === "classic" ? "CLASSIC BR" : "PVP ARENA");
            const buildTag = buildEnabled ? "BUILD" : "ZERO BUILD";
            document.getElementById("hud-mode-badge").textContent = `DEAD SHOT (${aliveTotal}/${totalParticipants} Alive)`;
            document.getElementById("hud-submode-badge").textContent = `${modeText} • ${buildTag}`;

            const partyLiveEl = document.getElementById("hud-party-live");
            if (mpActiveRoom) {
                const partyNames = [player.name, ...Object.values(mpRemotePlayers).map(r => (r.data && r.data.name) || "Player")];
                partyLiveEl.textContent = `PARTY (${mpActiveRoom}): ${partyNames.join(", ")}`;
                partyLiveEl.classList.remove("hidden");
            } else {
                partyLiveEl.classList.add("hidden");
            }

            document.getElementById("hud-health-val").textContent = player.health;
            document.getElementById("hud-health-bar").style.width = `${player.health}%`;
            document.getElementById("hud-kills-val").textContent = player.kills;
            updateWoodHUD();

            const woodPanel = document.getElementById("hud-wood-panel");
            const controlsHint = document.getElementById("hud-controls-hint");
            if (buildEnabled) {
                woodPanel.classList.remove("hidden");
                if (controlsHint) controlsHint.textContent = "[Q] Build • [H] Medkit";
            } else {
                woodPanel.classList.add("hidden");
                if (controlsHint) controlsHint.textContent = "[H] Medkit • [E] Loot";
            }

            const buildBar = document.getElementById("hud-build-bar");
            if (buildEnabled && player.buildMode) {
                buildBar.classList.remove("hidden");
                buildBar.classList.add("flex");
                ["wall", "floor", "ramp"].forEach(p => {
                    const btn = document.getElementById(`bpiece-${p}`);
                    if (!btn) return;
                    if (player.buildPiece === p) {
                        btn.className = "px-2 py-0.5 rounded bg-sky-500 text-stone-950 font-black border-2 border-white";
                    } else {
                        btn.className = "px-2 py-0.5 rounded bg-stone-800 text-white border border-stone-700";
                    }
                });
            } else {
                buildBar.classList.add("hidden");
                buildBar.classList.remove("flex");
            }

            const w = player.weapons[player.activeSlot];
            if (buildEnabled && player.buildMode) {
                const rotDeg = player.buildRotationOffset * 90;
                document.getElementById("hud-weapon-name").textContent = `BLUEPRINT: ${player.buildPiece.toUpperCase()} (${rotDeg}°)`;
                document.getElementById("hud-ammo-mag").textContent = "10 WOOD";
                document.getElementById("hud-ammo-res").textContent = `/ ${player.wood}`;
            } else {
                document.getElementById("hud-weapon-name").textContent = player.reloading ? `${w.name} (RELOADING...)` : w.name;
                if (w.id === "knife") {
                    document.getElementById("hud-ammo-mag").textContent = "MELEE";
                    document.getElementById("hud-ammo-res").textContent = "";
                } else if (w.id === "medkit") {
                    document.getElementById("hud-ammo-mag").textContent = `${player.medkits}`;
                    document.getElementById("hud-ammo-res").textContent = "KITS";
                } else {
                    document.getElementById("hud-ammo-mag").textContent = w.mag;
                    document.getElementById("hud-ammo-res").textContent = `/ ${w.reserve}`;
                }
            }

            document.getElementById("slot-4-label").textContent = `MED (${player.medkits})`;
            for (let i = 0; i < 5; i++) {
                const btn = document.getElementById(`slot-${i}`);
                const owned = player.weapons[i].owned || (i === 4 && player.medkits > 0);
                btn.style.opacity = owned ? "1" : "0.38";
                if (i === player.activeSlot && !player.buildMode) {
                    btn.classList.remove("border-stone-700");
                    btn.classList.add("border-2", "border-lime-400");
                } else {
                    btn.classList.remove("border-2", "border-lime-400");
                    btn.classList.add("border", "border-stone-700");
                }
            }
        }

        function refreshRosterUI() {
            const rosterEl = document.getElementById("roster-list");
            const aliveBots = bots.filter(b => b.alive).length;
            document.getElementById("menu-alive-count").textContent = gameMode === "pvp" ? "PVP ARENA" : `${aliveBots} REMAINING`;
            rosterEl.innerHTML = "";
            if (gameMode === "pvp") {
                const note = document.createElement("div");
                note.className = "text-stone-400 text-xs py-2";
                note.textContent = "PvP Arena active — fight live players in your room!";
                rosterEl.appendChild(note);
                return;
            }
            bots.forEach(b => {
                const row = document.createElement("div");
                row.className = "flex items-center justify-between px-2.5 py-1 rounded-lg bg-stone-950/70";
                row.innerHTML = `
                    <span class="${b.alive ? "text-stone-200 font-semibold" : "text-stone-500 line-through"}">${b.name}</span>
                    <span class="mono text-[10px] font-bold ${b.alive ? "text-lime-400" : "text-red-400"}">${b.alive ? b.health + " HP" : "ELIMINATED"}</span>
                `;
                rosterEl.appendChild(row);
            });
        }

        function openPauseMenu() {
            if (inMainLobby) return;
            isPaused = true;
            isMouseDown = false;
            unlockLookControl();
            refreshRosterUI();
            const overlay = document.getElementById("pause-menu-overlay");
            overlay.classList.remove("hidden");
            overlay.classList.add("flex");
        }

        function resumeGameplay() {
            isPaused = false;
            const overlay = document.getElementById("pause-menu-overlay");
            overlay.classList.add("hidden");
            overlay.classList.remove("flex");
            engageLookControl();
        }

        function showMatchEndBanner(isVictory, placement) {
            isPaused = true;
            isMouseDown = false;
            unlockLookControl();

            const totalPlayers = (gameMode === "pvp" ? 1 : (BOT_COUNT + 1)) + Object.keys(mpRemotePlayers).length;
            const tag = document.getElementById("end-banner-tag");
            const title = document.getElementById("end-banner-title");
            const sub = document.getElementById("end-banner-sub");

            if (isVictory) {
                tag.textContent = "VICTORY ROYALE";
                tag.className = "inline-block px-3 py-1 rounded-full bg-lime-400/20 text-lime-300 text-xs font-black uppercase tracking-widest mb-2";
                title.textContent = "VICTORY ROYALE!";
                sub.textContent = gameMode === "zombies"
                    ? `${player.name} wiped out all 19 infected in Dead Shot!`
                    : `${player.name} is the last survivor standing in Dead Shot!`;
            } else {
                tag.textContent = "ELIMINATED";
                tag.className = "inline-block px-3 py-1 rounded-full bg-red-500/20 text-red-300 text-xs font-black uppercase tracking-widest mb-2";
                title.textContent = "YOU WERE ELIMINATED";
                sub.textContent = `${player.name} fell in combat. Deploy again to reclaim Victory Royale.`;
            }

            document.getElementById("end-stat-kills").textContent = player.kills;
            document.getElementById("end-stat-place").textContent = `#${placement} / ${totalPlayers}`;

            const overlay = document.getElementById("match-end-overlay");
            overlay.classList.remove("hidden");
            overlay.classList.add("flex");
        }

        function toggleSimMode() {
            simModeActive = !simModeActive;
            const btn = document.getElementById("btn-sim-toggle");
            if (simModeActive) {
                btn.textContent = "[M] Mode: Auto-Play AI";
                btn.classList.remove("text-emerald-300");
                btn.classList.add("text-amber-300");
                unlockLookControl();
            } else {
                btn.textContent = "[M] Mode: Manual Play";
                btn.classList.remove("text-amber-300");
                btn.classList.add("text-emerald-300");
            }
        }

        function setSimSpeed(mult) {
            simSpeed = mult;
            [1, 2, 4].forEach(s => {
                const btn = document.getElementById(`spd-${s}`);
                if (s === mult) {
                    btn.className = "px-2 py-0.5 rounded bg-lime-600 text-stone-950 font-bold text-[10px]";
                } else {
                    btn.className = "px-2 py-0.5 rounded bg-stone-800 text-stone-300 font-bold text-[10px]";
                }
            });
        }

        function updateLobbyBriefingUI() {
            const topBadge = document.getElementById("lobby-top-badge");
            const subEl = document.getElementById("lobby-subtitle");
            const enemyLabel = document.getElementById("lobby-enemy-label");
            const enemyCount = document.getElementById("lobby-enemy-count");
            const mapLabel = document.getElementById("lobby-map-label");

            const mapNames = { woodland: "Woodland", city: "City", desert: "Desert" };
            if (mapLabel) mapLabel.textContent = `${mapNames[selectedMap] || "Woodland"} Supply Chests`;

            const buildLabel = buildEnabled ? "Build Mode" : "Zero Build";
            if (gameMode === "zombies") {
                topBadge.textContent = "OUTBREAK ZONE • 19 INFECTED";
                subEl.textContent = `Zombie Survival • ${buildLabel} • 100 HP • 25 Headshot / 15 Body`;
                enemyLabel.textContent = "Infected Horde (x19)";
                enemyCount.textContent = "19 ZOMBIES DETECTED";
            } else if (gameMode === "classic") {
                topBadge.textContent = "BATTLE ROYALE • 19 SOLDIERS";
                subEl.textContent = `Classic Battle Royale • ${buildLabel} • 100 HP • 25 Headshot / 15 Body`;
                enemyLabel.textContent = "Military Squads (x19)";
                enemyCount.textContent = "19 BOTS FIGHTING";
            } else {
                topBadge.textContent = "ONLINE PVP ARENA • LIVE PLAYERS";
                subEl.textContent = `Multiplayer PvP • ${buildLabel} • 100 HP • 25 Headshot / 15 Body`;
                enemyLabel.textContent = "Match Format";
                enemyCount.textContent = "LIVE PLAYERS ONLY";
            }
        }

        function setLobbyGameMode(mode) {
            gameMode = mode;
            ["zombies", "classic", "pvp"].forEach(m => {
                const btn = document.getElementById(`mode-${m}`);
                if (!btn) return;
                if (m === mode) {
                    btn.className = "py-2 px-1 rounded-xl bg-lime-500 text-stone-950 font-black text-[11px] uppercase transition";
                } else {
                    btn.className = "py-2 px-1 rounded-xl bg-stone-800 text-stone-300 hover:bg-stone-700 font-extrabold text-[11px] uppercase transition";
                }
            });
            updateLobbyBriefingUI();
        }

        function setLobbyBuildMode(enabled) {
            buildEnabled = enabled;
            const onBtn = document.getElementById("build-opt-on");
            const offBtn = document.getElementById("build-opt-off");
            if (enabled) {
                onBtn.className = "py-2 rounded-xl bg-lime-500 text-stone-950 font-black text-xs uppercase transition";
                offBtn.className = "py-2 rounded-xl bg-stone-800 text-stone-300 hover:bg-stone-700 font-extrabold text-xs uppercase transition";
            } else {
                offBtn.className = "py-2 rounded-xl bg-lime-500 text-stone-950 font-black text-xs uppercase transition";
                onBtn.className = "py-2 rounded-xl bg-stone-800 text-stone-300 hover:bg-stone-700 font-extrabold text-xs uppercase transition";
            }
            updateLobbyBriefingUI();
        }

        function setLobbyDifficulty(diff) {
            lobbyDifficulty = diff;
            ["normal", "hard", "nightmare"].forEach(d => {
                const btn = document.getElementById(`diff-${d}`);
                if (d === diff) {
                    btn.className = "py-2 rounded-xl bg-lime-500 text-stone-950 font-black text-xs uppercase transition";
                } else {
                    btn.className = "py-2 rounded-xl bg-stone-800 text-stone-300 hover:bg-stone-700 font-extrabold text-xs uppercase transition";
                }
            });
        }

        function setLobbyMap(mapName) {
            selectedMap = mapName;
            ["woodland", "city", "desert"].forEach(m => {
                const btn = document.getElementById(`map-${m}`);
                if (m === mapName) {
                    btn.className = "py-2 rounded-xl bg-lime-500 text-stone-950 font-black text-xs uppercase transition";
                } else {
                    btn.className = "py-2 rounded-xl bg-stone-800 text-stone-300 hover:bg-stone-700 font-extrabold text-xs uppercase transition";
                }
            });
            buildSelectedMap(selectedMap);
            updateLobbyBriefingUI();
        }

        function applyEquippedSkinMaterials() {
            const cfg = PLAYER_SKINS[selectedSkin] || PLAYER_SKINS.ranger;
            if (skinPrimaryMat) {
                skinPrimaryMat.color.setHex(cfg.gunPrimary);
                skinPrimaryMat.metalness = cfg.metalness;
            }
            if (skinAccentMat) {
                skinAccentMat.color.setHex(cfg.gunAccent);
            }
            if (sleeveMesh && sleeveMesh.material) {
                sleeveMesh.material.color.setHex(cfg.sleeve);
            }
            if (gloveMesh && gloveMesh.material) {
                gloveMesh.material.color.setHex(cfg.glove);
            }
            updateLockerPreviewMaterials();
        }

        function setLobbySkin(skinKey) {
            selectedSkin = skinKey;
            const cfg = PLAYER_SKINS[skinKey] || PLAYER_SKINS.ranger;
            ["ranger", "crimson", "arctic", "gold"].forEach(k => {
                const btn = document.getElementById(`skin-${k}`);
                if (!btn) return;
                if (k === skinKey) {
                    btn.className = "p-3 rounded-2xl bg-lime-500 text-stone-950 font-black text-xs uppercase text-left transition flex flex-col gap-1";
                } else {
                    btn.className = "p-3 rounded-2xl bg-stone-800 text-stone-300 hover:bg-stone-700 font-extrabold text-xs uppercase text-left transition flex flex-col gap-1";
                }
            });
            const badge = document.getElementById("lobby-skin-badge");
            if (badge) {
                badge.textContent = cfg.label.toUpperCase();
                badge.style.backgroundColor = cfg.hudHex;
            }
            const summary = document.getElementById("gameplay-skin-summary");
            if (summary) summary.textContent = cfg.label.toUpperCase();
            const briefingSkin = document.getElementById("lobby-briefing-skin");
            if (briefingSkin) briefingSkin.textContent = cfg.label;
            const lockerOutfitDetail = document.getElementById("locker-detail-outfit");
            if (lockerOutfitDetail) lockerOutfitDetail.textContent = `${cfg.label} Tactical`;

            applyEquippedSkinMaterials();
        }

        function syncLobbyCallsignPreview() {
            const raw = document.getElementById("lobby-player-name").value.trim() || "Survivor-01";
            player.name = raw;
            const previewEl = document.getElementById("lobby-preview-name");
            if (previewEl) previewEl.textContent = `${raw} (You)`;
            const lockerTagEl = document.getElementById("locker-detail-tag");
            if (lockerTagEl) lockerTagEl.textContent = raw;
            if (window.mpRefreshLobbyRoster) window.mpRefreshLobbyRoster();
        }

        // =========================================================================
        // 11B. LOCKER 3D CHARACTER & WEAPON SKIN PREVIEW SCENE
        // =========================================================================
        let lockerScene, lockerCamera, lockerRenderer, lockerCharGroup;
        let lockerSleeveMeshes = [], lockerGloveMeshes = [], lockerVestMeshes = [], lockerGunPrimaryMeshes = [], lockerGunAccentMeshes = [];
        let lockerTargetYaw = 0.45;
        let lockerIsDragging = false;
        let lockerPrevMouseX = 0;

        function initLocker3DPreview() {
            const canvas = document.getElementById("locker-preview-canvas");
            if (!canvas || typeof THREE === "undefined") return;

            lockerScene = new THREE.Scene();
            lockerCamera = new THREE.PerspectiveCamera(42, 1.8, 0.1, 50);
            lockerCamera.position.set(0, 1.35, 3.35);
            lockerCamera.lookAt(0, 1.08, 0);

            lockerRenderer = new THREE.WebGLRenderer({ canvas, alpha: true, antialias: true });
            lockerRenderer.setPixelRatio(Math.min(window.devicePixelRatio, 2));

            const hemi = new THREE.HemisphereLight(0xffffff, 0x1e293b, 1.05);
            lockerScene.add(hemi);
            const keyLight = new THREE.DirectionalLight(0xfef9c3, 1.25);
            keyLight.position.set(2.5, 4.0, 3.0);
            lockerScene.add(keyLight);
            const rimLight = new THREE.DirectionalLight(0x38bdf8, 0.7);
            rimLight.position.set(-2.5, 2.5, -2.5);
            lockerScene.add(rimLight);

            const pedMat = new THREE.MeshStandardMaterial({ color: 0x18181b, metalness: 0.7, roughness: 0.3 });
            const ringMat = new THREE.MeshBasicMaterial({ color: 0xa3e635 });
            const pedestal = new THREE.Mesh(new THREE.CylinderGeometry(0.85, 0.95, 0.12, 24), pedMat);
            pedestal.position.y = -0.06;
            const glowRing = new THREE.Mesh(new THREE.TorusGeometry(0.86, 0.018, 8, 36), ringMat);
            glowRing.rotation.x = Math.PI / 2;
            glowRing.position.y = 0.01;
            lockerScene.add(pedestal, glowRing);

            lockerCharGroup = new THREE.Group();
            const cfg = PLAYER_SKINS[selectedSkin] || PLAYER_SKINS.ranger;

            const outfitMat = new THREE.MeshStandardMaterial({ color: cfg.sleeve, roughness: 0.65 });
            const gearMat = new THREE.MeshStandardMaterial({ color: cfg.glove, roughness: 0.45, metalness: 0.25 });
            const pantsMat = new THREE.MeshStandardMaterial({ color: 0x27272a, roughness: 0.75 });
            const bootMat = new THREE.MeshStandardMaterial({ color: 0x09090b, roughness: 0.65 });
            const skinMat = new THREE.MeshStandardMaterial({ color: 0xd6a57c, roughness: 0.6 });
            const visorMat = new THREE.MeshBasicMaterial({ color: 0x38bdf8 });
            const gunMainMat = new THREE.MeshStandardMaterial({ color: cfg.gunPrimary, metalness: cfg.metalness, roughness: 0.3 });
            const gunTrimMat = new THREE.MeshStandardMaterial({ color: cfg.gunAccent, metalness: 0.45, roughness: 0.4 });

            const leftLeg = new THREE.Mesh(new THREE.BoxGeometry(0.28, 0.58, 0.28), pantsMat);
            leftLeg.position.set(-0.17, 0.48, 0);
            const rightLeg = new THREE.Mesh(new THREE.BoxGeometry(0.28, 0.58, 0.28), pantsMat);
            rightLeg.position.set(0.17, 0.48, 0);

            const leftKnee = new THREE.Mesh(new THREE.BoxGeometry(0.3, 0.14, 0.31), gearMat);
            leftKnee.position.set(-0.17, 0.49, 0.02);
            const rightKnee = new THREE.Mesh(new THREE.BoxGeometry(0.3, 0.14, 0.31), gearMat);
            rightKnee.position.set(0.17, 0.49, 0.02);

            const leftBoot = new THREE.Mesh(new THREE.BoxGeometry(0.3, 0.2, 0.35), bootMat);
            leftBoot.position.set(-0.17, 0.1, 0.03);
            const rightBoot = new THREE.Mesh(new THREE.BoxGeometry(0.3, 0.2, 0.35), bootMat);
            rightBoot.position.set(0.17, 0.1, 0.03);

            const torso = new THREE.Mesh(new THREE.BoxGeometry(0.68, 0.72, 0.38), outfitMat);
            torso.position.set(0, 1.15, 0);
            const vest = new THREE.Mesh(new THREE.BoxGeometry(0.72, 0.56, 0.44), gearMat);
            vest.position.set(0, 1.17, 0);

            const pouchL = new THREE.Mesh(new THREE.BoxGeometry(0.16, 0.22, 0.1), outfitMat);
            pouchL.position.set(-0.16, 1.09, 0.24);
            const pouchR = new THREE.Mesh(new THREE.BoxGeometry(0.16, 0.22, 0.1), outfitMat);
            pouchR.position.set(0.16, 1.09, 0.24);
            const backpack = new THREE.Mesh(new THREE.BoxGeometry(0.5, 0.54, 0.22), gearMat);
            backpack.position.set(0, 1.19, -0.28);

            const leftArm = new THREE.Mesh(new THREE.BoxGeometry(0.24, 0.64, 0.24), outfitMat);
            leftArm.position.set(-0.48, 1.17, 0.04);
            leftArm.rotation.z = 0.06;

            const rightArm = new THREE.Mesh(new THREE.BoxGeometry(0.24, 0.64, 0.24), outfitMat);
            rightArm.position.set(0.48, 1.18, 0.14);
            rightArm.rotation.x = -0.5;

            const leftShoulder = new THREE.Mesh(new THREE.BoxGeometry(0.28, 0.22, 0.28), gearMat);
            leftShoulder.position.set(-0.49, 1.39, 0.02);
            const rightShoulder = new THREE.Mesh(new THREE.BoxGeometry(0.28, 0.22, 0.28), gearMat);
            rightShoulder.position.set(0.49, 1.39, 0.02);

            const head = new THREE.Mesh(new THREE.BoxGeometry(0.56, 0.56, 0.56), skinMat);
            head.position.set(0, 1.81, 0);
            const helmetTop = new THREE.Mesh(new THREE.BoxGeometry(0.62, 0.28, 0.62), gearMat);
            helmetTop.position.set(0, 1.99, -0.01);
            const helmetRim = new THREE.Mesh(new THREE.BoxGeometry(0.64, 0.08, 0.65), gearMat);
            helmetRim.position.set(0, 1.89, 0.02);

            const goggleFrame = new THREE.Mesh(new THREE.BoxGeometry(0.58, 0.2, 0.1), bootMat);
            goggleFrame.position.set(0, 1.82, 0.27);
            const visor = new THREE.Mesh(new THREE.BoxGeometry(0.48, 0.13, 0.04), visorMat);
            visor.position.set(0, 1.82, 0.31);

            const gunGroup = new THREE.Group();
            const gunBody = new THREE.Mesh(new THREE.BoxGeometry(0.11, 0.14, 0.62), gunMainMat);
            const gunBarrel = new THREE.Mesh(new THREE.CylinderGeometry(0.022, 0.022, 0.42, 8), gunMainMat);
            gunBarrel.rotation.x = Math.PI / 2;
            gunBarrel.position.set(0, 0.02, 0.45);
            const gunHandguard = new THREE.Mesh(new THREE.BoxGeometry(0.12, 0.11, 0.32), gunTrimMat);
            gunHandguard.position.set(0, 0.01, 0.24);
            const gunMag = new THREE.Mesh(new THREE.BoxGeometry(0.075, 0.24, 0.11), gunTrimMat);
            gunMag.position.set(0, -0.14, 0.05);
            gunMag.rotation.x = -0.2;
            gunGroup.add(gunBody, gunBarrel, gunHandguard, gunMag);
            gunGroup.position.set(0.46, 1.15, 0.36);

            lockerSleeveMeshes = [torso, pouchL, pouchR, leftArm, rightArm];
            lockerGloveMeshes = [vest, leftKnee, rightKnee, backpack, leftShoulder, rightShoulder, helmetTop, helmetRim];
            lockerGunPrimaryMeshes = [gunBody, gunBarrel];
            lockerGunAccentMeshes = [gunHandguard, gunMag];

            lockerCharGroup.add(
                leftLeg, rightLeg, leftKnee, rightKnee, leftBoot, rightBoot,
                torso, vest, pouchL, pouchR, backpack,
                leftArm, rightArm, leftShoulder, rightShoulder,
                head, helmetTop, helmetRim, goggleFrame, visor, gunGroup
            );
            lockerCharGroup.rotation.y = lockerTargetYaw;
            lockerScene.add(lockerCharGroup);

            canvas.addEventListener("mousedown", (e) => {
                lockerIsDragging = true;
                lockerPrevMouseX = e.clientX;
            });
            window.addEventListener("mousemove", (e) => {
                if (!lockerIsDragging) return;
                lockerTargetYaw += (e.clientX - lockerPrevMouseX) * 0.015;
                lockerPrevMouseX = e.clientX;
            });
            window.addEventListener("mouseup", () => {
                lockerIsDragging = false;
            });
            canvas.addEventListener("touchstart", (e) => {
                lockerIsDragging = true;
                lockerPrevMouseX = e.changedTouches[0].clientX;
            }, { passive: true });
            canvas.addEventListener("touchmove", (e) => {
                if (!lockerIsDragging) return;
                const x = e.changedTouches[0].clientX;
                lockerTargetYaw += (x - lockerPrevMouseX) * 0.018;
                lockerPrevMouseX = x;
            }, { passive: true });
            canvas.addEventListener("touchend", () => {
                lockerIsDragging = false;
            });

            resizeLockerPreview();
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
            lockerSleeveMeshes.forEach(m => m.material.color.setHex(cfg.sleeve));
            lockerGloveMeshes.forEach(m => m.material.color.setHex(cfg.glove));
            lockerGunPrimaryMeshes.forEach(m => {
                m.material.color.setHex(cfg.gunPrimary);
                m.material.metalness = cfg.metalness;
            });
            lockerGunAccentMeshes.forEach(m => m.material.color.setHex(cfg.gunAccent));
        }

        function renderLockerPreview(dt) {
            const lockerTab = document.getElementById("lobby-tab-locker");
            if (!inMainLobby || !lockerTab || lockerTab.classList.contains("hidden")) return;
            if (!lockerRenderer || !lockerScene || !lockerCamera) return;
            if (!lockerIsDragging) {
                lockerTargetYaw += dt * 0.85;
            }
            if (lockerCharGroup) {
                lockerCharGroup.rotation.y = lockerTargetYaw;
            }
            lockerRenderer.render(lockerScene, lockerCamera);
        }

        function switchLobbyTab(tabName) {
            const gpTab = document.getElementById("lobby-tab-gameplay");
            const lkTab = document.getElementById("lobby-tab-locker");
            const gpBtn = document.getElementById("tab-btn-gameplay");
            const lkBtn = document.getElementById("tab-btn-locker");

            if (tabName === "locker") {
                gpTab.classList.add("hidden");
                lkTab.classList.remove("hidden");
                lkBtn.className = "px-4 py-1.5 rounded-xl bg-lime-400 text-stone-950 font-black text-xs uppercase tracking-wider shadow transition";
                gpBtn.className = "px-4 py-1.5 rounded-xl text-stone-300 hover:text-white font-extrabold text-xs uppercase tracking-wider transition";
                requestAnimationFrame(() => resizeLockerPreview());
            } else {
                lkTab.classList.add("hidden");
                gpTab.classList.remove("hidden");
                gpBtn.className = "px-4 py-1.5 rounded-xl bg-lime-400 text-stone-950 font-black text-xs uppercase tracking-wider shadow transition";
                lkBtn.className = "px-4 py-1.5 rounded-xl text-stone-300 hover:text-white font-extrabold text-xs uppercase tracking-wider transition";
            }
        }

        function deployFromLobby() {
            initAudio();
            const rawName = document.getElementById("lobby-player-name").value.trim();
            player.name = rawName || "Survivor-01";
            applyEquippedSkinMaterials();
            inMainLobby = false;
            document.getElementById("main-lobby-overlay").classList.add("hidden");
            resetSoloMatch();
        }

        function returnToMainLobby() {
            inMainLobby = true;
            isPaused = true;
            isMouseDown = false;
            unlockLookControl();
            document.getElementById("pause-menu-overlay").classList.add("hidden");
            document.getElementById("pause-menu-overlay").classList.remove("flex");
            document.getElementById("match-end-overlay").classList.add("hidden");
            document.getElementById("match-end-overlay").classList.remove("flex");
            document.getElementById("main-lobby-overlay").classList.remove("hidden");
            switchLobbyTab("gameplay");
            if (window.mpRefreshLobbyRoster) window.mpRefreshLobbyRoster();
        }

        function resetSoloMatch() {
            buildSelectedMap(selectedMap);

            player.pos.set(0, 0, 0);
            player.vel.set(0, 0, 0);
            player.yaw = 0;
            player.pitch = 0;
            player.health = PLAYER_MAX_HEALTH;
            player.alive = true;
            player.kills = 0;
            player.medkits = 0;
            player.wood = buildEnabled ? 150 : 0;
            player.buildMode = false;
            player.buildPiece = "wall";
            player.buildRotationOffset = 0;
            if (blueprintViewModel) blueprintViewModel.visible = false;
            if (buildPreviewGroup) buildPreviewGroup.visible = false;
            cancelMedkitHeal();

            builtStructures.forEach(s => {
                if (s.mesh) {
                    s.mesh.visible = false;
                    scene.remove(s.mesh);
                }
            });
            builtStructures = [];
            mpRenderedBuilds.clear();

            player.weapons[0].owned = false;
            player.weapons[0].mag = 30;
            player.weapons[0].reserve = 150;

            player.weapons[1].owned = false;
            player.weapons[1].mag = 6;
            player.weapons[1].reserve = 36;

            player.weapons[2].owned = true;
            player.weapons[2].mag = 15;
            player.weapons[2].reserve = 90;

            player.weapons[3].owned = true;
            player.weapons[4].owned = false;
            player.weapons[4].mag = 0;

            switchWeaponSlot(2);
            spawnSupplyChests();
            spawnAllBots();

            document.getElementById("kill-feed").innerHTML = "";
            document.getElementById("match-end-overlay").classList.add("hidden");
            document.getElementById("match-end-overlay").classList.remove("flex");
            document.getElementById("pause-menu-overlay").classList.add("hidden");
            document.getElementById("pause-menu-overlay").classList.remove("flex");

            isPaused = false;
            updateHUD();
            const mapDisplay = selectedMap.charAt(0).toUpperCase() + selectedMap.slice(1);
            const modeDisplay = gameMode === "zombies" ? "Zombies" : (gameMode === "classic" ? "Classic BR" : "PvP Arena");
            addKillFeedEntry(`${player.name} deployed to ${mapDisplay} (${modeDisplay} • ${buildEnabled ? "Build" : "Zero Build"})!`, "text-lime-300");
            engageLookControl();
        }

        // =========================================================================
        // 12. MAIN ANIMATION & FIXED-STEP SIMULATION LOOP
        // =========================================================================
        function gameLoop() {
            requestAnimationFrame(gameLoop);

            const rawDt = Math.min(clock.getDelta(), 0.05);
            renderLockerPreview(rawDt);

            if (!isPaused && !inMainLobby) {
                const steps = simModeActive ? simSpeed : 1;
                for (let s = 0; s < steps; s++) {
                    updatePlayer(rawDt);
                    updateBots(rawDt);
                    updateEffects(rawDt);
                }
                if (mpActiveRoom && window.mpTickMultiplayer) {
                    window.mpTickMultiplayer(rawDt);
                }
                renderMinimap();
            }

            renderer.render(scene, camera);
        }

        window.onload = function () {
            initThreeScene();
            initLocker3DPreview();
            buildSelectedMap(selectedMap);
            spawnSupplyChests();
            spawnAllBots();
            setLobbyGameMode("zombies");
            setLobbyBuildMode(true);
            setLobbySkin("ranger");
            syncLobbyCallsignPreview();
            updateHUD();
            gameLoop();
        };
    </script>

    <!-- =========================================================================
         13. LIVE ONLINE MULTIPLAYER ENGINE (FIREBASE + CROSS-TAB FALLBACK)
         ========================================================================= -->
    <script type="module">
        import { initializeApp } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-app.js";
        import { getAuth, signInAnonymously, signInWithCustomToken, onAuthStateChanged } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-auth.js";
        import {
            getFirestore, doc, setDoc, getDoc, updateDoc, deleteDoc,
            collection, addDoc, onSnapshot, serverTimestamp
        } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-firestore.js";

        let db = null;
        let auth = null;
        let currentUser = null;
        let localChannel = null;
        const appId = typeof __app_id !== "undefined" ? __app_id : "deadshot-fps-default";
        const localTabId = "local-" + Math.random().toString(36).substring(2, 9).toUpperCase();
        const remoteMeshCache = {};

        function setMpFeedback(msg, isError = false) {
            const el = document.getElementById("mp-feedback-msg");
            if (!el) return;
            if (!msg) {
                el.classList.add("hidden");
                return;
            }
            el.textContent = msg;
            el.className = `mt-2 text-[11px] font-bold ${isError ? "text-red-400" : "text-amber-300"}`;
            el.classList.remove("hidden");
        }

        function createRemotePlayerMesh(skinKey) {
            const cfg = PLAYER_SKINS[skinKey] || PLAYER_SKINS.crimson;
            const root = new THREE.Group();
            const sleeveMat = new THREE.MeshStandardMaterial({ color: cfg.sleeve, roughness: 0.65 });
            const vestMat = new THREE.MeshStandardMaterial({ color: cfg.glove, roughness: 0.45, metalness: 0.25 });
            const pantsMat = new THREE.MeshStandardMaterial({ color: 0x27272a, roughness: 0.78 });
            const bootMat = new THREE.MeshStandardMaterial({ color: 0x09090b, roughness: 0.7 });
            const skinMat = new THREE.MeshStandardMaterial({ color: 0xd6a57c, roughness: 0.65 });
            const goggleFrameMat = new THREE.MeshStandardMaterial({ color: 0x18181b, roughness: 0.4 });
            const visorMat = new THREE.MeshBasicMaterial({ color: 0x38bdf8 });
            const gunMat = new THREE.MeshStandardMaterial({ color: cfg.gunPrimary, metalness: cfg.metalness, roughness: 0.3 });

            const leftLeg = new THREE.Mesh(new THREE.BoxGeometry(0.28, 0.58, 0.28), pantsMat);
            leftLeg.position.set(-0.17, 0.47, 0);
            const rightLeg = new THREE.Mesh(new THREE.BoxGeometry(0.28, 0.58, 0.28), pantsMat);
            rightLeg.position.set(0.17, 0.47, 0);

            const leftBoot = new THREE.Mesh(new THREE.BoxGeometry(0.3, 0.2, 0.35), bootMat);
            leftBoot.position.set(-0.17, 0.1, 0.03);
            const rightBoot = new THREE.Mesh(new THREE.BoxGeometry(0.3, 0.2, 0.35), bootMat);
            rightBoot.position.set(0.17, 0.1, 0.03);

            const torso = new THREE.Mesh(new THREE.BoxGeometry(0.68, 0.72, 0.38), sleeveMat);
            torso.position.set(0, 1.14, 0);
            const vest = new THREE.Mesh(new THREE.BoxGeometry(0.72, 0.56, 0.44), vestMat);
            vest.position.set(0, 1.16, 0);

            const leftShoulder = new THREE.Mesh(new THREE.BoxGeometry(0.28, 0.22, 0.28), vestMat);
            leftShoulder.position.set(-0.49, 1.38, 0.02);
            const rightShoulder = new THREE.Mesh(new THREE.BoxGeometry(0.28, 0.22, 0.28), vestMat);
            rightShoulder.position.set(0.49, 1.38, 0.02);

            const head = new THREE.Mesh(new THREE.BoxGeometry(0.56, 0.56, 0.56), skinMat);
            head.position.set(0, 1.8, 0);
            const helmetTop = new THREE.Mesh(new THREE.BoxGeometry(0.62, 0.28, 0.62), vestMat);
            helmetTop.position.set(0, 1.98, -0.01);
            const goggleFrame = new THREE.Mesh(new THREE.BoxGeometry(0.58, 0.2, 0.1), goggleFrameMat);
            goggleFrame.position.set(0, 1.81, -0.27);
            const visor = new THREE.Mesh(new THREE.BoxGeometry(0.48, 0.13, 0.04), visorMat);
            visor.position.set(0, 1.81, -0.31);

            const gun = new THREE.Mesh(new THREE.BoxGeometry(0.11, 0.14, 0.72), gunMat);
            gun.position.set(0.46, 1.16, -0.36);

            root.add(leftLeg, rightLeg, leftBoot, rightBoot, torso, vest, leftShoulder, rightShoulder, head, helmetTop, goggleFrame, visor, gun);
            scene.add(root);
            return { root, head, torso, leftLeg, rightLeg, vest, sleeveMat, vestMat };
        }

        function removeRemotePlayer(uid) {
            if (mpRemotePlayers[uid] && mpRemotePlayers[uid].mesh) {
                scene.remove(mpRemotePlayers[uid].mesh.root);
            }
            delete mpRemotePlayers[uid];
            refreshLobbyPartyList();
            updateHUD();
        }

        function clearAllRemotePlayers() {
            Object.keys(mpRemotePlayers).forEach(uid => removeRemotePlayer(uid));
        }

        function refreshLobbyPartyList() {
            const listEl = document.getElementById("mp-players-list");
            const headerEl = document.getElementById("mp-party-header");
            const countEl = document.getElementById("mp-party-count");
            if (!listEl) return;

            if (!mpActiveRoom) {
                listEl.classList.add("hidden");
                listEl.classList.remove("flex");
                if (headerEl) {
                    headerEl.classList.add("hidden");
                    headerEl.classList.remove("flex");
                }
                return;
            }

            const myName = (document.getElementById("lobby-player-name")?.value.trim()) || player.name || "Survivor-01";
            const remoteList = Object.entries(mpRemotePlayers).map(([uid, r]) => ({
                uid,
                name: (r.data && r.data.name) || "Player",
                skin: (r.data && r.data.skin) || "crimson",
                isSelf: false
            }));

            const allMembers = [
                { uid: getActiveUid(), name: myName, skin: selectedSkin, isSelf: true },
                ...remoteList
            ];

            if (headerEl && countEl) {
                headerEl.classList.remove("hidden");
                headerEl.classList.add("flex");
                countEl.textContent = `${allMembers.length} ${allMembers.length === 1 ? "PLAYER" : "PLAYERS"} IN PARTY`;
            }

            listEl.classList.remove("hidden");
            listEl.classList.add("flex");
            listEl.innerHTML = "";

            allMembers.forEach(m => {
                const skinCfg = PLAYER_SKINS[m.skin] || PLAYER_SKINS.ranger;
                const badge = document.createElement("span");
                badge.className = m.isSelf
                    ? "px-2.5 py-1 rounded-xl bg-lime-500/20 text-lime-200 border border-lime-400/50 font-black flex items-center gap-1.5"
                    : "px-2.5 py-1 rounded-xl bg-sky-500/20 text-sky-200 border border-sky-400/50 font-bold flex items-center gap-1.5";
                badge.innerHTML = `
                    <span class="w-2 h-2 rounded-full inline-block" style="background:${skinCfg.hudHex}"></span>
                    <span>${m.name}</span>
                    <span class="text-[9px] uppercase opacity-75">(${m.isSelf ? "YOU" : "SQUAD"})</span>
                `;
                listEl.appendChild(badge);
            });
        }

        function spawnRemoteBuild(bId, data) {
            if (mpRenderedBuilds.has(bId)) return;
            mpRenderedBuilds.add(bId);
            const plankMat = new THREE.MeshStandardMaterial({ color: 0xa16207, roughness: 0.78 });
            const group = new THREE.Group();
            group.rotation.y = data.rotY || 0;
            let structObj = null;
            const gx = data.gridX !== undefined ? data.gridX : Math.round(data.x / BUILD_GRID_SIZE) * BUILD_GRID_SIZE;
            const gz = data.gridZ !== undefined ? data.gridZ : Math.round(data.z / BUILD_GRID_SIZE) * BUILD_GRID_SIZE;

            if (data.type === "wall") {
                const wall = new THREE.Mesh(new THREE.BoxGeometry(BUILD_GRID_SIZE, BUILD_WALL_HEIGHT, 0.26), plankMat);
                group.add(wall);
                group.position.set(data.x, data.baseY + BUILD_WALL_HEIGHT / 2, data.z);
                const isZAligned = Math.abs(Math.sin(data.rotY || 0)) > 0.5;
                const col = addBoxCollider(
                    data.x, data.z,
                    isZAligned ? 0.35 : BUILD_GRID_SIZE / 2,
                    isZAligned ? BUILD_GRID_SIZE / 2 : 0.35,
                    data.baseY, data.baseY + BUILD_WALL_HEIGHT
                );
                structObj = { type: "wall", gridX: gx, gridZ: gz, x: data.x, z: data.z, baseY: data.baseY, y: data.baseY + BUILD_WALL_HEIGHT, rotY: data.rotY || 0, mesh: group, plankMesh: wall, collider: col, health: BUILD_MAX_HEALTH, maxHealth: BUILD_MAX_HEALTH, destroyed: false };
            } else if (data.type === "floor") {
                const floor = new THREE.Mesh(new THREE.BoxGeometry(BUILD_GRID_SIZE, 0.26, BUILD_GRID_SIZE), plankMat);
                group.add(floor);
                const topY = data.baseY + 0.26;
                group.position.set(data.x, data.baseY + 0.13, data.z);
                const col = addBoxCollider(
                    data.x, data.z,
                    BUILD_GRID_SIZE / 2,
                    BUILD_GRID_SIZE / 2,
                    data.baseY,
                    topY
                );
                structObj = { type: "floor", gridX: gx, gridZ: gz, x: data.x, z: data.z, baseY: data.baseY, y: topY, rotY: data.rotY || 0, mesh: group, plankMesh: floor, collider: col, health: BUILD_MAX_HEALTH, maxHealth: BUILD_MAX_HEALTH, destroyed: false };
            } else {
                const slopeLen = Math.hypot(BUILD_GRID_SIZE, BUILD_WALL_HEIGHT);
                const ramp = new THREE.Mesh(new THREE.BoxGeometry(BUILD_GRID_SIZE, 0.26, slopeLen), plankMat);
                ramp.rotation.x = -Math.atan2(BUILD_WALL_HEIGHT, BUILD_GRID_SIZE);
                group.add(ramp);
                group.position.set(data.x, data.baseY + BUILD_WALL_HEIGHT / 2, data.z);
                structObj = { type: "ramp", gridX: gx, gridZ: gz, x: data.x, z: data.z, baseY: data.baseY, y: data.baseY + BUILD_WALL_HEIGHT, rotY: data.rotY || 0, mesh: group, plankMesh: ramp, collider: null, health: BUILD_MAX_HEALTH, maxHealth: BUILD_MAX_HEALTH, destroyed: false };
            }
            if (structObj) {
                structObj.plankMesh.userData.buildStruct = structObj;
                builtStructures.push(structObj);
            }
            scene.add(group);
        }

        function getActiveUid() {
            return currentUser ? currentUser.uid : localTabId;
        }

        function getRoomDocRef(roomCode) {
            return doc(db, "artifacts", appId, "public", "data", "deadshot_rooms", roomCode);
        }

        function getPlayersCollRef(roomCode) {
            return collection(db, "artifacts", appId, "public", "data", "deadshot_players");
        }

        function getEventsCollRef() {
            return collection(db, "artifacts", appId, "public", "data", "deadshot_events");
        }

        function getBuildsCollRef() {
            return collection(db, "artifacts", appId, "public", "data", "deadshot_builds");
        }

        async function attachRoomListeners(roomCode) {
            detachRoomListeners();
            mpActiveRoom = roomCode;
            mpProcessedEvents.clear();

            document.getElementById("mp-room-badge").textContent = `ROOM: ${roomCode}`;
            document.getElementById("mp-room-badge").className = "mono text-[10px] font-black px-2 py-0.5 rounded bg-sky-500/30 text-sky-200 border border-sky-400/50";
            document.getElementById("mp-leave-btn").classList.remove("hidden");

            const myUid = getActiveUid();

            if (db && currentUser) {
                mpUnsubRoom = onSnapshot(getRoomDocRef(roomCode), (snap) => {
                    if (!snap.exists()) return;
                    const rData = snap.data();
                    if (rData.gameMode && rData.gameMode !== gameMode) setLobbyGameMode(rData.gameMode);
                    if (typeof rData.buildEnabled === "boolean" && rData.buildEnabled !== buildEnabled) setLobbyBuildMode(rData.buildEnabled);
                    if (rData.map && rData.map !== selectedMap) setLobbyMap(rData.map);
                    if (rData.difficulty && rData.difficulty !== lobbyDifficulty) setLobbyDifficulty(rData.difficulty);
                }, (err) => console.warn("Room snapshot error:", err));

                mpUnsubPlayers = onSnapshot(getPlayersCollRef(roomCode), (snap) => {
                    const activeUidsInRoom = new Set();
                    snap.forEach(docSnap => {
                        const pData = docSnap.data();
                        if (pData.roomCode !== roomCode) return;
                        const uid = docSnap.id;
                        if (uid === myUid) return;
                        activeUidsInRoom.add(uid);

                        if (!mpRemotePlayers[uid]) {
                            const mesh = createRemotePlayerMesh(pData.skin || "crimson");
                            mesh.root.position.set(pData.x || 0, pData.y || 0, pData.z || 0);
                            mpRemotePlayers[uid] = {
                                data: pData,
                                mesh,
                                targetPos: new THREE.Vector3(pData.x || 0, pData.y || 0, pData.z || 0)
                            };
                            addKillFeedEntry(`${pData.name || "Player"} joined Room ${roomCode}`, "text-sky-300");
                        } else {
                            mpRemotePlayers[uid].data = pData;
                            mpRemotePlayers[uid].targetPos.set(pData.x || 0, pData.y || 0, pData.z || 0);
                            mpRemotePlayers[uid].mesh.root.visible = !!pData.alive;
                        }
                    });

                    Object.keys(mpRemotePlayers).forEach(existingUid => {
                        if (!activeUidsInRoom.has(existingUid)) {
                            removeRemotePlayer(existingUid);
                        }
                    });

                    refreshLobbyPartyList();
                    updateHUD();
                }, (err) => console.warn("Players snapshot error:", err));

                mpUnsubEvents = onSnapshot(getEventsCollRef(), (snap) => {
                    snap.docChanges().forEach(change => {
                        if (change.type !== "added") return;
                        const evId = change.doc.id;
                        if (mpProcessedEvents.has(evId)) return;
                        mpProcessedEvents.add(evId);

                        const ev = change.doc.data();
                        if (ev.roomCode !== roomCode) return;
                        if (ev.targetUid === myUid && ev.type === "damage") {
                            applyDamageToPlayer(ev.amount || 15, ev.attackerName || "Enemy Player", !!ev.isHead);
                        }
                    });
                }, (err) => console.warn("Events snapshot error:", err));

                mpUnsubBuilds = onSnapshot(getBuildsCollRef(), (snap) => {
                    snap.docChanges().forEach(change => {
                        if (change.type !== "added") return;
                        const bData = change.doc.data();
                        if (bData.roomCode !== roomCode || bData.ownerUid === myUid) return;
                        spawnRemoteBuild(change.doc.id, bData);
                    });
                }, (err) => console.warn("Builds snapshot error:", err));
            }

            await pushMyPlayerState();
            refreshLobbyPartyList();
        }

        function detachRoomListeners() {
            if (mpUnsubRoom) { mpUnsubRoom(); mpUnsubRoom = null; }
            if (mpUnsubPlayers) { mpUnsubPlayers(); mpUnsubPlayers = null; }
            if (mpUnsubEvents) { mpUnsubEvents(); mpUnsubEvents = null; }
            if (mpUnsubBuilds) { mpUnsubBuilds(); mpUnsubBuilds = null; }
        }

        async function pushMyPlayerState() {
            if (!mpActiveRoom) return;
            const myUid = getActiveUid();
            const currentName = (document.getElementById("lobby-player-name")?.value.trim()) || player.name || "Survivor-01";
            player.name = currentName;

            const payload = {
                roomCode: mpActiveRoom,
                uid: myUid,
                name: currentName,
                skin: selectedSkin,
                x: Number(player.pos.x.toFixed(2)),
                y: Number(player.pos.y.toFixed(2)),
                z: Number(player.pos.z.toFixed(2)),
                yaw: Number(player.yaw.toFixed(3)),
                health: player.health,
                alive: player.alive,
                kills: player.kills,
                inLobby: inMainLobby,
                updatedAt: Date.now()
            };

            if (db && currentUser) {
                try {
                    await setDoc(doc(db, "artifacts", appId, "public", "data", "deadshot_players", myUid), payload, { merge: true });
                } catch (e) {}
            }
            if (localChannel) {
                localChannel.postMessage({ kind: "player_sync", payload });
            }
        }

        window.createMultiplayerRoom = async function () {
            const code = Math.random().toString(36).substring(2, 6).toUpperCase();
            const roomInput = document.getElementById("mp-room-input");
            if (roomInput) roomInput.value = code;

            if (db && currentUser) {
                try {
                    await setDoc(getRoomDocRef(code), {
                        roomCode: code,
                        hostUid: currentUser.uid,
                        gameMode,
                        buildEnabled,
                        map: selectedMap,
                        difficulty: lobbyDifficulty,
                        createdAt: serverTimestamp()
                    });
                } catch (e) {
                    setMpFeedback("Could not create cloud room: " + e.message, true);
                    return;
                }
            }
            await attachRoomListeners(code);
            if (localChannel) {
                localChannel.postMessage({ kind: "room_announce", roomCode: code, gameMode, buildEnabled, map: selectedMap, difficulty: lobbyDifficulty });
            }
            setMpFeedback(`Room ${code} hosted! Share code "${code}" with friends to join.`, false);
        };

        window.joinMultiplayerRoom = async function () {
            const rawCode = (document.getElementById("mp-room-input")?.value || "").trim().toUpperCase();
            if (!rawCode) {
                setMpFeedback("Enter a 4-character Room Code first.", true);
                return;
            }

            if (db && currentUser) {
                try {
                    const snap = await getDoc(getRoomDocRef(rawCode));
                    if (!snap.exists()) {
                        setMpFeedback(`Room "${rawCode}" not found. Check the code or click Host Room.`, true);
                        return;
                    }
                    const rData = snap.data();
                    if (rData.gameMode) setLobbyGameMode(rData.gameMode);
                    if (typeof rData.buildEnabled === "boolean") setLobbyBuildMode(rData.buildEnabled);
                    if (rData.map) setLobbyMap(rData.map);
                    if (rData.difficulty) setLobbyDifficulty(rData.difficulty);
                } catch (e) {
                    setMpFeedback("Failed to join room: " + e.message, true);
                    return;
                }
            }
            await attachRoomListeners(rawCode);
            if (localChannel) {
                localChannel.postMessage({ kind: "player_join_req", roomCode: rawCode });
            }
            setMpFeedback(`Joined Room ${rawCode}! Ready to deploy together.`, false);
        };

        window.leaveMultiplayerRoom = async function () {
            const oldRoom = mpActiveRoom;
            const myUid = getActiveUid();
            detachRoomListeners();
            mpActiveRoom = null;
            clearAllRemotePlayers();

            document.getElementById("mp-room-badge").textContent = "SOLO / OFFLINE";
            document.getElementById("mp-room-badge").className = "mono text-[10px] font-black px-2 py-0.5 rounded bg-stone-800 text-stone-300";
            document.getElementById("mp-leave-btn").classList.add("hidden");
            setMpFeedback(oldRoom ? `Left Room ${oldRoom}.` : "");
            refreshLobbyPartyList();

            if (db && currentUser && oldRoom) {
                try {
                    await deleteDoc(doc(db, "artifacts", appId, "public", "data", "deadshot_players", myUid));
                } catch (e) {}
            }
            if (localChannel && oldRoom) {
                localChannel.postMessage({ kind: "player_leave", roomCode: oldRoom, uid: myUid });
            }
        };

        window.mpSendDamage = async function (targetUid, targetName, amount, isHead) {
            if (!mpActiveRoom) return;
            const evPayload = {
                roomCode: mpActiveRoom,
                type: "damage",
                attackerUid: getActiveUid(),
                attackerName: player.name,
                targetUid,
                targetName,
                amount,
                isHead,
                createdAt: Date.now()
            };
            if (db && currentUser) {
                try {
                    await addDoc(getEventsCollRef(), evPayload);
                } catch (e) {}
            }
            if (localChannel) {
                localChannel.postMessage({ kind: "event_sync", payload: evPayload });
            }
        };

        window.mpPublishBuild = async function (buildData) {
            if (!mpActiveRoom) return;
            const bPayload = {
                roomCode: mpActiveRoom,
                ownerUid: getActiveUid(),
                type: buildData.type,
                gridX: buildData.gridX,
                gridZ: buildData.gridZ,
                x: buildData.x,
                z: buildData.z,
                baseY: buildData.baseY,
                rotY: buildData.rotY,
                createdAt: Date.now()
            };
            if (db && currentUser) {
                try {
                    await addDoc(getBuildsCollRef(), bPayload);
                } catch (e) {}
            }
            if (localChannel) {
                localChannel.postMessage({ kind: "build_sync", buildId: `${getActiveUid()}_${Date.now()}_${Math.random()}`, payload: bPayload });
            }
        };

        window.mpRefreshLobbyRoster = refreshLobbyPartyList;

        window.mpTickMultiplayer = function (dt) {
            if (!mpActiveRoom) return;
            mpSyncAccumulator += dt;
            if (mpSyncAccumulator >= 0.08) {
                mpSyncAccumulator = 0;
                pushMyPlayerState();
            }
            Object.values(mpRemotePlayers).forEach(rp => {
                if (!rp || !rp.mesh || !rp.targetPos) return;
                rp.mesh.root.position.lerp(rp.targetPos, Math.min(1, dt * 14));
                if (rp.data && typeof rp.data.yaw === "number") {
                    rp.mesh.root.rotation.y = rp.data.yaw;
                }
            });
        };

        function setupLocalBroadcastChannel() {
            if (typeof BroadcastChannel === "undefined") return;
            localChannel = new BroadcastChannel("deadshot_mp_channel");
            localChannel.onmessage = (event) => {
                const msg = event.data;
                if (!msg || !mpActiveRoom) return;
                const myUid = getActiveUid();

                if (msg.kind === "player_sync" && msg.payload && msg.payload.roomCode === mpActiveRoom) {
                    const p = msg.payload;
                    if (p.uid === myUid) return;
                    if (!mpRemotePlayers[p.uid]) {
                        const mesh = createRemotePlayerMesh(p.skin || "crimson");
                        mesh.root.position.set(p.x || 0, p.y || 0, p.z || 0);
                        mpRemotePlayers[p.uid] = { data: p, mesh, targetPos: new THREE.Vector3(p.x || 0, p.y || 0, p.z || 0) };
                        addKillFeedEntry(`${p.name} joined Room ${mpActiveRoom}`, "text-sky-300");
                    } else {
                        mpRemotePlayers[p.uid].data = p;
                        mpRemotePlayers[p.uid].targetPos.set(p.x || 0, p.y || 0, p.z || 0);
                        mpRemotePlayers[p.uid].mesh.root.visible = !!p.alive;
                    }
                    refreshLobbyPartyList();
                    updateHUD();
                } else if (msg.kind === "player_join_req" && msg.roomCode === mpActiveRoom) {
                    pushMyPlayerState();
                } else if (msg.kind === "player_leave" && msg.roomCode === mpActiveRoom) {
                    removeRemotePlayer(msg.uid);
                } else if (msg.kind === "event_sync" && msg.payload && msg.payload.roomCode === mpActiveRoom) {
                    const ev = msg.payload;
                    if (ev.targetUid === myUid && ev.type === "damage") {
                        applyDamageToPlayer(ev.amount || 15, ev.attackerName || "Enemy Player", !!ev.isHead);
                    }
                } else if (msg.kind === "build_sync" && msg.payload && msg.payload.roomCode === mpActiveRoom) {
                    if (msg.payload.ownerUid !== myUid) {
                        spawnRemoteBuild(msg.buildId, msg.payload);
                    }
                }
            };
        }

        async function initMultiplayerService() {
            setupLocalBroadcastChannel();
            const statusDot = document.getElementById("mp-status-dot");
            const connBadge = document.getElementById("mp-conn-mode");
            const idEl = document.getElementById("mp-user-id");

            try {
                if (typeof __firebase_config !== "undefined" && __firebase_config) {
                    const firebaseConfig = JSON.parse(__firebase_config);
                    const app = initializeApp(firebaseConfig);
                    auth = getAuth(app);
                    db = getFirestore(app);

                    if (typeof __initial_auth_token !== "undefined" && __initial_auth_token) {
                        await signInWithCustomToken(auth, __initial_auth_token);
                    } else {
                        await signInAnonymously(auth);
                    }

                    onAuthStateChanged(auth, (user) => {
                        currentUser = user;
                        if (user) {
                            if (statusDot) statusDot.className = "w-2 h-2 rounded-full bg-lime-400";
                            if (connBadge) {
                                connBadge.textContent = "CLOUD ONLINE";
                                connBadge.className = "mono text-[9px] font-black px-2 py-0.5 rounded bg-lime-500/20 text-lime-300 border border-lime-400/40";
                            }
                            if (idEl) idEl.textContent = user.uid;
                        }
                    });
                    return;
                }
            } catch (e) {
                console.warn("Using local BroadcastChannel multiplayer fallback:", e);
            }

            if (statusDot) statusDot.className = "w-2 h-2 rounded-full bg-sky-400";
            if (connBadge) {
                connBadge.textContent = "LOCAL TAB ONLY";
                connBadge.className = "mono text-[9px] font-black px-2 py-0.5 rounded bg-sky-500/20 text-sky-300 border border-sky-400/40";
            }
            if (idEl) idEl.textContent = localTabId;
        }

        initMultiplayerService();
    </script>
</body>
</html>
