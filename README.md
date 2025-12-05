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
<canvas id="game" width="800" height="400"></canvas>
<script>
const canvas = document.getElementById('game');
const ctx = canvas.getContext('2d');

const W = 800, H = 400;

// player
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

// Game states
let levelIndex = 0;
let levelComplete = false;
let gameOver = false;
let started = false;

// LEVEL DATA
// obstacles + platforms + end goal
let levels = [
    {
        length: 2000,
        obstacles: [
            {x: 400, y: 340, w: 40, h: 60},
            {x: 700, y: 320, w: 50, h: 80},
            {x: 1100, y: 340, w: 40, h: 60}
        ],
        platforms: [
            {x: 900, y: 280, w: 100, h: 20}
        ]
    },
    {
        length: 2600,
        obstacles: [
            {x: 350, y: 340, w: 50, h: 60},
            {x: 800, y: 320, w: 60, h: 80},
            {x: 1300, y: 340, w: 50, h: 60},
            {x: 1800, y: 330, w: 70, h: 70}
        ],
        platforms: [
            {x: 600, y: 260, w: 120, h: 20},
            {x: 1400, y: 260, w: 100, h: 20},
        ]
    }
];

let scroll = 0;

// controls
function jump() {
    if (!started) {
        started = true;
        return;
    }
    if (gameOver || levelComplete) {
        restart();
        return;
    }
    if (player.grounded) {
        player.dy = -player.jumpPower;
        player.grounded = false;
        player.rotating = true;
    }
}
document.addEventListener("keydown", e => {
    if (e.code === "Space" || e.code === "ArrowUp") jump();
});
canvas.addEventListener("mousedown", jump);

// reset
function restart() {
    player.x = 50;
    player.y = 300;
    player.dy = 0;
    scroll = 0;
    player.rot = 0;
    player.rotating = false;
    gameOver = false;
    levelComplete = false;
}

// update logic
function update() {
    if (!started || gameOver || levelComplete) return;

    scroll += 4;

    player.dy += player.gravity;
    player.y += player.dy;

    if (player.rotating) {
        player.rot += 0.3;
    }

    // ground collision
    if (player.y + player.size > H) {
        player.y = H - player.size;
        player.dy = 0;
        player.grounded = true;
        player.rot = 0;
        player.rotating = false;
    }

    let lvl = levels[levelIndex];

    // platform collisions
    for (let p of lvl.platforms) {
        let px = p.x - scroll;
        if (player.x < px + p.w &&
            player.x + player.size > px &&
            player.y + player.size > p.y &&
            player.y + player.size < p.y + p.h + 10 &&
            player.dy >= 0) {
            player.y = p.y - player.size;
            player.dy = 0;
            player.grounded = true;
            player.rot = 0;
            player.rotating = false;
        }
    }

    // obstacle collisions
    for (let ob of lvl.obstacles) {
        let ox = ob.x - scroll;
        if (player.x < ox + ob.w &&
            player.x + player.size > ox &&
            player.y < ob.y + ob.h &&
            player.y + player.size > ob.y) {
            gameOver = true;
        }
    }

    // finish line
    if (scroll >= lvl.length) {
        levelComplete = true;
        levelIndex = (levelIndex + 1) % levels.length;
    }
}

// draw function
function draw() {
    ctx.clearRect(0,0,W,H);

    // title screen
    if (!started) {
        ctx.fillStyle = "white";
        ctx.font = "60px Arial";
        ctx.textAlign = "center";
        ctx.fillText("Cube Flipper", W/2, 150);
        ctx.font = "24px Arial";
        ctx.fillText("Enjoy the game! Made by Ian", W/2, 210);
        ctx.fillText("Click or Space to Start", W/2, 270);
        return;
    }

    // ground
    ctx.fillStyle = "#555";
    ctx.fillRect(0, H-10, W, 10);

    let lvl = levels[levelIndex];

    // platforms
    ctx.fillStyle = "#00aaff";
    for (let p of lvl.platforms) {
        ctx.fillRect(p.x - scroll, p.y, p.w, p.h);
    }

    // obstacles
    ctx.fillStyle = "#ff4c4c";
    for (let ob of lvl.obstacles) {
        ctx.fillRect(ob.x - scroll, ob.y, ob.w, ob.h);
    }

    // player cube with rotation
    ctx.save();
    ctx.translate(player.x + player.size/2, player.y + player.size/2);
    ctx.rotate(player.rot);
    ctx.fillStyle = "#00ffcc";
    ctx.fillRect(-player.size/2, -player.size/2, player.size, player.size);
    ctx.restore();

    // finish line
    ctx.fillStyle = "yellow";
    ctx.fillRect(lvl.length - scroll, 0, 10, H);

    // text overlays
    if (levelComplete) {
        ctx.fillStyle = "white";
        ctx.font = "40px Arial";
        ctx.textAlign = "center";
        ctx.fillText("Level Complete!", W/2, 190);
        ctx.font = "25px Arial";
        ctx.fillText("Click to load next level", W/2, 240);
    }

    if (gameOver) {
        ctx.fillStyle = "white";
        ctx.font = "40px Arial";
        ctx.textAlign = "center";
        ctx.fillText("Game Over", W/2, 190);
        ctx.font = "25px Arial";
        ctx.fillText("Click to Restart", W/2, 240);
    }
}

// loop
function loop() {
    update();
    draw();
    requestAnimationFrame(loop);
}
loop();
</script>
</body>
</html>
