<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>My First Three.js 3D Model</title>
    <style>
        body { margin: 0; overflow: hidden; } / Remove default margins and scrollbars /
        canvas { display: block; } / Ensure the canvas takes up the whole space /
    </style>
</head>
<body>
    <!--
        The Three.js script is loaded from a CDN (Content Delivery Network).
        We use 'type="module"' because Three.js is distributed as an ES module,
        and this allows us to use 'import' statements in our 'main.js'.
        OrbitControls is also an ES module from the examples directory.
    -->
    <script type="module" src="https://unpkg.com/three@0.160.0/build/three.module.js"></script>
    <script type="module" src="https://unpkg.com/three@0.160.0/examples/jsm/controls/OrbitControls.js"></script>
    // Import necessary modules from Three.js
// 'THREE' is the main Three.js library
// 'OrbitControls' allows for mouse interaction (rotate, zoom)
import  as THREE from 'https://unpkg.com/three@0.160.0/build/three.module.js';
import { OrbitControls } from 'https://unpkg.com/three@0.160.0/examples/jsm/controls/OrbitControls.js';

// --- 1. Set up the Scene, Camera, and Renderer ---

// Create a new Three.js Scene. This is where all objects, cameras, and lights are placed.
const scene = new THREE.Scene();

// Create a PerspectiveCamera. This mimics how the human eye sees things.
// Arguments: field of view (degrees), aspect ratio, near clipping plane, far clipping plane
const camera = new THREE.PerspectiveCamera(75, window.innerWidth / window.innerHeight, 0.1, 1000);

// Create a WebGLRenderer. This is responsible for rendering the scene onto a canvas.
const renderer = new THREE.WebGLRenderer({ antialias: true }); // antialias for smoother edges
renderer.setSize(window.innerWidth, window.innerHeight); // Set the renderer size to fill the window
document.body.appendChild(renderer.domElement); // Add the renderer's canvas to the HTML body

// --- 2. Create the 3D Model (Geometry + Material = Mesh) ---

// Define the geometry (shape) of our 3D model. Here, a simple box.
// Arguments: width, height, depth
const geometry = new THREE.BoxGeometry(1, 1, 1); // A 1x1x1 unit cube

// Define the material (appearance) of our 3D model.
// MeshStandardMaterial reacts to light, making it more realistic.
// You can change 'color' to any hex code (e.g., 0xff0000 for red).
const material = new THREE.MeshStandardMaterial({ color: 0x00ff00 }); // A green color

// Create a Mesh by combining the geometry and material. This is our actual 3D object.
const cube = new THREE.Mesh(geometry, material);
scene.add(cube); // Add the cube to the scene

// --- 3. Add Lights to the Scene ---
// Without lights, MeshStandardMaterial objects will appear black.

// AmbientLight: Illuminates all objects in the scene equally from all directions.
// Arguments: color, intensity (0-1)
const ambientLight = new THREE.AmbientLight(0xffffff, 0.5); // Soft white light
scene.add(ambientLight);

// DirectionalLight: Simulates light from a distant source (like the sun).
// Arguments: color, intensity (0-1)
const directionalLight = new THREE.DirectionalLight(0xffffff, 0.8);
directionalLight.position.set(1, 1, 1).normalize(); // Position the light (relative to scene origin)
scene.add(directionalLight);

// --- 4. Position the Camera ---
camera.position.z = 5; // Move the camera back 5 units so we can see the cube

// --- 5. Add Interactive Controls ---
// OrbitControls allows you to rotate, pan, and zoom the camera with your mouse/touch.
const controls = new OrbitControls(camera, renderer.domElement);
controls.enableDamping = true; // Give a sense of weight to the controls
controls.dampingFactor = 0.05;
controls.screenSpacePanning = false;
controls.minDistance = 2; // Minimum zoom distance
controls.maxDistance = 10; // Maximum zoom distance
controls.maxPolarAngle = Math.PI / 2; // Limit vertical rotation

// --- 6. The Animation/Render Loop ---
// This function will be called repeatedly, roughly 60 times per second (if possible).
function animate() {
    requestAnimationFrame(animate); // Request the browser to call 'animate' again for the next frame

    // Update controls (important if damping is enabled)
    controls.update();

    // Optional: Rotate the cube on each frame (uncomment to see it spin)
    // cube.rotation.x += 0.005;
    // cube.rotation.y += 0.005;

    renderer.render(scene, camera); // Render the scene from the perspective of the camera
}

animate(); // Start the animation loop

// --- 7. Handle Window Resizing ---
// This ensures the 3D scene resizes correctly when the browser window is resized.
window.addEventListener('resize', () => {
    // Update camera aspect ratio
    camera.aspect = window.innerWidth / window.innerHeight;
    camera.updateProjectionMatrix(); // Recalculate projection matrix for the camera
