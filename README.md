<!DOCTYPE html>
<html lang="es">
<head>
  <meta charset="UTF-8" />
  <title>¿Quién hizo más historia en el fútbol?</title>
  <style>
    body {
      font-family: Arial, sans-serif;
      background: #f2f2f2;
      color: #222;
      text-align: center;
      padding: 40px;
      transition: background 0.3s, color 0.3s;
      position: relative;
      min-height: 100vh;
    }

    body.dark {
      background: #121212;
      color: #f0f0f0;
    }

    h1 {
      color: inherit;
    }

    .container {
      display: flex;
      justify-content: center;
      gap: 30px;
      flex-wrap: wrap;
    }

    .player {
      background: white;
      border-radius: 10px;
      padding: 20px;
      width: 180px;
      box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
      cursor: pointer;
      transition: transform 0.3s, box-shadow 0.3s, border-color 0.3s;
      border: 4px solid transparent;
      user-select: none;
      display: flex;
      align-items: center;
      justify-content: center;
      font-weight: bold;
      font-size: 20px;
    }

    body.dark .player {
      background: #1e1e1e;
      box-shadow: 0 4px 8px rgba(255, 255, 255, 0.1);
    }

    .player:hover {
      transform: scale(1.05);
      box-shadow: 0 6px 12px rgba(0, 0, 0, 0.15);
    }

    #result {
      margin-top: 30px;
      font-size: 22px;
      color: inherit;
    }

    #score,
    #round,
    #timer {
      font-size: 20px;
      margin-top: 10px;
    }

    button {
      margin: 10px 5px;
      padding: 10px 20px;
      font-size: 16px;
      border: none;
      border-radius: 8px;
      cursor: pointer;
      transition: background 0.3s, color 0.3s;
    }

    #resetBtn {
      background: #444;
      color: white;
    }
    #resetBtn:hover {
      background: #333;
    }

    #muteBtn,
    #themeBtn,
    #skipBtn {
      background: #777;
      color: white;
    }

    #muteBtn:hover,
    #themeBtn:hover,
    #skipBtn:hover {
      background: #555;
    }

    /* Animaciones para acierto/error/empate */
    .correct {
      animation: borderGlowGreen 1.2s forwards;
    }

    .wrong {
      animation: borderGlowRed 1.2s forwards;
    }

    .tie {
      animation: borderGlowBlue 1.2s forwards;
    }

    @keyframes borderGlowGreen {
      0% {
        border-color: transparent;
        box-shadow: 0 0 0 rgba(0, 255, 0, 0);
      }
      50% {
        border-color: #28a745;
        box-shadow: 0 0 15px #28a745;
      }
      100% {
        border-color: transparent;
        box-shadow: 0 0 0 rgba(0, 255, 0, 0);
      }
    }

    @keyframes borderGlowRed {
      0% {
        border-color: transparent;
        box-shadow: 0 0 0 rgba(255, 0, 0, 0);
      }
      50% {
        border-color: #dc3545;
        box-shadow: 0 0 15px #dc3545;
      }
      100% {
        border-color: transparent;
        box-shadow: 0 0 0 rgba(255, 0, 0, 0);
      }
    }

    @keyframes borderGlowBlue {
      0% {
        border-color: transparent;
        box-shadow: 0 0 0 rgba(0, 123, 255, 0);
      }
      50% {
        border-color: #007bff;
        box-shadow: 0 0 15px #007bff;
      }
      100% {
        border-color: transparent;
        box-shadow: 0 0 0 rgba(0, 123, 255, 0);
      }
    }
  </style>
