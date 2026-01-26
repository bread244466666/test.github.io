<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<title>Popcorn Multiplayer Arcade</title>
<meta name="viewport" content="width=device-width, initial-scale=1">
<script src="https://cdn.jsdelivr.net/npm/pako@2.1.0/dist/pako.min.js"></script>

<style>
body{
  margin:0;
  background:#020617;
  color:#e5e7eb;
  font-family:Arial;
  text-align:center;
}
header{
  padding:20px;
  background:linear-gradient(135deg,#2563eb,#4f46e5);
}
canvas{
  background:black;
  border-radius:12px;
  display:block;
  margin:20px auto;
}
button{
  padding:10px 18px;
  margin:6px;
  border:none;
  border-radius:8px;
  background:#2563eb;
  color:white;
  cursor:pointer;
}
#status{margin-top:10px}
</style>
</head>

<body>

<header>
<h1>🍿 Popcorn Multiplayer</h1>
<p>Move paddle • Break popcorn • One-click invite</p>
</header>

<button onclick="hostGame()">Host Game</button>

<div id="status">Not connected</div>

<canvas id="game" width="400" height="500"></canvas>

<script>
/* ================= WEBRTC ONE CLICK INVITE ================= */

let pc, dc

function encodeSDP(obj){
  const json = JSON.stringify(obj)
  const compressed = pako.deflate(json)
  return btoa(String.fromCharCode(...compressed))
    .replace(/\+/g,'-').replace(/\//g,'_').replace(/=+$/,'')
}

function decodeSDP(str){
  str=str.replace(/-/g,'+').replace(/_/g,'/')
  const bin=atob(str)
  const arr=Uint8Array.from(bin,c=>c.charCodeAt(0))
  return JSON.parse(pako.inflate(arr,{to:'string'}))
}

function createPeer(isHost){
  pc=new RTCPeerConnection()

  if(isHost){
    dc=pc.createDataChannel("game")
    bindChannel()
  }else{
    pc.ondatachannel=e=>{
      dc=e.channel
      bindChannel()
    }
  }

  pc.onicecandidate=e=>{
    if(e.candidate) return
    if(pc.localDescription.type==="offer"){
      const code=encodeSDP(pc.localDescription)
      const link=location.origin+location.pathname+"#join="+code
      navigator.clipboard.writeText(link)
      alert("Invite link copied:\n\n"+link)
    }
    if(pc.localDescription.type==="answer"){
      const code=encodeSDP(pc.localDescription)
      location.hash="answer="+code
    }
  }
}

function bindChannel(){
  dc.onopen=()=>{
    status.textContent="Connected"
  }
  dc.onmessage=e=>{
    const s=JSON.parse(e.data)
    ball=s.ball
    blocks=s.blocks
  }
}

async function hostGame(){
  createPeer(true)
  await pc.setLocalDescription(await pc.createOffer())
}

window.onload=async()=>{
  if(location.hash.startsWith("#join=")){
    const offer=decodeSDP(location.hash.slice(6))
    createPeer(false)
    await pc.setRemoteDescription(offer)
    await pc.setLocalDescription(await pc.createAnswer())
  }
  if(location.hash.startsWith("#answer=")){
    await pc.setRemoteDescription(decodeSDP(location.hash.slice(8)))
  }
}

/* ================= POPCORN GAME ================= */

const canvas=document.getElementById("game")
const ctx=canvas.getContext("2d")

let paddle={x:170,w:60}
let ball={x:200,y:250,vx:3,vy:-3}
let blocks=[]
let isHost=false

function initBlocks(){
  blocks=[]
  for(let y=0;y<4;y++){
    for(let x=0;x<7;x++){
      blocks.push({x:x*55+10,y:y*30+30})
    }
  }
}
initBlocks()

document.addEventListener("mousemove",e=>{
  const r=canvas.getBoundingClientRect()
  paddle.x=e.clientX-r.left-paddle.w/2
})

function update(){
  ctx.clearRect(0,0,400,500)

  ctx.fillStyle="cyan"
  ctx.fillRect(paddle.x,470,paddle.w,10)

  ball.x+=ball.vx
  ball.y+=ball.vy

  if(ball.x<0||ball.x>400) ball.vx*=-1
  if(ball.y<0) ball.vy*=-1

  if(ball.y>460&&ball.x>paddle.x&&ball.x<paddle.x+paddle.w){
    ball.vy*=-1
  }

  blocks=blocks.filter(b=>{
    if(ball.x>b.x&&ball.x<b.x+50&&ball.y>b.y&&ball.y<b.y+20){
      ball.vy*=-1
      return false
    }
    return true
  })

  ctx.fillStyle="yellow"
  ctx.beginPath()
  ctx.arc(ball.x,ball.y,6,0,7)
  ctx.fill()

  ctx.fillStyle="orange"
  blocks.forEach(b=>ctx.fillRect(b.x,b.y,50,20))

  if(dc&&dc.readyState==="open"){
    dc.send(JSON.stringify({ball,blocks}))
  }

  requestAnimationFrame(update)
}
update()
</script>

</body>
</html>
