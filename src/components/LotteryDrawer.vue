<script setup>
import { ref, computed } from 'vue'

const nameInput = ref('')
const names = ref([])
const selectedName = ref(null)
const isDrawing = ref(false)
const drawHistory = ref([])
const isExcluded = ref(false)

// 加載保存的數據
const loadData = () => {
  const saved = localStorage.getItem('lotteryData')
  if (saved) {
    const data = JSON.parse(saved)
    names.value = data.names || []
    drawHistory.value = data.history || []
  }
}

// 保存數據
const saveData = () => {
  localStorage.setItem('lotteryData', JSON.stringify({
    names: names.value,
    history: drawHistory.value
  }))
}

// 添加名字
const addName = () => {
  if (nameInput.value.trim()) {
    names.value.push(nameInput.value.trim())
    nameInput.value = ''
    saveData()
  }
}

// 刪除名字
const removeName = (index) => {
  names.value.splice(index, 1)
  saveData()
}

// 清空列表
const clearNames = () => {
  if (confirm('確定要清空所有名單嗎？')) {
    names.value = []
    saveData()
  }
}

// 抽籤動畫
const drawName = async () => {
  if (names.value.length === 0) {
    alert('請先添加至少一個名字')
    return
  }

  isDrawing.value = true
  selectedName.value = null

  // 動畫效果：快速變換名字
  for (let i = 0; i < 30; i++) {
    selectedName.value = names.value[Math.floor(Math.random() * names.value.length)]
    await new Promise(resolve => setTimeout(resolve, 50))
  }

  // 最終結果
  selectedName.value = names.value[Math.floor(Math.random() * names.value.length)]

  // 添加到歷史記錄
  drawHistory.value.unshift({
    name: selectedName.value,
    time: new Date().toLocaleTimeString()
  })

  if (drawHistory.value.length > 10) {
    drawHistory.value = drawHistory.value.slice(0, 10)
  }

  saveData()
  isDrawing.value = false
}

// 排除已抽取的名字
const excludeName = () => {
  if (selectedName.value) {
    const index = names.value.indexOf(selectedName.value)
    if (index > -1) {
      names.value.splice(index, 1)
      selectedName.value = null
      saveData()
      alert('已排除此名字')
    }
  }
}

// 導入 CSV/文本
const importNames = (event) => {
  const file = event.target.files[0]
  if (!file) return

  const reader = new FileReader()
  reader.onload = (e) => {
    const text = e.target.result
    const importedNames = text.split('\n')
      .map(n => n.trim())
      .filter(n => n.length > 0)
    
    names.value = [...new Set([...names.value, ...importedNames])]
    saveData()
    event.target.value = ''
  }
  reader.readAsText(file)
}

// 初始化
loadData()

const availableNames = computed(() => {
  if (isExcluded.value && selectedName.value) {
    return names.value.filter(n => n !== selectedName.value)
  }
  return names.value
})
</script>

<template>
  <div class="lottery-container">
    <h2>🎲 隨機抽籤機</h2>

    <div class="lottery-layout">
      <!-- 左側：名單管理 -->
      <div class="section input-section">
        <h3>名單管理</h3>

        <div class="input-group">
          <input
            v-model="nameInput"
            type="text"
            placeholder="輸入名字"
            @keyup.enter="addName"
            class="input-field"
          />
          <button @click="addName" class="btn btn-primary">添加</button>
        </div>

        <div class="file-upload">
          <label class="file-label">
            📁 導入名單
            <input
              type="file"
              accept=".txt,.csv"
              @change="importNames"
              style="display: none"
            />
          </label>
        </div>

        <div class="names-list">
          <div
            v-for="(name, index) in names"
            :key="index"
            class="name-item"
          >
            <span>{{ name }}</span>
            <button @click="removeName(index)" class="btn-remove">✕</button>
          </div>
        </div>

        <div v-if="names.length > 0" class="list-stats">
          總數：<strong>{{ names.length }}</strong> 人
        </div>

        <button
          v-if="names.length > 0"
          @click="clearNames"
          class="btn btn-danger"
        >
          清空名單
        </button>
      </div>

      <!-- 右側：抽籤區 -->
      <div class="section draw-section">
        <h3>抽籤結果</h3>

        <div class="draw-display">
          <div class="selected-name" :class="{ active: selectedName }">
            {{ selectedName || '準備抽籤...' }}
          </div>
        </div>

        <div class="button-group">
          <button
            @click="drawName"
            :disabled="names.length === 0 || isDrawing"
            class="btn btn-draw"
          >
            {{ isDrawing ? '抽籤中...' : '開始抽籤' }}
          </button>

          <button
            v-if="selectedName"
            @click="excludeName"
            class="btn btn-secondary"
          >
            排除此名字
          </button>
        </div>

        <!-- 歷史記錄 -->
        <div v-if="drawHistory.length > 0" class="history">
          <h4>抽籤歷史</h4>
          <div class="history-list">
            <div
              v-for="(record, index) in drawHistory"
              :key="index"
              class="history-item"
            >
              <span class="history-rank">{{ index + 1 }}</span>
              <span class="history-name">{{ record.name }}</span>
              <span class="history-time">{{ record.time }}</span>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<style scoped>
