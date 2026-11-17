<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Minecraft Bedrock Style</title>

<style>
* { box-sizing:border-box; }

body {
    margin:0;
    overflow:hidden;
    background:#87CEEB;
    font-family:Arial,sans-serif;
    color:white;
}

#game {
    position:fixed;
    inset:0;
}

#crosshair {
    position:fixed;
    top:50%;
    left:50%;
    transform:translate(-50%,-50%);
    font-size:26px;
    text-shadow:1px 1px 3px black;
    pointer-events:none;
}

#hotbar {
    position:fixed;
    bottom:20px;
    left:50%;
    transform:translateX(-50%);
    display:flex;
    gap:4px;
    background:#2229;
    padding:6px;
    border:2px solid #ffffff55;
}

.slot {
    width:48px;
    height:48px;
    border:2px solid #777;
    background:#333b;
    display:flex;
    align-items:center;
    justify-content:center;
    font-size:25px;
    position:relative;
}

.slot.selected {
    border:3px solid white;
    transform:scale(1.12);
}

#inventory {
    display:none;
    position:fixed;
    top:50%;
    left:50%;
    transform:translate(-50%,-50%);
    width:440px;
    padding:20px;
    background:#c6c6c6;
    color:#222;
    border:5px solid #555;
    z-index:10;
}

#inventory h2 { text-align:center; }

#items {
    display:grid;
    grid-template-columns:repeat(5,1fr);
    gap:7px;
}

.item {
    height:55px;
    background:#777;
    border:2px solid #333;
    display:flex;
    align-items:center;
    justify-content:center;
    font-size:25px;
    cursor:pointer;
}

#close {
    width:100%;
    padding:10px;
    margin-top:15px;
}

#start {
    position:fixed;
    inset:0;
    background:linear-gradient(#0008,#000b);
    z-index:20;
    display:flex;
    align-items:center;
    justify-content:center;
    flex-direction:column;
    text-align:center;
}

#start h1 {
    font-size:55px;
    text-shadow:4px 4px #333;
    margin-bottom:5px;
}

#start button {
    padding:15px 45px;
    background:#65a832;
    color:white;
    font-size:22px;
    border:3px solid #333;
    cursor:pointer;
}

#stats {
    position:fixed;
    top:12px;
    left:12px;
    font-size:17px;
    text-shadow:2px 2px 3px black;
}

#help {
    position:fixed;
    right:12px;
    top:12px;
    text-align:right;
    font-size:13px;
    text-shadow:2px 2px 3px black;
}

#mode {
    position:fixed;
    top:65px;
    left:12px;
    text-shadow:2px 2px 3px black;
}
</style>
</head>

<body>

<div id="game"></div>

<div id="crosshair">+</div>

<div id="stats">
    ❤️ <span id="health">20</span><br>
    🍗 <span id="hunger">20</span>
</div>

<div id="mode">Survival Mode</div>

<div id="help">
    WASD: Move<br>
    SPACE: Jump<br>
    SHIFT: Run<br>
    Left Click: Break block<br>
    Right Click: Place block<br>
    E: Inventory<br>
    1-9: Select block<br>
    M: Change mode
</div>

<div id="hotbar"></div>

<div id="inventory">
    <h2>Inventory</h2>
    <div id="items"></div>
    <button id="close">Close</button>
</div>

<div id="start">
    <h1>MINECRAFT</h1>
    <h3>BEDROCK-STYLE BLOCK WORLD</h3>
    <p>Explore, build, and survive.</p>
    <button id="play">PLAY</button>
</div>

<script type="module">

import * as THREE from
"https://cdn.jsdelivr.net/npm/three@0.160.0/build/three.module.js";

/* ---------------- SETUP ---------------- */

const scene = new THREE.Scene();
scene.background = new THREE.Color(0x87ceeb);
scene.fog = new THREE.Fog(0x87ceeb,35,90);

const camera = new THREE.PerspectiveCamera(
    75, window.innerWidth/window.innerHeight,0.1,150
);

const renderer = new THREE.WebGLRenderer({
    antialias:false
});

renderer.setSize(window.innerWidth,window.innerHeight);
renderer.setPixelRatio(Math.min(devicePixelRatio,2));
renderer.shadowMap.enabled=true;
renderer.shadowMap.type=THREE.PCFSoftShadowMap;

document.getElementById("game").appendChild(renderer.domElement);

/* ---------------- LIGHTING ---------------- */

const sun = new THREE.DirectionalLight(0xffffff,1.3);
sun.position.set(30,60,20);
sun.castShadow=true;
scene.add(sun);

scene.add(new THREE.AmbientLight(0xffffff,0.55));

/* ---------------- BLOCKS ---------------- */

