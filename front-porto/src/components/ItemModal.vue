<!-- src/components/ItemModal.vue -->
<template>
  <Teleport to="body">
    <div
      v-if="show"
      class="fixed inset-0 z-50 flex items-center justify-center bg-black/60 backdrop-blur-sm p-4 overflow-y-auto"
      @click.self="$emit('close')"
    >
      <div class="bg-white dark:bg-gray-800 rounded-2xl shadow-2xl w-full max-w-xl my-8 overflow-hidden border border-gray-200 dark:border-gray-700 transition-colors duration-200">
        <!-- Modal Header -->
        <div class="px-6 py-5 border-b border-gray-100 dark:border-gray-700 flex items-center justify-between">
          <div>
            <h3 class="text-xl font-bold text-gray-900 dark:text-white">
              {{ isEditing ? 'Edit ' + titleLabel : 'Add New ' + titleLabel }}
            </h3>
            <p class="text-xs text-gray-500 dark:text-gray-400 mt-0.5">Fill in the details below</p>
          </div>
          <button
            @click="$emit('close')"
            class="w-8 h-8 rounded-full flex items-center justify-center text-gray-400 hover:text-gray-600 dark:hover:text-gray-200 hover:bg-gray-100 dark:hover:bg-gray-700 transition-colors"
          >
            ✕
          </button>
        </div>

        <!-- Form Content -->
        <form @submit.prevent="handleSubmit" class="p-6 space-y-4 max-h-[75vh] overflow-y-auto">

          <!-- OVERVIEW FORM -->
          <template v-if="itemType === 'overview'">
            <div>
              <label class="block text-xs font-semibold uppercase tracking-wider text-gray-600 dark:text-gray-300 mb-1.5">
                Bio / Summary Content
              </label>
              <textarea
                v-model="formData.content"
                rows="8"
                required
                class="w-full px-3.5 py-2.5 text-sm rounded-xl border border-gray-300 dark:border-gray-600 bg-white dark:bg-gray-700 text-gray-900 dark:text-white placeholder-gray-400 focus:ring-2 focus:ring-blue-500 focus:outline-none"
                placeholder="Write an engaging overview about your background, strengths, and current focus..."
              ></textarea>
            </div>
          </template>

          <!-- EXPERIENCE FORM -->
          <template v-else-if="itemType === 'experience'">
            <div class="grid grid-cols-1 sm:grid-cols-2 gap-4">
              <div>
                <label class="block text-xs font-semibold uppercase tracking-wider text-gray-600 dark:text-gray-300 mb-1.5">Company *</label>
                <input
                  v-model="formData.company"
                  type="text"
                  required
                  class="w-full px-3.5 py-2.5 text-sm rounded-xl border border-gray-300 dark:border-gray-600 bg-white dark:bg-gray-700 text-gray-900 dark:text-white focus:ring-2 focus:ring-blue-500 focus:outline-none"
                  placeholder="e.g. Google, Stripe"
                />
              </div>
              <div>
                <label class="block text-xs font-semibold uppercase tracking-wider text-gray-600 dark:text-gray-300 mb-1.5">Role / Position *</label>
                <input
                  v-model="formData.role"
                  type="text"
                  required
                  class="w-full px-3.5 py-2.5 text-sm rounded-xl border border-gray-300 dark:border-gray-600 bg-white dark:bg-gray-700 text-gray-900 dark:text-white focus:ring-2 focus:ring-blue-500 focus:outline-none"
                  placeholder="e.g. Senior Software Engineer"
                />
              </div>
            </div>

            <div class="grid grid-cols-1 sm:grid-cols-2 gap-4">
              <div>
                <label class="block text-xs font-semibold uppercase tracking-wider text-gray-600 dark:text-gray-300 mb-1.5">Start Date (YYYY-MM-DD) *</label>
                <input
                  v-model="formData.startDate"
                  type="date"
                  required
                  class="w-full px-3.5 py-2.5 text-sm rounded-xl border border-gray-300 dark:border-gray-600 bg-white dark:bg-gray-700 text-gray-900 dark:text-white focus:ring-2 focus:ring-blue-500 focus:outline-none"
                />
              </div>
              <div>
                <label class="block text-xs font-semibold uppercase tracking-wider text-gray-600 dark:text-gray-300 mb-1.5">End Date (Leave blank if Present)</label>
                <input
                  v-model="formData.endDate"
                  type="date"
                  class="w-full px-3.5 py-2.5 text-sm rounded-xl border border-gray-300 dark:border-gray-600 bg-white dark:bg-gray-700 text-gray-900 dark:text-white focus:ring-2 focus:ring-blue-500 focus:outline-none"
                />
              </div>
            </div>

            <div>
              <label class="block text-xs font-semibold uppercase tracking-wider text-gray-600 dark:text-gray-300 mb-1.5">Description</label>
              <textarea
                v-model="formData.description"
                rows="4"
                class="w-full px-3.5 py-2.5 text-sm rounded-xl border border-gray-300 dark:border-gray-600 bg-white dark:bg-gray-700 text-gray-900 dark:text-white focus:ring-2 focus:ring-blue-500 focus:outline-none"
                placeholder="Key achievements, technologies used, responsibilities..."
              ></textarea>
            </div>
          </template>

          <!-- PROJECT FORM -->
          <template v-else-if="itemType === 'project'">
            <div>
              <label class="block text-xs font-semibold uppercase tracking-wider text-gray-600 dark:text-gray-300 mb-1.5">Project Name *</label>
              <input
                v-model="formData.name"
                type="text"
                required
                class="w-full px-3.5 py-2.5 text-sm rounded-xl border border-gray-300 dark:border-gray-600 bg-white dark:bg-gray-700 text-gray-900 dark:text-white focus:ring-2 focus:ring-blue-500 focus:outline-none"
                placeholder="e.g. Distributed Task Queue"
              />
            </div>

            <div>
              <label class="block text-xs font-semibold uppercase tracking-wider text-gray-600 dark:text-gray-300 mb-1.5">Project URL</label>
              <input
                v-model="formData.url"
                type="url"
                class="w-full px-3.5 py-2.5 text-sm rounded-xl border border-gray-300 dark:border-gray-600 bg-white dark:bg-gray-700 text-gray-900 dark:text-white focus:ring-2 focus:ring-blue-500 focus:outline-none"
                placeholder="https://github.com/... or live demo URL"
              />
            </div>

            <div>
              <label class="block text-xs font-semibold uppercase tracking-wider text-gray-600 dark:text-gray-300 mb-1.5">Image URL</label>
              <input
                v-model="formData.imageUrl"
                type="url"
                class="w-full px-3.5 py-2.5 text-sm rounded-xl border border-gray-300 dark:border-gray-600 bg-white dark:bg-gray-700 text-gray-900 dark:text-white focus:ring-2 focus:ring-blue-500 focus:outline-none"
                placeholder="https://images.unsplash.com/... or preview image link"
              />
            </div>

            <div>
              <label class="block text-xs font-semibold uppercase tracking-wider text-gray-600 dark:text-gray-300 mb-1.5">Description</label>
              <textarea
                v-model="formData.description"
                rows="4"
                class="w-full px-3.5 py-2.5 text-sm rounded-xl border border-gray-300 dark:border-gray-600 bg-white dark:bg-gray-700 text-gray-900 dark:text-white focus:ring-2 focus:ring-blue-500 focus:outline-none"
                placeholder="What problem does it solve? Architecture, technologies used, metrics..."
              ></textarea>
            </div>
          </template>

          <!-- SKILL FORM -->
          <template v-else-if="itemType === 'skill'">
            <div>
              <label class="block text-xs font-semibold uppercase tracking-wider text-gray-600 dark:text-gray-300 mb-1.5">Skill Name *</label>
              <input
                v-model="formData.name"
                type="text"
                required
                class="w-full px-3.5 py-2.5 text-sm rounded-xl border border-gray-300 dark:border-gray-600 bg-white dark:bg-gray-700 text-gray-900 dark:text-white focus:ring-2 focus:ring-blue-500 focus:outline-none"
                placeholder="e.g. Node.js, PostgreSQL, Vue.js, Go, Docker"
              />
            </div>

            <div>
              <label class="block text-xs font-semibold uppercase tracking-wider text-gray-600 dark:text-gray-300 mb-1.5">URL (Documentation / Repo)</label>
              <input
                v-model="formData.url"
                type="url"
                class="w-full px-3.5 py-2.5 text-sm rounded-xl border border-gray-300 dark:border-gray-600 bg-white dark:bg-gray-700 text-gray-900 dark:text-white focus:ring-2 focus:ring-blue-500 focus:outline-none"
                placeholder="https://..."
              />
            </div>

            <div>
              <label class="block text-xs font-semibold uppercase tracking-wider text-gray-600 dark:text-gray-300 mb-1.5">Description / Proficiency Context</label>
              <textarea
                v-model="formData.description"
                rows="3"
                class="w-full px-3.5 py-2.5 text-sm rounded-xl border border-gray-300 dark:border-gray-600 bg-white dark:bg-gray-700 text-gray-900 dark:text-white focus:ring-2 focus:ring-blue-500 focus:outline-none"
                placeholder="Years of experience, production use cases, favorite tools..."
              ></textarea>
            </div>
          </template>

          <!-- CERTIFICATION FORM -->
          <template v-else-if="itemType === 'certification'">
            <div class="grid grid-cols-1 sm:grid-cols-2 gap-4">
              <div>
                <label class="block text-xs font-semibold uppercase tracking-wider text-gray-600 dark:text-gray-300 mb-1.5">Title *</label>
                <input
                  v-model="formData.title"
                  type="text"
                  required
                  class="w-full px-3.5 py-2.5 text-sm rounded-xl border border-gray-300 dark:border-gray-600 bg-white dark:bg-gray-700 text-gray-900 dark:text-white focus:ring-2 focus:ring-blue-500 focus:outline-none"
                  placeholder="e.g. AWS Certified Solutions Architect"
                />
              </div>
              <div>
                <label class="block text-xs font-semibold uppercase tracking-wider text-gray-600 dark:text-gray-300 mb-1.5">Issuer *</label>
                <input
                  v-model="formData.issuer"
                  type="text"
                  required
                  class="w-full px-3.5 py-2.5 text-sm rounded-xl border border-gray-300 dark:border-gray-600 bg-white dark:bg-gray-700 text-gray-900 dark:text-white focus:ring-2 focus:ring-blue-500 focus:outline-none"
                  placeholder="e.g. Amazon Web Services, Cisco, HashiCorp"
                />
              </div>
            </div>

            <div class="grid grid-cols-1 sm:grid-cols-2 gap-4">
              <div>
                <label class="block text-xs font-semibold uppercase tracking-wider text-gray-600 dark:text-gray-300 mb-1.5">Issue Date</label>
                <input
                  v-model="formData.issueDate"
                  type="text"
                  class="w-full px-3.5 py-2.5 text-sm rounded-xl border border-gray-300 dark:border-gray-600 bg-white dark:bg-gray-700 text-gray-900 dark:text-white focus:ring-2 focus:ring-blue-500 focus:outline-none"
                  placeholder="e.g. Jan 2024"
                />
              </div>
              <div>
                <label class="block text-xs font-semibold uppercase tracking-wider text-gray-600 dark:text-gray-300 mb-1.5">Expiration Date</label>
                <input
                  v-model="formData.expirationDate"
                  type="text"
                  class="w-full px-3.5 py-2.5 text-sm rounded-xl border border-gray-300 dark:border-gray-600 bg-white dark:bg-gray-700 text-gray-900 dark:text-white focus:ring-2 focus:ring-blue-500 focus:outline-none"
                  placeholder="e.g. Jan 2027 or No Expiration"
                />
              </div>
            </div>

            <div class="grid grid-cols-1 sm:grid-cols-2 gap-4">
              <div>
                <label class="block text-xs font-semibold uppercase tracking-wider text-gray-600 dark:text-gray-300 mb-1.5">Credential ID</label>
                <input
                  v-model="formData.credentialId"
                  type="text"
                  class="w-full px-3.5 py-2.5 text-sm rounded-xl border border-gray-300 dark:border-gray-600 bg-white dark:bg-gray-700 text-gray-900 dark:text-white focus:ring-2 focus:ring-blue-500 focus:outline-none"
                  placeholder="e.g. ABC-12345-XYZ"
                />
              </div>
              <div>
                <label class="block text-xs font-semibold uppercase tracking-wider text-gray-600 dark:text-gray-300 mb-1.5">Credential Verification URL</label>
                <input
                  v-model="formData.credentialUrl"
                  type="url"
                  class="w-full px-3.5 py-2.5 text-sm rounded-xl border border-gray-300 dark:border-gray-600 bg-white dark:bg-gray-700 text-gray-900 dark:text-white focus:ring-2 focus:ring-blue-500 focus:outline-none"
                  placeholder="https://credly.com/..."
                />
              </div>
            </div>

            <div>
              <label class="block text-xs font-semibold uppercase tracking-wider text-gray-600 dark:text-gray-300 mb-1.5">Description / Skills Covered</label>
              <textarea
                v-model="formData.description"
                rows="3"
                class="w-full px-3.5 py-2.5 text-sm rounded-xl border border-gray-300 dark:border-gray-600 bg-white dark:bg-gray-700 text-gray-900 dark:text-white focus:ring-2 focus:ring-blue-500 focus:outline-none"
                placeholder="Cloud architecture, VPC design, IAM policies, High Availability..."
              ></textarea>
            </div>
          </template>

          <!-- Error Alert -->
          <div v-if="error" class="text-sm text-red-600 dark:text-red-400 bg-red-50 dark:bg-red-900/30 border border-red-200 dark:border-red-800 rounded-xl px-4 py-2.5">
            {{ error }}
          </div>

          <!-- Modal Actions -->
          <div class="pt-4 flex items-center justify-end gap-3 border-t border-gray-100 dark:border-gray-700">
            <button
              type="button"
              @click="$emit('close')"
              class="px-4 py-2.5 text-sm font-medium text-gray-600 dark:text-gray-300 hover:bg-gray-100 dark:hover:bg-gray-700 rounded-xl transition-colors"
            >
              Cancel
            </button>
            <button
              type="submit"
              :disabled="loading"
              class="px-5 py-2.5 text-sm font-semibold text-white bg-blue-600 hover:bg-blue-700 dark:bg-blue-500 dark:hover:bg-blue-600 rounded-xl transition-colors flex items-center gap-2 shadow-sm disabled:opacity-50"
            >
              <svg v-if="loading" class="animate-spin h-4 w-4 text-white" fill="none" viewBox="0 0 24 24">
                <circle class="opacity-25" cx="12" cy="12" r="10" stroke="currentColor" stroke-width="4"></circle>
                <path class="opacity-75" fill="currentColor" d="M4 12a8 8 0 018-8V0C5.373 0 0 5.373 0 12h4z"></path>
              </svg>
              {{ isEditing ? 'Save Changes' : 'Create Item' }}
            </button>
          </div>
        </form>
      </div>
    </div>
  </Teleport>
