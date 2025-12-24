<template>
  <div class="date-screen">
    <button class="back-button" @click="goBack">← Назад</button>

    <div class="container">
      <h2 class="title">{{ dateData.title }}</h2>

      <!-- ЭТАП 1: Отгадать загадку (для уровней 1-3) -->
      <div v-if="stage === 'riddle' && dateNumber < 4" class="stage">
        <div class="riddle-box">
          <p class="riddle-label">📜 Загадка:</p>
          <p class="riddle-text">{{ dateData.riddle }}</p>
        </div>

        <div class="hint-box">
          <p class="hint-text">💡 Подсказка: первая буква — "{{ dateData.answer.charAt(0).toUpperCase() }}"</p>
        </div>

        <div class="answer-input-box">
          <label class="answer-label">Введи ответ на загадку:</label>
          <input 
            v-model="inputAnswer"
            type="text"
            class="code-input"
            placeholder="Введи ответ..."
            @keyup.enter="checkAnswer"
          />
          <button class="submit-button" @click="checkAnswer">✓ Проверить</button>
        </div>
      </div>

      <!-- СПЕЦИАЛЬНЫЙ ЭКРАН для 4-го уровня -->
      <div v-if="dateNumber === 4 && stage === 'riddle'" class="stage">
        <div class="final-puzzle-box">
          <p class="final-puzzle-text">
            🔐 Ты прошла первые 3 свидания и получила 3 части ссылки!
          </p>
          <p class="final-puzzle-text">
            Теперь совмести все коды которые ты получила и введи полную ссылку:
          </p>
        </div>

        <div class="hint-box">
          <p class="hint-text">
            💡 Напомню: 
            <br>От 1 свидания: https://
            <br>От 2 свидания: shorturl.at/
            <br>От 3 свидания: kNyfG
          </p>
        </div>

        <div class="answer-input-box">
          <label class="answer-label">Введи полную ссылку:</label>
          <input 
            v-model="inputAnswer"
            type="text"
            class="code-input"
            placeholder="https://shorturl.at/..."
            @keyup.enter="checkAnswer"
          />
          <button class="submit-button" @click="checkAnswer">✓ Проверить</button>
        </div>
      </div>

      <!-- ЭТАП 2: Показать адрес -->
      <div v-if="stage === 'coordinates'" class="stage">
        <div v-if="dateNumber < 4" class="coordinates-box">
          <p class="coordinates-label">📍 Место свидания:</p>
          <p class="coordinates-text">{{ dateData.address }}</p>
        </div>

        <div v-else class="final-destination-box">
          <p class="final-puzzle-text">🎉 Ссылка верна!</p>
          <p class="coordinates-label">📍 Последнее свидание ждёт тебя здесь:</p>
          <p class="coordinates-text">Перейди по ссылке :)</p>
        </div>

        <p class="message">Отправляйтесь на это свидание! 🚗💕</p>
        
        <button class="submit-button" @click="handleCoordinatesButton">
          {{ dateNumber < 4 ? '✓ Свидание пройдено, вводить код' : '✓ Ура, поехали на свидание!' }}
        </button>
      </div>


      <!-- ЭТАП 3: Ввести код для разблокировки следующего -->
      <div v-if="stage === 'code'" class="stage">
        <div v-if="dateNumber < 4" class="code-input-box">
          <label class="code-label">Введи код, который я тебе дал после свидания:</label>
          <input 
            v-model="inputCode"
            type="text"
            class="code-input"
            placeholder="Введи код здесь..."
            @keyup.enter="submitCode"
          />
          <button class="submit-button" @click="submitCode">✓ Подтвердить</button>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import { ref } from 'vue'

