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
  Array.from({ length: 30 }, (_, i) => ({
    id: `day-${i}`,
    day: i + 1,
    revenue: Math.floor(Math.random() * 50000 + 30000)
  }))
)

const topProducts = ref<Product[]>([
  { id: 'prod-1', name: 'Wireless Headphones', sales: 1243 },
  { id: 'prod-2', name: 'Smart Watch', sales: 982 },
  { id: 'prod-3', name: 'Laptop Stand', sales: 756 },
  { id: 'prod-4', name: 'USB-C Hub', sales: 654 },
  { id: 'prod-5', name: 'Mechanical Keyboard', sales: 543 }
])

const customerSegments = ref<Segment[]>([
  { id: 'seg-1', name: 'New', value: 35, color: '#00ff88' },
  { id: 'seg-2', name: 'Returning', value: 45, color: '#ff4fd8' },
  { id: 'seg-3', name: 'VIP', value: 20, color: '#00d4ff' }
])

let statsInterval: ReturnType<typeof setInterval> | null = null

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

const revenuePath = computed(() => {
  const points = revenueData.value
  if (!points.length) return ''

  const width = 100
  const height = 40
  const min = Math.min(...points.map((p) => p.revenue))
  const max = Math.max(...points.map((p) => p.revenue))
  const range = Math.max(max - min, 1)

  return points
    .map((p, i) => {
      const x = (i / (points.length - 1)) * width
      const y = height - ((p.revenue - min) / range) * height
      return `${x},${y}`
    })
    .join(' ')
})

const fetchDailyStats = () => {
  dailyStats.value = {
    totalRevenue: Math.floor(Math.random() * 100000 + 100000),
    avgOrderValue: Math.floor(Math.random() * 100 + 150),
    newCustomers: Math.floor(Math.random() * 300 + 200),
    totalOrders: Math.floor(Math.random() * 2000 + 15000)
  }
}

onMounted(() => {
  fetchDailyStats()
  statsInterval = setInterval(fetchDailyStats, 300000)
})

onUnmounted(() => {
  if (statsInterval) clearInterval(statsInterval)
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
        <svg viewBox="0 0 100 40" preserveAspectRatio="none" class="revenue-svg">
          <polyline :points="revenuePath" fill="none" stroke="#00ff88" stroke-width="1.5" />
        </svg>
      </div>

      <div class="flex items-center justify-between text-xs text-gray-500 mt-3">
        <span>Day 1</span>
        <span>Day 10</span>
        <span>Day 20</span>
        <span>Day 30</span>
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
  height: 280px;
  border: 1px solid rgba(45, 45, 45, 0.5);
  border-radius: 12px;
  background: rgba(0, 0, 0, 0.2);
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
    height: 220px;
  }
}
</style>