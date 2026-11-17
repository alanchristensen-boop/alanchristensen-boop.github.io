<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1">
<title>BlockWorld 3D</title>

<style>
*{box-sizing:border-box}
body{margin:0;overflow:hidden;background:#87ceeb;font-family:Arial}
canvas{display:block}
#overlay{
 position:fixed;inset:0;display:flex;align-items:center;
 justify-content:center;flex-direction:column;
 background:linear-gradient(#0008,#000b);
 color:white;z-index:10;text-align:center
}
h1{font-size:65px;text-shadow:4px 5px #333;margin:0 0 10px}
button{
 padding:14px 45px;background:#64a936;color:white;
 border:3px solid #263b19;font-size:21px;cursor:pointer
}
#crosshair{
 position:fixed;left:50%;top:50%;transform:translate(-50%,-50%);
 color:white;font-size:26px;text-shadow:1px 1px 3px black;
 pointer-events:none
}
#hotbar{
 position:fixed;bottom:18px;left:50%;transform:translateX(-50%);
 display:flex;gap:4px;padding:6px;background:#2229
}
.slot{
 width:50px;height:50px;background:#555b;border:3px solid #999;
 display:flex;align-items:center;justify-content:center;
 font-size:26px;position:relative;color:white
}
.slot.selected{border:3px solid white;transform:scale(1.1)}
.count{position:absolute;right:3px;bottom:1px;font-size:12px}
#stats{
 position:fixed;top:15px;left:15px;color:white;
 text-shadow:2px 2px 3px black;font-size:20px
}
#help{
 position:fixed;right:12px;top:12px;color:white;
 text-shadow:2px 2px 3px black;text-align:right;font-size:13px
}
#inventory{
 display:none;position:fixed;top:50%;left:50%;
 transform:translate(-50%,-50%);width:420px;
 background:#aaa;border:5px solid #555;padding:20px;z-index:5
}
#inventory h2{text-align:center}
#items{display:grid;grid-template-columns:repeat(5,1fr);gap:8px}
.item{
 height:55px;background:#777;border:2px solid #444;
 display:flex;align-items:center;justify-content:center;
 font-size:25px;cursor:pointer
}
#inventory button{width:100%;margin-top:15px}
</style>
</head>

<body>

<div id="overlay">
<h1>BLOCKWORLD</h1>
<p>A high-detail block-building adventure</p>
<button id="play">PLAY</button>
<p>WASD to move • Mouse to look • Left click to mine</p>
</div>

<div id="crosshair">+</div>

<div id="stats">
❤️ <span id="health">20</span><br>
🍗 <span id="hunger">20</span>
</div>

<div id="help">
WASD — Move<br>
SPACE — Jump<br>
SHIFT — Sprint<br>
LEFT CLICK — Mine<br>
RIGHT CLICK — Place<br>
E — Inventory<br>
1–9 — Hotbar
</div>

<div id="hotbar"></div>

<div id="inventory">
<h2>Inventory</h2>
<div id="items"></div>
<button id="close">CLOSE</button>
</div>

<script type="module">

import * as THREE from
"https://cdn.jsdelivr.net/npm/three@0.160.0/build/three.module.js";

/* =================================
   RENDERING ENGINE
================================= */

const scene=new THREE.Scene();
scene.background=new THREE.Color(0x8bcdf2);
scene.fog=new THREE.Fog(0x8bcdf2,35,105);

const camera=new THREE.PerspectiveCamera(
 75,innerWidth/innerHeight,0.1,200
);

const renderer=new THREE.WebGLRenderer({
 antialias:true,
 powerPreference:"high-performance"
});

renderer.setSize(innerWidth,innerHeight);
renderer.setPixelRatio(Math.min(devicePixelRatio,2));
renderer.shadowMap.enabled=true;
renderer.shadowMap.type=THREE.PCFSoftShadowMap;
renderer.outputColorSpace=THREE.SRGBColorSpace;
renderer.toneMapping=THREE.ACESFilmicToneMapping;
renderer.toneMappingExposure=1.15;