const materials = {
    grass: new THREE.MeshLambertMaterial({color:0x59a832}),
    dirt: new THREE.MeshLambertMaterial({color:0x79502e}),
    stone: new THREE.MeshLambertMaterial({color:0x858585}),
    wood: new THREE.MeshLambertMaterial({color:0x80512c}),
    leaves: new THREE.MeshLambertMaterial({color:0x287b27}),
    sand: new THREE.MeshLambertMaterial({color:0xd8c27b}),
    water: new THREE.MeshLambertMaterial({
        color:0x287dd1,
        transparent:true,
        opacity:0.75
    }),
    bedrock: new THREE.MeshLambertMaterial({color:0x333333}),
    coal: new THREE.MeshLambertMaterial({color:0x252525}),
    iron: new THREE.MeshLambertMaterial({color:0xb6a99b}),
    gold: new THREE.MeshLambertMaterial({color:0xe8c52b}),
    diamond: new THREE.MeshLambertMaterial({color:0x26e1d2})
};

const icons = {
    grass:"🟩",
    dirt:"🟫",
    stone:"⬜",
    wood:"🪵",
    leaves:"🌳",
    sand:"🟨",
    water:"🟦",
    coal:"⚫",
    iron:"◻️",
    gold:"🟡",
    diamond:"💎"
};

const geometry = new THREE.BoxGeometry(1,1,1);

const world = new Map();
const meshes = new Map();

function key(x,y,z) {
    return `${x},${y},${z}`;
}

function getBlock(x,y,z) {
    return world.get(key(x,y,z));
}

function addBlock(x,y,z,type) {
    const k=key(x,y,z);

    if(world.has(k)) return;

    world.set(k,type);

    const mesh=new THREE.Mesh(
        geometry,
        materials[type] || materials.dirt
    );

    mesh.position.set(x,y,z);
    mesh.castShadow=true;
    mesh.receiveShadow=true;

    mesh.userData={x,y,z,type};

    scene.add(mesh);
    meshes.set(k,mesh);
}

function removeBlock(x,y,z) {
    const k=key(x,y,z);
    const mesh=meshes.get(k);

    if(!mesh) return;

    scene.remove(mesh);
    meshes.delete(k);
    world.delete(k);
}

/* ---------------- TERRAIN ---------------- */

function terrainHeight(x,z) {
    return Math.floor(
        4+
        Math.sin(x*0.23)*2+
        Math.cos(z*0.19)*2+
        Math.sin((x+z)*0.12)*1.5
    );
}

function generateWorld() {

    for(let x=-20;x<=20;x++) {
        for(let z=-20;z<=20;z++) {

            const h=terrainHeight(x,z);

            for(let y=-2;y<=h;y++) {

                let type="stone";

                if(y===-2) type="bedrock";
                else if(y===h) type="grass";
                else if(y>h-4) type="dirt";

                if(y===h-3 && Math.random()<0.035)
                    type="coal";

                if(y===h-2 && Math.random()<0.015)
                    type="iron";

                if(y<h-4 && Math.random()<0.003)
                    type="gold";

                addBlock(x,y,z,type);
            }

            if(h<3) {
                addBlock(x,3,z,"water");
            }
        }
    }

    generateTrees();
}

/* ---------------- TREES ---------------- */

function generateTrees() {

    for(let i=0;i<45;i++) {

        const x=Math.floor(Math.random()*36)-18;
        const z=Math.floor(Math.random()*36)-18;

        if(Math.abs(x)<4 && Math.abs(z)<4) continue;

        const y=terrainHeight(x,z);

        for(let t=1;t<=4;t++)
            addBlock(x,y+t,z,"wood");

        for(let dx=-2;dx<=2;dx++) {
            for(let dz=-2;dz<=2;dz++) {
                for(let dy=3;dy<=5;dy++) {

                    if(Math.abs(dx)+Math.abs(dz)>3) continue;

                    if(Math.random()>0.12)
                        addBlock(x+dx,y+dy,z+dz,"leaves");
                }
            }
        }
    }
}

generateWorld();

/* ---------------- PLAYER ---------------- */

const player={
    x:0,
    y:12,
    z:0,
    velocityY:0,
    health:20,
    hunger:20,
    grounded:false,
    speed:5
};

camera.position.set(player.x,player.y,player.z);

let yaw=0;
let pitch=0;
let locked=false;
let playing=false;

const keys={};

const playerHeight=1.8;
const playerRadius=0.3;

function solidAt(x,y,z) {
    const block=getBlock(
        Math.floor(x),
        Math.floor(y),
        Math.floor(z)
    );

    return block && block!=="water";
}

