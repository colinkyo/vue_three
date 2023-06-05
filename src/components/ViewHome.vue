<template>
  <div class="container" ref="container">
  </div>
</template>

<script setup>
import { ref,onMounted } from 'vue'

import * as THREE from 'three'
// 添加轨道控制器
import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls'

// 初始化场景
const scene = new THREE.Scene()
// 初始化相机
const camera = new THREE.PerspectiveCamera(75,window.innerWidth / window.innerHeight,0.1,1000)
// 设置相机位置
camera.position.z = 0.1

// 初始化渲染器
const renderder = new THREE.WebGLRenderer()
renderder.setSize(window.innerWidth,window.innerHeight)

const container = ref(null)
// 定义渲染函数
const render = () =>{ 
  renderder.render(scene,camera)
  requestAnimationFrame(render)
}

// 添加物体
const geometry = new THREE.BoxGeometry(10,10,10)

// 贴图名称
var arr = ["20_l","20_r","20_u","20_d","20_b","20_f"]
var boxMaterials = []
arr.forEach((item)=>{
  // 纹理加载
  let texture = new THREE.TextureLoader().load(`/src/assets/viewHome/bedroom/${item}.jpg`)
  // 创建材质
  if(item === '20_u' || item === '20_d'){
    texture.rotation = Math.PI
    texture.center = new THREE.Vector2(0.5,0.5)
    boxMaterials.push(new THREE.MeshBasicMaterial({map: texture}))
  }else{
    boxMaterials.push(new THREE.MeshBasicMaterial({map:texture}))
  }
})
const cube = new THREE.Mesh(geometry,boxMaterials)
cube.geometry.scale(1,1,-1)
scene.add(cube)

// 挂载完成之后获取dom
onMounted(()=>{
  // 添加控制器
  const controls = new OrbitControls(camera,container.value)
  controls.enableDamping = true 
  container.value.appendChild(renderder.domElement)
  render()
})
</script>
<style lang="less">

</style>