document.body.appendChild(renderer.domElement);

/* =================================
   LIGHTING
================================= */

const ambient=new THREE.HemisphereLight(
 0xc9e8ff,0x55442b,1.2
);
scene.add(ambient);

const sun=new THREE.DirectionalLight(0xfff1d6,3);
sun.position.set(35,55,20);
sun.castShadow=true;
sun.shadow.mapSize.set(2048,2048);
sun.shadow.camera.left=-55;
sun.shadow.camera.right=55;
sun.shadow.camera.top=55;
sun.shadow.camera.bottom=-55;
sun.shadow.bias=-0.0002;
scene.add(sun);

/* =================================
   PROCEDURAL TEXTURES
================================= */

function texture(base,seed=1){
 const c=document.createElement("canvas");
 c.width=c.height=64;
 const ctx=c.getContext("2d");

 ctx.fillStyle=base;
 ctx.fillRect(0,0,64,64);

 let s=seed;
 function random(){
  s=(s*1664525+1013904223)>>>0;
  return s/4294967296;
 }

 for(let i=0;i<500;i++){
  const x=random()*64;
  const y=random()*64;
  const size=1+random()*5;
  const shade=random()>.5?255:0;
  ctx.fillStyle=`rgba(${shade},${shade},${shade},${random()*.16})`;
  ctx.fillRect(x,y,size,size);
 }

 const t=new THREE.CanvasTexture(c);
 t.magFilter=THREE.NearestFilter;
 t.minFilter=THREE.NearestMipmapNearestFilter;
 t.colorSpace=THREE.SRGBColorSpace;
 return t;
}

const mats={
 grassTop:new THREE.MeshStandardMaterial({
  map:texture("#65a83c",3),roughness:1
 }),
 grassSide:new THREE.MeshStandardMaterial({
  map:texture("#668c35",4),roughness:1
 }),
 dirt:new THREE.MeshStandardMaterial({
  map:texture("#80532e",5),roughness:1
 }),
 stone:new THREE.MeshStandardMaterial({
  map:texture("#858585",6),roughness:.95
 }),
 wood:new THREE.MeshStandardMaterial({
  map:texture("#79502c",7),roughness:.9
 }),
 leaves:new THREE.MeshStandardMaterial({
  map:texture("#347b32",8),roughness:1
 }),
 sand:new THREE.MeshStandardMaterial({
  map:texture("#d7c58b",9),roughness:1
 }),
 coal:new THREE.MeshStandardMaterial({
  map:texture("#393939",10),roughness:1
 }),
 iron:new THREE.MeshStandardMaterial({
  map:texture("#a9a29b",11),roughness:1
 }),
 gold:new THREE.MeshStandardMaterial({
  map:texture("#d7ad31",12),roughness:.7
 }),
 diamond:new THREE.MeshStandardMaterial({
  map:texture("#37d7ce",13),roughness:.4
 })
};

const waterMat=new THREE.MeshPhysicalMaterial({
 color:0x268dd1,
 roughness:.16,
 metalness:.05,
 transparent:true,
 opacity:.78,
 clearcoat:1
});

/* =================================
   BLOCK WORLD
================================= */

const blocks=new Map();
const meshes=new Map();

const blockGeometry=new THREE.BoxGeometry(1,1,1);

function key(x,y,z){
 return `${x},${y},${z}`;
}

function addBlock(x,y,z,type){
 const k=key(x,y,z);
 if(blocks.has(k))return;

 blocks.set(k,type);

 const material=type==="grass"?[
  mats.grassSide,mats.grassSide,
  mats.grassTop,mats.dirt,
  mats.grassSide,mats.grassSide
 ]:mats[type]||mats.dirt;

 const mesh=new THREE.Mesh(blockGeometry,material);
 mesh.position.set(x+.5,y+.5,z+.5);
 mesh.castShadow=true;
 mesh.receiveShadow=true;
 mesh.userData={x,y,z,type};

 scene.add(mesh);
 meshes.set(k,mesh);
}

