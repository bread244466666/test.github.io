<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>3D Image Viewer with Three.js</title>
    <style>
        body { margin: 0; overflow: hidden; } / Remove default margins and scrollbars /
        canvas { display: block; } / Make canvas fill the div/body /
        #loading-screen {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: #222;
            color: white;
            display: flex;
            justify-content: center;
            align-items: center;
            font-family: sans-serif;
            font-size: 2em;
            z-index: 100;
        }
    </style>
</head>
<body>
    <!-- Loading screen while image loads -->
    <div id="loading-screen">
        Loading image...
    </div>

    <!-- The JavaScript code will append the Three.js canvas here -->
