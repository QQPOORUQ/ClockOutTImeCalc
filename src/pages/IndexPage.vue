<template>
  <q-page padding 
  style="background-image: url('src/assets/melody.jpg'); background-size: cover; background-position: center;"
  class="flex flex-center">
    <q-card class="q-pa-md" style="min-width: 320px; max-width: 400px; width: 100%;">
      <div class="text-h6 q-mb-md">今天上班時間</div>
      <q-input
        outlined
        dense
        type="time"
        v-model="startTime"
        label="請輸入打卡時間"
        @change="calculateEndTime"
      />
      <div v-if="endTime" class="q-mt-md">
        <div>預計下班時間：<strong>{{ endTime }}</strong></div>
        <div class="q-mt-sm">離下班還有：<strong>{{ remainingTime }}</strong></div>
      </div>
    </q-card>
  </q-page>
</template>

<script setup lang="ts">
import { ref, onMounted, onUnmounted } from 'vue'

const startTime = ref<string>('09:00')
const endTime = ref<string | null>(null)
const remainingTime = ref<string>('')

let interval: ReturnType<typeof setInterval> | null = null

// 幫助格式化時間（HH:mm）
function formatTime(date: Date): string {
  const h = date.getHours().toString().padStart(2, '0')
  const m = date.getMinutes().toString().padStart(2, '0')
  return `${h}:${m}`
}

// 計算下班時間（上班時間 + 9.5 小時）
function calculateEndTime(): void {
  if (!startTime.value) {
    endTime.value = null
    remainingTime.value = ''
    clearTimer()
    return
  }
  // 解析 HH:mm 字串為 Date
  const [hourStr, minuteStr] = startTime.value.split(':')
  const startDate = new Date()
  startDate.setHours(Number(hourStr), Number(minuteStr), 0, 0)

  // 加 9 小時 30 分鐘 = 570 分鐘
  const endDate = new Date(startDate.getTime() + 570 * 60 * 1000)
  endTime.value = formatTime(endDate)

  startCountdown(endDate)
}

// 開始倒數計時，參數為結束時間 Date
function startCountdown(endDate: Date): void {
  clearTimer()

  function updateRemaining(): void {
    const now = new Date()
    const diffMs = endDate.getTime() - now.getTime()

    if (diffMs <= 0) {
      remainingTime.value = '已下班囉 🎉'
      clearTimer()
      return
    }

    const diffSec = Math.floor(diffMs / 1000)
    const h = Math.floor(diffSec / 3600)
    const m = Math.floor((diffSec % 3600) / 60)
    const s = diffSec % 60
    remainingTime.value = `${h} 小時 ${m} 分 ${s} 秒`
  }

  updateRemaining()
  interval = setInterval(updateRemaining, 1000)
}

// 清除計時器
function clearTimer(): void {
  if (interval !== null) {
    clearInterval(interval)
    interval = null
  }
}

// 頁面卸載時清除計時器
onUnmounted(() => {
  clearTimer()
})

// 頁面掛載時先計算一次
onMounted(() => {
  document.title = '下班吃章魚燒'
  calculateEndTime()

})
</script>

<style scoped>
</style>
