
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>Our Space — A little space that's just yours.</title>
    <!-- Include PeerJS for real-time peer-to-peer communication -->
    <script src="https://unpkg.com/peerjs@1.4.7/dist/peerjs.min.js"></script>
    <style>
        :root {
            --bg-canvas: #FFFFFF;
            --ui-bg: #F7F7F8;
            --ui-border: #E5E5EA;
            --text-main: #1C1C1E;
            --text-muted: #8E8E93;
            --accent-pink: #FFD1DC;
            --accent-red: #FF3B30;
            --accent-purple: #AF52DE;
            --accent-blue: #007AFF;
            --accent-green: #34C759;
            --accent-yellow: #FFCC00;
        }

        * {
            box-sizing: border-box;
            margin: 0;
            padding: 0;
            font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, Helvetica, Arial, sans-serif;
            -webkit-tap-highlight-color: transparent;
        }

        body {
            background-color: #EFEFF4;
            color: var(--text-main);
            display: flex;
            justify-content: center;
            align-items: center;
            min-height: 100vh;
            overflow-y: auto;
            padding: 20px 0;
        }

        .phone-frame {
            width: 100%;
            max-width: 414px;
            height: 100%;
            max-height: 896px;
            background: var(--bg-canvas);
            position: relative;
            display: flex;
            flex-direction: column;
            box-shadow: 0 20px 40px rgba(0,0,0,0.1);
            overflow: hidden;
        }

        @media (min-width: 415px) {
            .phone-frame {
                border-radius: 40px;
                border: 10px solid #1C1C1E;
                height: 844px;
                max-height: 90vh;
            }
        }

        .screen {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: var(--bg-canvas);
            display: flex;
            flex-direction: column;
            z-index: 10;
            transition: opacity 0.3s ease;
        }

        .screen.hidden {
            opacity: 0;
            pointer-events: none;
            z-index: 0;
        }

        .onboarding-content {
            flex: 1;
            display: flex;
            flex-direction: column;
            justify-content: center;
            align-items: center;
            padding: 40px;
            text-align: center;
        }

        .onboarding-title {
            font-size: 28px;
            font-weight: 700;
            letter-spacing: -0.5px;
            margin-bottom: 12px;
        }

        .onboarding-subtitle {
            font-size: 16px;
            color: var(--text-muted);
            line-height: 1.4;
            margin-bottom: 30px;
        }

        .btn {
            background: var(--text-main);
            color: white;
            border: none;
            border-radius: 24px;
            padding: 16px 32px;
            font-size: 16px;
            font-weight: 600;
            cursor: pointer;
            width: 100%;
            max-width: 280px;
            transition: transform 0.1s ease, opacity 0.2s;
            margin-top: 10px;
        }

        .btn:active {
            transform: scale(0.97);
        }

        .btn-secondary {
            background: var(--ui-bg);
            color: var(--text-main);
        }

        .input-field {
            width: 100%;
            max-width: 280px;
            padding: 16px;
            border-radius: 16px;
            border: 1px solid var(--ui-border);
            font-size: 16px;
            text-align: center;
            outline: none;
            margin-bottom: 10px;
            background: var(--ui-bg);
        }

        .app-header {
            height: 60px;
            display: flex;
            align-items: center;
            justify-content: space-between;
            padding: 0 20px;
            border-bottom: 1px solid var(--ui-border);
            z-index: 5;
            background: rgba(255,255,255,0.9);
            backdrop-filter: blur(10px);
        }

        .presence-pill {
            font-size: 13px;
            color: var(--text-muted);
            display: flex;
            align-items: center;
            gap: 6px;
            background: var(--ui-bg);
            padding: 6px 12px;
            border-radius: 20px;
        }

        .presence-dot {
            width: 8px;
            height: 8px;
            background: var(--accent-red);
            border-radius: 50%;
            transition: background 0.3s;
        }
        .presence-dot.connected {
            background: var(--accent-green);
        }

        .header-actions {
            display: flex;
            gap: 12px;
        }

        .icon-btn {
            background: none;
            border: none;
            font-size: 20px;
            cursor: pointer;
            padding: 4px;
        }

        .canvas-container {
            flex: 1;
            position: relative;
            overflow: hidden;
            touch-action: none;
        }

        canvas {
            display: block;
            width: 100%;
            height: 100%;
        }

        .empty-canvas-msg {
            position: absolute;
            top: 40%;
            left: 50%;
            transform: translate(-50%, -50%);
            text-align: center;
            pointer-events: none;
            color: var(--text-muted);
            transition: opacity 0.5s;
        }

        .empty-canvas-msg h3 {
            font-size: 18px;
            font-weight: 500;
            color: var(--text-main);
            margin-bottom: 4px;
        }

        .floating-heart-btn {
            position: absolute;
            bottom: 90px;
            right: 20px;
            width: 56px;
            height: 56px;
            background: white;
            border: 1px solid var(--ui-border);
            border-radius: 50%;
            box-shadow: 0 4px 12px rgba(0,0,0,0.08);
            display: flex;
            align-items: center;
            justify-content: center;
            font-size: 24px;
            cursor: pointer;
            z-index: 5;
            transition: transform 0.2s;
        }

        .floating-heart-btn:active {
            transform: scale(0.9);
        }

        /* Stickers & Messaging Action bar */
        .extra-actions-bar {
            position: absolute;
            top: 15px;
            left: 20px;
            display: flex;
            gap: 8px;
            z-index: 5;
        }

        .pill-btn {
            background: rgba(255,255,255,0.9);
            border: 1px solid var(--ui-border);
            padding: 6px 14px;
            border-radius: 20px;
            font-size: 13px;
            font-weight: 500;
            cursor: pointer;
            box-shadow: 0 2px 8px rgba(0,0,0,0.05);
            backdrop-filter: blur(8px);
        }

        /* Sticker Tray Popup */
        .sticker-tray {
            position: absolute;
            bottom: 90px;
            left: 20px;
            background: white;
            border: 1px solid var(--ui-border);
            border-radius: 20px;
            padding: 12px;
            display: grid;
            grid-template-columns: repeat(4, 1fr);
            gap: 8px;
            box-shadow: 0 10px 25px rgba(0,0,0,0.1);
            z-index: 10;
            display: none;
        }

        .sticker-option {
            background: var(--ui-bg);
            border: none;
            font-size: 24px;
            padding: 8px;
            border-radius: 12px;
            cursor: pointer;
            transition: transform 0.1s;
        }

        .sticker-option:active {
            transform: scale(0.9);
        }

        /* Draggable / Placed Stickers on Canvas */
        .placed-sticker {
            position: absolute;
            font-size: 36px;
            cursor: grab;
            user-select: none;
            z-index: 4;
            transform: translate(-50%, -50%);
            animation: popIn 0.2s ease-out;
        }

        @keyframes popIn {
            0% { transform: translate(-50%, -50%) scale(0); }
            80% { transform: translate(-50%, -50%) scale(1.2); }
            100% { transform: translate(-50%, -50%) scale(1); }
        }

        /* Chat Tab Styling */
        .chat-container {
            flex: 1;
            display: flex;
            flex-direction: column;
            height: 100%;
        }

        .chat-messages {
            flex: 1;
            padding: 16px;
            overflow-y: auto;
            display: flex;
            flex-direction: column;
            gap: 10px;
        }

        .chat-bubble {
            max-width: 75%;
            padding: 10px 14px;
            border-radius: 16px;
            font-size: 14px;
            line-height: 1.4;
            word-break: break-word;
        }

        .chat-bubble.mine {
            background: var(--text-main);
            color: white;
            align-self: flex-end;
            border-bottom-right-radius: 4px;
        }

        .chat-bubble.theirs {
            background: var(--ui-bg);
            color: var(--text-main);
            align-self: flex-start;
            border-bottom-left-radius: 4px;
        }

        .chat-input-area {
            padding: 12px 16px;
            border-top: 1px solid var(--ui-border);
            display: flex;
            gap: 8px;
            background: white;
        }

        .chat-input {
            flex: 1;
            padding: 10px 14px;
            border-radius: 20px;
            border: 1px solid var(--ui-border);
            outline: none;
            font-size: 14px;
            background: var(--ui-bg);
        }

        .send-msg-btn {
            background: var(--text-main);
            color: white;
            border: none;
            padding: 0 16px;
            border-radius: 20px;
            font-weight: 600;
            font-size: 14px;
            cursor: pointer;
        }

        .toolbar {
            position: absolute;
            bottom: 20px;
            left: 50%;
            transform: translateX(-50%);
            background: rgba(255, 255, 255, 0.9);
            backdrop-filter: blur(12px);
            border: 1px solid var(--ui-border);
            padding: 8px 16px;
            border-radius: 30px;
            display: flex;
            align-items: center;
            gap: 16px;
            box-shadow: 0 8px 24px rgba(0,0,0,0.06);
            z-index: 5;
        }

        .tool-btn {
            background: none;
            border: none;
            font-size: 18px;
            cursor: pointer;
            padding: 6px;
            border-radius: 50%;
            transition: background 0.2s;
        }

        .tool-btn.active {
            background: var(--ui-bg);
        }

        .color-dot {
            width: 20px;
            height: 20px;
            border-radius: 50%;
            cursor: pointer;
            border: 2px solid transparent;
        }

        .color-dot.active {
            border-color: var(--text-main);
            transform: scale(1.1);
        }

        .bottom-nav {
            height: 64px;
            background: var(--bg-canvas);
            border-top: 1px solid var(--ui-border);
            display: flex;
            justify-content: space-around;
            align-items: center;
            z-index: 5;
            flex-shrink: 0;
        }

        .nav-item {
            background: none;
            border: none;
            font-size: 12px;
            color: var(--text-muted);
            cursor: pointer;
            display: flex;
            flex-direction: column;
            align-items: center;
            gap: 4px;
        }

        .nav-item.active {
            color: var(--text-main);
            font-weight: 600;
        }

        .nav-item span.icon {
            font-size: 20px;
        }

        .tab-view {
            flex: 1;
            padding: 20px;
            overflow-y: auto;
            display: none;
        }

        .tab-view.active-tab {
            display: block;
        }

        .section-title {
            font-size: 22px;
            font-weight: 700;
            margin-bottom: 16px;
            letter-spacing: -0.5px;
        }

        .moment-card, .memory-card {
            background: var(--ui-bg);
            padding: 16px;
            border-radius: 16px;
            margin-bottom: 12px;
            display: flex;
            align-items: center;
            gap: 12px;
            font-size: 15px;
        }

        .moment-icon {
            font-size: 20px;
        }

        @keyframes floatUp {
            0% { transform: translateY(0) scale(0.5); opacity: 1; }
            100% { transform: translateY(-150px) scale(1.4); opacity: 0; }
        }

        .floating-heart-anim {
            position: absolute;
            font-size: 28px;
            pointer-events: none;
            animation: floatUp 1.5s ease-out forwards;
            z-index: 100;
        }
    </style>
