<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width,initial-scale=1">
<title>ZYVO Challenge</title>

<style>
*{box-sizing:border-box}
body{
 margin:0;
 font-family:Arial,sans-serif;
 background:#070a12;
 color:white;
}
.app{
 max-width:480px;
 min-height:100vh;
 margin:auto;
 padding:22px 16px 35px;
 background:linear-gradient(180deg,#111827,#05070c);
}
.logo{
 text-align:center;
 font-size:48px;
 font-weight:900;
 letter-spacing:8px;
 margin:10px 0 3px;
}
.tag{
 text-align:center;
 color:#9ca3af;
 margin-bottom:20px;
}
.card{
 background:#151c29;
 border:1px solid #293448;
 border-radius:22px;
 padding:20px;
 margin-bottom:15px;
}
input{
 width:100%;
 padding:15px;
 border-radius:12px;
 border:1px solid #374151;
 background:#0b1019;
 color:white;
 font-size:16px;
 outline:none;
}
button{
 border:0;
 border-radius:13px;
 padding:14px 18px;
 font-weight:bold;
 font-size:16px;
 cursor:pointer;
}
.primary{
 width:100%;
 background:#4f46e5;
 color:white;
 margin-top:10px;
}
.mode{
 display:flex;
 gap:10px;
}
.mode button{
 flex:1;
 background:#222b3b;
 color:white;
}
.mode button.active{
 background:#4f46e5;
}
.stats{
 display:grid;
 grid-template-columns:repeat(3,1fr);
 gap:8px;
 text-align:center;
}
.stat{
 background:#0c121d;
 padding:12px 5px;
 border-radius:13px;
}
.stat b{
 display:block;
 font-size:21px;
}
.stat span{
 color:#9ca3af;
 font-size:12px;
}
.hidden{display:none}
.timer{
 text-align:center;
 font-size:62px;
 font-weight:bold;
}
.score{
 text-align:center;
 font-size:24px;
 color:#a5b4fc;
 margin-bottom:15px;
}
.tap{
 display:block;
 width:220px;
 height:220px;
 border-radius:50%;
 margin:10px auto 20px;
 background:linear-gradient(145deg,#7c3aed,#2563eb);
 color:white;
 font-size:32px;
 box-shadow:0 15px 40px #2563eb55;
}
.tap:active{transform:scale(.93)}
.rank{
 text-align:center;
 font-size:26px;
 font-weight:bold;
 margin:8px 0;
}
.coin{
 text-align:center;
 color:#fbbf24;
 font-size:20px;
}
.result{
 text-align:center;
 padding-top:8px;
}
.result h2{font-size:30px}
.leader{
 display:flex;
 justify-content:space-between;
 padding:12px 5px;
 border-bottom:1px solid #283142;
}
.small{
 color:#9ca3af;
 font-size:13px;
}
.danger{
 background:#252b36;
 color:#ddd;
 margin-top:10px;
}
</style>
</head>

<body>
<div class="app">

<div class="logo">ZYVO</div>
<div class="tag">CHALLENGE HUB</div>

<div id="home">

<div class="card">
<h2>👤 Player</h2>
<input id="playerName" maxlength="16" placeholder="Enter your name">
</div>

<div class="card">
<h2>⚡ Choose Mode</h2>
<div class="mode">
<button class="active" onclick="chooseMode(10,this)">10s</button>
<button onclick="chooseMode(30,this)">30s</button>
<button onclick="chooseMode(60,this)">60s</button>
</div>
<button class="primary" onclick="startGame()">START CHALLENGE</button>
</div>

<div class="card">
<div class="stats">
<div class="stat"><b id="coins">0</b><span>COINS</span></div>
<div class="stat"><b id="games">0</b><span>GAMES</span></div>
<div class="stat"><b id="best">0</b><span>BEST</span></div>
</div>
</div>

<div class="card">
<h2>🔥 Daily Challenge</h2>
<p class="small">Beat today's challenge and improve your rank.</p>
<button class="primary" onclick="startDaily()">PLAY DAILY</button>
</div>

<div class="card">
<h2>🏆 Leaderboard</h2>
<div id="leaderboard"></div>
</div>

</div>

<div id="game" class="hidden">

<div class="card">

<div class="timer" id="timer">10</div>
<div class="score">SCORE: <span id="score">0</span></div>

<button class="tap" id="tap" onclick="tap()">TAP!</button>

<div class="coin">🪙 +<span id="earned">0</span></div>

</div>

</div>

<div id="result" class="hidden">

<div class="card result">

<h2>🔥 TIME UP!</h2>

<p>Your Score</p>
<h1 id="finalScore">0</h1>

<div class="rank" id="rank">BRONZE</div>

<p class="coin">🪙 +<span id="finalCoins">0</span> Coins</p>

<button class="primary" onclick="goHome()">BACK TO HOME</button>

</div>

</div>

</div>

<script>

let mode=10;
let time=10;
let score=0;
let playing=false;
let interval=null;

let data=JSON.parse(localStorage.getItem("zyvoData"))||{
 coins:0,
 games:0,
 best:0,
 scores:[]
};

function save(){
 localStorage.setItem("zyvoData",JSON.stringify(data));
 updateHome();
}

function updateHome(){

document.getElementById("coins").textContent=data.coins;
document.getElementById("games").textContent=data.games;
document.getElementById("best").textContent=data.best;

renderLeaderboard();
}

function chooseMode(seconds,btn){

mode=seconds;

document.querySelectorAll(".mode button")
.forEach(b=>b.classList.remove("active"));

btn.classList.add("active");
}

function startGame(){

let name=document.getElementById("playerName").value.trim();

if(!name){
 alert("Please enter your player name.");
 return;
}

start(name,false);
}

function startDaily(){

let name=document.getElementById("playerName").value.trim();

if(!name){
 alert("Please enter your player name first.");
 return;
}

mode=20;
start(name,true);
}

function start(name,daily){

window.currentName=name;
window.daily=daily;

time=mode;
score=0;
playing=true;

document.getElementById("home").classList.add("hidden");
document.getElementById("result").classList.add("hidden");
document.getElementById("game").classList.remove("hidden");

document.getElementById("score").textContent=0;
document.getElementById("timer").textContent=time;
document.getElementById("earned").textContent=0;

clearInterval(interval);

interval=setInterval(()=>{

time--;
document.getElementById("timer").textContent=time;

if(time<=0) finish();

},1000);
}

function tap(){

if(!playing)return;

score++;

document.getElementById("score").textContent=score;

document.getElementById("earned").textContent=Math.floor(score/2);
}

function finish(){

if(!playing)return;

playing=false;
clearInterval(interval);

let coins=Math.floor(score/2)+5;

if(window.daily) coins+=10;

data.coins+=coins;
data.games++;

if(score>data.best)data.best=score;

data.scores.push({
 name:window.currentName,
 score:score
});

data.scores.sort((a,b)=>b.score-a.score);
data.scores=data.scores.slice(0,10);

save();

document.getElementById("game").classList.add("hidden");
document.getElementById("result").classList.remove("hidden");

document.getElementById("finalScore").textContent=score;
document.getElementById("finalCoins").textContent=coins;
document.getElementById("rank").textContent=getRank(score);
}

function getRank(score){

if(score>=60)return"💎 DIAMOND";
if(score>=40)return"🥇 GOLD";
if(score>=25)return"🥈 SILVER";
return"🥉 BRONZE";
}

function goHome(){

document.getElementById("result").classList.add("hidden");
document.getElementById("home").classList.remove("hidden");

updateHome();
}

function renderLeaderboard(){

let box=document.getElementById("leaderboard");

if(!data.scores.length){
box.innerHTML='<p class="small">No scores yet. Be the first!</p>';
return;
}

box.innerHTML=data.scores.map((x,i)=>`

<div class="leader">
<span>${i+1}. ${escapeHTML(x.name)}</span>
<b>${x.score}</b>
</div>

`).join("");
}

function escapeHTML(text){

return text.replace(/[&<>"']/g,function(m){
return({
"&":"&amp;",
"<":"&lt;",
">":"&gt;",
'"':"&quot;",
"'":"&#039;"
})[m];
});
}

updateHome();

</script>
</body>
</html>
