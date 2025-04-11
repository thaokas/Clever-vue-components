<template>
  <div class="volume-control">
    <!-- 播放/暂停按钮 -->
    <button class="play-btn" @click="togglePlay" :disabled="!audioReady">
      {{ isPlaying ? 'Pause' : 'Play' }}
    </button>

    <!-- 原有音量控制按钮 -->
    <button class="volume-btn" @mouseover="startAdjust(-1)" @mouseleave="stopAdjust">-</button>
    <div class="volume-bar">
      <div class="volume-fill" :style="{ width: volume + '%' }"></div>
    </div>
    <button class="volume-btn" @mouseover="startAdjust(1)" @mouseleave="stopAdjust">+</button>
    <span class="volume-text">{{ volume }}%</span>
  </div>
</template>

<script lang="ts">
import { ref, onUnmounted, watch } from 'vue'

export default {
  setup() {
    // 音量控制
    const volume = ref<number>(50)
    let intervalId: number | null = null
    let currentDirection: number = 1

    // 音频相关
    const audio = new Audio('public/audio/audio.mp3') // 替换为实际音频路径
    audio.loop = true
    const isPlaying = ref<boolean>(false)
    const audioReady = ref<boolean>(false)

    // 监听音频加载完成
    audio.addEventListener('canplaythrough', () => {
      audioReady.value = true
    })

    // 音量同步
    watch(volume, (newVal) => {
      audio.volume = newVal / 100
    })

    // 播放/暂停切换
    const togglePlay = () => {
      if (isPlaying.value) {
        audio.pause()
      } else {
        audio.play()
      }
      isPlaying.value = !isPlaying.value
    }

    // 音量调整逻辑（原逻辑保持不变）
    const startAdjust = (direction: number) => {
      stopAdjust()
      currentDirection = direction
      let interval = 150 // 初始间隔

      const adjust = () => {
        volume.value = Math.min(
          Math.max(volume.value + currentDirection * 1, 0),
          100
        )
        interval = Math.max(interval * 0.9, 50)
        intervalId = setTimeout(adjust, interval)
      }

      intervalId = setTimeout(adjust, interval)
    }

    const stopAdjust = () => {
      if (intervalId) {
        clearTimeout(intervalId)
        intervalId = null
      }
    }

    // 组件卸载时清理
    onUnmounted(() => {
      audio.pause()
      audio.src = ''
      stopAdjust()
    })

    return {
      volume,
      startAdjust,
      stopAdjust,
      togglePlay,
      isPlaying,
      audioReady
    }
  }
}
</script>

<style scoped>
.volume-control {
  display: flex;
  align-items: center;
  gap: 10px;
  padding: 20px;
  background: #f5f5f5;
  border-radius: 8px;
}

.volume-bar {
  width: 100%;
  height: 20px;
  background-color: #eee;
  border-radius: 10px;
  overflow: hidden;
}

.volume-fill {
  height: 100%;
  background-color: #4CAF50;
  transition: width 0.3s ease;
}

.volume-btn {
  padding: 8px 16px;
  cursor: pointer;
  border: none;
  background-color: #ddd;
  border-radius: 5px;
  font-weight: bold;
  transition: background-color 0.2s;
}

.volume-text {
  margin-left: 10px;
  font-size: 16px;
  font-weight: bold;
  color: #333;
}

.volume-btn:hover {
  background-color: #ccc;
}

/* 在<style scoped>中添加 */
.play-btn {
  margin-right: 10px;
  padding: 8px 16px;
  cursor: pointer;
  border: none;
  background-color: #4CAF50;
  color: white;
  border-radius: 5px;
  transition: background-color 0.2s;
}

.play-btn:hover {
  background-color: #45a049;
}

.play-btn:disabled {
  background-color: #ccc;
  cursor: not-allowed;
}
</style>
