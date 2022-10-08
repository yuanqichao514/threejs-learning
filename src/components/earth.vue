<template>
  <div>
    <canvas id="canvas"></canvas>
    <canvas id="earth"></canvas>
  </div>
</template>

<script>
import { controllers } from "dat.gui";
import * as THREE from "three";
import { OrbitControls } from "three/examples/jsm/controls/OrbitControls";
import { FirstPersonControls } from "three/examples/jsm/controls/FirstPersonControls";
import { TWEEN } from "three/examples/jsm/libs/tween.module.min.js";

export default {
    data() {
        return {
            start: {
                x: 0, y: 1000, z: 1000,
            }
        }
    },
    mounted() {
    // this.createStar();
    /**
     * scene
     */
    const scene = new THREE.Scene();
    let helper = new THREE.AxesHelper(100, 100)
    scene.background = new THREE.CubeTextureLoader().setPath('/src/assets/cubeMap/') // 这里的天空盒是地球出来视角变化的关键，不然就看不出效果
        .load([
          'nx.png',
          'ny.png',
          'nz.png',
          'px.png',
          'py.png',
          'pz.png'
        ])
    scene.add(helper)
    /**
     * camera
     */
    const camera = new THREE.PerspectiveCamera(
      45,
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
    const geometry = new THREE.SphereBufferGeometry(20, 32, 32);

    /**
     * material
     */
    // const material = new THREE.MeshBasicMaterial( { color: 0x00ff00 } );
    const material = new THREE.MeshBasicMaterial({
      //创建材质
      map: textureLoader.load("/src/assets/earth1.jpeg"), 
    });

    /**
     * sprite
     */

    // const spriteMaterial = new THREE.SpriteMaterial()
    // const sprite = new THREE.Sprite(spriteMaterial)
    // sprite.scale.set(40,40,50)
    // scene.add(sprite)

    /**
     * render
     */
    const renderer = new THREE.WebGLRenderer({
      canvas: document.getElementById("earth"),
      alpha: true,
    });
    renderer.setSize(window.innerWidth, window.innerHeight);
    // renderer.setPixelRatio(Math.min(window.devicePixelRatio, 2))
    document.body.appendChild(renderer.domElement);

    /**
     * cube
     */
    const text = new THREE.Mesh(geometry, material);
    scene.add(text);

    /**
     * controls
     */
    const controls = new OrbitControls(camera, renderer.domElement);
    controls.enableDamping = true;

    // camera.position.copy(this.start.x, this.start.y, this.start.z);
    camera.position.z = 2
    controls.update();
    /**
     * tween
     */
    let tween = new TWEEN.Tween(this.start).to({x: 100, y: 20, z: 100}, 3000).easing(TWEEN. Easing.Quadratic.InOut)
    let that = this
    tween.onUpdate((object) => {
        console.log(object); // 这里的object我们可以得到运动过程中的坐标
        camera.position.set(this.start.x, this.start.y, this.start.z)
        camera.lookAt(0, 0, 0) // 让相机始终从原点的角度去看
        controls.target.set(0,0,0)
        controls.update()
    })
    tween.start()
    /**
     * animate
     * 如果没有这个循环渲染，我们是看不见任何东西的
     */
    const animate = () => {
        // text.rotation.x += 0.01
        text.rotation.y += 0.01;
        requestAnimationFrame(animate);
        /**
         * 一定需要在animate中进行tween的更新
         */
        TWEEN.update()
        renderer.render(scene, camera);
    };
    animate();
  },
  methods: {
    createStar() {
      // 星空背景
      var canvas = document.getElementById("canvas"),
        ctx = canvas.getContext("2d"),
        w = (canvas.width = window.innerWidth),
        h = (canvas.height = window.innerHeight),
        hue = 217, // 角度
        stars = [],
        count = 0,
        maxStars = 1300; //星星数量

      var canvas2 = document.createElement("canvas"),
        ctx2 = canvas2.getContext("2d");
      canvas2.width = 100;
      canvas2.height = 100;
      var half = canvas2.width / 2,
        gradient2 = ctx2.createRadialGradient(half, half, 0, half, half, half); // 通过两个圆创建一个gradient，前三个坐标是第一个圆的坐标，后三个是第二个圆的坐标
      gradient2.addColorStop(0.025, "#CCC"); // 在gradient的不同位置添加颜色，0是开始，1是结束
      gradient2.addColorStop(0.1, "hsl(" + hue + ", 61%, 33%)"); // hsl（x,y,z）x是角度，y是饱和度，z是亮度， 后面还可能跟一个透明度也就是hsla
      gradient2.addColorStop(0.25, "hsl(" + hue + ", 64%, 6%)");
      gradient2.addColorStop(1, "transparent");

      ctx2.fillStyle = gradient2;
      ctx2.beginPath();
      ctx2.arc(half, half, half, 0, Math.PI * 2); // 绘制圆的状态（x,y,z,a,b）x,y是圆心，z是半径，a是开始绘制的角度，b是结束绘制的角度
      ctx2.fill();

      // End cache

      function random(min, max) {
        if (arguments.length < 2) {
          max = min;
          min = 0;
        }

        if (min > max) {
          var hold = max;
          max = min;
          min = hold;
        }

        return Math.floor(Math.random() * (max - min + 1)) + min;
      }

      function maxOrbit(x, y) {
        var max = Math.max(x, y),
          diameter = Math.round(Math.sqrt(max * max + max * max));
        return diameter / 2;
        //星星移动范围，值越大范围越小，
      }

      var Star = function () {
        this.orbitRadius = random(maxOrbit(w, h));
        this.radius = random(60, this.orbitRadius) / 8;
        //星星大小
        this.orbitX = w / 2;
        this.orbitY = h / 2;
        this.timePassed = random(0, maxStars);
        this.speed = random(this.orbitRadius) / 50000;
        //星星移动速度
        this.alpha = random(2, 10) / 10;

        count++;
        stars[count] = this;
      };

      Star.prototype.draw = function () {
        var x = Math.sin(this.timePassed) * this.orbitRadius + this.orbitX,
          y = Math.cos(this.timePassed) * this.orbitRadius + this.orbitY,
          twinkle = random(10);

        if (twinkle === 1 && this.alpha > 0) {
          this.alpha -= 0.05;
        } else if (twinkle === 2 && this.alpha < 1) {
          this.alpha += 0.05;
        }

        ctx.globalAlpha = this.alpha;
        ctx.drawImage(
          canvas2,
          x - this.radius / 2,
          y - this.radius / 2,
          this.radius,
          this.radius
        );
        this.timePassed += this.speed;
      };

      for (var i = 0; i < maxStars; i++) {
        new Star();
      }

      function animation() {
        // console.log(111,ctx);
        ctx.globalCompositeOperation = "source-over"; // globalCompositeOperation是合成操作，主要是两个canvas之间的合成
        ctx.globalAlpha = 0.5; //尾巴
        ctx.fillStyle = "hsla(" + hue + ", 64%, 6%, 2)";
        ctx.fillRect(0, 0, w, h);

        ctx.globalCompositeOperation = "lighter";
        for (var i = 1, l = stars.length; i < l; i++) {
          stars[i].draw();
        }

        window.requestAnimationFrame(animation);
      }

      animation();
    },
  },
};
</script>

<style>
#earth {
  z-index: 1;
  margin: 0 auto;
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
}
div {
  position: relative;
}
</style>