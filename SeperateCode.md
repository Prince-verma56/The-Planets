import './style.css'

import * as THREE from 'three';
import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls.js';
import { RGBELoader } from 'three/examples/jsm/loaders/RGBELoader.js';
import gsap from 'gsap';
import { Expo } from 'gsap';

// Scene
const scene = new THREE.Scene();

// Camera
const camera = new THREE.PerspectiveCamera(
  25,
  window.innerWidth / window.innerHeight,
  0.1,
  1000
);
camera.position.z = 5.5;

const starTexture = new THREE.TextureLoader().load("./stars.jpg")
const starGeometry = new THREE.SphereGeometry(50, 64, 64)
const starMaterial = new THREE.MeshStandardMaterial({
  map: starTexture,
  opacity: 0.1,
  side: THREE.BackSide
})
starTexture.colorSpace = THREE.SRGBColorSpace
const starSphere = new THREE.Mesh(starGeometry, starMaterial)
scene.add(starSphere)

const sphereMesh = [];

// HDRI Lighting
const hdrLoader = new RGBELoader();
hdrLoader.load('https://dl.polyhaven.org/file/ph-assets/HDRIs/hdr/1k/venice_sunset_1k.hdr', (texture) => {
  texture.mapping = THREE.EquirectangularReflectionMapping;
  scene.environment = texture;
});

// Renderer
const canvas = document.querySelector("#canvas")
const renderer = new THREE.WebGLRenderer({ canvas });
renderer.setSize(window.innerWidth, window.innerHeight);

// Geometry and Material
const radius = 0.5;
const segments = 32;
const textures = ["./csilla/color.png", "./earth/map.jpg", "./venus/map.jpg", "./volcanic/color.png"]
const spheres = new THREE.Group();

const orbitRadius = 2.5;
for (let i = 0; i < 4; i++) {
  const textureLoader = new THREE.TextureLoader();
  const texture = textureLoader.load(textures[i]);
  texture.colorSpace = THREE.SRGBColorSpace;

  const geometry = new THREE.SphereGeometry(radius, segments, segments);
  const material = new THREE.MeshStandardMaterial({ map: texture });
  const sphere = new THREE.Mesh(geometry, material);
  sphereMesh.push(sphere)

  const angle = (i / 4) * Math.PI * 2;
  sphere.position.x = orbitRadius * Math.cos(angle);
  sphere.position.z = orbitRadius * Math.sin(angle);
  spheres.rotation.x = 0.1
  spheres.position.y = -0.1
  spheres.add(sphere);
}
scene.add(spheres);

// Controls
const controls = new OrbitControls(camera, renderer.domElement);
controls.enableDamping = true;
controls.dampingFactor = 0.1;

// Heading animation setup
const headings = document.querySelectorAll(".heading");

// Auto and Scroll Sync
let scrollCount = 0;
const totalPlanets = 4;

function goToIndex(index) {
  scrollCount = index % totalPlanets;

  const targetAngle = (scrollCount / totalPlanets) * Math.PI * 2;

  gsap.to(spheres.rotation, {
    y: targetAngle,
    duration: 1,
    ease: "power2.inOut",
  });

  gsap.to(headings, {
    duration: 1,
    y: `-${scrollCount * 100}%`,
    ease: "power2.inOut",
  });
}

let lastWheelEventTime = 0;
const throttleDelay = 500;

function throttleWheelHandler(event) {
  const currentTime = Date.now();
  if (currentTime - lastWheelEventTime >= throttleDelay) {
    lastWheelEventTime = currentTime;

    const direction = event.deltaY > 0 ? 1 : -1;
    scrollCount = (scrollCount + direction + totalPlanets) % totalPlanets;

    goToIndex(scrollCount);
    resetAutoRotation();
  }
}

window.addEventListener("wheel", throttleWheelHandler);

// Auto-rotation every 4s
let autoRotateInterval;
function startAutoRotation() {
  autoRotateInterval = setInterval(() => {
    scrollCount = (scrollCount + 1) % totalPlanets;
    goToIndex(scrollCount);
  }, 4000);
}

function resetAutoRotation() {
  clearInterval(autoRotateInterval);
  startAutoRotation();
}

startAutoRotation();

const clock = new THREE.Clock()

// Animate function
function animate() {
  requestAnimationFrame(animate);

  for (let i = 0; i < sphereMesh.length; i++) {
    const sphere = sphereMesh[i]
    sphere.rotation.y += clock.getElapsedTime() * 0.001;
  }

  controls.update();
  renderer.render(scene, camera);
}

// Handle window resize
window.addEventListener('resize', () => {
  renderer.setSize(window.innerWidth, window.innerHeight);
  camera.aspect = window.innerWidth / window.innerHeight;
  camera.updateProjectionMatrix();
});

animate();
