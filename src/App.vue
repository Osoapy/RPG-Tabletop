<template>
  <div id="app">
    <div ref="container" class="three-container"></div>
    <button @click="addImage" class="add-button">+</button>
  </div>
</template>

<script>
import { onMounted, ref } from 'vue';
import * as THREE from 'three';
import Swal from 'sweetalert2';

export default {
  name: 'App',
  setup() {
    const container = ref(null);
    let scene, camera, renderer;
    let objects = [];
    let isDraggingCamera = false;
    let isDraggingImage = false;
    let draggedObject = null;
    let previousMousePosition = { x: 0, y: 0 };

    onMounted(() => {
      // Scene
      scene = new THREE.Scene();

      // Camera (static, always looking in the same direction)
      camera = new THREE.PerspectiveCamera(75, window.innerWidth / window.innerHeight, 0.1, 1000);
      camera.position.set(0, 5, 61); // initial position

      // Renderer
      renderer = new THREE.WebGLRenderer({ antialias: true });
      renderer.setSize(window.innerWidth, window.innerHeight);
      renderer.shadowMap.enabled = true; // Enable shadow maps
      renderer.outputEncoding = THREE.sRGBEncoding; // Ensure the renderer outputs in sRGB color space
      container.value.appendChild(renderer.domElement);

      // Ambient light
      const ambientLight = new THREE.AmbientLight(0x404040);
      scene.add(ambientLight);

      // Directional light
      const directionalLight = new THREE.DirectionalLight(0xffffff, 1);
      directionalLight.position.set(5, 5, 5).normalize();
      directionalLight.castShadow = true; // Enable shadows for the light
      scene.add(directionalLight);

      // Load map texture
      const textureLoader = new THREE.TextureLoader();
      const texture = textureLoader.load('assets/map.webp');
      texture.colorSpace = THREE.SRGBColorSpace; // Set the color space to sRGB

      // Create material with the texture
      const material = new THREE.MeshBasicMaterial({ map: texture });

      // Create map geometry (a thin plane)
      const geometry = new THREE.BoxGeometry(100, 50, 0.2);
      const map = new THREE.Mesh(geometry, material);
      map.position.set(0, 5, 0);
      map.receiveShadow = true; // Map receives shadows
      scene.add(map);

      // Zoom control
      window.addEventListener('wheel', (event) => {
        const zoomSpeed = 0.1;
        camera.position.z += event.deltaY * zoomSpeed;
        camera.position.z = Math.max(10, Math.min(150, camera.position.z));
      });

      // Animation function
      function animate() {
        renderer.render(scene, camera);
        requestAnimationFrame(animate);
      }

      animate();

      // Enable dragging for all objects
      enableDragging();
    });

    const addImage = () => {
      Swal.fire({
        title: 'Escolha uma imagem',
        input: 'file',
        inputAttributes: {
          accept: 'image/*'
        },
        showCancelButton: true
      }).then((result) => {
        if (result.value) {
          const file = result.value;
          const reader = new FileReader();

          reader.onload = (e) => {
            const textureLoader = new THREE.TextureLoader();
            textureLoader.load(e.target.result, (texture) => {
              texture.colorSpace = THREE.SRGBColorSpace; // Set the color space to sRGB
              const material = new THREE.MeshBasicMaterial({ map: texture, transparent: true });
              const geometry = new THREE.PlaneGeometry(3, 3);
              const imagePlane = new THREE.Mesh(geometry, material);
              imagePlane.position.set(0, 5, 0.1); // Set z position to 0.1 to make it appear on the map
              
              scene.add(imagePlane);

              objects.push(imagePlane);
            });
          };
          reader.readAsDataURL(file);
        }
      });
    };

    const enableDragging = () => {
      const onMouseDown = (event) => {
        const raycaster = new THREE.Raycaster();
        const mouse = new THREE.Vector2();

        mouse.x = (event.clientX / window.innerWidth) * 2 - 1;
        mouse.y = -(event.clientY / window.innerHeight) * 2 + 1;

        raycaster.setFromCamera(mouse, camera);
        const intersects = raycaster.intersectObjects(objects);

        if (intersects.length > 0) {
          isDraggingImage = true;
          draggedObject = intersects[0].object;
          previousMousePosition = { x: event.clientX, y: event.clientY };
        } else {
          isDraggingCamera = true;
          previousMousePosition = { x: event.clientX, y: event.clientY };
        }
      };

      const onMouseMove = (event) => {
        if (isDraggingImage && draggedObject) {
          const deltaX = (event.clientX - previousMousePosition.x) * 0.03;
          const deltaY = (event.clientY - previousMousePosition.y) * 0.03;

          // Move the image on the screen
          draggedObject.position.x += deltaX;
          draggedObject.position.y -= deltaY;

          previousMousePosition = { x: event.clientX, y: event.clientY };
        } else if (isDraggingCamera) {
          // Move the camera horizontally (X axis) and vertically (Y axis)
          const deltaX = (event.clientX - previousMousePosition.x) * 0.1;
          const deltaY = (event.clientY - previousMousePosition.y) * 0.1;

          camera.position.x -= deltaX;
          camera.position.y += deltaY; 

          previousMousePosition = { x: event.clientX, y: event.clientY };
        }
      };

      const onMouseUp = () => {
        isDraggingImage = false;
        isDraggingCamera = false;
        draggedObject = null;
      };

      window.addEventListener('mousedown', onMouseDown);
      window.addEventListener('mousemove', onMouseMove);
      window.addEventListener('mouseup', onMouseUp);
    };

    return {
      container,
      addImage
    };
  }
};
</script>

<style scoped>
.three-container {
  width: 100%;
  height: 100vh;
}

.add-button {
  position: absolute;
  top: 20px;
  right: 20px;
  width: 40px;
  height: 40px;
  font-size: 24px;
  background: #007bff;
  color: white;
  border: none;
  border-radius: 50%;
  cursor: pointer;
}
</style>
