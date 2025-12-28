<template>
  <div class="main-screen">
    <div class="header">
      <h1 class="flicker">🎄 Новогоднее приключение для Киры! 🎄</h1>
      <p class="subtitle">Пройди 4 свидания и получи сюрприз!</p>
    </div>

    <div class="dates-grid">
      <button 
        v-for="dateNum in 5" 
        :key="dateNum"
        class="date-button"
        :class="{ unlocked: unlockedDates[dateNum], locked: !unlockedDates[dateNum] }"
        @click="startDate(dateNum)"
        :disabled="!unlockedDates[dateNum]"
      >
        <div class="button-content">
          <div class="level-number">УРОВЕНЬ {{ dateNum }}</div>
          <div class="level-title">Свидание {{ dateNum }}</div>
          <div v-if="!unlockedDates[dateNum]" class="lock-icon">🔒</div>
          <div v-else class="unlock-icon">🔓</div>
        </div>
      </button>
    </div>

    <div class="progress-bar">
      <div class="progress" :style="{ width: progressPercent + '%' }"></div>
    </div>
    <p class="progress-text">Прогресс: {{ unlockedCount }}/5</p>
  </div>
</template>

<script>
import { computed } from 'vue'

export default {
  props: {
    unlockedDates: {
      type: Object,
      required: true
    }
  },
  emits: ['startDate'],
  setup(props, { emit }) {
    const startDate = (dateNum) => {
      if (props.unlockedDates[dateNum]) {
        emit('startDate', dateNum)
      }
    }

    const unlockedCount = computed(() => {
      return Object.values(props.unlockedDates).filter(v => v).length
    })

    const progressPercent = computed(() => {
      return (unlockedCount.value / 5) * 100
    })

    return {
      startDate,
      unlockedCount,
      progressPercent
    }
  }
}
</script>

<style scoped>
.main-screen {
  max-width: 1000px;
  margin: 0 auto;
  padding: 40px 20px;
  text-align: center;
  animation: fadeIn 0.5s ease-in;
}

@keyframes fadeIn {
  from {
    opacity: 0;
    transform: scale(0.9);
  }
  to {
    opacity: 1;
    transform: scale(1);
  }
}

.header {
  margin-bottom: 60px;
}

.header h1 {
  font-size: 24px;
  color: #00ff00;
  margin-bottom: 20px;
  text-shadow: 0 0 10px #00ff00, 0 0 20px #ff00ff;
  letter-spacing: 3px;
}

.subtitle {
  font-size: 14px;
  color: #ffff00;
  text-shadow: 0 0 5px #ffff00;
  margin-bottom: 40px;
}

.dates-grid {
  display: flex;
  flex-wrap: wrap;
  gap: 30px;
  margin-bottom: 50px;
  justify-content: center;  /* Центрирует все элементы */
  align-items: flex-start;
}


.date-button {
  padding: 30px 20px;
  border: 4px solid #00ff00;
  background: rgba(0, 50, 100, 0.8);
  color: #00ff00;
  font-family: 'Press Start 2P', cursive;
  font-size: 12px;
  cursor: pointer;
  transition: all 0.3s ease;
  box-shadow: 0 0 10px rgba(0, 255, 0, 0.5), inset 0 0 10px rgba(0, 255, 0, 0.2);
  position: relative;
  overflow: hidden;
}

.date-button:not(.locked):hover {
  transform: scale(1.05);
  box-shadow: 0 0 20px rgba(0, 255, 0, 1), inset 0 0 20px rgba(0, 255, 0, 0.4);
  text-shadow: 0 0 10px #00ff00;
}

.date-button.locked {
  opacity: 0.5;
  border-color: #999999;
  color: #999999;
  cursor: not-allowed;
  background: rgba(30, 30, 30, 0.8);
  box-shadow: 0 0 10px rgba(150, 150, 150, 0.3);
}

.button-content {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 10px;
}

.level-number {
  font-size: 10px;
  opacity: 0.8;
}

.level-title {
  font-size: 14px;
  font-weight: bold;
}

.lock-icon,
.unlock-icon {
  font-size: 30px;
  margin-top: 10px;
}

.progress-bar {
  width: 100%;
  height: 20px;
  background: rgba(0, 0, 0, 0.5);
  border: 3px solid #00ff00;
  margin-bottom: 20px;
  overflow: hidden;
  box-shadow: 0 0 10px rgba(0, 255, 0, 0.5);
}

.progress {
  height: 100%;
  background: linear-gradient(90deg, #00ff00, #00ffff);
  transition: width 0.3s ease;
  box-shadow: 0 0 10px rgba(0, 255, 0, 1);
}

.progress-text {
  font-size: 12px;
  color: #00ff00;
  text-shadow: 0 0 5px #00ff00;
}

@media (max-width: 768px) {
  .header h1 {
    font-size: 16px;
  }

  .dates-grid {
    grid-template-columns: 1fr;
  }

  .date-button {
    padding: 20px 15px;
    font-size: 10px;
  }
}
</style>
