<template>
  <div class="canvasContainer" ref="canvasContainer"></div>
</template>

<script setup>
import * as THREE from 'three'
import { ref, onMounted } from 'vue'
import { GLTFLoader } from "three/examples/jsm/loaders/GLTFLoader.js"
// 添加轨道控制器
import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls'
import bgimg from "../assets/imgs/050.jpg"

const canvasContainer = ref(null)

// 初始化场景
const scene = new THREE.Scene()
// 初始化相机
const camera = new THREE.PerspectiveCamera(75,window.innerWidth/window.innerHeight,0.1,1000)
camera.position.set(1.5,1,1.5) // 设置相机位置
camera.aspect = window.innerWidth / window.innerHeight // 更新摄像头宽高比例
camera.updateProjectionMatrix() // 更新相机投影矩阵
scene.add(camera)

// 初始化渲染器
const renderer = new THREE.WebGLRenderer({ antialias: true })
renderer.setSize(window.innerWidth,window.innerHeight) // 设置渲染器的大小

// 监听屏幕大小变化
window.addEventListener("resize",()=>{ 
  renderer.setSize(window.innerWidth,window.innerHeight)
  camera.aspect = window.innerWidth/window.innerHeight
  camera.updateProjectionMatrix()
})

// 定义渲染函数
const render = () =>{ 
  renderer.render(scene,camera) // 渲染场景
  requestAnimationFrame(render) // 循环渲染
}

// 加载背景纹理
const loader = new THREE.TextureLoader()
const bgTexture = loader.load(bgimg)
bgTexture.mapping = THREE.EquirectangularReflectionMapping // 设置有折射效果的全局背景
scene.background = bgTexture
scene.environment = bgTexture

// 加载小熊模型
const gltfLoader = new GLTFLoader()
gltfLoader.load("src/assets/model/bear.gltf",(gltf)=>{
  const model = gltf.scene.children[0]
  model.material = new THREE.MeshPhongMaterial({
    color: 0xffffff,
    envMap: bgTexture,
    refractionRatio: 0.7,
    reflectivity: 0.99,
    opacity: 0.5
  })
  scene.add(model)
})

// 添加环境光
const ambient = new THREE.AmbientLight(0xffffff,0.85)
scene.add(ambient)

// 挂载完成之后获取dom
onMounted(()=>{
  // 添加控制器
  const controls = new OrbitControls(camera,canvasContainer.value)
  controls.enableDamping = true 
  canvasContainer.value.appendChild(renderer.domElement)
  render()
})

</script>