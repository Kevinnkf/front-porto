<!-- src/App.vue -->
<template>
  <div id="app">
    <div class="flex min-h-screen bg-white dark:bg-gray-900 transition-colors duration-300">
      <!-- Sidebar -->
      <aside class="fixed h-screen w-[clamp(350px,35vw,500px)] overflow-y-auto border-r border-gray-200 bg-gray-50 p-8 flex flex-col dark:border-gray-700 dark:bg-gray-800">
        <SidebarContent
          :darkMode="darkMode"
          :currentUser="currentUser"
          :profileName="profileName"
          :profileProfession="profileProfession"
          :summary="summary"
          @toggle-dark-mode="toggleDarkMode"
          @open-profile="openProfile"
          @open-login="showLogin = true"
          @logout="handleLogout"
        />
      </aside>

      <!-- Main Content -->
      <main v-if="currentPage === 'portfolio'" class="ml-[clamp(350px,35vw,500px)] flex-1 p-8 text-gray-900 dark:text-gray-100">
        <section id="experience" class="py-16 max-w-3xl border-b border-gray-200 dark:border-gray-700">
          <ExperienceSection :currentUser="currentUser" />
        </section>

        <section id="projects" class="py-16 max-w-3xl border-b border-gray-200 dark:border-gray-700">
          <ProjectsSection :currentUser="currentUser" />
        </section>

        <section id="skills" class="py-16 max-w-3xl">
          <SkillsSection :currentUser="currentUser" />
        </section>
      </main>

      <ProfileEditorPage
        v-else-if="currentPage === 'profile' && currentUser"
        :userId="currentUser.id ?? currentUser.userId ?? currentUser.user_id"
        :currentUser="currentUser"
        :initialSummary="summary"
        @close="closeProfile"
        @summary-updated="summary = $event"
      />
    </div>

    <!-- Login Modal -->
    <LoginModal
      :show="showLogin"
      :darkMode="darkMode"
      @close="showLogin = false"
      @login-success="handleLoginSuccess"
    />
  </div>
</template>

<script>
import { ref, onMounted } from 'vue'
import SidebarContent from '@/components/layout/SidebarContent.vue'
import ExperienceSection from '@/components/sections/ExperienceSection.vue'
import ProjectsSection from '@/components/sections/ProjectSection.vue'
import SkillsSection from '@/components/sections/SkillsSection.vue'
import LoginModal from '@/components/LoginModal.vue'
import ProfileEditorPage from '@/components/ProfileEditorPage.vue'
import { profileAPI, summaryAPI } from '@/services/api'

export default {
  name: 'App',
  components: {
    SidebarContent,
    ExperienceSection,
    ProjectsSection,
    SkillsSection,
    LoginModal,
    ProfileEditorPage
  },
  setup() {
    const darkMode = ref(false)
    const showLogin = ref(false)
    const currentUser = ref(null)
    const currentPage = ref('portfolio')
    const profileName = ref('Kevin Khalfani Fadillah')
    const profileProfession = ref('Software Engineer')
    const summary = ref('I build performant systems that elevate user experience and product scalability. I\'m also actively exploring how AI can power the next generation of products.')

    const getSummaryRecord = (data, userId) => {
      const result = data?.data ?? data
      if (Array.isArray(result) && userId != null) {
        return result.find((item) => String(item.userId ?? item.user_id ?? item.user?.id) === String(userId)) ?? result[0]
      }
      return Array.isArray(result) ? result[0] : result
    }

    const readProfileField = (data, fields) => {
      const value = data?.data ?? data
      if (typeof value === 'string') return value
      if (Array.isArray(value)) return readProfileField(value[0], fields)
      if (!value || typeof value !== 'object') return ''
      for (const field of fields) {
        if (typeof value[field] === 'string') return value[field]
      }
      return ''
    }

    const loadSidebarProfile = async () => {
      const userId = currentUser.value?.id ?? currentUser.value?.userId ?? currentUser.value?.user_id
      const results = await Promise.allSettled([
        profileAPI.getName(),
        profileAPI.getProfession(),
        summaryAPI.getAll()
      ])

      if (results[0].status === 'fulfilled') {
        profileName.value = readProfileField(results[0].value.data, ['name', 'fullName', 'full_name']) || profileName.value
      }
      if (results[1].status === 'fulfilled') {
        profileProfession.value = readProfileField(results[1].value.data, ['profession', 'title', 'jobTitle', 'job_title']) || profileProfession.value
      }
      if (results[2].status === 'fulfilled') {
        const record = getSummaryRecord(results[2].value.data, userId)
        const value = typeof record === 'string' ? record : record?.summary ?? record?.description ?? record?.text
        if (typeof value === 'string') summary.value = value
      }
    }

    const syncRoute = () => {
      currentPage.value = window.location.hash.includes('profile') && currentUser.value
        ? 'profile'
        : 'portfolio'
    }

    const openProfile = () => {
      window.location.hash = '/profile'
    }

    const closeProfile = () => {
      window.location.hash = '/'
    }

    onMounted(() => {
      const storedUser = localStorage.getItem('user')
      if (storedUser) {
        currentUser.value = JSON.parse(storedUser)
      }
      syncRoute()
      window.addEventListener('hashchange', syncRoute)
      loadSidebarProfile()
      // Apply persisted dark mode
      const savedDark = localStorage.getItem('darkMode')
      if (savedDark === 'true') {
        darkMode.value = true
        document.documentElement.classList.add('dark')
      }
    })

    const toggleDarkMode = () => {
      darkMode.value = !darkMode.value
      if (darkMode.value) {
        document.documentElement.classList.add('dark')
      } else {
        document.documentElement.classList.remove('dark')
      }
      localStorage.setItem('darkMode', darkMode.value)
    }

    const handleLoginSuccess = (user) => {
      currentUser.value = user
      showLogin.value = false
      loadSidebarProfile()
    }

    const handleLogout = () => {
      localStorage.removeItem('token')
      localStorage.removeItem('user')
      currentUser.value = null
      closeProfile()
    }

    return {
      darkMode,
      toggleDarkMode,
      showLogin,
      currentUser,
      currentPage,
      profileName,
      profileProfession,
      summary,
      openProfile,
      closeProfile,
      handleLoginSuccess,
      handleLogout
    }
  }
}
</script>
