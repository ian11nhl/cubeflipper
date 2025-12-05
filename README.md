<html>
<head>
<title>Cube Flipper</title>
<style>
    body {
        background:black;
        margin:0;
        overflow:hidden;
        display:flex;
        justify-content:center;
        align-items:center;
        height:100vh;
        color:white;
        font-family:Arial;
    }
    canvas {
        border:2px solid white;
    }
</style>
</head>
<body>

<canvas id="game" width="800" height="400"></canvas>

<script>
let canvas = document.getElementById("game");
let ctx = canvas.getContext("2d");

let player = {
    x: 50,
    y: 300,
    size: 30,
    dy: 0,
    gravity: 0.6,
    jumpPower: 15,
    grounded: false,
    rotating: false,
    rot: 0
};

let scrollSpeed = 4;
let obstacles = [];
let gameRunning = false;
let gameOver = false;
let inMenu = true;
let score = 0;

function spawnObstacle() {
    let height = 40 + Math.random() * 60;
    let y = canvas.height - height - 20;
    obstacles.push({ x: 800, y: y, w: 40, h: height });
}

function jump() {
    if (player.grounded && !gameOver && gameRunning) {
        player.dy = -player.jumpPower;
        player.grounded = false;
        player.rotating = true;
    }
}

document.addEventListener("keydown", e => {
    if (e.code === "Space") {
        if (inMenu) {
            inMenu = false;
            resetGame();
        } else {
            jump();
        }
    }
});

canvas.addEventListener("mousedown", () => {
    if (inMenu) {
        inMenu = false;
        resetGame();
    } else {
        jump();
    }
});

function resetGame() {
    obstacles = [];
    spawnObstacle();
    player.x = 50;
    player.y = 300;
    player.dy = 0;
    gameRunning = true;
    gameOver = false;
    score = 0;
}

function gameLoop() {
    ctx.clearRect(0, 0, canvas.width, canvas.height);

    if (inMenu) {
        ctx.fillStyle = "white";
        ctx.textAlign = "center";
        ctx.font = "45px Arial";
        ctx.fillText("Cube Flipper", canvas.width / 2, 150);
        ctx.font = "25px Arial";
        ctx.fillText("Enjoy the game made by Ian", canvas.width / 2, 200);
        ctx.fillText("Click or Press Space to Start", canvas.width / 2, 260);
        requestAnimationFrame(gameLoop);
        return;
    }

    if (gameRunning) {
        score++;

        player.dy += player.gravity;
        player.y += player.dy;

        let ground = canvas.height - 20 - player.size;
        if (player.y >= ground) {
            player.y = ground;
            player.dy = 0;
            player.grounded = true;
            player.rotating = false;
            player.rot = 0;
        }

        for (let o of obstacles) {
            o.x -= scrollSpeed;

            if (
                player.x < o.x + o.w &&
                player.x + player.size > o.x &&
                player.y < o.y + o.h &&
                player.y + player.size > o.y
            ) {
                gameRunning = false;
                gameOver = true;
            }
        }

        if (obstacles[obstacles.length - 1].x < 300) {
            spawnObstacle();
        }

        if (obstacles[0].x < -50) {
            obstacles.shift();
        }
    }

    if (player.rotating) player.rot += 0.25;
    ctx.save();
    ctx.translate(player.x + player.size / 2, player.y + player.size / 2);
    ctx.rotate(player.rot);
    ctx.fillStyle = "blue";
    ctx.fillRect(-player.size / 2, -player.size / 2, player.size, player.size);
    ctx.restore();

    ctx.fillStyle = "red";
    for (let o of obstacles) {
        ctx.fillRect(o.x, o.y, o.w, o.h);
    }

    ctx.fillStyle = "white";
    ctx.font = "20px Arial";
    ctx.textAlign = "left";
    ctx.fillText("Score: " + score, 10, 27);

    if (gameOver) {
        ctx.textAlign = "center";
        ctx.font = "40px Arial";
        ctx.fillText("Game Over! Tap to Restart", canvas.width / 2, canvas.height / 2);
        canvas.addEventListener("mousedown", resetGame);
        document.addEventListener("keydown", e => { if (e.code === "Space") resetGame(); });
    }

    requestAnimationFrame(gameLoop);
}

spawnObstacle();
gameLoop();
</script>

</body>
</html>
