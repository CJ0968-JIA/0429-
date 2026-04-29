<script setup>
import { ref, computed } from 'vue'

const customMinutes = ref(5)
const customSeconds = ref(0)
const totalSeconds = ref(0)
const remainingSeconds = ref(0)
const isRunning = ref(false)
const isPaused = ref(false)
const timerInterval = ref(null)

// 預設時間
const presetTimes = [
  { label: '3分鐘', minutes: 3 },
  { label: '5分鐘', minutes: 5 },
  { label: '10分鐘', minutes: 10 },
  { label: '15分鐘', minutes: 15 }
]

// 格式化時間
const formatTime = (seconds) => {
  const mins = Math.floor(seconds / 60)
  const secs = seconds % 60
  return `${String(mins).padStart(2, '0')}:${String(secs).padStart(2, '0')}`
}

// 時間顯示
const displayTime = computed(() => {
  return formatTime(remainingSeconds.value)
})

// 時間百分比
const timePercentage = computed(() => {
  if (totalSeconds.value === 0) return 100
  return (remainingSeconds.value / totalSeconds.value) * 100
})

// 顏色根據時間
const timeColor = computed(() => {
  return '#888'
})

// 播放提示音
const playSound = () => {
  const audioContext = new (window.AudioContext || window.webkitAudioContext)()
  const oscillator = audioContext.createOscillator()
  const gain = audioContext.createGain()

  oscillator.connect(gain)
  gain.connect(audioContext.destination)

  oscillator.frequency.value = 800
  oscillator.type = 'sine'

  gain.gain.setValueAtTime(0.3, audioContext.currentTime)
  gain.gain.exponentialRampToValueAtTime(0.01, audioContext.currentTime + 0.5)

  oscillator.start(audioContext.currentTime)
  oscillator.stop(audioContext.currentTime + 0.5)
}

// 設置時間
const setTime = (minutes = null, seconds = null) => {
  if (isRunning.value || isPaused.value) {
    alert('計時進行中，請先停止')
    return
  }

  if (minutes !== null) {
    totalSeconds.value = minutes * 60
    customMinutes.value = minutes
    customSeconds.value = 0
  } else {
    totalSeconds.value = customMinutes.value * 60 + customSeconds.value
  }

  remainingSeconds.value = totalSeconds.value
  isPaused.value = false
}

// 開始計時
const startTimer = () => {
  if (remainingSeconds.value === 0) {
    alert('請先設置時間')
    return
  }

  if (isPaused.value) {
    isPaused.value = false
    isRunning.value = true
  } else {
    isRunning.value = true
  }

  timerInterval.value = setInterval(() => {
    remainingSeconds.value--

    // 最後10秒閃爍效果
    if (remainingSeconds.value <= 10 && remainingSeconds.value > 0) {
      const element = document.querySelector('.timer-display')
      if (element) {
        element.style.animation = 'blink 0.5s infinite'
      }
    }

    // 時間到
    if (remainingSeconds.value <= 0) {
      remainingSeconds.value = 0
      clearInterval(timerInterval.value)
      isRunning.value = false

      // 播放多次提示音
      for (let i = 0; i < 3; i++) {
        setTimeout(() => playSound(), i * 300)
      }

      const element = document.querySelector('.timer-display')
      if (element) {
        element.style.animation = 'pulse 0.5s infinite'
      }
    }
  }, 1000)
}

// 暫停計時
const pauseTimer = () => {
  if (isRunning.value) {
    clearInterval(timerInterval.value)
    isRunning.value = false
    isPaused.value = true
  }
}

// 停止計時
const stopTimer = () => {
  clearInterval(timerInterval.value)
  isRunning.value = false
  isPaused.value = false
  remainingSeconds.value = totalSeconds.value
  const element = document.querySelector('.timer-display')
  if (element) {
    element.style.animation = 'none'
  }
}

// 重置計時
const resetTimer = () => {
  clearInterval(timerInterval.value)
  isRunning.value = false
  isPaused.value = false
  remainingSeconds.value = 0
  totalSeconds.value = 0
  customMinutes.value = 5
  customSeconds.value = 0
  const element = document.querySelector('.timer-display')
  if (element) {
    element.style.animation = 'none'
  }
}
</script>

