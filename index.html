<!DOCTYPE html>
<html>
<head>
  <meta charset="UTF-8">
  <title>Pixel Platformer</title>
  <style>
    body {
      margin: 0;
      background: #1e1e1e;
      display: flex;
      justify-content: center;
      align-items: center;
      height: 100vh;
    }
    canvas {
      image-rendering: pixelated; /* keeps the retro look */
      border: 4px solid #fff;
      background: #87CEEB;
    }
  </style>
</head>
<body>
<canvas id="game" width="640" height="360"></canvas>
<script>
const canvas = document.getElementById("game");
const ctx = canvas.getContext("2d");

const keys = {};
document.addEventListener("keydown", e => keys[e.code] = true);
document.addEventListener("keyup", e => keys[e.code] = false);

// Player settings
const player = {
  x: 50,
  y: 0,
  width: 20,
  height: 20,
  dx: 0,
  dy: 0,
  speed: 3,
  jump: -8,
  grounded: false,
  color: "#ff4444"
};

// Gravity
const gravity = 0.4;

// Platforms
const platforms = [
  {x: 0, y: 340, w: 640, h: 20}, // ground
  {x: 120, y: 280, w: 100, h: 20},
  {x: 280, y: 220, w: 100, h: 20},
  {x: 440, y: 160, w: 100, h: 20}
];

function update() {
  // Horizontal movement
  if (keys["ArrowLeft"]) player.dx = -player.speed;
  else if (keys["ArrowRight"]) player.dx = player.speed;
  else player.dx = 0;

  // Jump
  if (keys["Space"] && player.grounded) {
    player.dy = player.jump;
    player.grounded = false;
  }

  // Apply gravity
  player.dy += gravity;

  // Move
  player.x += player.dx;
  player.y += player.dy;

  // Collision detection
  player.grounded = false;
  for (let p of platforms) {
    if (player.x < p.x + p.w &&
        player.x + player.width > p.x &&
        player.y < p.y + p.h &&
        player.y + player.height > p.y) {
      // Collision from top
      if (player.dy > 0 && player.y + player.height - player.dy <= p.y) {
        player.y = p.y - player.height;
        player.dy = 0;
        player.grounded = true;
      }
    }
  }

  // Keep player inside canvas
  if (player.x < 0) player.x = 0;
  if (player.x + player.width > canvas.width) player.x = canvas.width - player.width;
}

function draw() {
  ctx.imageSmoothingEnabled = false;
  ctx.clearRect(0, 0, canvas.width, canvas.height);

  // Sky
  ctx.fillStyle = "#87CEEB";
  ctx.fillRect(0, 0, canvas.width, canvas.height);

  // Platforms
  ctx.fillStyle = "#654321";
  for (let p of platforms) {
    ctx.fillRect(p.x, p.y, p.w, p.h);
  }

  // Player
  ctx.fillStyle = player.color;
  ctx.fillRect(Math.round(player.x), Math.round(player.y), player.width, player.height);
}

function loop() {
  update();
  draw();
  requestAnimationFrame(loop);
}

loop();
</script>
</body>
</html>
