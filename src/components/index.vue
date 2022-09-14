<template>
  <canvas id="webgl"></canvas>
</template>

<script>
import * as THREE from 'three'
import gsap from 'gsap'
import { render } from 'vue';
import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls'; // 可以使得相机围绕目标进行轨道运动
import * as dat from 'dat.gui'

export default {
  methods: {},

  mounted() {
    /**
    * Debug
     */
    const gui = new dat.GUI()

    const parameters = {
      color: 0xff0000,
      spin: () => {
        gsap.to(cube.rotation, { y: cube.rotation.y + 10, duration: 1 })
      }
    }

    /**
     * Scene
     * @type {Scene}
     */
    const scene = new THREE.Scene();
    /**
     * 透视相机
     * @type {PerspectiveCamera}
     */
    const camera = new THREE.PerspectiveCamera(
        75,
        window.innerWidth / window.innerHeight,
        0.1,
        1000
    ); // (FOV:field of view-degrees, aspect ratio, near, far)
    /**
     * 正交相机
     * @type {OrthographicCamera}
     */
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

    /**
     * textures
     */
    // const image = new Image()
    //
    // const texture = new THREE.Texture(image)
    //
    // image.onload = () => {
    //   texture.needsUpdate = true
    //   // console.log(texture)
    // }
    //
    // image.src = '/src/assets/tiger.jpeg'

    const loadingManager = new THREE.LoadingManager()
    // loadingManager.onStart = () => {
    //   console.log('onStart')
    // }
    // loadingManager.onProgress = () => {
    //   console.log('onProgress')
    // }
    // loadingManager.onLoad = () => {
    //   console.log('onLoad')
    // }
    // loadingManager.onError = () => {
    //   console.log('onError')
    // }

    const textureLoader = new THREE.TextureLoader(loadingManager)
    const cubeTextureLoader = new THREE.CubeTextureLoader()
    const tigerTexture = textureLoader.load('/src/assets/tiger.jpeg')
    const woodTexture = textureLoader.load('/src/assets/Material_2080.webp')
    const matcapTexture = textureLoader.load('/src/static/textures/1.png')
    const environmentMapTexture = cubeTextureLoader
        .setPath('/src/assets/cubeMap/')
        .load([
          'nx.png',
          'ny.png',
          'nz.png',
          'px.png',
          'py.png',
          'pz.png'
        ]) // cube的图片必须是长宽相等的正方形
    // 以下都是对单独一个面的纹理的操作
    // textureLoader.repeat.x = 4 // 在x方向上重复的次数
    // textureLoader.repeat.y = 2 // 在y方向上重复的次数
    // textureLoader.wrapS = THREE.RepeatWrapping
    // textureLoader.wrapT = THREE.MirroredRepeatWrapping
    //
    // textureLoader.offset.x = 0.5 // 偏移量
    // textureLoader.rotation = Math.PI/2 // 旋转量
    // const texture2 = textureLoader.load('/src/assets/tiger2.jpg')
    textureLoader.minFilter = THREE.NearestFilter // 当纹素覆盖小于一个像素时，贴图的采样方式
    textureLoader.generateMipmaps = false // 使用上面这种赋值时我们不需要创建mipmaps，也就是适应各种像素状态下的面
    textureLoader.magFilter = THREE.NearestFilter // 当纹素覆盖大于一个像素时，贴图的采样方式

    /**
     * Fonts
     */
    const fontLoader = new THREE.FontLoader()

    fontLoader.load(
        '/src/static/fonts/helvetiker_bold.typeface.json',
        (font) => {
          const textGeometry = new THREE.TextBufferGeometry(
              'Hello Three.js',
              {
                font,
                size: 0.5,
                height: 0.2,
                curveSegments: 4,
                bevelEnabled: true,
                bevelThickness: 0.03,
                bevelSize: 0.02,
                bevelOffset: 0,
                bevelSegments: 2
              }
          )
          // textGeometry.computeBoundingBox()
          // console.log(textGeometry.boundingBox)
          // 手动移动中心点到文字正中心
          // textGeometry.translate(
          //      - (textGeometry.boundingBox.max.x - 0.02) * 0.5,
          //      - (textGeometry.boundingBox.max.y - 0.02) * 0.5,
          //      - (textGeometry.boundingBox.max.z - 0.03) * 0.5,
          // )
          // or
          textGeometry.center()

          const textMaterial = new THREE.MeshMatcapMaterial({matcap: matcapTexture})
          const text = new THREE.Mesh(textGeometry, textMaterial)
          scene.add(text)

          console.time('donuts')
          
          const donutGeometry = new THREE.TorusBufferGeometry(0.3,0.2,20,45)
          const donutMaterial = new THREE.MeshMatcapMaterial({matcap: matcapTexture})

          for (let i = 0; i < 100; i++) {
            const donut = new THREE.Mesh(donutGeometry, donutMaterial)

            donut.position.x = (Math.random() - 0.5) * 10
            donut.position.y = (Math.random() - 0.5) * 10
            donut.position.z = (Math.random() - 0.5) * 10

            donut.rotation.x = Math.random() * Math.PI
            donut.rotation.y = Math.random() * Math.PI

            const scale = Math.random()
            donut.scale.set(scale,scale,scale)
            scene.add(donut)
          }

          console.timeEnd('donuts')
        }
    )

    /**
     * Objects
    /**
     * MeshBasicMaterial
     * @type {Mesh<MeshBasicMaterial, *>}
     * @description 不设置的话就是白色
     */
    // const material = new THREE.MeshBasicMaterial()
    // material.map = textureLoader
    // material.color.set('#e82255')
    // or
    // material.color = new THREE.Color(0xe82255)
    // material.wireframe = true
    // opacity和transparent搭配使用，opacity才会生效，即使是texture中的透明度也是一样
    // material.opacity = 0.8
    // material.transparent = true
    // material.side = THREE.DoubleSide
    /**
     * MeshNormalMaterial
     * @type {Mesh<MeshNormalMaterial, *>}
     * @description 自动获取颜色，看起来是彩色的
     */
    // const material = new THREE.MeshNormalMaterial()
    // material.flatShading = true // 使表面不在光滑，变成顶点之间的矩形连线
    /**
     * MeshMatcapMaterial
     * @type {Mesh<MeshMatcapMaterial, *>}
     * @description 材质捕捉，会根据运动情况来改变材料，模拟灯光的照射情况，颜色取自你提供的texture
     */
    // const material = new THREE.MeshMatcapMaterial()
    // material.matcap = woodTexture
    /**
     * MeshDepthMaterial
     * @type {Mesh<MeshDepthMaterial, *>}
     */
    // const material = new THREE.MeshDepthMaterial()
    /**
     * MeshLambertMaterial
     * @type {MeshLambertMaterial}
     */
    // const material = new THREE.MeshLambertMaterial()
    /**
     * MeshPhongMaterial
     * @type {MeshPhongMaterial}
     */
    // const  material = new THREE.MeshPhongMaterial()
    // material.shininess = 100
    // material.specular = new THREE.Color(0xfff222)
    /**
     * MeshToonMaterial
     * @type {MeshToonMaterial}
     */
    // const material = new THREE.MeshToonMaterial()
    /**
     * MeshStandardMaterial
     * @type {MeshStandardMaterial}
     */
    const material = new THREE.MeshStandardMaterial()
    material.metalness = 0.7
    material.roughness = 0.2
    material.envMap = environmentMapTexture
    // material.map = tigerTexture
    // material.aoMap = woodTexture // 该纹理的红色通道用作环境遮挡贴图
    // material.aoMapIntensity = 1 // 环境遮挡效果的强度。默认值为1。零是不遮挡效果。
    // gui.add(material, 'metalness').min(0).max(1).step(0.0001)
    // gui.add(material, 'roughness').min(0).max(1).step(0.0001)

    /**
     * Light
     * @type {Mesh<SphereBufferGeometry, Mesh<MeshDepthMaterial, *>>}
     */
    /**
     * AmbientLight
     * @type {AmbientLight}
     */
    const ambientLight = new THREE.AmbientLight(0xffffff, 0.5)
    scene.add(ambientLight)
    /**
     * PointLight
     * @type {PointLight}
     */
    const pointLight = new THREE.PointLight(0xffffff, 0.5)
    pointLight.position.x = 2
    pointLight.position.y = 3
    pointLight.position.z = 4
    scene.add(pointLight)


    // Mesh构成网格，Mesh的构成又是由立方体和材料组成的，然后在Scene中添加Mesh使其能够在camera中被看见
    const sphere = new THREE.Mesh(
        new THREE.SphereBufferGeometry(0.5, 16,16),
        material
    )
    sphere.geometry.setAttribute('uv2', new THREE.BufferAttribute(sphere.geometry.attributes.uv.array, 2)) // uv2：网格的第二套纹理坐标
    sphere.position.x = -1.5
    const plane = new THREE.Mesh(
        new THREE.PlaneBufferGeometry(1, 1),
        material
    )
    plane.geometry.setAttribute('uv2', new THREE.BufferAttribute(plane.geometry.attributes.uv.array, 2))
    const torus = new THREE.Mesh(
        new THREE.TorusBufferGeometry(0.3, 0.2, 16, 32),
        material
    )
    torus.geometry.setAttribute('uv2', new THREE.BufferAttribute(torus.geometry.attributes.uv.array, 2))
    torus.position.x = 1.5
    const cubeBox = new THREE.Mesh(
        new THREE.BoxBufferGeometry(1,1,1),
        material
    )
    cubeBox.position.x = -4.5
    // scene.add(sphere, plane, torus, cubeBox)
    /**
     * Base
     * @type {{x: number, y: number}}
     */
        // Cursor
    const cursor = {
        x: 0,
        y: 0
    }
    window.addEventListener('mousemove', (event) => {
       cursor.x = event.clientX / window.innerWidth - 0.5
       cursor.y = -(event.clientY / window.innerHeight - 0.5)
    })
    

    const renderer = new THREE.WebGLRenderer({
      canvas: document.getElementById("webgl"), // 如果我们不指定一个canvas，renderer会自己创建一个camera
    });
    renderer.setSize(window.innerWidth, window.innerHeight);
    renderer.setPixelRatio(Math.min(window.devicePixelRatio, 2)) // 让物体适应屏幕分辨率，来达到光滑减少边缘锯齿的效果，最好是不要超过2
    document.body.appendChild(renderer.domElement);

    // window.addEventListener('resize', () => {
    //     renderer.setSize(window.innerWidth, window.innerHeight);
    //     renderer.setPixelRatio(Math.min(window.devicePixelRatio, 2))
    //     camera.aspect = window.innerWidth / window.innerHeight
    //     camera.updateProjectionMatrix() // 首次计算的时候使用render，后续更新使用updateProjectionMatrix
    // })

    // window.addEventListener('dblclick', () => { // 双击进入全屏
    //     if(!document.fullscreenElement) {
    //         document.querySelector('canvas').requestFullscreen()
    //     }else {
    //         document.exitFullscreen()
    //     }
    // })

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

    // const material = new THREE.MeshBasicMaterial({  map: textureLoader }); // 颜色
    const cube = new THREE.Mesh(geometry, material); // 立方体
    const cube1 = new THREE.Mesh(
      new THREE.BoxGeometry(1, 2, 1),
      new THREE.MeshBasicMaterial({ color: 0xe82255 })
    );

    // const group = new THREE.Group();
    // group.add(cube, cube1);
    // scene.add(group); // 物体将会被添加到(0,0,0)坐标。但将使得摄像机和立方体彼此在一起
    // scene.add(cube)
    // cube.position.x = 0.1
    // cube.position.y = -0.1
    // cube.position.z = 1

    // gui
    //     .add(cube.position, 'y')
    //     .min(-3)
    //     .max(3)
    //     .step(0.01)
    //     .name('elevation')
    // gui
    //     .add(cube, 'visible')
    //
    // gui
    //     .add(material, 'wireframe')
    // gui
    //     .addColor(parameters, 'color')
    //     .onChange(() => {
    //       material.color.set(parameters.color)
    //     })
    // gui
    //     .add(parameters, 'spin')

    // Controls
    const controls = new OrbitControls(camera, renderer.domElement)
    controls.enableDamping = true // 启用惯性
    // controls.target.y = 2

    camera.position.z = 2; // 将摄像机向外移动一些， 实际上这个z值是我们从上看这个立方体的距离，如果不设置话，我们就什么也看不见
    controls.update()
    // camera.lookAt(new THREE.Vector3(0, 2,0)) // 提供一个向量，作为camera的视角出发点
    camera1.position.z = 1


    const axesHelper = new THREE.AxesHelper(5);
    scene.add(axesHelper);

    let time = new Date();

    const clock = new THREE.Clock();

    // gsap.to(cube.position, { duration: 1, delay: 1, x: 3})

    function animate() {

        const elapsedTime = clock.getElapsedTime()

        sphere.rotation.y = elapsedTime * 0.1
        plane.rotation.y = elapsedTime * 0.1
        torus.rotation.y = elapsedTime * 0.1

        sphere.rotation.x = elapsedTime * 0.15
        plane.rotation.x = elapsedTime * 0.15
        torus.rotation.x = elapsedTime * 0.15
        //  更改camera的角度: 我们使用OrbitControl 来更好的控制
        // camera.position.x = Math.sin(cursor.x * Math.PI * 2) * 3
        // camera.position.z = Math.cos(cursor.x * Math.PI * 2) * 3
        // camera.position.y = Math.cos(cursor.y * Math.PI * 2) * 3
        // camera.lookAt(cube.position)

        // 每次运动都要更新controls
        controls.update()

        // cube.rotation.reorder('YXZ') // reorder是用来定义更新顺序的
        //   cube.rotation.x += 0.01
        // cube.rotation.y += 0.01;
        // cube.scale.x += 0.01
        // cube1.rotation.x -= 0.01;
        // cube1.rotation.y -= 0.01
        // console.log(cube.position.length()); // 计算从(0, 0, 0) 到 (x, y, z)的欧几里得长度 （Euclidean length，即直线长度），这是基于三维向量，我们给cube一个三维坐标，如果没有给的话，这个距离就是0
        renderer.render(scene, camera);

        // let currentTime = new Date()
        // let deltaTime = currentTime - time
        // time = currentTime

        // console.log(deltaTime); // 这里我们计算出来的差值可以让我们的物体运动的速度始终和浏览器的帧率保持一致,可以达到兼容的效果

        // console.log(clock.getElapsedTime()); // 定时器，从动画开始执行的时间
        requestAnimationFrame(animate); // 相比于setInterval的优点： 最重要的一点或许就是当用户切换到其它的标签页时，它会暂停，因此不会浪费用户宝贵的处理器资源，也不会损耗电池的使用寿命。
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
