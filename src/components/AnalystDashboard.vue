<script setup lang="ts">
import { computed, onMounted, onUnmounted, ref } from 'vue'
import MetricCard from './MetricCard.vue'

interface DailyStats {
  totalRevenue: number
  avgOrderValue: number
  newCustomers: number
  totalOrders: number
}

interface RevenuePoint {
  id: string
  day: number
  revenue: number
}

interface Product {
  id: string
  name: string
  sales: number
}

interface Segment {
  id: string
  name: string
  value: number
  color: string
}

const dailyStats = ref<DailyStats>({
  totalRevenue: 0,
  avgOrderValue: 0,
  newCustomers: 0,
  totalOrders: 0
})

const revenueData = ref<RevenuePoint[]>(
  []
)

const topProducts = ref<Product[]>([])

const customerSegments = ref<Segment[]>([])

let statsInterval: ReturnType<typeof setInterval> | null = null
let stream: EventSource | null = null

const maxProductSales = computed(() => {
  return Math.max(...topProducts.value.map((p) => p.sales), 1)
})

const donutStyle = computed(() => {
  const a = customerSegments.value[0]?.value ?? 0
  const b = customerSegments.value[1]?.value ?? 0
  const c = customerSegments.value[2]?.value ?? 0
  const c1 = customerSegments.value[0]?.color ?? '#00ff88'
  const c2 = customerSegments.value[1]?.color ?? '#ff4fd8'
  const c3 = customerSegments.value[2]?.color ?? '#00d4ff'
  return {
    background: `conic-gradient(${c1} 0% ${a}%, ${c2} ${a}% ${a + b}%, ${c3} ${a + b}% ${a + b + c}%)`
  }
})

const chartSize = {
  width: 860,
  height: 320
}

const chartPad = {
  top: 16,
  right: 12,
  bottom: 44,
  left: 58
}

const yTicks = [0, 20000, 40000, 60000, 80000]

const xStep = computed(() => {
  const n = Math.max(revenueData.value.length - 1, 1)
  return (chartSize.width - chartPad.left - chartPad.right) / n
})

const toX = (index: number) => chartPad.left + index * xStep.value
const toY = (value: number) => {
  const h = chartSize.height - chartPad.top - chartPad.bottom
  const max = 80000
  return chartPad.top + (1 - value / max) * h
}

const revenuePath = computed(() => {
  if (!revenueData.value.length) return ''
  return revenueData.value
    .map((p, i) => `${toX(i)},${toY(p.revenue)}`)
    .join(' ')
})

const revenueDots = computed(() => {
  return revenueData.value.map((p, i) => ({
    id: p.id,
    cx: toX(i),
    cy: toY(p.revenue)
  }))
})

const xTicks = computed(() => {
  return revenueData.value.map((p, i) => ({
    day: p.day,
    x: toX(i)
  }))
})

const yTickLayout = computed(() => {
  return yTicks.map((value) => ({
    value,
    y: toY(value)
  }))
})

const fetchDailyStats = async () => {
  try {
    const response = await fetch('/api/analyst/daily')
    if (!response.ok) return
    dailyStats.value = (await response.json()) as DailyStats
  } catch {
    // Keep previous values while backend reconnects.
  }
}

const fetchRevenueTrend = async () => {
  try {
    const response = await fetch('/api/analyst/revenue/trend?days=30')
    if (!response.ok) return
    revenueData.value = (await response.json()) as RevenuePoint[]
  } catch {
    // Keep previous values while backend reconnects.
  }
}

const fetchLatestReport = async () => {
  try {
    const response = await fetch('/api/analyst/report/latest')
    if (!response.ok) return
    const report = (await response.json()) as {
      topProducts: Product[]
      customerSegments: Segment[]
    }
    topProducts.value = report.topProducts ?? []
    customerSegments.value = report.customerSegments ?? []
  } catch {
    // Keep previous values while backend reconnects.
  }
}

const refreshAnalystData = async () => {
  await Promise.all([fetchDailyStats(), fetchRevenueTrend(), fetchLatestReport()])
}

