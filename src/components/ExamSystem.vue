<script setup>
import { ref, computed, watch } from 'vue'

const isEditMode = ref(false)
const currentQuestion = ref(0)
const userAnswers = ref({})
const showResults = ref(false)
const questions = ref([])

const defaultQuestions = [
  { id: 1, type: 'single', question: '下列何者為化學變化？', options: ['冰塊融化', '水蒸發', '燃燒', '切割紙張'], correct: 2 },
  { id: 2, type: 'multiple', question: '下列哪些是可再生能源？ (可複選)', options: ['太陽能', '風能', '煤炭', '地熱能'], correct: [0, 1, 3] },
  { id: 3, type: 'truefalse', question: '光速是自然界中最快的速度。', options: ['正確', '錯誤'], correct: 0 }
]

// 題目編輯用的暫存變數
const editingQuestion = ref({
  question: '',
  type: 'single',
  options: ['', ''],
  correct: null
})

// 加載題目數據
const loadQuestions = () => {
  const saved = localStorage.getItem('examQuestions')
  if (saved) {
    questions.value = JSON.parse(saved)
  } else {
    questions.value = [...defaultQuestions]
  }
}

// 保存題目數據
const saveQuestions = () => {
  localStorage.setItem('examQuestions', JSON.stringify(questions.value))
}

// 新增題目
const addQuestion = () => {
  const newId = questions.value.length > 0 ? Math.max(...questions.value.map(q => q.id)) + 1 : 1
  questions.value.push({
    id: newId,
    type: 'single',
    question: '新題目',
    options: ['選項 A', '選項 B'],
    correct: 0
  })
  saveQuestions()
}

// 刪除題目
const deleteQuestion = (index) => {
  if (confirm('確定要刪除這道題目嗎？')) {
    questions.value.splice(index, 1)
    if (currentQuestion.value >= questions.value.length) {
      currentQuestion.value = Math.max(0, questions.value.length - 1)
    }
    saveQuestions()
  }
}

// 處理題目類型切換時的答案重置
const handleTypeChange = (question) => {
  if (question.type === 'multiple') {
    question.correct = []
    question.options = ['選項 A', '選項 B', '選項 C']
  } else if (question.type === 'truefalse') {
    question.options = ['正確', '錯誤']
    question.correct = 0
  } else {
    question.correct = 0
    question.options = ['選項 A', '選項 B']
  }
  saveQuestions()
}

// 動態增減選項
const addOption = (question) => {
  question.options.push(`新選項 ${String.fromCharCode(65 + question.options.length)}`)
  saveQuestions()
}

const removeOption = (question, index) => {
  if (question.options.length <= 2) return
  question.options.splice(index, 1)
  // 如果刪掉的是正確答案，重置答案
  if (question.type === 'multiple') {
    question.correct = question.correct.filter(i => i !== index).map(i => i > index ? i - 1 : i)
  } else if (question.correct === index) {
    question.correct = 0
  } else if (question.correct > index) {
    question.correct--
  }
  saveQuestions()
}

// 編輯模式下的答案勾選邏輯
const toggleCorrectAnswer = (question, index) => {
  if (question.type === 'multiple') {
    const idx = question.correct.indexOf(index)
    if (idx > -1) question.correct.splice(idx, 1)
    else question.correct.push(index)
  } else {
    question.correct = index
  }
  saveQuestions()
}

// 加載已保存的答卷
const loadExam = () => {
  const saved = localStorage.getItem('examAnswers')
  if (saved) {
    userAnswers.value = JSON.parse(saved)
  }
}

// 保存答卷
const saveExam = () => {
  localStorage.setItem('examAnswers', JSON.stringify(userAnswers.value))
}

// 獲取當前題目
const getCurrentQuestion = computed(() => {
  return questions.value[currentQuestion.value]
})

