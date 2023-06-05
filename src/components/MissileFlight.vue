<template>
  <div class="">
    
  </div>
</template>

<script setup>
import * as THREE from 'three'
// 导入控制器
import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls'
import { GLTFLoader } from "three/examples/jsm/loaders/GLTFLoader";
import { DRACOLoader } from "three/examples/jsm/loaders/DRACOLoader";

// 创建场景
let scene = new THREE.Scene()
// 创建相机
const camera = new THREE.PerspectiveCamera(75,window.innerWidth/window.innerHeight,0.1,1000)
camera.position.set(0,5,10)
scene.add(camera)
// 创建渲染器
const renderer = new THREE.WebGLRenderer()
renderer.setSize(window.innerWidth,window.innerHeight)
document.body.appendChild(renderer.domElement)

// 创建控制器
const controls = new OrbitControls(camera,renderer.domElement)
controls.enableDamping = true

window.addEventListener("resize",()=>{ 
  renderer.setSize(window.innerWidth,window.innerHeight)
  camera.aspect = window.innerWidth/window.innerHeight
  camera.updateProjectionMatrix()
})

// 解压模型
const dracoLoader = new DRACOLoader()
dracoLoader.setDecoderPath('./draco/')
// 加载模型
let map1,map2,missile,curvePath
const gltfLoader = new GLTFLoader()
gltfLoader.setDRACOLoader(dracoLoader)
gltfLoader.load('./model/ew8.glb',(gltf)=>{
  const model = gltf.scene
  // 获取模型中的俩个地图
  map1 = model.children[0]
  map2 = model.children[1]
  // 获取模型中的导弹
  missile = model.children[3]
  // 获取模型中的曲线
  curvePath = model.children[2]
  // 将加载的模型添加到场景之中
  scene.add(map1,map2,missile)

  // 根据曲线点创建曲线
  let points = []
  // 将点添加到数组当中
  for(let i= curvePath.geometry.attributes.position.count - 1;i >= 0;i--){
    points.push(
      new THREE.Vector3(
        curvePath.geometry.attributes.position.array[i * 3],
        curvePath.geometry.attributes.position.array[i * 3 + 1],
        curvePath.geometry.attributes.position.array[i * 3 + 2],
      )
    )
  }
  // 创建曲线
  curvePath = new THREE.CatmullRomCurve3(points)
})

// 创建平行光
const light = new THREE.DirectionalLight(0xffffff,1)
// 设置光源位置
light.position.set(0,10,1)
scene.add(light) // 将光源添加到场景当中

// 创建clock
const clock = new THREE.Clock()
// 创建渲染函数
const render = () =>{ 
  // 设置5秒飞行一次
  let t = clock.getElapsedTime() % 5
  t = t / 5
  // 设置导弹飞行路径
  // 先判断curvePath是否存在,如果存在就设置导弹飞行路径
  if(curvePath){
    // 设置导弹飞行路径
    missile.position.copy(curvePath.getPoint(t))
    // 判断t+0.1是否小于1,如果小于1就设置导弹飞行方向
    if(t+0.05<1){
      // 设置导弹飞行方向
      missile.lookAt(curvePath.getPointAt(t+0.05))
    }
  }
  requestAnimationFrame(render)
  renderer.render(scene,camera)
} 
render()

</script>

<style lang="less">

  
</style>