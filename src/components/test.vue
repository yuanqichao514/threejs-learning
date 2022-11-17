<template>
  <div>
    <table>
      <tr>
        <th>截面</th>
        <th>原始图</th>
        <th>合成图</th>
        <th>编辑</th>
      </tr>
      <tr>
        <td style="width: 100px;">正面图</td>
        <td>
          <img :src="frontImage" class="frontImage" ref="fImg" />
          <button @click="frontUpload">选择</button>
        </td>
        <td rowspan="2">
          <img :src="generateImg" alt="">
          <button id="exportLink" @click="generate">合成</button>
        </td>
        <td rowspan="2"  style="width: 100px;">
          <button id="exportLink" @click="output">导出</button>
        </td>
      </tr>
      <tr>
        <td style="width: 100px;">左侧图</td>
        <td>
          <img :src="leftImage" class="leftImage" ref="lImg" />
          <button @click="leftUpload">选择</button>
        </td>
      </tr>
    </table>
    <input
      type="file"
      ref="upImg"
      accept="image/jpg,image/png,image/jpeg,image/bmp"
      id="onlyInput"
      hidden
      @change="changeInput($event)"
    />
  </div>
</template>

<script>
import * as THREE from "three";
import { OrbitControls } from "three/examples/jsm/controls/OrbitControls";
let scene= null,
      camera= null,
      renderer= null,
      textureLoader= null,
      boxGeometry= null,
      controls = null,
      boxMaterial= []
export default {
  data() {
    return {
      frontImage: "",
      leftImage: "",
      generateImg: "",
      fWidth: '',
      fHeight: '',
      lWidth: '',
      lHeight: '',
      t1: '',
      t2: '',
      type: 0, // 0:front, 1:left
    };
  },
  mounted() {
      this.initFirst()
  },
  methods: {
    animate() {
      /**
       * animate
       * 如果没有这个循环渲染，我们是看不见任何东西的
       */
      requestAnimationFrame(this.animate);
      renderer.render(scene, camera);
    },
    initFirst() {
      /**
       * scene
       */
      scene = new THREE.Scene();
      /**
       * camera
       */
      camera = new THREE.PerspectiveCamera(
        45,
        window.innerWidth / window.innerHeight,
        0.1,
        1000
      );
      camera.position.z = 500;
      // camera.position.y = 150
      camera.position.x = -400;
      /**
       * textures
       */
      textureLoader = new THREE.TextureLoader();

      /**
       * render
       */
      renderer = new THREE.WebGLRenderer({
        antialias: true,
        alpha: true,
      });
      document.body.appendChild(renderer.domElement);
    },
    initControls() {
      controls = new OrbitControls(camera, renderer.domElement);
      controls.enableDamping = true;
    },
    initGeometry() {
      /**
       * geometry
       */
      boxGeometry = new THREE.BoxBufferGeometry(this.fWidth/5, this.fHeight/5, this.lWidth/5);

      renderer.setSize(this.fWidth, this.fHeight); // 渲染的宽度应该和相机的aspect参数比例一致，这样就不会变形了
    },
    initMaterial() {
      /**
       * material
       */
      let t1 = this.t1;
      let t2 = this.t2;
      let t1Material = new THREE.MeshBasicMaterial({
        map: t1,
      });
      let t2Material = new THREE.MeshBasicMaterial({
        color: 0xffffff,
      });
      let t3Material = new THREE.MeshBasicMaterial({
        map: t2,
      });
      let t4Material = new THREE.MeshBasicMaterial({
        color: 0x000000,
      });
      let t5Material = new THREE.MeshBasicMaterial({
        color: 0x000000,
      });
      let t6Material = new THREE.MeshBasicMaterial({
        color: 0x000000,
      });
      boxMaterial = [
        t4Material,
        t3Material,
        t2Material,
        t5Material,
        t1Material,
        t6Material,
      ];
    },
    initCube() {
      /**
       * cube
       */
      const text = new THREE.Mesh(boxGeometry, boxMaterial);
      scene.add(text);
    },
    frontUpload() {
      this.type = 0;
      this.$refs.upImg.click();
    },
    leftUpload() {
      this.type = 1;
      this.$refs.upImg.click();
    },
    changeInput(e) {
      console.log(e);
      if (this.type == 0) {
        this.loadImage(e, this.$refs.fImg);
      } else {
        this.loadImage(e, this.$refs.lImg);
      }
    },
    loadImage(event, imgElement) {
      const reader = new FileReader();
      reader.onload = (e) => {
        imgElement.src = e.target.result;
        const img = new Image()
        img.onload = (e) => {
          if(this.type == 0) {
            this.fWidth = img.width
            this.fHeight = img.height
          }else {
            this.lWidth = img.width
            this.lHeight = img.height
          } 
        }
        img.src = e.target.result
        if(this.type == 0) {
            this.t1 = textureLoader.load(e.target.result)
        }else {
            this.t2 = textureLoader.load(e.target.result)
        }
      };
      reader.readAsDataURL(event.target.files[0]);
    },
    generate() {
      this.initControls()
      this.initGeometry()
      this.initMaterial()
      this.initCube()
      this.animate()
      const dataURL = renderer.domElement.toDataURL("image/png");
      this.generateImg = dataURL
    },
    output() {
      renderer.render(scene, camera);
      const dataURL = renderer.domElement.toDataURL("image/png");

      let a = document.createElement("a"); // Create a temporary anchor.
      a.href = dataURL;
      a.download = "export.png";
      a.click(); // Perform the navigation action to trigger the download.
    },
  },
};
</script>

<style scoped>
img {
  width: 300px;
}
td {
  border: 1px solid #000;
  height: 400px;
  width: 400px;
}
</style>
