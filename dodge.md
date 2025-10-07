<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Dodge the Blocks</title>
<style>
  body {
    margin: 0;
    padding: 0;
    background: linear-gradient(135deg, #667eea, #764ba2);
    display: flex;
    justify-content: center;
    align-items: center;
    height: 100vh;
    font-family: Arial, sans-serif;
  }
  canvas {
    background: #121212;
    border-radius: 10px;
    box-shadow: 0 0 20px rgba(0,0,0,0.5);
  }
  #info {
    position: absolute;
    top: 10px;
    color: white;
    font-size: 20px;
    font-weight: bold;
  }
  #restartBtn {
    position: absolute;
    top: 40px;
    display: none;
    padding: 10px 20px;
    font-size: 16px;
    border: none;
    border-radius: 8px;
    cursor: pointer;
    background: #00c6ff;
    color: #121212;
  }
  #restartBtn:hover {
    background: #0072ff;
    color: white;
  }
</style>
</head>
<body>

<div id="info">Score: 0</div>
<button id="restartBtn">Restart Game</button>
<canvas id="gameCanvas" width="400" height="600"></canvas>

<script>
const canvas = document.getElementById('gameCanvas');
const ctx = canvas.getContext('2d');

let player = {x: 180, y: 550, w: 40, h: 40, speed: 6};
let obstacles = [];
let score = 0;
let gameOver = false;
let obstacleSpeed = 2;

const info = document.getElementById('info');
const restartBtn = document.getElementById('restartBtn');

document.addEventListener('keydown', (e)=>{
  if(e.key === "ArrowLeft" || e.key === "a") player.x -= player.speed;
  if(e.key === "ArrowRight" || e.key === "d") player.x += player.speed;
  // clamp inside canvas
  if(player.x < 0) player.x = 0;
  if(player.x + player.w > canvas.width) player.x = canvas.width - player.w;
});

// create obstacles periodically
function spawnObstacle(){
  const x = Math.random() * (canvas.width - 40);
  obstacles.push({x: x, y: -40, w: 40, h: 40});
}

// collision detection
function isColliding(a,b){
  return !(b.x > a.x + a.w || 
           b.x + b.w < a.x || 
           b.y > a.y + a.h ||
           b.y + b.h < a.y);
}

// main game loop
let lastTime = 0;
function gameLoop(timestamp){
  if(gameOver) return;
  
  ctx.clearRect(0,0,canvas.width,canvas.height);
  
  // draw player
  ctx.fillStyle = 'blue';
  ctx.fillRect(player.x, player.y, player.w, player.h);
  
  // spawn obstacles every 1 second
  if(timestamp - lastTime > 1000){
    spawnObstacle();
    lastTime = timestamp;
  }
  
  // move obstacles
  for(let i=obstacles.length-1; i>=0; i--){
    obstacles[i].y += obstacleSpeed;
    ctx.fillStyle = 'red';
    ctx.fillRect(obstacles[i].x, obstacles[i].y, obstacles[i].w, obstacles[i].h);
    
    if(isColliding(player, obstacles[i])){
      endGame();
    }
    
    if(obstacles[i].y > canvas.height){
      obstacles.splice(i,1);
      score += 1;
      info.innerText = "Score: " + score;
      // increase speed slightly
      if(score % 5 === 0) obstacleSpeed += 0.3;
    }
  }
  
  requestAnimationFrame(gameLoop);
}

function endGame(){
  gameOver = true;
  info.innerText += " - Game Over!";
  restartBtn.style.display = 'block';
}

restartBtn.addEventListener('click', ()=>{
  obstacles = [];
  score = 0;
  obstacleSpeed = 2;
  player.x = 180;
  gameOver = false;
  info.innerText = "Score: 0";
  restartBtn.style.display = 'none';
  requestAnimationFrame(gameLoop);
});

// start game
requestAnimationFrame(gameLoop);
</script>

</body>
</html>
