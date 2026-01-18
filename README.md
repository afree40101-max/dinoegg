<!DOCTYPE html>
<html lang="zh-Hant">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width,initial-scale=1">
<title>恐龍蛋雞蛋糕 POS</title>
<style>
body{margin:0;font-family:system-ui;background:#f7fdf9;font-size:22px}
header{background:#86efac;text-align:center;padding:12px;font-size:28px;font-weight:bold}
button{border:none;border-radius:14px}
.channel,.pay{display:flex;justify-content:space-around;margin:8px}
.channel button,.pay button{padding:10px;font-size:20px}
.active{background:#22c55e;color:#fff}
.items{display:grid;grid-template-columns:repeat(2,1fr);gap:8px;padding:8px}
.item{background:#fff;padding:10px;border-radius:14px;text-align:center}
.qty{display:flex;justify-content:center;gap:12px;margin-top:6px}
.qty button{width:44px;height:44px;font-size:26px}
.cart{background:#fff;margin:8px;padding:10px;border-radius:14px}
.total{font-size:26px;font-weight:bold;text-align:right}
.cashpad{display:grid;grid-template-columns:repeat(3,1fr);gap:6px;margin-top:6px}
.cashpad button{padding:14px;font-size:22px}
.overlay{position:fixed;top:0;left:0;width:100%;height:100%;background:#0008;display:none;z-index:99}
.modal{background:#fff;margin:5%;padding:20px;border-radius:18px}
</style>
</head>

<body>
<header>🦖 恐龍蛋雞蛋糕 POS</header>

<div class="channel">
<button class="active" onclick="setChannel('store',this)">現場</button>
<button onclick="setChannel('uber',this)">Uber Eats</button>
<button onclick="setChannel('panda',this)">熊貓</button>
</div>

<div class="items" id="items"></div>

<div class="cart">
<div id="cart">尚未點餐</div>
<div class="total">總計 $<span id="total">0</span></div>
</div>

<div class="pay">
<button onclick="openCash()">現金</button>
<button onclick="pay('LINE Pay')">LINE Pay</button>
<button onclick="pay('全支付')">全支付</button>
<button onclick="pay('Uber Eats')">Uber Eats</button>
<button onclick="pay('熊貓')">熊貓</button>
</div>

<div class="pay">
<button onclick="openAdmin()">🔐 老闆模式</button>
</div>

<!-- 現金視窗 -->
<div class="overlay" id="cashBox">
<div class="modal">
<h3>現金收款</h3>
<div>收 $<span id="cash">0</span></div>
<div class="cashpad">
<button onclick="addCash(100)">100</button>
<button onclick="addCash(500)">500</button>
<button onclick="addCash(1000)">1000</button>
<button onclick="num(1)">1</button><button onclick="num(2)">2</button><button onclick="num(3)">3</button>
<button onclick="num(4)">4</button><button onclick="num(5)">5</button><button onclick="num(6)">6</button>
<button onclick="num(7)">7</button><button onclick="num(8)">8</button><button onclick="num(9)">9</button>
<button onclick="num(0)">0</button>
<button onclick="confirmCash()">確認</button>
</div>
</div>
</div>

<!-- 老闆模式 -->
<div class="overlay" id="adminBox">
<div class="modal">
<h3>📊 今日業績</h3>
<div id="adminData"></div>
<button onclick="closeAdmin()">關閉</button>
</div>
</div>

<script>
const SCRIPT_URL="請貼你的 Apps Script Web App 網址";
const ADMIN_PIN="1234";

let channel="store",cart={},cash=0,sales=[];

const products={
"原味":{store:50,uber:65,panda:65},
"OREO":{store:60,uber:75,panda:75},
"起司":{store:65,uber:80,panda:80},
"卡士達":{store:65,uber:75,panda:75},
"黑糖麻糬":{store:65,uber:80,panda:80},
"苦甜巧克力":{store:60,uber:80,panda:80},
"爆漿苦甜巧克力":{store:65,uber:85,panda:85},
"桂冠湯圓":{store:80,uber:95,panda:95}
};

function setChannel(c,el){
channel=c;
document.querySelectorAll(".channel button").forEach(b=>b.classList.remove("active"));
el.classList.add("active");
render();
}

function render(){
let html="",list="",t=0;
for(let p in products){
html+=`<div class="item">${p}<br>$${products[p][channel]}
<div class="qty">
<button onclick="chg('${p}',-1)">−</button>
<span>${cart[p]||0}</span>
<button onclick="chg('${p}',1)">＋</button>
</div></div>`;
}
for(let p in cart){
let s=cart[p]*products[p][channel];
t+=s;
list+=`${p} x${cart[p]} $${s}<br>`;
}
items.innerHTML=html;
cartBox=list||"尚未點餐";
cart.innerHTML=cartBox;
total.innerText=t;
}

function chg(p,n){
cart[p]=(cart[p]||0)+n;
if(cart[p]<=0)delete cart[p];
render();
}

function openCash(){cash=0;cashBox.style.display="block";updateCash();}
function addCash(v){cash+=v;updateCash();}
function num(n){cash=Number(String(cash)+n);updateCash();}
function updateCash(){cashSpan.innerText=cash;}
function confirmCash(){
let amt=Number(total.innerText);
alert("找零 $"+(cash-amt));
cashBox.style.display="none";
finalPay("現金");
}

function pay(type){finalPay(type);}

function finalPay(type){
let amt=Number(total.innerText);
if(!amt)return alert("沒有金額");
sales.push({pay:type,amount:amt,items:{...cart}});
fetch(SCRIPT_URL,{method:"POST",body:JSON.stringify({
type:"sale",channel,pay:type,amount:amt,items:cart
})});
cart={};render();alert("完成");
}

function openAdmin(){
let p=prompt("輸入 PIN");
if(p!==ADMIN_PIN)return alert("錯誤");
let total=0,count=sales.length,pay={},items={};
sales.forEach(s=>{
total+=s.amount;
pay[s.pay]=(pay[s.pay]||0)+s.amount;
for(let i in s.items){items[i]=(items[i]||0)+s.items[i];}
});
let html=`<div>總營收 $${total}</div><div>筆數 ${count}</div><hr>`;
for(let p in pay){html+=`${p} $${pay[p]}<br>`}
html+="<hr>";
for(let i in items){html+=`${i} x${items[i]}<br>`}
adminData.innerHTML=html;
adminBox.style.display="block";
}
function closeAdmin(){adminBox.style.display="none";}

render();
</script>
</body>
</html>
