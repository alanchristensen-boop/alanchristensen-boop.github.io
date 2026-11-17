<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Minecraft 1.8</title>

<style>
* {
    box-sizing: border-box;
}

html, body {
    margin: 0;
    width: 100%;
    height: 100%;
    overflow: hidden;
    background: #111;
    font-family: Arial, sans-serif;
}

#game {
    width: 100%;
    height: 100%;
    border: none;
    display: block;
}

#loading {
    position: fixed;
    inset: 0;
    background: #222;
    color: white;
    display: flex;
    justify-content: center;
    align-items: center;
    flex-direction: column;
    z-index: 5;
}

#loading h1 {
    font-size: 42px;
    text-shadow: 3px 3px #555;
}

#loading p {
    color: #ccc;
}

#open {
    background: #65a832;
    color: white;
    border: 2px solid #333;
    padding: 12px 30px;
    font-size: 18px;
    cursor: pointer;
}
</style>
</head>

<body>

<div id="loading">
    <h1>MINECRAFT 1.8</h1>
    <p>Loading Eaglercraft...</p>
    <button id="open" onclick="launch()">PLAY</button>
</div>

<iframe
    id="game"
    src="./eaglercraft.html"
    allow="fullscreen; autoplay; gamepad"
    allowfullscreen>
</iframe>

<script>
const frame = document.getElementById("game");

function launch() {
    document.getElementById("loading").style.display = "none";
    frame.focus();
}

frame.addEventListener("load", () => {
    // The client has loaded.
    document.getElementById("loading").style.display = "none";
});
</script>

</body>
</html>