// 選擇答案
const selectAnswer = (optionIndex) => {
  const qId = getCurrentQuestion.value.id
  const question = getCurrentQuestion.value

  if (question.type === 'multiple') {
    if (!userAnswers.value[qId]) {
      userAnswers.value[qId] = []
    }
    const answers = userAnswers.value[qId]
    const idx = answers.indexOf(optionIndex)
    if (idx > -1) {
      answers.splice(idx, 1)
    } else {
      answers.push(optionIndex)
    }
  } else {
    userAnswers.value[qId] = optionIndex
  }

  saveExam()
}

// 檢查答案是否被選中
const isAnswerSelected = (optionIndex) => {
  const qId = getCurrentQuestion.value.id
  const answer = userAnswers.value[qId]

  if (Array.isArray(answer)) {
    return answer.includes(optionIndex)
  }
  return answer === optionIndex
}

// 下一題
const nextQuestion = () => {
  if (currentQuestion.value < questions.value.length - 1) {
    currentQuestion.value++
  }
}

// 上一題
const previousQuestion = () => {
  if (currentQuestion.value > 0) {
    currentQuestion.value--
  }
}

// 跳轉到指定題目
const goToQuestion = (index) => {
  currentQuestion.value = index
}

// 計算分數
const calculateScore = computed(() => {
  let correct = 0

  questions.value.forEach(question => {
    const qId = question.id
    const userAnswer = userAnswers.value[qId]

    if (question.type === 'multiple') {
      const correctAnswers = question.correct.sort()
      const userAnswers_ = Array.isArray(userAnswer) ? userAnswer.sort() : []

      if (JSON.stringify(correctAnswers) === JSON.stringify(userAnswers_)) {
        correct++
      }
    } else {
      if (userAnswer === question.correct) {
        correct++
      }
    }
  })

  return {
    correct,
    total: questions.value.length,
    percentage: Math.round((correct / questions.value.length) * 100)
  }
})

// 提交答卷
const submitExam = () => {
  const unanswered = questions.value.filter(q => userAnswers.value[q.id] === undefined).length

  if (unanswered > 0) {
    if (!confirm(`還有 ${unanswered} 題未作答，是否提交？`)) {
      return
    }
  }

  showResults.value = true
}

// 重新開始考試
const restartExam = () => {
  if (confirm('確定要重新開始考試嗎？所有答案將被清除。')) {
    userAnswers.value = {}
    currentQuestion.value = 0
    showResults.value = false
    localStorage.removeItem('examAnswers')
  }
}

// 驗證答案是否正確
const isAnswerCorrect = (qId) => {
  const question = questions.value.find(q => q.id === qId)
  const userAnswer = userAnswers.value[qId]

  if (question.type === 'multiple') {
    const correctAnswers = question.correct.sort()
    const userAnswers_ = Array.isArray(userAnswer) ? userAnswer.sort() : []
    return JSON.stringify(correctAnswers) === JSON.stringify(userAnswers_)
  } else {
    return userAnswer === question.correct
  }
}

// 初始化
loadQuestions()
loadExam()
</script>

