<template>
  <div id="app">
    <div ref="container" class="three-container"></div>
  </div>
</template>

<script>
import { onMounted, ref } from 'vue';
import * as THREE from 'three';
import { MTLLoader } from 'three/examples/jsm/loaders/MTLLoader.js';
import { OBJLoader } from 'three/examples/jsm/loaders/OBJLoader.js';

export default {
  name: 'App',
  setup() {
    const container = ref(null);

    onMounted(() => {
      // Cena
      const scene = new THREE.Scene();
      
      // Câmera
      const camera = new THREE.PerspectiveCamera(75, window.innerWidth / window.innerHeight, 0.1, 1000);
      camera.position.set(0, 8, 30);  // Ajuste a posição da câmera

      // Renderizador
      const renderer = new THREE.WebGLRenderer();
      renderer.setSize(window.innerWidth, window.innerHeight);
      container.value.appendChild(renderer.domElement);

      // Luz ambiente para iluminar o modelo
      const ambientLight = new THREE.AmbientLight(0x404040); // Luz suave
      scene.add(ambientLight);

      // Luz direcional para melhor iluminação
      const directionalLight = new THREE.DirectionalLight(0xffffff, 1);
      directionalLight.position.set(5, 5, 5).normalize();
      scene.add(directionalLight);

      // Carregar materiais
      const mtlLoader = new MTLLoader();
      const objLoader = new OBJLoader();

      // Caminho para os arquivos MTL e OBJ
      const mtlPath = 'src/assets/Table_And_Chairs.mtl';  // Ajuste o caminho
      const objPath = 'src/assets/Table_And_Chairs.obj';  // Ajuste o caminho

      mtlLoader.load(mtlPath, function (materials) {
        materials.preload(); // Precarregar materiais

        // Associar materiais ao loader OBJ
        objLoader.setMaterials(materials);

        // Carregar o modelo OBJ
        objLoader.load(objPath, function (obj) {
          obj.position.set(0, 0, 0); // Posicione o modelo
          obj.scale.set(0.3, 0.3, 0.3); // Ajuste a escala do modelo
          scene.add(obj);
        }, undefined, function (error) {
          console.error('Erro ao carregar o OBJ:', error);
        });
      }, undefined, function (error) {
        console.error('Erro ao carregar o arquivo MTL:', error);
      });

      // Função de animação
      function animate() {
        renderer.render(scene, camera);
        requestAnimationFrame(animate);
      }

      animate();
    });

    return {
      container
    };
  }
};
</script>

<style scoped>
.three-container {
  width: 100%;
  height: 100vh;
}
</style>
