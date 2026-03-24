<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>3D Image Cube</title>
    <style>
        body {
            margin: 0;
            padding: 0;
            display: flex;
            justify-content: center;
            align-items: center;
            min-height: 100vh;
            background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
            perspective: 1000px;
        }

        .container {
            width: 300px;
            height: 300px;
            position: relative;
            transform-style: preserve-3d;
            animation: rotate 20s infinite linear;
        }

        .face {
            position: absolute;
            width: 300px;
            height: 300px;
            background-size: cover;
            background-position: center;
            border: 2px solid rgba(255, 255, 255, 0.3);
            box-shadow: 0 0 20px rgba(0, 0, 0, 0.5);
        }

        .front  { transform: rotateY(0deg) translateZ(150px); }
        .back   { transform: rotateY(180deg) translateZ(150px); }
        .right  { transform: rotateY(90deg) translateZ(150px); }
        .left   { transform: rotateY(-90deg) translateZ(150px); }
        .top    { transform: rotateX(90deg) translateZ(150px); }
        .bottom { transform: rotateX(-90deg) translateZ(150px); }

        @keyframes rotate {
            0% { transform: rotateX(0deg) rotateY(0deg); }
            100% { transform: rotateX(360deg) rotateY(360deg); }
        }

        .container:hover {
            animation-play-state: paused;
        }
    </style>
</head>
<body>
    <div class="container" id="cube">
        <div class="face front"></div>
        <div class="face back"></div>
        <div class="face right"></div>
        <div class="face left"></div>
        <div class="face top"></div>
        <div class="face bottom"></div>
    </div>

    <script>
        // Add your image URL here
        const imageUrl = 'https://picsum.photos/300/300';
        
        const faces = document.querySelectorAll('.face');
        faces.forEach(face => {
            face.style.backgroundImage = url(${imageUrl});
        });

        // Interactive rotation with mouse
        const cube = document.getElementById('cube');
        let isDragging = false;
        let previousMouseX = 0;
        let previousMouseY = 0;
        let rotationX = 0;
        let rotationY = 0;

        cube.addEventListener('mousedown', (e) => {
            isDragging = true;
            previousMouseX = e.clientX;
            previousMouseY = e.clientY;
            cube.style.animation = 'none';
        });

        document.addEventListener('mousemove', (e) => {
            if (isDragging) {
                const deltaX = e.clientX - previousMouseX;
                const deltaY = e.clientY - previousMouseY;
                
                rotationY += deltaX  0.5;
                rotationX -= deltaY  0.5;
                
                cube.style.transform = rotateX(${rotationX}deg) rotateY(${rotationY}deg);
                
                previousMouseX = e.clientX;
                previousMouseY = e.clientY;
            }
        });