<template>
  <div class="exam-container">
    <div class="exam-header-main">
      <h2>📝 數位測驗系統</h2>
      <div class="mode-toggle">
        <button @click="isEditMode = !isEditMode" class="btn btn-mode">
          {{ isEditMode ? '📖 結束設計' : '🛠️ 設計題目' }}
        </button>
      </div>
    </div>

    <!-- 題目設計模式 -->
    <div v-if="isEditMode" class="edit-page">
      <div class="exam-layout">
        <div class="section question-nav">
          <h3>題目清單</h3>
          <div class="edit-list">
            <div v-for="(q, index) in questions" :key="q.id" 
                 class="edit-nav-item" :class="{ active: currentQuestion === index }"
                 @click="currentQuestion = index">
              <span>#{{ index + 1 }} {{ q.question.substring(0, 10) }}...</span>
              <button @click.stop="deleteQuestion(index)" class="btn-mini-danger">✕</button>
            </div>
          </div>
          <button @click="addQuestion" class="btn btn-secondary btn-full">＋ 新增題目</button>
        </div>

        <div class="section question-section" v-if="questions.length > 0">
          <div class="edit-form">
            <div class="form-group">
              <label>題目內容：</label>
              <textarea v-model="questions[currentQuestion].question" @change="saveQuestions" class="input-field"></textarea>
            </div>

            <div class="form-group">
              <label>題型：</label>
              <select v-model="questions[currentQuestion].type" @change="handleTypeChange(questions[currentQuestion])" class="input-field">
                <option value="single">單選題</option>
                <option value="multiple">複選題</option>
                <option value="truefalse">是非題</option>
              </select>
            </div>

            <div class="form-group">
              <label>選項設定 (點擊圖示設定正確答案)：</label>
              <div v-for="(opt, idx) in questions[currentQuestion].options" :key="idx" class="edit-option-row">
                <div class="correct-marker" @click="toggleCorrectAnswer(questions[currentQuestion], idx)">
                  <span v-if="questions[currentQuestion].type === 'multiple'">
                    {{ questions[currentQuestion].correct.includes(idx) ? '✅' : '⬜' }}
                  </span>
                  <span v-else>
                    {{ questions[currentQuestion].correct === idx ? '🎯' : '⭕' }}
                  </span>
                </div>
                <input v-model="questions[currentQuestion].options[idx]" @change="saveQuestions" class="input-field" />
                <button v-if="questions[currentQuestion].type !== 'truefalse'" @click="removeOption(questions[currentQuestion], idx)" class="btn-mini-danger">✕</button>
              </div>
              <button v-if="questions[currentQuestion].type !== 'truefalse'" @click="addOption(questions[currentQuestion])" class="btn-mini-link">＋ 新增選項</button>
            </div>
          </div>
        </div>
        <div v-else class="section question-section empty-edit">
          <p>目前沒有題目，請點擊左側「新增題目」開始設計。</p>
        </div>
      </div>
    </div>

    <!-- 結果頁面 -->
    <div v-else-if="showResults" class="results-page">
      <div class="score-card">
        <div class="score-circle">
          <div class="score-number">{{ calculateScore.percentage }}%</div>
          <div class="score-text">{{ calculateScore.correct }}/{{ calculateScore.total }}</div>
        </div>

        <div class="score-message">
          <h3 v-if="calculateScore.percentage >= 80">🎉 優秀！</h3>
          <h3 v-else-if="calculateScore.percentage >= 60">✅ 合格</h3>
          <h3 v-else>💪 繼續加油</h3>
        </div>
      </div>

      <!-- 詳細答案檢視 -->
      <div class="results-detail">
        <h3>答案詳解</h3>

        <div
          v-for="(question, index) in questions"
          :key="question.id"
          :class="['result-item', { correct: isAnswerCorrect(question.id), incorrect: !isAnswerCorrect(question.id) }]"
        >
          <div class="result-header">
            <div class="result-number">第 {{ index + 1 }} 題</div>
            <div class="result-status">
              <span v-if="isAnswerCorrect(question.id)" class="badge correct">✓ 正確</span>
              <span v-else class="badge incorrect">✗ 錯誤</span>
            </div>
          </div>

          <p class="result-question">{{ question.question }}</p>

          <div class="result-answers">
            <div
              v-for="(option, idx) in question.options"
              :key="idx"
              :class="[
                'result-option',
                {
                  'user-answer': userAnswers[question.id]?.includes ? userAnswers[question.id].includes(idx) : userAnswers[question.id] === idx,
                  'correct-answer': question.type === 'multiple' ? question.correct.includes(idx) : question.correct === idx,
                  'incorrect-answer': userAnswers[question.id] === idx && !isAnswerCorrect(question.id)
                }
              ]"
            >
              <span class="option-letter">{{ String.fromCharCode(65 + idx) }}</span>
              <span>{{ option }}</span>
            </div>
          </div>
        </div>
      </div>

      <!-- 結果按鈕 -->
      <div class="result-actions">
        <button @click="restartExam" class="btn btn-primary">
          重新開始
        </button>
      </div>
    </div>

    <!-- 考試頁面 -->
    <div v-else class="exam-page">
      <div class="exam-layout">
        <!-- 左側：題目導覽 -->
        <div class="section question-nav">
          <h3>題目導覽</h3>

          <div class="nav-grid">
            <button
              v-for="(question, index) in questions"
              :key="question.id"
              @click="goToQuestion(index)"
              :class="[
                'nav-btn',
                {
                  active: currentQuestion === index,
                  answered: userAnswers[question.id] !== undefined
                }
              ]"
            >
              <span class="nav-number">{{ index + 1 }}</span>
              <span v-if="userAnswers[question.id] !== undefined" class="nav-check">✓</span>
            </button>
          </div>

          <div class="progress-section">
            <div class="progress-text">
              已作答：<strong>{{ Object.keys(userAnswers).length }}</strong>/{{ questions.length }}
            </div>
            <div class="progress-bar">
              <div
                class="progress-fill"
                :style="{ width: (Object.keys(userAnswers).length / questions.length) * 100 + '%' }"
              ></div>
            </div>
          </div>
        </div>

        <!-- 右側：題目區 -->
        <div class="section question-section">
          <!-- 題號和類型 -->
          <div class="question-header">
            <div class="question-meta">
              <span class="question-number">第 {{ currentQuestion + 1 }} 題 / 共 {{ questions.length }} 題</span>
              <span :class="['question-type', `type-${getCurrentQuestion.type}`]">
                {{ getCurrentQuestion.type === 'single' ? '單選' : getCurrentQuestion.type === 'multiple' ? '複選' : '是非' }}
              </span>
            </div>
          </div>

          <!-- 題目 -->
          <h3 class="question-text">{{ getCurrentQuestion.question }}</h3>

          <!-- 選項 -->
          <div class="options">
            <button
              v-for="(option, index) in getCurrentQuestion.options"
              :key="index"
              @click="selectAnswer(index)"
              :class="['option-btn', { selected: isAnswerSelected(index) }]"
            >
              <span class="option-letter">{{ String.fromCharCode(65 + index) }}</span>
              <span class="option-text">{{ option }}</span>
              <span v-if="isAnswerSelected(index)" class="option-check">✓</span>
            </button>
          </div>

          <!-- 導航按鈕 -->
          <div class="navigation">
            <button
              @click="previousQuestion"
              :disabled="currentQuestion === 0"
              class="btn btn-secondary"
            >
              ← 上一題
            </button>

            <button
              v-if="currentQuestion < questions.length - 1"
              @click="nextQuestion"
              class="btn btn-secondary"
            >
              下一題 →
            </button>

            <button
              v-else
              @click="submitExam"
              class="btn btn-submit"
            >
              提交答卷
            </button>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<style scoped>
