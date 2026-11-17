<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width,initial-scale=1">
<title>Games</title>
<style>
*{box-sizing:border-box}
body{margin:0;font-family:Arial,sans-serif;background:#f5f6fa;color:#30364b}
button,input,select{font:inherit}
button{cursor:pointer;border:0;border-radius:9px;padding:11px 16px;font-weight:bold}
button:hover{filter:brightness(.94)}
.sidebar{position:fixed;left:0;top:0;bottom:0;width:225px;background:white;padding:25px 16px;border-right:1px solid #e5e7ef;z-index:5}
.logo{font-size:29px;font-weight:900;color:#3978e9;margin:5px 12px 35px}
.nav{display:block;width:100%;text-align:left;background:transparent;color:#626b80;padding:14px;margin-bottom:5px;font-size:15px}
.nav.active,.nav:hover{background:#eaf1ff;color:#3475e8}
.nav small{float:right}
.main{margin-left:225px;padding:30px 40px}
.top{display:flex;align-items:center;justify-content:space-between;margin-bottom:30px}
.top h1{margin:0;font-size:29px}
.profile{background:white;padding:11px 17px;border-radius:12px}
.search{width:100%;padding:15px;border:1px solid #e1e4ed;border-radius:12px;margin-bottom:25px;background:white;outline:none}
.search:focus,input:focus{border-color:#3978e9}
.banner{background:linear-gradient(110deg,#3978e9,#7859e9);color:white;padding:27px;border-radius:16px;margin-bottom:30px}
.banner h2{margin:0 0 10px;font-size:26px}
.banner button{background:white;color:#3978e9;margin-top:10px}
.section-title{display:flex;justify-content:space-between;align-items:center;margin:25px 0 15px}
.section-title h2{font-size:21px}
.grid{display:grid;grid-template-columns:repeat(auto-fill,minmax(220px,1fr));gap:17px}
.card{background:white;border-radius:13px;padding:18px;border:1px solid #e7e9f0;transition:.15s}
.card:hover{transform:translateY(-3px);box-shadow:0 5px 17px #0000000d}
.card-icon{font-size:35px;margin-bottom:12px}
.card h3{margin:5px 0 9px}
.card p{color:#82899b;font-size:13px;line-height:1.5}
.card button{margin-top:10px;background:#3978e9;color:white}
.secondary{background:#edf1fa!important;color:#43516c!important}
.purple{background:#7958e9!important;color:white!important}
.green{background:#20a879!important;color:white!important}
.page{display:none}
.page.active{display:block}
.form{background:white;padding:25px;border-radius:14px;max-width:750px}
.form input,.form select,.form textarea{width:100%;padding:13px;border:1px solid #dfe3ed;border-radius:9px;margin:8px 0 18px}
.form label{font-weight:bold;font-size:14px}
.question-box{padding:16px;background:#f5f7fb;border-radius:10px;margin:15px 0}
.question-box input{margin-bottom:8px}
.answer-grid{display:grid;grid-template-columns:1fr 1fr;gap:8px}
.answer-grid input{margin:0}
.primary{background:#3978e9;color:white}
.danger{background:#ffe9e9;color:#d33}
#playScreen{position:fixed;inset:0;background:#f4f6fb;z-index:20;display:none;overflow:auto;padding:30px}
.play-box{max-width:750px;margin:35px auto;background:white;padding:30px;border-radius:18px;text-align:center}
.play-box h2{font-size:27px}
.answers{display:grid;grid-template-columns:1fr 1fr;gap:13px;margin-top:25px}
.answer{padding:22px;background:#3978e9;color:white;font-size:17px}
.answer:nth-child(2){background:#e95765}
.answer:nth-child(3){background:#d69b24}
.answer:nth-child(4){background:#269c77}
#feedback{font-weight:bold;margin-top:20px}
.progress{height:9px;background:#e8ebf3;border-radius:10px;margin:20px 0}
.progress div{height:100%;background:#3978e9;border-radius:10px;width:0}
.empty{text-align:center;background:white;padding:45px;border-radius:15px;color:#8790a3}
.mobile{display:none}
@media(max-width:700px){
.sidebar{width:65px;padding:15px 7px}
.logo{font-size:0;margin:8px 4px 25px}
.logo:first-letter{font-size:25px}
.nav{font-size:0;text-align:center;padding:14px 4px}
.nav small{display:none}
.main{margin-left:65px;padding:20px 13px}
.top h1{font-size:23px}
.grid{grid-template-columns:1fr 1fr;gap:9px}
.card{padding:12px}
.card button{padding:9px}
}
</style>
</head>
<body>

<aside class="sidebar">
<div class="logo">🎮 Games</div>
<button class="nav active" onclick="showPage('home',this)">🏠 Home</button>
<button class="nav" onclick="showPage('discover',this)">🔎 Discover</button>
<button class="nav" onclick="showPage('sets',this)">📚 My Sets</button>
<button class="nav" onclick="showPage('create',this)">➕ Create Set</button>
<button class="nav" onclick="showPage('folders',this)">📁 Folders</button>
<button class="nav" onclick="showPage('games',this)">🎮 Games</button>
</aside>

<main class="main">
<div class="top">
<h1 id="heading">Home</h1>
<div class="profile">👤 Player</div>
</div>

<section id="home" class="page active">
<div class="banner">
<h2>Welcome to Games! 🎮</h2>
<p>Discover games, create quizzes, and challenge yourself.</p>
<button onclick="showPage('discover')">Explore Games →</button>
</div>
<div class="section-title"><h2>Quick Actions</h2></div>
<div class="grid">
<div class="card"><div class="card-icon">➕</div><h3>Create a Set</h3><p>Create your own quiz with custom questions.</p><button onclick="showPage('create')">Create</button></div>
<div class="card"><div class="card-icon">📚</div><h3>My Sets</h3><p>View and manage your saved quizzes.</p><button onclick="showPage('sets')">View Sets</button></div>
<div class="card"><div class="card-icon">🎮</div><h3>Play Games</h3><p>Choose a quiz and start playing.</p><button onclick="showPage('games')">Play Now</button></div>
</div>
<div class="section-title"><h2>Recently Added</h2></div>
<div class="grid" id="recent"></div>
</section>

<section id="discover" class="page">
<h2>Discover</h2>
<p>Explore available quiz sets.</p>
<input class="search" id="discoverSearch" placeholder="🔎 Search for sets..." oninput="renderDiscover()">
<div class="grid" id="discoverGrid"></div>
</section>

<section id="sets" class="page">
<div class="section-title">
<h2>My Sets</h2>
<button class="primary" onclick="showPage('create')">+ Create Set</button>
</div>
<input class="search" id="setSearch" placeholder="🔎 Search your sets..." oninput="renderSets()">
<div class="grid" id="setsGrid"></div>
</section>

<section id="create" class="page">
<h2>Create a Set</h2>
<div class="form">
<label>Set Name</label>
<input id="setName" placeholder="Example: Science Quiz">
<label>Description</label>
<input id="setDescription" placeholder="What is this set about?">
<label>Category</label>
<select id="setCategory">
<option>General Knowledge</option>
<option>Science</option>
<option>Math</option>
<option>History</option>
<option>Sports</option>
<option>Other</option>
</select>
<label>Choose an icon</label>
<select id="setIcon">
<option>📚</option><option>🧪</option><option>⚽</option><option>🌎</option><option>🧠</option><option>🎯</option>
</select>
<div id="questions"></div>
<button class="secondary" onclick="addQuestion()">+ Add Question</button>
<br><br>
<button class="primary" onclick="saveSet()">Save Set</button>
<button class="secondary" onclick="resetForm()">Clear</button>
</div>
</section>

<section id="folders" class="page">
<div class="section-title">
<h2>My Folders</h2>
<button class="primary" onclick="createFolder()">+ New Folder</button>
</div>
<div class="grid" id="folderGrid"></div>
</section>

<section id="games" class="page">
<h2>Game Modes</h2>
<p>Choose a game mode and play your quiz sets.</p>
<div class="grid">
<div class="card"><div class="card-icon">⚡</div><h3>Classic</h3><p>Answer questions and earn points for every correct answer.</p><button onclick="chooseSet('classic')">Play Classic</button></div>
<div class="card"><div class="card-icon">⏱️</div><h3>Time Rush</h3><p>Answer questions as quickly as possible.</p><button class="purple" onclick="chooseSet('rush')">Play Time Rush</button></div>
<div class="card"><div class="card-icon">❤️</div><h3>Survival</h3><p>You have three lives. How many questions can you get right?</p><button class="green" onclick="chooseSet('survival')">Play Survival</button></div>
</div>
</section>

</main>

<div id="playScreen">
<div class="play-box">
<div style="display:flex;justify-content:space-between">
<button class="secondary" onclick="closeGame()">✕ Exit</button>
<strong id="gameModeLabel">CLASSIC</strong>
</div>
<div class="progress"><div id="progressBar"></div></div>
<h3 id="questionNumber"></h3>
<h2 id="questionText"></h2>
<div class="answers" id="answerButtons"></div>
<div id="feedback"></div>
<h3 id="scoreText">Score: 0</h3>
<button id="nextButton" class="primary" style="display:none;margin-top:20px" onclick="nextQuestion()">Next Question →</button>
</div>
</div>

<script>
let sets=JSON.parse(localStorage.getItem('gamesSets')||'[]');
let folders=JSON.parse(localStorage.getItem('gamesFolders')||'[]');
let editingId=null;
let mode='classic';
let currentSet=null;
let questionIndex=0;
let score=0;
let lives=3;
let answered=false;
let timer=null;
let timeLeft=15;

const demo=[
{id:'demo1',name:'General Knowledge',description:'Test your general knowledge.',category:'General Knowledge',icon:'🧠',questions:[
{q:'What is the capital of France?',a:['London','Paris','Rome','Madrid'],correct:1},
{q:'How many days are in a week?',a:['5','6','7','8'],correct:2},
{q:'Which planet is known as the Red Planet?',a:['Venus','Mars','Jupiter','Saturn'],correct:1},
{q:'What is the largest ocean?',a:['Atlantic','Indian','Arctic','Pacific'],correct:3}
]},
{id:'demo2',name:'Sports Trivia',description:'Test your sports knowledge.',category:'Sports',icon:'⚽',questions:[
{q:'How many players are on a soccer team on the field?',a:['9','10','11','12'],correct:2},
{q:'How many points is a basketball three-pointer worth?',a:['1','2','3','4'],correct:2},
{q:'What sport uses a puck?',a:['Football','Hockey','Baseball','Tennis'],correct:1}
]},
{id:'demo3',name:'Science Quiz',description:'A basic science challenge.',category:'Science',icon:'🧪',questions:[
{q:'What gas do plants absorb?',a:['Oxygen','Nitrogen','Carbon dioxide','Hydrogen'],correct:2},
{q:'What is H2O?',a:['Salt','Water','Oxygen','Sugar'],correct:1},
{q:'What force keeps us on the ground?',a:['Friction','Gravity','Magnetism','Electricity'],correct:1}
]}
];

function allSets(){return [...demo,...sets]}

function saveData(){
localStorage.setItem('gamesSets',JSON.stringify(sets));
localStorage.setItem('gamesFolders',JSON.stringify(folders));
}

function showPage(id,btn){
document.querySelectorAll('.page').forEach(p=>p.classList.remove('active'));
document.getElementById(id).classList.add('active');
document.querySelectorAll('.nav').forEach(n=>n.classList.remove('active'));
if(btn)btn.classList.add('active');
let names={home:'Home',discover:'Discover',sets:'My Sets',create:'Create a Set',folders:'My Folders',games:'Games'};
document.getElementById('heading').textContent=names[id];
if(id==='sets')renderSets();
if(id==='discover')renderDiscover();
if(id==='folders')renderFolders();
if(id==='home')renderRecent();
}

function addQuestion(data){
let box=document.createElement('div');
box.className='question-box';
box.innerHTML=`
<div style="display:flex;justify-content:space-between">
<strong>Question</strong>
<button class="danger" onclick="this.closest('.question-box').remove()">Remove</button>
</div>
<input class="qtext" placeholder="Enter your question" value="">
<div class="answer-grid">
<input class="ans" placeholder="Answer 1">
<input class="ans" placeholder="Answer 2">
<input class="ans" placeholder="Answer 3">
<input class="ans" placeholder="Answer 4">
</div>
<label>Correct Answer</label>
<select class="correct">
<option value="0">Answer 1</option>
<option value="1">Answer 2</option>
<option value="2">Answer 3</option>
<option value="3">Answer 4</option>
</select>`;
if(data){
box.querySelector('.qtext').value=data.q;
box.querySelectorAll('.ans').forEach((el,i)=>el.value=data.a[i]||'');
box.querySelector('.correct').value=data.correct;
}
document.getElementById('questions').appendChild(box);
}

function resetForm(){
editingId=null;
document.getElementById('setName').value='';
document.getElementById('setDescription').value='';
document.getElementById('setCategory').value='General Knowledge';
document.getElementById('setIcon').value='📚';
document.getElementById('questions').innerHTML='';
addQuestion();
addQuestion();
}

function saveSet(){
let name=document.getElementById('setName').value.trim();
let description=document.getElementById('setDescription').value.trim();
let questions=[];
document.querySelectorAll('.question-box').forEach(box=>{
let q=box.querySelector('.qtext').value.trim();
let a=[...box.querySelectorAll('.ans')].map(x=>x.value.trim());
let correct=Number(box.querySelector('.correct').value);
if(q&&a.every(Boolean))questions.push({q,a,correct});
});
if(!name)return alert('Please enter a set name.');
if(!questions.length)return alert('Add at least one complete question with four answers.');
let item={id:editingId||('set'+Date.now()),name,description,category:document.getElementById('setCategory').value,icon:document.getElementById('setIcon').value,questions};
let index=sets.findIndex(s=>s.id===editingId);
if(index>=0)sets[index]=item;
else sets.push(item);
saveData();
resetForm();
showPage('sets');
alert('Your set has been saved!');
}

function renderSets(){
let search=(document.getElementById('setSearch')?.value||'').toLowerCase();
let list=sets.filter(s=>s.name.toLowerCase().includes(search));
document.getElementById('setsGrid').innerHTML=list.length?list.map(cardHTML).join(''):'<div class="empty">You have not created any sets yet.</div>';
}

function renderDiscover(){
let search=(document.getElementById('discoverSearch')?.value||'').toLowerCase();
let list=allSets().filter(s=>(s.name+' '+s.category+' '+s.description).toLowerCase().includes(search));
document.getElementById('discoverGrid').innerHTML=list.map(cardHTML).join('');
}

function renderRecent(){
document.getElementById('recent').innerHTML=allSets().slice(-4).reverse().map(cardHTML).join('');
}

function cardHTML(s){
return `<div class="card">
<div class="card-icon">${s.icon}</div>
<h3>${escapeHTML(s.name)}</h3>
<p>${escapeHTML(s.description||'Quiz set')}<br>${s.questions.length} questions · ${escapeHTML(s.category)}</p>
<button onclick="startGame('${s.id}','classic')">Play Set</button>
${sets.some(x=>x.id===s.id)?`<button class="secondary" onclick="editSet('${s.id}')">Edit</button><button class="danger" onclick="deleteSet('${s.id}')">Delete</button>`:''}
</div>`;
}

function escapeHTML(s){
return String(s).replace(/[&<>"']/g,c=>({'&':'&amp;','<':'&lt;','>':'&gt;','"':'&quot;',"'":'&#39;'}[c]));
}

function editSet(id){
let s=sets.find(x=>x.id===id);
if(!s)return;
editingId=id;
document.getElementById('setName').value=s.name;
document.getElementById('setDescription').value=s.description;
document.getElementById('setCategory').value=s.category;
document.getElementById('setIcon').value=s.icon;
document.getElementById('questions').innerHTML='';
s.questions.forEach(q=>addQuestion(q));
showPage('create');
}

function deleteSet(id){
if(!confirm('Delete this set?'))return;
sets=sets.filter(s=>s.id!==id);
saveData();
renderSets();
renderDiscover();
}

function chooseSet(m){
mode=m;
showPage('discover');
}

function startGame(id,m){
currentSet=allSets().find(s=>s.id===id);
if(!currentSet||!currentSet.questions.length)return;
mode=m||'classic';
questionIndex=0;
score=0;
lives=3;
document.getElementById('playScreen').style.display='block';
document.getElementById('gameModeLabel').textContent=mode.toUpperCase();
showQuestion();
}

function showQuestion(){
clearInterval(timer);
answered=false;
let q=currentSet.questions[questionIndex];
document.getElementById('questionNumber').textContent='Question '+(questionIndex+1)+' of '+currentSet.questions.length;
document.getElementById('questionText').textContent=q.q;
document.getElementById('feedback').textContent='';
document.getElementById('nextButton').style.display='none';
document.getElementById('progressBar').style.width=(questionIndex/currentSet.questions.length*100)+'%';
document.getElementById('answerButtons').innerHTML=q.a.map((a,i)=>`<button class="answer" onclick="checkAnswer(${i},this)">${escapeHTML(a)}</button>`).join('');
document.getElementById('scoreText').textContent='Score: '+score+(mode==='survival'?' | Lives: ❤️ '+lives:'');
if(mode==='rush'){
timeLeft=15;
document.getElementById('feedback').textContent='⏱️ 15 seconds!';
timer=setInterval(()=>{
timeLeft--;
document.getElementById('feedback').textContent='⏱️ '+timeLeft+' seconds left!';
if(timeLeft<=0){
clearInterval(timer);
if(!answered){
answered=true;
document.getElementById('feedback').textContent='Time is up!';
document.getElementById('answerButtons').querySelectorAll('button').forEach(b=>b.disabled=true);
document.getElementById('nextButton').style.display='inline-block';
}
}
},1000);
}
}

function checkAnswer(i,button){
if(answered)return;
answered=true;
clearInterval(timer);
let q=currentSet.questions[questionIndex];
let correct=i===q.correct;
if(correct){
score+=mode==='rush'?150:100;
document.getElementById('feedback').textContent='✅ Correct! Great job!';
}else{
document.getElementById('feedback').textContent='❌ Incorrect! Correct answer: '+q.a[q.correct];
if(mode==='survival')lives--;
}
document.getElementById('answerButtons').querySelectorAll('button').forEach((b,index)=>{
b.disabled=true;
if(index===q.correct)b.style.outline='4px solid #25bd82';
});
document.getElementById('scoreText').textContent='Score: '+score+(mode==='survival'?' | Lives: ❤️ '+lives:'');
document.getElementById('nextButton').style.display='inline-block';
}

function nextQuestion(){
if(mode==='survival'&&lives<=0)return finishGame();
questionIndex++;
if(questionIndex>=currentSet.questions.length)finishGame();
else showQuestion();
}

function finishGame(){
clearInterval(timer);
document.getElementById('progressBar').style.width='100%';
document.getElementById('questionNumber').textContent='GAME OVER';
document.getElementById('questionText').textContent='You scored '+score+' points!';
document.getElementById('answerButtons').innerHTML='<h2>🏆</h2>';
document.getElementById('feedback').textContent='You completed '+Math.min(questionIndex+1,currentSet.questions.length)+' questions!';
document.getElementById('scoreText').textContent='Final Score: '+score;
document.getElementById('nextButton').style.display='none';
}

function closeGame(){
clearInterval(timer);
document.getElementById('playScreen').style.display='none';
}

function createFolder(){
let name=prompt('What would you like to name your folder?');
if(!name||!name.trim())return;
folders.push({id:Date.now(),name:name.trim(),sets:[]});
saveData();
renderFolders();
}

function renderFolders(){
document.getElementById('folderGrid').innerHTML=folders.length?folders.map(f=>`
<div class="card">
<div class="card-icon">📁</div>
<h3>${escapeHTML(f.name)}</h3>
<p>${f.sets.length} sets</p>
<button onclick="openFolder(${f.id})">Open Folder</button>
<button class="danger" onclick="deleteFolder(${f.id})">Delete</button>
</div>`).join(''):'<div class="empty">No folders yet. Create one to organize your sets!</div>';
}

function openFolder(id){
let f=folders.find(x=>x.id===id);
if(!f)return;
let available=sets.filter(s=>!f.sets.includes(s.id));
let choice=prompt('Folder: '+f.name+'\nEnter the number of a set to add:\n'+available.map((s,i)=>(i+1)+'. '+s.name).join('\n'));
if(choice&&available[Number(choice)-1]){
f.sets.push(available[Number(choice)-1].id);
saveData();
renderFolders();
}else if(choice){
alert('Invalid selection.');
}
}

function deleteFolder(id){
if(!confirm('Delete this folder?'))return;
folders=folders.filter(f=>f.id!==id);
saveData();
renderFolders();
}

resetForm();
renderRecent();
</script>
</body>
</html>
