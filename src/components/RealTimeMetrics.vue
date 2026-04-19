<script setup lang="ts">
import { onMounted, onUnmounted, ref, computed } from 'vue'
import MetricCard from './MetricCard.vue'

interface ValidationMetrics {
  windowStart: string
  windowEnd: string
  total: number
  valid: number
  invalid: number
  deserializationErrors: number
}

interface HistoryItem {
  id: string
  value: number
}

const metrics = ref<ValidationMetrics>({
  windowStart: new Date().toISOString(),
  windowEnd: new Date().toISOString(),
  total: 0,
  valid: 0,
  invalid: 0,
  deserializationErrors: 0
})

const history = ref<HistoryItem[]>([])
let counter = 0
let interval: ReturnType<typeof setInterval> | null = null

const validRate = computed(() => {
  return metrics.value.total > 0
    ? ((metrics.value.valid / metrics.value.total) * 100).toFixed(1)
    : '0'
})

const fetchMetrics = () => {
  // Simulación de datos del tópico orders-validation-metrics
  const mockData: ValidationMetrics = {
    windowStart: new Date(Date.now() - 60000).toISOString(),
    windowEnd: new Date().toISOString(),
    total: Math.floor(Math.random() * 200 + 100),
    valid: Math.floor(Math.random() * 150 + 80),
    invalid: Math.floor(Math.random() * 40 + 10),
    deserializationErrors: Math.floor(Math.random() * 10)
  }

  metrics.value = mockData
  history.value = [...history.value, { id: `hist-${Date.now()}-${counter++}`, value: mockData.total }].slice(-20)
}

onMounted(() => {
  // Inicializar con datos históricos
  const initialHistory = Array.from({ length: 10 }, (_, i) => ({
    id: `hist-init-${i}`,
    value: Math.floor(Math.random() * 200 + 100)
  }))
  history.value = initialHistory

  fetchMetrics()
  interval = setInterval(fetchMetrics, 60000) // Cada 1 minuto
})

onUnmounted(() => {
  if (interval) clearInterval(interval)
})
</script>

<template>
  <MetricCard
    title="Total Orders (Current Window)"
    :value="metrics.total.toLocaleString()"
    :subtitle="`Window: ${new Date(metrics.windowStart).toLocaleTimeString()} - ${new Date(metrics.windowEnd).toLocaleTimeString()}`"
    icon="total"
  />

  <MetricCard
    title="Valid Orders"
    :value="metrics.valid.toLocaleString()"
    :subtitle="`${validRate}% success rate`"
    icon="valid"
  />

  <MetricCard
    title="Invalid Orders"
    :value="metrics.invalid.toLocaleString()"
    icon="invalid"
    :danger="true"
  />

  <MetricCard
    title="Deserialization Errors"
    :value="metrics.deserializationErrors.toLocaleString()"
    subtitle="Invalid character/format errors"
    icon="error"
    :warning="true"
  />
</template>
