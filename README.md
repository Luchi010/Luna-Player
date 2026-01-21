<!DOCTYPE html>
<html lang="ja">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>LUNA Player Ver.3.1</title>
    <link href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.0.0/css/all.min.css" rel="stylesheet">
    <link href="https://fonts.googleapis.com/css2?family=M+PLUS+Rounded+1c:wght@800&display=swap" rel="stylesheet">
    <style>
        :root {
            --primary-color: #ffb7b2;
            --bg-color: #ffe6e6;
            --text-color: #fff;
            --accent-color: rgba(255, 255, 255, 0.4);
            --glass: rgba(255, 255, 255, 0.2);
        }

        * {
            box-sizing: border-box;
            margin: 0;
            padding: 0;
            user-select: none;
            font-family: 'Helvetica Neue', Arial, sans-serif;
            -webkit-tap-highlight-color: transparent;
        }

        body {
            background-color: var(--primary-color);
            height: 100vh;
            display: flex;
            justify-content: center;
            align-items: center;
            overflow: hidden;
            transition: background-color 0.5s ease;
        }

        .player-container {
            width: 100%;
            max-width: 400px;
            height: 95vh;
            padding: 20px;
            display: flex;
            flex-direction: column;
            position: relative;
            z-index: 10;
        }

        /* メニューボタン等 */
        .menu-btn {
            position: absolute;
            top: 20px;
            right: 20px;
            color: var(--text-color);
            font-size: 1.5rem;
            cursor: pointer;
            z-index: 100;
            opacity: 0.9;
            filter: drop-shadow(0 2px 4px rgba(0,0,0,0.1));
            transition: transform 0.2s;
        }
        .menu-btn:active { transform: scale(0.9); }

        .site-fullscreen-btn {
            position: fixed;
            bottom: 15px;
            right: 15px;
            width: 38px;
            height: 38px;
            background: rgba(255, 255, 255, 0.25);
            backdrop-filter: blur(5px);
            border-radius: 50%;
            display: flex;
            justify-content: center;
            align-items: center;
            color: white;
            font-size: 1.1rem;
            cursor: pointer;
            z-index: 90;
            box-shadow: 0 4px 15px rgba(0,0,0,0.1);
            border: 2px solid rgba(255,255,255,0.3);
            transition: transform 0.2s;
        }
        .site-fullscreen-btn:active { transform: scale(0.95); }

        /* メディア表示エリア */
        .media-wrapper {
            flex: 1;
            display: flex;
            flex-direction: column;
            justify-content: center;
            margin-top: 40px;
            margin-bottom: 20px;
            min-height: 0;
            position: relative;
        }

        .media-display {
            width: 100%;
            aspect-ratio: 1 / 1;
            background: white;
            border-radius: 25px;
            overflow: hidden;
            box-shadow: 0 15px 35px rgba(0,0,0,0.15);
            position: relative;
            display: flex;
            align-items: center;
            justify-content: center;
            flex-shrink: 0;
            transition: transform 0.2s, border 0.2s;
        }

        .media-display.drag-active {
            transform: scale(1.02);
            border: 4px dashed var(--primary-color);
            background: var(--bg-color);
        }
        .media-display.drag-active::after {
            content: "ここにドロップ！";
            position: absolute;
            color: var(--primary-color);
            font-family: 'M PLUS Rounded 1c', sans-serif;
            font-size: 1.5rem;
            font-weight: 800;
            pointer-events: none;
            z-index: 20;
        }

        video {
            width: 100%;
            height: 100%;
            object-fit: cover;
            position: absolute;
            top: 0; left: 0;
            z-index: 1;
        }

        #coverImageOverlay {
            width: 100%;
            height: 100%;
            object-fit: cover;
            position: absolute;
            top: 0; left: 0;
            z-index: 5;
            background: #000;
            display: none;
        }

        .placeholder-text {
            color: #ccc;
            font-size: 1rem;
            text-align: center;
            padding: 20px;
            font-weight: bold;
            position: relative;
            z-index: 2;
        }

        .file-name-display {
            text-align: center;
            color: var(--text-color);
            font-family: 'M PLUS Rounded 1c', sans-serif;
            font-weight: 800;
            font-size: 1.2rem;
            margin-top: 15px;
            white-space: nowrap;
            overflow: hidden;
            text-overflow: ellipsis;
            text-shadow: 0 2px 4px rgba(0,0,0,0.1);
            min-height: 1.4em;
        }

        /* コントロールエリア */
        .controls-area {
            width: 100%;
            display: flex;
            flex-direction: column;
            gap: 15px;
            padding-bottom: 10px;
        }

        .progress-container {
            width: 100%;
            height: 24px;
            cursor: pointer;
            display: flex;
            align-items: center;
            position: relative;
        }
        .progress-bg {
            width: 100%;
            height: 6px;
            background: var(--glass);
            border-radius: 4px;
            position: relative;
        }
        .progress-bar {
            height: 100%;
            background: white;
            border-radius: 4px;
            width: 0%;
            position: relative;
        }
        .progress-handle {
            position: absolute;
            top: 50%;
            left: 0%;
            transform: translate(-50%, -50%) rotate(15deg);
            width: 30px;
            height: 30px;
            display: flex;
            align-items: center;
            justify-content: center;
            color: white;
            font-size: 1.2rem;
            filter: drop-shadow(0 2px 3px rgba(0,0,0,0.3));
            pointer-events: none;
        }

        .time-display {
            display: flex;
            justify-content: space-between;
            color: var(--text-color);
            font-size: 0.85rem;
            margin-top: -5px;
            font-weight: bold;
            text-shadow: 0 1px 2px rgba(0,0,0,0.1);
        }

        .main-controls {
            display: flex;
            justify-content: center;
            align-items: center;
            gap: 20px;
            margin: 5px 0;
        }
        .control-btn {
            color: var(--text-color);
            font-size: 1.8rem;
            cursor: pointer;
            opacity: 0.95;
            transition: transform 0.1s;
            filter: drop-shadow(0 2px 3px rgba(0,0,0,0.1));
        }
        .control-btn:active { transform: scale(0.85); }
        .play-btn { font-size: 3.2rem; margin: 0 10px; }
        .skip-btn { font-size: 1.4rem; opacity: 0.8; }

        .slider-container {
            display: flex;
            align-items: center;
            gap: 12px;
            color: var(--text-color);
            font-size: 0.9rem;
        }
        .common-slider {
            flex: 1;
            -webkit-appearance: none;
            height: 6px;
            background: var(--glass);
            border-radius: 3px;
            outline: none;
        }
        .common-slider::-webkit-slider-thumb {
            -webkit-appearance: none;
            width: 16px;
            height: 16px;
            background: white;
            border-radius: 50%;
            cursor: pointer;
            box-shadow: 0 2px 5px rgba(0,0,0,0.2);
        }

        /* メニュー & サブスクリーン */
        .menu-overlay {
            position: fixed;
            top: 0; left: 0; width: 100%; height: 100%;
            background: rgba(0,0,0,0.6);
            backdrop-filter: blur(5px);
            z-index: 200;
            display: flex; justify-content: flex-end;
            opacity: 0; pointer-events: none;
            transition: opacity 0.3s ease;
        }
        .menu-overlay.active { opacity: 1; pointer-events: auto; }

        .side-menu {
            width: 80%; max-width: 280px; height: 100%;
            background: white;
            padding: 30px 20px;
            transform: translateX(100%);
            transition: transform 0.3s cubic-bezier(0.4, 0, 0.2, 1);
            display: flex; flex-direction: column; gap: 15px;
            box-shadow: -5px 0 25px rgba(0,0,0,0.15);
        }
        .menu-overlay.active .side-menu { transform: translateX(0); }
        
        .menu-item {
            padding: 15px;
            background: #f8f8f8;
            border-radius: 12px;
            display: flex; align-items: center; gap: 15px;
            cursor: pointer; color: #444; font-weight: bold;
            transition: background 0.2s;
        }
        .menu-item:hover { background: #eee; }
        .menu-item i { font-size: 1.2rem; color: var(--primary-color); width: 20px; text-align: center;}

        /* トグルスイッチ用 */
        .toggle-switch {
            position: relative;
            display: inline-block;
            width: 40px;
            height: 22px;
        }
        .toggle-switch input { opacity: 0; width: 0; height: 0; }
        .slider {
            position: absolute;
            cursor: pointer;
            top: 0; left: 0; right: 0; bottom: 0;
            background-color: #ccc;
            transition: .4s;
            border-radius: 22px;
        }
        .slider:before {
            position: absolute;
            content: "";
            height: 16px;
            width: 16px;
            left: 3px;
            bottom: 3px;
            background-color: white;
            transition: .4s;
            border-radius: 50%;
        }
        input:checked + .slider { background-color: var(--primary-color); }
        input:checked + .slider:before { transform: translateX(18px); }


        .sub-screen {
            position: fixed; top: 0; left: 0; width: 100%; height: 100%;
            background: #fafafa; z-index: 300;
            display: flex; flex-direction: column;
            transform: translateY(100%); transition: transform 0.3s ease-in-out;
            padding: 20px;
        }
        .sub-screen.active { transform: translateY(0); }
        .screen-header {
            display: flex; justify-content: space-between; align-items: center;
            margin-bottom: 25px; font-size: 1.3rem; font-family: 'M PLUS Rounded 1c', sans-serif; font-weight: 800; color: #333;
        }

        /* プレイリスト */
        .playlist-items { flex: 1; overflow-y: auto; }
        .playlist-item {
            display: flex; justify-content: space-between; align-items: center;
            padding: 12px; border-bottom: 1px solid #eee;
            background: white; margin-bottom: 5px; border-radius: 8px;
            cursor: grab;
            transition: transform 0.2s, box-shadow 0.2s;
        }
        .playlist-item:active { cursor: grabbing; }
        .playlist-item.dragging { opacity: 0.5; transform: scale(0.98); }
        .playlist-item.active-track { background: var(--bg-color); border-left: 4px solid var(--primary-color); }
        
        .track-info { display: flex; align-items: center; gap: 10px; flex: 1; overflow: hidden; }
        .drag-handle { color: #ccc; cursor: grab; padding: 0 5px; }
        .track-name { 
            font-size: 0.95rem; white-space: nowrap; overflow: hidden; text-overflow: ellipsis; color: #555;
        }
        
        .track-actions { display: flex; align-items: center; gap: 15px; margin-left: 10px; }
        .track-actions i { cursor: pointer; color: #aaa; font-size: 1.1rem; }
        .track-actions i:hover { color: var(--primary-color); }
        .icon-image-active { color: var(--primary-color) !important; }

        /* カラーピッカー（円盤型） */
        .color-ring-container { display: flex; flex-direction: column; align-items: center; justify-content: center; flex: 1; }
        .color-picker-bg {
            width: 280px; height: 280px; border-radius: 50%;
            background: 
                radial-gradient(circle, white 0%, transparent 100%),
                conic-gradient(red, yellow, lime, aqua, blue, magenta, red);
            position: relative; cursor: crosshair;
            box-shadow: 0 10px 30px rgba(0,0,0,0.1);
            border: 4px solid white;
        }
        .color-selector {
            width: 24px; height: 24px; border: 3px solid white; border-radius: 50%;
            position: absolute; 
            top: 50%; left: 50%;
            transform: translate(-50%, -50%);
            pointer-events: none; box-shadow: 0 2px 8px rgba(0,0,0,0.3);
            background: transparent;
        }
        .color-preview {
            margin-top: 20px;
            padding: 10px 20px;
            background: white;
            border-radius: 20px;
            color: var(--primary-color);
            font-weight: bold;
            font-family: 'M PLUS Rounded 1c';
            box-shadow: 0 5px 15px rgba(0,0,0,0.05);
        }

        /* EQ Settings */
        .settings-list { flex: 1; overflow-y: auto; padding-bottom: 30px; }
        .setting-group { background: white; padding: 20px; border-radius: 15px; box-shadow: 0 2px 10px rgba(0,0,0,0.05); margin-bottom: 20px;}
        .eq-slider-row { display: flex; align-items: center; justify-content: space-between; margin-bottom: 12px; }
        .eq-slider-row label { font-size: 0.8rem; color: #666; width: 60px; text-align: right; margin-right: 10px;}
        .eq-slider-row input { flex: 1; }
        .eq-slider-val { font-size: 0.8rem; width: 50px; text-align: right; color: #888;}
        
        .reset-btn {
            width: 100%; padding: 15px; background: #ff6b6b; color: white;
            border: none; border-radius: 12px; font-weight: bold; font-size: 1rem;
            cursor: pointer; font-family: 'M PLUS Rounded 1c';
        }
    </style>
</head>
<body>

    <div class="site-fullscreen-btn" onclick="toggleSiteFullscreen()"><i class="fas fa-expand" id="fsIcon"></i></div>

    <div class="player-container">
        <div class="menu-btn" onclick="toggleMenu()"><i class="fas fa-bars"></i></div>

        <div class="media-wrapper">
            <div class="media-display" id="dropZone">
                <img id="coverImageOverlay">
                <video id="mainVideo" playsinline webkit-playsinline></video>
                <div class="placeholder-text" id="placeholderText">
                    <i class="fas fa-cloud-upload-alt" style="font-size: 3rem; margin-bottom: 10px; opacity: 0.5;"></i><br>
                    Drop Files Here<br>Drag Playlist to Sort
                </div>
            </div>
            <div class="file-name-display" id="fileNameDisplay">No File Selected</div>
        </div>

        <div class="controls-area">
            <div class="progress-container" id="progressContainer">
                <div class="progress-bg">
                    <div class="progress-bar" id="progressBar"></div>
                    <div class="progress-handle" id="progressHandle"><i class="fas fa-paper-plane"></i></div>
                </div>
            </div>
            <div class="time-display">
                <span id="currentTime">0:00</span>
                <span id="duration">0:00</span>
            </div>

            <div class="main-controls">
                <i class="fas fa-backward-step control-btn" onclick="playPrev()" title="前の曲"></i>
                <i class="fas fa-rotate-left control-btn skip-btn" onclick="skipTime(-10)" title="10秒戻る"></i>
                <i class="fas fa-play control-btn play-btn" id="playPauseBtn" onclick="togglePlay()"></i>
                <i class="fas fa-rotate-right control-btn skip-btn" onclick="skipTime(10)" title="10秒進む"></i>
                <i class="fas fa-forward-step control-btn" onclick="playNext()" title="次の曲"></i>
            </div>

            <div class="slider-container">
                <i class="fas fa-tachometer-alt"></i>
                <input type="range" class="common-slider" min="0.5" max="2.0" step="0.1" value="1.0" oninput="setSpeed(this.value)">
                <span id="speedDisplay" style="font-size: 0.8rem; width: 30px; text-align: right;">1.0x</span>
            </div>

            <div class="slider-container">
                <i class="fas fa-volume-high" id="volumeIcon"></i>
                <input type="range" class="common-slider" min="0" max="1" step="0.01" value="1" oninput="setVolume(this.value)">
            </div>
        </div>
    </div>

    <div class="menu-overlay" id="menuOverlay" onclick="toggleMenu()">
        <div class="side-menu" onclick="event.stopPropagation()">
            <div style="align-self: flex-end; font-size: 1.5rem; cursor: pointer; margin-bottom: 10px;" onclick="toggleMenu()"><i class="fas fa-times"></i></div>
            
            <div class="menu-item" onclick="openScreen('playlistScreen')"><i class="fas fa-list-ul"></i> プレイリスト編集</div>
            <div class="menu-item" onclick="openScreen('colorScreen')"><i class="fas fa-palette"></i> テーマ色変更</div>
            <div class="menu-item" onclick="openScreen('settingsScreen')"><i class="fas fa-sliders"></i> 音質設定 (EQ/HRTF)</div>
            
            </div>
    </div>

    <div class="sub-screen" id="playlistScreen">
        <div class="screen-header"><span>プレイリスト</span><i class="fas fa-times" onclick="closeScreen('playlistScreen')"></i></div>
        <div style="margin-bottom: 15px;">
            <input type="file" id="fileInput" multiple accept="video/*,audio/*" style="display: none;" onchange="handleFiles(this.files)">
            <button onclick="document.getElementById('fileInput').click()" style="padding:12px; width:100%; border-radius:10px; border:2px dashed var(--primary-color); background:white; font-weight:bold; color:var(--primary-color); cursor:pointer; font-family: 'M PLUS Rounded 1c';">
                <i class="fas fa-plus-circle"></i> ファイルを追加 (複数可)
            </button>
        </div>
        <p style="font-size:0.8rem; color:#888; text-align:center; margin-bottom:10px;">ドラッグして曲順を変更できます</p>
        <div class="playlist-items" id="playlistContainer"></div>
    </div>

    <input type="file" id="coverInput" accept="image/*" style="display: none;" onchange="handleCoverUpload(this.files)">

    <div class="sub-screen" id="colorScreen">
        <div class="screen-header"><span>テーマカラー変更</span><i class="fas fa-times" onclick="closeScreen('colorScreen')"></i></div>
        <div class="color-ring-container">
            <div class="color-picker-bg" id="colorPickerBg">
                <div class="color-selector" id="colorSelector"></div>
            </div>
            <div class="color-preview" id="colorText">Hue: 0, Sat: 0%</div>
            <p style="margin-top:15px; color:#888; font-size:0.9rem; font-family: 'M PLUS Rounded 1c';">
                円の中をタッチして<br>色相と濃さを決めてね！
            </p>
        </div>
    </div>

    <div class="sub-screen" id="settingsScreen">
        <div class="screen-header"><span>音質調整 (Audio)</span><i class="fas fa-times" onclick="closeScreen('settingsScreen')"></i></div>
        <div class="settings-list">
            
            <div class="setting-group" style="display:flex; align-items:center; justify-content:space-between;">
                <div style="font-weight:bold; color:#555; display:flex; align-items:center; gap:10px;">
                    <i class="fas fa-cube" style="color:var(--primary-color);"></i> 空間オーディオ (HRTF)
                </div>
                <label class="toggle-switch">
                    <input type="checkbox" id="hrtfToggle" onchange="toggleHRTF()">
                    <span class="slider"></span>
                </label>
            </div>

            <div class="setting-group">
                <div class="setting-title" style="text-align:center; font-weight:bold; margin-bottom:15px; color:#555;">10-Band EQ</div>
                <div id="eqContainer"></div>
            </div>

            <button class="reset-btn" onclick="resetSettings()">設定をリセット</button>
        </div>
    </div>

    <script>
        /* === グローバル変数 === */
        const video = document.getElementById('mainVideo');
        const coverImageOverlay = document.getElementById('coverImageOverlay');
        const playPauseBtn = document.getElementById('playPauseBtn');
        const progressBar = document.getElementById('progressBar');
        const progressContainer = document.getElementById('progressContainer');
        const progressHandle = document.getElementById('progressHandle');
        const playlistContainer = document.getElementById('playlistContainer');
        const fileNameDisplay = document.getElementById('fileNameDisplay');
        const hrtfToggle = document.getElementById('hrtfToggle');

        let playlist = [];
        let currentFileIndex = 0;
        let targetCoverIndex = -1;
        
        // Audio Context関連
        let audioCtx;
        let sourceNode, gainNode;
        let pannerNode; // HRTF用
        let eqFilters = [];
        const frequencies = [60, 170, 310, 600, 1000, 3000, 6000, 12000, 14000, 16000];

        /* === オーディオ初期化 & HRTF === */
        function initAudioContext() {
            if (!audioCtx) {
                const AudioContext = window.AudioContext || window.webkitAudioContext;
                audioCtx = new AudioContext();
                sourceNode = audioCtx.createMediaElementSource(video);
                gainNode = audioCtx.createGain();

                // HRTF Panner Setup
                pannerNode = audioCtx.createPanner();
                pannerNode.panningModel = 'HRTF';
                pannerNode.distanceModel = 'inverse';
                pannerNode.refDistance = 1;
                pannerNode.maxDistance = 10000;
                pannerNode.rolloffFactor = 1;
                pannerNode.coneInnerAngle = 360;
                pannerNode.coneOuterAngle = 0;
                pannerNode.coneOuterGain = 0;
                pannerNode.setPosition(0, 0, 0.1);

                // EQ Chain creation
                let prevNode = sourceNode;
                frequencies.forEach(freq => {
                    const filter = audioCtx.createBiquadFilter();
                    filter.type = 'peaking'; filter.frequency.value = freq; filter.Q.value = 1; filter.gain.value = 0;
                    prevNode.connect(filter); prevNode = filter;
                    eqFilters.push(filter);
                });

                // 接続: EQの最後 -> 接続先制御へ
                prevNode.connect(gainNode);
                updateAudioGraph();
            }
            if(audioCtx.state === 'suspended') audioCtx.resume();
        }

        // 接続ルートの切り替え
        function updateAudioGraph() {
            if (!audioCtx) return;
            gainNode.disconnect();
            pannerNode.disconnect();

            if (hrtfToggle.checked) {
                // HRTF ON: Gain -> Panner -> Dest
                gainNode.connect(pannerNode);
                pannerNode.connect(audioCtx.destination);
            } else {
                // HRTF OFF: Gain -> Dest
                gainNode.connect(audioCtx.destination);
            }
        }

        function toggleHRTF() {
            initAudioContext();
            updateAudioGraph();
        }

        /* === プレイヤー制御 === */
        function togglePlay() {
            if (playlist.length === 0) return alert("ファイルを追加してください");
            initAudioContext();
            if (video.paused) {
                video.play().then(() => playPauseBtn.className = "fas fa-pause control-btn play-btn");
            } else {
                video.pause(); playPauseBtn.className = "fas fa-play control-btn play-btn";
            }
        }

        function playFile(index) {
            if(index < 0 || index >= playlist.length) return;
            currentFileIndex = index;
            const item = playlist[index];
            
            fileNameDisplay.innerText = item.name;
            video.src = URL.createObjectURL(item.file);
            
            if (item.coverImage) {
                coverImageOverlay.src = item.coverImage;
                coverImageOverlay.style.display = 'block';
            } else {
                coverImageOverlay.style.display = 'none';
                coverImageOverlay.src = "";
            }

            video.load();
            document.getElementById('placeholderText').style.display = 'none';
            renderPlaylist();
            setTimeout(() => { togglePlay(); if(video.paused) playPauseBtn.className = "fas fa-play control-btn play-btn"; }, 100);
        }

        function playNext() { 
            let next = currentFileIndex + 1; 
            if(next >= playlist.length) next = 0; 
            playFile(next); 
        }
        function playPrev() { 
            let prev = currentFileIndex - 1; 
            if(prev < 0) prev = playlist.length - 1; 
            playFile(prev); 
        }

        function skipTime(seconds) {
            if(video.duration) {
                video.currentTime = Math.min(Math.max(video.currentTime + seconds, 0), video.duration);
            }
        }

        /* === シークバー・音量・速度 === */
        let isDraggingProgress = false;
        function updateProgressUI(percent) {
            progressBar.style.width = percent + "%";
            progressHandle.style.left = percent + "%";
        }
        video.addEventListener('timeupdate', () => {
            if(!isDraggingProgress && video.duration) {
                const percent = (video.currentTime / video.duration) * 100 || 0;
                updateProgressUI(percent);
                document.getElementById('currentTime').innerText = formatTime(video.currentTime);
                document.getElementById('duration').innerText = formatTime(video.duration || 0);
            }
        });
        video.addEventListener('ended', playNext);

        function seekVideo(e) {
            if (!video.duration) return;
            const rect = progressContainer.getBoundingClientRect();
            let x = (e.clientX || e.touches[0].clientX) - rect.left;
            if (x < 0) x = 0; if (x > rect.width) x = rect.width;
            const percent = (x / rect.width) * 100;
            updateProgressUI(percent);
            const time = (x / rect.width) * video.duration;
            document.getElementById('currentTime').innerText = formatTime(time);
            if(!isDraggingProgress) video.currentTime = time; 
        }
        
        progressContainer.addEventListener('mousedown', e => { isDraggingProgress = true; seekVideo(e); });
        window.addEventListener('mousemove', e => { if(isDraggingProgress) seekVideo(e); });
        window.addEventListener('mouseup', e => { 
            if(isDraggingProgress) { 
                isDraggingProgress = false; 
                const rect=progressContainer.getBoundingClientRect(); 
                let x = e.clientX - rect.left; 
                if(x<0)x=0; if(x>rect.width)x=rect.width; 
                video.currentTime = (x/rect.width)*video.duration; 
            } 
        });
        progressContainer.addEventListener('touchstart', e => { isDraggingProgress = true; seekVideo(e); }, {passive:false});
        window.addEventListener('touchmove', e => { if(isDraggingProgress) { e.preventDefault(); seekVideo(e); } }, {passive:false});
        window.addEventListener('touchend', e => { 
            if(isDraggingProgress){
                isDraggingProgress = false; 
                if(e.changedTouches.length>0){
                    const rect=progressContainer.getBoundingClientRect(); 
                    let x=e.changedTouches[0].clientX - rect.left;
                    if(x<0)x=0; if(x>rect.width)x=rect.width; 
                    video.currentTime=(x/rect.width)*video.duration; 
                }
            } 
        });

        function setVolume(val) { 
            if(video) video.volume = val; 
            const vIcon = document.getElementById('volumeIcon');
            if(val==0) vIcon.className="fas fa-volume-xmark"; 
            else if(val<0.5) vIcon.className="fas fa-volume-low"; 
            else vIcon.className="fas fa-volume-high"; 
        }
        function setSpeed(val) { video.playbackRate = val; document.getElementById('speedDisplay').innerText = parseFloat(val).toFixed(1) + "x"; }
        function formatTime(s) { const m = Math.floor(s / 60); const sc = Math.floor(s % 60); return `${m}:${sc < 10 ? '0' : ''}${sc}`; }

        /* === プレイリスト & ドラッグ＆ドロップ (複数対応) === */
        const dropZone = document.getElementById('dropZone');
        ['dragenter', 'dragover'].forEach(eventName => dropZone.addEventListener(eventName, (e) => { e.preventDefault(); dropZone.classList.add('drag-active'); }, false));
        ['dragleave', 'drop'].forEach(eventName => dropZone.addEventListener(eventName, (e) => { e.preventDefault(); dropZone.classList.remove('drag-active'); }, false));
        dropZone.addEventListener('drop', (e) => { handleFiles(e.dataTransfer.files); });

        function handleFiles(files) {
            initAudioContext();
            let firstAdd = playlist.length === 0;
            // 複数ファイルをループ処理して追加
            for(let i=0; i<files.length; i++) {
                playlist.push({
                    file: files[i],
                    name: files[i].name,
                    coverImage: null
                });
            }
            renderPlaylist();
            if(firstAdd && playlist.length > 0) playFile(0);
            if(!firstAdd) {
                document.getElementById('playlistScreen').classList.add('active');
                document.getElementById('menuOverlay').classList.add('active');
            }
        }

        /* === プレイリスト並び替え (Drag & Drop) === */
        let draggingIndex = null;

        function renderPlaylist() {
            playlistContainer.innerHTML = "";
            playlist.forEach((item, index) => {
                const div = document.createElement('div');
                div.className = `playlist-item ${index === currentFileIndex ? 'active-track' : ''}`;
                div.draggable = true;
                
                div.addEventListener('dragstart', (e) => { draggingIndex = index; div.classList.add('dragging'); });
                div.addEventListener('dragend', () => { div.classList.remove('dragging'); draggingIndex = null; });
                div.addEventListener('dragover', (e) => { e.preventDefault(); });
                div.addEventListener('drop', (e) => {
                    e.preventDefault();
                    if (draggingIndex === null || draggingIndex === index) return;
                    
                    const movedItem = playlist.splice(draggingIndex, 1)[0];
                    playlist.splice(index, 0, movedItem);

                    if (currentFileIndex === draggingIndex) {
                        currentFileIndex = index;
                    } else if (currentFileIndex > draggingIndex && currentFileIndex <= index) {
                        currentFileIndex--;
                    } else if (currentFileIndex < draggingIndex && currentFileIndex >= index) {
                        currentFileIndex++;
                    }

                    renderPlaylist();
                });

                const hasCover = !!item.coverImage;
                const imgIconClass = hasCover ? "fas fa-image icon-image-active" : "far fa-image";
                const deleteBtnHtml = hasCover 
                    ? `<i class="fas fa-minus-circle" style="color:#ff6b6b; font-size:0.9rem;" onclick="removeCover(${index})" title="画像削除"></i>` 
                    : ``;

                div.innerHTML = `
                    <div class="track-info">
                        <i class="fas fa-grip-lines drag-handle"></i>
                        <span class="track-name" onclick="playFile(${index})">${index+1}. ${item.name}</span>
                    </div>
                    <div class="track-actions">
                        <i class="${imgIconClass}" onclick="selectCover(${index})" title="画像設定"></i>
                        ${deleteBtnHtml}
                        <i class="fas fa-trash-alt" onclick="removeFile(${index})" title="削除"></i>
                    </div>`;
                playlistContainer.appendChild(div);
            });
        }

        /* === 画像管理 === */
        function selectCover(index) { targetCoverIndex = index; document.getElementById('coverInput').click(); }
        function handleCoverUpload(files) {
            if (files.length === 0 || targetCoverIndex === -1) return;
            const reader = new FileReader();
            reader.onload = function(e) {
                playlist[targetCoverIndex].coverImage = e.target.result;
                if (targetCoverIndex === currentFileIndex) {
                    coverImageOverlay.src = e.target.result; coverImageOverlay.style.display = 'block';
                }
                renderPlaylist();
                targetCoverIndex = -1; document.getElementById('coverInput').value = "";
            };
            reader.readAsDataURL(files[0]);
        }
        function removeCover(index) {
            playlist[index].coverImage = null;
            if (index === currentFileIndex) { coverImageOverlay.style.display = 'none'; coverImageOverlay.src = ""; }
            renderPlaylist();
        }
        function removeFile(index) { 
            playlist.splice(index, 1); 
            if(index === currentFileIndex) { 
                if(playlist.length > 0) playNext(); 
                else { 
                    video.src=""; coverImageOverlay.style.display='none'; fileNameDisplay.innerText="No File"; 
                    document.getElementById('placeholderText').style.display='block'; 
                    playPauseBtn.className="fas fa-play control-btn play-btn"; 
                    progressBar.style.width="0%"; 
                } 
            } else if (index < currentFileIndex) { currentFileIndex--; }
            renderPlaylist(); 
        }

        /* === カラーピッカー === */
        const colorPickerBg = document.getElementById('colorPickerBg');
        const colorSelector = document.getElementById('colorSelector');
        const colorText = document.getElementById('colorText');
        
        function updateColor(e) {
            const rect = colorPickerBg.getBoundingClientRect();
            const x = (e.touches ? e.touches[0].clientX : e.clientX) - (rect.left + rect.width/2);
            const y = (e.touches ? e.touches[0].clientY : e.clientY) - (rect.top + rect.height/2);
            
            let angle = Math.atan2(y, x) * (180 / Math.PI) + 90;
            if(angle < 0) angle += 360;

            const radius = rect.width / 2;
            let dist = Math.sqrt(x*x + y*y);
            if (dist > radius) dist = radius;

            const sat = Math.round((dist / radius) * 100);
            
            const newColor = `hsl(${Math.round(angle)}, ${sat}%, 75%)`;
            const newBg = `hsl(${Math.round(angle)}, ${sat}%, 95%)`;

            document.documentElement.style.setProperty('--primary-color', newColor);
            document.documentElement.style.setProperty('--bg-color', newBg);
            
            colorText.innerText = `Hue: ${Math.round(angle)}°, Sat: ${sat}%`;
            colorText.style.color = newColor;

            const currentRad = Math.atan2(y, x);
            const setX = Math.cos(currentRad) * dist;
            const setY = Math.sin(currentRad) * dist;

            colorSelector.style.transform = `translate(calc(-50% + ${setX}px), calc(-50% + ${setY}px))`;
        }

        let isDragColor = false;
        ['mousedown','touchstart'].forEach(ev=>colorPickerBg.addEventListener(ev,e=>{isDragColor=true;updateColor(e);}));
        ['mousemove','touchmove'].forEach(ev=>window.addEventListener(ev,e=>{if(isDragColor){e.preventDefault(); updateColor(e);}}));
        ['mouseup','touchend'].forEach(ev=>window.addEventListener(ev,()=>{isDragColor=false}));

        /* === EQ & UI === */
        const eqContainer = document.getElementById('eqContainer');
        frequencies.forEach((freq, i) => {
            eqContainer.innerHTML += `<div class="eq-slider-row"><label>${freq < 1000 ? freq : freq/1000 + 'k'}Hz</label><input type="range" class="common-slider" min="-12" max="12" value="0" step="1" oninput="updateEQ(${i}, this.value)"><span class="eq-slider-val" id="eqVal${i}">0dB</span></div>`;
        });
        function updateEQ(i, val) { if(eqFilters[i]) eqFilters[i].gain.value = val; document.getElementById(`eqVal${i}`).innerText = val > 0 ? `+${val}dB` : `${val}dB`; }
        
        function resetSettings() {
            video.playbackRate = 1.0; video.volume = 1.0;
            document.querySelector('input[oninput="setSpeed(this.value)"]').value = 1.0; document.getElementById('speedDisplay').innerText="1.0x";
            document.querySelector('input[oninput="setVolume(this.value)"]').value = 1.0; setVolume(1.0);
            eqFilters.forEach((f, i) => { f.gain.value = 0; document.getElementById(`eqVal${i}`).innerText = "0dB"; });
            eqContainer.querySelectorAll('input[type="range"]').forEach(inpt => inpt.value = 0);
        }

        function toggleMenu() { document.getElementById('menuOverlay').classList.toggle('active'); }
        function openScreen(id) { document.getElementById(id).classList.add('active'); toggleMenu(); }
        function closeScreen(id) { document.getElementById(id).classList.remove('active'); }
        function toggleSiteFullscreen() {
            if (!document.fullscreenElement) { document.documentElement.requestFullscreen(); document.getElementById('fsIcon').className = "fas fa-compress"; }
            else { if (document.exitFullscreen) document.exitFullscreen(); document.getElementById('fsIcon').className = "fas fa-expand"; }
        }
    </script>
</body>
</html>
