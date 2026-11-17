<!DOCTYPE html>
<html>
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1">
<title>Games</title>
<style>
*{box-sizing:border-box}
body{margin:0;background:#f4f6fb;font-family:Arial;color:#30364b}
.sidebar{position:fixed;left:0;top:0;bottom:0;width:220px;background:white;padding:25px 15px;border-right:1px solid #ddd}
.logo{font-size:28px;font-weight:bold;color:#3978e9;margin:5px 10px 35px}
.nav{display:block;width:100%;padding:15px;text-align:left;border:0;background:white;border-radius:9px;margin:5px 0;font-size:15px;cursor:pointer}
.nav:hover,.nav.active{background:#eaf1ff;color:#3978e9}
.main{margin-left:220px;padding:30px}
.top{display:flex;justify-content:space-between;align-items:center}
.profile{background:white;padding:12px 20px;border-radius:10px;font-weight:bold}
.page{display:none}
.page.active{display:block}
.card{background:white;padding:20px;border-radius:15px;border:1px solid #e5e7ef}
.grid{display:grid;grid-template-columns:repeat(auto-fill,minmax(220px,1fr));gap:20px}
.card button,.primary{background:#3978e9;color:white;border:0;padding:12px 20px;border-radius:8px;cursor:pointer}
.gameIcon{font-size:60px}
#gameScreen{display:none;position:fixed;inset:0;background:#151b25;z-index:10;overflow:auto;color:white;text-align:center;padding:15px}
#gameCanvas{background:#77b95b;border-radius:12px;max-width:100%;touch-action:none}
.gameHeader{display:flex;justify-content:space-between;max-width:500px;margin:0 auto 12px;align-items:center}
.gameHeader button{padding:10px 15px;border:0;border-radius:8px;cursor:pointer}
.controls{display:flex;justify-content:center;gap:10px;margin:12px auto;flex-wrap:wrap}
.controls button{width:65px;height:55px;font-size:23px;border:0;border-radius:10px;background:#fff;color:#222;touch-action:none}
.instructions{color:#c5d0df;font-size:13px}
@media(max-width:650px){
.sidebar{width:65px;padding:15px 5px}
.logo{font-size:0}
.logo:first-letter{font-size:25px}
.nav{font-size:0;text-align:center;padding:14px 3px}
.main{margin-left:65px;padding:15px}
}
</style>
</head>
<body>

<aside class="sidebar">
<div class="logo">🎮 Games</div>
<button class="nav active" onclick="page('home',this)">🏠 Home</button>
<button class="nav" onclick="page('library',this)">🎮 Games</button>
</aside>

<main class="main">
<div class="top">
<h1 id="heading">Home</h1>
<div class="profile">🎮 Games</div>
</div>

<section id="home" class="page active">
<div style="background:linear-gradient(110deg,#3978e9,#7859e9);padding:30px;color:white;border-radius:15px;margin:25px 0">
<h2>Welcome to Games!</h2>
<p>Play your favorite games right here.</p>
<button class="primary" onclick="page('library')">Explore Games →</button>
</div>
<h2>Featured Game</h2>
<div class="grid">
<div class="card">
<div class="gameIcon">🐔</div>
<h2>Crossy Road</h2>
<p>Cross the roads, avoid traffic, jump across rivers and beat your high score!</p>
<button onclick="openGame()">Play Now</button>
</div>
</div>
</section>

<section id="library" class="page">
<h2>Game Library</h2>
<input id="search" placeholder="Search games..." oninput="searchGames()" style="padding:14px;width:100%;border:1px solid #ddd;border-radius:10px;margin-bottom:20px">
<div class="grid" id="gameList">
<div class="card gameCard">
<div class="gameIcon">🐔</div>
<h2>Crossy Road</h2>
<p>Cross roads, dodge cars and survive as long as possible!</p>
<button onclick="openGame()">Play Game</button>
</div>
</div>
</section>
</main>

<div id="gameScreen">
<div class="gameHeader">
<button onclick="closeGame()">← Back</button>
<div>
<strong>🐔 CROSSY ROAD</strong>
<div>Score: <span id="score">0</span> | Best: <span id="best">0</span></div>
</div>
<button onclick="restart()">↻ Restart</button>
</div>

<canvas id="gameCanvas" width="480" height="620"></canvas>

<div id="gameOver" style="display:none">
<h2>💥 GAME OVER!</h2>
<p>Your score: <span id="finalScore">0</span></p>
<button class="primary" onclick="restart()">Play Again</button>
</div>

<div class="controls">
<button onclick="move(0,-1)">⬆️</button>
<button onclick="move(-1,0)">⬅️</button>
<button onclick="move(0,1)">⬇️</button>
<button onclick="move(1,0)">➡️</button>
</div>
<p class="instructions">Use WASD or arrow keys to move. Cross as many roads as possible!</p>
</div>

<script>
const canvas=document.getElementById("gameCanvas");
const ctx=canvas.getContext("2d");
const W=480,H=620;
let player;
let lanes=[];
let score=0;
let best=Number(localStorage.getItem("crossyBest")||0);
let camera=0;
let gameRunning=false;
let gameEnded=false;
let animation;
let lastTime=0;
let particles=[];

document.getElementById("best").textContent=best;

function page(id,btn){
document.querySelectorAll(".page").forEach(x=>x.classList.remove("active"));
document.getElementById(id).classList.add("active");
document.querySelectorAll(".nav").forEach(x=>x.classList.remove("active"));
if(btn)btn.classList.add("active");
document.getElementById("heading").textContent=id==="home"?"Home":"Games";
}

function searchGames(){
let value=document.getElementById("search").value.toLowerCase();
document.querySelectorAll(".gameCard").forEach(card=>{
card.style.display=card.innerText.toLowerCase().includes(value)?"block":"none";
});
}

function openGame(){
document.getElementById("gameScreen").style.display="block";
restart();
}

function closeGame(){
gameRunning=false;
cancelAnimationFrame(animation);
document.getElementById("gameScreen").style.display="none";
}

function restart(){
cancelAnimationFrame(animation);
score=0;
camera=0;
gameEnded=false;
gameRunning=true;
lanes=[];
particles=[];
player={x:240,y:500,row:0};
document.getElementById("score").textContent=0;
document.getElementById("gameOver").style.display="none";
generateWorld();
lastTime=performance.now();
animation=requestAnimationFrame(loop);
}

function generateWorld(){
lanes=[];
for(let i=0;i<45;i++){
lanes.push(makeLane(i));
}
}

function makeLane(index){
let type;
if(index%8===0)type="water";
else if(index%5===0)type="grass";
else type="road";

let lane={
index,
y:H-100-index*65,
type,
speed:(Math.random()*1.2+.7)*(Math.random()<.5?-1:1),
objects:[],
offset:0
};

if(type==="road"){
let count=2+Math.floor(Math.random()*3);
for(let j=0;j<count;j++){
lane.objects.push({
x:j*(W/count)+Math.random()*40,
w:55+Math.random()*40,
h:29,
color:["#e74c3c","#3498db","#f1c40f","#9b59b6","#ecf0f1"][Math.floor(Math.random()*5)]
});
}
}
if(type==="water"){
let count=3;
for(let j=0;j<count;j++){
lane.objects.push({
x:j*180+Math.random()*20,
w:100+Math.random()*35
});
}
}
if(type==="grass"){
for(let j=0;j<7;j++){
if(Math.random()<.5){
lane.objects.push({x:j*75+10,w:35});
}
}
}
return lane;
}

function move(dx,dy){
if(!gameRunning||gameEnded)return;
let newX=player.x+dx*60;
if(newX<15||newX>465)return;
player.x=newX;
if(dy===-1){
player.y-=65;
player.row++;
score=Math.max(score,player.row);
document.getElementById("score").textContent=score;
if(player.y<250){
camera+=65;
player.y+=65;
}
while(lanes.length<player.row+10){
lanes.push(makeLane(lanes.length));
}
}
if(dy===1){
if(player.y<H-80)player.y+=65;
}
}

document.addEventListener("keydown",e=>{
if(!gameRunning)return;
let key=e.key.toLowerCase();
if(["arrowup","arrowdown","arrowleft","arrowright"," "].includes(key))e.preventDefault();
if(key==="arrowup"||key==="w")move(0,-1);
if(key==="arrowdown"||key==="s")move(0,1);
if(key==="arrowleft"||key==="a")move(-1,0);
if(key==="arrowright"||key==="d")move(1,0);
});

function drawLane(lane){
let y=H-100-(lane.index*65-camera)+500;
if(y<-80||y>H+80)return;

if(lane.type==="road"){
ctx.fillStyle="#555b61";
ctx.fillRect(0,y-32,W,65);
ctx.fillStyle="#e5dca8";
ctx.fillRect(0,y-34,W,3);
ctx.fillRect(0,y+31,W,3);
ctx.fillStyle="#e7e7e7";
for(let x=0;x<W;x+=70)ctx.fillRect(x,y-2,38,4);

lane.objects.forEach(o=>{
ctx.fillStyle=o.color;
ctx.fillRect(o.x,y-14,o.w,29);
ctx.fillStyle="#b8d9ef";
ctx.fillRect(o.x+9,y-11,o.w*.35,9);
ctx.fillRect(o.x+o.w-18,y-11,9,9);
ctx.fillStyle="#222";
ctx.beginPath();
ctx.arc(o.x+12,y+14,6,0,Math.PI*2);
ctx.arc(o.x+o.w-12,y+14,6,0,Math.PI*2);
ctx.fill();
});
}
else if(lane.type==="water"){
ctx.fillStyle="#298fce";
ctx.fillRect(0,y-32,W,65);
ctx.fillStyle="#54c4ed";
for(let x=0;x<W;x+=65)ctx.fillRect(x,y-22,30,3);
lane.objects.forEach(o=>{
ctx.fillStyle="#92562e";
ctx.fillRect(o.x,y-17,o.w,35);
ctx.fillStyle="#bd8250";
ctx.fillRect(o.x+5,y-12,o.w-10,8);
ctx.fillStyle="#633c22";
ctx.fillRect(o.x+8,y+8,o.w-16,4);
});
}
else{
ctx.fillStyle="#75b85b";
ctx.fillRect(0,y-32,W,65);
ctx.fillStyle="#5c9e47";
ctx.fillRect(0,y+27,W,5);
lane.objects.forEach(o=>{
ctx.fillStyle="#795331";
ctx.fillRect(o.x+12,y-4,12,25);
ctx.fillStyle="#218c42";
ctx.beginPath();
ctx.arc(o.x+18,y-12,22,0,Math.PI*2);
ctx.fill();
ctx.fillStyle="#37a94f";
ctx.beginPath();
ctx.arc(o.x+8,y-18,12,0,Math.PI*2);
ctx.fill();
});
}
}

function drawPlayer(){
let x=player.x,y=player.y;
ctx.fillStyle="#00000033";
ctx.beginPath();
ctx.ellipse(x,y+23,18,7,0,0,Math.PI*2);
ctx.fill();

// Chicken body
ctx.fillStyle="#fff";
ctx.beginPath();
ctx.ellipse(x,y,17,21,0,0,Math.PI*2);
ctx.fill();

// Wings
ctx.fillStyle="#e9e9e9";
ctx.beginPath();
ctx.ellipse(x-14,y+3,7,11,-.3,0,Math.PI*2);
ctx.fill();

// Eyes
ctx.fillStyle="#222";
ctx.beginPath();
ctx.arc(x-6,y-7,2.5,0,Math.PI*2);
ctx.arc(x+6,y-7,2.5,0,Math.PI*2);
ctx.fill();

// Beak
ctx.fillStyle="#ffad22";
ctx.beginPath();
ctx.moveTo(x-6,y-1);
ctx.lineTo(x+6,y-1);
ctx.lineTo(x,y+8);
ctx.closePath();
ctx.fill();

// Comb
ctx.fillStyle="#ed4545";
ctx.beginPath();
ctx.arc(x,y-20,7,Math.PI,0);
ctx.fill();
}

function update(dt){
lanes.forEach(lane=>{
if(lane.type==="road"||lane.type==="water"){
lane.objects.forEach(o=>{
o.x+=lane.speed*dt*.06;
if(lane.speed>0&&o.x>W+o.w)o.x=-o.w;
if(lane.speed<0&&o.x<-o.w)o.x=W+o.w;
});
}
});

let rowLane=lanes[player.row];
if(!rowLane)return;

let y=H-100-(rowLane.index*65-camera)+500;

if(rowLane.type==="road"){
for(let o of rowLane.objects){
if(player.x+13>o.x&&player.x-13<o.x+o.w&&player.y+15>y-16&&player.y-15<y+16){
endGame();
return;
}
}
}

if(rowLane.type==="water"){
let onLog=rowLane.objects.some(o=>player.x+14>o.x&&player.x-14<o.x+o.w);
if(!onLog){
endGame();
return;
}
player.x+=rowLane.speed*dt*.06;
if(player.x<0||player.x>W)endGame();
}

if(rowLane.type==="grass"){
for(let o of rowLane.objects){
if(player.x+12>o.x&&player.x-12<o.x+o.w){
endGame();
return;
}
}
}
}

function endGame(){
gameEnded=true;
gameRunning=false;
best=Math.max(best,score);
localStorage.setItem("crossyBest",best);
document.getElementById("best").textContent=best;
document.getElementById("finalScore").textContent=score;
document.getElementById("gameOver").style.display="block";
}

function draw(){
ctx.clearRect(0,0,W,H);
ctx.fillStyle="#75b85b";
ctx.fillRect(0,0,W,H);

for(let lane of lanes)drawLane(lane);

drawPlayer();

ctx.fillStyle="#ffffff";
ctx.font="bold 15px Arial";
ctx.fillText("SCORE: "+score,15,25);
}

function loop(time){
if(!gameRunning)return;
let dt=Math.min(time-lastTime,35);
lastTime=time;
update(dt);
draw();
if(gameRunning)animation=requestAnimationFrame(loop);
}

</script>
</body>
</html>
