<html lang="en">
<head>
<meta charset="utf-8" />
<title>3D Image Slices Parallax</title>
<meta name="viewport" content="width=device-width,initial-scale=1" />
<style>
html,body{height:100%;margin:0;font-family:Arial,Helvetica,sans-serif;background:#111;color:#fff;display:flex;align-items:center;justify-content:center}
.stage {
width: 720px;
max-width: 90vw;
height: 480px;
max-height: 70vh;
perspective: 1200px; / camera distance /
-webkit-perspective: 1200px;
position: relative;
}

.card {
width:100%;
height:100%;
position:relative;
transform-style: preserve-3d; / allow children to translateZ /
transition: transform 0.15s ease-out;
will-change: transform;
border-radius:8px;
overflow: hidden;
box-shadow: 0 18px 50px rgba(0,0,0,0.6), inset 0 1px 0 rgba(255,255,255,0.02);
background: #222;
}

/ each slice uses the image as background and is absolutely positioned /
.slice {
position: absolute;
top: 0;
bottom: 0;
transform-style: preserve-3d;
backface-visibility: hidden;
will-change: transform;
pointer-events: none;
}

/ small UI below /
.controls {
margin-top:14px;
width:720px;
max-width:90vw;
display:flex;
gap:12px;
align-items:center;
justify-content:center;
color:#ddd;
font-size:14px;
}
.controls input[type="range"] {width:180px}
.credits {position: absolute; left: 8px; bottom: 8px; font-size:12px; color: rgba(255,255,255,0.6)}
</style>
</head>
<body>

<div style="text-align:center">
<div class="stage" id="stage">
<div class="card" id="card"></div>
<div class="credits">Move mouse / touch to rotate</div>
</div>

<div class="controls" aria-hidden="true">
<label>Slices: <span id="sliceCountLabel">30</span>
<input id="sliceCount" type="range" min="6" max="120" value="30">
</label>
<label>Depth: <span id="depthLabel">220</span>px
<input id="depth" type="range" min="20" max="800" value="220">
</label>
<label>Rotate strength:
<input id="rotateStrength" type="range" min="4" max="40" value="14">
</label>
<button id="reload">Reload slices</button>
</div>
</div>

<script>
// Configuration: change this to use another image
const IMAGESRC = 'https://images.unsplash.com/photo-1503023345310-bd7c1de61c7d?q=80&w=2400&auto=format&fit=crop&ixlib=rb-4.0.3&s=4a2b6fcfb7d9a2b5c9f6d8f2d8a8a3d6';

const stage = document.getElementById('stage');
const card = document.getElementById('card');

const sliceRange = document.getElementById('sliceCount');
const sliceCountLabel = document.getElementById('sliceCountLabel');
const depthRange = document.getElementById('depth');
const depthLabel = document.getElementById('depthLabel');
const rotateRange = document.getElementById('rotateStrength');
const reloadBtn = document.getElementById('reload');

let slices = parseInt(sliceRange.value, 10);
let depth = parseInt(depthRange.value, 10);
let rotateStrength = parseInt(rotateRange.value, 10);

let img = new Image();
img.crossOrigin = 'anonymous'; // in case of remote images
img.src = IMAGESRC;

// update labels
sliceCountLabel.textContent = slices;
depthLabel.textContent = depth;

// when image loads, build slices
img.addEventListener('load', () => {
buildSlices();
});

// rebuild when controls change
sliceRange.addEventListener('input', () => {
slices = parseInt(sliceRange.value, 10);
sliceCountLabel.textContent = slices;
});
depthRange.addEventListener('input', () => {
depth = parseInt(depthRange.value, 10);
depthLabel.textContent = depth;
});
rotateRange.addEventListener('input', () => {
rotateStrength = parseInt(rotateRange.value, 10);
});

reloadBtn.addEventListener('click', () => buildSlices());

function buildSlices() {
// Clear existing slices
card.innerHTML = '';

const w = card.clientWidth;
const h = card.clientHeight;

// Use the natural image size for crisp background scaling
const imgW = img.naturalWidth;
const imgH = img.naturalHeight;

// We'll set background-size to match the card's display size so each slice lines up:
const bgSize = ${w}px ${h}px;

for (let i = 0; i < slices; i++) {
const sliceEl = document.createElement('div');
sliceEl.className = 'slice';

// width & left as percentage so they adapt to container size
const leftPct = (i / slices) 100;
const widthPct = (1 / slices) 100;
sliceEl.style.left = leftPct + '%';
sliceEl.style.width = widthPct + '%';

// background image uses the full image but positioned so each slice shows the correct part
// compute background-position-x in pixels relative to the displayed width
const bgPosX = - (i (w / slices));
sliceEl.style.backgroundImage = url(${IMAGESRC});
sliceEl.style.backgroundSize = bgSize;
sliceEl.style.backgroundPosition = ${bgPosX}px 0px;
sliceEl.style.backgroundRepeat = 'no-repeat';

// place each slice at a different z position to create depth
const z = ((i - (slices - 1) / 2) / (slices / 2)) (depth / 2);
sliceEl.style.transform = translateZ(${z}px);

// subtle layer ordering so close slices appear above further ones
sliceEl.style.zIndex = String(i);

card.appendChild(sliceEl);
}
}

// Mouse / touch interaction to rotate card
let rect = null;
function updateRect() {
rect = stage.getBoundingClientRect();
}
window.addEventListener('resize', updateRect);
updateRect();

let pointerDown = false;
let lastX = 0, lastY = 0;
function onPointerMove(clientX, clientY) {
if (!rect) updateRect();
const cx = rect.left + rect.width / 2;
const cy = rect.top + rect.height / 2;
const dx = clientX - cx;
const dy = clientY - cy;

// Normalize to [-1,1]
const nx = dx / (rect.width / 2);
const ny = dy / (rect.height / 2);

const rotY = -nx rotateStrength; // rotate around vertical axis
const rotX = ny (rotateStrength * 0.7); // rotate around horizontal axis (slightly less)

// Slight translateZ to emphasize perspective on hover
const translateZ = pointerDown ? 40 : 16;

card.style.transform = rotateX(${rotX}deg) rotateY(${rotY}deg) translateZ(${translateZ}px);
}

// mouse
stage.addEventListener('mousemove', (e) => {
onPointerMove(e.clientX, e.clientY);
});
stage.addEventListener('mouseenter', (e) => {
pointerDown = true;
onPointerMove(e.clientX, e.clientY);
});
stage.addEventListener('mouseleave', () => {
pointerDown = false;
// return to gentle rest
card.style.transform = rotateX(0deg) rotateY(0deg) translateZ(10px);
});

// touch
stage.addEventListener('touchstart', (e) => {
pointerDown = true;
const t = e.touches[0];
onPointerMove(t.clientX, t.clientY);
}, {passive: true});
stage.addEventListener('touchmove', (e) => {
const t = e.touches[0];
onPointerMove(t.clientX, t.clientY);
}, {passive: true});
stage.addEventListener('touchend', () => {
pointerDown = false;
card.style.transform = rotateX(0deg) rotateY(0deg) translateZ(10px);
});

// initial fallback transform
card.style.transform = translateZ(10px);

// ensure slices rebuild if container resizes (keeps background-size correct)
const resizeObserver = new ResizeObserver(() => {
// rebuild slices to update background-size/positions
if (img.complete) buildSlices();
updateRect();
});
resizeObserver.observe(card);

// If you want to allow user-provided images, you could add a file input and set IMAGESRC to a data URL.
</script>
