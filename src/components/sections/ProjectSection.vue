<!-- src/components/sections/ProjectSection.vue -->
<template>
  <div>
    <!-- Section Header -->
    <div class="flex items-center justify-between mb-8">
      <h2 class="text-3xl font-bold text-gray-900 dark:text-white tracking-tight">Projects</h2>
      <button
        v-if="currentUser"
        @click="openCreate"
        class="flex items-center gap-1 px-4 py-2 bg-blue-600 hover:bg-blue-700 dark:bg-blue-500 dark:hover:bg-blue-600 text-white text-sm font-medium rounded-lg transition-colors duration-200"
      >
        + Add Project
      </button>
    </div>

    <!-- Loading -->
    <div v-if="loading" class="text-center py-12 text-gray-500 dark:text-gray-400 text-lg">
      Loading projects...
    </div>

    <!-- Error -->
    <div v-else-if="error" class="text-center py-12 text-red-500 text-lg">{{ error }}</div>

    <!-- Grid -->
    <div v-else class="flex flex-col gap-6">
      <div
        v-for="project in projects"
        :key="project.id"
        class="bg-gray-50 dark:bg-gray-800 border border-gray-200 dark:border-gray-700 rounded-xl p-6 hover:-translate-y-1 hover:shadow-xl transition-all duration-300"
      >
        <!-- Card Header -->
        <div class="flex items-start justify-between mb-3">
          <h3 class="text-xl font-semibold text-gray-900 dark:text-white">{{ project.name }}</h3>
          <div class="flex items-center gap-2">
            <template v-if="canManage(project)">
              <button
                @click="openEdit(project)"
                class="px-2 py-1 text-xs border border-gray-300 dark:border-gray-600 text-gray-600 dark:text-gray-300 rounded hover:bg-blue-600 hover:border-blue-600 hover:text-white dark:hover:bg-blue-500 dark:hover:border-blue-500 transition-all duration-200"
              >Edit</button>
              <button
                @click="deleteProject(project.id)"
                class="px-2 py-1 text-xs border border-gray-300 dark:border-gray-600 text-gray-600 dark:text-gray-300 rounded hover:bg-red-600 hover:border-red-600 hover:text-white dark:hover:bg-red-500 dark:hover:border-red-500 transition-all duration-200"
              >Delete</button>
            </template>
            <a
              v-if="project.url"
              :href="project.url"
              target="_blank"
              class="p-2 text-gray-400 dark:text-gray-500 hover:text-blue-600 dark:hover:text-blue-400 hover:bg-gray-100 dark:hover:bg-gray-700 rounded-md transition-all duration-200"
            >
              <svg width="16" height="16" viewBox="0 0 24 24" fill="none">
                <path d="M18 13v6a2 2 0 0 1-2 2H5a2 2 0 0 1-2-2V8a2 2 0 0 1 2-2h6" stroke="currentColor" stroke-width="2"></path>
                <polyline points="15 3 21 3 21 9" stroke="currentColor" stroke-width="2"></polyline>
                <line x1="10" y1="14" x2="21" y2="3" stroke="currentColor" stroke-width="2"></line>
              </svg>
            </a>
          </div>
        </div>

        <p class="text-gray-600 dark:text-gray-400 leading-relaxed">{{ project.description }}</p>

        <div v-if="project.imageUrl" class="mt-4 rounded-lg overflow-hidden">
          <img :src="project.imageUrl" :alt="project.name" class="w-full h-auto rounded-lg" />
        </div>
      </div>
    </div>

    <RecordEditorModal
      :open="editorOpen"
      :title="editingRecord ? 'Edit project' : 'Add project'"
      :fields="fields"
      :initial-value="editingRecord || emptyRecord"
      :saving="saving"
      :error="editorError"
      @save="saveProject"
      @cancel="closeEditor"
    />
  </div>
</template>

<script>
import { ref, onMounted } from 'vue'
import { projectsAPI } from '@/services/api'
import RecordEditorModal from '@/components/RecordEditorModal.vue'

export default {
  name: 'ProjectsSection',
  components: { RecordEditorModal },
  props: { currentUser: Object },
  setup(props) {
    const projects = ref([])
    const loading = ref(false)
    const error = ref(null)
    const editorOpen = ref(false)
    const editingRecord = ref(null)
    const saving = ref(false)
    const editorError = ref('')
    const emptyRecord = { name: '', description: '', url: '', imageUrl: '' }
    const fields = [
      { key: 'name', label: 'Project name' },
      { key: 'description', label: 'Description', type: 'textarea' },
      { key: 'url', label: 'Link', type: 'url', required: false },
      { key: 'imageUrl', label: 'Image URL', type: 'url', required: false }
    ]

    const loadProjects = async () => {
      loading.value = true
      error.value = null
      try {
        const response = await projectsAPI.getAll()
        projects.value = response.data
      } catch (err) {
        error.value = 'Failed to load projects'
        console.error('Error loading projects:', err)
      } finally {
        loading.value = false
      }
    }

    const deleteProject = async (id) => {
      if (!window.confirm('Warning: this will permanently delete this project. Continue?')) return
      try {
        await projectsAPI.delete(id)
        projects.value = projects.value.filter(proj => String(proj.id) !== String(id))
      } catch (err) {
        console.error('Failed to delete project', err)
        window.alert(err.response?.data?.message || 'Failed to delete project')
      }
    }

    const canManage = (project) => {
      const userId = props.currentUser?.id ?? props.currentUser?.userId ?? props.currentUser?.user_id
      const ownerId = project.userId ?? project.user_id ?? project.ownerId
      return userId != null && ownerId != null && String(userId) === String(ownerId)
    }

    const openCreate = () => {
      editingRecord.value = null
      editorError.value = ''
      editorOpen.value = true
    }

    const openEdit = (project) => {
      editingRecord.value = { ...project }
      editorError.value = ''
      editorOpen.value = true
    }

    const closeEditor = () => {
      if (saving.value) return
      editorOpen.value = false
      editingRecord.value = null
    }

    const saveProject = async (form) => {
      saving.value = true
      editorError.value = ''
      try {
        if (editingRecord.value) {
          await projectsAPI.update(editingRecord.value.id, form)
        } else {
          await projectsAPI.create(form)
        }
        await loadProjects()
        editorOpen.value = false
        editingRecord.value = null
      } catch (err) {
        editorError.value = err.response?.data?.message || 'Could not save project. Check that the API is available.'
      } finally {
        saving.value = false
      }
    }

    onMounted(loadProjects)

    return {
      projects, loading, error, deleteProject, canManage,
      editorOpen, editingRecord, saving, editorError, emptyRecord, fields,
      openCreate, openEdit, closeEditor, saveProject
    }
  }
}
</script>
