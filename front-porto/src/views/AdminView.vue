<!-- src/views/AdminView.vue -->
<template>
  <div class="min-h-screen bg-gray-50 dark:bg-gray-900 text-gray-900 dark:text-white transition-colors duration-300">

    <!-- Top Admin Header -->
    <header class="bg-white dark:bg-gray-800 border-b border-gray-200 dark:border-gray-700 sticky top-0 z-30 shadow-sm">
      <div class="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8 h-16 flex items-center justify-between">
        <div class="flex items-center gap-4">
          <router-link
            to="/"
            class="flex items-center gap-1.5 text-xs sm:text-sm font-medium text-gray-500 hover:text-gray-900 dark:text-gray-400 dark:hover:text-white transition-colors"
          >
            ← Back to Portfolio
          </router-link>
          <span class="text-gray-300 dark:text-gray-600">|</span>
          <div class="flex items-center gap-2">
            <span class="w-2.5 h-2.5 rounded-full bg-indigo-500"></span>
            <h1 class="text-base sm:text-lg font-bold tracking-tight">Admin Management Panel</h1>
          </div>
        </div>

        <div class="flex items-center gap-3">
          <span class="text-xs text-gray-500 dark:text-gray-400 hidden sm:inline">
            Logged in as <strong class="text-indigo-600 dark:text-indigo-400">{{ currentUser?.name || currentUser?.username }}</strong>
          </span>

          <button
            @click="$emit('toggle-dark-mode')"
            class="w-8 h-8 rounded-lg flex items-center justify-center text-sm bg-gray-100 dark:bg-gray-700 hover:bg-gray-200 dark:hover:bg-gray-600 transition-colors"
            title="Toggle theme"
          >
            {{ darkMode ? '☀️' : '🌙' }}
          </button>

          <button
            @click="handleLogout"
            class="px-3 py-1.5 text-xs font-semibold rounded-lg border border-red-200 dark:border-red-800/60 text-red-600 dark:text-red-400 hover:bg-red-50 dark:hover:bg-red-900/30 transition-colors"
          >
            Logout
          </button>
        </div>
      </div>
    </header>

    <!-- Main Container -->
    <main class="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8 py-8">

      <!-- Navigation Tabs -->
      <div class="flex items-center gap-2 border-b border-gray-200 dark:border-gray-700 pb-4 mb-8 overflow-x-auto">
        <button
          v-for="tab in tabs"
          :key="tab.id"
          @click="activeTab = tab.id"
          class="flex items-center gap-2 px-4 py-2.5 rounded-xl text-xs sm:text-sm font-semibold transition-all duration-200 shrink-0"
          :class="activeTab === tab.id
            ? 'bg-indigo-600 text-white shadow-md shadow-indigo-500/20'
            : 'bg-white dark:bg-gray-800 text-gray-600 dark:text-gray-400 hover:bg-gray-100 dark:hover:bg-gray-750 border border-gray-200 dark:border-gray-700'"
        >
          <span>{{ tab.icon }}</span>
          <span>{{ tab.label }}</span>
          <span
            class="ml-1 px-1.5 py-0.5 rounded-full text-[10px]"
            :class="activeTab === tab.id ? 'bg-indigo-700 text-white' : 'bg-gray-200 dark:bg-gray-700 text-gray-700 dark:text-gray-300'"
          >
            {{ getItemCount(tab.id) }}
          </span>
        </button>
      </div>

      <!-- Tab Content Area -->
      <div class="bg-white dark:bg-gray-800 rounded-2xl border border-gray-200 dark:border-gray-700 shadow-sm overflow-hidden p-6">

        <!-- Top Header per Tab -->
        <div class="flex flex-col sm:flex-row sm:items-center justify-between gap-4 pb-6 mb-6 border-b border-gray-100 dark:border-gray-700">
          <div>
            <h2 class="text-xl font-bold capitalize">{{ currentTabInfo.label }} Management</h2>
            <p class="text-xs text-gray-500 dark:text-gray-400 mt-0.5">{{ currentTabInfo.description }}</p>
          </div>

          <button
            @click="openAddModal"
            class="flex items-center justify-center gap-2 px-4 py-2.5 bg-indigo-600 hover:bg-indigo-700 text-white text-xs sm:text-sm font-semibold rounded-xl shadow-sm transition-all duration-200"
          >
            <svg xmlns="http://www.w3.org/2000/svg" class="w-4 h-4" fill="none" viewBox="0 0 24 24" stroke="currentColor">
              <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M12 4v16m8-8H4" />
            </svg>
            Add New {{ currentTabSingular }}
          </button>
        </div>

        <!-- Loading State -->
        <div v-if="loading" class="text-center py-16 text-gray-500 dark:text-gray-400 animate-pulse text-sm">
          Loading {{ activeTab }}...
        </div>

        <!-- Empty State -->
        <div v-else-if="currentItems.length === 0" class="text-center py-16 px-4 rounded-xl border border-dashed border-gray-300 dark:border-gray-700">
          <p class="text-sm font-medium text-gray-600 dark:text-gray-400">No items found in {{ activeTab }}.</p>
          <button
            @click="openAddModal"
            class="mt-3 text-xs font-semibold text-indigo-600 dark:text-indigo-400 hover:underline"
          >
            + Create your first {{ currentTabSingular }}
          </button>
        </div>

        <!-- Items Table/Card List -->
        <div v-else class="space-y-3">
          <div
            v-for="item in currentItems"
            :key="item.id"
            class="p-4 rounded-xl border border-gray-100 dark:border-gray-700 bg-gray-50/50 dark:bg-gray-850 flex flex-col sm:flex-row sm:items-center justify-between gap-4 hover:border-gray-300 dark:hover:border-gray-600 transition-colors"
          >
            <div class="flex-1 min-w-0">
              <div class="flex items-center gap-2">
                <h3 class="text-sm font-bold text-gray-900 dark:text-white truncate">
                  {{ item.name || item.company || item.title || (item.content ? item.content.slice(0, 50) + '...' : 'Overview Bio') }}
                </h3>
                <span v-if="item.role" class="text-xs px-2 py-0.5 rounded bg-orange-100 dark:bg-orange-950/60 text-orange-700 dark:text-orange-300 font-medium">
                  {{ item.role }}
                </span>
                <span v-if="item.issuer" class="text-xs px-2 py-0.5 rounded bg-amber-100 dark:bg-amber-950/60 text-amber-700 dark:text-amber-300 font-medium">
                  {{ item.issuer }}
                </span>
              </div>

              <p v-if="item.description || item.content" class="text-xs text-gray-500 dark:text-gray-400 mt-1 line-clamp-2">
                {{ item.description || item.content }}
              </p>

              <div class="flex items-center gap-3 mt-2 text-[11px] text-gray-400 dark:text-gray-500">
                <span v-if="item.startDate">Started: {{ item.startDate }}</span>
                <span v-if="item.issueDate">Issued: {{ item.issueDate }}</span>
                <a
                  v-if="item.url || item.credentialUrl"
                  :href="item.url || item.credentialUrl"
                  target="_blank"
                  class="text-indigo-600 dark:text-indigo-400 hover:underline flex items-center gap-1"
                >
                  Link ↗
                </a>
              </div>
            </div>

            <!-- Actions -->
            <div class="flex items-center gap-2 self-end sm:self-center shrink-0">
              <button
                @click="openEditModal(item)"
                class="px-3 py-1.5 text-xs font-semibold rounded-lg bg-gray-100 dark:bg-gray-700 text-gray-700 dark:text-gray-200 hover:bg-gray-200 dark:hover:bg-gray-650 transition-colors"
              >
                Edit
              </button>
              <button
                @click="handleDeleteItem(item.id)"
                class="px-3 py-1.5 text-xs font-semibold rounded-lg border border-red-200 dark:border-red-900/50 text-red-600 dark:text-red-400 hover:bg-red-50 dark:hover:bg-red-900/30 transition-colors"
              >
                Delete
              </button>
            </div>
          </div>
        </div>

      </div>
    </main>

    <!-- Modal for adding/editing items -->
    <ItemModal
      :show="showModal"
      :itemType="modalType"
      :itemData="selectedItem"
      @close="showModal = false"
      @saved="fetchCurrentTabData"
    />

  </div>