<template>
  <div class="timer-container">
    <h2>⏱️ 報告計時器</h2>

    <div class="timer-layout">
      <!-- 計時顯示 -->
      <div class="timer-display-section">
        <div
          class="timer-display"
          :style="{ '--time-color': timeColor }"
        >
          {{ displayTime }}
        </div>

        <div class="progress-bar">
          <div
            class="progress-fill"
            :style="{ width: timePercentage + '%', background: timeColor }"
          ></div>
        </div>

        <div class="time-status">
          <span v-if="!isRunning && !isPaused" class="status-idle">就緒</span>
          <span v-else-if="isRunning" class="status-running">計時中... ⏱️</span>
          <span v-else-if="isPaused" class="status-paused">已暫停 ⏸️</span>
        </div>
      </div>

      <!-- 時間設置 -->
      <div class="section setup-section">
        <h3>時間設置</h3>

        <!-- 預設按鈕 -->
        <div class="preset-buttons">
          <button
            v-for="preset in presetTimes"
            :key="preset.minutes"
            @click="setTime(preset.minutes)"
            :disabled="isRunning || isPaused"
            class="btn btn-preset"
          >
            {{ preset.label }}
          </button>
        </div>

        <!-- 自訂時間 -->
        <div class="custom-time-section">
          <h4>自訂時間</h4>

          <div class="time-input-group">
            <div class="input-wrapper">
              <label>分鐘</label>
              <input
                v-model.number="customMinutes"
                type="number"
                min="0"
                max="99"
                :disabled="isRunning || isPaused"
              />
            </div>

            <div class="input-wrapper">
              <label>秒鐘</label>
              <input
                v-model.number="customSeconds"
                type="number"
                min="0"
                max="59"
                :disabled="isRunning || isPaused"
              />
            </div>

            <button
              @click="setTime()"
              :disabled="isRunning || isPaused"
              class="btn btn-primary"
            >
              設置
            </button>
          </div>
        </div>
      </div>

      <!-- 控制按鈕 -->
      <div class="section control-section">
        <h3>控制</h3>

        <div class="button-group">
          <button
            @click="startTimer"
            :disabled="isRunning || (totalSeconds === 0 && !isPaused)"
            class="btn btn-start"
          >
            {{ isPaused ? '繼續' : '開始' }}
          </button>

          <button
            v-if="isRunning"
            @click="pauseTimer"
            class="btn btn-pause"
          >
            暫停
          </button>

          <button
            @click="stopTimer"
            :disabled="!isRunning && !isPaused"
            class="btn btn-stop"
          >
            停止
          </button>

          <button
            @click="resetTimer"
            class="btn btn-reset"
          >
            重置
          </button>
        </div>
      </div>
    </div>
  </div>
</template>

<style scoped>
.timer-container h2 {
  color: #333;
  margin-bottom: 30px;
  text-align: center;
}

.timer-layout {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 30px;
}

