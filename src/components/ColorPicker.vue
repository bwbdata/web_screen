<script setup lang="ts">
import { ref } from 'vue'

const props = defineProps<{
  currentColor?: string
}>()

const emit = defineEmits<{
  close: []
  select: [color: string]
  fullscreen: []
}>()

const customColor = ref('#000000')

const presetColors = [
  { name: '黑色', value: '#000000' },
  { name: '深蓝', value: '#1a1a2e' },
  { name: '深紫', value: '#2d1b3d' },
  { name: '深绿', value: '#1a2e1a' },
  { name: '深红', value: '#2e1a1a' },
  { name: '白色', value: '#ffffff' },
  { name: '灰色', value: '#808080' },
  { name: '蓝色', value: '#4169e1' },
]

const wallpaperSizes = [
  { name: 'iPhone 14/15', width: 1170, height: 2532 },
  { name: 'iPhone 14 Pro Max', width: 1290, height: 2796 },
  { name: 'Android 1080p', width: 1080, height: 1920 },
  { name: 'Android 2K', width: 1440, height: 2560 },
  { name: '桌面 1080p', width: 1920, height: 1080 },
  { name: '桌面 2K', width: 2560, height: 1440 },
  { name: '桌面 4K', width: 3840, height: 2160 },
  { name: 'iPad Pro 12.9"', width: 2048, height: 2732 },
]

const customWidth = ref(1920)
const customHeight = ref(1080)

const selectColor = (color: string) => {
  emit('select', color)
  emit('close')
}

const selectCustomColor = () => {
  if (customColor.value) {
    emit('select', customColor.value)
    emit('close')
  }
}

const downloadWallpaper = (width: number, height: number, sizeName: string) => {
  const canvas = document.createElement('canvas')
  canvas.width = width
  canvas.height = height

  const ctx = canvas.getContext('2d')
  if (!ctx) return

  // 使用当前背景颜色
  const color = props.currentColor || '#000000'
  ctx.fillStyle = color
  ctx.fillRect(0, 0, width, height)

  // 转换为 Blob 并下载
  canvas.toBlob((blob) => {
    if (!blob) return

    const url = URL.createObjectURL(blob)
    const a = document.createElement('a')
    a.href = url
    a.download = `wallpaper-${sizeName.replace(/[^a-zA-Z0-9]/g, '-')}-${width}x${height}.png`
    document.body.appendChild(a)
    a.click()
    document.body.removeChild(a)
    URL.revokeObjectURL(url)
  }, 'image/png', 1.0)
}

const downloadCustomSize = () => {
  if (customWidth.value > 0 && customHeight.value > 0) {
    downloadWallpaper(customWidth.value, customHeight.value, 'custom')
  }
}
</script>

<template>
  <div class="modal-overlay" @click="emit('close')">
    <div class="modal-content" @click.stop>
      <h2>选择背景颜色</h2>

      <div class="preset-colors">
        <div
          v-for="color in presetColors"
          :key="color.value"
          class="color-item"
          @click="selectColor(color.value)"
        >
          <div class="color-box" :style="{ backgroundColor: color.value }"></div>
          <span>{{ color.name }}</span>
        </div>
      </div>

      <div class="custom-color">
        <label>自定义颜色：</label>
        <div class="input-group">
          <input
            v-model="customColor"
            type="text"
            placeholder="#000000 或 rgb(0,0,0)"
          />
          <input
            v-model="customColor"
            type="color"
            class="color-input"
          />
        </div>
        <button @click="selectCustomColor">确定</button>
      </div>

      <div class="divider"></div>

      <div class="fullscreen-section">
        <button class="fullscreen-btn" @click="emit('fullscreen')">
          <span>🖥️ 全屏展示当前颜色</span>
        </button>
      </div>

      <div class="divider"></div>

      <div class="wallpaper-section">
        <h3>下载壁纸</h3>
        <p class="hint">下载当前颜色的纯色壁纸</p>

        <div class="wallpaper-sizes">
          <button
            v-for="size in wallpaperSizes"
            :key="size.name"
            class="size-btn"
            @click="downloadWallpaper(size.width, size.height, size.name)"
          >
            {{ size.name }}<br>
            <span class="size-text">{{ size.width }}×{{ size.height }}</span>
          </button>
        </div>

        <div class="custom-size">
          <label>自定义尺寸：</label>
          <div class="size-input-group">
            <input
              v-model.number="customWidth"
              type="number"
              placeholder="宽度"
              min="1"
              max="10000"
            />
            <span class="separator">×</span>
            <input
              v-model.number="customHeight"
              type="number"
              placeholder="高度"
              min="1"
              max="10000"
            />
            <button class="download-custom-btn" @click="downloadCustomSize">
              下载
            </button>
          </div>
        </div>
      </div>

      <button class="close-btn" @click="emit('close')">取消</button>
    </div>
  </div>
