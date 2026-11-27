<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Cartoon Short Generator - Dynamic Settings & Full Cast</title>
    <style>
        body {
            font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, Helvetica, Arial, sans-serif;
            background: #0f172a;
            color: #f8fafc;
            margin: 0;
            padding: 20px;
            display: flex;
            flex-direction: column;
            align-items: center;
            min-height: 100vh;
        }
        .container {
            width: 100%;
            max-width: 600px;
            background: #1e293b;
            padding: 24px;
            border-radius: 12px;
            box-shadow: 0 10px 25px -5px rgba(0, 0, 0, 0.3);
            box-sizing: border-box;
        }
        h1 {
            font-size: 1.4rem;
            margin-top: 0;
            margin-bottom: 16px;
            color: #38bdf8;
            text-align: center;
        }
        .form-group {
            margin-bottom: 16px;
        }
        label {
            display: block;
            font-size: 0.875rem;
            font-weight: 600;
            margin-bottom: 6px;
            color: #94a3b8;
        }
        textarea {
            width: 100%;
            height: 100px;
            padding: 10px;
            background: #0f172a;
            border: 1px solid #334155;
            border-radius: 6px;
            color: #fff;
            font-size: 1rem;
            box-sizing: border-box;
            resize: vertical;
        }
        button {
            width: 100%;
            background: #0284c7;
            color: white;
            border: none;
            padding: 12px;
            font-size: 1rem;
            font-weight: 600;
            border-radius: 6px;
            cursor: pointer;
            transition: background 0.2s;
        }
        button:hover {
            background: #0369a1;
        }
        button:disabled {
            background: #475569;
            cursor: not-allowed;
        }
        #preview-container {
            margin-top: 20px;
            display: flex;
            flex-direction: column;
            align-items: center;
            position: relative;
        }
        canvas {
            width: 270px;
            height: 480px;
            background: #0f172a;
            border-radius: 8px;
            border: 2px solid #334155;
            margin-bottom: 16px;
        }
        #subtitle-overlay {
            position: absolute;
            bottom: 90px;
            width: 270px;
            text-align: center;
            font-size: 1.6rem;
            font-weight: 900;
            color: #ffffff;
            text-shadow: 2px 2px 0px #000, -2px -2px 0px #000, 2px -2px 0px #000, -2px 2px 0px #000;
            pointer-events: none;
            font-family: Impact, sans-serif;
            letter-spacing: 1px;
            min-height: 40px;
        }
        .progress-box {
            font-size: 0.875rem;
            color: #38bdf8;
            margin-bottom: 12px;
            text-align: center;
            min-height: 20px;
        }
        .actions {
            display: flex;
            gap: 10px;
            width: 100%;
        }
        .actions button {
            background: #10b981;
        }
        .actions button:hover {
            background: #059669;
        }
        .hidden {
            display: none !important;
        }
    </style>
</head>
<body>

<div class="container">
    <h1>Cartoon Short Generator</h1>
    
    <div class="form-group">
        <label for="script-input">Script (Supports Multiple Settings & Characters)</label>
        <textarea id="script-input">A man sold the Eiffel Tower twice. In a bustling Parisian market square, he shook hands with a wealthy buyer. Later inside a shadowy office, he signed the fraudulent papers.</textarea>
    </div>

    <button id="generate-btn">Generate Short & Download</button>

    <div id="preview-container">
        <div id="status-text" class="progress-box">Ready</div>
        <canvas id="renderCanvas" width="540" height="960"></canvas>
        <div id="subtitle-overlay"></div>
        
        <div id="result-actions" class="actions hidden">
            <button id="download-btn">Download MP4</button>
            <button id="save-device-btn">📱 Save to Device Photos</button>
        </div>
    </div>
</div>

