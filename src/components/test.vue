<template>
  <div>
  </div>
</template>

<script>
import * as THREE from "three";
import { OrbitControls } from "three/examples/jsm/controls/OrbitControls";

export default {
    mounted() {
    /**
     * scene
     */
    const scene = new THREE.Scene();
    /**
     * camera
     */
    const camera = new THREE.PerspectiveCamera(
      75,
      window.innerWidth / window.innerHeight,
      0.1,
      1000
    );

    /**
     * textures
     */
    const textureLoader = new THREE.TextureLoader();
    /**
     * geometry
     */
    const boxGeometry = new THREE.BoxBufferGeometry(158.4,108.8,47.2)
    /**
     * material
     */
    let t1 = textureLoader.load('/src/assets/boxMap/front.jpeg')
    let t2 = textureLoader.load('/src/assets/boxMap/top.jpeg')
    let t3 = textureLoader.load('/src/assets/boxMap/left.jpeg')
    let t1Material = new THREE.MeshBasicMaterial({
      map: t1
    })
    let t2Material = new THREE.MeshBasicMaterial({
      map: t2
    })
    let t3Material = new THREE.MeshBasicMaterial({
      map: t3
    })
    let t4Material = new THREE.MeshBasicMaterial({
      color: 0x000000
    })
    let t5Material = new THREE.MeshBasicMaterial({
      color: 0x000000
    })
    let t6Material = new THREE.MeshBasicMaterial({
      color: 0x000000
    })
    let boxMaterial = [
      t4Material,
      t3Material,
      t2Material,
      t5Material,
      t1Material,
      t6Material
    ]

    /**
     * render
     */
    const renderer = new THREE.WebGLRenderer({
      antialias:true,
      alpha:true
    });
    renderer.setSize(window.innerWidth, window.innerHeight);
    document.body.appendChild(renderer.domElement);

    /**
     * cube
     */
    const text = new THREE.Mesh(boxGeometry, boxMaterial);
    scene.add(text);

    /**
     * controls
     */
    const controls = new OrbitControls(camera, renderer.domElement);
    controls.enableDamping = true;

    camera.position.z = 160

    /**
     * animate
     * 如果没有这个循环渲染，我们是看不见任何东西的
     */
    const animate = () => {
        requestAnimationFrame(animate);
        renderer.render(scene, camera);
    };
    animate();
  },
};
</script>