</template>

<script>
import { ref, watch, computed } from 'vue'
import {
  experiencesAPI,
  projectsAPI,
  skillsAPI,
  certificationsAPI,
  summaryAPI
} from '@/services/api'

export default {
  name: 'ItemModal',
  props: {
    show: Boolean,
    itemType: {
      type: String,
      required: true // 'overview', 'experience', 'project', 'skill', 'certification'
    },
    itemData: {
      type: Object,
      default: null
    }
  },
  emits: ['close', 'saved'],
  setup(props, { emit }) {
    const loading = ref(false)
    const error = ref('')
    const formData = ref({})

    const isEditing = computed(() => !!props.itemData?.id)

    const titleLabel = computed(() => {
      switch (props.itemType) {
        case 'overview': return 'Overview / Bio'
        case 'experience': return 'Experience'
        case 'project': return 'Project'
        case 'skill': return 'Skill'
        case 'certification': return 'Certification'
        default: return 'Item'
      }
    })

    const resetForm = () => {
      error.value = ''
      if (props.itemData) {
        formData.value = { ...props.itemData }
      } else {
        formData.value = {}
      }
    }

    watch(() => props.show, (newVal) => {
      if (newVal) resetForm()
    })

    watch(() => props.itemData, () => {
      resetForm()
    })

    const handleSubmit = async () => {
      loading.value = true
      error.value = ''
      try {
        if (props.itemType === 'overview') {
          if (isEditing.value) {
            await summaryAPI.update(props.itemData.id, { content: formData.value.content })
          } else {
            await summaryAPI.create({ content: formData.value.content })
          }
        } else if (props.itemType === 'experience') {
          if (isEditing.value) {
            await experiencesAPI.update(props.itemData.id, formData.value)
          } else {
            await experiencesAPI.create(formData.value)
          }
        } else if (props.itemType === 'project') {
          if (isEditing.value) {
            await projectsAPI.update(props.itemData.id, formData.value)
          } else {
            await projectsAPI.create(formData.value)
          }
        } else if (props.itemType === 'skill') {
          if (isEditing.value) {
            await skillsAPI.update(props.itemData.id, formData.value)
          } else {
            await skillsAPI.create(formData.value)
          }
        } else if (props.itemType === 'certification') {
          if (isEditing.value) {
            await certificationsAPI.update(props.itemData.id, formData.value)
          } else {
            await certificationsAPI.create(formData.value)
          }
        }

        emit('saved')
        emit('close')
      } catch (err) {
        error.value = err.response?.data?.message || 'Failed to save item. Please check inputs.'
      } finally {
        loading.value = false
      }
    }

    return {
      formData,
      loading,
      error,
      isEditing,
      titleLabel,
      handleSubmit
    }
  }
}
</script>
