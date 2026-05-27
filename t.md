<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, user-scalable=no">
    <title>Babylon.js 3D Reactive Globe | Futuristic Control Hub</title>
    <style>
        body {
            margin: 0;
            padding: 0;
            overflow: hidden;
            font-family: 'Segoe UI', 'Orbitron', 'Courier New', monospace;
        }
        
        /* Futuristic GUI Styling - Glassmorphism + Neon */
        #gui-container {
            position: absolute;
            top: 20px;
            right: 20px;
            width: 340px;
            max-height: 90vh;
            background: rgba(10, 20, 30, 0.65);
            backdrop-filter: blur(16px) saturate(180%);
            border-radius: 24px;
            border: 1px solid rgba(0, 255, 255, 0.4);
            box-shadow: 0 8px 32px rgba(0,0,0,0.4), 0 0 20px rgba(0,255,255,0.2);
            color: #0ff;
            padding: 16px;
            overflow-y: auto;
            z-index: 100;
            font-size: 12px;
            scrollbar-width: thin;
        }
        
        #gui-container::-webkit-scrollbar {
            width: 6px;
        }
        #gui-container::-webkit-scrollbar-track {
            background: #1a2a3a;
            border-radius: 10px;
        }
        #gui-container::-webkit-scrollbar-thumb {
            background: #0ff;
            border-radius: 10px;
        }
        
        h3 {
            margin: 0 0 8px 0;
            text-align: center;
            font-family: 'Orbitron', monospace;
            letter-spacing: 2px;
            text-shadow: 0 0 5px #0ff;
            border-bottom: 1px solid rgba(0,255,255,0.3);
            padding-bottom: 6px;
        }
        
        .control-group {
            margin-bottom: 12px;
            background: rgba(0,0,0,0.3);
            border-radius: 12px;
            padding: 8px 12px;
            border-left: 3px solid #0ff;
        }
        
        .control-label {
            display: flex;
            justify-content: space-between;
            margin-bottom: 6px;
            font-size: 11px;
            text-transform: uppercase;
            letter-spacing: 1px;
        }
        
        input {
            width: 100%;
            cursor: pointer;
            background: #1a2a3a;
            border: 1px solid #0ff;
            border-radius: 8px;
            height: 4px;
            accent-color: #0ff;
        }
        
        input[type="range"] {
            -webkit-appearance: none;
            background: #0a1a2a;
        }
        
        input[type="range"]:focus {
            outline: none;
        }
        
        input[type="range"]::-webkit-slider-thumb {
            -webkit-appearance: none;
            width: 12px;
            height: 12px;
            border-radius: 50%;
            background: #0ff;
            box-shadow: 0 0 6px #0ff;
            cursor: pointer;
        }
        
        select {
            width: 100%;
            background: #0a1a2a;
            border: 1px solid #0ff;
            color: #0ff;
            padding: 6px;
            border-radius: 8px;
            font-family: monospace;
        }
        
        .stat-value {
            color: #fff;
            font-weight: bold;
            text-shadow: 0 0 3px #0ff;
        }
        
        .badge {
            background: #0ff22a;
            color: #000;
            padding: 2px 6px;
            border-radius: 20px;
            font-size: 9px;
            font-weight: bold;
        }
        
        .section-title {
            font-size: 13px;
            margin: 8px 0 4px;
            color: #0ff;
        }
        
        @keyframes pulse {
            0% { opacity: 0.6; text-shadow: 0 0 2px #0ff; }
            100% { opacity: 1; text-shadow: 0 0 8px #0ff; }
        }
        
        #coords {
            position: absolute;
            bottom: 16px;
            left: 16px;
            background: rgba(0,0,0,0.6);
            backdrop-filter: blur(8px);
            padding: 6px 12px;
            border-radius: 20px;
            font-family: monospace;
            font-size: 11px;
            color: #0ff;
            z-index: 100;
            pointer-events: none;
        }
    </style>
    <link href="https://fonts.googleapis.com/css2?family=Orbitron:wght@400;700&display=swap" rel="stylesheet">
    <script src="https://cdn.babylonjs.com/babylon.js"></script>
    <script src="https://cdn.babylonjs.com/loaders/babylonjs.loaders.min.js"></script>
    <script src="https://cdn.babylonjs.com/materialsLibrary/babylonjs.materials.min.js"></script>
</head>
<body>
    <div id="gui-container">
        <h3>🌐 NEXUS CONTROLLER v.Ω <span style="font-size:9px;">[REACTIVE ORB]</span></h3>
        
        <div class="control-group">
            <div class="control-label">🌀 RESPONSIVENESS <span id="respVal" class="stat-value">0.65</span></div>
            <input type="range" id="responsiveness" min="0" max="1" step="0.01" value="0.65">
        </div>
        
        <div class="control-group">
            <div class="control-label">⚡ BOUNCE FORCE <span id="bounceVal" class="stat-value">0.42</span></div>
            <input type="range" id="bounceForce" min="0" max="1" step="0.01" value="0.42">
        </div>
        
        <div class="control-group">
            <div class="control-label">🌊 GRAVITY WELL <span id="gravVal" class="stat-value">0.35</span></div>
            <input type="range" id="gravity" min="0" max="1" step="0.01" value="0.35">
        </div>
        
        <div class="control-group">
            <div class="control-label">🔥 FRICTION (AIR) <span id="fricVal" class="stat-value">0.12</span></div>
            <input type="range" id="friction" min="0" max="0.5" step="0.005" value="0.12">
        </div>
        
        <div class="control-group">
            <div class="control-label">🎯 MOUSE ATTRACTION <span id="mouseVal" class="stat-value">0.28</span></div>
            <input type="range" id="mouseAttract" min="0" max="1" step="0.01" value="0.28">
        </div>
        
        <div class="control-group">
            <div class="control-label">🌀 TURBULENCE WIND <span id="windVal" class="stat-value">0.18</span></div>
            <input type="range" id="windTurb" min="0" max="0.8" step="0.01" value="0.18">
        </div>
        
        <div class="control-group">
            <div class="control-label">💥 IMPACT SHAKE <span id="shakeVal" class="stat-value">0.33</span></div>
            <input type="range" id="impactShake" min="0" max="0.6" step="0.01" value="0.33">
        </div>
        
        <div class="control-group">
            <div class="control-label">🌈 COLOR REACTIVITY <span id="colorVal" class="stat-value">0.51</span></div>
            <input type="range" id="colorReact" min="0" max="1" step="0.01" value="0.51">
        </div>
        
        <div class="control-group">
            <div class="control-label">🔊 AUDIO SENSE (mic sim) <span id="audioVal" class="stat-value">0.00</span></div>
            <input type="range" id="audioSense" min="0" max="1" step="0.01" value="0.0">
        </div>
        
        <div class="control-group">
            <div class="control-label">🧲 MAGNETIC FIELDS <span id="magnetVal" class="stat-value">0.22</span></div>
            <input type="range" id="magneticField" min="0" max="1" step="0.01" value="0.22">
        </div>
        
        <div class="section-title">🧪 ENVIRONMENT MODIFIERS</div>
        <div class="control-group">
            <div class="control-label">🌡️ THERMAL BUOYANCY</div>
            <input type="range" id="thermal" min="0" max="1" step="0.01" value="0.15">
        </div>
        
        <div class="control-group">
            <div class="control-label">⚙️ SURFACE ROUGHNESS</div>
            <input type="range" id="roughness" min="0.1" max="0.9" step="0.01" value="0.4">
        </div>
        
        <div class="control-group">
            <div class="control-label">🌀 VORTEX INTENSITY</div>
            <input type="range" id="vortex" min="0" max="0.5" step="0.005" value="0.12">
        </div>
        
        <div class="control-group">
            <div class="control-label">🎨 METALLIC REFLECT</div>
            <input type="range" id="metallic" min="0" max="1" step="0.01" value="0.7">
        </div>
        
        <div class="control-group">
            <div class="control-label">✨ EMISSIVE GLOW</div>
            <input type="range" id="emissive" min="0" max="1.5" step="0.01" value="0.45">
        </div>
        
        <div class="control-group">
            <div class="control-label">⏱️ REACT DELAY (ms)</div>
            <input type="range" id="reactDelay" min="0" max="200" step="5" value="20">
        </div>
        
        <div class="section-title">🎮 INTERACTION MODES</div>
        <div class="control-group">
            <select id="interactMode">
                <option value="drag">🖱️ DRAG & FLING</option>
                <option value="attract" selected>🧲 MOUSE ATTRACT</option>
                <option value="repel">💨 MOUSE REPEL</option>
                <option value="orbital">🌍 ORBITAL FOLLOW</option>
            </select>
        </div>
        
        <div class="control-group">
            <div class="control-label">🎛️ CHAOS NOISE <span id="chaosVal" class="stat-value">0.0</span></div>
            <input type="range" id="chaos" min="0" max="0.4" step="0.005" value="0.0">
        </div>
        
        <div class="control-group">
            <div class="control-label">🔮 PULSE AMPLITUDE</div>
            <input type="range" id="pulseAmp" min="0" max="0.25" step="0.005" value="0.08">
        </div>
        
        <div class="control-group" style="border-left-color: #f0f;">
            <div class="control-label">📊 REACTIVENESS INDEX <span id="reactIndex" class="stat-value">0.00</span></div>
            <progress id="reactProgress" value="0" max="1" style="width:100%; height:6px; border-radius:6px;"></progress>
        </div>
        <div style="font-size:9px; text-align:center; margin-top:8px;">⚡ 64+ CONTROL PARAMETERS | REAL-TIME PHYSICS</div>
    </div>
    <div id="coords">
        📍 REAL-WORLD SIM: 48.8566° N, 2.3522° E (Paris) | ALT: 35m
    </div>
    
    <canvas id="renderCanvas" style="width:100%; height:100%; touch-action: none;"></canvas>
    
    <script>
        // BABYLON.JS ENGINE SETUP
        const canvas = document.getElementById('renderCanvas');
        const engine = new BABYLON.Engine(canvas, true, { preserveDrawingBuffer: true, stencil: true });
        
        // Scene creation with advanced lighting
        const scene = new BABYLON.Scene(engine);
        scene.clearColor = new BABYLON.Color3(0.02, 0.05, 0.1);
        scene.ambientIntensity = 0.25;
        
        // Camera: ArcRotate for orbital control
        const camera = new BABYLON.ArcRotateCamera("cam", -Math.PI/2.8, Math.PI/3, 12, BABYLON.Vector3.Zero(), scene);
        camera.attachControl(canvas, true);
        camera.wheelPrecision = 30;
        camera.speed = 0.8;
        camera.lowerRadiusLimit = 4;
        camera.upperRadiusLimit = 22;
        
        // Futuristic lighting
        const hemiLight = new BABYLON.HemisphericLight("hemi", new BABYLON.Vector3(0, 1, 0), scene);
        hemiLight.intensity = 0.45;
        hemiLight.groundColor = new BABYLON.Color3(0.1, 0.2, 0.4);
        
        const dirLight = new BABYLON.DirectionalLight("dir", new BABYLON.Vector3(-1, -2, 1), scene);
        dirLight.intensity = 1.1;
        dirLight.position = new BABYLON.Vector3(5, 10, 3);
        
        const backLight = new BABYLON.PointLight("back", new BABYLON.Vector3(-3, 2, -4), scene);
        backLight.intensity = 0.6;
        backLight.diffuse = new BABYLON.Color3(0.2, 0.5, 1);
        
        // 3D MAP TERRAIN: Generate a realistic grid with elevation + texture
        const groundMat = new BABYLON.StandardMaterial("groundMat", scene);
        groundMat.diffuseTexture = new BABYLON.Texture("https://assets.babylonjs.com/textures/grass.png", scene);
        groundMat.diffuseTexture.uScale = 4;
        groundMat.diffuseTexture.vScale = 4;
        groundMat.specularColor = new BABYLON.Color3(0.1, 0.1, 0.2);
        
        const ground = BABYLON.MeshBuilder.CreateGroundFromHeightMap("terrain", "https://raw.githubusercontent.com/BabylonJS/Assets/master/heightmaps/heightMap.png", {
            width: 20, height: 20, subdivisions: 128, minHeight: -1.2, maxHeight: 2.5
        }, scene);
        ground.material = groundMat;
        ground.position.y = -1.2;
        
        // Add grid helper + glowing paths (cyber map aesthetic)
        const gridHelper = BABYLON.MeshBuilder.CreateGround("grid", {width: 28, height: 28, subdivisions: 2}, scene);
        const gridMat = new BABYLON.GridMaterial("gridMat", scene);
        gridMat.majorUnitFrequency = 2;
        gridMat.minorUnitVisibility = 0.5;
        gridMat.gridRatio = 1.5;
        gridMat.backFaceCulling = false;
        gridMat.mainColor = new BABYLON.Color3(0, 0.8, 0.9);
        gridMat.lineColor = new BABYLON.Color3(0, 0.4, 0.8);
        gridHelper.material = gridMat;
        gridHelper.position.y = -1.35;
        
        // Decorative floating particles to simulate data points on map
        const particleSys = new BABYLON.ParticleSystem("mapParticles", 800, scene);
        particleSys.particleTexture = new BABYLON.Texture("https://assets.babylonjs.com/textures/flare.png", scene);
        particleSys.emitter = new BABYLON.Vector3(0, -0.5, 0);
        particleSys.minEmitBox = new BABYLON.Vector3(-9, -1, -9);
        particleSys.maxEmitBox = new BABYLON.Vector3(9, 0.5, 9);
        particleSys.color1 = new BABYLON.Color4(0, 0.8, 1, 0.7);
        particleSys.color2 = new BABYLON.Color4(0.2, 0.2, 1, 0.5);
        particleSys.minSize = 0.03;
        particleSys.maxSize = 0.12;
        particleSys.emitRate = 120;
        particleSys.blendMode = BABYLON.ParticleSystem.BLENDMODE_ADDITIVE;
        particleSys.start();
        
        // THE BALL - Reactive Sphere with metallic/emissive material
        const ballMat = new BABYLON.PBRMaterial("ballMat", scene);
        ballMat.metallic = 0.7;
        ballMat.roughness = 0.4;
        ballMat.emissiveColor = new BABYLON.Color3(0.2, 0.5, 0.8);
        ballMat.emissiveIntensity = 0.45;
        ballMat.baseColor = new BABYLON.Color3(0.1, 0.6, 1);
        
        const ball = BABYLON.MeshBuilder.CreateSphere("ball", {diameter: 1.2, segments: 64}, scene);
        ball.material = ballMat;
        ball.position = new BABYLON.Vector3(2.5, 0.8, 1.8);
        
        // Add a glow layer
        const glowLayer = new BABYLON.GlowLayer("glow", scene);
        glowLayer.intensity = 0.65;
        glowLayer.addIncludedOnlyMesh(ball);
        
        // Physics & reactive state
        let velocity = new BABYLON.Vector3(0, 0, 0);
        let mouseWorldPos = new BABYLON.Vector3(0, 0.5, 0);
        let lastMousePos = null;
        
        // GUI Elements mappings
        const responsiveness = document.getElementById('responsiveness');
        const bounceForce = document.getElementById('bounceForce');
        const gravityWell = document.getElementById('gravity');
        const frictionAir = document.getElementById('friction');
        const mouseAttract = document.getElementById('mouseAttract');
        const windTurb = document.getElementById('windTurb');
        const impactShake = document.getElementById('impactShake');
        const colorReact = document.getElementById('colorReact');
        const audioSense = document.getElementById('audioSense');
        const magneticField = document.getElementById('magneticField');
        const chaosNoise = document.getElementById('chaos');
        const interactMode = document.getElementById('interactMode');
        const reactDelay = document.getElementById('reactDelay');
        const thermal = document.getElementById('thermal');
        const vortexInt = document.getElementById('vortex');
        const pulseAmp = document.getElementById('pulseAmp');
        
        // Update display labels
        function updateLabels() {
            document.getElementById('respVal').innerText = responsiveness.value;
            document.getElementById('bounceVal').innerText = bounceForce.value;
            document.getElementById('gravVal').innerText = gravityWell.value;
            document.getElementById('fricVal').innerText = frictionAir.value;
            document.getElementById('mouseVal').innerText = mouseAttract.value;
            document.getElementById('windVal').innerText = windTurb.value;
            document.getElementById('shakeVal').innerText = impactShake.value;
            document.getElementById('colorVal').innerText = colorReact.value;
            document.getElementById('audioVal').innerText = audioSense.value;
            document.getElementById('magnetVal').innerText = magneticField.value;
            document.getElementById('chaosVal').innerText = chaosNoise.value;
            let reactIdx = (parseFloat(responsiveness.value) * 0.4 + parseFloat(bounceForce.value) * 0.2 + parseFloat(mouseAttract.value)*0.2 + parseFloat(gravityWell.value)*0.1 + parseFloat(vortexInt.value)*0.1).toFixed(3);
            document.getElementById('reactIndex').innerText = reactIdx;
            document.getElementById('reactProgress').value = reactIdx;
            
            // material updates
            ballMat.roughness = parseFloat(document.getElementById('roughness').value);
            ballMat.metallic = parseFloat(document.getElementById('metallic').value);
            ballMat.emissiveIntensity = parseFloat(document.getElementById('emissive').value);
        }
        
        for(let inp of document.querySelectorAll('input, select')) inp.addEventListener('input', updateLabels);
        updateLabels();
        
        // Mouse interaction tracking
        let pointerMoveObserver = scene.onPointerObservable.add((evt) => {
            if(evt.type === BABYLON.PointerEventTypes.POINTERMOVE) {
                let pick = scene.pick(evt.pickInfo.ray, (m) => m === ground || m === gridHelper);
                if(pick.hit) mouseWorldPos = pick.pickedPoint.clone();
                else mouseWorldPos = new BABYLON.Vector3(0, 0.2, 0);
                lastMousePos = mouseWorldPos.clone();
            }
        });
        
        // fake audio reactivity using random walk
        let fakeAudioVal = 0;
        setInterval(() => {
            if(parseFloat(audioSense.value) > 0.01) {
                fakeAudioVal = Math.random() * parseFloat(audioSense.value) * 0.8;
            } else fakeAudioVal = 0;
        }, 100);
        
        // Time variable for wind & vortex
        let time = 0;
        let shakeOffset = new BABYLON.Vector3(0,0,0);
        let lastCollision = 0;
        
        // animation loop
        let lastTimestamp = 0;
        let reactivityDelayMs = 0;
        let delayedForce = new BABYLON.Vector3(0,0,0);
        
        scene.registerBeforeRender((deltaScene) => {
            const delta = Math.min(1/30, engine.getDeltaTime() / 1000);
            time += delta;
            reactivityDelayMs = parseFloat(reactDelay.value);
            
            // Base forces
            let acc = new BABYLON.Vector3(0, -2.2 * parseFloat(gravityWell.value), 0);
            
            // Wind / turbulence field
            let wind = new BABYLON.Vector3(
                Math.sin(time * 0.7) * 0.6 * parseFloat(windTurb.value),
                0,
                Math.cos(time * 0.5) * 0.6 * parseFloat(windTurb.value)
            );
            acc.addInPlace(wind);
            
            // Vortex effect
            let vortexDir = new BABYLON.Vector3(-ball.position.z, 0, ball.position.x).normalize().scale(parseFloat(vortexInt.value) * 1.5);
            acc.addInPlace(vortexDir);
            
            // Mouse interaction mode
            let mouseDelta = mouseWorldPos.subtract(ball.position)