<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Cartoon Audio Short Generator</title>
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
        textarea, input[type="file"] {
            width: 100%;
            padding: 10px;
            background: #0f172a;
            border: 1px solid #334155;
            border-radius: 6px;
            color: #fff;
            font-size: 1rem;
            box-sizing: border-box;
        }
        textarea {
            height: 90px;
            resize: vertical;
        }
        .button-row {
            display: flex;
            gap: 10px;
            margin-bottom: 12px;
        }
        button {
            flex: 1;
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
        .record-btn {
            background: #ef4444 !important;
        }
        .record-btn:hover {
            background: #dc2626 !important;
        }
        .record-btn.recording {
            animation: pulse 1.5s infinite;
        }
        @keyframes pulse {
            0% { opacity: 1; }
            50% { opacity: 0.5; }
            100% { opacity: 1; }
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
        <label for="script-input">Narration Script / Story Idea</label>
        <textarea id="script-input">A man sold the Eiffel Tower twice using pure confidence and a fake business card.</textarea>
    </div>

    <div class="form-group">
        <label for="audio-file">Upload Audio Voiceover (MP3, WAV, M4A)</label>
        <input type="file" id="audio-file" accept=".mp3,.wav,.m4a,.aac,audio/*">
    </div>

    <div class="button-row">
        <button id="record-btn" class="record-btn">🎤 Record Voiceover</button>
        <button id="generate-btn">🎬 Generate Short</button>
    </div>

    <div id="preview-container">
        <div id="status-text" class="progress-box">Ready</div>
        <canvas id="renderCanvas" width="540" height="960"></canvas>
        <div id="subtitle-overlay"></div>
        
        <div id="result-actions" class="actions hidden">
            <button id="download-btn">Download MP4</button>
            <button id="save-device-btn">📱 Save to Photos</button>
        </div>
    </div>
</div>

<script src="https://cdnjs.cloudflare.com/ajax/libs/three.js/r128/three.min.js"></script>
<script>
    const scriptInput = document.getElementById('script-input');
    const audioFileInput = document.getElementById('audio-file');
    const recordBtn = document.getElementById('record-btn');
    const generateBtn = document.getElementById('generate-btn');
    const statusText = document.getElementById('status-text');
    const canvas = document.getElementById('renderCanvas');
    const subtitleOverlay = document.getElementById('subtitle-overlay');
    const resultActions = document.getElementById('result-actions');
    const downloadBtn = document.getElementById('download-btn');
    const saveDeviceBtn = document.getElementById('save-device-btn');

    let uploadedAudioBuffer = null;
    let mediaRecorder = null;
    let recordedChunks = [];
    let isRecording = false;

    // Handle Mic Recording
    recordBtn.addEventListener('click', async () => {
        if (!isRecording) {
            try {
                const stream = await navigator.mediaDevices.getUserMedia({ audio: true });
                mediaRecorder = new MediaRecorder(stream);
                recordedChunks = [];
                mediaRecorder.ondataavailable = e => recordedChunks.push(e.data);
                mediaRecorder.onstop = async () => {
                    const blob = new Blob(recordedChunks, { type: 'audio/webm' });
                    const arrayBuffer = await blob.arrayBuffer();
                    const audioCtx = new (window.AudioContext || window.webkitAudioContext)();
                    uploadedAudioBuffer = await audioCtx.decodeAudioData(arrayBuffer);
                    statusText.textContent = 'Voice recording loaded successfully!';
                };
                mediaRecorder.start();
                isRecording = true;
                recordBtn.textContent = '⏹️ Stop Recording';
                recordBtn.classList.add('recording');
                statusText.textContent = 'Recording microphone voice...';
            } catch (err) {
                alert('Microphone access denied or not supported.');
            }
        } else {
            mediaRecorder.stop();
            mediaRecorder.stream.getTracks().forEach(t => t.stop());
            isRecording = false;
            recordBtn.textContent = '🎤 Record Voiceover';
            recordBtn.classList.remove('recording');
        }
    });

    // Handle File Upload (.mp3, .wav, .m4a)
    audioFileInput.addEventListener('change', async (e) => {
        const file = e.target.files[0];
        if (!file) return;
        statusText.textContent = 'Loading audio file...';
        const arrayBuffer = await file.arrayBuffer();
        const audioCtx = new (window.AudioContext || window.webkitAudioContext)();
        uploadedAudioBuffer = await audioCtx.decodeAudioData(arrayBuffer);
        statusText.textContent = `Loaded: ${file.name} (${uploadedAudioBuffer.duration.toFixed(1)}s)`;
    });

    generateBtn.addEventListener('click', async () => {
        const text = scriptInput.value.trim();
        if (!text) {
            alert('Please enter a script.');
            return;
        }

        generateBtn.disabled = true;
        resultActions.classList.add('hidden');
        statusText.textContent = 'Rendering 3D World & Mixing Audio...';

        try {
            const renderer = new THREE.WebGLRenderer({ canvas, antialias: true, preserveDrawingBuffer: true });
            renderer.setSize(540, 960);

            const scene = new THREE.Scene();
            scene.background = new THREE.Color(0x0f172a);
            scene.fog = new THREE.FogExp2(0x0f172a, 0.04);

            const camera = new THREE.PerspectiveCamera(40, 540 / 960, 0.1, 100);
            camera.position.set(0, 1.6, 4.2);
            camera.lookAt(0, 1, 0);

            scene.add(new THREE.AmbientLight(0xffffff, 0.7));
            const dirLight = new THREE.DirectionalLight(0xffffff, 0.9);
            dirLight.position.set(5, 10, 7);
            scene.add(dirLight);

            const ground = new THREE.Mesh(new THREE.PlaneGeometry(30, 30), new THREE.MeshToonMaterial({ color: 0x334155 }));
            ground.rotation.x = -Math.PI / 2;
            scene.add(ground);

            // Character Cards (Stylized Bean-heads)
            const heroGroup = new THREE.Group();
            const body = new THREE.Mesh(new THREE.BoxGeometry(0.8, 1.2, 0.3), new THREE.MeshToonMaterial({ color: 0x38bdf8 }));
            body.position.y = 0.6;
            const head = new THREE.Mesh(new THREE.SphereGeometry(0.45, 16, 16), new THREE.MeshToonMaterial({ color: 0xfde047 }));
            head.position.y = 1.45;
            heroGroup.add(body, head);
            scene.add(heroGroup);

            // Audio Context Setup
            const audioCtx = new (window.AudioContext || window.webkitAudioContext)();
            const dest = audioCtx.createMediaStreamDestination();

            let durationSec = 6;
            const words = text.split(/\s+/);

            if (uploadedAudioBuffer) {
                durationSec = uploadedAudioBuffer.duration;
                const source = audioCtx.createBufferSource();
                source.buffer = uploadedAudioBuffer;
                source.connect(dest);
                source.connect(audioCtx.destination);
                source.start(0);
            } else {
                // Fallback synthetic voice if no audio provided
                durationSec = Math.max(5, words.length * 0.35);
                const utterance = new SpeechSynthesisUtterance(text);
                utterance.pitch = 1.2;
                utterance.rate = 1.1;
                window.speechSynthesis.speak(utterance);
            }

            // Background music bed at 50% volume (-6dB)
            const osc = audioCtx.createOscillator();
            const musicGain = audioCtx.createGain();
            musicGain.gain.value = 0.15;
            osc.type = 'triangle';
            osc.frequency.setValueAtTime(220, audioCtx.currentTime);
            osc.connect(musicGain);
            musicGain.connect(dest);
            osc.start();

            const stream = canvas.captureStream(30);
            const combinedStream = new MediaStream([
                ...stream.getVideoTracks(),
                ...dest.stream.getAudioTracks()
            ]);

            const recorder = new MediaRecorder(combinedStream, { mimeType: 'video/webm;codecs=vp9,opus' });
            const chunks = [];
            recorder.ondataavailable = e => chunks.push(e.data);
            recorder.onstop = () => {
                const videoBlob = new Blob(chunks, { type: 'video/mp4' });
                const videoUrl = URL.createObjectURL(videoBlob);
                
                resultActions.classList.remove('hidden');
                statusText.textContent = 'Short Generated Successfully!';
                subtitleOverlay.textContent = '';
                generateBtn.disabled = false;

                downloadBtn.onclick = () => {
                    const a = document.createElement('a');
                    a.href = videoUrl;
                    a.download = 'cartoon-short.mp4';
                    a.click();
                };

                saveDeviceBtn.onclick = async () => {
                    const file = new File([videoBlob], 'cartoon-short.mp4', { type: 'video/mp4' });
                    if (navigator.canShare && navigator.canShare({ files: [file] })) {
                        await navigator.share({ files: [file], title: 'Cartoon Short' });
                    } else {
                        downloadBtn.click();
                        alert('Check your Downloads or Photos app.');
                    }
                };
            };

            recorder.start();

            const fps = 30;
            const totalFrames = Math.ceil(durationSec * fps);
            let frame = 0;

            function renderLoop() {
                if (frame < totalFrames) {
                    const time = frame / fps;
                    const wordIndex = Math.min(Math.floor(time * (words.length / durationSec)), words.length - 1);
                    subtitleOverlay.textContent = words[wordIndex];

                    heroGroup.rotation.y = Math.sin(frame * 0.08) * 0.3;
                    head.position.y = 1.45 + Math.sin(frame * 0.4) * 0.06;

                    camera.position.x = Math.sin(frame * 0.015) * 0.5;
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
            statusText.textContent = 'Error during generation.';
            generateBtn.disabled = false;
        }
    });
</script>

</body>
</html>
