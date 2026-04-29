<template>
  <div class="random-picker-container">
    <h1>🎯 隨機抽籤機</h1>
    <p>請輸入名單 (每行一個名字)：</p>
    <textarea id="names-input" placeholder="例如：&#10;小明&#10;小華&#10;第一組&#10;第二組" v-model="namesInput"></textarea>
    
    <div class="controls">
      <button id="draw-btn" @click="startDraw" :disabled="isDrawing">開始抽籤！</button>
    </div>

    <div id="result-display" :class="{ 'shaking': isDrawing }">{{ resultDisplay }}</div>
  </div>
</template>

<script setup>
import { ref } from 'vue';
import confetti from 'canvas-confetti'; // 確保你已安裝此套件: npm install canvas-confetti

const namesInput = ref('');
const resultDisplay = ref('準備好了嗎？');
const isDrawing = ref(false);

let intervalTimer = null;

function startDraw() {
  const names = namesInput.value.split('\n').map(name => name.trim()).filter(name => name !== '');

  if (names.length === 0) {
    alert('請先輸入名單！');
    return;
  }

  isDrawing.value = true;
  resultDisplay.value = '抽籤中...'; // 動畫開始時的初始文字

  let duration = 2000; // 動畫持續 2 秒
  let startTime = Date.now();

  intervalTimer = setInterval(() => {
    const randomIndex = Math.floor(Math.random() * names.length);
    resultDisplay.value = names[randomIndex];

    if (Date.now() - startTime >= duration) {
      clearInterval(intervalTimer);
      finishDraw(names);
    }
  }, 80); // 每 80 毫秒更新一次，產生「晃動」效果
}

function finishDraw(names) {
  isDrawing.value = false;
  
  const finalWinner = names[Math.floor(Math.random() * names.length)];
  resultDisplay.value = finalWinner;

  confetti({
    particleCount: 150,
    spread: 70,
    origin: { y: 0.6 }
  });
}
</script>

<style scoped>
/* 抽籤機元件的局部樣式 */
.random-picker-container {
    background: white;
    padding: 2rem;
    border-radius: 15px;
    box-shadow: 0 10px 25px rgba(0,0,0,0.1);
    width: 90%;
    max-width: 500px;
    text-align: center;
    margin: 2rem auto;
    position: relative;
    z-index: 10;
}

h1 { color: #2f3542; margin-bottom: 1.5rem; }

textarea {
    width: 100%;
    height: 150px;
    padding: 10px;
    border: 2px solid #ddd;
    border-radius: 8px;
    resize: none;
    font-size: 1rem;
    box-sizing: border-box;
    margin-bottom: 1rem;
}

textarea:focus { outline: none; border-color: #4a90e2; }

button { background-color: #4a90e2; color: white; border: none; padding: 12px 30px; font-size: 1.1rem; border-radius: 50px; cursor: pointer; transition: transform 0.2s, background 0.2s; box-shadow: 0 4px 15px rgba(74, 144, 226, 0.3); }
button:hover { background-color: #357abd; transform: translateY(-2px); }
button:active { transform: translateY(0); }
button:disabled { background-color: #ccc; cursor: not-allowed; }

#result-display { font-size: 2.5rem; font-weight: bold; color: #ff4757; min-height: 4rem; display: flex; justify-content: center; align-items: center; margin-top: 1rem; word-break: break-all; }

/* 抽取時的晃動動畫 */
.shaking { animation: shake 0.1s infinite; }

@keyframes shake {
    0% { transform: translate(1px, 1px) rotate(0deg); }
    20% { transform: translate(-1px, -2px) rotate(-1deg); }
    40% { transform: translate(-3px, 0px) rotate(1deg); }
    60% { transform: translate(3px, 2px) rotate(0deg); }
    80% { transform: translate(1px, -1px) rotate(1deg); }
    100% { transform: translate(-1px, 2px) rotate(-1deg); }
}
</style>