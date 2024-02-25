<script setup lang="ts">
import {
  Scene,
  PerspectiveCamera,
  Mesh,
  SphereGeometry,
  MeshBasicMaterial,
  WebGLRenderer,
  AxesHelper,
  GridHelper,
  Color,
  Plane,
  PlaneGeometry,
  Euler,
  MeshStandardMaterial,
  DirectionalLight,
  DirectionalLightHelper,
  CameraHelper,
  TextureLoader,
} from 'three'
import { GLTFLoader } from 'three/examples/jsm/loaders/GLTFLoader.js'
import { OrbitControls } from 'three/addons/controls/OrbitControls.js'
import { useWindowSize } from '@vueuse/core'

let renderer: WebGLRenderer
let controls: OrbitControls
const experience: Ref<HTMLCanvasElement | null> = ref(null)

const { width, height } = useWindowSize()
const aspectRatio = computed(() => width.value / height.value)

const scene = new Scene()

const camera = new PerspectiveCamera(75, aspectRatio.value, 0.1, 1000)
camera.position.set(0, 2, 4)

scene.add(camera)

// *** Helpers
const textureLoader = new TextureLoader()

// *** Light
const mainDirectionalLight = new DirectionalLight('#ffffff', 1.2)
mainDirectionalLight.position.set(0, 7, 6)
mainDirectionalLight.castShadow = true
scene.add(mainDirectionalLight)

// *** Objects

// const sphere = new Mesh(new SphereGeometry(1, 32, 32), new MeshBasicMaterial({ color: 0x008080 }))
// scene.add(sphere)

const floor = new Mesh(
  new PlaneGeometry(20, 20),
  new MeshStandardMaterial({ color: '#ffffff', map: textureLoader.load('/concreteFloorTexture.jpeg') }),
)
floor.rotation.set(-Math.PI / 2, 0, 0)
floor.receiveShadow = true
scene.add(floor)

const modelsLoader = new GLTFLoader()
modelsLoader.load(
  '/DummyMan.glb',
  function (gltf) {
    const model = gltf.scene
    scene.add(model)

    model.traverse((node: any) => {
      if (node.isMesh) {
        node.castShadow = true
      }
    })
    model.position.set(0, 0, 0)
  },
  undefined,
)

// *** Helpers
const axcesHelper = new AxesHelper(5)
scene.add(axcesHelper)

const gridHelper = new GridHelper(20)
scene.add(gridHelper)

const mainDirectionalLightHelper = new DirectionalLightHelper(mainDirectionalLight)
scene.add(mainDirectionalLightHelper)

const mainDirectionalLightShadowHelper = new CameraHelper(mainDirectionalLight.shadow.camera)
scene.add(mainDirectionalLightShadowHelper)

function updateCamera() {
  camera.aspect = aspectRatio.value
  camera.updateProjectionMatrix()
}

function updateRenderer() {
  renderer.setSize(width.value, height.value)
  renderer.setClearColor(new Color('#000000'))
  renderer.shadowMap.enabled = true
  renderer.render(scene, camera)
}

function setRenderer() {
  if (experience.value) {
    renderer = new WebGLRenderer({ canvas: experience.value, alpha: true })
    renderer.setPixelRatio(Math.min(window.devicePixelRatio, 2))
    controls = new OrbitControls(camera, renderer.domElement)
    controls.enableDamping = true
    updateRenderer()
  }
}

watch(aspectRatio, () => {
  updateCamera()
  updateRenderer()
})

onMounted(() => {
  setRenderer()
  loop()
})

const loop = () => {
  controls.update()
  renderer.render(scene, camera)
  requestAnimationFrame(loop)
}
</script>

<template>
  <div>
    <canvas ref="experience" />
  </div>
</template>
