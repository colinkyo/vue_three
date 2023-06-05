<script setup>
import * as THREE from 'three'
import { onMounted } from 'vue';
// 控制器
import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls';
// 文字2D渲染器
import { CSS2DRenderer,CSS2DObject } from 'three/examples/jsm/renderers/CSS2DRenderer'

const scene = new THREE.Scene()
const camera = new THREE.PerspectiveCamera(45,window.innerWidth/window.innerHeight,0.1,200)
camera.position.set(10,5,20)

// 创建渲染器
const renderer = new THREE.WebGLRenderer({ 
  alpha: true
})
renderer.setPixelRatio(window.devicePixelRatio) // 根据屏幕显示像素比
renderer.setSize(window.innerWidth,window.innerHeight)
renderer.shadowMap.enabled = true // 渲染阴影
document.body.appendChild(renderer.domElement)

window.addEventListener("resize",()=>{ 
  renderer.setSize(window.innerWidth,window.innerHeight)
  camera.aspect = window.innerWidth/window.innerHeight
  camera.updateProjectionMatrix()
})

// 设置控制器
const controls = new OrbitControls(camera,renderer.domElement)
controls.enableDamping = true

let clock = new THREE.Clock();
let oldtime = 0
// 设置渲染函数
const render = () =>{ 
  const elapsed = clock.getElapsedTime()
  // 月球旋转
  moon.position.set(Math.sin(elapsed)*5,0,Math.cos(elapsed)*5)
  // 地球自转
  let axis = new THREE.Vector3(0,1,0)
  earth.rotateOnAxis(axis, (elapsed - oldtime) * Math.PI / 10)
  oldtime = elapsed
  renderer.render(scene,camera)
  labelRenderer.render(scene, camera)
  requestAnimationFrame(render)
}

// 设置地球和月球的半径大小
const EARTH_RADIUS = 2.5
const MOON_RADIUS = 0.27
// 实例化纹理加载器
const textureLoader = new THREE.TextureLoader()
// 创建标签选择器
const labelRenderer = new CSS2DRenderer();
labelRenderer.setSize(window.innerWidth, window.innerHeight)
labelRenderer.domElement.style.position = 'absolute';
labelRenderer.domElement.style.top = '0px';
document.body.appendChild(labelRenderer.domElement)
// 给CSS2DRenderer添加控制器
const controls1 = new OrbitControls(camera,labelRenderer.domElement)
controls1.enableDamping = true

// 创建月球
const moonGeometry = new THREE.SphereGeometry(MOON_RADIUS,16,16)
const moonMaterial = new THREE.MeshPhongMaterial({
  map: textureLoader.load('/public/textures/planets/moon_1024.jpg')
})
const moon = new THREE.Mesh(moonGeometry, moonMaterial)
moon.receiveShadow = true
moon.castShadow = true;
scene.add(moon)
// 创建月球标签
const moonDiv = document.createElement('div');
moonDiv.className = 'label';
moonDiv.textContent = 'Moon';
const moonLabel = new CSS2DObject(moonDiv)
moonLabel.position.set(0, MOON_RADIUS + 0.5, 0);
moon.add(moonLabel)

// 创建地球
const earthGeometry = new THREE.SphereGeometry(EARTH_RADIUS,16,16)
const earthMaterial = new THREE.MeshPhongMaterial({
  map: textureLoader.load('/public/textures/planets/earth_atmos_2048.jpg'),
  specularMap: textureLoader.load('/public/textures/planets/earth_specular_2048.jpg'),
  normalMap: textureLoader.load('/public/textures/planets/earth_normal_2048.jpg'),
})
const earth = new THREE.Mesh(earthGeometry,earthMaterial)
earth.receiveShadow = true
earth.castShadow = true
scene.add(earth)
// 创建地球标签
const earthDiv = document.createElement('div');
earthDiv.className = 'label';
earthDiv.textContent = 'Earch';
const eartchLabel = new CSS2DObject(earthDiv)
eartchLabel.position.set(0, EARTH_RADIUS + 0.5, 0);
earth.add(eartchLabel)

// 创建添加聚光灯光源
const dirLight = new THREE.SpotLight(0xffffff)
dirLight.position.set(0,0,10)
dirLight.intensity = 1
dirLight.castShadow = true
scene.add(dirLight)
// 添加环境光
const aLight = new THREE.AmbientLight(0xffffff)
aLight.intensity = 0.03
scene.add(aLight)

onMounted(()=>{
  render()
})
</script>
<style lang="less">
canvas{
  background-image: url('../../public/images/star.jpg');
  background-size: cover;
}
.label {
  color: #fff;
  font-size: 16px;
}
</style>