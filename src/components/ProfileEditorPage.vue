<template>
  <main class="ml-[clamp(350px,35vw,500px)] min-h-screen flex-1 p-8 text-gray-900 dark:text-gray-100">
    <div class="mx-auto max-w-3xl">
      <button
        type="button"
        class="mb-8 text-sm font-medium text-blue-600 hover:underline dark:text-blue-400"
        @click="$emit('close')"
      >← Back to portfolio</button>

      <header class="mb-8">
        <p class="text-sm font-semibold uppercase tracking-widest text-blue-600 dark:text-blue-400">Profile settings</p>
        <h1 class="mt-2 text-3xl font-bold text-gray-900 dark:text-white">Edit profile</h1>
        <p class="mt-2 text-gray-500 dark:text-gray-400">Update the summary shown in the portfolio sidebar.</p>
      </header>

      <div class="rounded-2xl border border-gray-200 bg-white p-6 shadow-sm dark:border-gray-700 dark:bg-gray-800">
        <div v-if="loading" class="py-8 text-center text-gray-500 dark:text-gray-400">Loading profile…</div>
        <form v-else class="flex flex-col gap-6" @submit.prevent="saveSummary">
          <div class="grid gap-4 sm:grid-cols-2">
            <div>
              <label for="profile-name" class="mb-1 block text-sm font-medium text-gray-700 dark:text-gray-300">Name</label>
              <input
                id="profile-name"
                :value="profileName"
                class="w-full rounded-lg border border-gray-200 bg-gray-50 px-3 py-2 text-gray-600 disabled:cursor-not-allowed disabled:opacity-80 dark:border-gray-700 dark:bg-gray-900 dark:text-gray-300"
              />
            </div>
            <div>
              <label for="profile-profession" class="mb-1 block text-sm font-medium text-gray-700 dark:text-gray-300">Profession</label>
              <input
                id="profile-profession"
                :value="profileProfession"
                class="w-full rounded-lg border border-gray-200 bg-gray-50 px-3 py-2 text-gray-600 disabled:cursor-not-allowed disabled:opacity-80 dark:border-gray-700 dark:bg-gray-900 dark:text-gray-300"
              />
            </div>
            <div>
              <label for="profile-username" class="mb-1 block text-sm font-medium text-gray-700 dark:text-gray-300">Username</label>
              <input
                id="profile-username"
                :value="username"
                disabled
                class="w-full rounded-lg border border-gray-200 bg-gray-50 px-3 py-2 text-gray-600 disabled:cursor-not-allowed disabled:opacity-80 dark:border-gray-700 dark:bg-gray-900 dark:text-gray-300"
              />
            </div>
            <div>
              <label for="profile-email" class="mb-1 block text-sm font-medium text-gray-700 dark:text-gray-300">Email</label>
              <input
                id="profile-email"
                type="email"
                :value="email"
                disabled
                class="w-full rounded-lg border border-gray-200 bg-gray-50 px-3 py-2 text-gray-600 disabled:cursor-not-allowed disabled:opacity-80 dark:border-gray-700 dark:bg-gray-900 dark:text-gray-300"
              />
            </div>
          </div>
          <p v-if="profileError" class="-mt-4 text-sm text-amber-700 dark:text-amber-300">{{ profileError }}</p>

          <div>
            <label for="profile-summary" class="mb-1 block text-sm font-medium text-gray-700 dark:text-gray-300">Summary</label>
            <textarea
              id="profile-summary"
              v-model="summary"
              rows="7"
              required
              placeholder="Write a short introduction…"
              class="w-full rounded-lg border border-gray-300 bg-white px-3 py-2 text-gray-900 focus:border-blue-500 focus:outline-none focus:ring-2 focus:ring-blue-500/20 dark:border-gray-600 dark:bg-gray-900 dark:text-white"
            ></textarea>
          </div>

          <p v-if="error" role="alert" class="rounded-lg bg-red-50 px-3 py-2 text-sm text-red-700 dark:bg-red-900/30 dark:text-red-300">{{ error }}</p>
          <p v-if="notice" role="status" class="rounded-lg bg-emerald-50 px-3 py-2 text-sm text-emerald-700 dark:bg-emerald-900/30 dark:text-emerald-300">{{ notice }}</p>

          <div class="flex flex-wrap justify-between gap-3">
            <button
              v-if="hasSavedSummary"
              type="button"
              :disabled="saving || deleting"
              class="rounded-lg border border-red-300 px-4 py-2 text-sm font-medium text-red-700 hover:bg-red-50 disabled:opacity-50 dark:border-red-800 dark:text-red-300 dark:hover:bg-red-900/30"
              @click="deleteSummary"
            >{{ deleting ? 'Deleting…' : 'Delete summary' }}</button>
            <span v-else></span>
            <button
              type="submit"
              :disabled="saving || deleting"
              class="rounded-lg bg-blue-600 px-5 py-2 text-sm font-semibold text-white hover:bg-blue-700 disabled:opacity-50"
            >{{ saving ? 'Saving…' : 'Save summary' }}</button>
          </div>
        </form>
      </div>
    </div>
  </main>
