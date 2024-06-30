<template>
  <div class="wrapper" ref="wrapper">
    <div class="operate">
      <button @click="overview">全览</button>
      <button @click="walk">行走</button>
    </div>
    <div class="point-0" @click="numClick">
      <div class="title">#1</div>
      <div class="info">
        小船：梦中又见那宁静的大海，我前进了，驶向远方，我知道我是船，只属于远方。这一天，我用奋斗作为白帆，要和明天一起飘扬，呼喊。
      </div>
    </div>
    <svg
      class="user-1"
      xmlns="http://www.w3.org/2000/svg"
      width="1em"
      height="1em"
      viewBox="0 0 100 100"
    >
      <path
        fill="currentColor"
        d="M49.781 23.592C41.947 23.593 34.184 26.96 35 33.688l2 14.624C37.352 50.886 39.09 55 41.688 55h.185L44 80.53c.092 1.103.892 2 2 2h8c1.108 0 1.908-.897 2-2L58.127 55h.185c2.597 0 4.336-4.115 4.688-6.687l2-14.626c.523-6.733-7.384-10.097-15.219-10.095"
        color="currentColor"
      />
      <path
        fill="currentColor"
        d="m50.024 50.908l-.048.126c.016-.038.027-.077.043-.115zM34.006 69.057C19.88 71.053 10 75.828 10 82.857C10 92.325 26.508 100 50 100s40-7.675 40-17.143c0-7.029-9.879-11.804-24.004-13.8l-1.957 3.332C74.685 73.866 82 76.97 82 80.572c0 5.05-14.327 9.143-32 9.143c-17.673 0-32-4.093-32-9.143c-.001-3.59 7.266-6.691 17.945-8.174c-.645-1.114-1.294-2.226-1.94-3.341"
        color="currentColor"
      />
      <circle cx="50" cy="10.5" r="10.5" fill="currentColor" color="currentColor" />
    </svg>
  </div>
</template>

<script setup>
import { nextTick, onMounted, ref } from 'vue'
import * as THREE from 'three'
import { GLTFLoader } from 'three/examples/jsm/loaders/GLTFLoader.js'
import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls.js'
import { PointerLockControls } from 'three/examples/jsm/controls/PointerLockControls.js'
import SkyModule from '@/assets/images/sky.jpg'
import Animations from './animations.js'
import TWEEN from '@tweenjs/tween.js'
// 3D 图像
const wrapper = ref(null)
const renderer = new THREE.WebGLRenderer()
const scene = new THREE.Scene()

onMounted(() => {
  renderer.setSize(wrapper.value.clientWidth, wrapper.value.clientHeight)
  wrapper.value.appendChild(renderer.domElement)

  nextTick(() => {
    overview()
  })
})

var camera = new THREE.PerspectiveCamera(75, window.innerWidth / window.innerHeight, 0.1, 1000)
camera.position.z = 5

// 灯光
var light = new THREE.DirectionalLight(0xffffff, 1)
light.position.set(0, 1, 1)
scene.add(light)

// 环境光源
var ambientLight = new THREE.AmbientLight(0xffffff)
scene.add(ambientLight)

// 天空盒
new THREE.TextureLoader().load(
  SkyModule, // 替换为你的图片路径
  function (texture) {
    scene.background = texture
    scene.background.minFilter = THREE.LinearFilter
  },
  undefined,
  function () {
    console.error('An error occurred while loading the texture.')
  }
)

// 加载模型文件
for (var i = 0; i < 26; i++) {
  let url = `./models/${i + 1}.glb`
  new GLTFLoader().load(
    // 资源URL
    url,
    // onLoad callback
    function (gltf) {
      gltf.scene.position.y = -1.5
      scene.add(gltf.scene)
    },
    // onProgress callback
    function () {},
    // onError callback
    function () {
      console.log('An error happened')
    }
  )
}

// 定义控制器
let orbitControls = null
let pointerLockControls = null
let animationId = null

// 键盘控制
const keys = {
  w: false,
  a: false,
  s: false,
  d: false
}

document.addEventListener('keydown', function (event) {
  if (Object.prototype.hasOwnProperty.call(keys, event.key)) {
    keys[event.key] = true
  }
})

document.addEventListener('keyup', function (event) {
  if (Object.prototype.hasOwnProperty.call(keys, event.key)) {
    keys[event.key] = false
  }
})

renderer.domElement.addEventListener('mousedown', function () {
  if (pointerLockControls) {
    pointerLockControls.lock()
  }
})
renderer.domElement.addEventListener('mouseup', function () {
  if (pointerLockControls) {
    pointerLockControls.unlock()
  }
})