const connectAnalystStream = () => {
  stream = new EventSource('/api/analyst/daily/stream')
  stream.onmessage = async () => {
    await refreshAnalystData()
  }
}

onMounted(() => {
  refreshAnalystData()
  connectAnalystStream()
  statsInterval = setInterval(refreshAnalystData, 300000)
})

onUnmounted(() => {
  if (statsInterval) clearInterval(statsInterval)
  if (stream) stream.close()
})
</script>

<template>
  <div class="flex-1 overflow-auto p-8 analyst-shell">
    <h1 class="text-4xl font-bold text-white mb-8 tracking-tight">
      BUSINESS ANALYTICS
    </h1>

    <div class="grid grid-cols-4 gap-6 mb-8 analyst-metrics-grid">
      <MetricCard
        title="Total Daily Revenue"
        :value="`$${dailyStats.totalRevenue.toLocaleString()}`"
        subtitle="April 19, 2026"
        icon="total"
      />

      <MetricCard
        title="Avg. Order Value"
        :value="`$${dailyStats.avgOrderValue.toFixed(2)}`"
        icon="valid"
      />

      <MetricCard
        title="New Customers Today"
        :value="dailyStats.newCustomers.toLocaleString()"
        icon="valid"
      />

      <MetricCard
        title="Valid Orders Processed"
        :value="dailyStats.totalOrders.toLocaleString()"
        subtitle="Source: orders-validated topic"
        icon="total"
      />
    </div>

    <div class="grid analyst-grid-2 gap-6 mb-8">
      <div class="bg-gradient-to-br from-gray-900/80 to-gray-800/40 border border-gray-700/50 rounded-xl p-6 backdrop-blur-sm">
        <h3 class="text-lg font-bold text-white mb-4">Top 5 Products</h3>
        <div class="space-y-3">
          <div v-for="product in topProducts" :key="product.id" class="space-y-2">
            <div class="flex items-center justify-between text-sm">
              <span class="text-gray-300">{{ product.name }}</span>
              <span class="text-gray-400">{{ product.sales }}</span>
            </div>
            <div class="h-2 bg-gray-700 rounded overflow-hidden">
              <div class="h-2 bg-brand-60 rounded" :style="{ width: `${(product.sales / maxProductSales) * 100}%` }" />
            </div>
          </div>
        </div>
      </div>

      <div class="bg-gradient-to-br from-gray-900/80 to-gray-800/40 border border-gray-700/50 rounded-xl p-6 backdrop-blur-sm">
        <h3 class="text-lg font-bold text-white mb-4">Customer Segmentation</h3>
        <div class="flex items-center gap-6 analyst-segment-layout">
          <div class="donut-wrap">
            <div class="donut" :style="donutStyle" />
            <div class="donut-center" />
          </div>

          <div class="space-y-3 flex-1">
            <div v-for="segment in customerSegments" :key="segment.id" class="flex items-center justify-between text-sm">
              <div class="flex items-center gap-2">
                <span class="w-3 h-3 rounded" :style="{ backgroundColor: segment.color }" />
                <span class="text-gray-300">{{ segment.name }}</span>
              </div>
              <span class="text-gray-400">{{ segment.value }}%</span>
            </div>
          </div>
        </div>
      </div>
    </div>

    <div class="bg-gradient-to-br from-gray-900/80 to-gray-800/40 border border-gray-700/50 rounded-xl p-6 backdrop-blur-sm">
      <h3 class="text-lg font-bold text-white mb-4">Revenue Trend (Last 30 Days)</h3>

      <div class="revenue-chart">
        <svg :viewBox="`0 0 ${chartSize.width} ${chartSize.height}`" preserveAspectRatio="none" class="revenue-svg">
          <g>
            <line
              v-for="tick in yTickLayout"
              :key="`y-grid-${tick.value}`"
              :x1="chartPad.left"
              :x2="chartSize.width - chartPad.right"
              :y1="tick.y"
              :y2="tick.y"
              stroke="#4b5563"
              stroke-opacity="0.42"
              stroke-dasharray="4 5"
            />

            <line
              v-for="tick in xTicks"
              :key="`x-grid-${tick.day}`"
              :x1="tick.x"
              :x2="tick.x"
              :y1="chartPad.top"
              :y2="chartSize.height - chartPad.bottom"
              stroke="#4b5563"
              stroke-opacity="0.35"
              stroke-dasharray="4 5"
            />
          </g>

          <line
            :x1="chartPad.left"
            :x2="chartPad.left"
            :y1="chartPad.top"
            :y2="chartSize.height - chartPad.bottom"
            stroke="#9ca3af"
            stroke-opacity="0.65"
          />
          <line
            :x1="chartPad.left"
            :x2="chartSize.width - chartPad.right"
            :y1="chartSize.height - chartPad.bottom"
            :y2="chartSize.height - chartPad.bottom"
            stroke="#9ca3af"
            stroke-opacity="0.65"
          />

          <polyline
            :points="revenuePath"
            fill="none"
            stroke="#00ff88"
            stroke-width="4"
            stroke-linejoin="round"
            stroke-linecap="round"
          />

          <circle
            v-for="dot in revenueDots"
            :key="`dot-${dot.id}`"
            :cx="dot.cx"
            :cy="dot.cy"
            r="7"
            fill="#00ff88"
            stroke="#0b1026"
            stroke-width="3"
          />

          <text
            v-for="tick in yTickLayout"
            :key="`y-label-${tick.value}`"
            :x="chartPad.left - 8"
            :y="tick.y + 4"
            text-anchor="end"
            font-size="14"
            fill="#9ca3af"
          >
            {{ `$${Math.round(tick.value / 1000)}k` }}
          </text>

          <text
            v-for="tick in xTicks"
            :key="`x-label-${tick.day}`"
            :x="tick.x"
            :y="chartSize.height - chartPad.bottom + 18"
            text-anchor="middle"
            font-size="13"
            fill="#9ca3af"
          >
            {{ tick.day }}
          </text>

          <text
            :x="(chartPad.left + chartSize.width - chartPad.right) / 2"
            :y="chartSize.height - 6"
            text-anchor="middle"
            font-size="14"
            fill="#9ca3af"
          >
            Day of Month
          </text>
        </svg>
      </div>
    </div>
  </div>
