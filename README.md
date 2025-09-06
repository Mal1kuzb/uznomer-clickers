<!DOCTYPE html>
<html lang="uz">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Uznomer Clicker</title>
<style>
body {
    margin:0;
    font-family:Arial, sans-serif;
    background: linear-gradient(135deg, #000000, #1a1a1a, #3d2b1f);
    color:#ffd700;
    text-align:center;
    height:100vh;
    overflow:hidden;
}
#score { font-size:28px; margin-top:40px; margin-bottom:20px; }
.click-circle {
    width:180px; height:180px; margin:20px auto; border-radius:50%;
    background:#2c2c2c; display:flex; align-items:center; justify-content:center;
    font-size:20px; font-weight:bold; color:#ffd700; cursor:pointer;
    transition: transform 0.2s;
}
.click-circle:active { transform: scale(0.9); }

#limitBarContainer { width:80%; height:25px; background:#444; margin:20px auto; border-radius:12px; overflow:hidden; }
#limitBar { height:100%; width:100%; display:flex; align-items:center; justify-content:center; font-weight:bold; }

.panels { display:grid; grid-template-columns:repeat(2,1fr); gap:10px; width:80%; margin:30px auto; }
.panel-btn, .promokod-btn {
    padding:15px; background:#2c2c2c; border:none; border-radius:10px;
    font-size:18px; font-weight:bold; color:#ffd700; cursor:pointer;
    transition: transform 0.3s, background 0.3s;
}
.panel-btn:hover, .promokod-btn:hover { transform:scale(1.05); background:#3d3d3d; }
.promokod-btn { width:80%; margin:20px auto; padding:20px; font-size:20px; border-radius:12px; }

.fullscreen-panel {
    position: fixed; top:0; left:0; width:100%; height:100%;
    background: rgba(0,0,0,0.95); display:none; flex-direction:column;
    align-items:center; justify-content:center; animation: fadeIn 0.5s ease forwards;
    z-index:100; color:#ffd700; padding:20px; overflow-y:auto;
}
.fullscreen-panel.active { display:flex; }
.back-btn {
    position:absolute; top:15px; left:15px; padding:10px 15px;
    background:#2c2c2c; border:none; border-radius:8px; color:#ffd700;
    font-weight:bold; cursor:pointer;
}
.plusOne {
    position:absolute; font-size:24px; color:yellow; font-weight:bold;
    animation: rise 1s forwards; pointer-events:none;
}
@keyframes rise {0% {opacity:1; transform:translateY(0);}100% {opacity:0; transform:translateY(-50px);}}
@keyframes fadeIn {from {opacity:0; transform:scale(0.8);} to {opacity:1; transform:scale(1);}}
</style>
</head>
<body>

<div id="score">0</div>
<div class="click-circle" id="clickCircle">Uznomer Clicker</div>

<div id="limitBarContainer">
    <div id="limitBar">1000/2500</div>
</div>

<div class="panels">
    <button class="panel-btn" onclick="openPanel('xizmatlar')">Xizmatlar</button>
    <button class="panel-btn" onclick="openPanel('sotuv')">Sotuv</button>
</div>
<div>
    <button class="promokod-btn" onclick="openPanel('promokod')">Promokod</button>
</div>

<!-- Xizmatlar paneli -->
<div class="fullscreen-panel" id="xizmatlar">
    <button class="back-btn" onclick="closePanel('xizmatlar')">⬅ Orqaga</button>
    <h2>Xizmatlar</h2>
    <p>2x 30s (150 Click)</p>
    <button onclick="buyDoubleClick()">Sotib olish</button>
    <p id="doubleAlert"></p>
    <button onclick="increaseLimitFunc()">Limitni oshirish</button>
</div>

<!-- Sotuv paneli -->
<div class="fullscreen-panel" id="sotuv">
    <button class="back-btn" onclick="closePanel('sotuv')">⬅ Orqaga</button>
    <h2>Sotuv</h2>
    <p>1 clicker = 0.01 $</p>
    <input type="number" id="clickerInput" placeholder="Clickers soni">
    <button onclick="calculateDollar()">Hisoblash</button>
    <p id="dollarValue">Sizning dollar qiymatingiz: 0 $</p>
    <button onclick="sellClickers()">Sotish</button>
    <p id="sellAlert"></p>
</div>

<!-- Promokod paneli -->
<div class="fullscreen-panel" id="promokod">
    <button class="back-btn" onclick="closePanel('promokod')">⬅ Orqaga</button>
    <h2>Promokod</h2>
    <input type="text" id="promoInput" placeholder="Promokod kiriting">
    <button onclick="applyPromo()">Qo'llash</button>
    <p id="promoAlert"></p>
</div>

<script>
// Click va limit
let clicks = localStorage.getItem('clicks') ? parseInt(localStorage.getItem('clicks')) : 0;
let currentLimit = localStorage.getItem('currentLimit') ? parseInt(localStorage.getItem('currentLimit')) : 1000;
let maxLimit = 2500;
let increaseCost = 150;

const scoreDiv=document.getElementById('score');
const clickCircle=document.getElementById('clickCircle');
const limitBar=document.getElementById('limitBar');

scoreDiv.textContent=clicks;
updateLimitBar();

clickCircle.addEventListener('click',(e)=>{
    if(currentLimit>0){
        clicks++;
        currentLimit--;
        scoreDiv.textContent=clicks;
        updateLimitBar();
        localStorage.setItem('clicks', clicks);
        localStorage.setItem('currentLimit', currentLimit);
        showPlusOne(e);
    } else { alert('Limit tugadi!'); }
});

function showPlusOne(event){
    const plus=document.createElement('div');
    plus.classList.add('plusOne');
    plus.innerText='+1';
    const x = event.clientX + (Math.random()*20-10);
    const y = event.clientY + (Math.random()*20-10);
    plus.style.left = x+'px';
    plus.style.top = y+'px';
    document.body.appendChild(plus);
    setTimeout(()=> plus.remove(),1000);
}

// Limit barni yangilash (o'zgartirilgan)
function updateLimitBar(){
    let percent = Math.min((currentLimit/maxLimit)*100, 100);
    limitBar.style.width = percent + '%';

    // Rangni limitga qarab o'zgartirish
    let red = Math.floor(255 * (1 - currentLimit / maxLimit));
    let green = Math.floor(255 * (currentLimit / maxLimit));
    limitBar.style.background = `rgb(${red},${green},0)`;

    // Matnni o'rtada saqlash
    limitBar.textContent = `${currentLimit}/${maxLimit}`;
    limitBar.style.display = 'flex';
    limitBar.style.alignItems = 'center';
    limitBar.style.justifyContent = 'center';
}

// Limit asta-sekin tiklanadi
setInterval(()=>{
    if(currentLimit<maxLimit){
        currentLimit++;
        updateLimitBar();
        localStorage.setItem('currentLimit', currentLimit);
    }
},1000);

// Xizmatlar
function buyDoubleClick(){ alert('2x aktiv 30s davomida!'); }
function increaseLimitFunc(){
    if(clicks>=increaseCost){
        clicks-=increaseCost;
        currentLimit+=1000;
        maxLimit+=1000;
        increaseCost+=100;
        scoreDiv.textContent=clicks;
        updateLimitBar();
        localStorage.setItem('clicks', clicks);
        localStorage.setItem('currentLimit', currentLimit);
    } else { alert('Click yetarli emas!'); }
}

// Sotuv
function calculateDollar(){
    const input = document.getElementById('clickerInput').value;
    document.getElementById('dollarValue').innerText = `Sizning dollar qiymatingiz: ${(input*0.01).toFixed(2)} $`;
}
function sellClickers(){ alert('Sotish imkoniyati hozir yo‘q'); }

// Promokod
function applyPromo(){
    const promo=document.getElementById('promoInput').value.trim();
    const promoAlert=document.getElementById('promoAlert');
    if(promo===''){ promoAlert.innerText='Iltimos promokod kiriting'; return; }
    promoAlert.innerText='Promokod ishlatildi!';
}

// Panel funktsiyalar
function openPanel(id){ document.getElementById(id).classList.add('active'); }
function closePanel(id){ document.getElementById(id).classList.remove('active'); }
</script>
</body>
</html>
