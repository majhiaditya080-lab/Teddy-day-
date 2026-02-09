# Teddy-day-
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>Teddy Quest</title>
    <style>
        :root { --pink: #ff4d6d; --dark: #0a0a0a; --gold: #ffd700; }
        * { box-sizing: border-box; touch-action: manipulation; }
        body, html { margin: 0; padding: 0; background: var(--dark); color: white; font-family: 'Courier New', Courier, monospace; overflow: hidden; height: 100%; }

        /* Glitch Effect */
        .glitch { font-size: 2.5rem; font-weight: bold; text-align: center; text-transform: uppercase; position: relative; text-shadow: 0.05em 0 0 #00fffc, -0.03em -0.04em 0 #fc00ff; animation: glitch 725ms infinite; }
        @keyframes glitch {
            0% { text-shadow: 0.05em 0 0 #00fffc, -0.03em -0.04em 0 #fc00ff; }
            50% { text-shadow: 0.025em 0.05em 0 #00fffc, 0.05em 0 0 #fc00ff; }
            100% { text-shadow: -0.025em 0 0 #00fffc, -0.025em -0.025em 0 #fc00ff; }
        }

        .screen { display: none; height: 100vh; width: 100vw; flex-direction: column; align-items: center; justify-content: center; padding: 20px; position: absolute; top: 0; left: 0; }
        .active { display: flex; }

        button { background: var(--pink); border: none; padding: 18px 35px; color: white; font-size: 1.2rem; font-weight: bold; cursor: pointer; border-radius: 10px; box-shadow: 0 0 15px var(--pink); margin-top: 20px; }

        /* Instructions Table */
        .instr-box { background: rgba(255,255,255,0.1); padding: 20px; border-radius: 15px; border: 1px solid var(--pink); max-width: 320px; margin-bottom: 20px; }
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