</template>

<style scoped>
.analyst-grid-2 {
  grid-template-columns: repeat(2, minmax(0, 1fr));
}

.donut-wrap {
  position: relative;
  width: 140px;
  height: 140px;
  flex-shrink: 0;
}

.donut {
  width: 100%;
  height: 100%;
  border-radius: 9999px;
}

.donut-center {
  position: absolute;
  width: 68px;
  height: 68px;
  border-radius: 9999px;
  background: #111111;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  border: 1px solid rgba(45, 45, 45, 0.5);
}

.revenue-chart {
  height: 320px;
  border: 1px solid rgba(45, 45, 45, 0.5);
  border-radius: 12px;
  background: radial-gradient(circle at 20% 0%, rgba(29, 78, 216, 0.12), rgba(8, 12, 34, 0.88));
  padding: 12px;
}

.revenue-svg {
  width: 100%;
  height: 100%;
}

@media (max-width: 1024px) {
  .analyst-shell {
    padding: 1rem;
  }

  .analyst-metrics-grid {
    grid-template-columns: repeat(2, minmax(0, 1fr));
  }

  .analyst-grid-2 {
    grid-template-columns: 1fr;
  }
}

@media (max-width: 768px) {
  .analyst-metrics-grid {
    grid-template-columns: 1fr;
  }

  .analyst-segment-layout {
    flex-direction: column;
    align-items: flex-start;
  }

  .donut-wrap {
    width: 120px;
    height: 120px;
  }

  .donut-center {
    width: 56px;
    height: 56px;
  }

  .revenue-chart {
    height: 250px;
  }
}
</style>