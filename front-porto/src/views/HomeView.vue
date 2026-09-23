<!-- src/views/HomeView.vue -->
<template>
  <div class="min-h-screen bg-white dark:bg-gray-900 transition-colors duration-300">

    <!-- Mobile Top Navigation Header (< lg) -->
    <header class="lg:hidden sticky top-0 z-40 bg-white/90 dark:bg-gray-900/90 backdrop-blur-md border-b border-gray-200 dark:border-gray-800 px-4 py-3 flex items-center justify-between">
      <div>
        <h1 class="text-base font-bold text-gray-900 dark:text-white leading-tight">Kevin Khalfani</h1>
        <p class="text-xs text-gray-500 dark:text-gray-400">Software Engineer</p>
      </div>

      <div class="flex items-center gap-2">
        <router-link
          v-if="isAdmin"
          to="/admin"
          class="px-2.5 py-1 text-xs font-semibold rounded-lg bg-indigo-600 text-white hover:bg-indigo-700 transition-colors"
        >
          Admin
        </router-link>

        <button
          v-if="!currentUser"
          @click="$emit('open-login')"
          class="px-2.5 py-1 text-xs font-semibold rounded-lg bg-blue-600 text-white hover:bg-blue-700 transition-colors"
        >
          Login
        </button>

        <button
          v-else
          @click="$emit('logout')"
          class="px-2 py-1 text-xs border border-gray-300 dark:border-gray-600 text-gray-600 dark:text-gray-300 rounded-lg hover:bg-red-50 dark:hover:bg-red-900/40"
          title="Logout"
        >
          Logout
        </button>

        <!-- Theme Toggle -->
        <button
          @click="$emit('toggle-dark-mode')"
          class="w-8 h-8 flex items-center justify-center rounded-lg text-sm bg-gray-100 dark:bg-gray-800 text-gray-700 dark:text-gray-200"
          aria-label="Toggle dark mode"
        >
          {{ darkMode ? '☀️' : '🌙' }}
        </button>
      </div>
    </header>

    <!-- Mobile Quick Navigation Bar (< lg) -->
    <nav class="lg:hidden sticky top-[57px] z-30 bg-gray-50/95 dark:bg-gray-850/95 backdrop-blur-sm border-b border-gray-200/80 dark:border-gray-800/80 px-3 py-2 overflow-x-auto flex items-center gap-2 scrollbar-none">
      <button
        v-for="section in navSections"
        :key="section.id"
        @click="scrollTo(section.id)"
        class="px-3 py-1 rounded-full text-xs font-semibold whitespace-nowrap transition-all duration-200"
        :class="activeSection === section.id
          ? 'bg-blue-600 text-white shadow-sm'
          : 'bg-white dark:bg-gray-850 text-gray-600 dark:text-gray-350 border border-gray-200 dark:border-gray-700'"
      >
        {{ section.label }}
      </button>
    </nav>

    <!-- Layout Container -->
    <div class="max-w-7xl mx-auto flex flex-col lg:flex-row justify-between lg:gap-12 px-4 sm:px-8 lg:px-12">

      <!-- Desktop Sidebar (>= lg) -->
      <aside class="hidden lg:flex lg:flex-col w-[38%] max-w-[460px] sticky top-0 h-screen py-16 justify-between shrink-0">
        <SidebarContent
          :darkMode="darkMode"
          :currentUser="currentUser"
          @toggle-dark-mode="$emit('toggle-dark-mode')"
          @open-login="$emit('open-login')"
          @logout="$emit('logout')"
        />
      </aside>

      <!-- Main Content Area -->
      <main class="w-full lg:w-[62%] py-8 sm:py-12 lg:py-16 flex flex-col gap-14 sm:gap-20 text-gray-900 dark:text-gray-100">

        <!-- 1. Overview Section -->
        <section id="overview" class="scroll-mt-28">
          <OverviewSection
            :currentUser="currentUser"
            @edit-overview="openEditOverview"
            ref="overviewRef"
          />
        </section>

        <!-- 2. Experience Section -->
        <section id="experience" class="scroll-mt-28 border-t border-gray-100 dark:border-gray-800/80 pt-12">
          <ExperienceSection
            :currentUser="currentUser"
            @add-experience="openAddModal('experience')"
            @edit-experience="openEditModal('experience', $event)"
            ref="experienceRef"
          />
        </section>

        <!-- 3. Projects Section -->
        <section id="projects" class="scroll-mt-28 border-t border-gray-100 dark:border-gray-800/80 pt-12">
          <ProjectsSection
            :currentUser="currentUser"
            @add-project="openAddModal('project')"
            @edit-project="openEditModal('project', $event)"
            ref="projectsRef"
          />
        </section>

        <!-- 4. Skills Section -->
        <section id="skills" class="scroll-mt-28 border-t border-gray-100 dark:border-gray-800/80 pt-12">
          <SkillsSection
            :currentUser="currentUser"
            @add-skill="openAddModal('skill')"
            @edit-skill="openEditModal('skill', $event)"
            ref="skillsRef"
          />
        </section>

        <!-- 5. Certifications Section -->
        <section id="certifications" class="scroll-mt-28 border-t border-gray-100 dark:border-gray-800/80 pt-12 pb-16">
          <CertificationsSection
            :currentUser="currentUser"
            @add-certification="openAddModal('certification')"
            @edit-certification="openEditModal('certification', $event)"
            ref="certificationsRef"
          />
        </section>

        <!-- Footer for Mobile (< lg) -->
        <footer class="lg:hidden border-t border-gray-200 dark:border-gray-800 pt-8 pb-12 flex flex-col items-center gap-4 text-center text-xs text-gray-500 dark:text-gray-400">
          <p>© {{ new Date().getFullYear() }} Kevin Khalfani Fadillah. Built with Vue 3 & Express.</p>
          <div class="flex items-center gap-4">
            <a href="https://github.com/Kevinnkf" target="_blank" rel="noopener noreferrer" class="hover:text-blue-600">GitHub</a>
            <span>•</span>
            <a href="https://linkedin.com/in/kevin-khalfani-f" target="_blank" rel="noopener noreferrer" class="hover:text-blue-600">LinkedIn</a>
            <span>•</span>
            <a href="mailto:khalfanifadillah@gmail.com" class="hover:text-blue-600">Email</a>
          </div>
        </footer>

      </main>
    </div>

    <!-- Dynamic Item Management Modal for Admin -->
    <ItemModal
      :show="showItemModal"
      :itemType="modalItemType"
      :itemData="modalItemData"
      @close="showItemModal = false"
      @saved="handleItemSaved"
    />

  </div>
