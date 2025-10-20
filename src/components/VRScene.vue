<template>
  <div id="vr-container">
    <a-scene background="color: #000000" embedded>

      <!-- Preload all assets -->
      <a-assets>
        <a-asset-item id="background1" src="/models/background1/background1.gltf"></a-asset-item>
        <a-asset-item id="background2" src="/models/background2/background2.gltf"></a-asset-item>
      </a-assets>

      <!-- Show model only after asset loads -->
      <a-entity
        v-if="assetsLoaded"
        :gltf-model="`#${currentScene}`"
        position="0 0 0"
        scale="1 1 1"
        animation-mixer
      ></a-entity>

      <!-- ENTER button -->
      <a-entity
        v-if="currentScene === 'background1' && !isTransitioning"
        position="0 1.6 -1.5"
        geometry="primitive: plane; height: 0.8; width: 2"
        material="color: #0088ff; opacity: 0.8"
        class="clickable"
        @click="startTransition"
      >
        <a-text
          value="ENTER"
          align="center"
          color="#fff"
          position="0 0 0.05"
          width="3"
          wrap-count="10"
        ></a-text>
      </a-entity>

      <!-- Camera with cursor -->
      <a-entity :position="cameraPosition">
        <a-camera look-controls="touchEnabled: true; magicWindowTrackingEnabled: true">
          <a-cursor color="#00ffaa" raycaster="objects: .clickable"></a-cursor>
        </a-camera>
      </a-entity>
    </a-scene>

    <!-- Fade overlay (DOM element over scene) -->
    <div
      class="fade-overlay"
      :class="{ active: fadeVisible }"
    ></div>
  </div>
</template>

<script setup>
import { ref, watch, onMounted } from 'vue'

const currentScene = ref('background1')
const assetsLoaded = ref(false)
const isTransitioning = ref(false)
const fadeVisible = ref(false)
const cameraPosition = ref('0 1.6 3')

// Asset loaded handler
function onAssetLoaded() {
  console.log('Asset loaded!')
  assetsLoaded.value = true
}

// Check if asset already loaded or attach event listener
function setupAssetListener(sceneId) {
  const asset = document.querySelector(`#${sceneId}`)
  if (!asset) {
    console.warn(`No asset found for scene: ${sceneId}`)
    return
  }
  if (asset.hasLoaded) {
    console.log(`Asset ${sceneId} already loaded (cached)`)
    assetsLoaded.value = true
  } else {
    assetsLoaded.value = false
    asset.addEventListener('loaded', onAssetLoaded, { once: true })
  }
}

// Watch for scene changes to setup listener
watch(currentScene, (newScene) => {
  console.log('Scene changed to:', newScene)
  setupAssetListener(newScene)
})

// Animate camera Z position
function animateZoom(fromZ, toZ, duration, callback) {
  const startTime = performance.now()
  function animate(time) {
    const elapsed = time - startTime
    const t = Math.min(elapsed / duration, 1)
    const currentZ = fromZ + t * (toZ - fromZ)
    cameraPosition.value = `0 1.6 ${currentZ}`
    if (t < 1) {
      requestAnimationFrame(animate)
    } else {
      callback && callback()
    }
  }
  requestAnimationFrame(animate)
}

// DOM-based fade (using CSS)
function fadeIn(callback) {
  fadeVisible.value = true
  setTimeout(() => {
    callback && callback()
  }, 1000)
}

function fadeOut(callback) {
  fadeVisible.value = false
  setTimeout(() => {
    callback && callback()
  }, 1000)
}

// Start transition
function startTransition() {
  if (isTransitioning.value) return
  console.log('ENTER clicked!')
  isTransitioning.value = true

  const zoomIn = 0.5
  const zoomStart = 3
  const duration = 1000

  animateZoom(zoomStart, zoomIn, duration, () => {
    fadeIn(() => {
      currentScene.value = 'background2'
      fadeOut(() => {
        animateZoom(zoomIn, zoomStart, duration, () => {
          isTransitioning.value = false
        })
      })
    })
  })
}

onMounted(() => {
  setupAssetListener(currentScene.value)
})
</script>

<style scoped>
#vr-container {
  position: relative;
  width: 100vw;
  height: 100vh;
}

.fade-overlay {
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background: black;
  opacity: 0;
  transition: opacity 1s ease-in-out;
  pointer-events: none;
  z-index: 10;
}

.fade-overlay.active {
  opacity: 1;
}

.clickable {
  cursor: pointer;
}
</style>
