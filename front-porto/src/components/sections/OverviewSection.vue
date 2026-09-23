<!-- src/components/sections/OverviewSection.vue -->
<template>
  <div>
    <!-- Section Header -->
    <div class="flex items-center justify-between mb-6">
      <div class="flex items-center gap-3">
        <h2 class="text-2xl sm:text-3xl font-bold text-gray-900 dark:text-white tracking-tight">About Me</h2>
      </div>
      <button
        v-if="isAdmin"
        @click="$emit('edit-overview', activeSummary)"
        class="flex items-center gap-1.5 px-3 py-1.5 text-xs font-semibold rounded-lg bg-indigo-50 text-indigo-600 hover:bg-indigo-100 dark:bg-indigo-900/40 dark:text-indigo-300 dark:hover:bg-indigo-900/60 border border-indigo-200 dark:border-indigo-700/50 transition-all duration-200"
      >
        <svg xmlns="http://www.w3.org/2000/svg" class="w-3.5 h-3.5" fill="none" viewBox="0 0 24 24" stroke="currentColor">
          <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M11 5H6a2 2 0 00-2 2v11a2 2 0 002 2h11a2 2 0 002-2v-5m-1.414-9.414a2 2 0 112.828 2.828L11.828 15H9v-2.828l8.586-8.586z" />
        </svg>
        Edit Overview
      </button>
    </div>

    <!-- Loading -->
    <div v-if="loading" class="animate-pulse space-y-4 py-4">
      <div class="h-4 bg-gray-200 dark:bg-gray-700 rounded w-full"></div>
      <div class="h-4 bg-gray-200 dark:bg-gray-700 rounded w-5/6"></div>
      <div class="h-4 bg-gray-200 dark:bg-gray-700 rounded w-4/6"></div>
    </div>

    <!-- Content -->
    <div v-else class="relative group">
      <div class="bg-gradient-to-br from-gray-50 to-white dark:from-gray-800/80 dark:to-gray-800/30 border border-gray-200/80 dark:border-gray-700/80 rounded-2xl p-6 sm:p-8 shadow-sm hover:shadow-md transition-shadow duration-300">
        <div class="prose dark:prose-invert max-w-none text-gray-600 dark:text-gray-300 text-base leading-relaxed whitespace-pre-line">
          {{ displayContent }}
        </div>

        <!-- Quick Stats / Highlights -->
        <!-- <div class="mt-6 pt-6 border-t border-gray-200/60 dark:border-gray-700/60 grid grid-cols-2 sm:grid-cols-3 gap-4">
          <div class="p-3 rounded-xl bg-gray-100/60 dark:bg-gray-700/40">
            <span class="block text-xs font-medium text-gray-500 dark:text-gray-400 uppercase tracking-wider">Role</span>
            <span class="text-sm font-semibold text-gray-800 dark:text-gray-200">Software Engineer</span>
          </div>
          <div class="p-3 rounded-xl bg-gray-100/60 dark:bg-gray-700/40">
            <span class="block text-xs font-medium text-gray-500 dark:text-gray-400 uppercase tracking-wider">Location</span>
            <span class="text-sm font-semibold text-gray-800 dark:text-gray-200">Jakarta, Indonesia</span>
          </div>
          <div class="p-3 rounded-xl bg-gray-100/60 dark:bg-gray-700/40 col-span-2 sm:col-span-1">
            <span class="block text-xs font-medium text-gray-500 dark:text-gray-400 uppercase tracking-wider">Focus</span>
            <span class="text-sm font-semibold text-gray-800 dark:text-gray-200">Scalable Systems & AI</span>
          </div>
        </div> -->
      </div>
    </div>
  </div>
</template>

<script>
import { ref, computed, onMounted } from 'vue'
import { summaryAPI } from '@/services/api'

export default {
  name: 'OverviewSection',
  props: {
    currentUser: Object
  },
  emits: ['edit-overview'],
  setup(props) {
    const summaries = ref([])
    const loading = ref(false)

    const defaultFallback = `I am a Software Engineer passionate about building scalable, resilient backend architectures, intuitive frontend experiences, and performant digital products. With a strong foundation in full-stack engineering, distributed systems, and modern web frameworks, I focus on transforming complex workflows into clean, reliable, and user-centric software. I am also actively exploring artificial intelligence integrations to accelerate developer efficiency and product intelligence.`

    const isAdmin = computed(() => props.currentUser?.role === 'admin')

    const activeSummary = computed(() => {
      if (summaries.value && summaries.value.length > 0) {
        return summaries.value[0]
      }
      return null
    })

    const displayContent = computed(() => {
      return activeSummary.value?.content || defaultFallback
    })

    const fetchSummary = async () => {
      loading.value = true
      try {
        const res = await summaryAPI.getAll()
        summaries.value = Array.isArray(res.data) ? res.data : []
      } catch (err) {
        console.warn('Could not fetch remote overview summary, using default.', err)
      } finally {
        loading.value = false
      }
    }

    onMounted(fetchSummary)

    return {
      summaries,
      loading,
      isAdmin,
      activeSummary,
      displayContent,
      fetchSummary
    }
  }
}
</script>
