<script setup>
import { ref, computed } from 'vue'

const today = new Date()
const currentYear = ref(today.getFullYear())
const currentMonth = ref(today.getMonth())

// 資料結構: { 'YYYY-MM-DD': { text: '', emoji: '' } }
const calendarData = ref({})

const selectedDate = ref(null)
const editingText = ref('')
const editingEmoji = ref('')

const months = ['一月', '二月', '三月', '四月', '五月', '六月', '七月', '八月', '九月', '十月', '十一月', '十二月']
const emojis = [
  { icon: '😢', label: '哭臉' },
  { icon: '🙂', label: '笑臉' },
  { icon: '😐', label: '無感' }
]

const calendarDays = computed(() => {
  const firstDay = new Date(currentYear.value, currentMonth.value, 1).getDay()
  const daysInMonth = new Date(currentYear.value, currentMonth.value + 1, 0).getDate()
  
  const days = []
  for (let i = 0; i < firstDay; i++) {
    days.push(null)
  }
  for (let d = 1; d <= daysInMonth; d++) {
    const dateStr = `${currentYear.value}-${String(currentMonth.value + 1).padStart(2, '0')}-${String(d).padStart(2, '0')}`
    days.push({ day: d, dateStr })
  }
  return days
})

const openEditor = (day) => {
  if (!day) return
  selectedDate.value = day.dateStr
  const existing = calendarData.value[day.dateStr] || { text: '', emoji: '' }
  editingText.value = existing.text
  editingEmoji.value = existing.emoji
}

const saveData = () => {
  calendarData.value[selectedDate.value] = {
    text: editingText.value,
    emoji: editingEmoji.value
  }
  selectedDate.value = null
}

const changeMonth = (delta) => {
  const date = new Date(currentYear.value, currentMonth.value + delta, 1)
  currentYear.value = date.getFullYear()
  currentMonth.value = date.getMonth()
}
</script>

<template>
  <div class="calendar-app">
    <div class="header">
      <button @click="changeMonth(-1)">上個月</button>
      <h2>{{ currentYear }}年 {{ months[currentMonth] }}</h2>
      <button @click="changeMonth(1)">下個月</button>
    </div>

    <div class="calendar-grid">
      <div v-for="w in ['日', '一', '二', '三', '四', '五', '六']" :key="w" class="weekday">{{ w }}</div>
      <div 
        v-for="(day, index) in calendarDays" 
        :key="index" 
        class="day-cell"
        :class="{ 'has-day': day }"
        @click="openEditor(day)"
      >
        <template v-if="day">
          <span class="day-number">{{ day.day }}</span>
          <div class="day-content">
            <div class="emoji-display">{{ calendarData[day.dateStr]?.emoji }}</div>
            <div class="text-preview">{{ calendarData[day.dateStr]?.text }}</div>
          </div>
        </template>
      </div>
    </div>

    <!-- 編輯彈窗 -->
    <div v-if="selectedDate" class="modal-overlay" @click.self="selectedDate = null">
      <div class="modal">
        <h3>紀錄日期: {{ selectedDate }}</h3>
        <div class="emoji-selector">
          <button 
            v-for="e in emojis" 
            :key="e.icon" 
            @click="editingEmoji = e.icon"
            :class="{ active: editingEmoji === e.icon }"
          >
            {{ e.icon }}
          </button>
        </div>
        <textarea v-model="editingText" placeholder="在此輸入文字訊息..."></textarea>
        <div class="modal-actions">
          <button @click="selectedDate = null">取消</button>
          <button @click="saveData" class="save-btn">儲存紀錄</button>
        </div>
      </div>
    </div>
  </div>
</template>

<style scoped>
.calendar-app { max-width: 800px; margin: 0 auto; padding: 20px; font-family: sans-serif; }
.header { display: flex; justify-content: space-between; align-items: center; margin-bottom: 20px; }
.calendar-grid { display: grid; grid-template-columns: repeat(7, 1fr); gap: 1px; background-color: #ddd; border: 1px solid #ddd; }
.weekday { background-color: #f8f9fa; padding: 10px; text-align: center; font-weight: bold; }
.day-cell { background-color: white; min-height: 90px; padding: 5px; cursor: pointer; display: flex; flex-direction: column; transition: background 0.2s; }
.day-cell:hover { background-color: #f0f7ff; }
.day-number { font-size: 0.85em; color: #666; }
.day-content { flex-grow: 1; display: flex; flex-direction: column; align-items: center; justify-content: center; }
.emoji-display { font-size: 1.4em; margin-bottom: 2px; }
.text-preview { font-size: 0.75em; color: #444; width: 100%; text-align: center; overflow: hidden; text-overflow: ellipsis; white-space: nowrap; }

.modal-overlay { position: fixed; top: 0; left: 0; width: 100%; height: 100%; background: rgba(0,0,0,0.5); display: flex; justify-content: center; align-items: center; z-index: 100; }
.modal { background: white; padding: 20px; border-radius: 12px; width: 320px; box-shadow: 0 4px 15px rgba(0,0,0,0.2); }
.emoji-selector { display: flex; justify-content: space-around; margin: 15px 0; }
.emoji-selector button { font-size: 1.8em; padding: 8px; border: 2px solid transparent; background: none; cursor: pointer; border-radius: 50%; transition: all 0.2s; }
.emoji-selector button.active { border-color: #4CAF50; background: #f0fdf4; transform: scale(1.1); }
textarea { width: 100%; height: 100px; margin-bottom: 15px; padding: 10px; border: 1px solid #ccc; border-radius: 6px; resize: none; box-sizing: border-box; }
.modal-actions { display: flex; justify-content: flex-end; gap: 10px; }
.save-btn { background-color: #4CAF50; color: white; border: none; padding: 8px 20px; border-radius: 6px; cursor: pointer; font-weight: bold; }
.save-btn:hover { background-color: #45a049; }
</style>
