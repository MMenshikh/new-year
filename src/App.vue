<template>
  <div class="game-container">
    <MainScreen 
      v-if="currentScreen === 'main'" 
      :unlockedDates="unlockedDates" 
      @startDate="goToDate" 
    />
    
    <DateScreen 
      v-if="currentScreen === 'date'" 
      :dateNumber="currentDateNumber"
      :dateData="dates[currentDateNumber]"
      @codeMissingCode="goToMain"
      @codeSubmitted="handleCodeSubmitted"
    />
  </div>
</template>

<script>
import { ref } from 'vue'
import MainScreen from './components/MainScreen.vue'
import DateScreen from './components/DateScreen.vue'

export default {
  components: {
    MainScreen,
    DateScreen
  },
  setup() {
    const currentScreen = ref('main')
    const currentDateNumber = ref(null)
    
    const dates = ref({
      1: {
        title: 'Свидание 1',
        riddle: 'Белая пушистая, в воздухе летает, а земли коснется - тут же и растает',
        answer: 'снежинка',
        address: 'Котокафе котовский\nМонастырская ул., 14',
        correctCode: 'https://',
      },
      2: {
        title: 'Свидание 2',
        riddle: 'Среди самых разных фруктов, Новогодний есть один - кислый, сладкий очень спелый, это вкусный - ',
        answer: 'мандарин', 
        address: 'Гончарная мастерская\nСоветская ул., 94',
        correctCode: 'shorturl.at/',
      },
      3: {
        title: 'Свидание 3',
        riddle: 'Красный нос и борода, ходит он туда-сюда, всем подарки он принёс, кто же это?',
        answer: 'дед мороз',
        address: 'Трамвайное кафе\nост. Разгуляй (на кольце)',
        correctCode: 'kNyfG',
      },
      4: {
        title: 'Свидание 4 - ФИНАЛ!',
        riddle: 'Днём и тёмными ночами, она мёрзнет под снегами, но погреться в Новый год, её каждый в дом зовёт',
        answer: 'ёлка',
        address: 'ул. Фридриха Энгельса, 18',
        correctCode: '',
      }
    })


    const unlockedDates = ref({
      1: true,
      2: false,
      3: false,
      4: false
    })

    const loadProgress = () => {
      const saved = localStorage.getItem('kira-adventure-progress')
      if (saved) {
        unlockedDates.value = JSON.parse(saved)
      }
    }

    const saveProgress = () => {
      localStorage.setItem('kira-adventure-progress', JSON.stringify(unlockedDates.value))
    }

    const goToDate = (dateNumber) => {
      if (unlockedDates.value[dateNumber]) {
        currentDateNumber.value = dateNumber
        currentScreen.value = 'date'
      }
    }

    const goToMain = () => {
      currentScreen.value = 'main'
    }

    const handleCodeSubmitted = (code) => {
      const correctCode = dates.value[currentDateNumber.value].correctCode
      
      if (currentDateNumber.value < 4) {
        if (code === correctCode) {
          // Код верный - разблокируем следующее свидание
          unlockedDates.value[currentDateNumber.value + 1] = true
          saveProgress()
          alert('✅ Код верный! Следующее свидание разблокировано!')
          goToMain()
        } else {
          alert('❌ Неверный код! Проверь, что ты правильно переписал.')
        }
      } else {
        // Финальное свидание
        alert('🎉 Ты дошла до финала!')
        goToMain()
      }
    }

    loadProgress()

    return {
      currentScreen,
      currentDateNumber,
      dates,
      unlockedDates,
      goToDate,
      goToMain,
      handleCodeSubmitted
    }
  }
}
</script>

<style scoped>
.game-container {
  width: 100%;
  min-height: 100vh;
  padding: 20px;
}
</style>
