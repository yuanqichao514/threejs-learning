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
    const boxGeometry = new THREE.BoxBufferGeometry(25.6,25.6,25.6)
    /**
     * material
     */
    let t1 = textureLoader.load('/src/assets/cubeMap/nx.png')
    let t2 = textureLoader.load('/src/assets/cubeMap/px.png')
    let t3 = textureLoader.load('/src/assets/cubeMap/ny.png')
    let t4 = textureLoader.load('/src/assets/cubeMap/py.png')
    let t5 = textureLoader.load('/src/assets/cubeMap/nz.png')
    let t6 = textureLoader.load('/src/assets/cubeMap/pz.png')
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
      map: t4
    })
    let t5Material = new THREE.MeshBasicMaterial({
      map: t5
    })
    let t6Material = new THREE.MeshBasicMaterial({
      map: t6
    })
    let boxMaterial = [
      t1Material,
      t2Material,
      t3Material,
      t4Material,
      t5Material,
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