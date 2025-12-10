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
<canvas id="game" width="800" height="400"></canvas>
<script>
const canvas = document.getElementById('game');
const ctx = canvas.getContext('2d');

const BASE_WIDTH = 800;
const BASE_HEIGHT = 400;

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
        ctx.font = `60px Arial`;
        ctx.textAlign = 'center';
        ctx.fillText('Cube Flipper', BASE_WIDTH / 2, 150);

        ctx.font = `24px Arial`;
        ctx.fillText('Enjoy the game! Made by Ian', BASE_WIDTH / 2, 220);
        ctx.fillText('Tap or Press Space to Start', BASE_WIDTH / 2, 270);
        return;
    }

    // Background grid
    for (let i = 0; i < BASE_WIDTH; i += 40) {
        ctx.strokeStyle = '#333';
        ctx.beginPath();
        ctx.moveTo(i - (frame * 2 % 40), 0);
        ctx.lineTo(i - (frame * 2 % 40), BASE_HEIGHT);
        ctx.stroke();
    }

    // Ground
    ctx.fillStyle = '#555';
    ctx.fillRect(0, BASE_HEIGHT - 10, BASE_WIDTH, 10);

    // Player
    ctx.save();
    ctx.translate(player.x + player.width/2, player.y + player.height/2);
    ctx.rotate(player.rotation);
    ctx.fillStyle = '#00ffcc';
    ctx.fillRect(-player.width/2, -player.height/2, player.width, player.height);
    ctx.restore();

    // Obstacles
    ctx.fillStyle = '#ff4c4c';
    obstacles.forEach(ob => ctx.fillRect(ob.x, ob.y, ob.width, ob.height));

    // Particles
    particles.forEach(p => {
        ctx.globalAlpha = p.alpha;
        ctx.fillStyle = 'yellow';
        ctx.fillRect(p.x, p.y, p.size, p.size);
        ctx.globalAlpha = 1;
    });

    // Score
    ctx.fillStyle = 'white';
    ctx.font = `20px Arial`;
    ctx.fillText('Score: ' + score, 10, 60);

    if (gameOver) {
        ctx.fillStyle = 'white';
        ctx.font = `40px Arial`;
        ctx.fillText('Game Over! Tap to Restart', 80, 200);
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