</template>

<script>
import { onMounted, ref } from 'vue'
import { profileAPI, summaryAPI } from '@/services/api'

const unwrap = (data) => data?.data ?? data

const readValue = (data, keys) => {
  let value = unwrap(data)
  if (Array.isArray(value)) value = value[0]
  if (typeof value === 'string') return value
  if (!value || typeof value !== 'object') return ''
  for (const key of keys) {
    if (typeof value[key] === 'string') return value[key]
  }
  return ''
}

const describeRequestError = (error, endpoint, label) => {
  if (error?.response?.status === 404) {
    return `${endpoint} returned 404. Register this route in the backend.`
  }
  return `Could not load ${label}.`
}

export default {
  name: 'ProfileEditorPage',
  props: {
    userId: { type: [String, Number], default: null },
    currentUser: { type: Object, default: () => ({}) },
    initialSummary: { type: String, default: '' }
  },
  emits: ['close', 'summary-updated'],
  setup(props, { emit }) {
    const profileName = ref(props.currentUser.name || props.currentUser.fullName || '')
    const profileProfession = ref(props.currentUser.profession || props.currentUser.title || '')
    const username = ref(props.currentUser.username || '')
    const email = ref(props.currentUser.email || '')
    const summary = ref(props.initialSummary)
    const loading = ref(true)
    const saving = ref(false)
    const deleting = ref(false)
    const hasSavedSummary = ref(false)
    const error = ref('')
    const profileError = ref('')
    const notice = ref('')

    const loadProfile = async () => {
      const results = await Promise.allSettled([
        profileAPI.getProfile().name,
        summaryAPI.getAll()
      ])

      if (results[0].status === 'fulfilled') {
        profileName.value = readValue(results[0].value.data, ['name', 'fullName', 'full_name']) || profileName.value
      } else {
        profileError.value = describeRequestError(results[0].reason, 'GET /api/user/name', 'the profile name')
      }

      if (results[1].status === 'fulfilled') {
        profileProfession.value = readValue(results[1].value.data, ['profession', 'title', 'jobTitle', 'job_title']) || profileProfession.value
      } else {
        profileError.value = [profileError.value, describeRequestError(results[1].reason, 'GET /api/user/profession', 'the profession')].filter(Boolean).join(' ')
      }

      const summaryResult = results[2]
      if (summaryResult?.status === 'fulfilled') {
        const record = unwrap(summaryResult.value.data)
        const summaries = Array.isArray(record) ? record : record ? [record] : []
        const candidate = summaries.find((item) => {
          const ownerId = item?.userId ?? item?.user_id ?? item?.user?.id
          return ownerId != null && props.userId != null && String(ownerId) === String(props.userId)
        }) ?? summaries[0]
        const summaryValue = readValue(candidate, ['summary', 'description', 'text'])
        if (summaryValue) summary.value = summaryValue
        hasSavedSummary.value = Boolean(candidate && (typeof candidate === 'string' || typeof candidate === 'object'))
      } else {
        profileError.value = [profileError.value, 'Could not load the saved summary list.'].filter(Boolean).join(' ')
      }

      loading.value = false
    }

    const saveSummary = async () => {
      saving.value = true
      error.value = ''
      notice.value = ''
      try {
        if (props.userId != null && hasSavedSummary.value) {
          await summaryAPI.updateByUserId(props.userId, { summary: summary.value })
        } else {
          await summaryAPI.create({ summary: summary.value })
        }
        hasSavedSummary.value = true
        emit('summary-updated', summary.value)
        notice.value = 'Summary saved.'
      } catch (err) {
        error.value = err.response?.data?.message || 'Could not save the summary. Please try again.'
      } finally {
        saving.value = false
      }
    }

    const deleteSummary = async () => {
      if (!window.confirm('Warning: this will permanently delete your profile summary. Continue?')) return
      if (props.userId == null) {
        error.value = 'A user ID is required to delete the summary.'
        return
      }
      deleting.value = true
      error.value = ''
      notice.value = ''
      try {
        await summaryAPI.deleteByUserId(props.userId)
        summary.value = ''
        hasSavedSummary.value = false
        emit('summary-updated', '')
        notice.value = 'Summary deleted.'
      } catch (err) {
        error.value = err.response?.data?.message || 'Could not delete the summary. Please try again.'
      } finally {
        deleting.value = false
      }
    }

    onMounted(loadProfile)

    return {
      profileName, profileProfession, username, email, summary, loading, saving, deleting,
      hasSavedSummary, error, profileError, notice, saveSummary, deleteSummary
    }
  }
}
</script>