function collides(x,y,z) {

    const points=[
        [-playerRadius,0,-playerRadius],
        [playerRadius,0,-playerRadius],
        [-playerRadius,0,playerRadius],
        [playerRadius,0,playerRadius],
        [-playerRadius,1,-playerRadius],
        [playerRadius,1,playerRadius]
    ];

    return points.some(p=>
        solidAt(x+p[0],y+p[1],z+p[2])
    );
}

/* ---------------- CONTROLS ---------------- */

document.addEventListener("keydown",e=>{

    keys[e.code]=true;

    if(e.code==="KeyE") toggleInventory();

    if(e.code==="KeyM") toggleMode();

    if(e.code.startsWith("Digit")) {
        const n=Number(e.code.replace("Digit",""))-1;
        if(n>=0 && n<9) selectSlot(n);
    }

});

document.addEventListener("keyup",e=>{
    keys[e.code]=false;
});

document.addEventListener("mousemove",e=>{

    if(!locked) return;

    yaw-=e.movementX*0.002;
    pitch-=e.movementY*0.002;

    pitch=Math.max(-1.5,Math.min(1.5,pitch));

    camera.rotation.order="YXZ";
    camera.rotation.y=yaw;
    camera.rotation.x=pitch;
});

renderer.domElement.addEventListener("click",()=>{

    if(!playing) return;

    if(!locked) {
        renderer.domElement.requestPointerLock();
    }
});

document.addEventListener("pointerlockchange",()=>{
    locked=document.pointerLockElement===renderer.domElement;
});

/* ---------------- INVENTORY ---------------- */

const blockTypes=[
    "grass","dirt","stone","wood","leaves",
    "sand","water","coal","iron","gold","diamond"
];

let selected=0;
let creative=false;

const inventory={};

blockTypes.forEach(b=>inventory[b]=20);

function selectSlot(n) {
    selected=n;
    updateHotbar();
}

function updateHotbar() {

    const bar=document.getElementById("hotbar");
    bar.innerHTML="";

    for(let i=0;i<9;i++) {

        const type=blockTypes[i];

        const div=document.createElement("div");
        div.className="slot"+(selected===i?" selected":"");

        div.innerHTML=
            `<span>${icons[type]}</span>`;

        bar.appendChild(div);
    }
}

function toggleInventory() {

    const panel=document.getElementById("inventory");

    if(panel.style.display==="block") {
        panel.style.display="none";
        if(playing) renderer.domElement.requestPointerLock();
        return;
    }

    document.exitPointerLock();
    panel.style.display="block";

    const items=document.getElementById("items");
    items.innerHTML="";

    blockTypes.forEach((type,i)=>{

        const div=document.createElement("div");
        div.className="item";

        div.innerHTML=
            `${icons[type]}<small>${inventory[type]}</small>`;

        div.onclick=()=>{
            selectSlot(i);
            panel.style.display="none";
        };

        items.appendChild(div);
    });
}

document.getElementById("close").onclick=()=>{
    document.getElementById("inventory").style.display="none";
};

/* ---------------- BREAKING AND PLACING ---------------- */

const raycaster=new THREE.Raycaster();
raycaster.far=6;

function targetBlock() {

    raycaster.setFromCamera(
        new THREE.Vector2(0,0),
        camera
    );

    const hits=raycaster.intersectObjects(
        Array.from(meshes.values())
    );

    return hits.length?hits[0]:null;
}

document.addEventListener("mousedown",e=>{

    if(!playing || !locked) return;

    const hit=targetBlock();

    if(!hit) return;

    const b=hit.object.userData;

    if(e.button===0) {

        if(b.type==="bedrock") return;

        removeBlock(b.x,b.y,b.z);

        if(!creative) {
            inventory[b.type]=(inventory[b.type]||0)+1;
        }

    }

    if(e.button===2) {

        const normal=hit.face.normal;

        const x=b.x+Math.round(normal.x);
        const y=b.y+Math.round(normal.y);
        const z=b.z+Math.round(normal.z);

        const type=blockTypes[selected];

        if(!creative && inventory[type]<=0) return;

        if(!getBlock(x,y,z)) {

            if(
                x<player.x+0.6 && x>player.x-0.6 &&
                z<player.z+0.6 && z>player.z-0.6 &&
                y>=player.y-0.2 && y<=player.y+2
            ) return;

            addBlock(x,y,z,type);

            if(!creative)
                inventory[type]--;
        }
    }
});

document.addEventListener("contextmenu",e=>e.preventDefault());

/* ---------------- GAME MODE ---------------- */

function toggleMode() {

    creative=!creative;

    document.getElementById("mode").textContent=
        creative?"Creative Mode":"Survival Mode";

    if(creative) {
        player.velocityY=0;
        player.health=20;
    }
}

