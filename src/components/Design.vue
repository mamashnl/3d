<script setup>
import * as THREE from 'three';
import { GLTFLoader } from 'three/addons/loaders/GLTFLoader.js';
import { OrbitControls } from 'three/addons/controls/OrbitControls.js';
import { onBeforeUnmount, onMounted, ref } from 'vue';

const container = ref(null);
const isExpanded = ref(false);
const isModelReady = ref(false);
let renderer;
let resizeObserver;
let animationFrame;
let parts = [];
let controls;

function renderScene(scene, camera) {
    if (!container.value || !renderer) return;
    const { width, height } = container.value.getBoundingClientRect();
    if (!width || !height) return;
    renderer.setSize(width, height, false);
    camera.aspect = width / height;
    camera.updateProjectionMatrix();
    renderer.render(scene, camera);
}

function toggleExplodedView() {
    if (!isModelReady.value) return;
    isExpanded.value = !isExpanded.value;
    cancelAnimationFrame(animationFrame);

    const moves = parts.map(({ object, baseX, offset }) => ({
        object,
        from: object.position.x,
        to: baseX + (isExpanded.value ? offset : 0),
    }));
    const startedAt = performance.now();
    const duration = 500;

    const animate = (now) => {
        const progress = Math.min((now - startedAt) / duration, 1);
        const eased = 1 - (1 - progress) ** 3;
        moves.forEach(({ object, from, to }) => {
            object.position.x = THREE.MathUtils.lerp(from, to, eased);
        });
        if (container.value && renderer) {
            renderer.render(scene, camera);
        }
        if (progress < 1) animationFrame = requestAnimationFrame(animate);
    };

    animate(startedAt);
}

let scene;
let camera;

onMounted(() => {
    if (!container.value) return;

    scene = new THREE.Scene();
    camera = new THREE.PerspectiveCamera(45, 10, 0.1, 1000);
    renderer = new THREE.WebGLRenderer({ antialias: true, alpha: true });
    renderer.setPixelRatio(Math.min(window.devicePixelRatio, 2));
    renderer.outputColorSpace = THREE.SRGBColorSpace;
    renderer.toneMapping = THREE.ACESFilmicToneMapping;
    container.value.appendChild(renderer.domElement);

    scene.add(new THREE.HemisphereLight(0xffffff, 0xffffff, 1.1));
    const keyLight = new THREE.DirectionalLight(0xffffff, 1.0);
    keyLight.position.set(4, 6, 5);
    scene.add(keyLight);

    resizeObserver = new ResizeObserver(() => renderScene(scene, camera));
    resizeObserver.observe(container.value);

    const loader = new GLTFLoader();
    loader.load('/assets/scene.glb', ({ scene: model }) => {
        model.traverse((object) => {
            if (!object.isMesh) return;
            const materials = Array.isArray(object.material) ? object.material : [object.material];
            materials.forEach((material) => {
                if (!(material instanceof THREE.MeshStandardMaterial)) return;
                material.color.set(0xffffff);
                material.metalness = Math.min(material.metalness, 0.85);
                material.roughness = Math.max(material.roughness, 0.10);
            });
        });

        const modelBounds = new THREE.Box3().setFromObject(model);
        const modelCenter = modelBounds.getCenter(new THREE.Vector3());
        const modelSize = modelBounds.getSize(new THREE.Vector3());
        const modelSpan = Math.max(modelSize.x, modelSize.y, modelSize.z);
        const partObjects = model.children;

        parts = partObjects.map((object, index) => {
            const bounds = new THREE.Box3().setFromObject(object);
            const partCenterX = bounds.getCenter(new THREE.Vector3()).x;
            const distanceFromCenter = partCenterX - modelCenter.x;
            const direction = distanceFromCenter === 0
                ? (index % 2 === 0 ? 1 : -1)
                : Math.sign(distanceFromCenter);
            const name = object.name.toLowerCase();
            const movementFactor = name.includes('barjoin')
                ? 0.15
                : name.includes('barmiddle')
                    ? 0.20
                    : name.includes('barmounting')
                        ? 0.25
                        : 0.0;

            return {
                object,
                baseX: object.position.x,
                offset: direction * Math.max(Math.abs(distanceFromCenter) * movementFactor, modelSpan * 0.035),
            };
        });

        model.position.sub(modelCenter);
        scene.add(model);

        const radius = modelSpan / 2;
        const distance = radius / Math.sin(THREE.MathUtils.degToRad(camera.fov / 2)) * 1;
        camera.position.set(-7, 5, 8);
        controls = new OrbitControls(camera, renderer.domElement);
        controls.target.set(0, 0, 0);
        controls.minDistance = distance * 0.4;
        controls.maxDistance = distance * 4;
        controls.addEventListener('change', () => renderScene(scene, camera));
        renderer.domElement.style.touchAction = 'none';

        isModelReady.value = true;
        renderScene(scene, camera);
    }, undefined, (error) => {
        console.error('Gagal memuat scene.glb:', error);
});
});

onBeforeUnmount(() => {
    cancelAnimationFrame(animationFrame);
    resizeObserver?.disconnect();
    controls?.dispose();
    renderer?.dispose();
});

</script>

<template>
  <section id="center">
    <div class="wrapper">
        <div class="frame-design">
            <div id="pullroll" ref="container"></div>
            <div class="footer-3d">
                <button id="expand" type="button" :disabled="!isModelReady" @click="toggleExplodedView">
                    <img class="button-icon" :src=" isExpanded ? '/assets/collapse.svg' : '/assets/expand.svg'" alt="" srcset="">
                    {{ isExpanded ?  'Collapse' : 'Expand' }}
                </button>
                <p>Gambar Interaktif: Klik dan gerakkan Mouse untuk memutar objek 3D ini!</p>
            </div>
        </div>
    </div>
  </section>
</template>