</template>

<script>
import { ref, computed, onMounted, onUnmounted } from 'vue'
import SidebarContent from '@/components/layout/SidebarContent.vue'
import OverviewSection from '@/components/sections/OverviewSection.vue'
import ExperienceSection from '@/components/sections/ExperienceSection.vue'
import ProjectsSection from '@/components/sections/ProjectSection.vue'
import SkillsSection from '@/components/sections/SkillsSection.vue'
import CertificationsSection from '@/components/sections/CertificationsSection.vue'
import ItemModal from '@/components/ItemModal.vue'

export default {
  name: 'HomeView',
  components: {
    SidebarContent,
    OverviewSection,
    ExperienceSection,
    ProjectsSection,
    SkillsSection,
    CertificationsSection,
    ItemModal
  },
  props: {
    darkMode: Boolean,
    currentUser: Object
  },
  emits: ['toggle-dark-mode', 'open-login', 'logout'],
  setup(props) {
    const activeSection = ref('overview')
    const showItemModal = ref(false)
    const modalItemType = ref('project')
    const modalItemData = ref(null)

    // Component references for reloading
    const overviewRef = ref(null)
    const experienceRef = ref(null)
    const projectsRef = ref(null)
    const skillsRef = ref(null)
    const certificationsRef = ref(null)

    const navSections = [
      { id: 'overview',       label: 'Overview' },
      { id: 'experience',     label: 'Experience' },
      { id: 'projects',       label: 'Projects' },
      { id: 'skills',         label: 'Skills' },
      { id: 'certifications', label: 'Certifications' }
    ]

    const isAdmin = computed(() => props.currentUser?.role === 'admin')

    const scrollTo = (id) => {
      const el = document.getElementById(id)
      if (el) {
        el.scrollIntoView({ behavior: 'smooth' })
        activeSection.value = id
      }
    }

    const openAddModal = (type) => {
      modalItemType.value = type
      modalItemData.value = null
      showItemModal.value = true
    }

    const openEditModal = (type, item) => {
      modalItemType.value = type
      modalItemData.value = item
      showItemModal.value = true
    }

    const openEditOverview = (summaryItem) => {
      modalItemType.value = 'overview'
      modalItemData.value = summaryItem || { content: '' }
      showItemModal.value = true
    }

    const handleItemSaved = () => {
      if (modalItemType.value === 'overview' && overviewRef.value?.fetchSummary) {
        overviewRef.value.fetchSummary()
      } else if (modalItemType.value === 'experience' && experienceRef.value?.loadExperiences) {
        experienceRef.value.loadExperiences()
      } else if (modalItemType.value === 'project' && projectsRef.value?.loadProjects) {
        projectsRef.value.loadProjects()
      } else if (modalItemType.value === 'skill' && skillsRef.value?.loadSkills) {
        skillsRef.value.loadSkills()
      } else if (modalItemType.value === 'certification' && certificationsRef.value?.loadCertifications) {
        certificationsRef.value.loadCertifications()
      }
    }

    const handleScroll = () => {
      const ids = ['overview', 'experience', 'projects', 'skills', 'certifications']
      const pos = window.scrollY + 140
      for (const id of ids) {
        const el = document.getElementById(id)
        if (el) {
          const top = el.offsetTop
          const height = el.offsetHeight
          if (pos >= top && pos < top + height) {
            activeSection.value = id
            break
          }
        }
      }
    }

    onMounted(() => window.addEventListener('scroll', handleScroll))
    onUnmounted(() => window.removeEventListener('scroll', handleScroll))

    return {
      activeSection,
      navSections,
      isAdmin,
      scrollTo,
      showItemModal,
      modalItemType,
      modalItemData,
      openAddModal,
      openEditModal,
      openEditOverview,
      handleItemSaved,
      overviewRef,
      experienceRef,
      projectsRef,
      skillsRef,
      certificationsRef
    }
  }
}
</script>
