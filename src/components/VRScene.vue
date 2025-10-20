<template>
  <a-scene background="color: #000000">
    <a-assets timeout="10000">
      <a-asset-item id="background-scene" src="/models/background1/background1.gltf"></a-asset-item>
    </a-assets>

    <!-- Load 3D model after assets are ready -->
    <a-entity
      v-if="assetsLoaded"
      gltf-model="#background-scene"
      position="0 0 0"
      scale="1 1 1"
      animation-mixer
    ></a-entity>

    <!-- Camera rig with multitouch-look-controls for mobile and look-controls fallback -->
    <a-entity position="0 1.6 3">
      <a-camera
        multitouch-look-controls
        look-controls="touchEnabled: true; mouseEnabled: true"
      ></a-camera>
    </a-entity>
  </a-scene>
</template>

<script setup>
import { ref, onMounted } from 'vue'

const assetsLoaded = ref(false)

onMounted(() => {
  const assetItem = document.querySelector('#background-scene')
  if (!assetItem) {
    console.error('Background asset not found')
    return
  }
  assetItem.addEventListener('loaded', () => {
    assetsLoaded.value = true
  })
})
</script>

<style scoped>
/* Optional full-screen styling */
a-scene {
  width: 100vw;
  height: 100vh;
  margin: 0;
  padding: 0;
}
</style>
