# Teddy-day-
<!DOCTYPE HAPPY-TEDDY-DAY-JANN>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>Teddy Challenge</title>
    <style>
        :root { --pink: #ff4d6d; --dark: #0a0a0a; }
        * { box-sizing: border-box; touch-action: none; -webkit-tap-highlight-color: transparent; }
        body, html { margin: 0; padding: 0; background: var(--dark); color: white; font-family: sans-serif; overflow: hidden; height: 100%; width: 100%; }

        .glitch { font-size: 2rem; font-weight: bold; text-align: center; text-transform: uppercase; color: white; text-shadow: 2px 2px #ff4d6d, -2px -2px #00fffc; animation: shake 0.5s infinite; }
        @keyframes shake { 0% { transform: translate(0); } 20% { transform: translate(-2px, 2px); } 40% { transform: translate(-2px, -2px); } 60% { transform: translate(2px, 2px); } 80% { transform: translate(2px, -2px); } 100% { transform: translate(0); } }

        .screen { display: none; height: 100vh; width: 100vw; flex-direction: column; align-items: center; justify-content: center; position: absolute; padding: 20px; }
        .active { display: flex; }

        /* Game Elements */
        #game-area { width: 90%; max-width: 400px; height: 50vh; background: #111; border: 4px solid #333; position: relative; overflow: hidden; border-radius: 15px; }
        #claw-line { position: absolute; top: 0; width: 2px; background: #eee; height: 0; z-index: 5; }
        #claw { position: absolute; top: 0; width: 50px; height: 40px; background: #555; border-radius: 0 0 10px 10px; display: flex; justify-content: center; align-items: center; z-index: 10; font-weight: bold; color: white; border: 1px solid #777; transition: top 0.7s ease-in; }
        .item { position: absolute; font-size: 30px; z-index: 8; transition: top 0.7s ease-in; }

        /* Buttons */
        .controls { display: grid; grid-template-columns: 1fr 1fr; gap: 10px; width: 90%; max-width: 400px; margin-top: 20px; }
        .btn { background: #333; color: white; border: 2px solid #555; padding: 20px; font-size: 1.5rem; border-radius: 10px; text-align: center; }
        .btn-grab { grid-column: span 2; background: var(--pink); border: none; font-weight: bold; }
        .btn:active { background: #555; transform: scale(0.95); }

        #score-tag { font-size: 1.5rem; color: var(--pink); margin-bottom: 10px; font-weight: bold; }
        #timer-box { font-size: 3rem; color: var(--pink); text-align: center; }
    </style>
</head>
<body>

    <section id="intro" class="screen active"><h1 id="day-text" class="glitch">7th Feb: Rose Day</h1></section>

    <section id="challenge" class="screen">
        <h2 class="glitch">ARE YOU READY MY LADY?</h2>
        <button class="btn btn-grab" onclick="showScreen('instructions')">I AM READY</button>
    </section>

    <section id="instructions" class="screen">
        <h2 style="color:var(--pink)">MISSION RULES</h2>
        <p style="text-align: center;">Move ⬅️ ➡️ and GRAB!<br>Avoid 💣 (-500)<br>Target: 2500 Points</p>
        <button class="btn btn-grab" onclick="startGame()">START</button>
    </section>

    <section id="game-screen" class="screen">
        <div id="score-tag">SCORE: <span id="score">0</span> / 2500</div>
        <div id="game-area">
            <div id="claw-line"></div>
            <div id="claw">U</div>
        </div>
        <div class="controls">
            <div class="btn" id="leftBtn">⬅️</div>
            <div class="btn" id="rightBtn">➡️</div>
            <button class="btn btn-grab" onclick="attemptGrab()">GRAB TEDDY 🧸</button>
        </div>
    </section>

    <section id="slideshow" class="screen"><div id="quote-box" style="text-align:center"></div></section>

    <section id="final" class="screen">
        <h1 class="glitch">COMING SOON</h1>
        <div id="timer-box">00:00:00</div>
        <p>Until Promise Day</p>
    </section>

    <script>
        let score = 0, clawX = 150, isMoving = 0, grabbing = false;
        const days = ["7th Feb: Rose Day", "8th Feb: Propose Day", "9th Feb: Chocolate Day", "10th Feb: TEDDY DAY"];
        
        // Sequence logic
        let dIdx = 0;
        const introTimer = setInterval(() => {
            dIdx++;
            if(dIdx < 4) document.getElementById('day-text').innerText = days[dIdx];
            else { clearInterval(introTimer); showScreen('challenge'); }
        }, 3900);

        function showScreen(id) {
            document.querySelectorAll('.screen').forEach(s => s.classList.remove('active'));
            document.getElementById(id).classList.add('active');
        }

        function startGame() {
            showScreen('game-screen');
            spawnItems();
            requestAnimationFrame(gameLoop);
        }

        // Fixed Movement Logic
        const lBtn = document.getElementById('leftBtn');
        const rBtn = document.getElementById('rightBtn');
        lBtn.onpointerdown = () => isMoving = -5;
        rBtn.onpointerdown = () => isMoving = 5;
        lBtn.onpointerup = lBtn.onpointerleave = rBtn.onpointerup = rBtn.onpointerleave = () => isMoving = 0;

        function gameLoop() {
            if (!grabbing && isMoving !== 0) {
                const areaWidth = document.getElementById('game-area').offsetWidth;
                clawX += isMoving;
                if(clawX < 0) clawX = 0;
                if(clawX > areaWidth - 50) clawX = areaWidth - 50;
                
                document.getElementById('claw').style.left = clawX + 'px';
                document.getElementById('claw-line').style.left = (clawX + 24) + 'px';
            }
            requestAnimationFrame(gameLoop);
        }

        function spawnItems() {
            const area = document.getElementById('game-area');
            const types = [{e:'✨',p:250}, {e:'🐶',p:200}, {e:'🧸',p:100}, {e:'👩‍❤️‍👨',p:300}, {e:'💣',p:-500}];
            for(let i=0; i<12; i++) {
                let t = types[Math.floor(Math.random()*types.length)];
                let d = document.createElement('div');
                d.className = 'item';
                d.innerHTML = t.e;
                d.dataset.p = t.p;
                d.style.left = Math.random()*80 + 5 + '%';
                d.style.top = Math.random()*40 + 50 + '%';
                area.appendChild(d);
            }
        }

        function attemptGrab() {
            if(grabbing) return;
            grabbing = true;
            const claw = document.getElementById('claw');
            const line = document.getElementById('claw-line');
            const targetY = document.getElementById('game-area').offsetHeight - 80;

            claw.style.top = targetY + 'px';
            line.style.height = targetY + 'px';

            setTimeout(() => {
                const cRect = claw.getBoundingClientRect();
                document.querySelectorAll('.item').forEach(it => {
                    const iRect = it.getBoundingClientRect();
                    if(Math.abs(cRect.left - iRect.left) < 40 && Math.abs(cRect.top - iRect.top) < 40) {
                        score += parseInt(it.dataset.p);
                        document.getElementById('score').innerText = Math.max(0, score);
                        it.style.top = "-50px"; 
                        setTimeout(() => { it.remove(); if(document.querySelectorAll('.item').length < 8) spawnItems(); }, 500);
                    }
                });

                claw.style.top = "0px";
                line.style.height = "0px";
                setTimeout(() => { 
                    grabbing = false; 
                    if(score >= 2500) startFinalPhase();
                }, 700);
            }, 700);
        }

        function startFinalPhase() {
            showScreen('slideshow');
            const quotes = ["You're my favorite teddy!","Warm hugs only.","Soft heart, big love.","Janki & Aditya forever.","Happy Teddy Day!","Cuddle mode: ON.","My heart is plush for you.","Love you to the moon.","Stay cute always.","The best hugger ever."];
            let qIdx = 0;
            const qBox = document.getElementById('quote-box');
            const qInt = setInterval(() => {
                if(qIdx < quotes.length) { qBox.innerHTML = `<h2 class="glitch">${quotes[qIdx]}</h2>`; qIdx++; }
                else { 
                    clearInterval(qInt); 
                    showScreen('final'); 
                    updateCountdown();
                }
            }, 3000);
        }

        function updateCountdown() {
            const target = new Date("Feb 11, 2026 00:00:00").getTime();
            setInterval(() => {
                const now = new Date().getTime();
                const diff = target - now;
                const h = Math.floor(diff / 36e5), m = Math.floor((diff % 36e5)/6e4), s = Math.floor((diff % 6e4)/1000);
                document.getElementById('timer-box').innerText = `${h}h ${m}m ${s}s`;
            }, 1000);
        }
    </script>
</body>
</html>

        #countdown { font-size: 3.2rem; color: var(--pink); font-weight: bold; }
    </style>
</head>
<body>

    <section id="intro" class="screen active"><h1 id="day-text" class="glitch">7th Feb: Rose Day</h1></section>

    <section id="challenge" class="screen">
        <h2 class="glitch">ARE YOU READY MY LADY?</h2>
        <button onclick="showScreen('instructions')">YES, I'M READY</button>
    </section>

    <section id="instructions" class="screen">
        <h3 style="color:var(--pink)">MISSION RULES</h3>
        <p>Use ⬅️ ➡️ to move claw.<br>Press <b>GRAB</b> to drop.<br>Avoid 💣 (-500 pts).<br>Target: <b>2500 pts</b>.</p>
        <button onclick="startGame()">START MISSION</button>
    </section>

    <section id="game-screen" class="screen">
        <div id="score-board">SCORE: <span id="score">0</span> / 2500</div>
        <div id="game-canvas">
            <div id="claw-line" class="claw-line"></div>
            <div id="claw" class="claw">U</div>
            <div class="basket">🧺</div>
        </div>
        
        <div class="controls">
            <div class="btn-ctrl" onpointerdown="startMove(-1)" onpointerup="stopMove()">⬅️</div>
            <div class="btn-ctrl" onpointerdown="startMove(1)" onpointerup="stopMove()">➡️</div>
            <div class="btn-ctrl btn-grab" onpointerdown="dropClaw()">GRAB TEDDY 🧸</div>
        </div>
    </section>

    <section id="slideshow" class="screen"><div id="quote-container"></div></section>

    <section id="final" class="screen">
        <h2 class="glitch">COMING SOON</h2>
        <div id="countdown">00:00:00</div>
        <p>UNTIL PROMISE DAY</p>
    </section>

    <script>
        const days = ["7th Feb: Rose Day", "8th Feb: Propose Day", "9th Feb: Chocolate Day", "10th Feb: TEDDY DAY"];
        let dayIdx = 0, score = 0, isDropping = false, clawPos = 150, moveDir = 0;

        const introInterval = setInterval(() => {
            dayIdx++;
            if (dayIdx < days.length) document.getElementById('day-text').innerText = days[dayIdx];
            else { clearInterval(introInterval); showScreen('challenge'); }
        }, 3900);

        function showScreen(id) {
            document.querySelectorAll('.screen').forEach(s => s.classList.remove('active'));
            document.getElementById(id).classList.add('active');
        }

        const items = [{ n: '✨', p: 250 }, { n: '🐶', p: 200 }, { n: '🧸', p: 100 }, { n: '👩‍❤️‍👨', p: 300 }, { n: '💣', p: -500 }];

        function startGame() {
            showScreen('game-screen');
            spawnItems(10);
            updateClaw();
            setInterval(processMovement, 30);
        }

        function spawnItems(count) {
            const canvas = document.getElementById('game-canvas');
            for(let i=0; i<count; i++) {
                let item = items[Math.floor(Math.random() * items.length)];
                let div = document.createElement('div');
                div.className = 'teddy-obj';
                div.innerHTML = item.n;
                div.dataset.pts = item.p;
                div.style.left = (Math.random() * 80 + 5) + '%';
                div.style.top = (Math.random() * 30 + 60) + '%';
                canvas.appendChild(div);
            }
        }

        // Gamepad logic
        function startMove(dir) { moveDir = dir; }
        function stopMove() { moveDir = 0; }
        
        function processMovement() {
            if (isDropping || moveDir === 0) return;
            const limit = document.getElementById('game-canvas').offsetWidth - 55;
            clawPos += moveDir * 6;
            if (clawPos < 0) clawPos = 0;
            if (clawPos > limit) clawPos = limit;
            updateClaw();
        }

        function updateClaw() {
            const claw = document.getElementById('claw');
            const line = document.getElementById('claw-line');
            claw.style.left = clawPos + 'px';
            line.style.left = (clawPos + 24) + 'px';
        }

        function dropClaw() {
            if (isDropping) return;
            isDropping = true;
            const claw = document.getElementById('claw');
            const line = document.getElementById('claw-line');
            const target = document.getElementById('game-canvas').offsetHeight - 85;
            
            claw.style.top = target + 'px';
            line.style.height = target + 'px';

            setTimeout(() => {
                checkCatch(target);
                claw.style.top = '0px';
                line.style.height = '0px';
                setTimeout(() => { 
                    isDropping = false; 
                    if(score >= 2500) startSlideshow();
                }, 800);
            }, 800);
        }

        function checkCatch(yPos) {
            const clawRect = document.getElementById('claw').getBoundingClientRect();
            document.querySelectorAll('.teddy-obj').forEach(obj => {
                const objRect = obj.getBoundingClientRect();
                const dist = Math.sqrt(Math.pow(clawRect.left - objRect.left, 2) + Math.pow(clawRect.top - objRect.top, 2));
                if (dist < 50) {
                    score += parseInt(obj.dataset.pts);
                    document.getElementById('score').innerText = Math.max(0, score);
                    obj.style.top = '-50px'; // Move up with claw
                    setTimeout(() => { obj.remove(); spawnItems(1); }, 500);
                }
            });
        }

        const quotes = [
            "You're the softest part of my world. Happy Teddy Day!",
            "Giving you a teddy so you have something to hug when I'm not there.",
            "To the girl who is as cute as a plushie—I love you!",
            "Janki & Aditya: A bond softer than any teddy bear. 🧸",
            "May your life be filled with fluffy moments and warm hugs.",
            "Sending you a giant hug in the form of this digital teddy!",
            "Every time you hug your teddy, remember I'm hugging you too.",
            "You don't need a teddy because you're already my favorite cuddle partner.",
            "Happy Teddy Day to the one who makes my heart go 'Awww'.",
            "This teddy is a witness to how much I care for you."
        ];

        function startSlideshow() {
            showScreen('slideshow');
            let i = 0;
            const q = document.getElementById('quote-container');
            const interval = setInterval(() => {
                if(i < quotes.length) { q.innerHTML = `<h2 class="glitch">${quotes[i]}</h2>`; i++; }
                else { clearInterval(interval); document.body.style.background = "white"; setTimeout(() => { document.body.style.background = "black"; showScreen('final'); runCountdown(); }, 150); }
            }, 3000);
        }

        function runCountdown() {
            const target = new Date("Feb 11, 2026 00:00:00").getTime();
            setInterval(() => {
                const diff = target - new Date().getTime();
                const h = Math.floor(diff / 36e5), m = Math.floor((diff % 36e5) / 6e4), s = Math.floor((diff % 6e4) / 1000);
                document.getElementById('countdown').innerText = `${h}h ${m}m ${s}s`;
            }, 1000);
        }
    </script>
</body>
</html>

        .instr-box table { width: 100%; font-size: 0.9rem; border-collapse: collapse; }
        .instr-box td { padding: 5px; border-bottom: 1px solid #333; }

        /* Game UI */
        #game-canvas { background: #1a1a1a; border: 3px solid #333; position: relative; width: 95%; max-width: 400px; height: 55vh; border-radius: 10px; overflow: hidden; margin-top: 10px; }
        .claw-line { position: absolute; top: 0; width: 2px; background: #fff; height: 0; transition: height 1s; }
        .claw { width: 50px; height: 40px; background: #444; position: absolute; top: 0; display: flex; justify-content: center; align-items: center; font-size: 20px; transition: top 1s, left 0.1s; z-index: 10; border-radius: 0 0 10px 10px; border: 1px solid #666; }
        .teddy-obj { position: absolute; font-size: 32px; user-select: none; }
        .basket { position: absolute; bottom: 0; left: 0; width: 60px; height: 60px; background: #4a2c2c; border-radius: 0 10px 0 0; display: flex; align-items: center; justify-content: center; font-size: 25px; }
        
        #score-board { font-size: 1.5rem; margin-bottom: 5px; color: var(--pink); font-weight: bold; }
        #countdown { font-size: 3.5rem; color: var(--pink); text-shadow: 0 0 20px var(--pink); }
    </style>
</head>
<body>

    <section id="intro" class="screen active">
        <h1 id="day-text" class="glitch">7th Feb: Rose Day</h1>
    </section>

    <section id="challenge" class="screen">
        <h2 class="glitch">ARE YOU READY MY LADY?</h2>
        <p>A mission awaits you...</p>
        <button onclick="showInstructions()">YES, I'M READY</button>
    </section>

    <section id="instructions" class="screen">
        <h2 style="color:var(--pink)">MISSION: CLAW TEDDY</h2>
        <div class="instr-box">
            <p style="font-size: 0.8rem; color: #aaa;">HOW TO PLAY:</p>
            <p>1. <b>Slide</b> to move the claw.<br>2. <b>Tap</b> to drop it.<br>3. Reach <b>2500 pts</b> to win.</p>
            <table>
                <tr><td>✨ Golden</td><td>250 pts</td></tr>
                <tr><td>👩‍❤️‍👨 Bubu Dudu</td><td>300 pts</td></tr>
                <tr><td>🐶 Pet / 🧸 Plush</td><td>200/100 pts</td></tr>
                <tr style="color: #ff4444;"><td>💣 BOMB</td><td>-500 pts</td></tr>
            </table>
        </div>
        <button onclick="startGame()">START GAME</button>
    </section>

    <section id="game-screen" class="screen">
        <div id="score-board">SCORE: <span id="score">0</span> / 2500</div>
        <div id="game-canvas">
            <div id="claw-line" class="claw-line"></div>
            <div id="claw" class="claw">▼</div>
            <div class="basket">🧺</div>
        </div>
    </section>

    <section id="slideshow" class="screen">
        <div id="quote-container" style="text-align: center; padding: 20px;"></div>
    </section>

    <section id="final" class="screen">
        <h2 class="glitch">COMING SOON</h2>
        <div id="countdown">00:00:00</div>
        <p>UNTIL PROMISE DAY BEGINS</p>
    </section>

    <script>
        const days = ["7th Feb: Rose Day", "8th Feb: Propose Day", "9th Feb: Chocolate Day", "10th Feb: TEDDY DAY"];
        let dayIdx = 0;

        // Intro Logic
        const introInterval = setInterval(() => {
            dayIdx++;
            if (dayIdx < days.length) {
                document.getElementById('day-text').innerText = days[dayIdx];
            } else {
                clearInterval(introInterval);
                showScreen('challenge');
            }
        }, 3900);

        function showScreen(id) {
            document.querySelectorAll('.screen').forEach(s => s.classList.remove('active'));
            document.getElementById(id).classList.add('active');
        }

        function showInstructions() {
            showScreen('instructions');
        }

        // Game Logic
        let score = 0;
        let isDropping = false;
        const canvas = document.getElementById('game-canvas');
        const claw = document.getElementById('claw');
        const clawLine = document.getElementById('claw-line');
        
        const items = [
            { n: '✨', p: 250 }, { n: '🐶', p: 200 }, { n: '🧸', p: 100 },
            { n: '🧸', p: 100 }, { n: '👩‍❤️‍👨', p: 300 }, { n: '💣', p: -500 }
        ];

        function startGame() {
            showScreen('game-screen');
            spawnItems(8);
            
            canvas.addEventListener('touchmove', (e) => {
                e.preventDefault();
                moveHandler(e.touches[0]);
            }, {passive: false});
            canvas.addEventListener('mousemove', moveHandler);

            canvas.addEventListener('touchstart', (e) => {
                e.preventDefault();
                dropClaw();
            }, {passive: false});
            canvas.addEventListener('mousedown', dropClaw);
        }

        function spawnItems(count) {
            for(let i=0; i<count; i++) {
                let item = items[Math.floor(Math.random() * items.length)];
                let div = document.createElement('div');
                div.className = 'teddy-obj';
                div.innerHTML = item.n;
                div.dataset.pts = item.p;
                div.style.left = (Math.random() * 80 + 5) + '%';
                div.style.top = (Math.random() * 40 + 40) + '%';
                canvas.appendChild(div);
            }
        }

        function moveHandler(e) {
            if (isDropping) return;
            let rect = canvas.getBoundingClientRect();
            let x = e.clientX - rect.left;
            let pos = Math.max(0, Math.min(x - 25, rect.width - 50));
            claw.style.left = pos + 'px';
            clawLine.style.left = (pos + 24) + 'px';
        }

        function dropClaw() {
            if (isDropping) return;
            isDropping = true;
            let targetTop = canvas.offsetHeight - 80;
            claw.style.top = targetTop + 'px';
            clawLine.style.height = targetTop + 'px';

            setTimeout(() => {
                checkCatch();
                claw.style.top = '0px';
                clawLine.style.height = '0px';
                setTimeout(() => { 
                    isDropping = false; 
                    if(score >= 2500) startSlideshow();
                }, 1000);
            }, 1000);
        }

        function checkCatch() {
            let clawRect = claw.getBoundingClientRect();
            let objs = document.querySelectorAll('.teddy-obj');
            objs.forEach(obj => {
                let objRect = obj.getBoundingClientRect();
                if (Math.abs(clawRect.left - objRect.left) < 35 && Math.abs(clawRect.top - objRect.top) < 50) {
                    score += parseInt(obj.dataset.pts);
                    if (score < 0) score = 0;
                    document.getElementById('score').innerText = score;
                    obj.remove();
                    spawnItems(1); 
                }
            });
        }

        const quotes = [
            "You're the softest part of my world. Happy Teddy Day!",
            "Giving you a teddy so you have something to hug when I'm not there.",
            "To the girl who is as cute as a plushie—I love you!",
            "Janki & Aditya: A bond softer than any teddy bear. 🧸",
            "May your life be filled with fluffy moments and warm hugs.",
            "Sending you a giant hug in the form of this digital teddy!",
            "Every time you hug your teddy, remember I'm hugging you too.",
            "You don't need a teddy because you're already my favorite cuddle partner.",
            "Happy Teddy Day to the one who makes my heart go 'Awww'.",
            "This teddy is a witness to how much I care for you."
        ];

        function startSlideshow() {
            showScreen('slideshow');
            let i = 0;
            const interval = setInterval(() => {
                if(i < quotes.length) {
                    document.getElementById('quote-container').innerHTML = `<h2 class="glitch">${quotes[i]}</h2>`;
                    i++;
                } else {
                    clearInterval(interval);
                    document.body.style.background = "white";
                    setTimeout(() => {
                        document.body.style.background = "var(--dark)";
                        showScreen('final');
                        runCountdown();
                    }, 150);
                }
            }, 3000);
        }

        function runCountdown() {
            const target = new Date("Feb 11, 2026 00:00:00").getTime();
            setInterval(() => {
                const diff = target - new Date().getTime();
                const h = Math.floor(diff / (1000 * 60 * 60));
                const m = Math.floor((diff % (1000 * 60 * 60)) / (1000 * 60));
                const s = Math.floor((diff % (1000 * 60)) / 1000);
                document.getElementById('countdown').innerText = `${h}h ${m}m ${s}s`;
            }, 1000);
        }
    </script>
</body>
</html>
