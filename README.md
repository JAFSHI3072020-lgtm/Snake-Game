<!doctype html>
<html lang="en">
<head>
  <meta charset="utf-8">
  <title>Snake Game</title>
  <meta name="viewport" content="width=device-width, initial-scale=1">

  <style>
    :root {
      --bg: #cfcfcf;
      --grid: #b8b8b8;
      --snake: #22c55e;
      --snake-head: #10b981;
      --fruit: #ef4444;
      --text: #111;
      --accent: #0077cc;
    }

    body {
      margin: 0;
      min-height: 100vh;
      display: grid;
      place-items: center;
      background: var(--bg);
      font-family: Arial, sans-serif;
      color: var(--text);
    }

    .container {
      display: grid;
      gap: 16px;
      text-align: center;
    }

    canvas {
      background: var(--grid);
      border: 6px solid #444;
      border-radius: 8px;
    }

    .controls {
      display: flex;
      gap: 12px;
      justify-content: center;
    }

    button {
      padding: 10px 14px;
      border-radius: 8px;
      border: 1px solid #333;
      background: #eee;
      cursor: pointer;
      font-weight: bold;
    }

    #gameOverScreen {
      position: absolute;
      background: rgba(0,0,0,0.7);
      color: white;
      padding: 40px;
      border-radius: 16px;
      font-size: 24px;
      display: none;
      text-align: center;
    }

    #gameOverScreen button {
      margin-top: 20px;
      padding: 10px 20px;
      background: #22c55e;
      border: none;
      font-size: 18px;
      border-radius: 10px;
    }
  </style>
</head>

<body>
  <div class="container">
    <h1>Snake</h1>

    <div>
      Score: <span id="score">0</span> |
      High Score: <span id="highScore">0</span>
    </div>

    <canvas id="board" width="600" height="400"></canvas>

    <div class="controls">
      <button id="restart">Restart</button>
      <button id="pause">Pause</button>
    </div>
  </div>

  <div id="gameOverScreen">
    <div id="gameOverText">GAME OVER</div>
    <button id="goRestart">RESTART</button>
  </div>

  <script>
    const canvas = document.getElementById("board");
    const ctx = canvas.getContext("2d");

    const scoreEl = document.getElementById("score");
    const highScoreEl = document.getElementById("highScore");   // ⭐ ADDED
    const restartBtn = document.getElementById("restart");
    const pauseBtn = document.getElementById("pause");

    const gameOverScreen = document.getElementById("gameOverScreen");
    const goRestart = document.getElementById("goRestart");

    const CELL = 20;
    const COLS = canvas.width / CELL;
    const ROWS = canvas.height / CELL;

    const DIR = {
      UP: {x: 0, y: -1},
      DOWN: {x: 0, y: 1},
      LEFT: {x: -1, y: 0},
      RIGHT: {x: 1, y: 0}
    };

    let dir, snake, fruit, score, paused, speed, lastStep, gameOver;
    let highScore = Number(localStorage.getItem("snakeHighScore")) || 0; // ⭐ ADDED
    highScoreEl.textContent = highScore; // ⭐ ADDED

    function randCell() {
      return {
        x: Math.floor(Math.random() * COLS),
        y: Math.floor(Math.random() * ROWS)
      };
    }

    function init() {
      snake = [
        {x: 10, y: 10},
        {x: 9, y: 10},
        {x: 8, y: 10}
      ];
      dir = DIR.RIGHT;
      fruit = randCell();
      score = 0;
      paused = false;
      gameOver = false;
      lastStep = 0;
      speed = 8;

      scoreEl.textContent = score;
      highScoreEl.textContent = highScore;
      gameOverScreen.style.display = "none";
    }

    function drawCell(x, y, color) {
      ctx.fillStyle = color;
      ctx.fillRect(x * CELL + 1, y * CELL + 1, CELL - 2, CELL - 2);
    }

    function draw() {
      ctx.clearRect(0, 0, canvas.width, canvas.height);

      drawCell(fruit.x, fruit.y, "#ef4444");

      snake.forEach((s, i) => {
        drawCell(s.x, s.y, i === 0 ? "#10b981" : "#22c55e");
      });
    }

    function step() {
      const head = snake[0];
      const next = {x: head.x + dir.x, y: head.y + dir.y};

      if (next.x < 0 || next.x >= COLS || next.y < 0 || next.y >= ROWS) {
        triggerGameOver();
        return;
      }

      if (snake.some((s, i) => i !== 0 && s.x === next.x && s.y === next.y)) {
        triggerGameOver();
        return;
      }

      snake.unshift(next);

      if (next.x === fruit.x && next.y === fruit.y) {
        score += 10;
        scoreEl.textContent = score;

        // ⭐ UPDATE HIGH SCORE
        if (score > highScore) {
          highScore = score;
          highScoreEl.textContent = highScore;
          localStorage.setItem("snakeHighScore", highScore);
        }

        do {
          fruit = randCell();
        } while (snake.some(s => s.x === fruit.x && s.y === fruit.y));
      } else {
        snake.pop();
      }
    }

    function triggerGameOver() {
      paused = true;
      gameOver = true;

      gameOverScreen.style.display = "block";
      document.getElementById("gameOverText").innerText =
        "GAME OVER\nSCORE: " + score + "\nHIGH SCORE: " + highScore; // ⭐ ADDED
    }

    function loop(ts) {
      if (!paused && !gameOver) {
        const interval = 1000 / speed;
        if (ts - lastStep >= interval) {
          step();
          draw();
          lastStep = ts;
        }
      }
      requestAnimationFrame(loop);
    }

    window.addEventListener("keydown", (e) => {
      const k = e.key.toLowerCase();
      const prev = dir;

      if (k === "arrowup" || k === "w") dir = DIR.UP;
      else if (k === "arrowdown" || k === "s") dir = DIR.DOWN;
      else if (k === "arrowleft" || k === "a") dir = DIR.LEFT;
      else if (k === "arrowright" || k === "d") dir = DIR.RIGHT;
      else if (k === " ") paused = !paused;

      if (prev.x + dir.x === 0 && prev.y + dir.y === 0) dir = prev;
    });

    restartBtn.addEventListener("click", () => { init(); draw(); });
    goRestart.addEventListener("click", () => { init(); draw(); });
    pauseBtn.addEventListener("click", () => (paused = !paused));

    init();
    draw();
    requestAnimationFrame(loop);
  </script>
</body>
</html>
