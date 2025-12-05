
<html>
<head>
<title>Cube Flipper</title>
<style>
    body {
        margin: 0;
        text-align: center;
        background: #111;
        color: white;
        font-family: Arial, sans-serif;
    }
    canvas {
        background: #333;
        display: block;
        margin: auto;
        margin-top: 20px;
        border: 2px solid white;
    }
</style>
</head>
<body>

<h1>Cube Flipper</h1>
<p>Enjoy the game made by Ian!</p>
<p id="scoreDisplay">Score: 0 | Level: 1</p>
<canvas id="gameCanvas" width="500" height="300"></canvas>

<script>
const canvas = document.getElementById("gameCanvas");
const ctx = canvas.getContext("2d");

// Player state
let player = {
    x: 50,
    y: 240,
    size: 30,
    velocityY: 0,
    gravity: 0.6,
    onGround: true,
    rotation: 0
};

// Jump power increased
const JUMP_POWER = -13;

let score = 0;
let level = 1;
let gameRunning = true;
let message = "";

// Level set – now includes more levels (extendable!)
const levels = [
    [100, 200, 300, 450],               // Level 1
    [120, 200, 320, 380, 450],          // Level 2
    [140, 220, 300, 360, 430, 480],     // Level 3
    [150, 260, 330, 400, 450, 500],     // Level 4
    [160, 230, 300, 370, 430, 480, 520] // Level 5
];

let obstacles = [];
let playerForwardSpeed = 2;

function loadLevel(lvl) {
    if (lvl > levels.length) {
        message = "🎉 YOU BEAT ALL LEVELS! Tap to restart";
        gameRunning = false;
        return;
    }
    
    obstacles = levels[lvl - 1].map(x => ({ x, y: 260, w: 30, h: 30 }));
    player.x = 50;
    player.y = 240;
    player.velocityY = 0;
    score = 0;
    player.rotation = 0;

    document.getElementById("scoreDisplay").innerText =
        `Score: ${score} | Level: ${level}`;
}

canvas.addEventListener("click", () => {
    if (!gameRunning) {
        level = 1;
        gameRunning = true;
        message = "";
        loadLevel(level);
        return;
    }
    if (player.onGround) {
        player.velocityY = JUMP_POWER;
        player.onGround = false;
    }
});

function drawPlayer() {
    ctx.save();
    ctx.translate(player.x + player.size / 2, player.y + player.size / 2);
    ctx.rotate(player.rotation);
    ctx.fillStyle = "cyan";
    ctx.fillRect(-player.size/2, -player.size/2, player.size, player.size);
    ctx.restore();
}

function updatePlayer() {
    player.rotation += (!player.onGround ? 0.2 : 0);

    player.velocityY += player.gravity;
    player.y += player.velocityY;

    if (player.y > 240) {
        player.y = 240;
        player.velocityY = 0;
        player.onGround = true;
        player.rotation = 0;
    }
}

function drawObstacles() {
    ctx.fillStyle = "red";
    obstacles.forEach(ob => {
        ctx.fillRect(ob.x, ob.y, ob.w, ob.h);
    });
}

function checkCollision() {
    for (let ob of obstacles) {
        if (
            player.x < ob.x + ob.w &&
            player.x + player.size > ob.x &&
            player.y < ob.y + ob.h &&
            player.y + player.size > ob.y
        ) {
            message = "Game Over! Tap to restart";
            gameRunning = false;
        }
    }
}

function gameLoop() {
    ctx.clearRect(0, 0, canvas.width, canvas.height);

    if (gameRunning) {
        updatePlayer();
        drawPlayer();
        drawObstacles();

        player.x += playerForwardSpeed;
        score++;

        // Off-screen = level complete
        if (player.x > canvas.width) {
            level++;
            message = `✔ Level ${level - 1} Complete!`;
            gameRunning = false;
            setTimeout(() => {
                gameRunning = true;
                message = "";
                loadLevel(level);
            }, 800);
        }

        checkCollision();
        
        document.getElementById("scoreDisplay").innerText =
            `Score: ${score} | Level: ${level}`;
    } else {
        drawPlayer();
        drawObstacles();
        ctx.fillStyle = "white";
        ctx.font = "20px Arial";
        ctx.fillText(message, canvas.width/2 - 100, canvas.height/2);
    }

    requestAnimationFrame(gameLoop);
}

loadLevel(level);
gameLoop();

</script>
</body>
</html>