function removeBlock(x,y,z){
 const k=key(x,y,z);
 const mesh=meshes.get(k);
 if(!mesh)return;

 scene.remove(mesh);
 meshes.delete(k);
 blocks.delete(k);
}

/* =================================
   TERRAIN GENERATION
================================= */

function noise(x,z){
 return Math.sin(x*.16)*2+
        Math.cos(z*.19)*2+
        Math.sin((x+z)*.09)*2+
        Math.cos((x-z)*.05)*2;
}

function heightAt(x,z){
 return Math.floor(5+noise(x,z));
}

function generateTerrain(){

 for(let x=-25;x<=25;x++){
  for(let z=-25;z<=25;z++){

   const h=heightAt(x,z);

   for(let y=-3;y<=h;y++){

    let type="stone";

    if(y===-3)type="stone";
    else if(y===h)type="grass";
    else if(y>=h-3)type="dirt";

    if(y<h-3 && Math.random()<.035)
     type="coal";

    if(y<h-4 && Math.random()<.018)
     type="iron";

    if(y<-1 && Math.random()<.006)
     type="gold";

    if(y<-2 && Math.random()<.002)
     type="diamond";

    addBlock(x,y,z,type);
   }
  }
 }
}

/* =================================
   TREES
================================= */

function makeTree(x,z){

 const ground=heightAt(x,z);

 for(let y=1;y<=5;y++)
  addBlock(x,ground+y,z,"wood");

 for(let dx=-2;dx<=2;dx++){
  for(let dz=-2;dz<=2;dz++){
   for(let dy=3;dy<=7;dy++){

    if(Math.abs(dx)+Math.abs(dz)>4)continue;

    if(Math.random()>.12)
     addBlock(x+dx,ground+dy,z+dz,"leaves");
   }
  }
 }
}

generateTerrain();

for(let i=0;i<65;i++){
 const x=Math.floor(Math.random()*46)-23;
 const z=Math.floor(Math.random()*46)-23;
 if(Math.abs(x)>4||Math.abs(z)>4)
  makeTree(x,z);
}

/* =================================
   WATER
================================= */

const water=new THREE.Mesh(
 new THREE.PlaneGeometry(100,100,50,50),
 waterMat
);

water.rotation.x=-Math.PI/2;
water.position.set(0,3,0);
water.receiveShadow=true;
scene.add(water);

/* =================================
   CLOUDS
================================= */

const clouds=[];

const cloudMat=new THREE.MeshLambertMaterial({
 color:0xffffff
});

for(let i=0;i<18;i++){

 const group=new THREE.Group();

 const x=Math.random()*100-50;
 const y=25+Math.random()*12;
 const z=Math.random()*100-50;

 for(let j=0;j<6;j++){
  const b=new THREE.Mesh(
   new THREE.BoxGeometry(
    3+Math.random()*3,
    1.5+Math.random()*1.5,
    3+Math.random()*3
   ),
   cloudMat
  );

  b.position.set(
   j*2-5,
   Math.random(),
   Math.random()*3
  );

  group.add(b);
 }

 group.position.set(x,y,z);
 scene.add(group);
 clouds.push(group);
}

/* =================================
   PLAYER
================================= */

const player={
 x:0,y:15,z:0,
 vy:0,
 health:20,
 hunger:20,
 grounded:false,
 speed:5
};

let yaw=0;
let pitch=0;
let playing=false;
let locked=false;
let creative=false;

camera.position.set(0,15,0);

const keys={};

