
<html lang="en">
<head>
<meta charset="UTF-8">
<title>Popcorn Multiplayer</title>
<meta name="viewport" content="width=device-width, initial-scale=1">
<style>
body{
  margin:0;
  font-family:Arial, sans-serif;
  background:#f8fafc;
  display:flex;
  justify-content:center;
  align-items:center;
  height:100vh;
}
#game{
  width:360px;
  height:560px;
  background:white;
  border-radius:18px;
  box-shadow:0 10px 30px rgba(0,0,0,.15);
  position:relative;
  overflow:hidden;
}
header{
  padding:12px;
  text-align:center;
  background:#facc15;
  font-weight:bold;
}
#info{
  display:flex;
  justify-content:space-between;
  padding:8px 12px;
  font-size:14px;
}
#field{
  position:relative;
  width:100%;
  height:380px;
  background:#fff7ed;
}
.kernel{
  position:absolute;
  width:30px;
  height:30px;
  background:#fde68a;
  border-radius:50%;
  cursor:pointer;
  display:flex;
  justify-content:center;
  align-items:center;
}
#mp{
  padding:10px;
  text-align:center;
}
#gameover{
  position:absolute;
  inset:0;
  background:rgba(0,0,0,.6);
  color:white;
  display:none;
  justify-content:center;
  align-items:center;
  flex-direction:column;
}
button{
  padding:8px 14px;
  border:none;
  border-radius:8px;
  background:#facc15;
  font-weight:bold;
  cursor:pointer;
  margin:4px;
}
</style>
</head>

<body>
<div id="game">
<header>🍿 Popcorn Multiplayer</header>

<div id="info">
  <div>You: <span id="myScore">0</span></div>
  <div>Friend: <span id="opScore">0</span></div>
  <div>Miss: <span id="miss">0</span>/5</div>
</div>

<div id="field"></div>

<div id="mp">
  <button onclick="host()">Host</button>
  <button onclick="join()">Join</button>
  <div id="status">Offline</div>
</div>

<div id="gameover">
  <h2>Game Over</h2>
  <p>You: <span id="fMy"></span> | Friend: <span id="fOp"></span></p>
  <button onclick="resetGame()">Restart</button>
</div>
</div>

<script>
const field=document.getElementById("field")
const myScoreEl=document.getElementById("myScore")
const opScoreEl=document.getElementById("opScore")
const missEl=document.getElementById("miss")
const statusEl=document.getElementById("status")
const gameOverEl=document.getElementById("gameover")
const fMy=document.getElementById("fMy")
const fOp=document.getElementById("fOp")

let myScore=0, opScore=0, miss=0
let speed=2000
let spawnLoop
let pc, dc
let multiplayer=false

/* ---------- MULTIPLAYER ---------- */
function host(){
  multiplayer=true
  setup(true)
}
function join(){
  multiplayer=true
  setup(false)
}

function setup(isHost){
  pc=new RTCPeerConnection()
  if(isHost){
    dc=pc.createDataChannel("pop")
    bind()
  }else{
    pc.ondatachannel=e=>{dc=e.channel;bind()}
  }

  pc.onicecandidate=e=>{
    if(e.candidate) console.log(JSON.stringify(e.candidate))
  }

  if(isHost){
    pc.createOffer().then(o=>{
      pc.setLocalDescription(o)
      alert("Send OFFER:\n"+JSON.stringify(o))
    })
  }else{
    const offer=JSON.parse(prompt("Paste OFFER"))
    pc.setRemoteDescription(offer)
    pc.createAnswer().then(a=>{
      pc.setLocalDescription(a)
      alert("Send ANSWER:\n"+JSON.stringify(a))
    })
  }
}

function bind(){
  dc.onopen=()=>{
    statusEl.textContent="Connected"
    start()
  }
  dc.onmessage=e=>{
    const d=JSON.parse(e.data)
    if(d.t==="spawn") spawnKernel(d.id,d.x,d.y)
    if(d.t==="pop"){
      const k=document.getElementById(d.id)
      if(k){k.remove();opScore++;opScoreEl.textContent=opScore}
    }
    if(d.t==="miss"){
      miss=d.m
      missEl.textContent=miss
      if(miss>=5) endGame()
    }
    if(d.t==="over") endGame()
  }
}

/* ---------- GAME ---------- */
function start(){
  clearInterval(spawnLoop)
  spawnLoop=setInterval(()=>{
    const id="k"+Date.now()
    const x=Math.random()*(field.clientWidth-30)
    const y=Math.random()*(field.clientHeight-30)
    spawnKernel(id,x,y)
    dc.send(JSON.stringify({t:"spawn",id,x,y}))
  },speed)
}

function spawnKernel(id,x,y){
  const k=document.createElement("div")
  k.className="kernel"
  k.id=id
  k.style.left=x+"px"
  k.style.top=y+"px"
  field.appendChild(k)

  let timer=setTimeout(()=>{
    if(k.parentNode){
      k.remove()
      miss++
      missEl.textContent=miss
      dc.send(JSON.stringify({t:"miss",m:miss}))
      if(miss>=5) dc.send(JSON.stringify({t:"over"}))
    }
  },1500)

  k.onclick=()=>{
    clearTimeout(timer)
    if(!k.parentNode)return
    k.remove()
    myScore++
    myScoreEl.textContent=myScore
    dc.send(JSON.stringify({t:"pop",id}))
  }
}

function endGame(){
  clearInterval(spawnLoop)
  fMy.textContent=myScore
  fOp.textContent=opScore
  gameOverEl.style.display="flex"
}

function resetGame(){
  location.reload()
}
</script>
</body>
</html>