/* 計時顯示區 */
.timer-display-section {
  background: linear-gradient(135deg, #ff8fa3 0%, #ffccd5 100%);
  border-radius: 15px;
  padding: 40px 20px;
  text-align: center;
  box-shadow: 0 10px 40px rgba(255, 143, 163, 0.3);
}

.timer-display {
  font-size: 5em;
  font-weight: bold;
  color: white;
  font-family: 'Courier New', monospace;
  letter-spacing: 10px;
  margin-bottom: 20px;
  text-shadow: 0 4px 10px rgba(0, 0, 0, 0.3);
}

.progress-bar {
  width: 100%;
  height: 10px;
  background: rgba(255, 255, 255, 0.2);
  border-radius: 10px;
  overflow: hidden;
  margin-bottom: 20px;
}

.progress-fill {
  height: 100%;
  transition: width 1s linear, background 0.3s ease;
  border-radius: 10px;
  box-shadow: 0 0 10px var(--time-color);
}

.time-status {
  font-size: 1.2em;
  color: white;
  font-weight: bold;
}

.status-idle {
  opacity: 0.7;
}

.status-running {
  animation: pulse 1s infinite;
}

.status-paused {
  opacity: 0.8;
}

@keyframes pulse {
  0%, 100% { opacity: 1; }
  50% { opacity: 0.6; }
}

@keyframes blink {
  0%, 100% { opacity: 1; }
  50% { opacity: 0.5; }
}

/* 設置區 */
.section {
  background: #f8f9fa;
  padding: 20px;
  border-radius: 8px;
  border-left: 4px solid #ff8fa3;
}

.section h3 {
  color: #333;
  margin-top: 0;
  margin-bottom: 20px;
}

.section h4 {
  color: #555;
  margin-bottom: 15px;
}

.preset-buttons {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 10px;
  margin-bottom: 20px;
}

.btn-preset {
  padding: 12px 15px;
  border: 2px solid #ff8fa3;
  background: #ff8fa3;
  color: #ffffff;
  border-radius: 5px;
  cursor: pointer;
  font-weight: bold;
  transition: all 0.3s ease;
}

.btn-preset:hover:not(:disabled) {
  opacity: 0.9;
  transform: translateY(-2px);
}

.btn-preset:disabled {
  opacity: 0.5;
  cursor: not-allowed;
}

/* 自訂時間輸入 */
.custom-time-section {
  background: white;
  padding: 15px;
  border-radius: 5px;
  border: 1px solid #ddd;
}

.time-input-group {
  display: grid;
  grid-template-columns: 1fr 1fr auto;
  gap: 10px;
  align-items: flex-end;
}

.input-wrapper {
  display: flex;
  flex-direction: column;
  gap: 5px;
}

.input-wrapper label {
  font-size: 0.9em;
  color: #666;
  font-weight: bold;
}

.input-wrapper input {
  padding: 8px;
  border: 1px solid #ddd;
  border-radius: 4px;
  font-size: 1em;
  text-align: center;
}

.input-wrapper input:focus {
  outline: none;
  border-color: #888;
  box-shadow: 0 0 5px rgba(136, 136, 136, 0.3);
}

.input-wrapper input:disabled {
  background: #f0f0f0;
  cursor: not-allowed;
}

/* 按鈕 */
.btn {
  padding: 10px 20px;
  border: none;
  border-radius: 5px;
  cursor: pointer;
  font-weight: bold;
  font-size: 1em;
  transition: all 0.3s ease;
}

.btn-primary {
  background: #ff8fa3;
  color: #ffffff;
}

.btn-primary:hover:not(:disabled) {
  opacity: 0.9;
  transform: scale(1.05);
}

.button-group {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 10px;
}

.btn-start {
  background: #ff8fa3;
  color: #ffffff;
  grid-column: 1 / -1;
  padding: 15px;
  font-size: 1.1em;
}

.btn-start:hover:not(:disabled) {
  opacity: 0.9;
  transform: scale(1.05);
  box-shadow: 0 5px 15px rgba(255, 143, 163, 0.4);
}

.btn-start:disabled {
  opacity: 0.5;
  cursor: not-allowed;
}

.btn-pause {
  background: #ff8fa3;
  color: #ffffff;
}

.btn-pause:hover {
  opacity: 0.9;
  transform: scale(1.05);
}

.btn-stop {
  background: #ff8fa3;
  color: #ffffff;
}

.btn-stop:hover:not(:disabled) {
  opacity: 0.9;
  transform: scale(1.05);
}

.btn-stop:disabled {
  opacity: 0.5;
  cursor: not-allowed;
}

.btn-reset {
  background: #ff8fa3;
  color: #ffffff;
}

.btn-reset:hover {
  opacity: 0.9;
  transform: scale(1.05);
}

/* 響應式設計 */
@media (max-width: 768px) {
  .timer-layout {
    grid-template-columns: 1fr;
  }

  .timer-display {
    font-size: 3em;
    letter-spacing: 5px;
  }

  .preset-buttons {
    grid-template-columns: 1fr;
  }

  .time-input-group {
    grid-template-columns: 1fr;
  }
}
</style>
