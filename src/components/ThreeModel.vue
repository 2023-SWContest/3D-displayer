<script setup>
import * as THREE from 'three'
import * as dat from 'dat.gui'
import {onBeforeUnmount, onMounted, ref, watch} from "vue";
import {DRACOLoader} from "three/examples/jsm/loaders/DRACOLoader";
import {GLTFLoader} from "three/examples/jsm/loaders/GLTFLoader";
import {OrbitControls} from "three/examples/jsm/controls/OrbitControls";

const blockId = ref(2);
const scene = new THREE.Scene()
let gltfLoader = null;
let mixer = null;

const updateModel = () => {
  if (!gltfLoader) {
    return;
  }
  gltfLoader.load(
      "https://blockdream-imagebed-1310215785.cos.ap-shanghai.myqcloud.com/models/" + blockId.value.toString() + ".gltf",
      (gltf) => {

        gltf.scene.scale.set(0.01, 0.01, 0.01)
        gltf.scene.rotation.x = -Math.PI / 2;

        scene.add(gltf.scene)

        // Animation
        mixer = new THREE.AnimationMixer(gltf.scene)
        const action = mixer.clipAction(gltf.animations[2])
        action.play()
      }
  )
}

// 处理接收到的消息
function handleMessage(event) {
  if (event.data && event.data.type === 'blockId') {
    blockId.value = event.data.data
    // 移除旧模型
    scene.children.forEach(child => {
      if (child.type === 'Group') {
        scene.remove(child);
      }
    });
    updateModel()
    console.log('Received blockId:', blockId)
  }
}

onMounted(() => {

  window.addEventListener('message', handleMessage);

  /**
   * Models
   */
  const gui = new dat.GUI()
  const canvas = document.getElementById("canvas")

  const dracoLoader = new DRACOLoader()
  dracoLoader.setDecoderPath('/draco/')
  gltfLoader = new GLTFLoader()
  gltfLoader.setDRACOLoader(dracoLoader)

  updateModel()

  /**
   * Lights
   */
  const ambientLight = new THREE.AmbientLight(0xffffff, 0.8)
  scene.add(ambientLight)

  const directionalLight = new THREE.DirectionalLight(0xffffff, 0.6)
  directionalLight.castShadow = true
  directionalLight.shadow.mapSize.set(1024, 1024)
  directionalLight.shadow.camera.far = 15
  directionalLight.shadow.camera.left = -7
  directionalLight.shadow.camera.top = 7
  directionalLight.shadow.camera.right = 7
  directionalLight.shadow.camera.bottom = -7
  directionalLight.position.set(-5, 5, 0)
  scene.add(directionalLight)

  /**
   * Sizes
   */
  const sizes = {
    width: window.innerWidth,
    height: window.innerHeight
  }

  window.addEventListener('resize', () => {
    // Update sizes
    sizes.width = window.innerWidth
    sizes.height = window.innerHeight

    // Update camera
    camera.aspect = sizes.width / sizes.height
    camera.updateProjectionMatrix()

    // Update renderer
    renderer.setSize(sizes.width, sizes.height)
    renderer.setPixelRatio(Math.min(window.devicePixelRatio, 2))
  })

  /**
   * Camera
   */
      // Base camera
  const camera = new THREE.PerspectiveCamera(75, sizes.width / sizes.height, 0.1, 100)
  camera.position.set(2, 2, 2)
  scene.add(camera)

  // Controls
  const controls = new OrbitControls(camera, canvas)
  controls.target.set(0, 0.75, 0)
  controls.enableDamping = true

  // const axes = new AxesHelper;
  // scene.add(axes)

  /**
   * Renderer
   */
  const renderer = new THREE.WebGLRenderer({
    canvas: canvas
  })
  renderer.shadowMap.enabled = true
  renderer.shadowMap.type = THREE.PCFSoftShadowMap
  renderer.setSize(sizes.width, sizes.height)
  renderer.setPixelRatio(Math.min(window.devicePixelRatio, 2))

  const tick = () => {

    // Update controls
    controls.update()

    // Render
    renderer.render(scene, camera)

    // Call tick again on the next frame
    window.requestAnimationFrame(tick)
  }

  tick()

})

// 在组件销毁时移除事件监听
onBeforeUnmount(() => {
  window.removeEventListener('message', handleMessage);
});

</script>

<template>
  <canvas id="canvas"></canvas>
</template>

<style>
* {
  margin: 0;
  padding: 0;
}

html,
body {
  overflow: hidden;
}

#canvas {
  position: fixed;
  top: 0;
  left: 0;
  outline: none;
}

</style>