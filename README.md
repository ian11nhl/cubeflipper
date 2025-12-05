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
    jumpPower:12,
    grounded:false
};

let obstacle = {x:600, y:340, w:40, h:60};
let speed = 4;
let gameOver = false;

function jump(){
    if(player.grounded && !gameOver){
        player.dy = -player.jumpPower;
        player.grounded=false;
    }
}

document.addEventListener("keydown",(e)=>{
    if(e.code==="Space") jump();
});

canvas.addEventListener("mousedown",()=> jump());

function resetGame(){
    obstacle.x = 600;
    player.y = 300;
    player.dy = 0;
    gameOver=false;
}

function gameLoop(){
    ctx.clearRect(0,0,canvas.width,canvas.height);

    if(!gameOver){
        obstacle.x -= speed;

        if(obstacle.x < -50){
            obstacle.x = 800 + Math.random()*200;
        }

        player.dy += player.gravity;
        player.y += player.dy;

        let ground = 340;
        if(player.y + player.size >= ground){
            player.y = ground - player.size;
            player.dy = 0;
            player.grounded=true;
        }

        if(
            player.x < obstacle.x + obstacle.w &&
            player.x + player.size > obstacle.x &&
            player.y < obstacle.y + obstacle.h &&
            player.y + player.size > obstacle.y
        ){
            gameOver=true;
        }
    }

    ctx.fillStyle="blue";
    ctx.fillRect(player.x, player.y, player.size, player.size);

    ctx.fillStyle="red";
    ctx.fillRect(obstacle.x, obstacle.y, obstacle.w, obstacle.h);

    if(gameOver){
        ctx.fillStyle="white";
        ctx.font="40px Arial";
        ctx.textAlign="center";
        ctx.fillText("Game Over", canvas.width/2, canvas.height/2);
        ctx.font="20px Arial";
        ctx.fillText("Click to Restart", canvas.width/2, canvas.height/2 + 40);

        canvas.addEventListener("mousedown", resetGame);
    }

    requestAnimationFrame(gameLoop);
}

gameLoop();
</script>

</body>
</html>
