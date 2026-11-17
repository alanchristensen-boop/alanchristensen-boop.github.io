<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Games</title>
<style>
*{box-sizing:border-box}
body{margin:0;background:#f5f6fa;font-family:Arial,sans-serif;color:#30364b}
header{height:70px;background:white;border-bottom:1px solid #ddd;display:flex;align-items:center;justify-content:space-between;padding:0 30px}
.logo{font-size:27px;font-weight:900;color:#3978e9}
header button{background:#3978e9;color:white;border:0;padding:12px 22px;border-radius:9px;font-weight:bold;cursor:pointer}
.layout{display:flex;min-height:calc(100vh - 70px)}
aside{width:220px;background:white;padding:20px 12px;border-right:1px solid #ddd}
.nav{width:100%;padding:15px;text-align:left;border:0;background:white;border-radius:9px;margin:4px 0;font-weight:bold;cursor:pointer}
.nav:hover,.nav.active{background:#eaf1ff;color:#3978e9}
main{flex:1;padding:30px}
.page{display:none}
.page.active{display:block}
.hero{background:linear-gradient(110deg,#3978e9,#7859e9);color:white;padding:30px;border-radius:16px;margin-bottom:30px}
.hero h1{margin-top:0}
.card{background:white;padding:20px;border-radius:15px;border:1px solid #e5e8ef;max-width:350px}
.art{height:150px;background:linear-gradient(#9ddafa 45%,#78bd61 45%);border-radius:12px;display:grid;place-items:center;font-size:65px;margin-bottom:15px}
.card button,.primary{background:#3978e9;color:white;border:0;padding:12px 20px;border-radius:9px;font-weight:bold;cursor:pointer}
.card p{color:#7c8496;line-height:1.5}
#player{display:none}
.playerHead{display:flex;justify-content:space-between;align-items:center;margin-bottom:15px;gap:10px}
.back{background:white;border:1px solid #ddd;padding:12px 17px;border-radius:9px;cursor:pointer;font-weight:bold}
.gamebox{height:75vh;min-height:450px;background:#111;border-radius:13px;overflow:hidden;position:relative}
iframe{width:100%;height:100%;border:0}
.fallback{display:none;position:absolute;inset:0;background:white;text-align:center;align-content:center;padding:25px}
.fallback p{color:#777;line-height:1.5}
.fallback a{display:inline-block;background:#3978e9;color:white;text-decoration:none;padding:14px 20px;border-radius:9px;font-weight:bold}
@media(max-width:650px){
aside{width:65px;padding:12px 4px}
.nav{font-size:0;text-align:center;padding:15px 3px}
.nav:first-letter{font-size:19px}
main{padding:15px}
header{padding:0 15px}
.gamebox{height:68vh;min-height:380px}
}
</style>
</head>
<body>

<header>
<div class="logo">🎮 Games</div>
<button onclick="showGames()">Games</button>
</header>

<div class="layout">
<aside>
<button class="nav active" onclick="showHome()">🏠 Home</button>
<button class="nav" onclick="showGames()">🎮 Games</button>
</aside>

<main>

<section id="home" class="page active">
<div class="hero">
<h1>Welcome to Games!</h1>
<p>Play your favorite games right here.</p>
<button class="primary" onclick="showGames()">Explore Games →</button>
</div>

<h2>Featured Game</h2>

<div class="card">
<div class="art">🐔🚗</div>
<h3>Crossy Road</h3>
<p>Cross roads, dodge traffic, jump across rivers and beat your high score!</p>
<button onclick="launchGame()">Play Now</button>
</div>
</section>

<section id="games" class="page">
<h1>Games Library</h1>
<p>Choose a game to play.</p>

<input id="search" placeholder="Search games..." oninput="searchGames()" style="width:100%;padding:14px;border:1px solid #ddd;border-radius:9px;margin-bottom:20px">

<div class="card gameCard">
<div class="art">🐔🚗</div>
<h3>Crossy Road</h3>
<p>Play the original Crossy Road browser game.</p>
<button onclick="launchGame()">Play Game</button>
</div>
</section>

<section id="player">
<div class="playerHead">
<div>
<h2>🐔 Crossy Road</h2>
<p>Original browser game</p>
</div>
<button class="back" onclick="showGames()">← Back</button>
</div>

<div class="gamebox">
<iframe id="gameFrame"
src="about:blank"
allow="fullscreen; autoplay; gamepad"
allowfullscreen>
</iframe>

<div class="fallback" id="fallback">
<h2>Game cannot be embedded</h2>
<p>The original game host may prevent embedding on other websites.</p>
<a href="https://poki.com/en/g/crossy-road" target="_blank" rel="noopener">
Play Original Crossy Road
</a>
</div>
</div>
</section>

</main>
</div>

<script>
const gameURL="https://poki.com/en/g/crossy-road";

function showPage(id){
document.querySelectorAll(".page").forEach(p=>p.classList.remove("active"));
document.getElementById("player").style.display="none";
document.getElementById(id).classList.add("active");
document.querySelectorAll(".nav").forEach(n=>n.classList.remove("active"));
document.querySelectorAll(".nav")[id==="home"?0:1].classList.add("active");
}

function showHome(){
document.getElementById("gameFrame").src="about:blank";
showPage("home");
}

function showGames(){
document.getElementById("gameFrame").src="about:blank";
showPage("games");
}

function launchGame(){
document.querySelectorAll(".page").forEach(p=>p.classList.remove("active"));
document.getElementById("player").style.display="block";
document.getElementById("fallback").style.display="none";
document.getElementById("gameFrame").src=gameURL;

setTimeout(()=>{
document.getElementById("fallback").style.display="grid";
},9000);
}

function searchGames(){
let search=document.getElementById("search").value.toLowerCase();
document.querySelectorAll(".gameCard").forEach(card=>{
card.style.display=card.innerText.toLowerCase().includes(search)?"block":"none";
});
}
</script>

</body>
</html>
