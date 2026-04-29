<script setup>
import { ref, computed } from 'vue'

const teamName = ref('')
const teams = ref([])
const sortOrder = ref('desc')

// 加載數據
const loadData = () => {
  const saved = localStorage.getItem('scoreBoardData')
  if (saved) {
    teams.value = JSON.parse(saved)
  }
}

// 保存數據
const saveData = () => {
  localStorage.setItem('scoreBoardData', JSON.stringify(teams.value))
}

// 添加小組
const addTeam = () => {
  if (teamName.value.trim()) {
    teams.value.push({
      id: Date.now(),
      name: teamName.value.trim(),
      score: 0
    })
    teamName.value = ''
    saveData()
  }
}

// 刪除小組
const removeTeam = (id) => {
  const index = teams.value.findIndex(t => t.id === id)
  if (index > -1) {
    teams.value.splice(index, 1)
    saveData()
  }
}

// 更新分數
const updateScore = (id, delta) => {
  const team = teams.value.find(t => t.id === id)
  if (team) {
    team.score = Math.max(0, team.score + delta)
    saveData()
  }
}

// 設置分數
const setScore = (id, newScore) => {
  const team = teams.value.find(t => t.id === id)
  if (team) {
    const score = parseInt(newScore)
    if (!isNaN(score)) {
      team.score = Math.max(0, score)
      saveData()
    }
  }
}

// 清空所有分數
const resetScores = () => {
  if (confirm('確定要清空所有分數嗎？')) {
    teams.value.forEach(team => team.score = 0)
    saveData()
  }
}

// 刪除所有小組
const clearAllTeams = () => {
  if (confirm('確定要刪除所有小組嗎？')) {
    teams.value = []
    saveData()
  }
}

// 排序的小組
const sortedTeams = computed(() => {
  const sorted = [...teams.value].sort((a, b) => {
    return sortOrder.value === 'desc' ? b.score - a.score : a.score - b.score
  })
  return sorted
})

// 排名
const getRank = (index) => {
  if (index === 0) return '🥇'
  if (index === 1) return '🥈'
  if (index === 2) return '🥉'
  return `${index + 1}、`
}

// 初始化
loadData()
</script>

<template>
  <div class="scoreboard-container">
    <h2>🏆 小組競賽計分板</h2>

    <div class="scoreboard-layout">
      <!-- 左側：小組管理 -->
      <div class="section team-input">
        <h3>小組管理</h3>

        <div class="input-group">
          <input
            v-model="teamName"
            type="text"
            placeholder="輸入小組名稱"
            @keyup.enter="addTeam"
            class="input-field"
          />
          <button @click="addTeam" class="btn btn-primary">新增</button>
        </div>

        <div class="teams-input-list">
          <div
            v-for="team in teams"
            :key="team.id"
            class="team-input-item"
          >
            <span class="team-input-name">{{ team.name }}</span>
            <button @click="removeTeam(team.id)" class="btn-remove">✕</button>
          </div>
        </div>

        <div v-if="teams.length > 0" class="team-stats">
          小組數：<strong>{{ teams.length }}</strong>
        </div>

        <div class="button-group">
          <button
            v-if="teams.length > 0"
            @click="resetScores"
            class="btn btn-secondary"
          >
            重置分數
          </button>
          <button
            v-if="teams.length > 0"
            @click="clearAllTeams"
            class="btn btn-danger"
          >
            刪除全部
          </button>
        </div>
      </div>

      <!-- 右側：計分區 -->
      <div class="section scoreboard-display">
        <h3>即時排名</h3>

        <div class="sort-controls">
          <label>
            <input
              type="radio"
              value="desc"
              v-model="sortOrder"
            />
            從高到低
          </label>
          <label>
            <input
              type="radio"
              value="asc"
              v-model="sortOrder"
            />
            從低到高
          </label>
        </div>

        <div v-if="teams.length === 0" class="empty-state">
          <p>還沒有任何小組，請先新增小組</p>
        </div>

        <div v-else class="ranking-table">
          <div
            v-for="(team, index) in sortedTeams"
            :key="team.id"
            :class="['ranking-row', { 'top-rank': index < 3 }]"
          >
            <div class="rank-medal">{{ getRank(index) }}</div>

            <div class="team-info">
              <div class="team-name">{{ team.name }}</div>
              <div class="team-score">{{ team.score }}</div>
            </div>

            <div class="score-controls">
              <button
                @click="updateScore(team.id, -5)"
                class="btn-score btn-minus"
              >
                -5
              </button>
              <button
                @click="updateScore(team.id, -1)"
                class="btn-score btn-minus"
              >
                -1
              </button>

              <input
                :value="team.score"
                @change="e => setScore(team.id, e.target.value)"
                type="number"
                class="score-input"
              />

              <button
                @click="updateScore(team.id, 1)"
                class="btn-score btn-plus"
              >
                +1
              </button>
              <button
                @click="updateScore(team.id, 5)"
                class="btn-score btn-plus"
              >
                +5
              </button>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<style scoped>