// 点
const raycaster = new THREE.Raycaster()
let points = []
onMounted(() => {
  points = [
    {
      position: new THREE.Vector3(0, 4, -3),
      element: document.querySelector('.point-0')
    }
  ]
})
// 人物移动
let user = { x: 10, y: 0, z: 0 }
let screenPosition = null
const animate = () => {
  animationId = requestAnimationFrame(animate)
  if (orbitControls) {
    orbitControls.update()
  } else {
    var speed = 0.1
    if (keys.w) pointerLockControls.moveForward(speed)
    if (keys.a) pointerLockControls.moveRight(-speed)
    if (keys.s) pointerLockControls.moveForward(-speed)
    if (keys.d) pointerLockControls.moveRight(speed)
  }
  user.x = user.x - 0.01

  const userPosition = new THREE.Vector3(user.x, user.y, user.z).clone()
  userPosition.project(camera)
  let userDom = document.querySelector('.user-1')
  raycaster.setFromCamera(userPosition, camera)
  const userIntersects = raycaster.intersectObjects(scene.children, true)
  if (userIntersects.length === 0) {
    // 未找到相交点，显示
    userDom.classList.add('visible')
  } else {
    // 找到相交点
    // 获取相交点的距离和点的距离
    // 相交点距离比点距离近，隐藏；相交点距离比点距离远，显示
    userIntersects[0].distance < userPosition.distanceTo(camera.position)
      ? userDom.classList.remove('visible')
      : userDom.classList.add('visible')
  }
  const translateX = userPosition.x * wrapper.value.clientWidth * 0.5
  const translateY = -userPosition.y * wrapper.value.clientHeight * 0.5
  userDom.style.transform = `translateX(${translateX}px) translateY(${translateY}px)`

  // 遍历每个点
  for (const point of points) {
    // 获取2D屏幕位置
    screenPosition = point.position.clone()
    screenPosition.project(camera)
    raycaster.setFromCamera(screenPosition, camera)
    const intersects = raycaster.intersectObjects(scene.children, true)
    if (intersects.length === 0) {
      // 未找到相交点，显示
      point.element.classList.add('visible')
    } else {
      // 找到相交点
      // 获取相交点的距离和点的距离
      const intersectionDistance = intersects[0].distance
      const pointDistance = point.position.distanceTo(camera.position)
      // 相交点距离比点距离近，隐藏；相交点距离比点距离远，显示
      intersectionDistance < pointDistance
        ? point.element.classList.remove('visible')
        : point.element.classList.add('visible')
    }
    const translateX = screenPosition.x * wrapper.value.clientWidth * 0.5
    const translateY = -screenPosition.y * wrapper.value.clientHeight * 0.5
    point.element.style.transform = `translateX(${translateX}px) translateY(${translateY}px)`
  }
  TWEEN.update()
  renderer.render(scene, camera)
}

// 销毁控制器
const disposeControls = () => {
  if (orbitControls) {
    orbitControls.dispose()
    orbitControls = null
  }
  if (pointerLockControls) {
    pointerLockControls.dispose()
    pointerLockControls = null
  }
}

// 全览
const overview = () => {
  disposeControls()

  if (animationId !== null) {
    cancelAnimationFrame(animationId)
    animationId = null
  }

  orbitControls = new OrbitControls(camera, renderer.domElement)

  // 设置初始化位置
  camera.position.set(-32.88642851115013, 18.38740821135217, 16.111444880366214)

  animate()
}

// 行走
const walk = () => {
  disposeControls()

  if (animationId !== null) {
    cancelAnimationFrame(animationId)
    animationId = null
  }

  pointerLockControls = new PointerLockControls(camera, renderer.domElement)

  // 设置初始化位置
  pointerLockControls
    .getObject()
    .position.set(-31.706153247589267, 1.54062542685844e-14, 0.23836989274958773)

  // 初始化相机角度
  pointerLockControls.getObject().rotation.x = 0
  pointerLockControls.getObject().rotation.y = -1.2
  pointerLockControls.getObject().rotation.z = 0

  animate()
}

const numClick = () => {
  Animations.animateCamera(
    camera,
    orbitControls || pointerLockControls,
    { x: 0, y: 0, z: 0 },
    { x: 0, y: 0, z: 0 },
    1600,
    () => {}
  )
}
</script>

<style lang="scss" scoped>
.wrapper {
  width: 100vw;
  height: 100vh;
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  .operate {
    position: absolute;
    top: 10px;
    left: 10px;
    display: flex;
    gap: 10px;
    z-index: 100;
    button {
      cursor: pointer;
    }
  }

  .point-0 {
    z-index: 10;
    color: #fff;
    position: absolute;
    top: calc(50% - 15px);
    left: calc(50% - 15px);
    transition: all 0.3s;
    opacity: 0;

    .title {
      display: inline-flex;
      justify-content: center;
      align-items: center;
      width: 30px;
      height: 30px;
      border-radius: 15px;
      font-size: 10px;
      background-color: rgba(0, 0, 0, 0.8);
    }

    .info {
      width: 200px;
      background-color: rgba(0, 0, 0, 0.8);
      display: none;
      position: absolute;
      border-radius: 15px;
    }

    &.visible {
      opacity: 1;
    }

    &:hover {
      .info {
        display: block;
        font-size: 8px;
        padding: 10px;
        top: calc(100% + 5px);
      }
    }
  }

  .user-1 {
    width: 20px;
    height: 20px;
    z-index: 10;
    color: #fff;
    fill: #fff;
    position: absolute;
    top: calc(50% - 10px);
    left: calc(50% - 10px);
  }
}
</style>
