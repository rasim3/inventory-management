<template>
  <div class="app">
    <div class="app-shell">
      <AppSidebar :mobile-open="mobileNavOpen" @close="mobileNavOpen = false" />
      <div class="app-main" :inert="mobileNavOpen || undefined">
        <header class="topbar">
          <div class="topbar-left">
            <button
              type="button"
              ref="menuButton"
              class="menu-button"
              :aria-expanded="mobileNavOpen"
              aria-controls="app-sidebar"
              :aria-label="t('nav.menu')"
              @click="mobileNavOpen = true"
            >
              <svg width="20" height="20" viewBox="0 0 20 20" fill="none" stroke="currentColor" stroke-width="1.5" stroke-linecap="round" aria-hidden="true">
                <path d="M3 5.5h14M3 10h14M3 14.5h14" />
              </svg>
            </button>
            <h1 class="page-title">{{ pageTitle }}</h1>
          </div>
          <div class="topbar-right">
            <LanguageSwitcher />
            <ProfileMenu
              @show-profile-details="showProfileDetails = true"
              @show-tasks="showTasks = true"
            />
          </div>
        </header>
        <FilterBar />
        <main class="main-content">
          <router-view />
        </main>
      </div>
    </div>

    <ProfileDetailsModal
      :is-open="showProfileDetails"
      @close="showProfileDetails = false"
    />

    <TasksModal
      :is-open="showTasks"
      :tasks="tasks"
      @close="showTasks = false"
      @add-task="addTask"
      @delete-task="deleteTask"
      @toggle-task="toggleTask"
    />
  </div>
</template>

<script>
import { ref, watch, nextTick, onMounted, computed } from 'vue'
import { useRoute } from 'vue-router'
import { api } from './api'
import { useAuth } from './composables/useAuth'
import { useI18n } from './composables/useI18n'
import FilterBar from './components/FilterBar.vue'
import ProfileMenu from './components/ProfileMenu.vue'
import ProfileDetailsModal from './components/ProfileDetailsModal.vue'
import TasksModal from './components/TasksModal.vue'
import LanguageSwitcher from './components/LanguageSwitcher.vue'
import AppSidebar from './components/AppSidebar.vue'

export default {
  name: 'App',
  components: {
    FilterBar,
    ProfileMenu,
    ProfileDetailsModal,
    TasksModal,
    LanguageSwitcher,
    AppSidebar
  },
  setup() {
    const { currentUser } = useAuth()
    const { t } = useI18n()
    const showProfileDetails = ref(false)
    const showTasks = ref(false)
    const apiTasks = ref([])
    const route = useRoute()
    const mobileNavOpen = ref(false)
    const menuButton = ref(null)

    watch(mobileNavOpen, async (open, wasOpen) => {
      if (wasOpen && !open) {
        await nextTick()
        if (menuButton.value && menuButton.value.offsetParent !== null) menuButton.value.focus()
      }
    })

    const titleKeys = {
      '/': 'nav.overview',
      '/inventory': 'nav.inventory',
      '/orders': 'nav.orders',
      '/spending': 'nav.finance',
      '/demand': 'nav.demandForecast',
      '/reports': 'nav.reports'
    }
    const pageTitle = computed(() => {
      const key = titleKeys[route.path]
      return key ? t(key) : t('nav.companyName')
    })

    // Merge mock tasks from currentUser with API tasks
    const tasks = computed(() => {
      return [...currentUser.value.tasks, ...apiTasks.value]
    })

    const loadTasks = async () => {
      try {
        apiTasks.value = await api.getTasks()
      } catch (err) {
        console.error('Failed to load tasks:', err)
      }
    }

    const addTask = async (taskData) => {
      try {
        const newTask = await api.createTask(taskData)
        // Add new task to the beginning of the array
        apiTasks.value.unshift(newTask)
      } catch (err) {
        console.error('Failed to add task:', err)
      }
    }

    const deleteTask = async (taskId) => {
      try {
        // Check if it's a mock task (from currentUser)
        const isMockTask = currentUser.value.tasks.some(t => t.id === taskId)

        if (isMockTask) {
          // Remove from mock tasks
          const index = currentUser.value.tasks.findIndex(t => t.id === taskId)
          if (index !== -1) {
            currentUser.value.tasks.splice(index, 1)
          }
        } else {
          // Remove from API tasks
          await api.deleteTask(taskId)
          apiTasks.value = apiTasks.value.filter(t => t.id !== taskId)
        }
      } catch (err) {
        console.error('Failed to delete task:', err)
      }
    }

    const toggleTask = async (taskId) => {
      try {
        // Check if it's a mock task (from currentUser)
        const mockTask = currentUser.value.tasks.find(t => t.id === taskId)

        if (mockTask) {
          // Toggle mock task status
          mockTask.status = mockTask.status === 'pending' ? 'completed' : 'pending'
        } else {
          // Toggle API task
          const updatedTask = await api.toggleTask(taskId)
          const index = apiTasks.value.findIndex(t => t.id === taskId)
          if (index !== -1) {
            apiTasks.value[index] = updatedTask
          }
        }
      } catch (err) {
        console.error('Failed to toggle task:', err)
      }
    }

    onMounted(loadTasks)

    return {
      t,
      mobileNavOpen,
      menuButton,
      pageTitle,
      showProfileDetails,
      showTasks,
      tasks,
      addTask,
      deleteTask,
      toggleTask
    }
  }
}
</script>

<style>
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

body {
  font-family: var(--font-sans);
  font-size: var(--text-base);
  line-height: var(--leading-normal);
  background: var(--paper);
  color: var(--ink);
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
}