</template>

<style scoped>
.modal-overlay {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background-color: rgba(0, 0, 0, 0.7);
  display: flex;
  justify-content: center;
  align-items: center;
  z-index: 1000;
  padding: 20px;
}

.modal-content {
  background-color: #2a2a2a;
  border-radius: 12px;
  padding: 20px;
  max-width: 400px;
  width: 90%;
  max-height: 90vh;
  overflow-y: auto;
  color: #fff;
}

h2 {
  margin-bottom: 16px;
  font-size: 20px;
  text-align: center;
}

.preset-colors {
  display: grid;
  grid-template-columns: repeat(4, 1fr);
  gap: 10px;
  margin-bottom: 16px;
}

.color-item {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 6px;
  cursor: pointer;
  padding: 6px;
  border-radius: 8px;
  transition: background-color 0.2s;
}

.color-item:hover {
  background-color: rgba(255, 255, 255, 0.1);
}

.color-box {
  width: 45px;
  height: 45px;
  border-radius: 8px;
  border: 2px solid rgba(255, 255, 255, 0.3);
}

.color-item span {
  font-size: 11px;
  text-align: center;
}

.custom-color {
  margin-bottom: 12px;
}

.custom-color label {
  display: block;
  margin-bottom: 8px;
  font-size: 14px;
}

.input-group {
  display: flex;
  gap: 8px;
  margin-bottom: 12px;
}

.input-group input[type="text"] {
  flex: 1;
  padding: 8px 12px;
  border: 1px solid rgba(255, 255, 255, 0.3);
  border-radius: 6px;
  background-color: rgba(255, 255, 255, 0.1);
  color: #fff;
  font-size: 14px;
}

.color-input {
  width: 50px;
  height: 36px;
  border: 1px solid rgba(255, 255, 255, 0.3);
  border-radius: 6px;
  cursor: pointer;
  background-color: transparent;
}

button {
  width: 100%;
  padding: 10px;
  border: none;
  border-radius: 6px;
  background-color: #4169e1;
  color: #fff;
  font-size: 14px;
  cursor: pointer;
  transition: background-color 0.2s;
}

button:hover {
  background-color: #3557c7;
}

.close-btn {
  background-color: rgba(255, 255, 255, 0.1);
  margin-top: 4px;
}

.close-btn:hover {
  background-color: rgba(255, 255, 255, 0.2);
}

.divider {
  height: 1px;
  background-color: rgba(255, 255, 255, 0.2);
  margin: 16px 0;
}

.fullscreen-section {
  margin-bottom: 0;
}

.fullscreen-btn {
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  font-size: 14px;
  font-weight: 500;
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 8px;
}

.fullscreen-btn:hover {
  background: linear-gradient(135deg, #5568d3 0%, #63408a 100%);
  transform: translateY(-1px);
  box-shadow: 0 4px 12px rgba(102, 126, 234, 0.4);
}

.wallpaper-section h3 {
  font-size: 16px;
  margin-bottom: 6px;
}

.hint {
  font-size: 11px;
  color: rgba(255, 255, 255, 0.6);
  margin-bottom: 12px;
}

.wallpaper-sizes {
  display: grid;
  grid-template-columns: repeat(2, 1fr);
  gap: 8px;
  margin-bottom: 12px;
}

.size-btn {
  padding: 10px 8px;
  background-color: rgba(255, 255, 255, 0.1);
  font-size: 12px;
  line-height: 1.3;
}

.size-btn:hover {
  background-color: rgba(255, 255, 255, 0.15);
}

.size-text {
  font-size: 10px;
  color: rgba(255, 255, 255, 0.7);
}

.custom-size {
  margin-bottom: 12px;
}

.custom-size label {
  display: block;
  margin-bottom: 8px;
  font-size: 14px;
}

.size-input-group {
  display: flex;
  gap: 8px;
  align-items: center;
}

.size-input-group input[type="number"] {
  flex: 1;
  padding: 8px 12px;
  border: 1px solid rgba(255, 255, 255, 0.3);
  border-radius: 6px;
  background-color: rgba(255, 255, 255, 0.1);
  color: #fff;
  font-size: 14px;
}

.separator {
  color: rgba(255, 255, 255, 0.6);
  font-size: 16px;
}

.download-custom-btn {
  padding: 8px 16px;
  background-color: #4169e1;
  white-space: nowrap;
}

.download-custom-btn:hover {
  background-color: #3557c7;
}
</style>
