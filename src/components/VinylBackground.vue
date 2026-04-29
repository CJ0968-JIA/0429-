<template>
  <div class="bg-container">
    <div class="record-player" :class="{ 'is-spinning': isSpinning }">
      <div class="record">
        <div class="record-label"></div>
      </div>
      <!-- 唱針效果 -->
      <div class="tone-arm"></div>

      <!-- 漂浮音符 -->
      <div class="music-notes">
        <span class="note">♪</span>
        <span class="note">♫</span>
        <span class="note">♩</span>
        <span class="note">♬</span>
        <span class="note">♪</span>
        <span class="note">♫</span>
        <span class="note">♩</span>
        <span class="note">♬</span>
        <span class="note">♪</span>
        <span class="note">♬</span>
        <span class="note">♪</span>
        <span class="note">♫</span>
        <span class="note">♩</span>
        <span class="note">♬</span>
        <span class="note">♪</span>
        <span class="note">♫</span>
        <span class="note">♩</span>
        <span class="note">♬</span>
        <span class="note">♪</span>
        <span class="note">♫</span>
        <span class="note">♩</span>
        <span class="note">♬</span>
        <span class="note">♪</span>
        <span class="note">♫</span>
        <span class="note">♩</span>
        <span class="note">♬</span>
        <span class="note">♪</span>
        <span class="note">♫</span>
        <span class="note">♩</span>
        <span class="note">♬</span>
        <span class="note">♪</span>
        <span class="note">♫</span>
        <span class="note">♩</span>
        <span class="note">♬</span>
        <span class="note">♪</span>
        <span class="note">♫</span>
        <span class="note">♩</span>
        <span class="note">♬</span>
        <span class="note">♪</span>
        <span class="note">♬</span>
      </div>
    </div>
  </div>
</template>

<script setup>
defineProps({
  isSpinning: Boolean
})
</script>

<style scoped>
.bg-container {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  z-index: -1;
  background-color: #ffccd5; /* 主色調 */
  display: flex;
  justify-content: center;
  align-items: center;
  overflow: hidden;
}

.record-player {
  position: relative;
  width: 500px;
  height: 500px;
  opacity: 0.6;
}

.record {
  width: 100%;
  height: 100%;
  background: repeating-radial-gradient(
    circle,
    #333 0px,
    #333 2px,
    #111 3px,
    #111 4px
  );
  border-radius: 50%;
  border: 15px solid #ffb3c1; /* 次色調 */
  box-shadow: 0 0 50px rgba(0,0,0,0.3);
  position: absolute;
  animation: spin 5s linear infinite; /* 調整為更適合抒情節奏的 5秒 */
  animation-play-state: paused;
  transition: transform 0.5s ease;
}

/* 增加唱片上的反光感，讓旋轉更明顯 */
.record::after {
  content: '';
  position: absolute;
  top: 0; left: 0; width: 100%; height: 100%;
  background: linear-gradient(
    45deg,
    transparent 40%,
    rgba(255, 255, 255, 0.1) 50%,
    transparent 60%
  );
  border-radius: 50%;
}

.is-spinning .record {
  animation-play-state: running;
}

.record-label {
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  width: 140px;
  height: 140px;
  background-color: #ffb3c1;
  border: 4px solid #ffccd5;
  border-radius: 50%;
}

.tone-arm {
  position: absolute;
  top: 50px;
  right: -20px;
  width: 20px;
  height: 250px;
  background: #666;
  transform-origin: top center;
  transform: rotate(0deg);
  border-radius: 10px;
  transition: transform 0.8s cubic-bezier(0.4, 0, 0.2, 1);
  z-index: 10;
}

.is-spinning .tone-arm {
  transform: rotate(25deg);
}

