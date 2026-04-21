<!DOCTYPE html>
<html lang="ms">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>JOM CANTUM BUNYI</title>
    <script src="https://cdn.jsdelivr.net/npm/canvas-confetti@1.5.1/dist/confetti.browser.min.js"></script>
    <style>
        :root {
            --primary: #FF6B6B;
            --secondary: #4ECDC4;
            --accent: #FFE66D;
            --bg: #F7FFF7;
        }
        body { font-family: 'Comic Sans MS', cursive; background: var(--bg); margin: 0; text-align: center; overflow-x: hidden; }
        .screen { display: none; padding: 20px; animation: fadeIn 0.5s; }
        .active { display: block; }
        .btn { background: var(--secondary); border: none; padding: 15px 30px; border-radius: 50px; color: white; font-size: 1.2rem; cursor: pointer; box-shadow: 0 4px #45b7af; margin: 10px; }
        .btn:active { transform: translateY(4px); box-shadow: 0 0px; }
        .grid { display: grid; grid-template-columns: repeat(2, 1fr); gap: 10px; padding: 20px; }
        .drop-zone { border: 3px dashed #ccc; min-height: 50px; margin: 10px; background: white; border-radius: 10px; }
        .word-piece { background: var(--accent); padding: 10px; border-radius: 10px; cursor: move; display: inline-block; margin: 5px; }
        canvas { background: black; display: block; margin: 0 auto; border-radius: 20px; }
    </style>
</head>
<body>

    <div id="login-screen" class="screen active">
        <h1 style="color: var(--primary); font-size: 3rem;">JOM CANTUM BUNYI 🎨</h1>
        <div style="background: white; padding: 30px; border-radius: 20px; display: inline-block;">
            <input type="text" id="stuName" placeholder="Nama Anda" style="padding:10px; width:80%; margin-bottom:10px;"><br>
            <input type="text" id="stuClass" placeholder="Kelas" style="padding:10px; width:80%;"><br><br>
            <button class="btn" onclick="startGame()">MULA MAIN!</button>
        </div>
    </div>

    <div id="main-menu" class="screen">
        <h2 id="welcome-msg">Hi!</h2>
        <div class="grid">
            <button class="btn" onclick="loadGame(1)">GABUNG SUKU KATA</button>
            <button class="btn" onclick="loadGame(2)">PADANKAN KVKV</button>
            <button class="btn" onclick="loadGame(3)">PACMAN APAKAH ITU?</button>
            <button class="btn" onclick="loadGame(4)">BINA AYAT KVKV</button>
        </div>
        <button class="btn" style="background:#ff9f43" onclick="changeUser()">TUKAR USER</button>
        <p>Skor Terkumpul: <span id="total-score">0</span></p>
    </div>

    <div id="game1-screen" class="screen">
        <button class="btn" onclick="goHome()">🏠 Home</button>
        <h3>Susun Suku Kata</h3>
        <div id="game1-content">
            <img id="g1-img" src="" width="150">
            <button onclick="playAudio()">🔊 Dengar</button>
            <div style="display:flex; justify-content: center;">
                <div id="box1" class="drop-zone" ondrop="drop(event)" ondragover="allowDrop(event)"></div>
                <div id="box2" class="drop-zone" ondrop="drop(event)" ondragover="allowDrop(event)"></div>
            </div>
            <div id="syllables-pool"></div>
        </div>
    </div>

    <div id="game4-screen" class="screen">
        <button class="btn" onclick="goHome()">🏠 Home</button>
        <div id="game4-content">
            <img id="g4-img" src="" width="200">
            <div id="sentence-slots" style="margin:20px;"></div>
            <div id="word-pool"></div>
        </div>
    </div>

    <script>
        let userData = { name: "", class: "", score: 0 };
        const wordBank = ["baca", "baju", "bola", "gigi", "ceri", "dagu", "kaki"]; // Contoh sebahagian
        
        function startGame() {
            userData.name = document.getElementById('stuName').value;
            userData.class = document.getElementById('stuClass').value;
            if(userData.name == "") return alert("Sila masukkan nama!");
            
            document.getElementById('welcome-msg').innerText = `Jom Belajar, ${userData.name}!`;
            showScreen('main-menu');
            syncToGoogle();
        }

        function showScreen(id) {
            document.querySelectorAll('.screen').forEach(s => s.classList.remove('active'));
            document.getElementById(id).classList.add('active');
        }

        function goHome() { showScreen('main-menu'); }
        function changeUser() { location.reload(); }

        // LOGIK GAME 1: GABUNG SUKU KATA
        function loadGame(num) {
            if(num === 1) {
                const target = "baju"; // Contoh rawak
                document.getElementById('g1-img').src = `https://img.icons8.com/color/150/shirt.png`; // Placeholder
                const pool = document.getElementById('syllables-pool');
                pool.innerHTML = `
                    <div class="word-piece" draggable="true" ondragstart="drag(event)" id="s1">ba</div>
                    <div class="word-piece" draggable="true" ondragstart="drag(event)" id="s2">ju</div>
                `;
                showScreen('game1-screen');
            }
            if(num === 4) {
                loadGame4(0);
                showScreen('game4-screen');
            }
        }

        // FUNGSI SERET & LETAK (DRAG & DROP)
        function allowDrop(ev) { ev.preventDefault(); }
        function drag(ev) { ev.dataTransfer.setData("text", ev.target.id); }
        function drop(ev) {
            ev.preventDefault();
            var data = ev.dataTransfer.getData("text");
            ev.target.appendChild(document.getElementById(data));
            checkAnswerG1();
        }

        function checkAnswerG1() {
            const b1 = document.getElementById('box1').innerText;
            const b2 = document.getElementById('box2').innerText;
            if(b1 + b2 === "baju") {
                celebrate();
            }
        }

        function celebrate() {
            confetti({ particleCount: 100, spread: 70, origin: { y: 0.6 } });
            userData.score += 10;
            document.getElementById('total-score').innerText = userData.score;
            alert("✅ BETUL! Tahniah!");
            syncToGoogle();
        }

        function syncToGoogle() {
            // Gantian URL Web App Google Script anda
            const scriptURL = 'URL_GOOGLE_SCRIPT_ANDA';
            fetch(scriptURL, {
                method: 'POST',
                mode: 'no-cors',
                body: JSON.stringify(userData)
            });
        }

        // LOGIK GAME 4: BINA AYAT
        const g4Data = [
            { img: "book", words: ["Rina", "baca", "buku"], correct: "Rina baca buku" },
            { img: "shirt", words: ["Baju", "bapa", "baru"], correct: "Baju bapa baru" }
        ];

        function loadGame4(idx) {
            const data = g4Data[idx];
            document.getElementById('g4-img').src = `https://img.icons8.com/color/200/${data.img}.png`;
            const pool = document.getElementById('word-pool');
            pool.innerHTML = '';
            data.words.sort(() => Math.random() - 0.5).forEach(w => {
                pool.innerHTML += `<button class="btn" onclick="addWord('${w}', '${data.correct}')">${w}</button>`;
            });
            document.getElementById('sentence-slots').innerText = "";
        }

        let currentSentence = "";
        function addWord(w, correct) {
            currentSentence += w + " ";
            document.getElementById('sentence-slots').innerText = currentSentence;
            if(currentSentence.trim() === correct) {
                celebrate();
            }
        }
    </script>
</body>
</html>