document.addEventListener("keydown",e=>{
 keys[e.code]=true;

 if(e.code==="KeyE")toggleInventory();

 if(e.code==="Digit1")select(0);
 if(e.code==="Digit2")select(1);
 if(e.code==="Digit3")select(2);
 if(e.code==="Digit4")select(3);
 if(e.code==="Digit5")select(4);
 if(e.code==="Digit6")select(5);
 if(e.code==="Digit7")select(6);
 if(e.code==="Digit8")select(7);
 if(e.code==="Digit9")select(8);
});

document.addEventListener("keyup",e=>{
 keys[e.code]=false;
});

document.addEventListener("mousemove",e=>{
 if(!locked)return;

 yaw-=e.movementX*.002;
 pitch-=e.movementY*.002;
 pitch=Math.max(-1.55,Math.min(1.55,pitch));

 camera.rotation.order="YXZ";
 camera.rotation.y=yaw;
 camera.rotation.x=pitch;
});

renderer.domElement.addEventListener("click",()=>{
 if(playing&&!locked)
  renderer.domElement.requestPointerLock();
});

document.addEventListener("pointerlockchange",()=>{
 locked=document.pointerLockElement===renderer.domElement;
});

/* =================================
   COLLISION AND MOVEMENT
================================= */

function solid(x,y,z){
 const b=blocks.get(key(
  Math.floor(x),Math.floor(y),Math.floor(z)
 ));
 return b!==undefined;
}

function collision(x,y,z){

 const r=.3;

 for(const dx of [-r,r]){
  for(const dz of [-r,r]){
   for(const dy of [0,1,1.7]){
    if(solid(x+dx,y+dy,z+dz))return true;
   }
  }
 }

 return false;
}

function updatePlayer(dt){

 const forward=new THREE.Vector3(
  -Math.sin(yaw),0,-Math.cos(yaw)
 );

 const right=new THREE.Vector3(
  Math.cos(yaw),0,-Math.sin(yaw)
 );

 const direction=new THREE.Vector3();

 if(keys.KeyW)direction.add(forward);
 if(keys.KeyS)direction.sub(forward);
 if(keys.KeyD)direction.add(right);
 if(keys.KeyA)direction.sub(right);

 if(direction.length()>0)direction.normalize();

 let speed=keys.ShiftLeft?8:5;

 const nx=player.x+direction.x*speed*dt;
 const nz=player.z+direction.z*speed*dt;

 if(!collision(nx,player.y,player.z))
  player.x=nx;

 if(!collision(player.x,player.y,nz))
  player.z=nz;

 if(creative){

  if(keys.Space)player.y+=speed*dt;
  if(keys.ControlLeft)player.y-=speed*dt;

 }else{

  if(keys.Space&&player.grounded){
   player.vy=8;
   player.grounded=false;
  }

  player.vy-=22*dt;

  const ny=player.y+player.vy*dt;

  if(collision(player.x,ny,player.z)){
   if(player.vy<0)player.grounded=true;
   player.vy=0;
  }else{
   player.y=ny;
   player.grounded=false;
  }
 }

 camera.position.set(
  player.x,player.y+1.6,player.z
 );

 if(player.y<-15){
  player.x=0;
  player.z=0;
  player.y=15;
  player.health=Math.max(1,player.health-5);
 }

 document.getElementById("health").textContent=player.health;
}

/* =================================
   INVENTORY
================================= */

const types=[
 "grass","dirt","stone","wood","leaves",
 "sand","coal","iron","gold"
];

const icons={
 grass:"🟩",dirt:"🟫",stone:"⬜",
 wood:"🪵",leaves:"🌿",sand:"🟨",
 coal:"⚫",iron:"◽",gold:"🟡"
};

const inventory={};
types.forEach(t=>inventory[t]=32);

let selected=0;

function select(n){
 selected=n;
 updateHotbar();
}

