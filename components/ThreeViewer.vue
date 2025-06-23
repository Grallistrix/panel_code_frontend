<script setup lang="ts">
import { onMounted, ref } from 'vue'
import { OrbitControls } from 'three-stdlib'
import * as THREE from 'three'
import { FBXLoader } from 'three-stdlib'

const container = ref<HTMLDivElement | null>(null)

onMounted(() => {
  if (!container.value) return

  const scene = new THREE.Scene()
  scene.background = new THREE.Color('#ffffff')

  const camera = new THREE.PerspectiveCamera(75, container.value.clientWidth / container.value.clientHeight, 0.1, 1000)
  camera.position.set(0, 10, 10)
  camera.lookAt(0, 0, 0)

  const renderer = new THREE.WebGLRenderer({ antialias: true })
  renderer.setSize(container.value.clientWidth, container.value.clientHeight)
  container.value.appendChild(renderer.domElement)

  const controls = new OrbitControls(camera, renderer.domElement)
  controls.enableDamping = true
  controls.dampingFactor = 0.05
  controls.screenSpacePanning = false
  controls.minDistance = 2
  controls.maxDistance = 100
  controls.maxPolarAngle = Math.PI / 2  // zapobiega obracaniu pod model

  const light = new THREE.HemisphereLight(0xffffff, 0x444444)
  scene.add(light)

  const directionalLight = new THREE.DirectionalLight(0xffffff, 1)
  directionalLight.position.set(5, 10, 7.5)
  scene.add(directionalLight)

  const loader = new FBXLoader()
  loader.load('/abc.fbx', (object) => {
    object.traverse((child) => {
      if ((child as THREE.Mesh).isMesh) {
        const mesh = child as THREE.Mesh
        if (mesh.name === 'Kuchnia') {
          mesh.material = new THREE.MeshStandardMaterial({ color: 'red' })
        } else if (mesh.name === 'Salon') {
          mesh.material = new THREE.MeshStandardMaterial({ color: 'blue' })
        } else {
          mesh.material = new THREE.MeshStandardMaterial({ color: 'gray' })
        }
      }
    })
    scene.add(object)
  })

  window.addEventListener('resize', () => {
    if (!container.value) return
    camera.aspect = container.value.clientWidth / container.value.clientHeight
    camera.updateProjectionMatrix()
    renderer.setSize(container.value.clientWidth, container.value.clientHeight)
  })

  const animate = () => {
    requestAnimationFrame(animate)
    controls.update()
    renderer.render(scene, camera)
  }
  animate()
})

</script>

<template>
  <section
    class="w-full max-w-6xl p-6 text-gray-900 bg-white border border-blue-300 shadow-lg rounded-2xl"
  >
    <h2 class="pb-2 mb-4 text-2xl font-bold border-b border-blue-300">Podgląd 3D - Model budynku</h2>
    <div ref="container" class="w-full h-[600px] rounded-lg overflow-hidden border border-gray-300" />
  </section>
</template>