export default {
  props: {
    dateNumber: {
      type: Number,
      required: true
    },
    dateData: {
      type: Object,
      required: true
    }
  },
  emits: ['codeMissingCode', 'codeSubmitted'],
  setup(props, { emit }) {
    const stage = ref('riddle') // 'riddle' -> 'coordinates' -> 'code'
    const inputAnswer = ref('')
    const inputCode = ref('')
    
    // Генерируем случайные координаты для свидания
    const coordinates = ref({
      latitude: 55.7558 + (Math.random() - 0.5) * 0.1,
      longitude: 37.6173 + (Math.random() - 0.5) * 0.1,
      address: generateRandomAddress()
    })

    function generateRandomAddress() {
      const addresses = [
        'Кафе "У причала", Москва',
        'Парк Горького, вход 1',
        'Большой театр, площадь',
        'Красная площадь, у памятника',
        'Арбат, середина улицы',
        'ВДНХ, у фонтана',
        'МГУ, главное здание',
        'Цветной бульвар, середина'
      ]
      return addresses[Math.floor(Math.random() * addresses.length)]
    }

    const goBack = () => {
      emit('codeMissingCode')
    }

    const checkAnswer = () => {
      const userAnswer = inputAnswer.value.trim().toLowerCase()
      
      let correctAnswer
      if (props.dateNumber < 4) {
        correctAnswer = props.dateData.answer.toLowerCase()
      } else {
        // Для 4-го уровня правильный ответ - полная ссылка
        correctAnswer = 'https://shorturl.at/kNyfG'.toLowerCase()
      }
      
      if (userAnswer === '') {
        alert('Пожалуйста, введи ответ!')
        return
      }

      if (userAnswer === correctAnswer) {
        stage.value = 'coordinates'
        inputAnswer.value = ''
      } else {
        if (props.dateNumber === 4) {
          alert('❌ Неверная ссылка! Проверь, правильно ли собрала коды.')
        } else {
          alert('❌ Неверно! Попробуй ещё раз.')
        }
        inputAnswer.value = ''
      }
    }

    const handleCoordinatesButton = () => {
      if (props.dateNumber < 4) {
        // Для уровней 1-3: переходим к вводу кода
        stage.value = 'code'
      } else {
        // Для 4-го уровня: сразу отправляем пустой код и закрываем
        emit('codeSubmitted', '')
        goBack()
      }
    }

    const submitCode = () => {
      if (inputCode.value.trim() === '') {
        alert('Пожалуйста, введи код!')
        return
      }
      emit('codeSubmitted', inputCode.value.trim())
      inputCode.value = ''
    }

    return {
      stage,
      inputAnswer,
      inputCode,
      coordinates,
      checkAnswer,
      submitCode,
      handleCoordinatesButton,
      goBack
    }
  }
}
</script>

<style scoped>
.date-screen {
  min-height: 100vh;
  padding: 20px;
  display: flex;
  justify-content: center;
  align-items: center;
  animation: slideIn 0.3s ease;
}