</head>
<body>

<div class="phone-frame">

    <!-- SCREEN 1: ONBOARDING WELCOME -->
    <div id="screen-welcome" class="screen">
        <div class="onboarding-content">
            <div style="font-size: 48px; margin-bottom: 24px;">✨</div>
            <h1 class="onboarding-title">A little space that's just yours.</h1>
            <p class="onboarding-subtitle">Draw live together, chat, drop stickers, and share moments.</p>
            <button class="btn" onclick="goToScreen('screen-name')">Get Started</button>
        </div>
    </div>

    <!-- SCREEN 2: CHOOSE NICKNAME -->
    <div id="screen-name" class="screen hidden">
        <div class="onboarding-content">
            <h2 class="onboarding-title">What should we call you?</h2>
            <p class="onboarding-subtitle">Enter your name for your shared space.</p>
            <input type="text" id="nickname-input" class="input-field" placeholder="e.g. Alex" value="Alex">
            <button class="btn" onclick="saveNickname()">Continue</button>
        </div>
    </div>

    <!-- SCREEN 3: CONNECT SPACE -->
    <div id="screen-connect" class="screen hidden">
        <div class="onboarding-content">
            <h2 class="onboarding-title">Your Live Space</h2>
            <p class="onboarding-subtitle" id="connection-status-text">Create a space or connect using your partner's code.</p>
            
            <div id="host-controls">
                <button class="btn" onclick="hostSpace()">Create a Space</button>
            </div>
            
            <div id="join-controls" style="margin-top: 15px; width: 100%; display: flex; flex-direction: column; align-items: center;">
                <input type="text" id="room-id-input" class="input-field" placeholder="Enter Partner's Space ID">
                <button class="btn btn-secondary" onclick="joinSpace()">Join Partner's Space</button>
            </div>
        </div>
    </div>

    <!-- MAIN APP INTERFACE -->
    <div id="screen-app" class="screen hidden" style="display: flex; flex-direction: column;">
        
        <!-- Header -->
        <header class="app-header">
            <div class="presence-pill">
                <div id="presence-dot" class="presence-dot"></div>
                <span id="presence-text">Waiting for partner...</span>
            </div>
            <div class="header-actions">
                <button class="icon-btn" onclick="triggerThinkingOfYou()" title="Thinking of You">💭</button>
                <button class="icon-btn" onclick="openSettings()">⚙️</button>
            </div>
        </header>

        <!-- Tab 1: Canvas (Default) -->
        <div id="tab-canvas" class="canvas-container tab-view active-tab" style="padding:0; display:block;">
            
            <!-- Extra Interactive Tools Overlay -->
            <div class="extra-actions-bar">
                <button class="pill-btn" onclick="toggleStickerTray()">✨ Stickers</button>
            </div>

            <!-- Sticker Tray Popup -->
            <div id="sticker-tray" class="sticker-tray">
                <button class="sticker-option" onclick="placeSticker('💖')">💖</button>
                <button class="sticker-option" onclick="placeSticker('⭐')">⭐</button>
                <button class="sticker-option" onclick="placeSticker('🐱')">🐱</button>
                <button class="sticker-option" onclick="placeSticker('☕')">☕</button>
                <button class="sticker-option" onclick="placeSticker('💌')">💌</button>
                <button class="sticker-option" onclick="placeSticker('🌸')">🌸</button>
                <button class="sticker-option" onclick="placeSticker('🍕')">🍕</button>
                <button class="sticker-option" onclick="placeSticker('🔥')">🔥</button>
            </div>

            <div id="empty-msg" class="empty-canvas-msg">
                <h3>This space is yours.</h3>
                <p>Draw and drop stickers together live!</p>
            </div>
            <canvas id="drawing-canvas"></canvas>

            <!-- Floating ❤️ Button -->
            <button class="floating-heart-btn" onclick="sendHeartEffect(true)">❤️</button>

            <!-- Drawing Toolbar -->
            <div class="toolbar">
                <button class="tool-btn active" id="tool-pen" onclick="setTool('pen')">✏️</button>
                <button class="tool-btn" id="tool-eraser" onclick="setTool('eraser')">🧹</button>
                <div style="width: 1px; height: 20px; background: var(--ui-border);"></div>
                <div class="color-dot active" style="background: #1C1C1E;" onclick="setColor('#1C1C1E', this)"></div>
                <div class="color-dot" style="background: #FF3B30;" onclick="setColor('#FF3B30', this)"></div>
                <div class="color-dot" style="background: #AF52DE;" onclick="setColor('#AF52DE', this)"></div>
                <div class="color-dot" style="background: #007AFF;" onclick="setColor('#007AFF', this)"></div>
                <div class="color-dot" style="background: #FFD1DC;" onclick="setColor('#FFD1DC', this)"></div>
                <button class="tool-btn" onclick="clearCanvas(true)" style="font-size: 14px; margin-left: 4px;">Clear</button>
            </div>
        </div>

        <!-- Tab 2: Chat / Messages -->
        <div id="tab-chat" class="tab-view" style="padding: 0;">
            <div class="chat-container">
                <div id="chat-messages" class="chat-messages">
                    <div class="chat-bubble theirs">Hey! Welcome to our shared space. Drop a message or check out the canvas!</div>
                </div>
                <div class="chat-input-area">
                    <input type="text" id="chat-input-field" class="chat-input" placeholder="Type a message..." onkeydown="if(event.key === 'Enter') sendChatMessage()">
                    <button class="send-msg-btn" onclick="sendChatMessage()">Send</button>
                </div>
            </div>
        </div>

        <!-- Tab 3: Moments -->
        <div id="tab-moments" class="tab-view">
            <h2 class="section-title">Little Moments</h2>
            <div id="moments-list">
                <div class="moment-card">
                    <span class="moment-icon">✨</span>
                    <div>Space initialized successfully</div>
                </div>
            </div>
        </div>

        <!-- Bottom Navigation Bar -->
        <nav class="bottom-nav">
            <button class="nav-item active" onclick="switchTab('canvas', this)">
                <span class="icon">✏️</span>
                Canvas
            </button>
            <button class="nav-item" onclick="switchTab('chat', this)">
                <span class="icon">💬</span>
                Chat
            </button>
            <button class="nav-item" onclick="switchTab('moments', this)">
                <span class="icon">✨</span>
                Moments
            </button>
        </nav>

    </div>

