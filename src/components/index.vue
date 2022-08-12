<template>
  <canvas id="webgl"></canvas>
</template>

<script>
import * as THREE from 'three'
import gsap from 'gsap'
import { render } from 'vue';
import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls'; // 可以使得相机围绕目标进行轨道运动

export default {
  methods: {},

  mounted() {
    // Cursor
    const cursor = {
        x: 0,
        y: 0
    }
    window.addEventListener('mousemove', (event) => {
       cursor.x = event.clientX / window.innerWidth - 0.5
       cursor.y = -(event.clientY / window.innerHeight - 0.5)
    })

    // Scene
    const scene = new THREE.Scene();
    //  透视相机
    const camera = new THREE.PerspectiveCamera(
      75,
      window.innerWidth / window.innerHeight,
      0.1,
      1000
    ); // (FOV:field of view-degrees, aspect ratio, near, far)
    // 正交相机
    let aspect =  window.innerWidth / window.innerHeight
     const camera1 = new THREE.OrthographicCamera(
     -1 * aspect, 1  * aspect, 1, -1, 0.1, 1000
    ); 
    // left — 摄像机视锥体左侧面。
    // right — 摄像机视锥体右侧面。
    // top — 摄像机视锥体上侧面。
    // bottom — 摄像机视锥体下侧面。
    // near — 摄像机视锥体近端面。
    // far — 摄像机视锥体远端面。
    

    const renderer = new THREE.WebGLRenderer({
      canvas: document.getElementById("webgl"), // 如果我们不指定一个canvas，renderer会自己创建一个camera
    });
    renderer.setSize(window.innerWidth, window.innerHeight);
    renderer.setPixelRatio(Math.min(window.devicePixelRatio, 2)) // 让物体适应屏幕分辨率，来达到光滑减少边缘锯齿的效果，最好是不要超过2
    document.body.appendChild(renderer.domElement);

    window.addEventListener('resize', () => {
        renderer.setSize(window.innerWidth, window.innerHeight);
        renderer.setPixelRatio(Math.min(window.devicePixelRatio, 2))
        camera.aspect = window.innerWidth / window.innerHeight
        camera.updateProjectionMatrix() // 首次计算的时候使用render，后续更新使用updateProjectionMatrix
    })

    window.addEventListener('dblclick', () => { // 双击进入全屏
        if(!document.fullscreenElement) {
            document.querySelector('canvas').requestFullscreen()
        }else {
            document.exitFullscreen()
        }
    })

    const geometry = new THREE.BoxGeometry(1, 1, 1, 3, 3, 3); // 几何体型

    // 自定义形状
    // const geometry = new THREE.Geometry() // 在124版本之后Geometry已经被移除了，配合课程使用的是118版本
    // // 创建顶点
    // const vertex1 = new THREE.Vector3(0,0,0)
    // geometry.vertices.push(vertex1)

    // const vertex2 = new THREE.Vector3(1,0,0)
    // geometry.vertices.push(vertex2)

    // const vertex3 = new THREE.Vector3(0,1,0)
    // geometry.vertices.push(vertex3)

    // const face = new THREE.Face3(0, 1, 2) // 这里的0，1，2是顶点在vertices中的索引，所以用别的数字只能看见一条线，相当于三个点成一个面
    // geometry.faces.push(face)

    // 创建多面体
    // for(let i = 0; i < 50; i++) {
    //     for(let j = 0; j < 50; j++) {
    //         geometry.vertices.push(new THREE.Vector3(
    //             (Math.random() - 0.5 ) * 3,
    //             (Math.random() - 0.5 ) * 3,
    //             (Math.random() - 0.5 ) * 3
    //         ))
    //     }
    //     const verticesIndex = i * 3
    //     geometry.faces.push(new THREE.Face3(verticesIndex, verticesIndex+1, verticesIndex+2))
    // }

    // const positionArray = new Float32Array(9)

    // positionArray[0] = 0
    // positionArray[1] = 0
    // positionArray[2] = 0

    // positionArray[3] = 0
    // positionArray[4] = 1
    // positionArray[5] = 0

    // positionArray[6] = 1
    // positionArray[7] = 0
    // positionArray[8] = 0

    // const positionAttribute = new THREE.BufferAttribute(positionArray, 3)

    // const geometry = new THREE.BufferGeometry()
    // geometry.setAttribute('position', positionAttribute)


    // const geometry = new THREE.BufferGeometry()

    // const count = 1
    // const positionArray = new Float32Array(count *3 *3)

    // for(let i = 0; i <  count *3*3; i++) {
    //     positionArray[i] = Math.random() 
    // }

    // const positionAttribute = new THREE.BufferAttribute(positionArray, 3)
    // geometry.setAttribute('position', positionAttribute)

    const material = new THREE.MeshBasicMaterial({ color: 0x33ff22, wireframe: true }); // 颜色
    const cube = new THREE.Mesh(geometry, material); // 立方体
    const cube1 = new THREE.Mesh(
      new THREE.BoxGeometry(1, 2, 1),
      new THREE.MeshBasicMaterial({ color: 0xe82255 })
    );

    // const group = new THREE.Group();
    // group.add(cube, cube1);
    // scene.add(group); // 物体将会被添加到(0,0,0)坐标。但将使得摄像机和立方体彼此在一起
    scene.add(cube)
    // cube.position.x = 0.1
    // cube.position.y = -0.1
    // cube.position.z = 1


    // Controls
    const controls = new OrbitControls(camera, renderer.domElement)
    controls.enableDamping = true // 启用惯性
    // controls.target.y = 2

    camera.position.z = 2; // 将摄像机向外移动一些， 实际上这个z值是我们从上看这个立方体的距离，如果不设置话，我们就什么也看不见
    controls.update()
    // camera.lookAt(new THREE.Vector3(0, 2,0)) // 提供一个向量，作为camera的视角出发点
    camera1.position.z = 1


    // const axesHelper = new THREE.AxesHelper(5);
    // scene.add(axesHelper);

    let time = new Date();

    const clock = new THREE.Clock();

    // gsap.to(cube.position, { duration: 1, delay: 1, x: 3})

    function animate() {
    
        //  更改camera的角度: 我们使用OrbitControl 来更好的控制
        // camera.position.x = Math.sin(cursor.x * Math.PI * 2) * 3
        // camera.position.z = Math.cos(cursor.x * Math.PI * 2) * 3
        // camera.position.y = Math.cos(cursor.y * Math.PI * 2) * 3
        // camera.lookAt(cube.position)

        requestAnimationFrame(animate); // 相比于setInterval的优点： 最重要的一点或许就是当用户切换到其它的标签页时，它会暂停，因此不会浪费用户宝贵的处理器资源，也不会损耗电池的使用寿命。

        // 每次运动都要更新controls
        controls.update()

        // cube.rotation.reorder('YXZ') // reorder是用来定义更新顺序的
        //   cube.rotation.x += 0.01
        cube.rotation.y += 0.01;
        // cube.scale.x += 0.01
        cube1.rotation.x -= 0.01;
        cube1.rotation.y -= 0.01
        // console.log(cube.position.length()); // 计算从(0, 0, 0) 到 (x, y, z)的欧几里得长度 （Euclidean length，即直线长度），这是基于三维向量，我们给cube一个三维坐标，如果没有给的话，这个距离就是0
        renderer.render(scene, camera);

        // let currentTime = new Date()
        // let deltaTime = currentTime - time
        // time = currentTime

        // console.log(deltaTime); // 这里我们计算出来的差值可以让我们的物体运动的速度始终和浏览器的帧率保持一致,可以达到兼容的效果

        // console.log(clock.getElapsedTime()); // 定时器，从动画开始执行的时间
    }
    animate();

    // 绘制线条
    // const scene = new THREE.Scene()

    // const renderer = new THREE.WebGLRenderer()
    // renderer.setSize(window.innerWidth, window.innerHeight)
    // document.body.appendChild(renderer.domElement)

    // const camera = new THREE.PerspectiveCamera(45, window.innerWidth/window.innerHeight, 1, 500)
    // camera.position.set(0, 0, 100)
    // camera.lookAt(0,0,0)

    // const material = new THREE.LineBasicMaterial({color: 0x0000ff})

    // const points = []
    // points.push( new THREE.Vector3( -10, 0, 0 ) )
    // points.push( new THREE.Vector3( 0, 10, 0 ) )
    // points.push( new THREE.Vector3( 10, 0, 0 ) )

    // const geometry = new THREE.BufferGeometry().setFromPoints(points)

    // const line = new THREE.Line(geometry, material)

    // scene.add(line)

    // renderer.render(scene, camera)
  },
};
</script>

<style></style>