.lottery-container {
  width: 100%;
}

.lottery-container h2 {
  color: #333;
  margin-bottom: 30px;
  text-align: center;
}

.lottery-layout {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 30px;
}

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

/* 輸入區 */
.input-group {
  display: flex;
  gap: 10px;
  margin-bottom: 15px;
}

.input-field {
  flex: 1;
  padding: 10px;
  border: 1px solid #ddd;
  border-radius: 5px;
  font-size: 1em;
}

.input-field:focus {
  outline: none;
  border-color: #ff8fa3;
  box-shadow: 0 0 5px rgba(102, 126, 234, 0.3);
}

.file-upload {
  margin-bottom: 15px;
}

.file-label {
  display: block;
  padding: 10px;
  background: #ff8fa3;
  color: #ffffff;
  border-radius: 5px;
  cursor: pointer;
  text-align: center;
  transition: all 0.3s ease;
  font-weight: bold;
}

.file-label:hover {
  opacity: 0.9;
}

/* 名單列表 */
.names-list {
  background: white;
  border-radius: 5px;
  padding: 10px;
  margin-bottom: 15px;
  max-height: 300px;
  overflow-y: auto;
  border: 1px solid #ddd;
}

.name-item {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 8px 10px;
  background: white;
  margin-bottom: 5px;
  border-radius: 3px;
  border-left: 3px solid #ff8fa3;
}

.name-item:last-child {
  margin-bottom: 0;
}

.btn-remove {
  background: #ff8fa3;
  color: #ffffff;
  border: none;
  border-radius: 3px;
  width: 25px;
  height: 25px;
  cursor: pointer;
  font-size: 0.9em;
  transition: all 0.3s ease;
}

.btn-remove:hover {
  background: #ff5252;
}

.list-stats {
  text-align: center;
  padding: 10px;
  background: white;
  border-radius: 5px;
  margin-bottom: 10px;
  border: 1px solid #ddd;
}

/* 抽籤區 */
.draw-display {
  background: linear-gradient(135deg, #ff8fa3 0%, #ffccd5 100%);
  border-radius: 10px;
  padding: 60px 20px;
  text-align: center;
  margin-bottom: 20px;
  min-height: 200px;
  display: flex;
  align-items: center;
  justify-content: center;
}

.selected-name {
  font-size: 3em;
  font-weight: bold;
  color: white;
  opacity: 0.3;
  transition: all 0.3s ease;
  text-shadow: 0 0 10px rgba(0, 0, 0, 0.3);
}

.selected-name.active {
  opacity: 1;
  transform: scale(1.1);
  animation: pulse 0.5s ease;
}

@keyframes pulse {
  0%, 100% { transform: scale(1); }
  50% { transform: scale(1.15); }
}

/* 按鈕 */
.btn {
  padding: 10px 20px;
  border: none;
  border-radius: 5px;
  cursor: pointer;
  font-size: 1em;
  font-weight: bold;
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

.btn-draw {
  background: #ff8fa3;
  color: #ffffff;
  padding: 15px 40px;
  font-size: 1.2em;
  width: 100%;
}

.btn-draw:hover:not(:disabled) {
  transform: scale(1.05);
  box-shadow: 0 5px 15px rgba(255, 143, 163, 0.4);
}

.btn-draw:disabled {
  opacity: 0.5;
  cursor: not-allowed;
}

.btn-secondary {
  background: #ff8fa3;
  color: #ffffff;
  width: 100%;
  margin-top: 10px;
}

.btn-secondary:hover {
  background: #45b8b0;
}

.btn-danger {
  background: #ff8fa3;
  color: #ffffff;
  width: 100%;
}

.btn-danger:hover {
  background: #ff5252;
}

.button-group {
  display: flex;
  flex-direction: column;
  gap: 10px;
}

/* 歷史記錄 */
.history {
  margin-top: 20px;
  background: white;
  padding: 15px;
  border-radius: 5px;
  border: 1px solid #ddd;
}

.history h4 {
  margin-top: 0;
  color: #333;
}

.history-list {
  display: flex;
  flex-direction: column;
  gap: 8px;
}

.history-item {
  display: grid;
  grid-template-columns: 30px 1fr 80px;
  gap: 10px;
  align-items: center;
  padding: 8px;
  background: #f8f9fa;
  border-radius: 3px;
  border-left: 3px solid #ff8fa3;
}

.history-rank {
  font-weight: bold;
  color: #ff8fa3;
  text-align: center;
}

.history-name {
  font-weight: 500;
}

.history-time {
  text-align: right;
  font-size: 0.9em;
  color: #999;
}

/* 響應式設計 */
@media (max-width: 768px) {
  .lottery-layout {
    grid-template-columns: 1fr;
  }

  .selected-name {
    font-size: 2em;
  }

  .draw-display {
    padding: 40px 20px;
  }
}
</style>
