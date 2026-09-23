<!-- src/components/sections/CertificationsSection.vue -->
<template>
  <div>
    <!-- Section Header -->
    <div class="flex items-center justify-between mb-8">
      <div class="flex items-center gap-3">
        <h2 class="text-2xl sm:text-3xl font-bold text-gray-900 dark:text-white tracking-tight">Certifications & Credentials</h2>
      </div>
      <button
        v-if="isAdmin"
        @click="$emit('add-certification')"
        class="flex items-center gap-1.5 px-3.5 py-1.5 bg-blue-600 hover:bg-blue-700 dark:bg-blue-500 dark:hover:bg-blue-600 text-white text-xs sm:text-sm font-medium rounded-lg shadow-sm hover:shadow transition-all duration-200"
      >
        <svg xmlns="http://www.w3.org/2000/svg" class="w-4 h-4" fill="none" viewBox="0 0 24 24" stroke="currentColor">
          <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M12 4v16m8-8H4" />
        </svg>
        Add Certification
      </button>
    </div>

    <!-- Loading -->
    <div v-if="loading" class="text-center py-12 text-gray-500 dark:text-gray-400 text-base animate-pulse">
      Loading certifications...
    </div>

    <!-- Error -->
    <div v-else-if="error" class="text-center py-8 text-red-500 text-sm">
      {{ error }}
    </div>

    <!-- Empty State -->
    <div v-else-if="certifications.length === 0" class="text-center py-12 px-4 rounded-2xl border border-dashed border-gray-300 dark:border-gray-700">
      <div class="inline-flex items-center justify-center w-12 h-12 rounded-full bg-amber-50 dark:bg-amber-900/30 text-amber-500 mb-3">
        <svg xmlns="http://www.w3.org/2000/svg" class="w-6 h-6" fill="none" viewBox="0 0 24 24" stroke="currentColor">
          <path stroke-linecap="round" stroke-linejoin="round" stroke-width="1.8" d="M9 12l2 2 4-4M7.835 4.697a3.42 3.42 0 001.946-.806 3.42 3.42 0 014.438 0 3.42 3.42 0 001.946.806 3.42 3.42 0 013.138 3.138 3.42 3.42 0 00.806 1.946 3.42 3.42 0 010 4.438 3.42 3.42 0 00-.806 1.946 3.42 3.42 0 01-3.138 3.138 3.42 3.42 0 00-1.946.806 3.42 3.42 0 01-4.438 0 3.42 3.42 0 00-1.946-.806 3.42 3.42 0 01-3.138-3.138 3.42 3.42 0 00-.806-1.946 3.42 3.42 0 010-4.438 3.42 3.42 0 00.806-1.946 3.42 3.42 0 013.138-3.138z" />
        </svg>
      </div>
      <p class="text-sm font-medium text-gray-600 dark:text-gray-300">No certifications listed yet.</p>
      <p v-if="isAdmin" class="text-xs text-gray-400 mt-1">Use the button above to add verified credentials.</p>
    </div>

    <!-- Cards Grid -->
    <div v-else class="grid grid-cols-1 md:grid-cols-2 gap-5">
      <div
        v-for="cert in certifications"
        :key="cert.id"
        class="bg-gray-50/90 dark:bg-gray-800/80 border border-gray-200 dark:border-gray-700 rounded-2xl p-5 hover:-translate-y-1 hover:shadow-lg transition-all duration-300 flex flex-col justify-between"
      >
        <div>
          <!-- Header -->
          <div class="flex items-start justify-between gap-3 mb-2">
            <div class="flex items-center gap-2.5">
              <div class="w-9 h-9 rounded-xl bg-amber-500/10 text-amber-600 dark:text-amber-400 flex items-center justify-center shrink-0">
                <svg xmlns="http://www.w3.org/2000/svg" class="w-5 h-5" fill="none" viewBox="0 0 24 24" stroke="currentColor">
                  <path stroke-linecap="round" stroke-linejoin="round" stroke-width="1.8" d="M9 12l2 2 4-4m5.618-4.016A11.955 11.955 0 0112 2.944a11.955 11.955 0 01-8.618 3.04A12.02 12.02 0 003 9c0 5.591 3.824 10.29 9 11.622 5.176-1.332 9-6.03 9-11.622 0-1.042-.133-2.052-.382-3.016z" />
                </svg>
              </div>
              <div>
                <h3 class="text-base font-bold text-gray-900 dark:text-white leading-snug">{{ cert.title }}</h3>
                <span class="text-xs font-semibold text-amber-600 dark:text-amber-400">{{ cert.issuer }}</span>
              </div>
            </div>

            <!-- Admin Actions -->
            <div v-if="isAdmin" class="flex items-center gap-1.5 shrink-0">
              <button
                @click="$emit('edit-certification', cert)"
                class="px-2 py-1 text-xs font-medium border border-gray-300 dark:border-gray-600 text-gray-600 dark:text-gray-300 rounded-md hover:bg-blue-600 hover:border-blue-600 hover:text-white dark:hover:bg-blue-500 dark:hover:border-blue-500 transition-all duration-200"
              >
                Edit
              </button>
              <button
                @click="handleDelete(cert.id)"
                class="px-2 py-1 text-xs font-medium border border-gray-300 dark:border-gray-600 text-gray-600 dark:text-gray-300 rounded-md hover:bg-red-600 hover:border-red-600 hover:text-white dark:hover:bg-red-500 dark:hover:border-red-500 transition-all duration-200"
              >
                Delete
              </button>
            </div>
          </div>

          <!-- Description -->
          <p v-if="cert.description" class="text-xs sm:text-sm text-gray-600 dark:text-gray-400 mt-2 leading-relaxed">
            {{ cert.description }}
          </p>
        </div>

        <!-- Card Footer -->
        <div class="mt-4 pt-3 border-t border-gray-200/60 dark:border-gray-700/60 flex items-center justify-between text-xs text-gray-500 dark:text-gray-400">
          <div class="flex items-center gap-2">
            <span v-if="cert.issueDate">Issued: {{ cert.issueDate }}</span>
            <span v-if="cert.expirationDate" class="hidden sm:inline">• Exp: {{ cert.expirationDate }}</span>
          </div>

          <a
            v-if="cert.credentialUrl"
            :href="cert.credentialUrl"
            target="_blank"
            rel="noopener noreferrer"
            class="inline-flex items-center gap-1 text-blue-600 hover:text-blue-700 dark:text-blue-400 dark:hover:text-blue-300 font-medium transition-colors"
          >
            <span>Verify</span>
            <svg xmlns="http://www.w3.org/2000/svg" class="w-3.5 h-3.5" fill="none" viewBox="0 0 24 24" stroke="currentColor">
              <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M10 6H6a2 2 0 00-2 2v10a2 2 0 002 2h10a2 2 0 002-2v-4M14 4h6m0 0v6m0-6L10 14" />
            </svg>
          </a>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import { ref, computed, onMounted } from 'vue'
import { certificationsAPI } from '@/services/api'

export default {
  name: 'CertificationsSection',
  props: {
    currentUser: Object
  },
  emits: ['add-certification', 'edit-certification'],
  setup(props) {
    const certifications = ref([])
    const loading = ref(false)
    const error = ref(null)

    const isAdmin = computed(() => props.currentUser?.role === 'admin')

    const loadCertifications = async () => {
      loading.value = true
      error.value = null
      try {
        const response = await certificationsAPI.getAll()
        certifications.value = Array.isArray(response.data) ? response.data : []
      } catch (err) {
        error.value = 'Failed to load certifications'
        console.error('Error loading certifications:', err)
      } finally {
        loading.value = false
      }
    }

    const handleDelete = async (id) => {
      if (!confirm('Are you sure you want to delete this certification?')) return
      try {
        await certificationsAPI.delete(id)
        certifications.value = certifications.value.filter(c => c.id !== id)
      } catch (err) {
        alert('Failed to delete certification')
      }
    }

    onMounted(loadCertifications)

    return {
      certifications,
      loading,
      error,
      isAdmin,
      loadCertifications,
      handleDelete
    }
  }
}
</script>
