<html>
<head>
<title>Cube Flipper</title>
<style>
    body {
        margin:0;
        background:black;
        display:flex;
        justify-content:center;
        align-items:center;
        height:100vh;
        color:white;
        font-family:Arial;
    }
    .container {
        display:flex;
        flex-direction:column;
        align-items:center;
        gap:10px;
    }
    canvas {
        background:#111;
        border:2px solid white;
    }
    .version {
        font-size:14px;
        opacity:0.8;
        margin-top:5px;
    }
</style>
</head>
<body>

<div class="container">
    <h1>Cube Flipper</h1>
    <canvas id="game" width="800" height="400"></canvas>
    <div class="version">Version 1 Beta</div>
</div>

<script>
let canvas = document.getElementById("game");
let ctx = canvas.getContext("2d");

let player = {
    x:50,
    y:300,
    size:30,
    dy:0,
    gravity:0.6,
    jumpPower:15,
    grounded:false,
    rotating:false,
    rot:0
};

let obstacles = [];
let speed = 4;
let score = 0;
let gameOver = false;

// Create a random obstacle
function spawnObstacle() {
    let height = 40 + Math.random()*80;
    let y = canvas.height - 20 - height;
    obstacles.push({x:canvas.width, y:y, w:40, h:height});
}

// Jump
function jump(){
    if(player.grounded && !gameOver){
        player.dy = -player.jumpPower;
        player.grounded=false;
        player.rotating=true;
    }
}

document.addEventListener("keydown", e => { if(e.code==="Space") jump(); });
canvas.addEventListener("mousedown", jump);

canvas.addEventListener("mousedown", ()=>{
    if(gameOver){
        resetGame();
    } else {
        jump();
    }
});

document.addEventListener("keydown", e => {
    if(e.code === "Space"){
        if(gameOver){
            resetGame();
        } else {
            jump();
        }
    }
});

function resetGame(){
    obstacles = [];
    spawnObstacle();
    player.x = 50;          // reset player x position
    player.y = 300;
    player.dy = 0;
    player.rot = 0;
    player.grounded = false;
    player.rotating = false;
    speed = 4;
    score = 0;
    gameOver = false;
}

// Game loop
function gameLoop(){
    ctx.clearRect(0,0,canvas.width,canvas.height);

    if(!gameOver){
        // Move obstacles
        obstacles.forEach(ob => { ob.x -= speed; });

        // Spawn new obstacles
        if(obstacles[obstacles.length-1].x < canvas.width - 250){
            spawnObstacle();
        }

        // Remove off-screen obstacles
        if(obstacles[0].x < -50){
            obstacles.shift();
        }

        // Update player
        player.dy += player.gravity;
        player.y += player.dy;

        let ground = canvas.height - 20 - player.size;
        if(player.y > ground){
            player.y = ground;
            player.dy = 0;
            player.grounded = true;
            player.rot = 0;
            player.rotating=false;
        }

        // Collision
        obstacles.forEach(ob => {
            if(player.x < ob.x + ob.w &&
               player.x + player.size > ob.x &&
               player.y < ob.y + ob.h &&
               player.y + player.size > ob.y){
                gameOver = true;
            }
        });

        // Rotate player if jumping
        if(player.rotating) player.rot += 0.25;

        // Increase speed gradually
        speed = 4 + score/1000;

        score++;
    }

    // Draw player
    ctx.save();
    ctx.translate(player.x + player.size/2, player.y + player.size/2);
    ctx.rotate(player.rot);
    ctx.fillStyle = "cyan";
    ctx.fillRect(-player.size/2, -player.size/2, player.size, player.size);
    ctx.restore();

    // Draw obstacles
    ctx.fillStyle = "red";
    obstacles.forEach(ob => {
        ctx.fillRect(ob.x, ob.y, ob.w, ob.h);
    });

    // Draw ground
    ctx.fillStyle = "green";
    ctx.fillRect(0, canvas.height-20, canvas.width, 20);

    // Draw score
    ctx.fillStyle = "white";
    ctx.font="20px Arial";
    ctx.textAlign="left";
    ctx.fillText("Score: " + score, 10, 27);

    if(gameOver){
        ctx.textAlign="center";
        ctx.font="40px Arial";
        ctx.fillText("Game Over", canvas.width/2, canvas.height/2);
        ctx.font="20px Arial";
        ctx.fillText("Click to Restart", canvas.width/2, canvas.height/2 + 40);

        canvas.addEventListener("mousedown", resetGame);
    }

    requestAnimationFrame(gameLoop);
}

// Start
spawnObstacle();
gameLoop();
</script>

</body>
</html>
