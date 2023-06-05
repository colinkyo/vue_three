<template>
  <div class="">
    <h1>{{ percentage }}</h1>
  </div>
</template>

<script setup>
import * as THREE from 'three'
import * as CANNON from 'cannon-es'
import { ref } from 'vue'
import gsap from 'gsap'
import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls'
import { GLTFLoader } from "three/examples/jsm/loaders/GLTFLoader";
import { DRACOLoader } from "three/examples/jsm/loaders/DRACOLoader";

let percentage = ref(30)

gsap.to(percentage, {
  duration: 1,
  value: 100,
  ease: "linear",
  repeat: -1,
  onUpdate: () =>{
    percentage.value = Math.floor(percentage.value)
  }
})

// 初始化场景
const scene = new THREE.Scene()
// 初始化相机
const camera = new THREE.PerspectiveCamera(75,window.innerWidth/window.innerHeight,0.1,1000)
camera.position.set(4,2,0)
camera.updateProjectionMatrix()

// 初始化渲染器
const renderer = new THREE.WebGLRenderer({
  antialias: true, // 设置抗锯齿
  logarithmicDepthBuffer: true, // 使用对数缓存
})
renderer.setSize(window.innerWidth,window.innerHeight)
// 设置色调映射
renderer.toneMapping = THREE.ACESFilmicToneMapping
renderer.toneMappingExposure = 1
// 设置阴影
renderer.shadowMap.enabled = true
// 设置阴影类型,实现更加平滑和真实的阴影效果
renderer.shadowMap.type = THREE.PCFSoftShadowMap
document.body.appendChild(renderer.domElement)

// 初始化控制器
const controls = new OrbitControls(camera,renderer.domElement)
controls.enableDamping = true

// 监听屏幕大小变化
window.addEventListener("resize",()=>{ 
  renderer.setSize(window.innerWidth,window.innerHeight)
  camera.aspect = window.innerWidth/window.innerHeight
  camera.updateProjectionMatrix()
})

// 设置环境纹理
const texture = new THREE.TextureLoader()
texture.load("./textures/outdoor.jpg", (texture)=>{
  texture.mapping = THREE.EquirectangularReflectionMapping
  // 设置环境纹理
  scene.background = texture
  scene.environment = texture
  scene.backgroundBlurriness = 0.3 // 设置模糊度
})

let ball,ballBody

// 解压模型
const dracoLoader = new DRACOLoader()
dracoLoader.setDecoderPath('./draco/')
// 加载模型
const gltfLoader = new GLTFLoader()
gltfLoader.setDRACOLoader(dracoLoader)
gltfLoader.load('./model/playground02.glb',(glb) => {
  const model = glb.scene
  model.traverse((child)=>{
    if(child.isMesh && child.name.search(/Solid/ == -1)) {
      child.castShadow = true
      child.receiveShadow = true
      // 创建trimesh类型
      const trimesh = new CANNON.Trimesh(
        child.geometry.attributes.position.array,
        child.geometry.index.array
      )
      // 创建刚体
      const trimeshBody = new CANNON.Body({
        mass: 0,
        shape: trimesh
      })
      // 获取世界位置和旋转给到物理世界
      trimeshBody.position.copy(child.getWorldPosition(new THREE.Vector3()))
      trimeshBody.quaternion.copy(
        child.getWorldQuaternion(new THREE.Quaternion())
      )
      // 添加刚体到物理世界
      world.addBody(trimeshBody)

      if(child.name === "door"){
        child.material = new THREE.MeshBasicMaterial({
          color: 0x000000,
          opacity:0,
          transparent:true
        })
      }
    }
    if(child.name === 'Soccer_Ball'){
      ball = child
      // 创建球体
      const ballShape = new CANNON.Sphere(0.15)
      // 创建刚体
      ballBody = new CANNON.Body({
        mass: 1,
        position: new CANNON.Vec3(0,5,0),
        shape: ballShape
      })
      // 添加刚体到物理世界
      world.addBody(ballBody)
    }
    setTimeout(()=>{
      ballBody.position.set(0,15,0)
      ballBody.velocity.set(0,0,0)
      ballBody.angularVelocity.set(0,0,0)
    },2000)
  })
  scene.add(model)
})

// 添加聚光灯
const spotLight = new THREE.SpotLight(0xffffff)
spotLight.position.set(10,50,0)
spotLight.castShadow = true
spotLight.shadow.mapSize.width = 2048
spotLight.shadow.mapSize.height = 2048
spotLight.shadow.camera.near = 0.5
spotLight.shadow.camera.far = 500
spotLight.shadow.camera.fov = 30
spotLight.shadow.bias = -0.00008
spotLight.intensity = 0.1
scene.add(spotLight)

// 初始化物理世界
const world = new CANNON.World()
world.gravity.set(0,-9.82,0)

let clock = new THREE.Clock()
// 设置渲染函数
const render = () =>{ 
  let delta = clock.getDelta()
  world.step(delta) // 更新物理世界
  if(ball && ballBody) {
    ball.position.copy(ballBody.position)
    ball.quaternion.copy(ballBody.quaternion)
  }
  controls.update()
  requestAnimationFrame(render)
  renderer.render(scene,camera)
}
render()

let isClick = false
// 设置点击事件
window.addEventListener('click',()=>{
  if(isClick) return
  isClick = true
  ballBody.applyForce(
    new CANNON.Vec3(
      -1*percentage.value,
      6*percentage.value,
      (Math.random() - 0.5)*percentage.value
    ),ballBody.position)
  setTimeout(()=>{
    isClick = false
    ballBody.position.set(0,15,0)
    ballBody.velocity.set(0,0,0)
    ballBody.angularVelocity.set(0,0,0)
  },4000)
})
</script>

<style lang="less">
canvas {
  position: fixed;
  left: 0;
  top: 0;
  width: 100vw;
  height: 100vh;
}
h1{
  position: fixed;
  left: 0;
  top: 0;
  z-index: 10;
  color: #fff;
}
</style>