<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>NEMESIS - Fully Loaded Edition</title>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/three.js/r128/three.min.js"></script>
    <style>
        * {
            box-sizing: border-box;
            user-select: none;
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
        }
        body, html {
            margin: 0;
            padding: 0;
            width: 100%;
            height: 100%;
            overflow: hidden;
            background-color: #000;
        }

        /* 3D Viewport Layer */
        #canvas-container {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            z-index: 1;
        }

        /* General UI Control Overlay */
        #ui-layer {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            z-index: 10;
            pointer-events: none;
        }
        
        .interactive {
            pointer-events: auto;
        }

        /* TOP CORNER HUD INFO (Always Visible outside active combat maps) */
        #currency-hud {
            position: absolute;
            top: 15px;
            right: 15px;
            background: rgba(13, 27, 62, 0.85);
            border: 2px solid #00d2ff;
            border-radius: 12px;
            padding: 8px 16px;
            color: #fff;
            font-weight: bold;
            font-size: 18px;
            z-index: 100;
            display: flex;
            gap: 15px;
            box-shadow: 0 4px 10px rgba(0,0,0,0.5);
        }
        .coin-count { color: #ffd700; }
        .strength-count { color: #ff3366; }

        /* SCREEN 1 & 2: MAIN MENU & SIDEBAR SYSTEM */
        #main-menu-screen {
            position: absolute;
            width: 100%;
            height: 100%;
            background: orange url('Schermafbeelding 2026-06-09 215210.png') no-repeat center center;
            background-size: cover;
            display: flex;
            justify-content: center;
            align-items: center;
        }

        .sidebar-toggle {
            position: absolute;
            top: 10px;
            left: 10px;
            background: linear-gradient(to bottom, #111a40, #05081a);
            color: white;
            border: 1px solid #222;
            padding: 8px 16px;
            font-weight: bold;
            cursor: pointer;
            z-index: 20;
            font-size: 14px;
        }

        .play-btn {
            position: absolute;
            bottom: 20%;
            background: linear-gradient(to right, #43e0ff, #0044ff);
            color: white;
            border: none;
            padding: 15px 50px;
            font-size: 28px;
            font-weight: bold;
            border-radius: 8px;
            cursor: pointer;
            box-shadow: 0 4px 15px rgba(0,68,255,0.4);
            transition: transform 0.1s ease;
        }
        .play-btn:hover { transform: scale(1.05); }

        #sidebar {
            position: absolute;
            top: 0;
            left: -320px;
            width: 320px;
            height: 100%;
            background: linear-gradient(to bottom, #7b94ff, #4fe3ff);
            transition: left 0.3s cubic-bezier(0.1, 0.7, 0.1, 1);
            display: flex;
            flex-direction: column;
            padding: 80px 20px 20px 20px;
            gap: 25px;
            z-index: 15;
            box-shadow: 5px 0 15px rgba(0,0,0,0.3);
        }
        #sidebar.open { left: 0; }

        .menu-item {
            display: flex;
            align-items: center;
            gap: 15px;
            color: #1a237e;
            font-size: 24px;
            font-weight: bold;
            cursor: pointer;
            padding: 10px;
            border-radius: 8px;
            transition: background 0.2s;
        }
        .menu-item:hover { background: rgba(255,255,255,0.2); }
        .menu-icon { width: 40px; height: 40px; text-align: center; }

        /* MODAL SCREEN BASE HOOKS */
        .modal-screen {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            display: none;
            justify-content: center;
            align-items: center;
            z-index: 25;
        }

        /* SCREEN 4: WEAPONS */
        #weapons-screen {
            background: linear-gradient(to bottom, #f7d09c, #593525);
        }
        .weapons-box {
            width: 500px;
            height: 430px;
            background: #888;
            border-radius: 20px;
            padding: 20px;
            box-shadow: inset 0 0 20px rgba(0,0,0,0.4);
            display: flex;
            flex-direction: column;
        }
        .search-bar {
            width: 100%;
            padding: 12px;
            border-radius: 10px;
            border: none;
            background: linear-gradient(to bottom, #fff, #bbb);
            font-size: 18px;
            font-weight: bold;
            color: #444;
            margin-bottom: 15px;
        }
        .weapon-inv-list {
            flex-grow: 1;
            overflow-y: auto;
            background: rgba(0,0,0,0.2);
            border-radius: 10px;
            padding: 10px;
        }
        .inv-item-row {
            background: #555;
            padding: 10px;
            margin-bottom: 8px;
            border-radius: 8px;
            display: flex;
            justify-content: space-between;
            color: white;
            font-weight: bold;
            align-items: center;
        }
        .equip-btn {
            background: #4caf50;
            border: none;
            color: white;
            padding: 5px 10px;
            border-radius: 5px;
            cursor: pointer;
        }

        /* SCREEN 5: PROFILE */
        #profile-screen {
            background: #e0e0e0 url('Schermafbeelding 2026-06-09 223259.png') no-repeat center center;
            background-size: contain;
        }

        /* SCREEN 6: SETTINGS */
        #settings-screen {
            background: linear-gradient(135deg, #110033, #ff99bb);
            flex-direction: column;
        }
        .settings-container {
            background: linear-gradient(to right, #e033ff, #00f0ff);
            width: 85%;
            max-width: 700px;
            border-radius: 20px;
            padding: 30px;
            color: white;
        }
        .setting-row {
            display: flex;
            justify-content: space-between;
            align-items: center;
            margin-bottom: 20px;
            font-size: 22px;
            font-weight: bold;
        }
        .toggle-switch {
            background: #4caf50;
            color: white;
            padding: 6px 18px;
            border-radius: 15px;
            font-size: 16px;
            cursor: pointer;
            border: 2px solid white;
        }

        /* SCREEN 3: SHOP (Enriched Grid) */
        #shop-screen {
            background: linear-gradient(135deg, #020212, #201554, #8a2be2);
        }
        .shop-container {
            width: 90%;
            max-width: 850px;
            height: 80%;
            display: flex;
            flex-direction: column;
            justify-content: space-between;
        }
        .shop-header {
            display: flex;
            justify-content: space-between;
            align-items: center;
            color: #00d2ff;
            font-size: 32px;
            font-weight: bold;
        }
        .shop-carousel {
            background: linear-gradient(to right, #0044ff, #b033ff);
            height: 65%;
            border-radius: 25px;
            display: flex;
            padding: 20px;
            gap: 20px;
            overflow-x: auto;
            align-items: center;
            border-bottom: 15px solid #444;
        }
        .shop-item {
            min-width: 170px;
            height: 100%;
            background: #0d1b3e;
            border-radius: 15px;
            border: 3px solid #1e3c72;
            cursor: pointer;
            transition: transform 0.2s, border-color 0.2s;
            display: flex;
            flex-direction: column;
            justify-content: space-between;
            padding: 15px;
            color: white;
            text-align: center;
        }
        .shop-item:hover { transform: scale(1.05); border-color: #00ffff; }
        .item-title { font-weight: bold; font-size: 18px; color: #fff; }
        .item-bonus { font-size: 13px; color: #00ffcc; margin-top: 5px; }
        .item-cost { background: rgba(0,0,0,0.4); padding: 5px; border-radius: 8px; font-weight: bold; color: #ffd700; }

        /* Universal Navigation Controls */
        .back-btn {
            position: absolute;
            bottom: 30px;
            left: 30px;
            background: #ff2a6d;
            color: white;
            border: none;
            padding: 10px 25px;
            font-size: 18px;
            font-weight: bold;
            border-radius: 5px;
            cursor: pointer;
            z-index: 30;
        }

        /* ACTIVE BATTLEGROUND FIGHTING HUD */
        #game-hud {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            padding: 25px;
            display: none;
            flex-direction: column;
            gap: 10px;
            z-index: 5;
        }
        .hud-main-row {
            display: flex;
            justify-content: space-between;
            align-items: center;
        }
        .health-bar-container {
            width: 42%;
            background: #222;
            border: 3px solid #fff;
            height: 35px;
            border-radius: 6px;
            overflow: hidden;
            position: relative;
        }
        .health-fill {
            width: 100%;
            height: 100%;
            background: #ff2e2e;
            transition: width 0.15s ease-out;
        }
        #p1-health { background: #00ff66; }
        .fighter-label {
            position: absolute;
            top: 5px;
            left: 10px;
            color: white;
            font-weight: bold;
            text-shadow: 1px 1px 3px black;
        }
        .enemy-label { right: 10px; left: auto; }
        
        #boss-announcer {
            text-align: center;
            color: #ffd700;
            font-size: 26px;
            font-weight: bold;
            text-shadow: 0 0 10px rgba(255,215,0,0.6);
        }

        #controls-hint {
            position: absolute;
            bottom: 15px;
            left: 50%;
            transform: translateX(-50%);
            color: white;
            background: rgba(0,0,0,0.75);
            padding: 10px 20px;
            border-radius: 25px;
            font-size: 15px;
            display: none;
            pointer-events: none;
            text-align: center;
            border: 1px solid #ff2a6d;
        }
    </style>
</head>
<body>

    <div id="canvas-container"></div>

    <div id="ui-layer">

        <div id="currency-hud" class="interactive">
            <div>🪙 Coins: <span id="hud-coins" class="coin-count">100</span></div>
            <div>💪 Strength: <span id="hud-strength" class="strength-count">10</span></div>
        </div>

        <button class="back-btn interactive" id="global-back-btn" style="display:none;" onclick="returnToMenu()">← BACK MENU</button>

        <div id="main-menu-screen" class="interactive">
            <button class="sidebar-toggle" id="sidebar-toggle-btn" onclick="toggleSidebar()">Open</button>
            <button class="play-btn" onclick="startBossSelection()">PLAY</button>

            <div id="sidebar">
                <div class="menu-item" onclick="openScreen('weapons')">
                    <span class="menu-icon">🥊</span> Weopons
                </div>
                <div class="menu-item" onclick="openScreen('profile')">
                    <span class="menu-icon">👤</span> Profile
                </div>
                <div class="menu-item" onclick="openScreen('settings')">
                    <span class="menu-icon">⚙️</span> Settings
                </div>
                <div class="menu-item" onclick="openScreen('shop')" style="margin-top:auto; background: rgba(255,255,255,0.15);">
                    <span class="menu-icon">🛍️</span> Shop Panel
                </div>
            </div>
        </div>

        <div id="shop-screen" class="modal-screen interactive">
            <div class="shop-container">
                <div class="shop-header">
                    <span>🛍️ SHOP</span>
                    <span style="font-size:20px; background:#0d1b3e; padding:5px 15px; border-radius:10px;">CODES: ........</span>
                </div>
                <div class="shop-carousel">
                    <div class="shop-item" onclick="buyItem('Spike Gloves', 40, 6)">
                        <div class="item-title">Spike Gloves</div>
                        <div class="item-bonus">+6 DMG Bonus</div>
                        <div class="item-cost">🪙 40</div>
                    </div>
                    <div class="shop-item" onclick="buyItem('Fire Knuckles', 75, 12)">
                        <div class="item-title">Fire Knuckles</div>
                        <div class="item-bonus">+12 DMG Bonus</div>
                        <div class="item-cost">🪙 75</div>
                    </div>
                    <div class="shop-item" onclick="buyItem('Plasma Wraps', 120, 22)">
                        <div class="item-title">Plasma Wraps</div>
                        <div class="item-bonus">+22 DMG Bonus</div>
                        <div class="item-cost">🪙 120</div>
                    </div>
                    <div class="shop-item" onclick="buyItem('Nemesis Claws', 200, 45)">
                        <div class="item-title">Nemesis Claws</div>
                        <div class="item-bonus">+45 DMG Bonus</div>
                        <div class="item-cost">🪙 200</div>
                    </div>
                </div>
            </div>
        </div>

        <div id="weapons-screen" class="modal-screen interactive">
            <div class="weapons-box">
                <input type="text" class="search-bar" placeholder="Search:">
                <div class="weapon-inv-list" id="inventory-container">
                    </div>
            </div>
        </div>

        <div id="profile-screen" class="modal-screen interactive"></div>

        <div id="settings-screen" class="modal-screen interactive">
            <div class="settings-container">
                <div style="font-size:32px; font-weight:bold; margin-bottom:30px;">⚙️ Settings</div>
                <div class="setting-row"><span>Music:</span> <div class="toggle-switch" onclick="toggleSetting(this)">ON</div></div>
                <div class="setting-row"><span>Visual effects:</span> <div class="toggle-switch" onclick="toggleSetting(this)">ON</div></div>
                <div class="setting-row"><span>Shakes:</span> <div class="toggle-switch" onclick="toggleSetting(this)">ON</div></div>
                <div class="setting-row"><span>Gifts Allowed:</span> <div class="toggle-switch" onclick="toggleSetting(this)">ON</div></div>
            </div>
        </div>

        <div id="game-hud">
            <div id="boss-announcer">BOSS FIGHT</div>
            <div class="hud-main-row">
                <div class="health-bar-container">
                    <div id="p1-health" class="health-fill"></div>
                    <span class="fighter-label">PLAYER (YOU)</span>
                </div>
                <div style="color:white; font-size:22px; font-weight:bold; background:rgba(0,0,0,0.6); padding:4px 12px; border-radius:4px;">VS</div>
                <div class="health-bar-container">
                    <div id="p2-health" class="health-fill"></div>
                    <span class="fighter-label enemy-label" id="hud-boss-name">NEMESIS BOSS</span>
                </div>
            </div>
        </div>

        <div id="controls-hint">
            <strong>CONTROLS:</strong> Press <strong>[ SPACEBAR ]</strong> to Punch! / Enemy moves and attacks automatically!
        </div>

    </div>

    <script>
        /* ECONOMY & RPG DATA STRUCTURES */
        let gameState = {
            coins: 100,
            baseStrength: 10,
            weaponBonus: 0,
            inventory: ["Standard Fists"],
            equippedWeapon: "Standard Fists"
        };

        // Boss Ladder Array configuration
        const bossLadder = [
            { name: "Scrap Brawler", health: 80, strength: 5, color: 0x90a4ae, coinReward: 30 },
            { name: "Magma Core", health: 150, strength: 12, color: 0xff5722, coinReward: 60 },
            { name: "NEMESIS PRIME", health: 300, strength: 25, color: 0x212121, coinReward: 150 }
        ];
        let currentBossIndex = 0;

        /* SAVE & LOAD CORE VALUES UPWARDS TO DISPLAY */
        function updateHudDisplays() {
            document.getElementById('hud-coins').innerText = gameState.coins;
            document.getElementById('hud-strength').innerText = gameState.baseStrength + gameState.weaponBonus;
            renderInventoryUI();
        }

        function buyItem(itemName, cost, bonusDmg) {
            if (gameState.inventory.includes(itemName)) {
                alert("You already purchased this weapon modification!");
                return;
            }
            if (gameState.coins >= cost) {
                gameState.coins -= cost;
                gameState.inventory.push(itemName);
                alert(`Successfully bought and added ${itemName}! Head to Weopons panel to equip it.`);
                updateHudDisplays();
            } else {
                alert("Insufficient coins! Defeat more bosses to earn gold reward drops.");
            }
        }

        function renderInventoryUI() {
            const container = document.getElementById('inventory-container');
            container.innerHTML = "";
            
            gameState.inventory.forEach(wpn => {
                const isEquipped = gameState.equippedWeapon === wpn;
                // Calculate item attributes
                let displayBonus = 0;
                if(wpn === 'Spike Gloves') displayBonus = 6;
                if(wpn === 'Fire Knuckles') displayBonus = 12;
                if(wpn === 'Plasma Wraps') displayBonus = 22;
                if(wpn === 'Nemesis Claws') displayBonus = 45;

                const row = document.createElement('div');
                row.className = "inv-item-row";
                row.innerHTML = `
                    <span>🥊 ${wpn} (${displayBonus > 0 ? '+' + displayBonus : 'Base'} DMG)</span>
                    <button class="equip-btn" style="background:${isEquipped ? '#555' : '#4caf50'}" onclick="equipWeapon('${wpn}', ${displayBonus})">
                        ${isEquipped ? 'EQUIPPED' : 'EQUIP'}
                    </button>
                `;
                container.appendChild(row);
            });
        }

        function equipWeapon(wpnName, dmgBonus) {
            gameState.equippedWeapon = wpnName;
            gameState.weaponBonus = dmgBonus;
            alert(`Equipped ${wpnName}!`);
            updateHudDisplays();
        }

        /* NAVIGATION SCREEN CONTROLLER */
        let isSidebarOpen = false;

        function toggleSidebar() {
            const sidebar = document.getElementById('sidebar');
            const toggleBtn = document.getElementById('sidebar-toggle-btn');
            isSidebarOpen = !isSidebarOpen;
            
            if (isSidebarOpen) {
                sidebar.classList.add('open');
                toggleBtn.innerText = "Close";
                toggleBtn.style.left = "260px";
            } else {
                sidebar.classList.remove('open');
                toggleBtn.innerText = "Open";
                toggleBtn.style.left = "10px";
            }
        }

        function openScreen(screenId) {
            document.getElementById('main-menu-screen').style.display = 'none';
            document.getElementById('global-back-btn').style.display = 'block';
            document.getElementById(`${screenId}-screen`).style.display = 'flex';
        }

        function returnToMenu() {
            const modals = document.querySelectorAll('.modal-screen');
            modals.forEach(m => m.style.display = 'none');
            
            document.getElementById('global-back-btn').style.display = 'none';
            document.getElementById('main-menu-screen').style.display = 'flex';
            document.getElementById('currency-hud').style.display = 'flex';
            if (isSidebarOpen) toggleSidebar();
            
            document.getElementById('game-hud').style.display = 'none';
            document.getElementById('controls-hint').style.display = 'none';
            gameActive = false;
        }

        function toggleSetting(element) {
            if (element.innerText === "ON") {
                element.innerText = "OFF";
                element.style.background = "#ff4a4a";
            } else {
                element.innerText = "ON";
                element.style.background = "#4caf50";
            }
        }

        /* THREE.JS 3D AUTOMATED ENGINE & AI COMBAT CORE */
        let scene, camera, renderer;
        let player1, player2, bossMeshMaterial;
        let gameActive = false;
        
        // Runtime Interactive Variable Pools
        let p1MaxHp = 100, p1CurrentHp = 100;
        let p2MaxHp = 100, p2CurrentHp = 100;
        let p1PunchAnimTimer = 0, p2PunchAnimTimer = 0;
        let aiAttackCooldown = 0;

        function init3D() {
            const container = document.getElementById('canvas-container');
            
            scene = new THREE.Scene();
            scene.background = new THREE.Color(0x120524);
            scene.fog = new THREE.FogExp2(0x120524, 0.04);

            camera = new THREE.PerspectiveCamera(45, window.innerWidth / window.innerHeight, 0.1, 1000);
            camera.position.set(0, 4, 9);
            camera.lookAt(0, 1.3, 0);

            renderer = new THREE.WebGLRenderer({ antialias: true });
            renderer.setSize(window.innerWidth, window.innerHeight);
            renderer.shadowMap.enabled = true;
            container.appendChild(renderer.domElement);

            const ambientLight = new THREE.AmbientLight(0xffffff, 0.5);
            scene.add(ambientLight);

            const spotLight = new THREE.PointLight(0xff00aa, 1.5, 30);
            spotLight.position.set(0, 8, 2);
            scene.add(spotLight);

            // Ring Base Platform Setup
            const stageGeo = new THREE.CylinderGeometry(5, 5.3, 0.4, 10);
            const stageMat = new THREE.MeshStandardMaterial({ color: 0x1f113a, roughness: 0.7 });
            const stage = new THREE.Mesh(stageGeo, stageMat);
            stage.position.y = -0.2;
            scene.add(stage);

            // Spawn Low-Poly Rigged Models
            player1 = createFighterModel(0x00d2ff, -1.8);
            player2 = createFighterModel(0xff2a6d, 1.8);
            scene.add(player1);
            scene.add(player2);

            player1.rotation.y = Math.PI / 2;
            player2.rotation.y = -Math.PI / 2;

            // Retain handle reference to alter Boss textures color dynamically per match tier
            bossMeshMaterial = player2.children[0].material;

            animate();
        }

        function createFighterModel(colorHex, posX) {
            const group = new THREE.Group();
            const mat = new THREE.MeshStandardMaterial({ color: colorHex, roughness: 0.6, flatShading: true });

            // Torso
            const torso = new THREE.Mesh(new THREE.BoxGeometry(0.7, 1.1, 0.5), mat);
            torso.position.y = 1.3;
            group.add(torso);

            // Head
            const head = new THREE.Mesh(new THREE.BoxGeometry(0.45, 0.45, 0.45), mat);
            head.position.y = 2.2;
            group.add(head);

            // Arms
            const armGeo = new THREE.BoxGeometry(0.25, 0.7, 0.25);
            
            const leftArm = new THREE.Mesh(armGeo, mat);
            leftArm.name = "LeftArm";
            leftArm.position.set(0.5, 1.3, 0);
            group.add(leftArm);

            const rightArm = new THREE.Mesh(armGeo, mat);
            rightArm.name = "RightArm";
            rightArm.position.set(-0.5, 1.3, 0.2);
            rightArm.rotation.x = -Math.PI / 4;
            group.add(rightArm);

            // Legs
            const legGeo = new THREE.BoxGeometry(0.25, 0.8, 0.25);
            const leftLeg = new THREE.Mesh(legGeo, mat);
            leftLeg.position.set(0.2, 0.4, 0);
            group.add(leftLeg);

            const rightLeg = new THREE.Mesh(legGeo, mat);
            rightLeg.position.set(-0.2, 0.4, 0);
            group.add(rightLeg);

            group.position.x = posX;
            return group;
        }

        /* RENDERING ANIMATION TICK & ENEMY COMBAT AI ENGINE */
        function animate() {
            requestAnimationFrame(animate);

            const time = Date.now() * 0.004;

            if (player1 && player2) {
                // Sway fighters smoothly
                player1.position.y = Math.sin(time) * 0.04;
                player2.position.y = Math.cos(time) * 0.04;

                // Process Player 1 Punch visual rotation cycles
                const p1Arm = player1.getObjectByName("RightArm");
                if (p1PunchAnimTimer > 0) {
                    p1PunchAnimTimer--;
                    if(p1Arm) p1Arm.rotation.x = -Math.PI / 2 - (p1PunchAnimTimer * 0.15);
                } else {
                    if(p1Arm) p1Arm.rotation.x = -Math.PI / 4;
                }

                // Process Opponent Boss AI Punch visual cycles
                const p2Arm = player2.getObjectByName("RightArm");
                if (p2PunchAnimTimer > 0) {
                    p2PunchAnimTimer--;
                    if(p2Arm) p2Arm.rotation.x = -Math.PI / 2 - (p2PunchAnimTimer * 0.15);
                } else {
                    if(p2Arm) p2Arm.rotation.x = -Math.PI / 4;
                }

                // SIMULATE ACTIVE BOSS FIGHT AI LOGIC
                if (gameActive) {
                    // Smoothly close distance gap step-by-step up into weapon range closer
                    if (player2.position.x > 0.9) {
                        player2.position.x -= 0.01;
                        player1.position.x += 0.01;
                    }

                    // Process AI Cooldown Attack ticks
                    aiAttackCooldown++;
                    // Faster attack rates depending on boss scaling
                    const targetRate = 50 - (currentBossIndex * 10); 
                    if (aiAttackCooldown >= targetRate) {
                        aiAttackCooldown = 0;
                        triggerAiBossStrike();
                    }
                } else {
                    // Reposition characters smoothly back to normal when not in game
                    if(player2.position.x < 1.8) player2.position.x += 0.05;
                    if(player1.position.x > -1.8) player1.position.x -= 0.05;
                }
            }

            renderer.render(scene, camera);
        }

        /* COMBAT ENGINES */
        function startBossSelection() {
            // Check progression scaling loops
            const currentBoss = bossLadder[currentBossIndex];
            
            // Adjust materials directly to render visual updates across scaling tiers
            bossMeshMaterial.color.setHex(currentBoss.color);

            // Initialize variables with modified stats
            p1MaxHp = 100; p1CurrentHp = 100;
            p2MaxHp = currentBoss.health; p2CurrentHp = currentBoss.health;

            document.getElementById('hud-boss-name').innerText = currentBoss.name.toUpperCase();
            document.getElementById('boss-announcer').innerText = `STAGE ${currentBossIndex + 1}: ${currentBoss.name}`;
            
            document.getElementById('p1-health').style.width = '100%';
            document.getElementById('p2-health').style.width = '100%';

            // View Configuration Transitions
            document.getElementById('main-menu-screen').style.display = 'none';
            document.getElementById('currency-hud').style.display = 'none';
            document.getElementById('global-back-btn').style.display = 'block';
            document.getElementById('game-hud').style.display = 'flex';
            document.getElementById('controls-hint').style.display = 'block';

            gameActive = true;
        }

        // PLAYER MOVES ATTACK HANDLER
        window.addEventListener('keydown', function(e) {
            if (!gameActive) return;

            if (e.code === 'Space') {
                p1PunchAnimTimer = 8;
                
                // Deal weapon-upgraded dynamic values
                const totalDmg = gameState.baseStrength + gameState.weaponBonus;
                p2CurrentHp -= totalDmg;
                if (p2CurrentHp < 0) p2CurrentHp = 0;

                document.getElementById('p2-health').style.width = ((p2CurrentHp / p2MaxHp) * 100) + '%';

                // Check Match Ending constraint vectors
                if (p2CurrentHp <= 0) {
                    gameActive = false;
                    const activeBoss = bossLadder[currentBossIndex];
                    gameState.coins += activeBoss.coinReward;
                    gameState.baseStrength += 3; // Permanent power increase for victory

                    setTimeout(() => {
                        alert(`🏆 VICTORY! Defeated ${activeBoss.name}!\nEarned: 🪙 ${activeBoss.coinReward} Coins\nStrength Permanent Upgrade: +3 Base Strength!`);
                        
                        // Advance to next ladder tier if possible
                        currentBossIndex++;
                        if(currentBossIndex >= bossLadder.length) {
                            alert("💥 AMAZING FIGHTING EXPERT PERFORMANCE! You defeated all the Nemesis Bosses and beat the game! Resetting back to Stage 1.");
                            currentBossIndex = 0;
                        }
                        returnToMenu();
                        updateHudDisplays();
                    }, 200);
                }
            }
        });

        // ENEMY BOSS MOVES ATTACK AUTOMATION LOOP
        function triggerAiBossStrike() {
            if (!gameActive) return;
            
            p2PunchAnimTimer = 8;
            const activeBoss = bossLadder[currentBossIndex];
            
            // Subtract health values
            p1CurrentHp -= activeBoss.strength;
            if (p1CurrentHp < 0) p1CurrentHp = 0;

            document.getElementById('p1-health').style.width = ((p1CurrentHp / p1MaxHp) * 100) + '%';

            // Check Player Defeat condition
            if (p1CurrentHp <= 0) {
                gameActive = false;
                setTimeout(() => {
                    alert(`🔴 DEFEAT! You were knocked out by ${activeBoss.name}. Upgrade your weapons or strength in the shop and try again!`);
                    returnToMenu();
                    updateHudDisplays();
                }, 200);
            }
        }

        window.addEventListener('resize', () => {
            camera.aspect = window.innerWidth / window.innerHeight;
            camera.updateProjectionMatrix();
            renderer.setSize(window.innerWidth, window.innerHeight);
        });

        window.onload = () => {
            init3D();
            updateHudDisplays();
        };
    </script>
</body>
</html>
