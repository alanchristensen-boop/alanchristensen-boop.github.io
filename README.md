]<!-- ... existing code ... -->
                    <div class="flex items-center gap-2">
                        <span class="w-2.5 h-2.5 rounded-full bg-lime-400 animate-pulse"></span>
                        <span id="hud-mode-badge" class="text-xs font-extrabold uppercase tracking-wider text-lime-300">Woodland BR (20)</span>
                    </div>
                    <span id="hud-lobby-tag" class="mono text-xs text-stone-400">SOLO</span>
                </div>
<!-- ... existing code ... -->
                <!-- Primary Action: Resume or Deploy Solo -->
                <div class="grid grid-cols-1 sm:grid-cols-2 gap-3">
                    <button id="btn-resume" class="w-full py-3.5 px-5 rounded-2xl bg-gradient-to-r from-lime-500 to-emerald-600 hover:from-lime-400 hover:to-emerald-500 text-stone-950 font-extrabold text-sm tracking-wide shadow-lg shadow-lime-500/20 transition transform active:scale-98 cursor-pointer">
                        RESUME BATTLE
                    </button>
                    <button id="btn-deploy-solo" class="w-full py-3.5 px-5 rounded-2xl bg-stone-800 hover:bg-stone-700 border border-stone-600 text-white font-bold text-sm tracking-wide transition cursor-pointer">
                        NEW SOLO MATCH (VS 19 BOTS)
                    </button>
                </div>
<!-- ... existing code ... -->
        // ==========================================
        // 1. CONFIGURATION & WEAPON DEFINITIONS
        // ==========================================
        const CONFIG = {
            ARENA_SIZE: 390,
            PLAYER_HEIGHT: 1.72,
            PLAYER_RADIUS: 0.45,
            PLAYER_MAX_HEALTH: 200,
            BOT_MAX_HEALTH: 200,
            HEADSHOT_DAMAGE: 25,
            BODYSHOT_DAMAGE: 15,
            WALK_SPEED: 7.2,
            SPRINT_SPEED: 10.2,
            JUMP_FORCE: 8.6,
            GRAVITY: 22.0,
            TOTAL_COMBATANTS: 20, // Player + 19 bots
            BOT_COUNT: 19,
            RESPAWN_DELAY: 3.0
        };
// ... existing code ...
        const BOT_CALLSIGNS = [
            'Sgt. Miller', 'Cpl. Vance', 'Lt. Reyes',
            'Pvt. Mercer', 'Spc. Novak', 'Sgt. Kael',
            'Cpl. Thorne', 'Pvt. Bishop', 'Lt. Orlov',
            'Sgt. Hayes', 'Cpl. Diaz', 'Pvt. Sterling',
            'Spc. Frost', 'Lt. Archer', 'Sgt. Boone',
            'Cpl. Park', 'Pvt. Walker', 'Spc. Cruz',
            'Lt. Kane'
        ];
// ... existing code ...
        function initThree() {
            scene = new THREE.Scene();
            scene.background = new THREE.Color(0x6e8b74);
            scene.fog = new THREE.FogExp2(0x6e8b74, 0.0055);

            camera = new THREE.PerspectiveCamera(75, window.innerWidth / window.innerHeight, 0.1, 480);
            camera.position.set(0, CONFIG.PLAYER_HEIGHT, 12);
            scene.add(camera);
// ... existing code ...
            dirLight.shadow.mapSize.width = 2048;
            dirLight.shadow.mapSize.height = 2048;
            const d = 195;
            dirLight.shadow.camera.left = -d;
            dirLight.shadow.camera.right = d;
            dirLight.shadow.camera.top = d;
            dirLight.shadow.camera.bottom = -d;
            scene.add(dirLight);
        }
// ... existing code ...
            const tex = new THREE.CanvasTexture(canvas);
            tex.wrapS = THREE.RepeatWrapping;
            tex.wrapT = THREE.RepeatWrapping;
            tex.repeat.set(84, 84);
            return tex;
        }
// ... existing code ...
            const structures = [
                // Central Outpost Cover Walls
                { x: -5.5, z: 0, w: 1.4, d: 8, h: 2.6, mat: bunkerMat },
                { x: 5.5, z: 0, w: 1.4, d: 8, h: 2.6, mat: bunkerMat },
                { x: 0, z: -6.5, w: 7, d: 1.4, h: 2.2, mat: bunkerMat },
                // Four Corner Timber Field Cabins (scaled to 3x map)
                { x: -84, z: -78, w: 9, d: 8, h: 4.0, mat: timberMat },
                { x: 84, z: 78, w: 9, d: 8, h: 4.0, mat: timberMat },
                { x: -78, z: 78, w: 8, d: 9, h: 4.0, mat: timberMat },
                { x: 78, z: -78, w: 8, d: 9, h: 4.0, mat: timberMat },
                // Mid-field Forward Outposts
                { x: -135, z: 0, w: 8, d: 8, h: 3.8, mat: bunkerMat },
                { x: 135, z: 0, w: 8, d: 8, h: 3.8, mat: bunkerMat },
                { x: 0, z: -135, w: 8, d: 8, h: 3.8, mat: bunkerMat },
                { x: 0, z: 135, w: 8, d: 8, h: 3.8, mat: bunkerMat }
            ];
