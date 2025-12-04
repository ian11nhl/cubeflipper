<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<title>Cube Flipper</title>
<style>
  html, body {
    margin: 0;
    padding: 0;
    overflow: hidden;
    height: 100%;
    background: #1e1e1e;
    display: flex;
    justify-content: center;
    align-items: center;
  }
  canvas {
    background: linear-gradient(#2b2b2b, #1e1e1e);
    display: block;
  }
</style>
</head>
<body>
<canvas id="game"></canvas>
<script>
const canvas = document.getElementById('game');
const ctx = canvas.getContext('2d');

const BASE_WIDTH = 800;
const BASE_HEIGHT = 400;

let scale = 1;
let offsetX = 0;
let offsetY = 0;

function resizeCanvas() {
    const windowRatio = window.innerWidth / window.innerHeight;
    const gameRatio = BASE_WIDTH / BASE_HEIGHT;

    if (windowRatio > gameRatio) {
        scale = window.innerHeight / BASE_HEIGHT;
        offsetX = (window.innerWidth - BASE_WIDTH * scale) / 2;
        offsetY = 0;
    } else {
        scale = window.innerWidth / BASE_WIDTH;
        offsetX = 0;
        offsetY = (window.innerHeight - BASE_HEIGHT * scale) / 2;
    }
    canvas.width = window.innerWidth;
    canvas.height = window.innerHeight;
}
window.addEventListener('resize', resizeCanvas);
resizeCanvas();

// Helper to convert game coords to canvas coords
function toCanvasX(x) { return x * scale + offsetX; }
function toCanvasY(y) { return y * scale + offsetY; }
function toCanvasSize(s) { return s * scale; }

let player = {
    x: 50,
    y: 300,
    width: 30,
    height: 30,
    dy: 0,
    jumpPower: 12,
    grounded: false,
    rotation: 0,
    jumping: false
};

let gravity = 0.6;
let obstacles = [];
let gameSpeed = 6;
let frame = 0;
let gameOver = false;
let score = 0;
let particles = [];
let gameStarted = false;

// Jump / start handler
function jump() {
    if (!gameStarted) {
        gameStarted = true;
        return;
    }
    if (player.grounded) {
        player.dy = -player.jumpPower;
        player.grounded = false;
        player.jumping = true;
    }
    if (gameOver) resetGame();
}
document.addEventListener('keydown', e => { if (e.code === 'Space' || e.code === 'ArrowUp') jump(); });
canvas.addEventListener('mousedown', jump);
canvas.addEventListener('touchstart', jump);

// Spawn obstacles
function spawnObstacle() {
    const height = 20 + Math.random() * 60;
    const width = 20 + Math.random() * 40;
    obstacles.push({ x: BASE_WIDTH, y: BASE_HEIGHT - height, width, height });
}

// Reset game
function resetGame() {
    player.y = 300;
    player.dy = 0;
    player.rotation = 0;
    player.jumping = false;
    obstacles = [];
    frame = 0;
    gameOver = false;
    gameSpeed = 6;
    score = 0;
    particles = [];
    gameStarted = false;
}

// Create landing particles
function createParticles(x, y) {
    for (let i = 0; i < 10; i++) {
        particles.push({
            x: x,
            y: y,
            dx: (Math.random() - 0.5) * 4,
            dy: Math.random() * -2,
            alpha: 1,
            size: 3 + Math.random() * 3
        });
    }
}

// Update game state
function update() {
    if (!gameStarted || gameOver) return;

    player.dy += gravity;
    player.y += player.dy;

    if (!player.grounded && player.jumping) {
        player.rotation += 0.3;
    }

    if (player.y + player.height >= BASE_HEIGHT) {
        if (!player.grounded) createParticles(player.x + player.width/2, BASE_HEIGHT - 5);
        player.y = BASE_HEIGHT - player.height;
        player.dy = 0;
        player.grounded = true;
        player.rotation = 0;
        player.jumping = false;
    }

    if (frame % 80 === 0) spawnObstacle();
    obstacles.forEach(ob => ob.x -= gameSpeed);
    obstacles = obstacles.filter(ob => ob.x + ob.width > 0);

    for (let ob of obstacles) {
        if (player.x < ob.x + ob.width &&
            player.x + player.width > ob.x &&
            player.y < ob.y + ob.height &&
            player.y + player.height > ob.y) {
            gameOver = true;
        }
    }

    particles.forEach(p => {
        p.x += p.dx;
        p.y += p.dy;
        p.dy += 0.1;
        p.alpha -= 0.03;
    });
    particles = particles.filter(p => p.alpha > 0);

    if (frame % 300 === 0) gameSpeed += 0.5;

    score = Math.floor(frame / 5);
    frame++;
}

// Draw everything
function draw() {
    ctx.clearRect(0, 0, canvas.width, canvas.height);

    if (!gameStarted) {
        ctx.fillStyle = 'white';
        ctx.font = `${60 * scale}px Arial`;
        ctx.textAlign = 'center';
        ctx.fillText('Cube Flipper', canvas.width/2, toCanvasY(150));

        ctx.font = `${24 * scale}px Arial`;
        ctx.fillText('Enjoy the game! Made by Ian', canvas.width/2, toCanvasY(220));
        ctx.fillText('Tap or Press Space to Start', canvas.width/2, toCanvasY(270));
        return;
    }

    // Background grid
    for (let i = 0; i < BASE_WIDTH; i += 40) {
        ctx.strokeStyle = '#333';
        ctx.beginPath();
        ctx.moveTo(toCanvasX(i - (frame * 2 % 40)), 0);
        ctx.lineTo(toCanvasX(i - (frame * 2 % 40)), canvas.height);
        ctx.stroke();
    }

    // Ground
    ctx.fillStyle = '#555';
    ctx.fillRect(0, toCanvasY(BASE_HEIGHT - 10), canvas.width, toCanvasSize(10));

    // Player
    ctx.save();
    ctx.translate(toCanvasX(player.x + player.width/2), toCanvasY(player.y + player.height/2));
    ctx.rotate(player.rotation);
    ctx.fillStyle = '#00ffcc';
    ctx.fillRect(-toCanvasSize(player.width)/2, -toCanvasSize(player.height)/2, toCanvasSize(player.width), toCanvasSize(player.height));
    ctx.restore();

    // Obstacles
    ctx.fillStyle = '#ff4c4c';
    obstacles.forEach(ob => ctx.fillRect(toCanvasX(ob.x), toCanvasY(ob.y), toCanvasSize(ob.width), toCanvasSize(ob.height)));

    // Particles
    particles.forEach(p => {
        ctx.globalAlpha = p.alpha;
        ctx.fillStyle = 'yellow';
        ctx.fillRect(toCanvasX(p.x), toCanvasY(p.y), toCanvasSize(p.size), toCanvasSize(p.size));
        ctx.globalAlpha = 1;
    });

    // Score
    ctx.fillStyle = 'white';
    ctx.font = `${20 * scale}px Arial`;
    ctx.fillText('Score: ' + score, toCanvasX(10), toCanvasY(60));

    if (gameOver) {
        ctx.fillStyle = 'white';
        ctx.font = `${40 * scale}px Arial`;
        ctx.fillText('Game Over! Tap to Restart', toCanvasX(80), toCanvasY(200));
    }
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