.app {
  min-height: 100vh;
}

.app-shell {
  display: grid;
  grid-template-columns: auto minmax(0, 1fr);
  min-height: 100vh;
}

.app-main {
  display: flex;
  flex-direction: column;
  min-width: 0;
}

.topbar {
  position: sticky;
  top: 0;
  z-index: 100;
  height: var(--topbar-h);
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: var(--space-4);
  padding: 0 var(--space-8);
  background: var(--surface);
  border-bottom: 1px solid var(--rule);
}

.topbar-left,
.topbar-right {
  display: flex;
  align-items: center;
  gap: var(--space-3);
  min-width: 0;
}

.topbar-left {
  flex: 1 1 auto;
}

.topbar-right {
  flex: 0 0 auto;
}

.page-title {
  font-size: var(--text-lg);
  font-weight: var(--weight-semibold);
  color: var(--ink);
  letter-spacing: -0.01em;
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
}

.menu-button {
  display: none;
  align-items: center;
  justify-content: center;
  width: var(--control-h);
  height: var(--control-h);
  background: var(--surface);
  border: 1px solid var(--rule);
  border-radius: var(--radius-control);
  color: var(--ink-2);
  cursor: pointer;
}

.menu-button:hover {
  background: var(--surface-sunken);
}

.main-content {
  flex: 1;
  width: 100%;
  max-width: var(--content-max);
  padding: var(--space-6) var(--space-8);
}

@media (max-width: 767.98px) {
  .menu-button {
    display: inline-flex;
  }

  .topbar {
    padding: 0 var(--space-4);
  }
}

@media (max-width: 639px) {
  .main-content {
    padding: var(--space-4);
  }
}

.page-header {
  margin-bottom: var(--space-6);
}

.page-header h2 {
  font-size: var(--text-xl);
  font-weight: var(--weight-semibold);
  color: var(--ink);
  letter-spacing: -0.01em;
  margin-bottom: var(--space-1);
}

.page-header p {
  color: var(--muted);
  font-size: var(--text-base);
}

.stats-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(220px, 1fr));
  gap: 0;
  margin-bottom: var(--space-6);
  background: var(--surface);
  border: 1px solid var(--rule);
  border-radius: var(--radius-panel);
  overflow: hidden;
}

.stat-card {
  padding: var(--space-5);
  box-shadow: -1px 0 0 var(--rule);
}

.stat-label {
  color: var(--muted);
  font-size: var(--text-sm);
  font-weight: var(--weight-medium);
  margin-bottom: var(--space-2);
}

.stat-value {
  font-size: var(--text-2xl);
  font-weight: var(--weight-semibold);
  color: var(--ink);
  line-height: var(--leading-tight);
  font-variant-numeric: tabular-nums;
}

.stat-card.warning .stat-value {
  color: var(--warn);
}

.stat-card.success .stat-value {
  color: var(--ok);
}

.stat-card.danger .stat-value {
  color: var(--alert);
}

.stat-card.info .stat-value {
  color: var(--info);
}

.card {
  background: var(--surface);
  border: 1px solid var(--rule);
  border-radius: var(--radius-panel);
  padding: var(--space-5);
  margin-bottom: var(--space-5);
}

.card-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: var(--space-4);
  padding-bottom: var(--space-3);
  border-bottom: 1px solid var(--rule);
}

.card-title {
  font-size: var(--text-md);
  font-weight: var(--weight-semibold);
  color: var(--ink);
}

.table-container {
  overflow-x: auto;
}

table {
  width: 100%;
  border-collapse: collapse;
}

thead {
  background: var(--surface-sunken);
  border-bottom: 1px solid var(--rule);
}

th {
  text-align: left;
  padding: var(--space-2) var(--space-3);
  font-weight: var(--weight-semibold);
  color: var(--muted);
  font-size: var(--text-xs);
  background: var(--surface-sunken);
}

td {
  padding: var(--space-2) var(--space-3);
  border-top: 1px solid var(--rule-soft);
  color: var(--ink-2);
  font-size: var(--text-base);
  font-variant-numeric: tabular-nums;
}

tbody tr {
  transition: background-color var(--dur-fast) var(--ease);
}

tbody tr:hover {
  background: var(--paper);
}

.badge {
  display: inline-flex;
  align-items: center;
  gap: 6px;
  padding: 2px 8px;
  border-radius: var(--radius-pill);
  font-size: var(--text-xs);
  font-weight: var(--weight-medium);
  line-height: var(--leading-normal);
  white-space: nowrap;
}

.badge::before {
  content: '';
  width: 6px;
  height: 6px;
  border-radius: 50%;
  background: currentColor;
  flex-shrink: 0;
}

.badge.success,
.badge.increasing {
  background: var(--ok-tint);
  color: var(--ok);
}

.badge.warning,
.badge.medium {
  background: var(--warn-tint);
  color: var(--warn);
}

.badge.danger,
.badge.high,
.badge.decreasing {
  background: var(--alert-tint);
  color: var(--alert);
}

.badge.info,
.badge.low {
  background: var(--info-tint);
  color: var(--info);
}

.badge.stable {
  background: var(--neutral-tint);
  color: var(--ink-2);
}

.loading {
  text-align: center;
  padding: var(--space-12);
  color: var(--muted);
  font-size: var(--text-base);
}

.error {
  background: var(--alert-tint);
  border: 1px solid rgba(185, 28, 28, 0.25);
  color: var(--alert);
  padding: var(--space-4);
  border-radius: var(--radius-panel);
  margin: var(--space-4) 0;
  font-size: var(--text-base);
}
</style>
