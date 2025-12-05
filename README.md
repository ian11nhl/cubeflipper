<html>
<head>
<title>Cube Flipper</title>
<style>
    body {
        background:black;
        margin:0;
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

let level = 0;
let gameRunning = false;
let gameOver = false;
let inMenu = true;
let score = 0;

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
let platforms = [];
let scrollX = 0;

// Levels with length 2000
let levels = [
    {
        length:2000,
        obstacles:[
            {x:400,y:340,w:40,h:60},
            {x:700,y:320,w:50,h:80},
            {x:1100,y:340,w:40,h:60},
            {x:1500,y:330,w:70,h:70}
        ],
        platforms:[
            {x:900,y:280,w:100,h:20}
        ]
    },
    {
        length:2000,
        obstacles:[
            {x:350,y:340,w:50,h:60},
            {x:800,y:320,w:60,h:80},
            {x:1300,y:340,w:50,h:60},
            {x:1700,y:330,w:70,h:70}
        ],
        platforms:[
            {x:600,y:260,w:120,h:20},
            {x:1300,y:240,w:140,h:20}
        ]
    },
    {
        length:2000,
        obstacles:[
            {x:300,y:340,w:60,h:60},
            {x:900,y:320,w:70,h:80},
            {x:1400,y:330,w:60,h:70},
            {x:1750,y:340,w:50,h:60}
        ],
        platforms:[
            {x:700,y:260,w:110,h:20},
            {x:1450,y:240,w:120,h:20}
        ]
    },
    {
        length:2000,
        obstacles:[
            {x:500,y:320,w:60,h:80},
            {x:1000,y:340,w:70,h:60},
            {x:1500,y:330,w:60,h:70},
            {x:1800,y:340,w:50,h:60}
        ],
        platforms:[
            {x:850,y:250,w:130,h:20},
            {x:1600,y:230,w:120,h:20}
        ]
    }
];

function loadLevel(i){
    if(i >= levels.length){
        inMenu = true;
        level = 0;
        return;
    }
    let l = levels[i];
    scrollX = 0;
    score = 0;
    obstacles = JSON.parse(JSON.stringify(l.obstacles));
    platforms = JSON.parse(JSON.stringify(l.platforms));
    gameOver = false;
    gameRunning = true;
    player.x = 50;
    player.y = 300;
    player.dy = 0;
    player.rot = 0;
}

function jump(){
    if(player.grounded && !gameOver && gameRunning){
        player.dy = -player.jumpPower;
        player.grounded = false;
        player.rotating = true;
    }
}

document.addEventListener("keydown", e=>{
    if(e.code === "Space"){
        if(inMenu){
            inMenu=false;
            loadLevel(level);
        } else jump();
    }
});

canvas.addEventListener("mousedown", ()=>{
    if(inMenu){
        inMenu=false;
        loadLevel(level);
    } else jump();
});

function gameLoop(){
    ctx.clearRect(0,0,canvas.width,canvas.height);

    if(inMenu){
        ctx.fillStyle="white";
        ctx.textAlign="center";
        ctx.font="45px Arial";
        ctx.fillText("Cube Flipper",canvas.width/2,150);
        ctx.font="25px Arial";
        ctx.fillText("Enjoy the game made by Ian",canvas.width/2,200);
        ctx.fillText("Click or Press Space to Start",canvas.width/2,260);
        requestAnimationFrame(gameLoop);
        return;
    }

    if(gameRunning){
        scrollX += 4;
        player.x += 4;

        score++;

        player.dy += player.gravity;
        player.y += player.dy;
        player.grounded = false;

        let ground = 340;
        if(player.y + player.size >= ground){
            player.y = ground - player.size;
            player.dy = 0;
            player.grounded = true;
            player.rotating = false;
            player.rot = 0;
        }

        platforms.forEach(p=>{
            let px = p.x - scrollX;
            if(player.x < px + p.w &&
               player.x + player.size > px &&
               player.y + player.size >= p.y &&
               player.y + player.size <= p.y + 10){
                player.y = p.y - player.size;
                player.dy = 0;
                player.grounded = true;
                player.rotating = false;
                player.rot = 0;
            }
        });

        obstacles.forEach(o=>{
            let ox = o.x - scrollX;
            if(player.x < ox + o.w &&
               player.x + player.size > ox &&
               player.y < o.y + o.h &&
               player.y + player.size > o.y){
                gameOver = true;
                gameRunning = false;
            }
        });

        let last = obstacles[obstacles.length-1];
        if(last && player.x > last.x + last.w + 50){
            level++;
            gameRunning=false;
            setTimeout(()=>loadLevel(level),800);
        }
    }

    ctx.fillStyle="blue";
    if(player.rotating) player.rot += 0.25;
    ctx.save();
    ctx.translate(player.x + player.size/2, player.y + player.size/2);
    ctx.rotate(player.rot);
    ctx.fillRect(-player.size/2, -player.size/2, player.size, player.size);
    ctx.restore();

    ctx.fillStyle="green";
    platforms.forEach(p=>{
        ctx.fillRect(p.x - scrollX, p.y, p.w, p.h);
    });

    ctx.fillStyle="red";
    obstacles.forEach(o=>{
        ctx.fillRect(o.x - scrollX, o.y, o.w, o.h);
    });

    ctx.fillStyle="white";
    ctx.font="20px Arial";
    ctx.textAlign="left";
    ctx.fillText("Score: "+score,10,27);

    if(gameOver){
        ctx.textAlign="center";
        ctx.font="40px Arial";
        ctx.fillText("Game Over! Tap to Restart",canvas.width/2,canvas.height/2);
    }

    requestAnimationFrame(gameLoop);
}

gameLoop();
</script>
</body>
</html>
