<!DOCTYPE html>
<html>
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>Ядерка по РКН</title>
    <script src="https://telegram.org/js/telegram-web-app.js"></script>
    <style>
        * { margin: 0; padding: 0; box-sizing: border-box; }
        body {
            background: #0a0a0a;
            font-family: Arial;
            touch-action: manipulation;
            user-select: none;
            -webkit-user-select: none;
            overflow: hidden;
        }
        #game {
            width: 100vw;
            height: 100vh;
            display: flex;
            flex-direction: column;
        }
        #header {
            background: rgba(255,0,0,0.3);
            padding: 10px;
            display: flex;
            justify-content: space-around;
            color: white;
            font-size: 16px;
            font-weight: bold;
            border-bottom: 2px solid red;
        }
        #canvas {
            flex: 1;
            background: #111;
            border: 2px solid #333;
            margin: 5px;
            border-radius: 8px;
        }
        #controls {
            display: grid;
            grid-template-columns: 1fr 1fr;
            gap: 8px;
            padding: 8px;
        }
        .player-controls {
            background: rgba(255,255,255,0.05);
            border-radius: 12px;
            padding: 8px;
            border: 1px solid rgba(255,255,255,0.1);
        }
        .player-name {
            text-align: center;
            font-size: 14px;
            margin-bottom: 5px;
            font-weight: bold;
        }
        .dpad {
            display: grid;
            grid-template-columns: 55px 55px 55px;
            grid-template-rows: 55px 55px 55px;
            gap: 4px;
            justify-content: center;
        }
        .btn {
            background: rgba(255,255,255,0.1);
            border: 2px solid rgba(255,255,255,0.3);
            color: white;
            font-size: 22px;
            border-radius: 12px;
            cursor: pointer;
            display: flex;
            align-items: center;
            justify-content: center;
            transition: 0.1s;
            -webkit-tap-highlight-color: transparent;
        }
        .btn:active {
            background: rgba(255,255,255,0.4);
            transform: scale(0.9);
        }
        .up { grid-column: 2; }
        .left { grid-column: 1; grid-row: 2; }
        .center { grid-column: 2; grid-row: 2; font-size: 20px; background: rgba(255,50,50,0.3); }
        .right { grid-column: 3; grid-row: 2; }
        .down { grid-column: 2; grid-row: 3; }
        #chat {
            height: 100px;
            background: rgba(0,0,0,0.5);
            margin: 5px 10px;
            border-radius: 8px;
            padding: 8px;
            overflow-y: auto;
            color: white;
            font-size: 12px;
        }
    </style>
</head>
<body>
    <div id="game">
        <div id="header">
            <div id="score1">🟢 ЯДЕРКА: 0</div>
            <div id="score2">🔴 РКН: 0</div>
        </div>
        <canvas id="canvas"></canvas>
        <div id="controls">
            <div class="player-controls">
                <div class="player-name" style="color:#0f0;">🟢 ИГРОК 1</div>
                <div class="dpad">
                    <div></div>
                    <button class="btn up" onmousedown="move1(0,-1)" ontouchstart="move1(0,-1)">⬆️</button>
                    <div></div>
                    <button class="btn left" onmousedown="move1(-1,0)" ontouchstart="move1(-1,0)">⬅️</button>
                    <button class="btn center" onclick="attack1()" ontouchstart="attack1()">☢️</button>
                    <button class="btn right" onmousedown="move1(1,0)" ontouchstart="move1(1,0)">➡️</button>
                    <div></div>
                    <button class="btn down" onmousedown="move1(0,1)" ontouchstart="move1(0,1)">⬇️</button>
                    <div></div>
                </div>
            </div>
            <div class="player-controls">
                <div class="player-name" style="color:#f00;">🔴 ИГРОК 2</div>
                <div class="dpad">
                    <div></div>
                    <button class="btn up" onmousedown="move2(0,-1)" ontouchstart="move2(0,-1)">⬆️</button>
                    <div></div>
                    <button class="btn left" onmousedown="move2(-1,0)" ontouchstart="move2(-1,0)">⬅️</button>
                    <button class="btn center" onclick="attack2()" ontouchstart="attack2()">☢️</button>
                    <button class="btn right" onmousedown="move2(1,0)" ontouchstart="move2(1,0)">➡️</button>
                    <div></div>
                    <button class="btn down" onmousedown="move2(0,1)" ontouchstart="move2(0,1)">⬇️</button>
                    <div></div>
                </div>
            </div>
        </div>
        <div id="chat"></div>
    </div>
    <script>
        const canvas = document.getElementById('canvas');
        const ctx = canvas.getContext('2d');

        function resize() {
            canvas.width = canvas.offsetWidth;
            canvas.height = canvas.offsetHeight;
        }
        resize();
        window.addEventListener('resize', resize);

        let p1 = { x: 80, y: 200, hp: 100, color: '#0f0' };
        let p2 = { x: 300, y: 200, hp: 100, color: '#f00' };
        let s1 = 0, s2 = 0;
        let coins = [];

        for(let i = 0; i < 3; i++) {
            spawnCoin();
        }

        function spawnCoin() {
            coins.push({
                x: Math.random() * (canvas.width - 60) + 30,
                y: Math.random() * (canvas.height - 60) + 30
            });
        }

        function move1(dx, dy) {
            p1.x += dx * 25;
            p1.y += dy * 25;
            p1.x = Math.max(25, Math.min(canvas.width - 25, p1.x));
            p1.y = Math.max(25, Math.min(canvas.height - 25, p1.y));
            checkCoins(1);
        }

        function move2(dx, dy) {
            p2.x += dx * 25;
            p2.y += dy * 25;
            p2.x = Math.max(25, Math.min(canvas.width - 25, p2.x));
