<script setup lang="ts">
import { onMounted, onUnmounted, ref } from 'vue'

interface MetricsHistory {
  id: string
  time: string
  total: number
  valid: number
  invalid: number
  errors: number
}

const history = ref<MetricsHistory[]>([])
let counter = 0
let interval: ReturnType<typeof setInterval> | null = null

const fetchHistory = () => {
  const newMetric: MetricsHistory = {
    id: `metric-${Date.now()}-${counter++}`,
    time: new Date().toLocaleTimeString(),
    total: Math.floor(Math.random() * 200 + 100),
    valid: Math.floor(Math.random() * 150 + 80),
    invalid: Math.floor(Math.random() * 40 + 10),
    errors: Math.floor(Math.random() * 10)
  }

  history.value = [...history.value, newMetric].slice(-20)
}

onMounted(() => {
  // Inicializar con algunos datos
  const initialData: MetricsHistory[] = []
  for (let i = 0; i < 10; i++) {
    initialData.push({
      id: `metric-init-${i}`,
      time: new Date(Date.now() - (10 - i) * 60000).toLocaleTimeString(),
      total: Math.floor(Math.random() * 200 + 100),
      valid: Math.floor(Math.random() * 150 + 80),
      invalid: Math.floor(Math.random() * 40 + 10),
      errors: Math.floor(Math.random() * 10)
    })
  }
  history.value = initialData

  interval = setInterval(fetchHistory, 60000) // Cada 1 minuto
})

onUnmounted(() => {
  if (interval) clearInterval(interval)
})
</script>

<template>
  <div class="bg-gradient-to-br from-gray-900/80 to-gray-800/40 border border-gray-700/50 rounded-xl p-6 backdrop-blur-sm">
    <div class="mb-4">
      <h3 class="text-lg font-bold text-white">Order Validation Metrics (Real-Time)</h3>
    </div>

    <div v-if="history.length > 0" class="space-y-4">
      <!-- Simple bar visualization -->
      <div class="grid grid-cols-1 gap-4">
        <div v-for="metric in history.slice(-5)" :key="metric.id" class="space-y-2">
          <div class="flex justify-between text-xs text-gray-400">
            <span>{{ metric.time }}</span>
            <span class="text-gray-500">Total: {{ metric.total }}</span>
          </div>
          <div class="flex gap-1 h-6">
            <div
              class="bg-brand-60 rounded"
              :style="{ width: `${(metric.valid / metric.total) * 100}%` }"
              title="Valid Orders"
            />
            <div
              class="bg-red-500/60 rounded"
              :style="{ width: `${(metric.invalid / metric.total) * 100}%` }"
              title="Invalid Orders"
            />
            <div
              class="bg-yellow-500/60 rounded"
              :style="{ width: `${(metric.errors / metric.total) * 100}%` }"
              title="Deserialization Errors"
            />
          </div>
        </div>
      </div>

      <!-- Legend -->
      <div class="flex gap-4 text-xs pt-4 border-t border-gray-700/50">
        <div class="flex items-center gap-2">
          <div class="w-3 h-3 bg-brand-60 rounded" />
          <span class="text-gray-400">Valid Orders</span>
        </div>
        <div class="flex items-center gap-2">
          <div class="w-3 h-3 bg-red-500/60 rounded" />
          <span class="text-gray-400">Invalid Orders</span>
        </div>
        <div class="flex items-center gap-2">
          <div class="w-3 h-3 bg-yellow-500/60 rounded" />
          <span class="text-gray-400">Deserialization Errors</span>
        </div>
      </div>
    </div>
  </div>
</template>
