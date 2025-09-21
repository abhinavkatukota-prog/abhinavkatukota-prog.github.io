<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Pixel Platformer</title>
  <style>
    body {
      margin: 0;
      background: #222;
      display: flex;
      justify-content: center;
      align-items: center;
      height: 100vh;
      overflow: hidden;
    }
    canvas {
      border: 3px solid #fff;
      background: #87CEEB;
      image-rendering: pixelated;
    }
    #overlay {
      position: absolute;
      top: 0; left: 0;
      width: 100%; height: 100%;
      background: rgba(0,0,0,0.6);
      color: white;
      display: none;
      justify-content: center;
      align-items: center;
      flex-direction: column;
      font-family: monospace;
      font-size: 28px;
    }
    button {
      margin-top: 15px;
      padding: 10px 20px;
      font-size: 18px;
      cursor: pointer;
      border: none;
      border-radius: 4px;
    }
  </style>
</head>
<body>
  <canvas id="game" width="640" height="480"></canvas>
  <div id="overlay">
    <div id="message"></div>
    <button id="restartBtn">Restart</button>
  </div>

  <script>
    const canvas = document.getElementById("game");
    const ctx = canvas.getContext("2d");
    const overlay = document.getElementById("overlay");
    const message = document.getElementById("message");
    const restartBtn = document.getElementById("restartBtn");

    // Load graphics
    const sprites = {};
    const spriteSources = {
      player: "https://raw.githubusercontent.com/kenneyNL/platformer-art-deluxe/master/Base pack/Player/p1_front.png",
      block: "https://raw.githubusercontent.com/kenneyNL/platformer-art-deluxe/master/Base pack/Tiles/grassMid.png",
      flag: "https://raw.githubusercontent.com/kenneyNL/platformer-art-deluxe/master/Base pack/Items/flagRed2.png",
      enemy: "https://raw.githubusercontent.com/kenneyNL/platformer-art-deluxe/master/Base pack/Enemies/snailWalk1.png",
      coin: "https://raw.githubusercontent.com/kenneyNL/platformer-art-deluxe/master/Base pack/Items/coinGold.png"
    };

    let loaded = 0;
    const totalSprites = Object.keys(spriteSources).length;
    for (let key in spriteSources) {
      sprites[key] = new Image();
      sprites[key].src = spriteSources[key];
      sprites[key].onload = () => {
        loaded++;
        if (loaded === totalSprites) startGame();
      };
    }

    // Grid size
    const tileSize = 32;

    // Player
    let player = { x: 50, y: 400, width: 28, height: 28, dx: 0, dy: 0, onGround: false, lives: 3, score: 0 };

    // Physics
    const gravity = 0.6, jumpPower = -12, moveSpeed = 4;

    // Controls
    let keys = {};

    // Levels (ASCII maps)
    const levels = [
      [
        "............................",
        "............................",
        "..............C.............",
        "............................",
        "..............E.............",
        "........#####...............",
        "............................",
        "....#####..........F........",
        "#########.............#######"
      ],
      [
        "............................",
        "........C...................",
        "............................",
        "......#####.................",
        "..............E.............",
        "............................",
        "....#####..............F....",
        "............................",
        "#########.............#######"
      ],
      [
        "............................",
        "....F.......................",
        "...............C............",
        ".....................#####..",
        "............................",
        "..........#####.............",
        "...............E............",
        "............................",
        "#########.............#######"
      ]
    ];

    let currentLevel = 0;
    let platforms = [], enemies = [], coins = [], flag = null;

    function loadLevel(n) {
      platforms = [];
      enemies = [];
      coins = [];
      flag = null;

      player.x = 50;
      player.y = 400;
      player.dx = 0;
      player.dy = 0;
      player.onGround = false;

      const map = levels[n];
      for (let row = 0; row < map.length; row++) {
        for (let col = 0; col < map[row].length; col++) {
          const char = map[row][col];
          const x = col * tileSize, y = row * tileSize;
          if (char === "#") {
            platforms.push({ x, y, width: tileSize, height: tileSize });
          } else if (char === "F") {
            flag = { x, y, width: tileSize, height: tileSize };
          } else if (char === "E") {
            enemies.push({ x, y: y+8, width: 28, height: 28, dir: 1 });
          } else if (char === "C") {
            coins.push({ x: x+8, y: y+8, width: 16, height: 16, collected: false });
          }
        }
      }
    }

    function resetGame() {
      currentLevel = 0;
      player.lives = 3;
      player.score = 0;
      overlay.style.display = "none";
      loadLevel(currentLevel);
      gameLoop();
    }

    function winGame() {
      message.innerText = `🏆 You Win! Score: ${player.score}`;
      restartBtn.innerText = "Play Again";
      overlay.style.display = "flex";
    }

    restartBtn.onclick = resetGame;

    // Handle input
    document.addEventListener("keydown", e => { keys[e.code] = true; });
    document.addEventListener("keyup", e => { keys[e.code] = false; });

    function update() {
      // Movement
      if (keys["ArrowLeft"] || keys["KeyA"]) player.dx = -moveSpeed;
      else if (keys["ArrowRight"] || keys["KeyD"]) player.dx = moveSpeed;
      else player.dx = 0;

      if ((keys["ArrowUp"] || keys["Space"] || keys["KeyW"]) && player.onGround) {
        player.dy = jumpPower;
        player.onGround = false;
      }

      player.dy += gravity;
      player.x += player.dx;
      player.y += player.dy;

      // Collisions with platforms
      player.onGround = false;
      for (let p of platforms) {
        if (collision(player, p)) {
          if (player.dy > 0 && player.y + player.height - player.dy <= p.y) {
            player.y = p.y - player.height;
            player.dy = 0;
            player.onGround = true;
          } else if (player.dy < 0) {
            player.y = p.y + p.height;
            player.dy = 0;
          } else if (player.dx > 0) {
            player.x = p.x - player.width;
          } else if (player.dx < 0) {
            player.x = p.x + p.width;
          }
        }
      }

      // Enemies
      for (let e of enemies) {
        e.x += e.dir * 2;
        if (Math.random() < 0.01) e.dir *= -1;
        if (collision(player, e)) {
          player.lives--;
          if (player.lives <= 0) {
            message.innerText = "💀 Game Over";
            restartBtn.innerText = "Restart";
            overlay.style.display = "flex";
            return false;
          } else {
            player.x = 50; player.y = 400; player.dy = 0;
          }
        }
      }

      // Coins
      for (let c of coins) {
        if (!c.collected && collision(player, c)) {
          c.collected = true;
          player.score += 10;
        }
      }

      // Flag
      if (flag && collision(player, flag)) {
        currentLevel++;
        if (currentLevel >= levels.length) {
          winGame();
          return false;
        } else {
          loadLevel(currentLevel);
        }
      }

      // Fell off
      if (player.y > canvas.height) {
        player.lives--;
        if (player.lives <= 0) {
          message.innerText = "💀 Game Over";
          restartBtn.innerText = "Restart";
          overlay.style.display = "flex";
          return false;
        } else {
          player.x = 50; player.y = 400; player.dy = 0;
        }
      }
      return true;
    }

    function draw() {
      ctx.clearRect(0, 0, canvas.width, canvas.height);
      ctx.fillStyle = "#87CEEB";
      ctx.fillRect(0, 0, canvas.width, canvas.height);

      for (let p of platforms) ctx.drawImage(sprites.block, p.x, p.y, p.width, p.height);
      for (let c of coins) if (!c.collected) ctx.drawImage(sprites.coin, c.x, c.y, c.width, c.height);
      for (let e of enemies) ctx.drawImage(sprites.enemy, e.x, e.y, e.width, e.height);
      if (flag) ctx.drawImage(sprites.flag, flag.x, flag.y-16, flag.width, flag.height+16);
      ctx.drawImage(sprites.player, player.x, player.y, player.width, player.height);

      ctx.fillStyle = "black";
      ctx.font = "20px monospace";
      ctx.fillText(`❤️ ${player.lives}   ⭐ ${player.score}`, 10, 25);
    }

    function collision(a, b) {
      return a.x < b.x + b.width &&
             a.x + a.width > b.x &&
             a.y < b.y + b.height &&
             a.y + a.height > b.y;
    }

    function gameLoop() {
      if (update()) {
        draw();
        requestAnimationFrame(gameLoop);
      } else {
        draw();
      }
    }

    function startGame() {
      resetGame();
    }
  </script>
</body>
</html>
