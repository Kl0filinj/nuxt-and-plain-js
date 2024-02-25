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
  Group,
  type Object3DEventMap,
} from 'three'
import { GLTFLoader } from 'three/examples/jsm/loaders/GLTFLoader.js'
import { OrbitControls } from 'three/addons/controls/OrbitControls.js'
import { useWindowSize } from '@vueuse/core'
import { Body, Box, Vec3, World } from 'cannon-es'
import Ammo from 'ammojs-typed'

// *** Main vars
let renderer: WebGLRenderer
let controls: OrbitControls
let world: World

let bodyModel: Group<Object3DEventMap>
let boxBody: Body

// *** Physic vars
let gravityConstant = -9.8
let physicsWorld: Ammo.btSoftRigidDynamicsWorld
let rigidBodies = []
let softBodies = []
let margin = 0.05
// let transformAux1 = new Ammo.btTransform()
// let softBodyHelpers = new Ammo.btSoftBodyHelpers()

let armMovement = 0

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
    bodyModel = gltf.scene
    scene.add(bodyModel)

    bodyModel.traverse((node: any) => {
      if (node.isMesh) {
        node.castShadow = true
      }
    })
    bodyModel.position.set(0, 0, 0)
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

function initCannon() {
  world = new World()
  world.gravity.set(0, -9.82, 0)

  const boxShape = new Box(new Vec3(1, 1, 1))
  boxBody = new Body({ mass: 1, shape: boxShape })
  world.addBody(boxBody)
}

function initAmmo() {
  var collisionConfiguration = new Ammo.btSoftBodyRigidBodyCollisionConfiguration()
  var dispatcher = new Ammo.btCollisionDispatcher(collisionConfiguration)
  var broadphase = new Ammo.btDbvtBroadphase()
  var solver = new Ammo.btSequentialImpulseConstraintSolver()
  var softBodySolver = new Ammo.btDefaultSoftBodySolver()
  physicsWorld = new Ammo.btSoftRigidDynamicsWorld(
    dispatcher,
    broadphase,
    solver,
    collisionConfiguration,
    softBodySolver,
  )
  physicsWorld.setGravity(new Ammo.btVector3(0, gravityConstant, 0))
  physicsWorld.getWorldInfo().set_m_gravity(new Ammo.btVector3(0, gravityConstant, 0))
}

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

onMounted(async () => {
  console.log({ Ammo })
  setRenderer()
  // initCannon()
  initAmmo()
  loop()
})

const loop = () => {
  // if (bodyModel && boxBody) {
  //   bodyModel.position.copy(boxBody.position)
  //   bodyModel.quaternion.copy(boxBody.quaternion)
  // }

  controls.update()
  renderer.render(scene, camera)
  physicsWorld.stepSimulation(1 / 60, 10)
  requestAnimationFrame(loop)
}
</script>

<template>
  <div>
    <canvas ref="experience" />
  </div>
</template>