</div>

<script>
    let userNickname = "Alex";
    let partnerName = "Partner";
    let peer, conn;
    let isHost = false;

    function goToScreen(screenId) {
        document.querySelectorAll('.screen').forEach(s => s.classList.add('hidden'));
        document.getElementById(screenId).classList.remove('hidden');
    }

    function saveNickname() {
        const input = document.getElementById('nickname-input').value;
        if(input) userNickname = input;
        goToScreen('screen-connect');
    }

    // --- PeerJS Real-Time Setup ---
    function initPeer(customId = null) {
        peer = customId ? new Peer(customId) : new Peer();

        peer.on('open', (id) => {
            if(isHost) {
                document.getElementById('connection-status-text').innerHTML = `Your Space Code: <b>${id}</b><br>Share this code with your partner!`;
            }
        });

        peer.on('connection', (connection) => {
            conn = connection;
            setupConnectionListeners();
            document.getElementById('connection-status-text').innerText = "Partner connected!";
            setTimeout(() => {
                goToScreen('screen-app');
                initCanvas();
            }, 1000);
        });
    }

    function hostSpace() {
        isHost = true;
        const randomCode = userNickname.toLowerCase() + '-' + Math.floor(1000 + Math.random() * 9000);
        initPeer(randomCode);
    }

    function joinSpace() {
        isHost = false;
        const roomId = document.getElementById('room-id-input').value.trim();
        if(!roomId) {
            alert("Please enter a valid space ID");
            return;
        }
        peer = new Peer();
        peer.on('open', () => {
            conn = peer.connect(roomId);
            setupConnectionListeners();
            goToScreen('screen-app');
            initCanvas();
        });
    }

    function setupConnectionListeners() {
        conn.on('open', () => {
            document.getElementById('presence-dot').classList.add('connected');
            document.getElementById('presence-text').innerText = "Connected live";
            addMoment('✨', 'You and your partner are connected live!');
            conn.send({ type: 'INIT_NAME', name: userNickname });
        });

        conn.on('data', (data) => {
            handleIncomingData(data);
        });

        conn.on('close', () => {
            document.getElementById('presence-dot').classList.remove('connected');
            document.getElementById('presence-text').innerText = "Partner disconnected";
            addMoment('⚠️', 'Partner disconnected.');
        });
    }

    function handleIncomingData(data) {
        if(data.type === 'INIT_NAME') {
            partnerName = data.name;
            document.getElementById('presence-text').innerText = `${partnerName} is here`;
        } else if (data.type === 'DRAW') {
            drawRemoteLine(data.x0, data.y0, data.x1, data.y1, data.color, data.size, data.isEraser);
        } else if (data.type === 'CLEAR') {
            executeClearCanvas();
        } else if (data.type === 'HEART') {
            spawnFloatingHeart(data.x, data.y);
            addMoment('❤️', `${partnerName} sent you a heart!`);
        } else if (data.type === 'CHAT') {
            appendChatMessage(data.message, 'theirs');
            addMoment('💬', `${partnerName} sent a message`);
        } else if (data.type === 'STICKER') {
            renderSticker(data.emoji, data.x, data.y);
            addMoment('✨', `${partnerName} placed a sticker!`);
        } else if (data.type === 'THINKING') {
            alert(`💖 Notification from ${partnerName}: Thinking of you!`);
            addMoment('💭', `${partnerName} is thinking of you.`);
        }
    }

    // --- Tabs Management ---
    function switchTab(tabName, btnElement) {
        document.querySelectorAll('.tab-view').forEach(t => {
            t.classList.remove('active-tab');
            t.style.display = 'none';
        });
        document.querySelectorAll('.nav-item').forEach(n => n.classList.remove('active'));

        const targetTab = document.getElementById('tab-' + tabName);
        targetTab.classList.add('active-tab');
        if(tabName === 'canvas' || tabName === 'chat') targetTab.style.display = 'block';
        btnElement.classList.add('active');
    }

    // --- Chat System ---
    function sendChatMessage() {
        const inputField = document.getElementById('chat-input-field');
        const text = inputField.value.trim();
        if(!text) return;

        appendChatMessage(text, 'mine');
        inputField.value = '';

        if(conn && conn.open) {
            conn.send({ type: 'CHAT', message: text });
        }
    }

    function appendChatMessage(text, senderClass) {
        const container = document.getElementById('chat-messages');
        const bubble = document.createElement('div');
        bubble.className = `chat-bubble ${senderClass}`;
        bubble.innerText = text;
        container.appendChild(bubble);
        container.scrollTop = container.scrollHeight;
    }

    // --- Sticker Tray System ---
    function toggleStickerTray() {
        const tray = document.getElementById('sticker-tray');
        tray.style.display = tray.style.display === 'grid' ? 'none' : 'grid';
    }

    function placeSticker(emoji) {
        toggleStickerTray();
        const container = document.querySelector('.canvas-container');
        const rect = container.getBoundingClientRect();
        const x = rect.width / 2;
        const y = rect.height / 2;

        renderSticker(emoji, x, y);

        if(conn && conn.open) {
            conn.send({ type: 'STICKER', emoji, x, y });
        }
    }

    function renderSticker(emoji, x, y) {
        const container = document.querySelector('.canvas-container');
        const sticker = document.createElement('div');
        sticker.className = 'placed-sticker';
        sticker.innerText = emoji;
        sticker.style.left = `${x}px`;
        sticker.style.top = `${y}px`;
        
        // Simple drag functionality for stickers inside canvas
        let isDraggingSticker = false;
        sticker.onmousedown = (e) => { isDraggingSticker = true; e.stopPropagation(); };
        window.onmouseup = () => { isDraggingSticker = false; };
        window.onmousemove = (e) => {
            if(isDraggingSticker) {
                const rect = container.getBoundingClientRect();
                sticker.style.left = `${e.clientX - rect.left}px`;
                sticker.style.top = `${e.clientY - rect.top}px`;
            }
        };

        container.appendChild(sticker);
    }

    // --- Canvas & Real-Time Drawing Engine ---
    let canvas, ctx;
    let isDrawing = false;
    let currentTool = 'pen';
    let currentColor = '#1C1C1E';
    let currentSize = 3;
    let hasDrawn = false;
    let lastX = 0, lastY = 0;

    function initCanvas() {
        canvas = document.getElementById('drawing-canvas');
        ctx = canvas.getContext('2d');
        resizeCanvas();

        window.addEventListener('resize', resizeCanvas);

        canvas.addEventListener('mousedown', startDrawing);
        canvas.addEventListener('mousemove', draw);
        canvas.addEventListener('mouseup', stopDrawing);
        canvas.addEventListener('mouseleave', stopDrawing);

        canvas.addEventListener('touchstart', handleTouch);
        canvas.addEventListener('touchmove', handleTouch);
        canvas.addEventListener('touchend', stopDrawing);
    }

    function resizeCanvas() {
        if(!canvas) return;
        const rect = canvas.parentElement.getBoundingClientRect();
        canvas.width = rect.width;
        canvas.height = rect.height;
    }

    function startDrawing(e) {
        isDrawing = true;
        hideEmptyMsg();
        const pos = getPos(e);
        lastX = pos.x;
        lastY = pos.y;
    }

    function draw(e) {
        if (!isDrawing) return;
        const pos = getPos(e);
        
        drawLineLocally(lastX, lastY, pos.x, pos.y, currentColor, currentSize, currentTool === 'eraser');

        if(conn && conn.open) {
            conn.send({
                type: 'DRAW',
                x0: lastX,
                y0: lastY,
                x1: pos.x,
                y1: pos.y,
                color: currentColor,
                size: currentTool === 'eraser' ? 24 : currentSize,
                isEraser: currentTool === 'eraser'
            });
        }

        lastX = pos.x;
        lastY = pos.y;
    }

    function drawLineLocally(x0, y0, x1, y1, color, size, isEraser) {
        ctx.beginPath();
        ctx.moveTo(x0, y0);
        ctx.lineTo(x1, y1);
        ctx.strokeStyle = isEraser ? '#FFFFFF' : color;
        ctx.lineWidth = isEraser ? 24 : size;
        ctx.lineCap = 'round';
        ctx.lineJoin = 'round';
        ctx.stroke();
    }

    function drawRemoteLine(x0, y0, x1, y1, color, size, isEraser) {
        hideEmptyMsg();
        drawLineLocally(x0, y0, x1, y1, color, size, isEraser);
    }

    function stopDrawing() {
        isDrawing = false;
    }

    function handleTouch(e) {
        e.preventDefault();
        const touch = e.touches[0];
        const mouseEvent = new MouseEvent(e.type === 'touchstart' ? 'mousedown' : e.type === 'touchmove' ? 'mousemove' : 'mouseup', {
            clientX: touch.clientX,
            clientY: touch.clientY
        });
        canvas.dispatchEvent(mouseEvent);
    }

    function getPos(e) {
        const rect = canvas.getBoundingClientRect();
        return {
            x: (e.clientX || e.touches[0].clientX) - rect.left,
            y: (e.clientY || e.touches[0].clientY) - rect.top
        };
    }

    function hideEmptyMsg() {
        if(!hasDrawn) {
            hasDrawn = true;
            const msg = document.getElementById('empty-msg');
            if(msg) {
                msg.style.opacity = '0';
                setTimeout(() => msg.style.display = 'none', 500);
            }
        }
    }

    function setTool(tool) {
        currentTool = tool;
        document.querySelectorAll('.tool-btn').forEach(b => b.classList.remove('active'));
        document.getElementById('tool-' + tool).classList.add('active');
    }

    function setColor(color, element) {
        currentColor = color;
        currentTool = 'pen';
        document.querySelectorAll('.color-dot').forEach(d => d.classList.remove('active'));
        element.classList.add('active');
        document.getElementById('tool-pen').classList.add('active');
        document.getElementById('tool-eraser').classList.remove('active');
    }

    function clearCanvas(isSender = false) {
        executeClearCanvas();
        if(isSender && conn && conn.open) {
            conn.send({ type: 'CLEAR' });
        }
    }

    function executeClearCanvas() {
        if(ctx && canvas) {
            ctx.clearRect(0, 0, canvas.width, canvas.height);
        }
        // Also clear any placed stickers on the canvas container
        document.querySelectorAll('.placed-sticker').forEach(s => s.remove());
    }

    function sendHeartEffect(isSender = false) {
        const container = document.querySelector('.canvas-container');
        const rect = container.getBoundingClientRect();
        const x = rect.width / 2;
        const y = rect.height / 2;
        
        spawnFloatingHeart(x, y);

        if(isSender && conn && conn.open) {
            conn.send({ type: 'HEART', x, y });
            addMoment('❤️', `You sent ${partnerName} a heart`);
        }
    }

    function spawnFloatingHeart(x, y) {
        const heart = document.createElement('div');
        heart.className = 'floating-heart-anim';
        heart.innerText = '❤️';
        heart.style.left = `${x - 14}px`;
        heart.style.top = `${y - 14}px`;
        document.querySelector('.canvas-container').appendChild(heart);
        setTimeout(() => heart.remove(), 1500);
    }

    function triggerThinkingOfYou() {
        if(conn && conn.open) {
            conn.send({ type: 'THINKING' });
            addMoment('💭', `You let ${partnerName} know you're thinking of them`);
            alert("💖 Sent 'Thinking of you' notification!");
        } else {
            alert("Not connected to a partner yet!");
        }
    }

    function openSettings() {
        alert("Space settings: You are connected as " + userNickname);
    }

    function addMoment(icon, text) {
        const list = document.getElementById('moments-list');
        if(!list) return;
        const card = document.createElement('div');
        card.className = 'moment-card';
        card.innerHTML = `<span class="moment-icon">${icon}</span><div>${text}</div>`;
        list.prepend(card);
    }
</script>
</body>
</html>
