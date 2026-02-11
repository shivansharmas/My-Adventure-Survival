<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>3D Forest Survival Game</title>
    <style>
        body { margin: 0; overflow: hidden; font-family: Arial, sans-serif; background: #87CEEB; }
        #ui {
            position: absolute;
            top: 20px;
            left: 20px;
            color: white;
            background: rgba(0,0,0,0.8);
            padding: 20px;
            border-radius: 10px;
            z-index: 100;
        }
        #healthBar {
            width: 200px;
            height: 20px;
            background: #333;
            border: 2px solid #fff;
            border-radius: 10px;
            overflow: hidden;
            margin-bottom: 10px;
        }
        #healthFill {
            height: 100%;
            background: linear-gradient(90deg, #ff0000, #ffff00);
            width: 100%;
            transition: width 0.3s;
        }
        #timeDisplay {
            margin-top: 10px;
            font-size: 18px;
        }
        #inventory {
            position: absolute;
            top: 20px;
            right: 20px;
            color: white;
            background: rgba(0,0,0,0.8);
            padding: 20px;
            border-radius: 10px;
            z-index: 100;
            min-width: 200px;
        }
        #inventory h3 {
            margin-top: 0;
            text-align: center;
        }
        .resource {
            display: flex;
            justify-content: space-between;
            margin: 5px 0;
            font-size: 16px;
        }
        /* QUESTS UI */
        #quests {
            position: absolute;
            top: 20px;
            right: 250px;
            color: white;
            background: rgba(0,0,0,0.9);
            padding: 20px;
            border-radius: 10px;
            z-index: 100;
            min-width: 220px;
            max-height: 200px;
            overflow-y: auto;
        }
        #quests h3 {
            margin-top: 0;
            text-align: center;
            color: #ffff00;
        }
        .quest {
            margin: 8px 0;
            font-size: 14px;
            padding: 5px;
            border-radius: 5px;
            transition: all 0.3s;
        }
        .quest.active {
            background: rgba(0,255,0,0.3);
            border: 1px solid #00ff00;
        }
        .quest.completed {
            background: rgba(128,128,128,0.5);
            text-decoration: line-through;
            color: #888;
            border: 1px solid #666;
        }
        .quest-progress {
            font-weight: bold;
            color: #ffff00;
        }
        #hotbar {
            position: absolute;
            bottom: 20px;
            left: 50%;
            transform: translateX(-50%);
            display: flex;
            gap: 20px;
            z-index: 100;
        }
        .hotbarSlot {
            width: 80px;
            height: 80px;
            background: rgba(0,0,0,0.7);
            border: 3px solid #fff;
            border-radius: 10px;
            display: flex;
            flex-direction: column;
            align-items: center;
            justify-content: center;
            color: white;
            font-weight: bold;
            font-size: 18px;
            cursor: pointer;
            transition: all 0.3s;
        }
        .hotbarSlot:hover {
            background: rgba(255,255,255,0.2);
            border-color: #ffff00;
        }
        .hotbarSlot.selected {
            border-color: #ffff00;
            box-shadow: 0 0 20px #ffff00;
        }
        #hands {
            position: fixed;
            top: 0;
            left: 0;
            width: 100vw;
            height: 100vh;
            pointer-events: none;
            z-index: 50;
        }
        .hand {
            position: absolute;
            width: 100px;
            height: 250px;
            background: linear-gradient(145deg, #D2B48C, #DEB887, #F4A460);
            border-radius: 30px 15px 15px 30px;
            box-shadow:
                inset 10px 10px 20px rgba(0,0,0,0.5),
                0 0 40px rgba(255,255,0,0.4),
                5px 0 15px rgba(0,0,0,0.3);
            opacity: 0;
            transition: all 0.2s ease;
            border: 4px solid #B8860B;
        }
        #leftHand {
            left: 2%;
            top: 40%;
            transform: rotate(-45deg);
        }
        #rightHand {
            right: 2%;
            top: 40%;
            transform: rotate(45deg);
        }
        .hand.fist-equipped {
            opacity: 0.9;
            transform: scale(1.1);
        }
        .hand.punch {
            animation: punchAnim 0.3s ease-out;
            box-shadow:
                inset 10px 10px 20px rgba(0,0,0,0.5),
                0 0 60px rgba(255,165,0,0.8),
                10px 0 25px rgba(255,0,0,0.6);
        }
        .hand.death-punch {
            animation: deathPunch 1s ease-out;
            box-shadow:
                inset 10px 10px 20px rgba(0,0,0,0.8),
                0 0 100px rgba(255,0,0,1),
                20px 0 50px rgba(255,100,0,0.8);
        }
        @keyframes punchAnim {
            0% { transform: scale(1.1) translateY(0); }
            50% { transform: scale(1.3) translateY(-20px); }
            100% { transform: scale(1.1) translateY(0); }
        }
        @keyframes deathPunch {
            0% { transform: scale(1.1) translateY(0) rotate(0deg); }
            50% { transform: scale(2.0) translateY(-50px) rotate(180deg); }
            100% { transform: scale(1.1) translateY(0) rotate(360deg); }
        }
        .crosshair {
            position: absolute;
            top: 50%;
            left: 50%;
            transform: translate(-50%, -50%);
            width: 20px;
            height: 20px;
            border: 2px solid white;
            border-radius: 50%;
            box-shadow: 0 0 10px white;
            z-index: 99;
        }
        #instructions {
            position: absolute;
            bottom: 120px;
            left: 50%;
            transform: translateX(-50%);
            color: white;
            background: rgba(0,0,0,0.8);
            padding: 10px 20px;
            border-radius: 10px;
            z-index: 100;
            display: none;
        }
        #deathScreen {
            position: absolute;
            top: 50%;
            left: 50%;
            transform: translate(-50%, -50%);
            color: #ff4444;
            font-size: 64px;
            font-weight: bold;
            text-align: center;
            background: rgba(0,0,0,0.95);
            padding: 40px;
            border-radius: 20px;
            border: 4px solid #ff4444;
            z-index: 1000;
            display: none;
            text-shadow: 3px 3px 6px black;
            box-shadow: 0 0 50px rgba(255,68,68,0.8);
        }
        #restartBtn {
            margin-top: 20px;
            padding: 15px 30px;
            font-size: 24px;
            background: #ff4444;
            color: white;
            border: none;
            border-radius: 10px;
            cursor: pointer;
            font-weight: bold;
        }
        #restartBtn:hover {
            background: #ff6666;
            box-shadow: 0 0 20px #ff4444;
        }
        #damageText {
            position: absolute;
            top: 50%;
            left: 50%;
            transform: translate(-50%, -50%);
            color: #ff4444;
            font-size: 48px;
            font-weight: bold;
            pointer-events: none;
            opacity: 0;
            z-index: 200;
            text-shadow: 2px 2px 4px black;
        }
        .enemy-health-bar {
            position: absolute;
            top: -15px;
            left: 50%;
            transform: translateX(-50%);
            width: 60px;
            height: 8px;
            background: rgba(0,0,0,0.8);
            border: 2px solid #ff2200;
            border-radius: 4px;
            overflow: hidden;
            font-size: 8px;
            color: white;
            text-align: center;
            line-height: 8px;
            font-weight: bold;
            pointer-events: none;
        }
        .enemy-health-fill {
            height: 100%;
            background: linear-gradient(90deg, #ff0000, #ff8800, #ffff00);
            transition: width 0.2s;
            border-radius: 2px;
        }
        #playButton {
            position: absolute;
            top: 50%;
            left: 50%;
            transform: translate(-50%, -50%);
            padding: 20px 50px;
            font-size: 32px;
            font-weight: bold;
            border-radius: 15px;
            border: none;
            cursor: pointer;
            z-index: 2000;
            background: linear-gradient(135deg, #00cc66, #33ff99);
            color: #fff;
            box-shadow: 0 0 25px rgba(0,255,128,0.8);
        }
        #playButton:hover {
            background: linear-gradient(135deg, #00ff88, #66ffbb);
            box-shadow: 0 0 35px rgba(0,255,160,1);
        }
    </style>
</head>
<body>
    <button id="playButton">PLAY</button>

    <div class="crosshair"></div>
    <div id="instructions">
        WASD | Space Jump | 0 Spin Attack | 1 Fist (5 DMG) | 2 Axe (chop trees) | CLICK to Punch! | Enemies ALWAYS ACTIVE!
    </div>
    <div id="damageText"></div>
    
    <div id="deathScreen">
        <div>💀 YOU DIED 💀</div>
        <div style="font-size: 24px; margin-top: 10px;">Final Revenge Punch!</div>
        <button id="restartBtn">RESTART</button>
    </div>

    <div id="ui">
        <div>Health: <span id="healthText">100/100</span></div>
        <div id="healthBar"><div id="healthFill"></div></div>
        <div>Level: <span id="level">1</span></div>
        <div>Day: <span id="dayCount">1</span></div>
    </div>

    <div id="quests">
        <h3>🗡️ QUESTS</h3>
        <div id="questList">
            <div class="quest active" id="killQuest1">
                Kill 5 Shadow Beasts <span class="quest-progress" id="killProgress1">0/5</span>
            </div>
        </div>
    </div>

    <div id="inventory">
        <h3>Inventory</h3>
        <div class="resource">✊ Fist <span id="fistCount">1</span></div>
        <div class="resource">🪓 Axe <span id="axeCount">1</span></div>
        <div class="resource">🪵 Wood <span id="woodCount">0</span></div>
        <div class="resource">🪨 Stone <span id="stoneCount">0</span></div>
        <div>Cash: $<span id="cash">0</span></div>
        <div id="timeDisplay">Daytime</div>
    </div>

    <div id="hotbar">
        <div class="hotbarSlot" data-slot="0">
            <div>0</div>
            <span>SPIN</span>
        </div>
        <div class="hotbarSlot selected" data-slot="1">
            <span id="fistIcon">✊</span>
            <div>1</div>
        </div>
        <div class="hotbarSlot" data-slot="2">
            <span>🪓</span>
            <div>2</div>
        </div>
    </div>

    <div id="hands">
        <div id="leftHand" class="hand fist-equipped"></div>
        <div id="rightHand" class="hand fist-equipped"></div>
    </div>

    <script type="importmap">
        {
            "imports": {
                "three": "https://unpkg.com/three@0.167.1/build/three.module.js",
                "three/addons/": "https://unpkg.com/three@0.167.1/examples/jsm/"
            }
        }
    </script>

    <script type="module">
        import * as THREE from 'three';
        import { PointerLockControls } from 'three/addons/controls/PointerLockControls.js';

        let gameRunning = false;

        const scene = new THREE.Scene();
        scene.background = new THREE.Color(0x87CEEB);
        scene.fog = new THREE.Fog(0x87CEEB, 20, 100);

        const camera = new THREE.PerspectiveCamera(
            75,
            window.innerWidth / window.innerHeight,
            0.1,
            1000
        );
        const renderer = new THREE.WebGLRenderer({ antialias: true });
        renderer.setSize(window.innerWidth, window.innerHeight);
        renderer.shadowMap.enabled = true;
        renderer.shadowMap.type = THREE.PCFSoftShadowMap;
        renderer.toneMappingExposure = 0.8;
        document.body.appendChild(renderer.domElement);

        const controls = new PointerLockControls(camera, document.body);
        scene.add(controls.getObject());
        camera.position.set(0, 1.8, 0);

        const instructions = document.getElementById('instructions');
        const playButton = document.getElementById('playButton');

        controls.addEventListener('lock', () => {
            if (gameRunning) instructions.style.display = 'none';
        });
        controls.addEventListener('unlock', () => {
            if (gameRunning) instructions.style.display = 'block';
        });

        playButton.addEventListener('click', () => {
            playButton.style.display = 'none';
            gameRunning = true;
            instructions.style.display = 'block';
            controls.lock();
        });

        let enemyKillCount = 0;
        const QUEST_GOAL = 5;
        const QUEST_REWARD = 10;

        const moveSpeed = 15;
        const velocity = new THREE.Vector3();
        let canJump = false;
        const keys = {};

        let totalYawRotation = 0;
        let lastYaw = 0;

        const ambientLight = new THREE.AmbientLight(0x404040, 0.8);
        scene.add(ambientLight);
        const directionalLight = new THREE.DirectionalLight(0xffffff, 1.2);
        directionalLight.position.set(50, 50, 50);
        directionalLight.castShadow = true;
        directionalLight.shadow.mapSize.width = 2048;
        directionalLight.shadow.mapSize.height = 2048;
        scene.add(directionalLight);

        const stars = new THREE.Points(
            new THREE.BufferGeometry().setAttribute(
                'position',
                new THREE.Float32BufferAttribute(
                    new Array(3000).fill(0).map(() => (Math.random() - 0.5) * 2000),
                    3
                )
            ),
            new THREE.PointsMaterial({ color: 0xffffff, size: 2 })
        );
        stars.visible = false;
        scene.add(stars);

        const ground = new THREE.Mesh(
            new THREE.PlaneGeometry(130, 130),
            new THREE.MeshLambertMaterial({ color: 0x2d5a27 })
        );
        ground.rotation.x = -Math.PI / 2;
        ground.receiveShadow = true;
        scene.add(ground);

        const boundarySize = 65;
        const stoneObjects = [];
        function createWall(x, z, width, height) {
            const wallGeo = new THREE.BoxGeometry(width, height, 2);
            const wallMat = new THREE.MeshLambertMaterial({ color: 0x666666 });
            const wall = new THREE.Mesh(wallGeo, wallMat);
            wall.position.set(x, height / 2, z);
            wall.castShadow = true;
            wall.receiveShadow = true;
            wall.userData = { type: 'stone', health: 100 };
            scene.add(wall);
            stoneObjects.push(wall);
            return wall;
        }
        createWall(0, boundarySize, 130, 8);
        createWall(0, -boundarySize, 130, 8);
        createWall(boundarySize, 0, 8, 8);
        createWall(-boundarySize, 0, 8, 8);

        const trees = [];
        class Tree {
            constructor(x, z) {
                this.group = new THREE.Group();
                this.health = 50;

                const trunk = new THREE.Mesh(
                    new THREE.CylinderGeometry(0.4, 0.8, 8),
                    new THREE.MeshLambertMaterial({ color: 0x654321 })
                );
                trunk.position.y = 4;
                trunk.castShadow = true;
                this.group.add(trunk);

                const foliageMat = new THREE.MeshLambertMaterial({ color: 0x228B22 });
                for (let i = 0; i < 4; i++) {
                    const foliage = new THREE.Mesh(
                        new THREE.SphereGeometry(4 - i * 0.8, 12, 8),
                        foliageMat
                    );
                    foliage.position.y = 7 + i * 1.5;
                    foliage.castShadow = true;
                    this.group.add(foliage);
                }

                this.group.position.set(x, 0, z);
                this.group.userData = { type: 'tree', ref: this };
                scene.add(this.group);
                trees.push(this);
            }
            takeDamage(amount) {
                this.health -= amount;
                if (this.health <= 0) {
                    scene.remove(this.group);
                    trees.splice(trees.indexOf(this), 1);
                    window.woodCount += 1;
                }
            }
        }

        for (let i = 0; i < 150; i++) {
            let x, z, attempts = 0;
            do {
                x = (Math.random() - 0.5) * 120;
                z = (Math.random() - 0.5) * 120;
                attempts++;
            } while (
                attempts < 30 &&
                trees.some(t => Math.hypot(t.group.position.x - x, t.group.position.z - z) < 8)
            );
            new Tree(x, z);
        }

        const enemies = [];
        class ShadowBeast {
            constructor(x, z) {
                this.group = new THREE.Group();
                this.maxHealth = 60;
                this.health = this.maxHealth;
                this.speed = 3 + Math.random() * 2;
                this.lastAttack = 0;
                this.floatingHeight = 0;
                this.daytimeSpeedMultiplier = 0.7;
                this.healthBar = null;
                this.healthText = null;

                const eyeMat = new THREE.MeshLambertMaterial({ color: 0xff2200, emissive: 0x440000 });
                const leftEye = new THREE.Mesh(new THREE.SphereGeometry(0.2, 8, 6), eyeMat);
                leftEye.position.set(-0.4, 2.2, 0.6);
                this.group.add(leftEye);
                
                const rightEye = new THREE.Mesh(new THREE.SphereGeometry(0.2, 8, 6), eyeMat);
                rightEye.position.set(0.4, 2.2, 0.6);
                this.group.add(rightEye);

                const bodyGeo = new THREE.CapsuleGeometry(0.8, 1.8, 4, 8);
                const bodyMat = new THREE.MeshLambertMaterial({ 
                    color: 0x1a1a2e, 
                    emissive: 0x0a0a14 
                });
                const body = new THREE.Mesh(bodyGeo, bodyMat);
                body.position.y = 1.2;
                body.castShadow = true;
                this.group.add(body);

                const tentacleMat = new THREE.MeshLambertMaterial({ color: 0x2a2a3e });
                for (let i = 0; i < 4; i++) {
                    const tentacle = new THREE.Mesh(
                        new THREE.CylinderGeometry(0.1, 0.3, 2.5, 6),
                        tentacleMat
                    );
                    const angle = (i / 4) * Math.PI * 2;
                    tentacle.position.set(
                        Math.cos(angle) * 1.2,
                        0.5,
                        Math.sin(angle) * 1.2 + 0.8
                    );
                    tentacle.rotation.z = angle + Math.PI / 2;
                    this.group.add(tentacle);
                }

                for (let i = 0; i < 6; i++) {
                    const spike = new THREE.Mesh(
                        new THREE.ConeGeometry(0.1, 0.8, 6),
                        new THREE.MeshLambertMaterial({ color: 0x111122 })
                    );
                    spike.position.set(
                        (Math.random() - 0.5) * 1.4,
                        2 + Math.random() * 0.5,
                        -0.2 + Math.random() * 0.4
                    );
                    spike.rotation.x = Math.PI / 2;
                    this.group.add(spike);
                }

                this.group.position.set(x, 0, z);
                this.group.userData = { type: 'shadowbeast', ref: this };
                scene.add(this.group);
                enemies.push(this);

                this.createHealthBar();
            }

            createHealthBar() {
                const healthBarContainer = document.createElement('div');
                healthBarContainer.className = 'enemy-health-bar';
                
                const healthFill = document.createElement('div');
                healthFill.className = 'enemy-health-fill';
                
                const healthText = document.createElement('div');
                healthText.textContent = `${this.health}/${this.maxHealth}`;
                
                healthBarContainer.appendChild(healthFill);
                healthBarContainer.appendChild(healthText);
                document.body.appendChild(healthBarContainer);

                this.healthBar = healthFill;
                this.healthText = healthText;
                this.healthBarContainer = healthBarContainer;
            }

            updateHealthBar() {
                if (!this.healthBar || !this.healthText) return;
                
                const percent = (this.health / this.maxHealth) * 100;
                this.healthBar.style.width = `${percent}%`;
                this.healthText.textContent = `${Math.ceil(this.health)}/${this.maxHealth}`;
                
                if (percent > 60) {
                    this.healthBar.style.background = 'linear-gradient(90deg, #00ff00, #88ff00)';
                } else if (percent > 30) {
                    this.healthBar.style.background = 'linear-gradient(90deg, #ff8800, #ffaa00)';
                } else {
                    this.healthBar.style.background = 'linear-gradient(90deg, #ff0000, #ff4400)';
                }
            }

            updateHealthBarPosition() {
                if (!this.healthBarContainer) return;

                const screenPos = this.group.position.clone();
                screenPos.project(camera);
                
                const rect = renderer.domElement.getBoundingClientRect();
                this.healthBarContainer.style.left = `${rect.left + screenPos.x * rect.width * 0.5 + rect.width / 2}px`;
                this.healthBarContainer.style.top = `${rect.top + (-screenPos.y + 1) * rect.height * 0.5}px`;
                this.healthBarContainer.style.display = screenPos.z < 1 ? 'block' : 'none';
            }

            update(deltaTime, playerPos) {
                this.updateHealthBarPosition();

                if (!gameRunning) return;
                
                const dist = this.group.position.distanceTo(playerPos);
                const speedMultiplier = isDaytime ? this.daytimeSpeedMultiplier : 1.0;
                
                if (dist < 40) {
                    const direction = new THREE.Vector3()
                        .subVectors(playerPos, this.group.position)
                        .normalize();
                    this.group.position.add(direction.multiplyScalar(this.speed * speedMultiplier * deltaTime));

                    this.floatingHeight += Math.sin(clock.getElapsedTime() * 2 + this.group.position.x) * 0.5 * deltaTime;
                    this.group.position.y = 0.5 + Math.abs(this.floatingHeight) * 0.8;

                    if (dist < 3 && clock.getElapsedTime() - this.lastAttack > 1.8) {
                        window.health -= 8;
                        this.lastAttack = clock.getElapsedTime();
                    }
                }

                this.group.rotation.y += 0.03;
                const t = clock.getElapsedTime();
                
                for (let i = 1; i <= 4; i++) {
                    if (this.group.children[i]) {
                        this.group.children[i].rotation.z += Math.sin(t * 3 + i) * 0.1 * deltaTime;
                    }
                }
            }

            takeDamage(amount) {
                this.health -= amount;
                this.updateHealthBar();
                
                if (this.health <= 0) {
                    enemyKillCount++;
                    updateQuestUI();
                    
                    if (enemyKillCount >= QUEST_GOAL) {
                        completeQuest();
                    }
                    
                    if (this.healthBarContainer) {
                        this.healthBarContainer.remove();
                    }
                    scene.remove(this.group);
                    enemies.splice(enemies.indexOf(this), 1);
                    window.cash += 15;
                }
            }
        }

        function updateQuestUI() {
            const progressEl = document.getElementById('killProgress1');
            if (progressEl) {
                progressEl.textContent = `${enemyKillCount}/${QUEST_GOAL}`;
            }
        }

        function completeQuest() {
            const questEl = document.getElementById('killQuest1');
            if (questEl) {
                questEl.classList.remove('active');
                questEl.classList.add('completed');
            }
            window.level += QUEST_REWARD;
            const questUI = document.getElementById('quests');
            questUI.style.background = 'rgba(0,255,0,0.9)';
            questUI.style.boxShadow = '0 0 30px #00ff00';
            setTimeout(() => {
                questUI.style.background = 'rgba(0,0,0,0.9)';
                questUI.style.boxShadow = 'none';
            }, 2000);
        }

        for (let i = 0; i < 8; i++) {
            const angle = (i / 8) * Math.PI * 2;
            const dist = 45 + Math.random() * 15;
            new ShadowBeast(Math.cos(angle) * dist, Math.sin(angle) * dist);
        }

        window.woodCount = 0;
        window.stoneCount = 0;
        window.fistCount = 1;
        window.axeCount = 1;
        window.cash = 0;
        window.level = 1;
        window.health = 100;
        window.maxHealth = 100;

        let fistEquipped = true;
        let axeEquipped = false;
        let canPunch = true;
        let punchCooldown = 0;
        let dayNightTimer = 0;
        let isDaytime = true;
        let dayCount = 1;

        const FIST_DAMAGE = 5;
        const AXE_DAMAGE = 15;
        const DEATH_PUNCH_DAMAGE = 20;
        const DEATH_PUNCH_RANGE = 5;

        const woodEl = document.getElementById('woodCount');
        const stoneEl = document.getElementById('stoneCount');
        const fistEl = document.getElementById('fistCount');
        const axeEl = document.getElementById('axeCount');
        const cashEl = document.getElementById('cash');
        const levelEl = document.getElementById('level');
        const healthTextEl = document.getElementById('healthText');
        const healthFillEl = document.getElementById('healthFill');
        const timeDisplay = document.getElementById('timeDisplay');
        const hotbarSlots = document.querySelectorAll('.hotbarSlot');
        const leftHand = document.getElementById('leftHand');
        const rightHand = document.getElementById('rightHand');
        const deathScreen = document.getElementById('deathScreen');
        const restartBtn = document.getElementById('restartBtn');
        const dayCountEl = document.getElementById('dayCount');

        document.addEventListener('keydown', (e) => {
            if (e.code === 'KeyR' && !gameRunning) {
                restartGame();
                return;
            }
            if (gameRunning) {
                keys[e.code] = true;
                if (e.code === 'Digit0') {
                    fistEquipped = false;
                    axeEquipped = false;
                    hotbarSlots.forEach(slot => slot.classList.remove('selected'));
                    hotbarSlots[0].classList.add('selected');
                    leftHand.classList.remove('fist-equipped');
                    rightHand.classList.remove('fist-equipped');
                } else if (e.code === 'Digit1' && window.fistCount > 0) {
                    fistEquipped = true;
                    axeEquipped = false;
                    hotbarSlots.forEach(slot => slot.classList.remove('selected'));
                    hotbarSlots[1].classList.add('selected');
                    leftHand.classList.add('fist-equipped');
                    rightHand.classList.add('fist-equipped');
                } else if (e.code === 'Digit2' && window.axeCount > 0) {
                    fistEquipped = false;
                    axeEquipped = true;
                    hotbarSlots.forEach(slot => slot.classList.remove('selected'));
                    hotbarSlots[2].classList.add('selected');
                    leftHand.classList.add('fist-equipped');
                    rightHand.classList.add('fist-equipped');
                }
            }
        });

        document.addEventListener('keyup', (e) => {
            if (gameRunning) keys[e.code] = false;
        });

        let mouseDown = false;
        let spinStartYaw = 0;
        document.addEventListener('mousedown', () => mouseDown = true);
        document.addEventListener('mouseup', () => {
            if (!gameRunning) return;
            mouseDown = false;
            if (Math.abs(totalYawRotation - spinStartYaw) >= Math.PI * 2 && !fistEquipped && !axeEquipped) {
                performSpinAttack();
                totalYawRotation = 0;
            }
        });

        function performSpinAttack() {
            const playerPos = controls.getObject().position;
            enemies.forEach(enemy => {
                const dist = enemy.group.position.distanceTo(playerPos);
                if (dist < 8) {
                    enemy.takeDamage(FIST_DAMAGE * 2);
                }
            });
            
            leftHand.style.animation = 'punchAnim 0.5s ease-out';
            rightHand.style.animation = 'punchAnim 0.5s ease-out';
            setTimeout(() => {
                leftHand.style.animation = '';
                rightHand.style.animation = '';
            }, 500);
        }

        const raycaster = new THREE.Raycaster();
        const mouse = new THREE.Vector2(0, 0);

        document.addEventListener('click', () => {
            if (!gameRunning || !controls.isLocked) return;
            if (!canPunch) return;

            raycaster.setFromCamera(mouse, camera);
            const intersects = raycaster.intersectObjects(scene.children, true);

            if (intersects.length > 0) {
                const obj = intersects[0].object;
                const target = obj.userData.type ? obj : obj.parent;

                if (fistEquipped) {
                    if (target?.userData?.type === 'tree') {
                        target.userData.ref.takeDamage(FIST_DAMAGE);
                    } else if (target?.userData?.type === 'stone') {
                        target.userData.health -= FIST_DAMAGE;
                        if (target.userData.health <= 0) {
                            scene.remove(target);
                            window.stoneCount += 1;
                        }
                    } else if (target?.userData?.type === 'shadowbeast') {
                        target.userData.ref.takeDamage(FIST_DAMAGE);
                    }

                    const playerPos = controls.getObject().position;
                    const distToPlayer = intersects[0].point.distanceTo(playerPos);
                    if (distToPlayer < 3) {
                        window.health -= FIST_DAMAGE * 0.5;
                    }
                } else if (axeEquipped) {
                    if (target?.userData?.type === 'tree') {
                        target.userData.ref.takeDamage(AXE_DAMAGE);
                    }
                    // axe does nothing to stone or enemies, no self‑damage
                }

                if (fistEquipped || axeEquipped) {
                    leftHand.classList.add('punch');
                    rightHand.classList.add('punch');
                    setTimeout(() => {
                        leftHand.classList.remove('punch');
                        rightHand.classList.remove('punch');
                    }, 300);
                }
            }

            if (fistEquipped || axeEquipped) {
                canPunch = false;
                punchCooldown = 0.8;
            }
        });

        function deathPunch() {
            const playerPos = controls.getObject().position;
            enemies.forEach(enemy => {
                const dist = enemy.group.position.distanceTo(playerPos);
                if (dist < DEATH_PUNCH_RANGE) {
                    enemy.takeDamage(DEATH_PUNCH_DAMAGE);
                }
            });

            leftHand.classList.add('death-punch');
            rightHand.classList.add('death-punch');
            setTimeout(() => {
                leftHand.classList.remove('death-punch');
                rightHand.classList.remove('death-punch');
            }, 1000);
        }

        function handleDeath() {
            gameRunning = false;
            deathPunch();
            
            setTimeout(() => {
                deathScreen.style.display = 'block';
                controls.unlock();
                document.body.style.cursor = 'default';
            }, 500);
        }

        restartBtn.addEventListener('click', restartGame);

        function restartGame() {
            window.health = 100;
            window.woodCount = 0;
            window.stoneCount = 0;
            window.cash = 0;
            window.level = 1;
            enemyKillCount = 0;
            dayNightTimer = 0;
            isDaytime = true;
            dayCount = 1;
            if (dayCountEl) dayCountEl.textContent = dayCount;

            const questEl = document.getElementById('killQuest1');
            if (questEl) {
                questEl.classList.add('active');
                questEl.classList.remove('completed');
                document.getElementById('killProgress1').textContent = '0/5';
            }
            
            enemies.forEach(enemy => {
                if (enemy.healthBarContainer) enemy.healthBarContainer.remove();
                scene.remove(enemy.group);
            });
            enemies.length = 0;
            
            document.querySelectorAll('.enemy-health-bar').forEach(bar => bar.remove());
            
            for (let i = 0; i < 8; i++) {
                const angle = (i / 8) * Math.PI * 2;
                const dist = 45 + Math.random() * 15;
                new ShadowBeast(Math.cos(angle) * dist, Math.sin(angle) * dist);
            }
            
            controls.getObject().position.set(0, 1.8, 0);
            gameRunning = true;
            deathScreen.style.display = 'none';
            controls.lock();
            document.body.style.cursor = '';
            
            fistEquipped = true;
            axeEquipped = false;
            canPunch = true;
            totalYawRotation = 0;
        }

        function updateUI() {
            woodEl.textContent = window.woodCount;
            stoneEl.textContent = window.stoneCount;
            fistEl.textContent = window.fistCount;
            axeEl.textContent = window.axeCount;
            cashEl.textContent = window.cash;
            levelEl.textContent = window.level;
            healthTextEl.textContent = `${Math.floor(window.health)}/${window.maxHealth}`;
            healthFillEl.style.width = `${(window.health / window.maxHealth) * 100}%`;
            timeDisplay.textContent = isDaytime ? 'Daytime' : 'NIGHT';
        }

        const clock = new THREE.Clock();
        const playerPos = controls.getObject().position;

        function animate() {
            requestAnimationFrame(animate);
            const deltaTime = Math.min(clock.getDelta(), 0.016);

            if (!gameRunning) {
                renderer.render(scene, camera);
                enemies.forEach(enemy => enemy.update(deltaTime, playerPos));
                return;
            }

            const currentYaw = controls.getObject().rotation.y;
            totalYawRotation += currentYaw - lastYaw;
            lastYaw = currentYaw;
            if (mouseDown && !fistEquipped && !axeEquipped) spinStartYaw = totalYawRotation;

            const moveDirection = new THREE.Vector3();
            const frontVector = new THREE.Vector3(0, 0, -1);
            controls.getDirection(frontVector);
            frontVector.y = 0;
            frontVector.normalize();

            const rightVector = new THREE.Vector3()
                .crossVectors(frontVector, new THREE.Vector3(0, 1, 0))
                .normalize();

            if (keys['KeyW']) moveDirection.add(frontVector);
            if (keys['KeyS']) moveDirection.sub(frontVector);
            if (keys['KeyD']) moveDirection.add(rightVector);
            if (keys['KeyA']) moveDirection.sub(rightVector);

            if (moveDirection.length() > 0) {
                moveDirection.normalize();
                velocity.x = moveDirection.x * moveSpeed;
                velocity.z = moveDirection.z * moveSpeed;
            } else {
                velocity.multiplyScalar(0.9);
            }

            const newX = playerPos.x + velocity.x * deltaTime;
            const newZ = playerPos.z + velocity.z * deltaTime;

            if (Math.abs(newX) <= boundarySize - 2) playerPos.x = newX;
            if (Math.abs(newZ) <= boundarySize - 2) playerPos.z = newZ;

            if (keys['Space'] && canJump) {
                velocity.y = 12;
                canJump = false;
            }
            velocity.y -= 30 * deltaTime;
            playerPos.y += velocity.y * deltaTime;
            if (playerPos.y < 1.8) {
                velocity.y = 0;
                playerPos.y = 1.8;
                canJump = true;
            }

            const PHASE_LENGTH = 60;
            dayNightTimer += deltaTime;

            if (dayNightTimer > PHASE_LENGTH) {
                isDaytime = !isDaytime;
                dayNightTimer = 0;
                if (isDaytime) {
                    dayCount++;
                    if (dayCountEl) dayCountEl.textContent = dayCount;
                }
            }

            const cycleProgress = dayNightTimer / PHASE_LENGTH;
            directionalLight.position.set(
                60 * Math.cos(cycleProgress * Math.PI),
                60 * Math.sin(cycleProgress * Math.PI) + 10,
                60 * Math.sin(cycleProgress * Math.PI)
            );
            directionalLight.intensity = isDaytime ? 1.2 : 0.2;
            ambientLight.intensity = isDaytime ? 0.8 : 0.2;
            stars.visible = !isDaytime;

            enemies.forEach(enemy => enemy.update(deltaTime, playerPos));

            if (!canPunch) {
                punchCooldown -= deltaTime;
                if (punchCooldown <= 0) canPunch = true;
            }

            if (fistEquipped || axeEquipped) {
                const time = clock.getElapsedTime();
                leftHand.style.transform = `rotate(-45deg) scale(1.1) translateY(${Math.sin(time * 3) * 3}px)`;
                rightHand.style.transform = `rotate(45deg) scale(1.1) translateY(${Math.sin(time * 3 + Math.PI) * 3}px)`;
            }

            window.health = Math.max(0, Math.min(window.maxHealth, window.health));
            if (window.health <= 0) {
                handleDeath();
            }

            renderer.render(scene, camera);
            updateUI();
        }

        animate();

        window.addEventListener('resize', () => {
            camera.aspect = window.innerWidth / window.innerHeight;
            camera.updateProjectionMatrix();
            renderer.setSize(window.innerWidth, window.innerHeight);
        });
    </script>
</body>
</html>