function updateHotbar(){

 const bar=document.getElementById("hotbar");
 bar.innerHTML="";

 for(let i=0;i<9;i++){
  const type=types[i];
  const div=document.createElement("div");

  div.className="slot"+(i===selected?" selected":"");

  div.innerHTML=`
   ${icons[type]}
   <span class="count">${inventory[type]}</span>
  `;

  bar.appendChild(div);
 }
}

function toggleInventory(){

 const panel=document.getElementById("inventory");

 if(panel.style.display==="block"){
  panel.style.display="none";
  renderer.domElement.requestPointerLock();
  return;
 }

 document.exitPointerLock();
 panel.style.display="block";

 const items=document.getElementById("items");
 items.innerHTML="";

 types.forEach((type,i)=>{

  const div=document.createElement("div");
  div.className="item";
  div.textContent=icons[type];

  div.onclick=()=>{
   select(i);
   panel.style.display="none";
   renderer.domElement.requestPointerLock();
  };

  items.appendChild(div);
 });
}

document.getElementById("close").onclick=toggleInventory;

/* =================================
   BLOCK MINING AND PLACING
================================= */

const raycaster=new THREE.Raycaster();
raycaster.far=6;

function getTarget(){

 raycaster.setFromCamera(
  new THREE.Vector2(0,0),camera
 );

 const hits=raycaster.intersectObjects(
  Array.from(meshes.values())
 );

 return hits.length?hits[0]:null;
}

document.addEventListener("mousedown",e=>{

 if(!playing||!locked)return;

 const hit=getTarget();
 if(!hit)return;

 const b=hit.object.userData;

 if(e.button===0){

  removeBlock(b.x,b.y,b.z);
  inventory[b.type]=(inventory[b.type]||0)+1;
  updateHotbar();

 }else if(e.button===2){

  const n=hit.face.normal;

  const x=b.x+Math.round(n.x);
  const y=b.y+Math.round(n.y);
  const z=b.z+Math.round(n.z);

  const type=types[selected];

  if(!inventory[type]&&!creative)return;

  if(!blocks.has(key(x,y,z))){

   if(collision(x+.5,y,z+.5))return;

   addBlock(x,y,z,type);

   if(!creative)inventory[type]--;

   updateHotbar();
  }
 }
});

document.addEventListener("contextmenu",e=>e.preventDefault());

/* =================================
   DAY AND NIGHT
================================= */

let clock=0;

function updateEnvironment(dt){

 clock+=dt*.025;

 const sunHeight=Math.sin(clock);

 sun.position.set(
  Math.cos(clock)*45,
  15+sunHeight*35,
  Math.sin(clock)*35
 );

 const daylight=Math.max(.12,(sunHeight+1)/2);

 sun.intensity=daylight*3;
 ambient.intensity=.3+daylight*.9;

 const sky=new THREE.Color();
 sky.setRGB(
  .025+daylight*.45,
  .04+daylight*.67,
  .12+daylight*.83
 );

 scene.background.copy(sky);
 scene.fog.color.copy(sky);

 clouds.forEach(c=>{
  c.position.x+=dt*.5;
  if(c.position.x>60)c.position.x=-60;
 });
}

/* =================================
   START AND GAME LOOP
================================= */

document.getElementById("play").onclick=()=>{
 document.getElementById("overlay").style.display="none";
 playing=true;
 renderer.domElement.requestPointerLock();
};

let last=performance.now();

function animate(now){

 requestAnimationFrame(animate);

 const dt=Math.min((now-last)/1000,.04);
 last=now;

 water.position.y=3+Math.sin(now*.001)*.08;

 if(playing&&locked){
  updatePlayer(dt);
  updateEnvironment(dt);
 }

 renderer.render(scene,camera);
}

updateHotbar();
animate(performance.now());

/* =================================
   RESIZE
================================= */

window.addEventListener("resize",()=>{
 camera.aspect=innerWidth/innerHeight;
 camera.updateProjectionMatrix();
 renderer.setSize(innerWidth,innerHeight);
});

</script>
</body>
</html>
