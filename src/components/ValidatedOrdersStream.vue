<script setup lang="ts">
import { onMounted, onUnmounted, ref } from 'vue'

interface OrderHeader {
  orderId: string
  timestamp: string
  source: string
}

interface Customer {
  customerId: string
  email: string
  name: string
}

interface Location {
  country: string
  city: string
  address: string
}

interface Payment {
  method: string
  amount: number
  currency: string
}

interface OrderItem {
  productId: string
  quantity: number
  price: number
}

interface OrderDetail {
  header: OrderHeader
  customer: Customer
  location: Location
  payment: Payment
  items: OrderItem[]
}

interface ValidatedOrder {
  orderId: string
  correlationId: string
  status: 'VALID' | 'INVALID'
  errors: string[]
  processedAt: string
  order: OrderDetail
}

const orders = ref<ValidatedOrder[]>([])
let interval: ReturnType<typeof setInterval> | null = null

const getRandomName = (): string => {
  const names = ['Alice Johnson', 'Bob Smith', 'Carol Davis', 'David Wilson'] as const
  return names[Math.floor(Math.random() * names.length)]!
}

const getRandomCountry = (): string => {
  const countries = ['USA', 'UK', 'Germany', 'Japan'] as const
  return countries[Math.floor(Math.random() * countries.length)]!
}

const getRandomCity = (): string => {
  const cities = ['New York', 'London', 'Berlin', 'Tokyo'] as const
  return cities[Math.floor(Math.random() * cities.length)]!
}

const getRandomPaymentMethod = (): string => {
  const methods = ['CREDIT_CARD', 'PAYPAL', 'DEBIT_CARD'] as const
  return methods[Math.floor(Math.random() * methods.length)]!
}

const fetchOrders = () => {
  // Simulación de datos del tópico orders-validated
  const mockOrder: ValidatedOrder = {
    orderId: `ORD-${Math.random().toString(36).substr(2, 9).toUpperCase()}`,
    correlationId: `CORR-${Math.random().toString(36).substr(2, 6).toUpperCase()}`,
    status: 'VALID',
    errors: [],
    processedAt: new Date().toISOString(),
    order: {
      header: {
        orderId: `ORD-${Math.random().toString(36).substr(2, 9).toUpperCase()}`,
        timestamp: new Date().toISOString(),
        source: 'WEB'
      },
      customer: {
        customerId: `USR-${Math.random().toString(36).substr(2, 6).toUpperCase()}`,
        email: `user${Math.floor(Math.random() * 1000)}@example.com`,
        name: getRandomName()
      },
      location: {
        country: getRandomCountry(),
        city: getRandomCity(),
        address: `${Math.floor(Math.random() * 999)} Main St`
      },
      payment: {
        method: getRandomPaymentMethod(),
        amount: Math.random() * 500 + 20,
        currency: 'USD'
      },
      items: [
        {
          productId: `PROD-${Math.random().toString(36).substr(2, 4).toUpperCase()}`,
          quantity: Math.floor(Math.random() * 5) + 1,
          price: Math.random() * 100 + 10
        }
      ]
    }
  }

  orders.value = [mockOrder, ...orders.value].slice(0, 10)
}

onMounted(() => {
  fetchOrders()
  interval = setInterval(fetchOrders, 15000) // Cada 15 segundos
})

onUnmounted(() => {
  if (interval) clearInterval(interval)
})
</script>

<template>
  <div class="bg-gradient-to-br from-gray-900/80 to-gray-800/40 border border-gray-700/50 rounded-xl p-6 backdrop-blur-sm">
    <div class="mb-4">
      <h3 class="text-lg font-bold text-white">Real-Time Stream: Validated Orders</h3>
    </div>

    <div class="space-y-3">
      <div
        v-for="(order, idx) in orders"
        :key="order.correlationId"
        :class="[
          'border rounded-lg p-4 transition-all',
          idx === 0
            ? 'bg-[#00ff88]/10 border-[#00ff88]/30 animate-pulse'
            : 'bg-gray-800/40 border-gray-700/30'
        ]"
      >
        <div class="grid grid-cols-4 gap-4 text-sm">
          <div>
            <div class="text-xs text-gray-500 mb-1">Order ID</div>
            <div class="text-[#00ff88] font-mono">{{ order.orderId }}</div>
          </div>
          <div>
            <div class="text-xs text-gray-500 mb-1">Customer</div>
            <div class="text-gray-300">{{ order.order.customer.name }}</div>
          </div>
          <div>
            <div class="text-xs text-gray-500 mb-1">Amount</div>
            <div class="text-gray-300">
              ${{ order.order.payment.amount.toFixed(2) }} {{ order.order.payment.currency }}
            </div>
          </div>
          <div>
            <div class="text-xs text-gray-500 mb-1">Processed At</div>
            <div class="text-gray-300">{{ new Date(order.processedAt).toLocaleTimeString() }}</div>
          </div>
        </div>
        <div class="mt-3 pt-3 border-t border-gray-700/50 grid grid-cols-3 gap-4 text-xs">
          <div>
            <span class="text-gray-500">Location:</span>
            <span class="text-gray-400"> {{ order.order.location.city }}, {{ order.order.location.country }}</span>
          </div>
          <div>
            <span class="text-gray-500">Payment:</span>
            <span class="text-gray-400"> {{ order.order.payment.method }}</span>
          </div>
          <div>
            <span class="text-gray-500">Status:</span>
            <span :class="order.status === 'VALID' ? 'text-[#00ff88]' : 'text-red-400'">
              {{ order.status }}
            </span>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>