/* 音符樣式與動畫 */
.music-notes {
  position: absolute;
  width: 100%;
  height: 100%;
  top: 0;
  left: 0;
  pointer-events: none;
}
.note {
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  color: #555;
  font-size: 3.5rem;
  opacity: 0;
  filter: drop-shadow(0 0 10px rgba(255, 255, 255, 0.8));
  animation: radiate-note 12s infinite linear; /* 讓音符飄散得更緩慢、優雅 */
}
/* 放射狀分佈與時間錯開 */
.note:nth-child(1) { animation-delay: 0s; --random-deg: 9deg; }
.note:nth-child(2) { animation-delay: 0.2s; --random-deg: 18deg; }
.note:nth-child(3) { animation-delay: 0.4s; --random-deg: 27deg; }
.note:nth-child(4) { animation-delay: 0.6s; --random-deg: 36deg; }
.note:nth-child(5) { animation-delay: 0.8s; --random-deg: 45deg; }
.note:nth-child(6) { animation-delay: 1.0s; --random-deg: 54deg; }
.note:nth-child(7) { animation-delay: 1.2s; --random-deg: 63deg; }
.note:nth-child(8) { animation-delay: 1.4s; --random-deg: 72deg; }
.note:nth-child(9) { animation-delay: 1.6s; --random-deg: 81deg; }
.note:nth-child(10) { animation-delay: 1.8s; --random-deg: 90deg; }
.note:nth-child(11) { animation-delay: 2.0s; --random-deg: 99deg; }
.note:nth-child(12) { animation-delay: 2.2s; --random-deg: 108deg; }
.note:nth-child(13) { animation-delay: 2.4s; --random-deg: 117deg; }
.note:nth-child(14) { animation-delay: 2.6s; --random-deg: 126deg; }
.note:nth-child(15) { animation-delay: 2.8s; --random-deg: 135deg; }
.note:nth-child(16) { animation-delay: 3.0s; --random-deg: 144deg; }
.note:nth-child(17) { animation-delay: 3.2s; --random-deg: 153deg; }
.note:nth-child(18) { animation-delay: 3.4s; --random-deg: 162deg; }
.note:nth-child(19) { animation-delay: 3.6s; --random-deg: 171deg; }
.note:nth-child(20) { animation-delay: 3.8s; --random-deg: 180deg; }
.note:nth-child(21) { animation-delay: 4.0s; --random-deg: 189deg; }
.note:nth-child(22) { animation-delay: 4.2s; --random-deg: 198deg; }
.note:nth-child(23) { animation-delay: 4.4s; --random-deg: 207deg; }
.note:nth-child(24) { animation-delay: 4.6s; --random-deg: 216deg; }
.note:nth-child(25) { animation-delay: 4.8s; --random-deg: 225deg; }
.note:nth-child(26) { animation-delay: 5.0s; --random-deg: 234deg; }
.note:nth-child(27) { animation-delay: 5.2s; --random-deg: 243deg; }
.note:nth-child(28) { animation-delay: 5.4s; --random-deg: 252deg; }
.note:nth-child(29) { animation-delay: 5.6s; --random-deg: 261deg; }
.note:nth-child(30) { animation-delay: 5.8s; --random-deg: 270deg; }
.note:nth-child(31) { animation-delay: 6.0s; --random-deg: 279deg; }
.note:nth-child(32) { animation-delay: 6.2s; --random-deg: 288deg; }
.note:nth-child(33) { animation-delay: 6.4s; --random-deg: 297deg; }
.note:nth-child(34) { animation-delay: 6.6s; --random-deg: 306deg; }
.note:nth-child(35) { animation-delay: 6.8s; --random-deg: 315deg; }
.note:nth-child(36) { animation-delay: 7.0s; --random-deg: 324deg; }
.note:nth-child(37) { animation-delay: 7.2s; --random-deg: 333deg; }
.note:nth-child(38) { animation-delay: 7.4s; --random-deg: 342deg; }
.note:nth-child(39) { animation-delay: 7.6s; --random-deg: 351deg; }
.note:nth-child(40) { animation-delay: 7.8s; --random-deg: 360deg; }

@keyframes radiate-note {
  0% {
    transform: translate(-50%, -50%) rotate(0deg) scale(0);
    opacity: 0;
  }
  10% {
    opacity: 0.8;
  }
  100% {
    transform: translate(
      calc(-50% + cos(var(--random-deg)) * 800px),
      calc(-50% + sin(var(--random-deg)) * 800px)
    ) rotate(720deg) scale(1.5);
    opacity: 0;
  }
}

@keyframes spin {
  from { transform: rotate(0deg); }
  to { transform: rotate(360deg); }
}
</style>