.exam-container h2 {
  color: #333;
  margin-bottom: 30px;
  text-align: center;
}

.exam-header-main {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 20px;
}

.btn-mode {
  background: #ff8fa3;
  color: white;
  padding: 8px 16px;
  border-radius: 20px;
  font-size: 0.9em;
}

.edit-list {
  max-height: 400px;
  overflow-y: auto;
  margin-bottom: 15px;
}

.edit-nav-item {
  display: flex;
  justify-content: space-between;
  padding: 10px;
  background: white;
  margin-bottom: 5px;
  border-radius: 5px;
  cursor: pointer;
  border: 1px solid #ddd;
}

.edit-nav-item.active {
  border-color: #ff8fa3;
  background: #fff0f3;
}

.btn-mini-danger {
  background: #ff6b6b;
  color: white;
  border: none;
  border-radius: 3px;
  padding: 2px 6px;
  cursor: pointer;
}

.edit-option-row {
  display: flex;
  gap: 10px;
  align-items: center;
  margin-bottom: 8px;
}

.correct-marker {
  cursor: pointer;
  font-size: 1.2em;
  width: 30px;
}

.btn-mini-link {
  background: none;
  border: none;
  color: #ff8fa3;
  cursor: pointer;
  font-weight: bold;
  padding: 5px;
}

.form-group {
  margin-bottom: 20px;
}

.form-group label {
  display: block;
  font-weight: bold;
  margin-bottom: 8px;
  color: #555;
}

.btn-full {
  width: 100%;
}

