<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>NEMESIS - 3D Fighting Game</title>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/three.js/r128/three.min.js"></script>
    <style>
        /* Base Styling & Reset */
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

        /* 3D Canvas Layer */
        #canvas-container {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            z-index: 1;
        }

        /* UI Overlay Container */
        #ui-layer {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            z-index: 10;
            pointer-events: none; /* Let clicks pass to canvas unless over UI elements */
        }
        
        .interactive {
            pointer-events: auto; /* Re-enable clicks for UI elements */
        }

        /* SCREEN 1 & 2: MAIN MENU & SIDEBAR */
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

        /* Top Left Toggle Button (Open/Close Sidebar) */
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

        /* Center Play Button */
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

        /* Sliding Sidebar Panel */
        #sidebar {
            position: absolute;
            top: 0;
            left: -320px; /* Hidden by default */
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

        /* Sidebar Menu Items */
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
        .menu-icon { width: 40px; height: 40px; }

        /* MODAL SCREEN BASE (Weapons, Profile, Settings, Shop) */
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
            height: 400px;
            background: #888;
            border-radius: 20px;
            padding: 20px;
            box-shadow: inset 0 0 20px rgba(0,0,0,0.4);
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
        }

        /* SCREEN 5: PROFILE */
        #profile-screen {
            background: #e0e0e0 url('Schermafbeelding 2026-06-09 223259.png') no-repeat center center;
            background-size: contain;
        }
        /* Custom UI element layer over the snapshot if image isn't loaded */
        .fallback-profile {
            padding: 30px;
            color: #333;
        }

        /* SCREEN 6: SETTINGS */
        #settings-screen {
            background: linear-gradient(135deg, #110033, #ff99bb);
            display: none; /* Handled dynamically */
            flex-direction: column;
            align-items: center;
            justify-content: center;
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
            text-shadow: 1px 1px 2px rgba(0,0,0,0.5);
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

        /* SCREEN 3: SHOP */
        #shop-screen {
            background: linear-gradient(135deg, #020212, #201554, #8a2be2);
        }
        .shop-container {
            width: 90%;
            max-width: 800px;
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
            height: 60%;
            border-radius: 25px;
            display: flex;
            padding: 20px;
            gap: 15px;
            overflow-x: auto;
            align-items: center;
            border-bottom: 15px solid #666;
        }
        .shop-item {
            min-width: 160px;
            height: 100%;
            background: #0d1b3e;
            border-radius: 15px;
            border: 2px solid #1e3c72;
            cursor: pointer;
            transition: transform 0.2s;
        }
        .shop-item:hover { transform: scale(1.05); border-color: #00ffff; }

        /* Back Navigation Button for Modals */
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

        /* HUD Overlay (Active during 3D Battle) */
        #game-hud {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            padding: 20px;
            display: none;
            justify-content: space-between;
            z-index: 5;
        }
        .health-bar-container {
            width: 40%;
            background: #333;
            border: 3px solid #fff;
            height: 30px;
            border-radius: 5px;
            overflow: hidden;
        }
        .health-fill {
            width: 100%;
            height: 100%;
            background: #ff2e2e;
            transition: width 0.2s ease;
        }
        #p2-health { background: #2eff77; } /* Opponent health color variant */

        /* Game Instructions Overlay */
        #controls-hint {
            position: absolute;
            bottom: 10px;
            left: 50%;
            transform: translateX(-50%);
            color: white;
            background: rgba(0,0,0,0.6);
            padding: 8px 15px;
            border-radius: 20px;
            font-size: 14px;
            display: none;
            pointer-events: none;
            text-align: center;
        }
    </style>
