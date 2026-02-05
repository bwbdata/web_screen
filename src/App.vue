<script setup lang="ts">
import { ref, onMounted, onUnmounted } from 'vue'
import ColorPicker from './components/ColorPicker.vue'

const backgroundColor = ref('#000000')
const showColorPicker = ref(false)
const isFullscreen = ref(false)
let clickTimer: number | null = null
let clickCount = 0

const handleClick = () => {
  if (isFullscreen.value) return

  clickCount++

  if (clickCount === 1) {
    clickTimer = window.setTimeout(() => {
      clickCount = 0
    }, 300)
  } else if (clickCount === 2) {
    if (clickTimer) {
      clearTimeout(clickTimer)
      clickTimer = null
    }
    clickCount = 0
    showColorPicker.value = true
  }
}

const closeColorPicker = () => {
  showColorPicker.value = false
}

const selectColor = (color: string) => {
  backgroundColor.value = color
}

const enterFullscreen = async () => {
  try {
    const elem = document.documentElement
    if (elem.requestFullscreen) {
      await elem.requestFullscreen()
    }
    isFullscreen.value = true
    showColorPicker.value = false
  } catch (err) {
    console.error('进入全屏失败:', err)
  }
}

const exitFullscreen = async () => {
  try {
    if (document.fullscreenElement) {
      await document.exitFullscreen()
    }
    isFullscreen.value = false
  } catch (err) {
    console.error('退出全屏失败:', err)
  }
}

const handleKeydown = (e: KeyboardEvent) => {
  if (e.key === 'Escape' && isFullscreen.value) {
    exitFullscreen()
  }
}

const handleFullscreenChange = () => {
  if (!document.fullscreenElement) {
    isFullscreen.value = false
  }
}

onMounted(() => {
  window.addEventListener('keydown', handleKeydown)
  document.addEventListener('fullscreenchange', handleFullscreenChange)
})

onUnmounted(() => {
  window.removeEventListener('keydown', handleKeydown)
  document.removeEventListener('fullscreenchange', handleFullscreenChange)
})
</script>

<template>
  <div
    class="app-container"
    :style="{ backgroundColor }"
    @click="handleClick"
  >
    <ColorPicker
      v-if="showColorPicker"
      :current-color="backgroundColor"
      @close="closeColorPicker"
      @select="selectColor"
      @fullscreen="enterFullscreen"
    />
  </div>
</template>

<style scoped>
.app-container {
  width: 100%;
  height: 100%;
  transition: background-color 0.3s ease;
  cursor: pointer;
}

.fullscreen-overlay {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  z-index: 2000;
  pointer-events: none;
}

.exit-fullscreen-btn {
  position: absolute;
  top: 20px;
  right: 20px;
  padding: 12px 24px;
  background-color: rgba(0, 0, 0, 0.6);
  color: #fff;
  border: 1px solid rgba(255, 255, 255, 0.3);
  border-radius: 8px;
  font-size: 14px;
  cursor: pointer;
  transition: all 0.2s;
  pointer-events: auto;
  backdrop-filter: blur(10px);
}

.exit-fullscreen-btn:hover {
  background-color: rgba(0, 0, 0, 0.8);
  border-color: rgba(255, 255, 255, 0.5);
  transform: scale(1.05);
}
</style>
