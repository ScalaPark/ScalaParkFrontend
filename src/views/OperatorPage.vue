<script setup lang="ts">
import { ref } from 'vue'
import Sidebar from '../components/Sidebar.vue'
import OperatorDashboard from '../components/OperatorDashboard.vue'
import GlyphIcon from '../components/GlyphIcon.vue'

const view = ref<'operator' | 'analyst'>('operator')

const handleViewChange = (newView: 'operator' | 'analyst') => {
  view.value = newView
}
</script>

<template>
  <div class="size-full flex bg-gradient-to-br from-brand-start via-brand-mid to-brand-end">
    <Sidebar :view="view" />

    <div class="flex-1 flex flex-col">
      <header class="h-16 border-b border-gray-700/50 bg-black/20 backdrop-blur-sm flex items-center justify-between px-8">
        <div class="flex items-center gap-4">
          <div class="text-sm text-gray-400">
            Real-Time E-commerce Monitoring System
          </div>
        </div>

        <div class="flex items-center gap-6">
          <div class="flex items-center gap-2 bg-gray-900/60 border border-gray-700/50 rounded-lg p-1">
            <button
              @click="handleViewChange('operator')"
              :class="[
                'flex items-center gap-2 px-4 py-2 rounded-md text-sm font-medium transition-all',
                view === 'operator'
                  ? 'bg-brand text-black'
                  : 'text-gray-400 hover:text-white'
              ]"
            >
              <GlyphIcon name="operator" :danger="false" />
              OPERATOR
            </button>
            <button
              @click="handleViewChange('analyst')"
              :class="[
                'flex items-center gap-2 px-4 py-2 rounded-md text-sm font-medium transition-all',
                view === 'analyst'
                  ? 'bg-brand text-black'
                  : 'text-gray-400 hover:text-white'
              ]"
            >
              <GlyphIcon name="analyst" :danger="false" />
              ANALYST
            </button>
          </div>

          <div class="flex items-center gap-3">
            <div class="w-8 h-8 bg-gradient-to-br from-brand-start to-brand-cyan rounded-full flex items-center justify-center text-black font-bold text-sm">
              SP
            </div>
          </div>
        </div>
      </header>

      <OperatorDashboard v-if="view === 'operator'" />

      <footer class="border-t border-gray-700/50 bg-black/20 backdrop-blur-sm px-8 py-4">
        <div class="flex items-center justify-center gap-8 text-sm text-gray-400">
          <div class="text-xs text-gray-500">Desarrolladores:</div>
          <div class="flex items-center gap-6">
            <span class="text-gray-300">Anderson Fabian Garcia Nieto</span>
            <span class="text-gray-600">•</span>
            <span class="text-gray-300">David Alejandro Patacon Henao</span>
            <span class="text-gray-600">•</span>
            <span class="text-gray-300">Tulio Riaño Sanchez</span>
          </div>
        </div>
      </footer>
    </div>
  </div>
</template>