.scoreboard-container h2 {
  color: #333;
  margin-bottom: 30px;
  text-align: center;
}

.scoreboard-layout {
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

/* 小組列表 */
.teams-input-list {
  background: white;
  border-radius: 5px;
  padding: 10px;
  margin-bottom: 15px;
  max-height: 300px;
  overflow-y: auto;
  border: 1px solid #ddd;
}

.team-input-item {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 8px 10px;
  margin-bottom: 5px;
  background: white;
  border-radius: 3px;
  border-left: 3px solid #ff8fa3;
}

.team-input-item:last-child {
  margin-bottom: 0;
}

.team-input-name {
  font-weight: 500;
}

.btn-remove {
  background: #ff8fa3;
  color: #ffffff;
  border: none;
  border-radius: 3px;
  width: 25px;
  height: 25px;
  cursor: pointer;
  transition: all 0.3s ease;
}

.btn-remove:hover {
  background: #ff5252;
}

.team-stats {
  text-align: center;
  padding: 10px;
  background: white;
  border-radius: 5px;
  margin-bottom: 10px;
  border: 1px solid #ddd;
}

.button-group {
  display: flex;
  flex-direction: column;
  gap: 10px;
}

.btn {
  padding: 10px 20px;
  border: none;
  border-radius: 5px;
  cursor: pointer;
  font-weight: bold;
  transition: all 0.3s ease;
}

.btn-primary {
  background: #ff8fa3;
  color: #ffffff;
  flex: 1;
}

.btn-primary:hover {
  opacity: 0.9;
}

.btn-secondary {
  background: #ff8fa3;
  color: #ffffff;
}

.btn-secondary:hover {
  opacity: 0.9;
}

.btn-danger {
  background: #ff8fa3;
  color: #ffffff;
}

.btn-danger:hover {
  background: #ff5252;
}

/* 排序控制 */
.sort-controls {
  display: flex;
  gap: 20px;
  margin-bottom: 20px;
  padding: 10px;
  background: white;
  border-radius: 5px;
  border: 1px solid #ddd;
}

.sort-controls label {
  display: flex;
  align-items: center;
  gap: 8px;
  cursor: pointer;
  font-weight: 500;
}

.sort-controls input[type="radio"] {
  cursor: pointer;
}

/* 空狀態 */
.empty-state {
  text-align: center;
  padding: 40px 20px;
  color: #999;
}

/* 排名表格 */
.ranking-table {
  display: flex;
  flex-direction: column;
  gap: 10px;
}

.ranking-row {
  display: grid;
  grid-template-columns: 50px 1fr auto;
  gap: 15px;
  align-items: center;
  padding: 15px;
  background: white;
  border-radius: 8px;
  border-left: 4px solid #ddd;
  transition: all 0.3s ease;
}

.ranking-row:hover {
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.1);
}

.ranking-row.top-rank {
  background: linear-gradient(135deg, rgba(255, 143, 163, 0.1) 0%, rgba(255, 204, 213, 0.1) 100%);
  border-left-color: #ff8fa3;
  transform: scale(1.02);
}

.rank-medal {
  font-size: 1.8em;
  text-align: center;
}

.team-info {
  display: flex;
  flex-direction: column;
  gap: 5px;
}

.team-name {
  font-weight: bold;
  font-size: 1.1em;
  color: #333;
}

.team-score {
  font-size: 0.9em;
  color: #ff8fa3;
}

.score-controls {
  display: flex;
  gap: 5px;
  align-items: center;
}

.btn-score {
  padding: 8px 12px;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  font-weight: bold;
  font-size: 0.85em;
  transition: all 0.2s ease;
}

.btn-plus {
  background: #ff8fa3;
  color: #ffffff;
}

.btn-plus:hover {
  opacity: 0.9;
  transform: scale(1.1);
}

.btn-minus {
  background: #ff8fa3;
  color: #ffffff;
}

.btn-minus:hover {
  opacity: 0.9;
  transform: scale(1.1);
}

.score-input {
  width: 50px;
  padding: 8px;
  border: 2px solid #ff8fa3;
  border-radius: 4px;
  text-align: center;
  font-weight: bold;
  font-size: 1.1em;
}

.score-input:focus {
  outline: none;
  box-shadow: 0 0 8px rgba(255, 143, 163, 0.4);
}

/* 響應式設計 */
@media (max-width: 768px) {
  .scoreboard-layout {
    grid-template-columns: 1fr;
  }

  .ranking-row {
    grid-template-columns: 40px 1fr;
  }

  .score-controls {
    grid-column: 1 / -1;
    margin-top: 10px;
  }
}
</style>