// ... existing code ...
            const trunkMat = new THREE.MeshStandardMaterial({ color: 0x4a321f, roughness: 0.9 });
            const foliageMat = new THREE.MeshStandardMaterial({ color: 0x223d23, roughness: 0.85 });

            // 260 Multi-Tiered Pine Trees across the 3x Woodland Map
            for (let i = 0; i < 260; i++) {
                const tx = (rand() - 0.5) * (size - 20);
                const tz = (rand() - 0.5) * (size - 20);
                if (Math.hypot(tx, tz) < 14) continue;
// ... existing code ...
            // 110 Mossy Cover Boulders across the expanded woodland
            for (let i = 0; i < 110; i++) {
                const rx = (rand() - 0.5) * (size - 22);
                const rz = (rand() - 0.5) * (size - 22);
                if (Math.hypot(rx, rz) < 12) continue;
// ... existing code ...
        function spawnLootChests() {
            lootChests.forEach(c => scene.remove(c.mesh));
            lootChests = [];

            const chestCoords = [
                { x: 0, z: -3.5 },      // Central bunker starter chest
                { x: -24, z: -20 },
                { x: 24, z: 20 },
                { x: -84, z: -72 },     // NW Cabin
                { x: 84, z: 72 },       // SE Cabin
                { x: -78, z: 72 },      // SW Cabin
                { x: 78, z: -72 },      // NE Cabin
                { x: -135, z: 6 },      // West Outpost
                { x: 135, z: -6 },      // East Outpost
                { x: 6, z: -135 },      // North Outpost
                { x: -6, z: 135 },      // South Outpost
                { x: -55, z: 0 },
                { x: 55, z: 0 },
                { x: 0, z: -55 },
                { x: 0, z: 55 },
                { x: -120, z: -115 },
                { x: 120, z: 115 },
                { x: -120, z: 115 },
                { x: 120, z: -115 },
                { x: -165, z: -65 },
                { x: 165, z: 65 },
                { x: -65, z: 165 },
                { x: 65, z: -165 }
            ];
// ... existing code ...
            takeDamage(amount, attackerName, isPlayerAttacker = false) {
                if (this.dead) return;
                this.health -= amount;
                this.updateHealthBar();

                if (this.health <= 0) {
                    this.health = 0;
                    this.dead = true;
                    this.deaths++;
                    this.mesh.visible = false;

                    if (isPlayerAttacker) {
                        playerKills++;
                    }
                    addKillfeedEntry(attackerName, this.name);
                    updateScoreboardHUD();
                }
            }

            update(dt) {
                // Eliminated bots remain permanently dead (no respawning)
                if (this.dead) {
                    return;
                }

                // Billboard health bar toward player camera
// ... existing code ...
        function updateScoreboardHUD() {
            const aliveBots = bots.filter(b => !b.dead).length;
            const aliveRemotes = Object.values(remotePlayers).filter(rp => !rp.dead).length;
            const totalAlive = aliveBots + aliveRemotes + (playerDead ? 0 : 1);
            document.getElementById('hud-mode-badge').textContent = `Woodland BR (${totalAlive} Alive)`;

            const entries = [
                { name: `${playerCallsign} (You)`, kills: playerKills, isYou: true, dead: playerDead },
                ...Object.values(remotePlayers).map(rp => ({ name: `${rp.name} [LIVE]`, kills: rp.kills, isYou: false, dead: rp.dead })),
                ...bots.map(b => ({ name: b.name, kills: b.kills, isYou: false, dead: b.dead }))
            ];
            entries.sort((a, b) => (a.dead === b.dead ? b.kills - a.kills : a.dead ? 1 : -1));

            const list = document.getElementById('scoreboard-list');
            list.innerHTML = entries.map((e, i) => `
                <div class="flex justify-between items-center px-2 py-0.5 rounded ${e.dead ? 'opacity-40 line-through text-stone-500' : e.isYou ? 'bg-lime-500/20 text-lime-300 font-bold' : 'text-stone-300'}">
                    <span class="truncate max-w-[165px]">#${i + 1} ${e.name}</span>
                    <span>${e.dead ? 'OUT' : `${e.kills} K`}</span>
                </div>
            `).join('');
        }
// ... existing code ...
            // Update HUD mode badges
            document.getElementById('hud-mode-badge').textContent = mode === 'lobby' ? 'Private Squad BR' : 'Woodland BR (20 Alive)';
            document.getElementById('hud-lobby-tag').textContent = (mode === 'lobby' && currentLobby) ? `CODE: ${currentLobby.code}` : 'SOLO';
            updateScoreboardHUD();
// ... existing code ...
