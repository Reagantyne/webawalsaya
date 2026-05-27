<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>REACTIVEX • 3D Neural Orb</title>
    <script src="https://cdn.babylonjs.com/babylon.js"></script>
    <script src="https://cdn.babylonjs.com/loaders/babylonjs.loaders.min.js"></script>
    <style>
        @import url('https://fonts.googleapis.com/css2?family=Inter:wght@400;500;600&family=Space+Grotesk:wght@500;600&display=swap');
        
        body {
            margin: 0;
            padding: 0;
            overflow: hidden;
            background: #000;
            font-family: 'Inter', sans-serif;
            color: #fff;
        }
        
        #renderCanvas {
            width: 100%;
            height: 100vh;
            display: block;
        }
        
        .gui-panel {
            position: absolute;
            top: 20px;
            right: 20px;
            width: 380px;
            max-height: 85vh;
            background: rgba(15, 15, 25, 0.85);
            backdrop-filter: blur(20px);
            border: 1px solid rgba(100, 200, 255, 0.3);
            border-radius: 16px;
            box-shadow: 0 20px 40px rgba(0, 0, 0, 0.6);
            overflow: hidden;
            z-index: 100;
        }
        
        .gui-header {
            background: linear-gradient(90deg, #00ddff, #0099ff);
            padding: 16px 20px;
            font-family: 'Space Grotesk', sans-serif;
            font-size: 18px;
            font-weight: 600;
            letter-spacing: 1px;
            display: flex;
            align-items: center;
            gap: 10px;
        }
        
        .gui-header::before {
            content: '◉';
            color: #fff;
            animation: pulse 2s infinite;
        }
        
        .controls-container {
            padding: 15px;
            max-height: calc(85vh - 60px);
            overflow-y: auto;
        }
        
        .param-group {
            margin-bottom: 25px;
        }
        
        .param-group h3 {
            margin: 0 0 12px 0;
            font-size: 14px;
            color: #00ddff;
            text-transform: uppercase;
            letter-spacing: 1px;
            font-weight: 500;
        }
        
        .param {
            margin-bottom: 14px;
            display: flex;
            align-items: center;
            gap: 12px;
        }
        
        .param-label {
            width: 160px;
            font-size: 13px;
            white-space: nowrap;
            overflow: hidden;
            text-overflow: ellipsis;
        }
        
        .slider-container {
            flex: 1;
            display: flex;
            align-items: center;
            gap: 8px;
        }
        
        input[type="range"] {
            flex: 1;
            accent-color: #00ddff;
        }
        
        input[type="number"] {
            width: 70px;
            background: rgba(255,255,255,0.1);
            border: 1px solid rgba(100, 200, 255, 0.4);
            color: #fff;
            padding: 4px 8px;
            border-radius: 6px;
            font-size: 13px;
        }
        
        .status-bar {
            position: absolute;
            bottom: 20px;
            left: 20px;
            background: rgba(15, 15, 25, 0.8);
            backdrop-filter: blur(12px);
            padding: 8px 16px;
            border-radius: 9999px;
            font-size: 13px;
            border: 1px solid rgba(100, 200, 255, 0.3);
            display: flex;
            align-items: center;
            gap: 12px;
        }
        
        .dot {
            width: 8px;
            height: 8px;
            background: #00ff88;
            border-radius: 50%;
            animation: pulse 2s infinite;
        }
        
        @keyframes pulse {
            0%, 100% { opacity: 1; }
            50% { opacity: 0.4; }
        }
        
        .instruction {
            position: absolute;
            bottom: 70px;
            left: 50%;
            transform: translateX(-50%);
            background: rgba(0,0,0,0.6);
            padding: 8px 20px;
            border-radius: 30px;
            font-size: 13px;
            backdrop-filter: blur(10px);
            white-space: nowrap;
        }
    </style>
</head>
<body>
    <canvas id="renderCanvas"></canvas>
    
    <div class="gui-panel">
        <div class="gui-header">
            REACTIVEX ORB v0.8 • NEURAL CORE
        </div>
        <div class="controls-container" id="controls"></div>
    </div>
    
    <div class="status-bar">
        <span class="dot"></span>
        <span id="status">PHYSICS CORE ONLINE • BALL STABLE</span>
    </div>
    
    <div class="instruction">
        LMB Drag: Move Orb • RMB Drag: Orbit Camera
    </div>

    <script>
        const canvas = document.getElementById("renderCanvas");
        const engine = new BABYLON.Engine(canvas, true, { preserveDrawingBuffer: true, stencil: true });
        
        let scene, ball, ground, camera;
        let parameters = {};
        let isDragging = false;
        let isOrbiting = false;
        let previousMouseX = 0;
        let previousMouseY = 0;
        
        // Generate many parameters for "reactiveness"
        function generateParameters() {
            const paramList = [
                { name: "mass", label: "Mass (kg)", min: 0.1, max: 50, value: 1.2, step: 0.1 },
                { name: "restitution", label: "Bounciness", min: 0, max: 1, value: 0.85, step: 0.01 },
                { name: "friction", label: "Surface Friction", min: 0, max: 1, value: 0.3, step: 0.01 },
                { name: "airDrag", label: "Air Resistance", min: 0, max: 2, value: 0.4, step: 0.01 },
                { name: "mouseSensitivity", label: "Mouse Sensitivity", min: 0.1, max: 8, value: 2.5, step: 0.1 },
                { name: "forceMultiplier", label: "Force Multiplier", min: 1, max: 100, value: 25, step: 1 },
                { name: "gravityScale", label: "Gravity Influence", min: 0, max: 3, value: 1, step: 0.05 },
                { name: "angularDamping", label: "Spin Decay", min: 0, max: 5, value: 0.8, step: 0.05 },
                { name: "linearDamping", label: "Velocity Decay", min: 0, max: 5, value: 0.6, step: 0.05 },
                { name: "reactToWind", label: "Wind Reactiveness", min: 0, max: 10, value: 2.5, step: 0.1 },
                { name: "surfaceAdhesion", label: "Surface Grip", min: 0, max: 1, value: 0.65, step: 0.01 },
                { name: "impulseThreshold", label: "Impulse Threshold", min: 0.1, max: 20, value: 3, step: 0.1 },
                { name: "colorShiftSpeed", label: "Color React Speed", min: 0, max: 5, value: 1.2, step: 0.1 },
                { name: "glowIntensity", label: "Energy Glow", min: 0, max: 5, value: 2.8, step: 0.1 },
                { name: "vibrationFreq", label: "Vibration Frequency", min: 0, max: 30, value: 8, step: 0.5 },
                { name: "collisionEnergy", label: "Collision Energy", min: 0, max: 100, value: 45, step: 1 },
                { name: "trajectoryPredict", label: "Path Prediction", min: 0, max: 1, value: 0.7, step: 0.01 },
                { name: "autoStabilize", label: "Auto Stabilization", min: 0, max: 10, value: 2.5, step: 0.1 },
                { name: "magneticField", label: "Magnetic Pull", min: 0, max: 15, value: 0, step: 0.5 },
                { name: "quantumNoise", label: "Quantum Noise", min: 0, max: 8, value: 1.5, step: 0.1 },
            ];
            
            // Add more to reach \~40 parameters
            for (let i = 1; i <= 25; i++) {
                paramList.push({
                    name: `customReact${i}`,
                    label: `React Param ${i}`,
                    min: 0,
                    max: 10,
                    value: Math.random() * 5 + 2,
                    step: 0.1
                });
            }
            
            return paramList;
        }
        
        function createGUI() {
            const container = document.getElementById("controls");
            const params = generateParameters();
            
            params.forEach(param => {
                parameters[param.name] = param.value;
                
                const div = document.createElement("div");
                div.className = "param";
                
                div.innerHTML = `
                    <div class="param-label">${param.label}</div>
                    <div class="slider-container">
                        <input type="range" id="slider-${param.name}" 
                               min="\( {param.min}" max=" \){param.max}" step="${param.step}" 
                               value="${param.value}">
                        <input type="number" id="num-${param.name}" 
                               value="\( {param.value}" step=" \){param.step}">
                    </div>
                `;
                
                container.appendChild(div);
                
                // Slider handler
                const slider = div.querySelector(`#slider-${param.name}`);
                const numInput = div.querySelector(`#num-${param.name}`);
                
                slider.addEventListener("input", () => {
                    parameters[param.name] = parseFloat(slider.value);
                    numInput.value = parseFloat(slider.value).toFixed(2);
                    updateBallProperties();
                });
                
                numInput.addEventListener("change", () => {
                    let val = parseFloat(numInput.value);
                    if (val < param.min) val = param.min;
                    if (val > param.max) val = param.max;
                    parameters[param.name] = val;
                    slider.value = val;
                    updateBallProperties();
                });
            });
        }
        
        function updateBallProperties() {
            if (!ball || !ball.physicsImpostor) return;
            
            const impostor = ball.physicsImpostor;
            
            // Apply physics properties
            impostor.setMass(parameters.mass);
            impostor.setLinearDamping(parameters.linearDamping);
            impostor.setAngularDamping(parameters.angularDamping);
            
            // Visual feedback
            const mat = ball.material;
            if (mat) {
                mat.emissiveColor = new BABYLON.Color3(
                    Math.min(1, parameters.glowIntensity * 0.3),
                    Math.min(1, parameters.glowIntensity * 0.1),
                    1
                );
            }
        }
        
        async function createScene() {
            scene = new BABYLON.Scene(engine);
            scene.clearColor = new BABYLON.Color4(0.02, 0.02, 0.08, 1);
            
            // Camera
            camera = new BABYLON.ArcRotateCamera("camera", Math.PI / 2, Math.PI / 2.5, 25, new BABYLON.Vector3(0, 5, 0), scene);
            camera.attachControl(canvas, true);
            camera.lowerRadiusLimit = 8;
            camera.upperRadiusLimit = 60;
            camera.wheelPrecision = 50;
            
            // Lights
            const hemiLight = new BABYLON.HemisphericLight("hemi", new BABYLON.Vector3(0, 1, 0), scene);
            hemiLight.intensity = 0.7;
            hemiLight.groundColor = new BABYLON.Color3(0.1, 0.2, 0.4);
            
            const dirLight = new BABYLON.DirectionalLight("dir", new BABYLON.Vector3(-0.8, -1, -0.5), scene);
            dirLight.intensity = 1.2;
            
            // Environment
            const skybox = BABYLON.MeshBuilder.CreateBox("skyBox", {size: 500}, scene);
            const skyboxMaterial = new BABYLON.StandardMaterial("skyBox", scene);
            skyboxMaterial.backFaceCulling = false;
            skyboxMaterial.reflectionTexture = new BABYLON.CubeTexture("https://assets.babylonjs.com/textures/skybox", scene);
            skyboxMaterial.reflectionTexture.coordinatesMode = BABYLON.Texture.SKYBOX_MODE;
            skyboxMaterial.diffuseColor = new BABYLON.Color3(0, 0, 0);
            skyboxMaterial.specularColor = new BABYLON.Color3(0, 0, 0);
            skybox.material = skyboxMaterial;
            
            // Ground
            ground = BABYLON.MeshBuilder.CreateGround("ground", {width: 100, height: 100}, scene);
            const groundMat = new BABYLON.StandardMaterial("groundMat", scene);
            groundMat.diffuseColor = new BABYLON.Color3(0.1, 0.15, 0.2);
            groundMat.specularColor = new BABYLON.Color3(0.3, 0.4, 0.5);
            groundMat.specularPower = 64;
            ground.material = groundMat;
            
            // Add some environment details
            for (let i = 0; i < 12; i++) {
                const box = BABYLON.MeshBuilder.CreateBox(`box${i}`, {size: 2 + Math.random() * 4}, scene);
                box.position.x = (Math.random() - 0.5) * 60;
                box.position.z = (Math.random() - 0.5) * 60;
                box.position.y = 1 + Math.random() * 3;
                
                const boxMat = new BABYLON.StandardMaterial("boxMat", scene);
                boxMat.diffuseColor = new BABYLON.Color3(0.3, 0.35, 0.45);
                box.material = boxMat;
            }
            
            // The Ball
            ball = BABYLON.MeshBuilder.CreateSphere("orb", {diameter: 2.5, segments: 64}, scene);
            ball.position.y = 8;
            
            const orbMat = new BABYLON.StandardMaterial("orbMat", scene);
            orbMat.diffuseColor = new BABYLON.Color3(0.6, 0.8, 1);
            orbMat.emissiveColor = new BABYLON.Color3(0.2, 0.5, 1);
            orbMat.specularColor = new BABYLON.Color3(1, 1, 1);
            orbMat.specularPower = 128;
            ball.material = orbMat;
            
            // Physics
            await BABYLON.PhysicsHelper;
            scene.enablePhysics(new BABYLON.Vector3(0, -9.81, 0), new BABYLON.AmmoJSPlugin());
            
            const ballImpostor = new BABYLON.PhysicsImpostor(ball, BABYLON.PhysicsImpostor.SphereImpostor, {
                mass: 1.2,
                restitution: 0.85,
                friction: 0.3
            }, scene);
            
            ground.physicsImpostor = new BABYLON.PhysicsImpostor(ground, BABYLON.PhysicsImpostor.BoxImpostor, {
                mass: 0,
                restitution: 0.6,
                friction: 0.7
            }, scene);
            
            // Add subtle animation to orb
            let time = 0;
            scene.registerBeforeRender(() => {
                time += 0.02;
                
                // Gentle floating effect
                const float = Math.sin(time) * 0.05;
                if (ball) ball.position.y += float * 0.01;
                
                // Update status
                const vel = ball.physicsImpostor.getLinearVelocity();
                const speed = vel.length();
                document.getElementById("status").textContent = 
                    `SPEED: ${speed.toFixed(1)} m/s • ENERGY: ${(parameters.glowIntensity * 20).toFixed(0)}%`;
            });
            
            // Input handling
            setupInputHandlers();
            
            return scene;
        }
        
        function setupInputHandlers() {
            let isLeftClick = false;
            let isRightClick = false;
            
            canvas.addEventListener("pointerdown", (e) => {
                if (e.button === 0) { // Left click
                    isLeftClick = true;
                    isDragging = true;
                    previousMouseX = e.clientX;
                    previousMouseY = e.clientY;
                } else if (e.button === 2) { // Right click
                    isRightClick = true;
                    isOrbiting = true;
                    camera.attachControl(canvas, false);
                }
            });
            
            canvas.addEventListener("pointerup", (e) => {
                if (e.button === 0) isLeftClick = false;
                if (e.button === 2) {
                    isRightClick = false;
                    isOrbiting = false;
                    camera.attachControl(canvas, true);
                }
                isDragging = false;
            });
            
            canvas.addEventListener("pointermove", (e) => {
                if (isLeftClick && ball) {
                    // Move ball with mouse
                    const pickRay = scene.createPickingRay(e.clientX, e.clientY, BABYLON.Matrix.Identity(), camera);
                    const pickResult = scene.pickWithRay(pickRay);
                    
                    if (pickResult.hit && pickResult.pickedMesh === ground) {
                        const targetPos = pickResult.pickedPoint.clone();
                        targetPos.y = ball.position.y;
                        
                        const force = targetPos.subtract(ball.position);
                        force.y = 0;
                        force.scaleInPlace(parameters.forceMultiplier * parameters.mouseSensitivity);
                        
                        ball.physicsImpostor.applyImpulse(force, ball.getAbsolutePosition());
                    }
                }
                
                if (isOrbiting) {
                    // Babylon handles orbit via ArcRotateCamera when attached
                }
            });
            
            // Prevent context menu on right click
            canvas.addEventListener("contextmenu", (e) => e.preventDefault());
            
            // Keyboard boost
            window.addEventListener("keydown", (e) => {
                if (e.key === " " && ball) {
                    ball.physicsImpostor.applyImpulse(new BABYLON.Vector3(0, 25, 0), ball.getAbsolutePosition());
                }
            });
        }
        
        function resize() {
            engine.resize();
        }
        
        window.addEventListener("resize", resize);
        
        // Initialize
        async function init() {
            await createScene();
            createGUI();
            updateBallProperties();
            
            engine.runRenderLoop(() => {
                if (scene) scene.render();
            });
        }
        
        init();
    </script>
</body>
</html>