</template>

<script>
import { ref, computed, onMounted, watch } from 'vue'
import { useRouter } from 'vue-router'
import {
  experiencesAPI,
  projectsAPI,
  skillsAPI,
  certificationsAPI,
  summaryAPI
} from '@/services/api'
import ItemModal from '@/components/ItemModal.vue'

export default {
  name: 'AdminView',
  components: { ItemModal },
  props: {
    darkMode: Boolean,
    currentUser: Object
  },
  emits: ['toggle-dark-mode', 'logout'],
  setup(props, { emit }) {
    const router = useRouter()
    const activeTab = ref('experiences')
    const loading = ref(false)

    // Data lists
    const experiences = ref([])
    const projects = ref([])
    const skills = ref([])
    const certifications = ref([])
    const summaries = ref([])

    // Modal state
    const showModal = ref(false)
    const selectedItem = ref(null)

    const tabs = [
      { id: 'experiences',    label: 'Experiences',    icon: '💼', description: 'Manage job history, roles, and descriptions.' },
      { id: 'projects',       label: 'Projects',       icon: '🚀', description: 'Manage portfolio projects, tech stacks, and links.' },
      { id: 'skills',         label: 'Skills',         icon: '⚡', description: 'Manage technologies, proficiencies, and categories.' },
      { id: 'certifications', label: 'Certifications', icon: '🏆', description: 'Manage professional certificates and credential badges.' },
      { id: 'overview',       label: 'Overview Bio',   icon: '📝', description: 'Manage personal bio and introductory summary text.' }
    ]

    const currentTabInfo = computed(() => {
      return tabs.find(t => t.id === activeTab.value) || tabs[0]
    })

    const currentTabSingular = computed(() => {
      switch (activeTab.value) {
        case 'experiences': return 'Experience'
        case 'projects': return 'Project'
        case 'skills': return 'Skill'
        case 'certifications': return 'Certification'
        case 'overview': return 'Overview Bio'
        default: return 'Item'
      }
    })

    const modalType = computed(() => {
      switch (activeTab.value) {
        case 'experiences': return 'experience'
        case 'projects': return 'project'
        case 'skills': return 'skill'
        case 'certifications': return 'certification'
        case 'overview': return 'overview'
        default: return 'project'
      }
    })

    const currentItems = computed(() => {
      switch (activeTab.value) {
        case 'experiences': return experiences.value
        case 'projects': return projects.value
        case 'skills': return skills.value
        case 'certifications': return certifications.value
        case 'overview': return summaries.value
        default: return []
      }
    })

    const getItemCount = (tabId) => {
      switch (tabId) {
        case 'experiences': return experiences.value.length
        case 'projects': return projects.value.length
        case 'skills': return skills.value.length
        case 'certifications': return certifications.value.length
        case 'overview': return summaries.value.length
        default: return 0
      }
    }

    const fetchAll = async () => {
      loading.value = true
      try {
        const [expRes, projRes, skillRes, certRes, sumRes] = await Promise.allSettled([
          experiencesAPI.getAll(),
          projectsAPI.getAll(),
          skillsAPI.getAll(),
          certificationsAPI.getAll(),
          summaryAPI.getAll()
        ])

        if (expRes.status === 'fulfilled') experiences.value = Array.isArray(expRes.value.data) ? expRes.value.data : []
        if (projRes.status === 'fulfilled') projects.value = Array.isArray(projRes.value.data) ? projRes.value.data : []
        if (skillRes.status === 'fulfilled') skills.value = Array.isArray(skillRes.value.data) ? skillRes.value.data : []
        if (certRes.status === 'fulfilled') certifications.value = Array.isArray(certRes.value.data) ? certRes.value.data : []
        if (sumRes.status === 'fulfilled') summaries.value = Array.isArray(sumRes.value.data) ? sumRes.value.data : []
      } finally {
        loading.value = false
      }
    }

    const fetchCurrentTabData = async () => {
      if (activeTab.value === 'experiences') {
        const res = await experiencesAPI.getAll()
        experiences.value = Array.isArray(res.data) ? res.data : []
      } else if (activeTab.value === 'projects') {
        const res = await projectsAPI.getAll()
        projects.value = Array.isArray(res.data) ? res.data : []
      } else if (activeTab.value === 'skills') {
        const res = await skillsAPI.getAll()
        skills.value = Array.isArray(res.data) ? res.data : []
      } else if (activeTab.value === 'certifications') {
        const res = await certificationsAPI.getAll()
        certifications.value = Array.isArray(res.data) ? res.data : []
      } else if (activeTab.value === 'overview') {
        const res = await summaryAPI.getAll()
        summaries.value = Array.isArray(res.data) ? res.data : []
      }
    }

    const openAddModal = () => {
      selectedItem.value = null
      showModal.value = true
    }

    const openEditModal = (item) => {
      selectedItem.value = item
      showModal.value = true
    }

    const handleDeleteItem = async (id) => {
      if (!confirm(`Are you sure you want to delete this ${currentTabSingular.value}?`)) return
      try {
        if (activeTab.value === 'experiences') await experiencesAPI.delete(id)
        else if (activeTab.value === 'projects') await projectsAPI.delete(id)
        else if (activeTab.value === 'skills') await skillsAPI.delete(id)
        else if (activeTab.value === 'certifications') await certificationsAPI.delete(id)
        else if (activeTab.value === 'overview') await summaryAPI.delete(id)

        await fetchCurrentTabData()
      } catch (err) {
        alert('Failed to delete item')
      }
    }

    const handleLogout = () => {
      emit('logout')
      router.push('/')
    }

    onMounted(() => {
      fetchAll()
    })

    return {
      activeTab,
      tabs,
      currentTabInfo,
      currentTabSingular,
      modalType,
      currentItems,
      loading,
      getItemCount,
      showModal,
      selectedItem,
      openAddModal,
      openEditModal,
      handleDeleteItem,
      fetchCurrentTabData,
      handleLogout
    }
  }
}
</script>