/* 結果頁面 */
.results-page {
  max-width: 900px;
  margin: 0 auto;
}

.score-card {
  background: linear-gradient(135deg, #ff8fa3 0%, #ffccd5 100%);
  border-radius: 15px;
  padding: 40px;
  text-align: center;
  color: #ffffff;
  margin-bottom: 30px;
  box-shadow: 0 10px 40px rgba(255, 143, 163, 0.3);
}

.score-circle {
  width: 150px;
  height: 150px;
  border-radius: 50%;
  background: rgba(255, 255, 255, 0.2);
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  margin: 0 auto 20px;
  box-shadow: 0 8px 25px rgba(0, 0, 0, 0.2);
}

.score-number {
  font-size: 3em;
  font-weight: bold;
}

.score-text {
  font-size: 1.2em;
  opacity: 0.9;
}

.score-message h3 {
  margin: 0;
  font-size: 1.8em;
}

/* 答案詳解 */
.results-detail {
  background: #f8f9fa;
  padding: 30px;
  border-radius: 10px;
  margin-bottom: 20px;
}

.results-detail h3 {
  color: #333;
  margin-top: 0;
  margin-bottom: 20px;
  border-bottom: 2px solid #ff8fa3;
  padding-bottom: 10px;
}

.result-item {
  background: white;
  padding: 20px;
  margin-bottom: 15px;
  border-radius: 8px;
  border-left: 4px solid #ddd;
}

.result-item.correct {
  border-left-color: #51cf66;
  background: linear-gradient(90deg, rgba(81, 207, 102, 0.05) 0%, transparent 100%);
}

.result-item.incorrect {
  border-left-color: #ff6b6b;
  background: linear-gradient(90deg, rgba(255, 107, 107, 0.05) 0%, transparent 100%);
}

.result-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 10px;
}

.result-number {
  font-weight: bold;
  color: #ff8fa3;
}

.badge {
  padding: 4px 12px;
  border-radius: 20px;
  font-size: 0.9em;
  font-weight: bold;
}

.badge.correct {
  background: #51cf66;
  color: white;
}

.badge.incorrect {
  background: #ff6b6b;
  color: white;
}

.result-question {
  font-weight: bold;
  color: #333;
  margin: 10px 0;
}

.result-answers {
  display: flex;
  flex-direction: column;
  gap: 8px;
}

.result-option {
  display: flex;
  gap: 10px;
  padding: 8px 12px;
  background: #f8f9fa;
  border-radius: 5px;
  border-left: 3px solid #ddd;
}

.result-option.correct-answer {
  background: rgba(81, 207, 102, 0.1);
  border-left-color: #51cf66;
  font-weight: 500;
}

.result-option.incorrect-answer {
  background: rgba(255, 107, 107, 0.1);
  border-left-color: #ff6b6b;
  text-decoration: line-through;
  opacity: 0.7;
}

.option-letter {
  font-weight: bold;
  color: #667eea;
  min-width: 25px;
}

