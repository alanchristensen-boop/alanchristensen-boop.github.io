<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Games</title>

<style>
* {
    box-sizing: border-box;
}

body {
    margin: 0;
    background: #f5f6fa;
    font-family: Arial, sans-serif;
}

header {
    height: 65px;
    background: white;
    border-bottom: 1px solid #ddd;
    display: flex;
    align-items: center;
    justify-content: space-between;
    padding: 0 25px;
}

.logo {
    font-size: 25px;
    font-weight: bold;
    color: #333;
}

header button {
    background: #3978f6;
    color: white;
    border: none;
    border-radius: 8px;
    padding: 12px 22px;
    font-size: 15px;
    cursor: pointer;
}

.container {
    display: flex;
    min-height: calc(100vh - 65px);
}

.sidebar {
    width: 210px;
    background: white;
    border-right: 1px solid #ddd;
    padding: 20px 12px;
}

.sidebar button {
    width: 100%;
    border: none;
    background: transparent;
    text-align: left;
    padding: 15px;
    font-size: 16px;
    border-radius: 8px;
    cursor: pointer;
}

.sidebar button:hover {
    background: #eeeeee;
}

main {
    flex: 1;
    padding: 30px;
}

h1 {
    color: #333;
}

.card {
    background: white;
    border-radius: 12px;
    padding: 25px;
    max-width: 500px;
    box-shadow: 0 2px 10px #00000012;
}

.card button {
    background: #3978f6;
    color: white;
    border: none;
    padding: 12px 22px;
    border-radius: 7px;
    cursor: pointer;
    font-size: 16px;
}

#gameFrame {
    display: none;
    width: 100%;
    height: calc(100vh - 110px);
    border: none;
    background: white;
    border-radius: 10px;
}

#gamePage {
    display: none;
}

#homePage {
    display: block;
}

#gamesPage {
    display: none;
}

.description {
    color: #777;
    line-height: 1.5;
}
</style>
</head>

<body>

<header>
    <div class="logo">Games</div>
    <button onclick="openOriginal()">Games</button>
</header>

<div class="container">

    <aside class="sidebar">
        <button onclick="showHome()">🏠 Home</button>
        <button onclick="showGames()">🎮 Games</button>
    </aside>

    <main>

        <section id="homePage">
            <h1>Welcome to Games</h1>
            <p>Play games from the original Games website.</p>

            <div class="card">
                <h2>🎮 Games Library</h2>
                <p class="description">
                    Open the games library and explore the available games.
                </p>
                <button onclick="showGames()">Browse Games</button>
            </div>
        </section>

        <section id="gamesPage">
            <h1>Games Library</h1>

            <div class="card">
                <h2>🎮 Original Games Library</h2>
                <p class="description">
                    This opens the original website you provided.
                </p>
                <button onclick="openOriginal()">Open Games</button>
            </div>
        </section>

        <section id="gamePage">
            <h1>Games</h1>

            <button onclick="openOriginal()">
                Open Original Website
            </button>

            <iframe
                id="gameFrame"
                src="https://sites.google.com/sacredheartgs.org/snek/home"
                allowfullscreen>
            </iframe>
        </section>

    </main>
</div>

<script>
const originalURL =
"https://sites.google.com/sacredheartgs.org/snek/home";

function hidePages() {
    document.getElementById("homePage").style.display = "none";
    document.getElementById("gamesPage").style.display = "none";
    document.getElementById("gamePage").style.display = "none";
}

function showHome() {
    hidePages();
    document.getElementById("homePage").style.display = "block";
}

function showGames() {
    hidePages();
    document.getElementById("gamesPage").style.display = "block";
}

function openOriginal() {
    window.open(originalURL, "_blank");
}
</script>

</body>
</html>