</head>
<body>
  <h1>¿Qué jugador hizo más historia?</h1>
  <div id="score">Puntos: 0</div>
  <div id="round">Ronda: 1</div>
  <div id="timer">Tiempo restante: 30s</div>
  <div>
    <button id="resetBtn" onclick="resetGame()">Reiniciar</button>
    <button id="skipBtn" onclick="skipRound()">⏭️ Saltar Ronda (3)</button>
    <button id="muteBtn" onclick="toggleSound()">🔊 Sonido</button>
    <button id="themeBtn" onclick="toggleTheme()">🌙 Modo Oscuro</button>
  </div>
  <div class="container" id="playersContainer"></div>
  <div id="result"></div>

  <!-- Sonidos -->
  <audio
    id="soundCorrect"
    src="https://actions.google.com/sounds/v1/cartoon/clang_and_wobble.ogg"
    preload="auto"
  ></audio>
  <audio
    id="soundWrong"
    src="https://actions.google.com/sounds/v1/cartoon/wood_plank_flicks.ogg"
    preload="auto"
  ></audio>

  <script>
    const players = [
      { name: "Pelé", score: 98 },
      { name: "Diego Maradona", score: 96 },
      { name: "Lionel Messi", score: 100 },
      { name: "Cristiano Ronaldo", score: 99 },
      { name: "Johan Cruyff", score: 95 },
      { name: "Zinedine Zidane", score: 92 },
      { name: "Franz Beckenbauer", score: 94 },
      { name: "Ronaldinho", score: 90 },
      { name: "Ronaldo Nazário", score: 93 },
      { name: "Andrés Iniesta", score: 92 },
      { name: "Xavi Hernández", score: 91 },
      { name: "Paolo Maldini", score: 90 },
      { name: "George Best", score: 87 },
      { name: "Roberto Baggio", score: 86 },
      { name: "Garrincha", score: 93 },
      { name: "Michel Platini", score: 88 },
      { name: "Kylian Mbappé", score: 94 },
      { name: "Erling Haaland", score: 91 },
      { name: "Jude Bellingham", score: 90 },
      { name: "Vinícius Jr", score: 89 },
      { name: "Pedri", score: 87 },
      { name: "Lamine Yamal", score: 85 },
    ];

    let currentPair = [];
    let totalPoints = 0;
    let round = 1;
    let recentPlayers = [];
    let soundEnabled = true;
    let darkMode = false;
    let skipUses = 3;
    let timerInterval;
    let timeLeft = 30;
    let inputLocked = false;

    const playersContainer = document.getElementById("playersContainer");
    const resultDiv = document.getElementById("result");
    const scoreDiv = document.getElementById("score");
    const roundDiv = document.getElementById("round");
    const timerDiv = document.getElementById("timer");
    const skipBtn = document.getElementById("skipBtn");

    function shuffle(array) {
      return array.sort(() => Math.random() - 0.5);
    }

    function pickPlayers() {
      const availablePlayers = players.filter(
        (p) => !recentPlayers.includes(p.name)
      );
      if (availablePlayers.length < 2) {
        recentPlayers = [];
      }
      let pair = [];
      do {
        pair = shuffle(players).slice(0, 2);
      } while (recentPlayers.includes(pair[0].name) || recentPlayers.includes(pair[1].name));
      recentPlayers.push(pair[0].name, pair[1].name);
      if (recentPlayers.length > 10) {
        recentPlayers.shift();
        recentPlayers.shift();
      }
      currentPair = pair;
    }

    function showPlayers() {
      playersContainer.innerHTML = "";
      currentPair.forEach((player, index) => {
        const div = document.createElement("div");
        div.className = "player";
        div.dataset.index = index;
        div.textContent = player.name;
        div.onclick = () => {
          if (!inputLocked) {
            selectPlayer(index);
          }
        };
        playersContainer.appendChild(div);
      });
    }

    function selectPlayer(index) {
      if (inputLocked) return;
      inputLocked = true;
      clearInterval(timerInterval);

      const selected = currentPair[index];
      const other = currentPair[1 - index];
      let message = "";
      let pointsChange = 0;

      const playersDivs = document.querySelectorAll(".player");
      playersDivs.forEach((div) => div.className = "player"); // reset classes

      if (selected.score > other.score) {
        message = `¡Correcto! ${selected.name} tiene más historia.`;
        pointsChange = 50;
        playersDivs[index].classList.add("correct");
        playSound("correct");
      } else if (selected.score < other.score) {
        message = `Incorrecto. ${other.name} tiene más historia.`;
        pointsChange = -25;
        playersDivs[index].classList.add("wrong");
        playersDivs[1 - index].classList.add("correct");
        playSound("wrong");
      } else {
        // empate
        message = `Empate técnico, ambos tienen igual historia.`;
        pointsChange = 0;
        playersDivs[0].classList.add("tie");
        playersDivs[1].classList.add("tie");
        // no sonido
      }

      totalPoints += pointsChange;
      if (totalPoints < 0) totalPoints = 0;

      scoreDiv.textContent = `Puntos: ${totalPoints}`;
      resultDiv.textContent = message;

      round++;
      roundDiv.textContent = `Ronda: ${round}`;

      setTimeout(() => {
        if (round > 20) {
          endGame();
        } else {
          nextRound();
        }
      }, 2000);
    }

    function nextRound() {
      resultDiv.textContent = "";
      inputLocked = false;
      timeLeft = 30;
      timerDiv.textContent = `Tiempo restante: ${timeLeft}s`;
      pickPlayers();
      showPlayers();
      startTimer();
    }

    function startTimer() {
      clearInterval(timerInterval);
      timerInterval = setInterval(() => {
        timeLeft--;
        timerDiv.textContent = `Tiempo restante: ${timeLeft}s`;
        if (timeLeft <= 0) {
          clearInterval(timerInterval);
          inputLocked = true;
          resultDiv.textContent = "Se acabó el tiempo. Ronda perdida.";
          totalPoints -= 25;
          if (totalPoints < 0) totalPoints = 0;
          scoreDiv.textContent = `Puntos: ${totalPoints}`;
          round++;
          roundDiv.textContent = `Ronda: ${round}`;
          setTimeout(() => {
            if (round > 20) {
              endGame();
            } else {
              nextRound();
            }
          }, 2000);
        }
      }, 1000);
    }

    function endGame() {
      clearInterval(timerInterval);
      playersContainer.innerHTML = "";
      resultDiv.textContent = `Juego terminado. Puntuación final: ${totalPoints} puntos. ¡Gracias por jugar!`;
    }

    function resetGame() {
      totalPoints = 0;
      round = 1;
      recentPlayers = [];
      resultDiv.textContent = "";
      scoreDiv.textContent = "Puntos: 0";
      roundDiv.textContent = "Ronda: 1";
      inputLocked = false;
      skipUses = 3;
      skipBtn.textContent = `⏭️ Saltar Ronda (${skipUses})`;
      timeLeft = 30;
      timerDiv.textContent = `Tiempo restante: ${timeLeft}s`;
      pickPlayers();
      showPlayers();
      startTimer();
    }

    function skipRound() {
      if (skipUses <= 0 || inputLocked) return;
      skipUses--;
      skipBtn.textContent = `⏭️ Saltar Ronda (${skipUses})`;
      round++;
      roundDiv.textContent = `Ronda: ${round}`;
      totalPoints -= 5;
      if (totalPoints < 0) totalPoints = 0;
      scoreDiv.textContent = `Puntos: ${totalPoints}`;
      inputLocked = false;
      resultDiv.textContent = "Ronda saltada (-5 puntos).";
      clearInterval(timerInterval);
      setTimeout(() => {
        if (round > 20) {
          endGame();
        } else {
          nextRound();
        }
      }, 1500);
    }

    function toggleSound() {
      soundEnabled = !soundEnabled;
      document.getElementById("muteBtn").textContent = soundEnabled ? "🔊 Sonido" : "🔇 Silencio";
    }

    function playSound(type) {
      if (!soundEnabled) return;
      if (type === "correct") {
        document.getElementById("soundCorrect").play();
      } else if (type === "wrong") {
        document.getElementById("soundWrong").play();
      }
    }

    function toggleTheme() {
      darkMode = !darkMode;
      if (darkMode) {
        document.body.classList.add("dark");
        document.getElementById("themeBtn").textContent = "☀️ Modo Claro";
      } else {
        document.body.classList.remove("dark");
        document.getElementById("themeBtn").textContent = "🌙 Modo Oscuro";
      }
    }

    // Inicio
    resetGame();
  </script>
</body>
</html>