<!-- Three.js for 3D Toon World Render -->
<script src="https://cdnjs.cloudflare.com/ajax/libs/three.js/r128/three.min.js"></script>
<script>
    const scriptInput = document.getElementById('script-input');
    const generateBtn = document.getElementById('generate-btn');
    const statusText = document.getElementById('status-text');
    const canvas = document.getElementById('renderCanvas');
    const subtitleOverlay = document.getElementById('subtitle-overlay');
    const resultActions = document.getElementById('result-actions');
    const downloadBtn = document.getElementById('download-btn');
    const saveDeviceBtn = document.getElementById('save-device-btn');

    let videoBlob = null;
    let videoUrl = null;

    generateBtn.addEventListener('click', async () => {
        const text = scriptInput.value.trim();
        if (!text) {
            alert('Please enter a script.');
            return;
        }

        generateBtn.disabled = true;
        resultActions.classList.add('hidden');
        statusText.textContent = 'Building World & Full Cast...';

        try {
            const renderer = new THREE.WebGLRenderer({ canvas, antialias: true, preserveDrawingBuffer: true });
            renderer.setSize(540, 960);

            const scene = new THREE.Scene();
            scene.background = new THREE.Color(0x0f172a);
            scene.fog = new THREE.FogExp2(0x0f172a, 0.04);

            const camera = new THREE.PerspectiveCamera(40, 540 / 960, 0.1, 100);
            camera.position.set(0, 1.6, 4.2);
            camera.lookAt(0, 1, 0);

            // Lighting
            const ambientLight = new THREE.AmbientLight(0xffffff, 0.7);
            scene.add(ambientLight);
            const dirLight = new THREE.DirectionalLight(0xffffff, 0.9);
            dirLight.position.set(5, 10, 7);
            scene.add(dirLight);

            // Environment Ground
            const groundGeo = new THREE.PlaneGeometry(30, 30);
            const groundMat = new THREE.MeshToonMaterial({ color: 0x334155 });
            const ground = new THREE.Mesh(groundGeo, groundMat);
            ground.rotation.x = -Math.PI / 2;
            scene.add(ground);

            // --- FULL CAST (Main Hero + Styled Outfits + Side Villagers) ---
            const heroGroup = new THREE.Group();
            const heroBody = new THREE.Mesh(new THREE.BoxGeometry(0.8, 1.2, 0.3), new THREE.MeshToonMaterial({ color: 0x38bdf8 }));
            heroBody.position.y = 0.6;
            const heroHead = new THREE.Mesh(new THREE.SphereGeometry(0.45, 16, 16), new THREE.MeshToonMaterial({ color: 0xfde047 }));
            heroHead.position.y = 1.45;
            
            // Add Hat / Outfit detail
            const hat = new THREE.Mesh(new THREE.CylinderGeometry(0.48, 0.48, 0.2, 16), new THREE.MeshToonMaterial({ color: 0x1e3a8a }));
            hat.position.y = 1.8;
            heroGroup.add(heroBody, heroHead, hat);
            heroGroup.position.set(0, 0, 0);
            scene.add(heroGroup);

            // Side Character 1 (Merchant)
            const merchant = new THREE.Group();
            const mBody = new THREE.Mesh(new THREE.BoxGeometry(0.7, 1.1, 0.3), new THREE.MeshToonMaterial({ color: 0xf43f5e }));
            mBody.position.y = 0.55;
            const mHead = new THREE.Mesh(new THREE.SphereGeometry(0.4, 16, 16), new THREE.MeshToonMaterial({ color: 0xfde047 }));
            mHead.position.y = 1.35;
            merchant.add(mBody, mHead);
            merchant.position.set(-1.3, 0, -0.6);
            scene.add(merchant);

            // Side Character 2 (Buyer / Guard)
            const buyer = new THREE.Group();
            const bBody = new THREE.Mesh(new THREE.BoxGeometry(0.75, 1.15, 0.3), new THREE.MeshToonMaterial({ color: 0x10b981 }));
            bBody.position.y = 0.58;
            const bHead = new THREE.Mesh(new THREE.SphereGeometry(0.4, 16, 16), new THREE.MeshToonMaterial({ color: 0xfde047 }));
            bHead.position.y = 1.38;
            buyer.add(bBody, bHead);
            buyer.position.set(1.3, 0, -0.9);
            scene.add(buyer);

            // Setting props (Building / Pillar structures)
            const propGeo = new THREE.BoxGeometry(1.2, 3.5, 1.2);
            const propMat = new THREE.MeshToonMaterial({ color: 0x475569 });
            const leftBuilding = new THREE.Mesh(propGeo, propMat);
            leftBuilding.position.set(-2.5, 1.75, -2);
            scene.add(leftBuilding);

            const rightBuilding = new THREE.Mesh(propGeo, propMat);
            rightBuilding.position.set(2.5, 1.75, -2);
            scene.add(rightBuilding);

            // Audio & Voice Timing Setup
            const audioCtx = new (window.AudioContext || window.webkitAudioContext)();
            const dest = audioCtx.createMediaStreamDestination();

            const osc = audioCtx.createOscillator();
            const gainNode = audioCtx.createGain();
            gainNode.gain.value = 0.15;
            osc.type = 'triangle';
            osc.frequency.setValueAtTime(220, audioCtx.currentTime);
            osc.connect(gainNode);
            gainNode.connect(dest);
            osc.start();

            const utterance = new SpeechSynthesisUtterance(text);
            utterance.pitch = 1.2;
            utterance.rate = 1.1;

            const words = text.split(/\s+/);
            const durationSec = Math.max(5, words.length * 0.35);

            const stream = canvas.captureStream(30);
            const combinedStream = new MediaStream([
                ...stream.getVideoTracks(),
                ...dest.stream.getAudioTracks()
            ]);

            const recorder = new MediaRecorder(combinedStream, { mimeType: 'video/webm;codecs=vp9,opus' });
            const chunks = [];
            recorder.ondataavailable = e => chunks.push(e.data);
            recorder.onstop = () => {
                videoBlob = new Blob(chunks, { type: 'video/mp4' });
                videoUrl = URL.createObjectURL(videoBlob);
                
                resultActions.classList.remove('hidden');
                statusText.textContent = 'Short Generated Successfully!';
                subtitleOverlay.textContent = '';
                generateBtn.disabled = false;
            };

            recorder.start();
            window.speechSynthesis.speak(utterance);

            const fps = 30;
            const totalFrames = Math.ceil(durationSec * fps);
            let frame = 0;

            function renderLoop() {
                if (frame < totalFrames) {
                    const time = frame / fps;
                    
                    // Dynamic Setting Shift halfway through
                    if (time > durationSec * 0.5) {
                        scene.background.setHex(0x1e1b4b); // Shift to night/office setting
                        groundMat.color.setHex(0x1e293b);
                        propMat.color.setHex(0x334155);
                    } else {
                        scene.background.setHex(0x0f172a); // Day market setting
                        groundMat.color.setHex(0x334155);
                        propMat.color.setHex(0x475569);
                    }

                    // 1-Word Subtitles
                    const wordIndex = Math.min(Math.floor(time * 2.8), words.length - 1);
                    subtitleOverlay.textContent = words[wordIndex];

                    // Character animations
                    heroGroup.rotation.y = Math.sin(frame * 0.08) * 0.3;
                    heroHead.position.y = 1.45 + Math.sin(frame * 0.35) * 0.06;
                    hat.position.y = 1.8 + Math.sin(frame * 0.35) * 0.06;
                    
                    merchant.rotation.y = Math.cos(frame * 0.06) * 0.15;
                    buyer.position.y = Math.sin(frame * 0.05) * 0.05;

                    // Camera moves across settings
                    camera.position.x = Math.sin(frame * 0.015) * 0.5;
                    camera.position.y = 1.6 + Math.cos(frame * 0.025) * 0.15;
                    camera.lookAt(0, 1, 0);

                    renderer.render(scene, camera);
                    frame++;
                    requestAnimationFrame(renderLoop);
                } else {
                    recorder.stop();
                    osc.stop();
                }
            }

            renderLoop();

        } catch (err) {
            console.error(err);
            statusText.textContent = 'Error during generation. Check console.';
            generateBtn.disabled = false;
        }
    });

    downloadBtn.addEventListener('click', () => {
        if (!videoUrl) return;
        const a = document.createElement('a');
        a.href = videoUrl;
        a.download = 'cartoon-short.mp4';
        document.body.appendChild(a);
        a.click();
        document.body.removeChild(a);
    });

    saveDeviceBtn.addEventListener('click', async () => {
        if (!videoBlob) return;
        const file = new File([videoBlob], 'cartoon-short.mp4', { type: 'video/mp4' });
        
        if (navigator.canShare && navigator.canShare({ files: [file] })) {
            try {
                await navigator.share({
                    files: [file],
                    title: 'Cartoon Short',
                    text: 'Generated with Cartoon Short Generator'
                });
            } catch (err) {
                console.log('Share canceled', err);
            }
        } else {
            const a = document.createElement('a');
            a.href = videoUrl;
            a.download = 'cartoon-short.mp4';
            a.click();
            alert('File downloaded! Check your Photos/Gallery app.');
        }
    });
</script>

</body>
</html>