</head>
<body>

    <div id="canvas-container"></div>

    <div id="ui-layer">

        <button class="back-btn interactive" id="global-back-btn" style="display:none;" onclick="returnToMenu()">← BACK MENU</button>

        <div id="main-menu-screen" class="interactive">
            <button class="sidebar-toggle" id="sidebar-toggle-btn" onclick="toggleSidebar()">Open</button>
            
            <button class="play-btn" onclick="startGame()">PLAY</button>

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
                    <div class="shop-item" onclick="alert('Item Unlocked!')"></div>
                    <div class="shop-item" onclick="alert('Item Unlocked!')"></div>
                    <div class="shop-item" onclick="alert('Item Unlocked!')"></div>
                    <div class="shop-item" onclick="alert('Item Unlocked!')"></div>
                </div>
            </div>
        </div>

        <div id="weapons-screen" class="modal-screen interactive">
            <div class="weapons-box">
                <input type="text" class="search-bar" placeholder="Search:">
                <p style="color:white; text-align:center; margin-top:100px; font-weight:bold;">Equip Fighting Gloves</p>
            </div>
        </div>

        <div id="profile-screen" class="modal-screen interactive">
            <div class="fallback-profile">
                </div>
        </div>

        <div id="settings-screen" class="modal-screen interactive">
            <div class="settings-container">
                <div style="font-size:32px; font-weight:bold; margin-bottom:30px; display:flex; align-items:center; gap:15px;">⚙️ Settings</div>
                <div class="setting-row">
                    <span>Music:</span> <div class="toggle-switch" onclick="toggleSetting(this)">ON</div>
                </div>
                <div class="setting-row">
                    <span>Visual effects:</span> <div class="toggle-switch" onclick="toggleSetting(this)">ON</div>
                </div>
                <div class="setting-row">
                    <span>Shakes:</span> <div class="toggle-switch" onclick="toggleSetting(this)">ON</div>
                </div>
                <div class="setting-row">
                    <span>Gifts Allowed:</span> <div class="toggle-switch" onclick="toggleSetting(this)">ON</div>
                </div>
            </div>
        </div>

        <div id="game-hud">
            <div class="health-bar-container"><div id="p1-health" class="health-fill"></div></div>
            <div style="color:white; font-size:24px; font-weight:bold; background:rgba(0,0,0,0.5); padding:0 10px;">VS</div>
            <div class="health-bar-container"><div id="p2-health" class="health-fill"></div></div>
        </div>

        <div id="controls-hint">
            <strong>CONTROLS:</strong> Press [ SPACEBAR ] to Strike / Punch your Nemesis opponent!
        </div>

    </div>

    <script>
        /* UI STATE CONTROLLER */
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
            // Hide menu and show the correct overlay container
            document.getElementById('main-menu-screen').style.display = 'none';
            document.getElementById('global-back-btn').style.display = 'block';
            document.getElementById(`${screenId}-screen`).style.display = 'flex';
        }

        function returnToMenu() {
            // Hide all sub-modals
            const modals = document.querySelectorAll('.modal-screen');
            modals.forEach(m => m.style.display = 'none');
            
            // Re-display landing screen
            document.getElementById('global-back-btn').style.display = 'none';
            document.getElementById('main-menu-screen').style.display = 'flex';
            if (isSidebarOpen) toggleSidebar(); // Reset bar toggle
            
            // Clean up 3D active fighting session if active
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

        /* THREE.JS 3D LOW-POLY ENGINE SETUP */
        let scene, camera, renderer;
        let player1, player2;
        let gameActive = false;
        let p2HealthVal = 100;
        let punchAnimationTimer = 0;

        function init3D() {
            const container = document.getElementById('canvas-container');
            
            // Initialize Scene
            scene = new THREE.Scene();
            scene.background = new THREE.Color(0x1a0933);
            scene.fog = new THREE.FogExp2(0x1a0933, 0.05);

            // Camera positioning
            camera = new THREE.PerspectiveCamera(45, window.innerWidth / window.innerHeight, 0.1, 1000);
            camera.position.set(0, 4, 10);
            camera.lookAt(0, 1.5, 0);

            // WebGL Renderer Setup
            renderer = new THREE.WebGLRenderer({ antialias: true });
            renderer.setSize(window.innerWidth, window.innerHeight);
            renderer.shadowMap.enabled = true;
            container.appendChild(renderer.domElement);

            // Lighting Design
            const ambientLight = new THREE.AmbientLight(0xffffff, 0.4);
            scene.add(ambientLight);

            const stageLight = new THREE.DirectionalLight(0xffaa44, 0.8);
            stageLight.position.set(5, 10, 5);
            stageLight.castShadow = true;
            scene.add(stageLight);

            // Low-Poly Fighting Stage Ring
            const stageGeo = new THREE.CylinderGeometry(5, 5.5, 0.5, 8);
            const stageMat = new THREE.MeshStandardMaterial({ color: 0x221144, roughness: 0.8 });
            const stage = new THREE.Mesh(stageGeo, stageMat);
            stage.position.y = -0.25;
            stage.receiveShadow = true;
            scene.add(stage);

            // Build Low-Poly Fighter Character Models
            player1 = createFighter(0x00d2ff, -2); // Cyan Champion
            player2 = createFighter(0xff2a6d, 2);  // Nemesis Opponent
            scene.add(player1);
            scene.add(player2);

            // Turn player models to face each other across the ring axis
            player1.rotation.y = Math.PI / 2;
            player2.rotation.y = -Math.PI / 2;

            // Start Render Loop frame ticks
            animate();
        }

        function createFighter(colorHex, posX) {
            const group = new THREE.Group();
            const mat = new THREE.MeshStandardMaterial({ color: colorHex, roughness: 0.5, flatShading: true });

            // Torso body element
            const torsoGeo = new THREE.BoxGeometry(0.8, 1.2, 0.5);
            const torso = new THREE.Mesh(torsoGeo, mat);
            torso.position.y = 1.4;
            torso.castShadow = true;
            group.add(torso);

            // Head element
            const headGeo = new THREE.BoxGeometry(0.5, 0.5, 0.5);
            const head = new THREE.Mesh(headGeo, mat);
            head.position.y = 2.4;
            head.castShadow = true;
            group.add(head);

            // Arms (Low poly blocks)
            const armGeo = new THREE.BoxGeometry(0.3, 0.8, 0.3);
            
            const leftArm = new THREE.Mesh(armGeo, mat);
            leftArm.name = "LeftArm";
            leftArm.position.set(0.6, 1.4, 0);
            leftArm.castShadow = true;
            group.add(leftArm);

            const rightArm = new THREE.Mesh(armGeo, mat);
            rightArm.name = "RightArm";
            rightArm.position.set(-0.6, 1.4, 0.2); // Sightly extended forward dynamic stance
            rightArm.rotation.x = -Math.PI / 4;
            rightArm.castShadow = true;
            group.add(rightArm);

            // Legs base
            const legGeo = new THREE.BoxGeometry(0.3, 0.9, 0.3);
            const leftLeg = new THREE.Mesh(legGeo, mat);
            leftLeg.position.set(0.25, 0.45, 0);
            group.add(leftLeg);

            const rightLeg = new THREE.Mesh(legGeo, mat);
            rightLeg.position.set(-0.25, 0.45, 0);
            group.add(rightLeg);

            group.position.x = posX;
            return group;
        }

        // Scene Render Frame Tick Engine
        function animate() {
            requestAnimationFrame(animate);

            // Idle micro-animations for players during runtime
            const time = Date.now() * 0.003;
            if(player1 && player2) {
                player1.position.y = Math.sin(time) * 0.05;
                player2.position.y = Math.cos(time) * 0.05;
                
                // Track visual rotation values if action event resets back down
                if (punchAnimationTimer > 0) {
                    punchAnimationTimer--;
                    const arm = player1.getObjectByName("RightArm");
                    if(arm) arm.rotation.x = -Math.PI / 2 - (punchAnimationTimer * 0.1);
                } else {
                    const arm = player1.getObjectByName("RightArm");
                    if(arm) arm.rotation.x = -Math.PI / 4;
                }
            }

            renderer.render(scene, camera);
        }

        /* START MATCH & IN-GAME ACTIONS */
        function startGame() {
            document.getElementById('main-menu-screen').style.display = 'none';
            document.getElementById('global-back-btn').style.display = 'block';
            document.getElementById('game-hud').style.display = 'flex';
            document.getElementById('controls-hint').style.display = 'block';
            
            // Reset state values
            p2HealthVal = 100;
            document.getElementById('p2-health').style.width = '100%';
            gameActive = true;
        }

        // Input Action Handlers for Fighting Mechanics
        window.addEventListener('keydown', function(e) {
            if (!gameActive) return;

            if (e.code === 'Space') {
                // Trigger Attack Strike Animation Event
                punchAnimationTimer = 10;
                
                // Deal Damage hit calculation
                p2HealthVal -= 8;
                if (p2HealthVal < 0) p2HealthVal = 0;
                
                document.getElementById('p2-health').style.width = p2HealthVal + '%';

                // Check Knockout win constraint condition
                if (p2HealthVal <= 0) {
                    gameActive = false;
                    setTimeout(() => {
                        alert('KO! Nemesis Defeated! Victory is yours!');
                        returnToMenu();
                    }, 200);
                }
            }
        });

        // Window resize responsive handler
        window.addEventListener('resize', onWindowResize, false);
        function onWindowResize() {
            camera.aspect = window.innerWidth / window.innerHeight;
            camera.updateProjectionMatrix();
            renderer.setSize(window.innerWidth, window.innerHeight);
        }

        // Init Engine Core on boot window cycle
        window.onload = () => {
            init3D();
        };
    </script>
</body>
</html>