/* 考試頁面 */
.exam-layout {
  display: grid;
  grid-template-columns: 250px 1fr;
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

/* 題目導覽 */
.nav-grid {
  display: grid;
  grid-template-columns: repeat(4, 1fr);
  gap: 8px;
  margin-bottom: 20px;
}

.nav-btn {
  position: relative;
  width: 100%;
  padding: 12px;
  border: 2px solid #ddd;
  background: white;
  border-radius: 5px;
  cursor: pointer;
  font-weight: bold;
  transition: all 0.3s ease;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  gap: 4px;
  font-size: 0.9em;
}

.nav-btn:hover {
  border-color: #667eea;
}

.nav-btn.active {
  background: #ff8fa3;
  color: #ffffff;
  border-color: #ff8fa3;
  box-shadow: 0 4px 12px rgba(255, 143, 163, 0.3);
}

.nav-btn.answered {
  border-color: #51cf66;
}

.nav-check {
  color: #51cf66;
  font-size: 0.8em;
}

.progress-section {
  background: white;
  padding: 15px;
  border-radius: 5px;
  border: 1px solid #ddd;
}

.progress-text {
  text-align: center;
  margin-bottom: 10px;
  color: #666;
  font-size: 0.9em;
}

.progress-bar {
  width: 100%;
  height: 8px;
  background: #ddd;
  border-radius: 10px;
  overflow: hidden;
}

.progress-fill {
  height: 100%;
  background: linear-gradient(90deg, #ff8fa3 0%, #ffccd5 100%);
  transition: width 0.5s ease;
}

/* 題目區 */
.question-section {
  display: flex;
  flex-direction: column;
  gap: 20px;
}

.question-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding-bottom: 15px;
  border-bottom: 2px solid #ddd;
}

.question-meta {
  display: flex;
  gap: 20px;
  align-items: center;
}

.question-number {
  color: #ff8fa3;
  font-weight: bold;
}

.question-type {
  padding: 4px 12px;
  border-radius: 20px;
  font-size: 0.85em;
  font-weight: bold;
  color: white;
}

.type-single {
  background: #ff8fa3;
}

.type-multiple {
  background: #ff8787;
}

.type-truefalse {
  background: #4ecdc4;
}

.question-text {
  color: #333;
  font-size: 1.3em;
  margin: 0 0 20px 0;
  line-height: 1.6;
}

/* 選項 */
.options {
  display: flex;
  flex-direction: column;
  gap: 12px;
  margin-bottom: 20px;
}

.option-btn {
  display: flex;
  align-items: center;
  gap: 15px;
  padding: 15px 20px;
  background: white;
  border: 2px solid #ddd;
  border-radius: 8px;
  cursor: pointer;
  transition: all 0.3s ease;
  text-align: left;
  font-size: 1em;
}

.option-btn:hover {
  border-color: #ff8fa3;
  background: #f0f0ff;
}

.option-btn.selected {
  background: linear-gradient(90deg, rgba(255, 143, 163, 0.1) 0%, transparent 100%);
  border-color: #ff8fa3;
  box-shadow: 0 4px 12px rgba(255, 143, 163, 0.2);
}

.option-letter {
  flex-shrink: 0;
  width: 35px;
  height: 35px;
  display: flex;
  align-items: center;
  justify-content: center;
  background: #ff8fa3;
  color: white;
  border-radius: 50%;
  font-weight: bold;
}

.option-text {
  flex: 1;
  color: #333;
}

.option-check {
  flex-shrink: 0;
  color: #51cf66;
  font-size: 1.3em;
}

/* 導航按鈕 */
.navigation {
  display: flex;
  gap: 10px;
  justify-content: space-between;
}

.btn {
  padding: 12px 24px;
  border: none;
  border-radius: 5px;
  cursor: pointer;
  font-weight: bold;
  transition: all 0.3s ease;
}

.btn-secondary {
  background: #ff8fa3;
  color: #ffffff;
  flex: 1;
}

.btn-secondary:hover:not(:disabled) {
  background: #45b8b0;
  transform: translateY(-2px);
}

.btn-secondary:disabled {
  opacity: 0.5;
  cursor: not-allowed;
}

.btn-submit {
  background: linear-gradient(135deg, #51cf66 0%, #40c057 100%);
  color: white;
  flex: 1;
  padding: 15px 24px;
  font-size: 1.1em;
}

.btn-submit:hover {
  transform: translateY(-2px);
  box-shadow: 0 5px 15px rgba(165, 56, 96, 0.4);
}

.result-actions {
  text-align: center;
}

.btn-primary {
  background: #a53860;
  color: white;
  padding: 15px 40px;
  font-size: 1.1em;
}

.btn-primary:hover {
  transform: translateY(-2px);
  box-shadow: 0 5px 15px rgba(102, 126, 234, 0.4);
}

/* 響應式設計 */
@media (max-width: 768px) {
  .exam-layout {
    grid-template-columns: 1fr;
  }

  .nav-grid {
    grid-template-columns: repeat(5, 1fr);
  }

  .question-text {
    font-size: 1.1em;
  }

  .option-btn {
    flex-direction: column;
    align-items: flex-start;
  }
}
</style>
