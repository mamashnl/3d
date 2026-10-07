<script>
import * as THREE from 'three';
import { GLTFLoader } from 'three/addons/loaders/GLTFLoader.js';

// const scene = new THREE.Scene();


async function design() {
  const loader = new THREE.GLTFLoader();
  await loader.load('./public/assets/pullroll.glb', (gltf) => {
    const model = gltf.scene;
    scene.add(model);
  }, undefined, (error) => {
    console.error('Gagal memuat GLB:', error);
  });
}



// const controls = new OrbitControls(camera, renderer.domElement);
const container = document.getElementById('pullroll');
const expandButton = document.getElementById('expand');
</script>

<template>
  <section id="center">
    <div class="wrapper">
        <div class="frame-design">
            <div id="pullroll">
                <canvas></canvas>
                Lorem ipsum dolor sit, amet consectetur adipisicing elit. Alias illo consequuntur porro nulla, laborum animi eum ea optio in, corrupti expedita commodi doloremque! Nihil porro dolores, consectetur officiis quas in.
            </div>
        </div>
        <button id="expand" type="button">Expand</button>
    </div>
  </section>
</template>