@keyframes slideIn {
  from {
    opacity: 0;
    transform: translateY(30px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
}

.back-button {
  position: absolute;
  top: 20px;
  left: 20px;
  padding: 10px 20px;
  background: rgba(0, 100, 200, 0.8);
  color: #00ff00;
  border: 2px solid #00ff00;
  font-family: 'Press Start 2P', cursive;
  font-size: 10px;
  cursor: pointer;
  transition: all 0.2s ease;
  box-shadow: 0 0 10px rgba(0, 255, 0, 0.3);
}

.back-button:hover {
  background: rgba(0, 100, 200, 1);
  box-shadow: 0 0 15px rgba(0, 255, 0, 0.7);
}

.container {
  max-width: 600px;
  width: 100%;
  padding: 40px;
  background: rgba(0, 30, 60, 0.95);
  border: 4px solid #00ff00;
  box-shadow: 0 0 30px rgba(0, 255, 0, 0.5), inset 0 0 20px rgba(0, 255, 0, 0.1);
  display: flex;
  flex-direction: column;
}

.title {
  text-align: center;
  color: #00ff00;
  font-size: 20px;
  margin-bottom: 30px;
  text-shadow: 0 0 10px #00ff00;
}

.stage {
  animation: fadeIn 0.4s ease;
  display: flex;
  flex-direction: column;
  gap: 15px;
}

@keyframes fadeIn {
  from {
    opacity: 0;
  }
  to {
    opacity: 1;
  }
}

/* ЭТАП 1: Загадка */
.riddle-box {
  background: rgba(50, 50, 100, 0.7);
  padding: 20px;
  border: 2px solid #ffff00;
  box-shadow: 0 0 10px rgba(255, 255, 0, 0.3);
}

/* Специальный экран для 4-го уровня */
.final-puzzle-box {
  background: rgba(100, 50, 100, 0.7);
  padding: 25px;
  border: 3px solid #ff00ff;
  box-shadow: 0 0 20px rgba(255, 0, 255, 0.5);
  animation: pulse-magic 1s infinite;
}

@keyframes pulse-magic {
  0%, 100% {
    transform: scale(1);
  }
  50% {
    transform: scale(1.01);
  }
}

.final-puzzle-text {
  color: #ffff00;
  font-size: 13px;
  line-height: 1.6;
  margin: 0;
  text-shadow: 0 0 10px #ffff00;
  font-family: 'Press Start 2P', cursive;
}

.final-puzzle-text + .final-puzzle-text {
  margin-top: 15px;
}

.riddle-label {
  color: #ffff00;
  font-size: 12px;
  margin-bottom: 10px;
  text-shadow: 0 0 5px #ffff00;
}

.riddle-text {
  color: #ffffff;
  font-size: 14px;
  line-height: 1.6;
  font-family: 'Press Start 2P', cursive;
}

.hint-box {
  background: rgba(100, 50, 50, 0.7);
  padding: 15px;
  border: 2px solid #ff6666;
  box-shadow: 0 0 10px rgba(255, 100, 100, 0.3);
}

.hint-text {
  color: #ff6666;
  font-size: 11px;
  text-shadow: 0 0 5px #ff6666;
  line-height: 1.5;
}

.answer-input-box {
  display: flex;
  flex-direction: column;
  gap: 15px;
}

.answer-label {
  color: #00ff00;
  font-size: 11px;
  text-shadow: 0 0 5px #00ff00;
}

/* ЭТАП 2: Координаты */
.coordinates-box {
  background: rgba(50, 100, 50, 0.7);
  padding: 25px;
  border: 3px solid #00ff00;
  box-shadow: 0 0 20px rgba(0, 255, 0, 0.5);
  text-align: center;
  animation: pulse 1.5s infinite;
}

.final-destination-box {
  background: rgba(100, 100, 50, 0.7);
  padding: 25px;
  border: 3px solid #ffff00;
  box-shadow: 0 0 20px rgba(255, 255, 0, 0.5);
  text-align: center;
  animation: pulse 1.5s infinite;
}

@keyframes pulse {
  0%, 100% {
    transform: scale(1);
  }
  50% {
    transform: scale(1.02);
  }
}

.coordinates-label {
  color: #ffff00;
  font-size: 12px;
  margin-bottom: 15px;
  text-shadow: 0 0 5px #ffff00;
}

.coordinates-text {
  color: #00ff00;
  font-size: 14px;
  font-weight: bold;
  margin-bottom: 10px;
  text-shadow: 0 0 10px #00ff00;
  font-family: 'Press Start 2P', cursive;
  white-space: pre-wrap;
  line-height: 1.8;
}


.coordinates-hint {
  color: #00ffff;
  font-size: 11px;
  text-shadow: 0 0 5px #00ffff;
}

.message {
  color: #ffff00;
  font-size: 12px;
  text-align: center;
  text-shadow: 0 0 5px #ffff00;
}

/* ЭТАП 3: Код */
.code-input-box {
  display: flex;
  flex-direction: column;
  gap: 15px;
}

.code-label {
  color: #00ff00;
  font-size: 11px;
  text-shadow: 0 0 5px #00ff00;
}

.code-input {
  padding: 12px;
  background: rgba(0, 50, 50, 0.8);
  border: 2px solid #00ff00;
  color: #00ff00;
  font-family: 'Press Start 2P', cursive;
  font-size: 12px;
  box-shadow: 0 0 10px rgba(0, 255, 0, 0.3), inset 0 0 5px rgba(0, 255, 0, 0.2);
}

.code-input::placeholder {
  color: rgba(0, 255, 0, 0.5);
}

.code-input:focus {
  outline: none;
  box-shadow: 0 0 20px rgba(0, 255, 0, 0.7), inset 0 0 10px rgba(0, 255, 0, 0.3);
}

.submit-button {
  padding: 15px 30px;
  background: rgba(0, 150, 0, 0.8);
  color: #ffffff;
  border: 2px solid #00ff00;
  font-family: 'Press Start 2P', cursive;
  font-size: 12px;
  cursor: pointer;
  transition: all 0.3s ease;
  box-shadow: 0 0 10px rgba(0, 255, 0, 0.5);
  width: 100%;
  text-align: center;
  align-self: center;
}

.submit-button:hover {
  background: rgba(0, 200, 0, 1);
  transform: scale(1.05);
  box-shadow: 0 0 20px rgba(0, 255, 0, 1);
  text-shadow: 0 0 10px #ffffff;
}

.submit-button:active {
  transform: scale(0.98);
}

@media (max-width: 600px) {
  .container {
    padding: 20px;
    border-width: 3px;
  }

  .title {
    font-size: 16px;
  }

  .riddle-text {
    font-size: 12px;
  }

  .final-puzzle-text {
    font-size: 11px;
  }

  .code-input {
    font-size: 11px;
  }

  .submit-button {
    font-size: 11px;
  }
}
</style>
