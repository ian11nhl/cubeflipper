
<html lang="en">
<head>
<meta charset="UTF-8">
<title>Cube Jumper</title>
</head>
<style>
  body { margin: 0; overflow: hidden; background: #1e1e1e; display: flex; justify-content: center; align-items: center; height: 100vh; }
  canvas { display: block; background: linear-gradient(#2b2b2b, #1e1e1e); }
  .bright {color: white;}
</style>

<body>
  <h1 class="bright">Enjoy this game</h1>
  <h2 class="bright">Made by Ian</h2>
<canvas id="game" width="800" height="400"></canvas>
<script>
const canvas = document.getElementById('game');
const ctx = canvas.getContext('2d');

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

// Jump handler
function jump() {
    if (player.grounded) {
        player.dy = -player.jumpPower;
        player.grounded = false;
        player.jumping = true; // start flip
    }
    if (gameOver) resetGame();
}
document.addEventListener('keydown', e => { if (e.code === 'Space' || e.code === 'ArrowUp') jump(); });
canvas.addEventListener('mousedown', jump);

// Spawn obstacles
function spawnObstacle() {
    let height = 20 + Math.random() * 60;
    let width = 20 + Math.random() * 40;
    obstacles.push({ x: canvas.width, y: canvas.height - height, width, height });
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
    if (gameOver) return;

    // Player physics
    player.dy += gravity;
    player.y += player.dy;

    // Flip animation: rotate while in the air
    if (!player.grounded && player.jumping) {
        player.rotation += 0.3; // adjust for speed of flip
    }

    // Ground collision
    if (player.y + player.height >= canvas.height) {
        if (!player.grounded) createParticles(player.x + player.width/2, canvas.height - 5);
        player.y = canvas.height - player.height;
        player.dy = 0;
        player.grounded = true;
        player.rotation = 0;
        player.jumping = false;
    }

    // Spawn obstacles
    if (frame % 80 === 0) spawnObstacle();
    obstacles.forEach(ob => ob.x -= gameSpeed);
    obstacles = obstacles.filter(ob => ob.x + ob.width > 0);

    // Collision detection
    for (let ob of obstacles) {
        if (player.x < ob.x + ob.width &&
            player.x + player.width > ob.x &&
            player.y < ob.y + ob.height &&
            player.y + player.height > ob.y) {
            gameOver = true;
        }
    }

    // Particles update
    particles.forEach(p => {
        p.x += p.dx;
        p.y += p.dy;
        p.dy += 0.1; // gravity
        p.alpha -= 0.03;
    });
    particles = particles.filter(p => p.alpha > 0);

    // Increase speed over time
    if (frame % 300 === 0) gameSpeed += 0.5;

    // Score
    score = Math.floor(frame / 5);
    frame++;
}

// Draw everything
function draw() {
    ctx.clearRect(0, 0, canvas.width, canvas.height);

    // Background grid effect
    for (let i = 0; i < canvas.width; i += 40) {
        ctx.strokeStyle = '#333';
        ctx.beginPath();
        ctx.moveTo(i - (frame * 2 % 40), 0);
        ctx.lineTo(i - (frame * 2 % 40), canvas.height);
        ctx.stroke();
    }

    // Ground
    ctx.fillStyle = '#555';
    ctx.fillRect(0, canvas.height - 10, canvas.width, 10);

    // Player with rotation
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
    ctx.font = '20px Arial';
    ctx.fillText('Score: ' + score, 10, 30);

    // Game Over
    if (gameOver) {
        ctx.fillStyle = 'white';
        ctx.font = '40px Arial';
        ctx.fillText('Game Over! Click to Restart', 120, 200);
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
