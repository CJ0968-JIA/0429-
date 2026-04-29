<script setup>
import { ref, watch } from 'vue'
import VinylBackground from './components/VinylBackground.vue'
import LotteryDrawer from './components/LotteryDrawer.vue'
import ScoreBoard from './components/ScoreBoard.vue'
import Timer from './components/Timer.vue'
import ExamSystem from './components/ExamSystem.vue'

const currentTool = ref('home')
const isSpinning = ref(false)
const audioRef = ref(null)

const tools = [
  { id: 'home', name: '首頁' },
  { id: 'lottery', name: '抽籤機' },
  { id: 'scoreboard', name: '計分板' },
  { id: 'timer', name: '計時器' },
  { id: 'exam', name: '測驗系統' }
]

// 監聽旋轉狀態來控制音樂
watch(isSpinning, (newVal) => {
  if (newVal) {
    audioRef.value?.play().catch(e => console.log('音樂播放被瀏覽器阻擋，需使用者點擊觸發', e))
  } else {
    audioRef.value?.pause()
  }
})
</script>

<template>
  <VinylBackground :isSpinning="isSpinning" />

  <!-- 隱藏的背景音樂，使用公開的鋼琴音樂連結 -->
  <audio ref="audioRef" src="https://www.soundhelix.com/examples/mp3/SoundHelix-Song-2.mp3" loop></audio>

  <div class="app">
    <header class="app-header">
      <h1>🎓 互動式課堂管理與測驗平台</h1>
      <nav class="nav-tabs">
        <button
          v-for="tool in tools"
          :key="tool.id"
          :class="['nav-btn', { active: currentTool === tool.id }]"
          @click="currentTool = tool.id"
        >
          {{ tool.name }}
        </button>
        <!-- 唱片控制按鈕 -->
        <button 
          class="nav-btn toggle-btn" 
          :class="{ 'active-spin': isSpinning }"
          @click="isSpinning = !isSpinning"
        >
          {{ isSpinning ? '⏸ 停止音樂' : '▶ 啟動唱片' }}
        </button>
      </nav>
    </header>

    <main class="app-main">
      <!-- 首頁：不顯示內容，僅露出背景唱片 -->
      <section v-if="currentTool === 'home'" class="home-hero">
        <div class="click-hint" v-if="!isSpinning" @click="isSpinning = true">
          <span>🖱️ 點擊上方按鈕或此處啟動唱片機</span>
        </div>
      </section>

      <!-- 抽籤機 -->
      <section v-if="currentTool === 'lottery'" class="tool-section">
        <LotteryDrawer />
      </section>

      <!-- 計分板 -->
      <section v-if="currentTool === 'scoreboard'" class="tool-section">
        <ScoreBoard />
      </section>

      <!-- 計時器 -->
      <section v-if="currentTool === 'timer'" class="tool-section">
        <Timer />
      </section>

      <!-- 測驗系統 -->
      <section v-if="currentTool === 'exam'" class="tool-section">
        <ExamSystem />
      </section>
    </main>
  </div>
</template>

<style scoped>
.app {
  min-height: 100vh;
  display: flex;
  flex-direction: column;
  background: transparent; /* 移除原本的背景，讓黑膠唱片機顯示出來 */
}

.app-header {
  background: rgba(0, 0, 0, 0.8);
  color: white;
  padding: 20px;
  text-align: center;
  box-shadow: 0 4px 10px rgba(0, 0, 0, 0.3);
}

.app-header h1 {
  margin: 0 0 20px 0;
  font-size: 2em;
  text-shadow: 0 2px 10px rgba(0, 0, 0, 0.2);
}

.nav-tabs {
  display: flex;
  gap: 10px;
  justify-content: center;
  flex-wrap: wrap;
}

.nav-btn {
  padding: 10px 20px;
  border: none;
  background: #ff8fa3;
  color: #ffffff;
  border-radius: 5px;
  cursor: pointer;
  font-size: 1em;
  font-weight: bold;
  transition: all 0.3s ease;
}

.nav-btn:hover {
  opacity: 0.9;
  color: white;
  transform: scale(1.05);
}

.nav-btn.active {
  background: #ff8fa3;
  color: #ffffff;
  box-shadow: 0 0 10px rgba(255, 143, 163, 0.5);
}

.toggle-btn {
  background: #ff8fa3;
  margin-left: 20px;
  border: 2px solid #ffffff;
}

.toggle-btn.active-spin {
  background: #ff8fa3;
  color: #ffffff;
}

.home-hero {
  height: 60vh;
  display: flex;
  align-items: center;
  justify-content: center;
}

.click-hint {
  cursor: pointer;
  font-size: 1.2rem;
  color: #555;
  background: rgba(255, 255, 255, 0.3);
  padding: 15px 30px;
  border-radius: 30px;
  transition: all 0.3s ease;
}

.app-main {
  flex: 1;
  padding: 30px;
  overflow-y: auto;
}

.tool-section {
  background: #ffffff;
  border-radius: 10px;
  padding: 30px;
  box-shadow: 0 8px 32px rgba(0, 0, 0, 0.2);
  animation: slideIn 0.3s ease;
}

@keyframes slideIn {
  from {
    opacity: 0;
    transform: translateY(20px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
}

.home-content h2 {
  text-align: center;
  color: #333;
  margin-bottom: 40px;
  font-size: 2em;
}

.feature-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(250px, 1fr));
  gap: 20px;
}

.feature-card {
  background: linear-gradient(135deg, #ff8fa3 0%, #ffccd5 100%);
  color: white;
  padding: 30px;
  border-radius: 10px;
  text-align: center;
  transition: all 0.3s ease;
  box-shadow: 0 4px 15px rgba(255, 143, 163, 0.3);
}

.feature-card:hover {
  transform: translateY(-10px);
  box-shadow: 0 8px 25px rgba(255, 143, 163, 0.5);
}

.feature-icon {
  font-size: 3em;
  margin-bottom: 15px;
}

.feature-card h3 {
  margin: 15px 0;
  font-size: 1.5em;
}

.feature-card p {
  margin: 0;
  opacity: 0.9;
}
</style>