/* ---------------- MOB SYSTEM ---------------- */

const mobs=[];

function spawnMob() {

    const material=new THREE.MeshLambertMaterial({
        color:0x427b35
    });

    const mob=new THREE.Mesh(
        new THREE.BoxGeometry(0.8,1.8,0.6),
        material
    );

    const x=Math.random()*30-15;
    const z=Math.random()*30-15;
    const y=terrainHeight(x,z)+1;

    mob.position.set(x,y,z);
    scene.add(mob);

    mobs.push({
        mesh:mob,
        speed:0.7
    });
}

for(let i=0;i<5;i++) spawnMob();

function updateMobs(dt) {

    if(creative) return;

    for(const mob of mobs) {

        const m=mob.mesh;

        const dx=player.x-m.position.x;
        const dz=player.z-m.position.z;

        const distance=Math.sqrt(dx*dx+dz*dz);

        if(distance<13 && distance>1.2) {

            m.position.x+=(dx/distance)*mob.speed*dt;
            m.position.z+=(dz/distance)*mob.speed*dt;

            m.position.y=terrainHeight(
                Math.round(m.position.x),
                Math.round(m.position.z)
            )+1;
        }

        if(distance<1.3) {
            player.health=Math.max(0,player.health-0.3);
        }
    }
}

/* ---------------- MOVEMENT ---------------- */

function updatePlayer(dt) {

    const forward=new THREE.Vector3(
        -Math.sin(yaw),0,-Math.cos(yaw)
    );

    const right=new THREE.Vector3(
        Math.cos(yaw),0,-Math.sin(yaw)
    );

    let move=new THREE.Vector3();

    if(keys.KeyW) move.add(forward);
    if(keys.KeyS) move.sub(forward);
    if(keys.KeyD) move.add(right);
    if(keys.KeyA) move.sub(right);

    if(move.length()>0) move.normalize();

    let speed=player.speed;

    if(keys.ShiftLeft) speed=8;

    const nx=player.x+move.x*speed*dt;
    const nz=player.z+move.z*speed*dt;

    if(!collides(nx,player.y,player.z))
        player.x=nx;

    if(!collides(player.x,player.y,nz))
        player.z=nz;

    if(creative) {

        if(keys.Space) player.y+=speed*dt;
        if(keys.ControlLeft) player.y-=speed*dt;

    } else {

        if(keys.Space && player.grounded) {
            player.velocityY=8;
            player.grounded=false;
        }

        player.velocityY-=20*dt;

        let ny=player.y+player.velocityY*dt;

        if(collides(player.x,ny,player.z)) {
            if(player.velocityY<0)
                player.grounded=true;

            player.velocityY=0;
        } else {
            player.y=ny;
            player.grounded=false;
        }

        if(player.y<-10) {
            player.x=0;
            player.z=0;
            player.y=12;
            player.health=Math.max(1,player.health-5);
        }
    }

    camera.position.set(
        player.x,
        player.y+1.5,
        player.z
    );

    document.getElementById("health").textContent=
        Math.ceil(player.health);

    document.getElementById("hunger").textContent=
        Math.floor(player.hunger);
}

/* ---------------- DAY AND NIGHT ---------------- */

let time=0;

function updateDayNight(dt) {

    time+=dt*0.025;

    const sunlight=(Math.sin(time)+1)/2;

    sun.intensity=0.15+sunlight*1.2;

    const sky=new THREE.Color();

    sky.setRGB(
        0.025+sunlight*0.5,
        0.035+sunlight*0.75,
        0.12+sunlight*0.85
    );

    scene.background.copy(sky);
    scene.fog.color.copy(sky);

    sun.position.set(
        Math.cos(time)*50,
        15+sunlight*50,
        Math.sin(time)*50
    );
}

/* ---------------- START ---------------- */

document.getElementById("play").onclick=()=>{

    document.getElementById("start").style.display="none";

    playing=true;

    renderer.domElement.requestPointerLock();
};

updateHotbar();

/* ---------------- MAIN LOOP ---------------- */

let last=performance.now();

function animate(now) {

    requestAnimationFrame(animate);

    const dt=Math.min((now-last)/1000,0.05);
    last=now;

    if(playing && locked) {

        updatePlayer(dt);
        updateMobs(dt);
        updateDayNight(dt);
    }

    renderer.render(scene,camera);
}

animate(performance.now());

/* ---------------- RESIZE ---------------- */

window.addEventListener("resize",()=>{

    camera.aspect=window.innerWidth/window.innerHeight;
    camera.updateProjectionMatrix();

    renderer.setSize(
        window.innerWidth,
        window.innerHeight
    );
});

</script>
</body>
</html>
