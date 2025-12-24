<!DOCTYPE html>  
<html>  
<head>  
<meta charset="UTF-8">  
<title>MiniVerse</title>  
<script src="https://www.gstatic.com/firebasejs/9.23.0/firebase-app-compat.js"></script>  
<script src="https://www.gstatic.com/firebasejs/9.23.0/firebase-auth-compat.js"></script>  
<script src="https://www.gstatic.com/firebasejs/9.23.0/firebase-database-compat.js"></script>  
<style>  
body{margin:0;background:#0b0b0b;color:white;font-family:Arial}  
.top{display:flex;justify-content:space-between;padding:10px;background:#111}  
button{background:#1f1f1f;color:white;border:1px solid #333;padding:8px;cursor:pointer}  
button:hover{background:#333}  
.menu{display:flex;justify-content:space-around;padding:40px}  
.card{width:260px;height:200px;background:#111;border:2px solid #333;padding:15px;display:flex;flex-direction:column;justify-content:space-between;}  
.card h3{font-size:26px;}  
.card button{font-size:18px;padding:10px;}  
.panel{padding:20px;position:relative}  
.hidden{display:none}  
input,textarea{background:#111;color:white;border:1px solid #333;width:100%}  
.bullet{position:absolute;width:6px;height:6px;border-radius:50%;}  
.bot{position:absolute;width:30px;height:30px;background:purple;border-radius:50%;cursor:pointer;}  
.boss{position:absolute;width:80px;height:80px;background:red;border-radius:10%;cursor:pointer;}  
.title-display{font-weight:bold;}  
.rainbow{background-image:linear-gradient(to right, red,orange,yellow,green,blue,indigo,violet);-webkit-background-clip:text;color:transparent;}  
</style>  
</head>  
<body>  
<div class="top">  
  <div><span id="titleDisplay" class="title-display"></span> <span id="playerNameTop">Player</span> 💰 <span id="money">0</span></div>  
  <div>  
    <button onclick="show('menu')">Menu</button>  
    <button onclick="show('friends')">Friends</button>  
    <button onclick="show('leaderboard')">Leaderboard</button>  
    <button onclick="show('create')">Create Game</button>  
    <button onclick="show('profile')">Profile</button>  
    <button onclick="show('shop')">Shop</button>  
  </div>  
</div>  
  
<div id="menu" class="menu">  
  <div class="card"><h3>Steal a Brainrot</h3><button onclick="show('brainrot')">Play</button></div>  
  <div class="card"><h3>Grow a Garden</h3><button onclick="show('garden')">Play</button></div>  
  <div class="card"><h3>Mini Shoot</h3><button onclick="show('shoot')">Play</button></div>  
  <div class="card"><h3>Daily Spin</h3><button onclick="spinWheel()">Spin</button></div>  
</div>  
  
<!-- Steal a Brainrot -->  
<div id="brainrot" class="panel hidden">  
  <h2>Steal a Brainrot</h2>  
  <button onclick="buyBrainrot()">Buy Brainrot (100$)</button>  
  <button onclick="showLuckyRewards()">Lucky Block Rewards</button>  
  <div id="inv"></div>  
</div>  
  
<!-- Grow a Garden -->  
<div id="garden" class="panel hidden">  
  <h2>Grow a Garden</h2>  
  <button onclick="plant()">Plant Seed</button>  
  <button onclick="showPlants()">Show Plants</button>  
  <div id="plants"></div>  
</div>  
  
<!-- Mini Shoot -->  
<div id="shoot" class="panel hidden">  
  <h2>Mini Shoot</h2>  
  <p>Gun: <span id="gun">Pistol</span></p>  
  <div>  
    <button onclick="selectGun('Pistol')">Pistol</button>  
    <button onclick="selectGun('AK-47')">AK-47</button>  
    <button onclick="selectGun('Grenade')">Grenade</button>  
    <button onclick="selectGun('Shotgun')">Shotgun</button>  
    <button onclick="selectGun('Sniper')">Sniper</button>  
  </div>  
  <button onclick="openGunBox()">Open Gun Box</button>  
  <p>Kills: <span id="kills">0</span></p>  
  <p>Coins: <span id="miniMoney">0</span></p>  
  <div id="bulletContainer" style="position:relative;width:100%;height:300px;"></div>  
</div>  
  
<div id="friends" class="panel hidden">  
  <h2>Friends</h2>  
  <input id="friendId" placeholder="Friend UID">  
  <button onclick="addFriend()">Add</button>  
  <div id="friendList"></div>  
</div>  
  
<div id="leaderboard" class="panel hidden">  
  <h2>Leaderboard</h2>  
  <div id="board"></div>  
</div>  
  
<div id="create" class="panel hidden">  
  <h2>Create Game</h2>  
  <input id="gname" placeholder="Game name">  
  <textarea id="gcode" rows="5" placeholder="JS code"></textarea>  
  <button onclick="createGame()">Create</button>  
  <div id="games"></div>  
</div>  
  
<div id="profile" class="panel hidden">  
  <h2>Profile</h2>  
  <input id="playerName" placeholder="Enter your name">  
  <input id="playerAvatar" placeholder="Enter avatar URL">  
  <button onclick="saveProfile()">Save</button>  
  <p>Your name: <span id="displayName"></span></p>  
  <img id="displayAvatar" src="" alt="Avatar" width="80">  
</div>  
  
<div id="shop" class="panel hidden">  
  <h2>Shop</h2>  
  <h3>Titles</h3>  
  <div id="titleShop"></div>  
  <h3>Avatars</h3>  
  <div id="avatarShop"></div>  
</div>  
  
<script>  
// Firebase init (replace with your config)  
firebase.initializeApp({apiKey:"YOUR_KEY",authDomain:"YOUR_PROJECT.firebaseapp.com",databaseURL:"https://YOUR_PROJECT-default-rtdb.firebaseio.com",projectId:"YOUR_PROJECT"});  
const auth=firebase.auth(); const db=firebase.database();  
let uid,money=100,kills=0,miniMoney=0,gun="Pistol",inv=[],garden=[],friends=[],gunsOwned=["Pistol"],lastSpin=0;  
let playerName="Player",playerAvatar="",playerTitle=null;  
  
// Brainrots  
const brainrots=[{n:"67",m:1,c:40},{n:"W or L",m:5,c:25},{n:"Meowl",m:15,c:15},{n:"Strawberry Elephant",m:40,c:8}];  
// Plants  
const plantsData=[{n:"Rose",m:2,price:20},{n:"Sunflower",m:6,price:40},{n:"Dandelion",m:15,price:60},{n:"Gold Rose",m:50,price:100},{n:"Sugar Apple",m:30,price:70},{n:"Tulip",m:5,price:25},{n:"Orchid",m:25,price:80},{n:"Lily",m:10,price:50},{n:"Carnation",m:12,price:45}];  
// Guns  
const gunsData={"Pistol":{damage:1,coin:1,bulletColor:"red"},"AK-47":{damage:3,coin:3,bulletColor:"orange"},"Grenade":{damage:5,coin:5,bulletColor:"yellow"},"Shotgun":{damage:7,coin:7,bulletColor:"blue"},"Sniper":{damage:10,coin:10,bulletColor:"green"}};  
// Titles  
const titles=[  
{name:"Novice Thief",price:1000,rarity:"common",color:"white"},  
{name:"Master Grower",price:500000,rarity:"rare",color:"green"},  
{name:"Legendary Shooter",price:10000000,rarity:"epic",color:"blue"},  
{name:"Galactic Overlord",price:1000000000,rarity:"legendary",color:"gold"},  
{name:"Never Touched Grass",price:100000000000,rarity:"mythic",color:"rainbow"}  
];  
// Avatars (example)  
const avatars=[{name:"Red Dragon",price:1000000,url:"https://i.imgur.com/xyz.png"},{name:"Blue Phoenix",price:5000000,url:"https://i.imgur.com/abc.png"}];  
  
auth.signInAnonymously().then(u=>{uid=u.user.uid; load(); loadProfile();});  
  
// Panels  
function show(id){document.querySelectorAll(".panel,.menu").forEach(x=>x.classList.add("hidden"));document.getElementById(id).classList.remove("hidden");}  
  
// Profile & save  
function saveProfile(){playerName=document.getElementById("playerName").value||playerName; playerAvatar=document.getElementById("playerAvatar").value||playerAvatar; db.ref("players/"+uid+"/profile").set({name:playerName,avatar:playerAvatar,title:playerTitle}); updateProfile();}  
function loadProfile(){db.ref("players/"+uid+"/profile").once("value").then(s=>{if(s.exists()){playerName=s.val().name||"Player"; playerAvatar=s.val().avatar||""; playerTitle=s.val().title||null;}updateProfile();});}  
function updateProfile(){document.getElementById("displayName").textContent=playerName; document.getElementById("playerNameTop").textContent=playerName; if(playerAvatar) document.getElementById("displayAvatar").src=playerAvatar; if(playerTitle){let t=document.getElementById("titleDisplay"); t.textContent=playerTitle.name; t.className="title-display"; if(playerTitle.color==="rainbow") t.classList.add("rainbow"); else t.style.color=playerTitle.color;}}  
  
// Money per second  
setInterval(()=>{let mps=inv.reduce((sum,b)=>sum+(b.m||0),0)+garden.reduce((sum,p)=>sum+(p.m||0),0); money+=mps; save(); update();},1000);  
  
// SAB  
function roll(){let r=Math.random()*100,t=0;for(let b of brainrots){t+=b.c;if(r<=t)return b;}return brainrots[0];}  
function buyBrainrot(){if(money<100){alert("Not enough!");return;} money-=100; let b=roll(); let r=Math.random()*100; if(r<70) inv.push({n:"Lucky Block",type:"lucky",subtype:"normal"}); else if(r<95) inv.push({n:"Secret Lucky Block",type:"lucky",subtype:"secret"}); else inv.push({n:"Admin Lucky Block",type:"lucky",subtype:"admin"}); save(); update();}  
function openLuckyBlock(i){const b=inv[i]; if(b.type!=="lucky") return; let reward; if(b.subtype==="normal"){reward=roll();} else if(b.subtype==="secret"){let r=Math.random()*100; if(r<75) reward={n:"Los Tralaletos",m:120}; else if(r<95) reward={n:"Le Grande Combination",m:300}; else if(r<95.1) reward={n:"Le Munchitas",m:800}; else reward={n:"Dragon Cannelloni",m:800};} else if(b.subtype==="admin"){let r=Math.random()*100; if(r<75) reward={n:"Sammy Spiderini",m:1500}; else if(r<125) reward={n:"Job Job Job Sahur",m:1200}; else if(r<145) reward={n:"Tung Tung Tung Sahur",m:1000}; else reward={n:"Los Marinos",m:2000};} inv.splice(i,1); inv.push(reward); alert("You got: "+reward.n); save(); update();}  
function showLuckyRewards(){alert("Normal: Anything from SAB\nSecret:\n- Los Tralaletos 75%\n- Le Grande Combination 20%\n- Le Munchitas 0.1%\n- Dragon Cannelloni 0.1%\nAdmin:\n- Sammy Spiderini 75%\n- Job Job Job Sahur 50%\n- Tung Tung Tung Sahur 20%\n- Los Marinos 1%");}  
function sellBrainrot(i){money+=inv[i].m||0; inv.splice(i,1); save(); update();}  
function showBrainrots(){alert(brainrots.map(b=>`${b.n} ${b.m}$/s`).join("\n"));}  
  
// GAG  
function plant(){const seed=plantsData[Math.floor(Math.random()*plantsData.length)]; if(money<seed.price){alert("Not enough!");return;} money-=seed.price; garden.push(seed); save(); update();}  
function showPlants(){alert(plantsData.map(p=>`${p.n} - Price: ${p.price}$ - ${p.m}$/s`).join("\n"));}  
  
// Mini Shoot  
let bots=[], bosses=[], bulletId=0;  
function selectGun(g){if(gunsOwned.includes(g)){gun=g;update();}else alert("You don't own this gun!");}  
function openGunBox(){const newGun=Object.keys(gunsData)[Math.floor(Math.random()*Object.keys(gunsData).length)]; if(!gunsOwned.includes(newGun)) { gunsOwned.push(newGun); alert("You got "+newGun+"!"); } else alert("You got "+newGun+" but already own it!"); save(); update();}  
function spawnBot(){const b=document.createElement("div"); b.className="bot"; b.style.left=Math.random()*270+"px"; b.style.top=Math.random()*150+"px"; b.onclick=()=>{money+=gunsData[gun].coin;miniMoney+=gunsData[gun].coin;kills++; b.remove(); save(); update();}; document.getElementById("bulletContainer").appendChild(b); bots.push({el:b,hp:10}); moveBot(b);}  
function spawnBoss(){const b=document.createElement("div"); b.className="boss"; b.style.left=Math.random()*220+"px"; b.style.top="0px"; b.onclick=()=>{money+=50; miniMoney+=50; b.remove(); save(); update();}; document.getElementById("bulletContainer").appendChild(b); bosses.push({el:b,hp:100}); moveBoss(b);}  
function moveBot(bot){const interval=setInterval(()=>{if(!document.body.contains(bot)) return; let top=parseInt(bot.style.top); top+=1; if(top>270){clearInterval(interval); bot.remove();} else bot.style.top=top+"px";},200);}  
function moveBoss(boss){const interval=setInterval(()=>{if(!document.body.contains(boss)) return; let top=parseInt(boss.style.top); top+=0.5; if(top>220){clearInterval(interval); boss.remove();} else boss.style.top=top+"px";},200);}  
// Auto spawn  
setInterval(()=>{spawnBot(); if(Math.random()<0.05) spawnBoss();},3000);  
  
// Daily Spin  
function spinWheel(){const today=new Date().toDateString(); if(lastSpin===today){alert("Already spun!"); return;} lastSpin=today; let r=Math.random()*100, reward=0; if(r<0.1) reward=10000; else if(r<1.1) reward=5000; else if(r<11.1) reward=3000; else if(r<31.1) reward=1000; else if(r<81.1) reward=800; else reward=500; money+=reward; miniMoney+=reward; alert("You won "+reward+" coins!"); save(); update();}  
  
// Friends & Create Game  
function addFriend(){friends.push(document.getElementById("friendId").value); save(); update();}  
function createGame(){db.ref("games").push({name:document.getElementById("gname").value,code:document.getElementById("gcode").value,plays:0});}  
  
// Shop  
function showShop(){let tShop=""; titles.forEach(t=>{tShop+=`${t.name} - ${t.price}$ <button onclick="buyTitle('${t.name}')">Buy</button><br>`}); document.getElementById("titleShop").innerHTML=tShop;  
let aShop=""; avatars.forEach(a=>{aShop=`${a.name} - ${a.price}$ <button onclick="buyAvatar('${a.name}')">Buy</button><br>`}); document.getElementById("avatarShop").innerHTML=aShop;}  
function buyTitle(name){const t=titles.find(x=>x.name===name); if(money<t.price){alert("Not enough!");return;} money-=t.price; playerTitle=t; save(); updateProfile(); update();}  
function buyAvatar(name){const a=avatars.find(x=>x.name===name); if(money<a.price){alert("Not enough!");return;} money-=a.price; playerAvatar=a.url; save(); updateProfile(); update();}  
  
// Update UI  
function update(){document.getElementById("money").textContent=money; document.getElementById("kills").textContent=kills; document.getElementById("miniMoney").textContent=miniMoney; document.getElementById("gun").textContent=gun;  
document.getElementById("inv").innerHTML=inv.map((b,i)=>{if(b.type==="lucky") return `${b.n} <button onclick="openLuckyBlock(${i})">Open</button>`; else return `${b.n} (${b.m}$/s) <button onclick="sellBrainrot(${i})">Sell</button>`;}).join("<br>");  
document.getElementById("plants").innerHTML=garden.map(p=>`${p.n} (${p.m}$/s)`).join("<br>");  
document.getElementById("friendList").innerHTML=friends.join("<br>");  
db.ref("leaderboard").once("value").then(s=>{let html=""; s.forEach(c=>{html+=`${c.val().name} - ${c.val().money}<br>`}); document.getElementById("board").innerHTML=html;});  
showShop();  
}  
  
// Save  
function save(){db.ref("players/"+uid+"/data").set({money,inv,garden,gunsOwned,kills,miniMoney,friends,lastSpin,playerTitle,playerAvatar,playerName});}  
  
load();  
  
</script>  
</body>  
</html>  
