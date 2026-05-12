<!DOCTYPE html>
<html>
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>Ядерка по РКН</title>
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
        .center { grid-column: 2; grid-row: 2; font-size: 18px; background: rgba(255,50,50,0.3); }
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
                    <button class="btn center" onmousedown="attack1()" ontouchstart="attack1()">☢️</button>
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
                    <button class="btn center" onmousedown="attack2()" ontouchstart="attack2()">☢️</button>
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

        let p1 = { x: 80, y: 200, hp: 100, color: '#0f0' };
        let p2 = { x: 300, y: 200, hp: 100, color: '#f00' };
        let s1 = 0, s2 = 0;
        let coins = [];

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
            p2.y = Math.max(25, Math.min(canvas.height - 25, p2.y));
            checkCoins(2);
        }

        function attack1() {
            let d = Math.hypot(p1.x - p2.x, p1.y - p2.y);
            if(d < 100) {
                p2.hp -= 20;
                addChat('☢️ ЯДЕРКА бьёт по РКН! -20');
                if(p2.hp <= 0) { s1++; reset(); addChat('💥 РКН УНИЧТОЖЕН!'); updateScore(); }
            }
        }

        function attack2() {
            let d = Math.hypot(p1.x - p2.x, p1.y - p2.y);
            if(d < 100) {
                p1.hp -= 20;
                addChat('🛡️ РКН блокирует! -20');
                if(p1.hp <= 0) { s2++; reset(); addChat('😡 РКН ЗАБЛОКИРОВАЛ!'); updateScore(); }
            }
        }

        function checkCoins(n) {
            let p = n === 1 ? p1 : p2;
            for(let i = coins.length - 1; i >= 0; i--) {
                let d = Math.hypot(p.x - coins[i].x, p.y - coins[i].y);
                if(d < 30) {
                    coins.splice(i, 1);
                    if(n === 1) s1 += 10;
                    else s2 += 10;
                    updateScore();
                    addChat('⭐ Монетка собрана!');
                    spawnCoin();
                }
            }
        }

        function reset() {
            p1.x = 80; p1.y = canvas.height/2; p1.hp = 100;
            p2.x = canvas.width - 80; p2.y = canvas.height/2; p2.hp = 100;
            coins = [];
            for(let i = 0; i < 3; i++) spawnCoin();
        }

        function updateScore() {
            document.getElementById('score1').textContent = '🟢 ЯДЕРКА: ' + s1;
            document.getElementById('score2').textContent = '🔴 РКН: ' + s2;
        }

        function addChat(msg) {
            let c = document.getElementById('chat');
            c.innerHTML += '<div>' + msg + '</div>';
            c.scrollTop = c.scrollHeight;
        }

        function draw() {
            ctx.clearRect(0, 0, canvas.width, canvas.height);
            ctx.fillStyle = '#111';
            ctx.fillRect(0, 0, canvas.width, canvas.height);

            ctx.strokeStyle = 'rgba(255,255,255,0.05)';
            for(let i = 0; i < canvas.width; i += 40) {
                ctx.beginPath(); ctx.moveTo(i, 0); ctx.lineTo(i, canvas.height); ctx.stroke();
            }
            for(let i = 0; i < canvas.height; i += 40) {
                ctx.beginPath(); ctx.moveTo(0, i); ctx.lineTo(canvas.width, i); ctx.stroke();
            }

            coins.forEach(c => {
                ctx.fillStyle = '#ffd700';
                ctx.beginPath(); ctx.arc(c.x, c.y, 12, 0, Math.PI*2); ctx.fill();
            });

            ctx.fillStyle = p1.color;
            ctx.fillRect(p1.x-20, p1.y-25, 40, 50);
            ctx.fillStyle = 'red';
            ctx.fillRect(p1.x-25, p1.y-35, 50, 5);
            ctx.fillStyle = 'green';
            ctx.fillRect(p1.x-25, p1.y-35, p1.hp*0.5, 5);

            ctx.fillStyle = p2.color;
            ctx.fillRect(p2.x-20, p2.y-25, 40, 50);
            ctx.fillStyle = 'red';
            ctx.fillRect(p2.x-25, p2.y-35, 50, 5);
            ctx.fillStyle = 'green';
            ctx.fillRect(p2.x-25, p2.y-35, p2.hp*0.5, 5);

            let d = Math.hypot(p1.x-p2.x, p1.y-p2.y);
            if(d < 100) {
                ctx.strokeStyle = 'yellow';
                ctx.lineWidth = 2;
                ctx.beginPath(); ctx.moveTo(p1.x, p1.y); ctx.lineTo(p2.x, p2.y); ctx.stroke();
            }

            requestAnimationFrame(draw);
        }

        setTimeout(function() {
            resize();
            reset();
            draw();
            addChat('☢️ Бой готов! Ядерка vs РКН');
        }, 500);
    </script>
</body>
</html>
