<template>
  <div class="app">
    <div class="app-layout">
      <aside class="sidebar" :class="{ collapsed: isCollapsed }">
        <div class="sidebar-header">
          <div class="sidebar-brand" v-show="!isCollapsed">
            <h1>{{ t('nav.companyName') }}</h1>
            <span class="sidebar-subtitle">{{ t('nav.subtitle') }}</span>
          </div>
          <button class="sidebar-toggle" @click="toggleSidebar" :title="isCollapsed ? 'Expand sidebar' : 'Collapse sidebar'">
            <svg width="16" height="16" viewBox="0 0 16 16" fill="none" class="toggle-icon" :class="{ rotated: isCollapsed }">
              <path d="M10 3L6 8L10 13" stroke="currentColor" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round"/>
            </svg>
          </button>
        </div>

        <nav class="sidebar-nav">
          <router-link to="/" :class="{ active: $route.path === '/' }" :data-label="t('nav.overview')">
            <svg width="16" height="16" viewBox="0 0 16 16" fill="none"><rect x="1" y="1" width="6" height="6" rx="1" stroke="currentColor" stroke-width="1.5"/><rect x="9" y="1" width="6" height="6" rx="1" stroke="currentColor" stroke-width="1.5"/><rect x="1" y="9" width="6" height="6" rx="1" stroke="currentColor" stroke-width="1.5"/><rect x="9" y="9" width="6" height="6" rx="1" stroke="currentColor" stroke-width="1.5"/></svg>
            <span class="nav-label">{{ t('nav.overview') }}</span>
          </router-link>
          <router-link to="/inventory" :class="{ active: $route.path === '/inventory' }" :data-label="t('nav.inventory')">
            <svg width="16" height="16" viewBox="0 0 16 16" fill="none"><path d="M2 4h12M2 8h12M2 12h12" stroke="currentColor" stroke-width="1.5" stroke-linecap="round"/></svg>
            <span class="nav-label">{{ t('nav.inventory') }}</span>
          </router-link>
          <router-link to="/orders" :class="{ active: $route.path === '/orders' }" :data-label="t('nav.orders')">
            <svg width="16" height="16" viewBox="0 0 16 16" fill="none"><path d="M2 2h12l-1.5 8H3.5L2 2z" stroke="currentColor" stroke-width="1.5" stroke-linejoin="round"/><circle cx="5.5" cy="13.5" r="1" fill="currentColor"/><circle cx="11.5" cy="13.5" r="1" fill="currentColor"/></svg>
            <span class="nav-label">{{ t('nav.orders') }}</span>
          </router-link>
          <router-link to="/spending" :class="{ active: $route.path === '/spending' }" :data-label="t('nav.finance')">
            <svg width="16" height="16" viewBox="0 0 16 16" fill="none"><circle cx="8" cy="8" r="6.5" stroke="currentColor" stroke-width="1.5"/><path d="M8 4.5v7M6 6.5h3a1 1 0 010 2H7a1 1 0 000 2h3" stroke="currentColor" stroke-width="1.5" stroke-linecap="round"/></svg>
            <span class="nav-label">{{ t('nav.finance') }}</span>
          </router-link>
          <router-link to="/demand" :class="{ active: $route.path === '/demand' }" :data-label="t('nav.demandForecast')">
            <svg width="16" height="16" viewBox="0 0 16 16" fill="none"><path d="M1.5 12.5l4-5 3 2 4.5-6" stroke="currentColor" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round"/></svg>
            <span class="nav-label">{{ t('nav.demandForecast') }}</span>
          </router-link>
          <router-link to="/reports" :class="{ active: $route.path === '/reports' }" data-label="Reports">
            <svg width="16" height="16" viewBox="0 0 16 16" fill="none"><rect x="2" y="2" width="12" height="12" rx="1.5" stroke="currentColor" stroke-width="1.5"/><path d="M5 8h6M5 5.5h6M5 10.5h4" stroke="currentColor" stroke-width="1.5" stroke-linecap="round"/></svg>
            <span class="nav-label">Reports</span>
          </router-link>
        </nav>

        <div class="sidebar-filters">
          <FilterBar />
        </div>

        <div class="sidebar-footer">
          <LanguageSwitcher />
          <ProfileMenu
            @show-profile-details="showProfileDetails = true"
            @show-tasks="showTasks = true"
          />
        </div>
      </aside>

      <main class="main-content">
        <router-view />
      </main>
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
import { ref, onMounted, onUnmounted, computed } from 'vue'
import { api } from './api'
import { useAuth } from './composables/useAuth'
import { useI18n } from './composables/useI18n'
import FilterBar from './components/FilterBar.vue'
import ProfileMenu from './components/ProfileMenu.vue'
import ProfileDetailsModal from './components/ProfileDetailsModal.vue'
import TasksModal from './components/TasksModal.vue'
import LanguageSwitcher from './components/LanguageSwitcher.vue'

export default {
  name: 'App',
  components: {
    FilterBar,
    ProfileMenu,
    ProfileDetailsModal,
    TasksModal,
    LanguageSwitcher
  },
  setup() {
    const { currentUser } = useAuth()
    const { t } = useI18n()
    const showProfileDetails = ref(false)
    const showTasks = ref(false)
    const apiTasks = ref([])

    // Initialise from localStorage, fallback to window width check
    const isCollapsed = ref(
      localStorage.getItem('sidebar-collapsed') === 'true' ||
      window.innerWidth < 1280
    )

    const toggleSidebar = () => {
      isCollapsed.value = !isCollapsed.value
      localStorage.setItem('sidebar-collapsed', isCollapsed.value)
    }

    // Auto-collapse/expand on resize
    const handleResize = () => {
      if (window.innerWidth < 1280 && !isCollapsed.value) {
        isCollapsed.value = true
        localStorage.setItem('sidebar-collapsed', 'true')
      }
    }

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

    onMounted(() => {
      loadTasks()
      window.addEventListener('resize', handleResize)
    })

    onUnmounted(() => {
      window.removeEventListener('resize', handleResize)
    })

    return {
      t,
      showProfileDetails,
      showTasks,
      tasks,
      addTask,
      deleteTask,
      toggleTask,
      isCollapsed,
      toggleSidebar
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
  font-family: 'Inter', -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, sans-serif;
  background: #f8fafc;
  color: #1e293b;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
}

.app {
  min-height: 100vh;
}

.app-layout {
  display: flex;
  min-height: 100vh;
}

/* ── Sidebar ── */
.sidebar {
  width: 240px;
  min-width: 240px;
  background: #0f172a;
  display: flex;
  flex-direction: column;
  position: fixed;
  top: 0;
  left: 0;
  height: 100vh;
  z-index: 100;
  border-right: 1px solid #1e293b;
  transition: width 0.2s ease;
  overflow: hidden;
}

/* Override overflow-y:auto only when expanded so content scrolls */
.sidebar:not(.collapsed) {
  overflow-y: auto;
}

/* ── Collapsed state ── */
.sidebar.collapsed {
  width: 56px;
  min-width: 56px;
}

.sidebar.collapsed ~ .main-content,
.app-layout:has(.sidebar.collapsed) .main-content {
  margin-left: 56px;
}

/* Hide text/filters/footer content when collapsed */
.sidebar.collapsed .sidebar-filters {
  display: none;
}

/* In collapsed mode, center icons in nav links */
.sidebar.collapsed .sidebar-nav a {
  justify-content: center;
  padding: 0.625rem;
}

/* In collapsed mode, center the toggle button */
.sidebar.collapsed .sidebar-header {
  justify-content: center;
  padding: 1rem 0.625rem;
}

/* In collapsed mode, hide the brand text area */
.sidebar.collapsed .sidebar-brand {
  display: none;
}

/* In collapsed mode, shrink footer to icon-only */
.sidebar.collapsed .sidebar-footer {
  flex-direction: column;
  align-items: center;
  gap: 0.25rem;
  padding: 0.5rem 0.375rem;
}

/* ── Sidebar header layout ── */
.sidebar-header {
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 1.25rem 1rem 1.125rem;
  border-bottom: 1px solid #1e293b;
  flex-shrink: 0;
}

.sidebar-brand {
  flex: 1;
  min-width: 0;
}

.sidebar-header h1 {
  font-size: 0.938rem;
  font-weight: 700;
  color: #f8fafc;
  letter-spacing: -0.02em;
}

.sidebar-subtitle {
  display: block;
  font-size: 0.688rem;
  color: #475569;
  margin-top: 2px;
  font-weight: 400;
}

/* ── Toggle button ── */
.sidebar-toggle {
  display: flex;
  align-items: center;
  justify-content: center;
  width: 28px;
  height: 28px;
  background: transparent;
  border: 1px solid #1e293b;
  border-radius: 6px;
  color: #64748b;
  cursor: pointer;
  transition: background 0.15s, color 0.15s, border-color 0.15s;
  flex-shrink: 0;
}

.sidebar-toggle:hover {
  background: #1e293b;
  border-color: #334155;
  color: #e2e8f0;
}

.toggle-icon {
  transition: transform 0.2s ease;
}

.toggle-icon.rotated {
  transform: rotate(180deg);
}

/* ── Nav links ── */
.sidebar-nav {
  padding: 0.625rem 0.625rem 0;
  display: flex;
  flex-direction: column;
  gap: 1px;
  flex-shrink: 0;
}

.sidebar-nav a {
  display: flex;
  align-items: center;
  gap: 0.625rem;
  padding: 0.5rem 0.75rem;
  border-radius: 6px;
  font-size: 0.813rem;
  font-weight: 500;
  color: #94a3b8;
  text-decoration: none;
  transition: background 0.15s ease, color 0.15s ease;
  position: relative;
}

.sidebar-nav a svg {
  flex-shrink: 0;
  opacity: 0.7;
  transition: opacity 0.15s;
}

.sidebar-nav a:hover {
  background: #1e293b;
  color: #e2e8f0;
}

.sidebar-nav a:hover svg {
  opacity: 1;
}

.sidebar-nav a.active {
  background: #1e293b;
  color: #f8fafc;
  font-weight: 600;
}

.sidebar-nav a.active svg {
  opacity: 1;
  color: #3b82f6;
}

/* Blue left accent bar on active item */
.sidebar-nav a.active::before {
  content: '';
  position: absolute;
  left: 0;
  top: 4px;
  bottom: 4px;
  width: 2px;
  background: #3b82f6;
  border-radius: 0 2px 2px 0;
}

/* ── Nav label span ── */
.nav-label {
  flex: 1;
  overflow: hidden;
  white-space: nowrap;
  transition: opacity 0.15s ease;
}

.sidebar.collapsed .nav-label {
  opacity: 0;
  width: 0;
}

/* ── Tooltips in collapsed mode ── */
.sidebar.collapsed .sidebar-nav a {
  position: relative;
}

.sidebar.collapsed .sidebar-nav a::after {
  content: attr(data-label);
  position: absolute;
  left: calc(100% + 10px);
  top: 50%;
  transform: translateY(-50%);
  background: #1e293b;
  color: #e2e8f0;
  font-size: 0.75rem;
  font-weight: 500;
  padding: 0.375rem 0.625rem;
  border-radius: 6px;
  white-space: nowrap;
  pointer-events: none;
  opacity: 0;
  transition: opacity 0.15s ease;
  border: 1px solid #334155;
  z-index: 200;
  box-shadow: 0 4px 12px rgba(0,0,0,0.3);
}

.sidebar.collapsed .sidebar-nav a:hover::after {
  opacity: 1;
}

/* ── Filters section ── */
.sidebar-filters {
  padding: 0.75rem 0.625rem 0.5rem;
  border-top: 1px solid #1e293b;
  margin-top: 0.625rem;
  flex-shrink: 0;
}

/* ── Footer (language + profile) ── */
.sidebar-footer {
  margin-top: auto;
  padding: 0.75rem 0.625rem;
  border-top: 1px solid #1e293b;
  display: flex;
  align-items: center;
  gap: 0.5rem;
  flex-shrink: 0;
}

/* ── Main content area ── */
.main-content {
  flex: 1;
  margin-left: 240px;
  padding: 1.75rem 2rem;
  min-width: 0;
  background: #f8fafc;
  transition: margin-left 0.2s ease;
}

/* ── Global shared styles ── */
.page-header {
  margin-bottom: 1.5rem;
}

.page-header h2 {
  font-size: 1.875rem;
  font-weight: 700;
  color: #0f172a;
  margin-bottom: 0.375rem;
  letter-spacing: -0.025em;
}

.page-header p {
  color: #64748b;
  font-size: 0.938rem;
}

.stats-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(280px, 1fr));
  gap: 1.25rem;
  margin-bottom: 1.5rem;
}

.stat-card {
  background: white;
  padding: 1.25rem;
  border-radius: 10px;
  border: 1px solid #e2e8f0;
  transition: all 0.2s ease;
}

.stat-card:hover {
  border-color: #cbd5e1;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.06);
}

.stat-label {
  color: #64748b;
  font-size: 0.875rem;
  font-weight: 600;
  text-transform: uppercase;
  letter-spacing: 0.5px;
  margin-bottom: 0.625rem;
}

.stat-value {
  font-size: 2.25rem;
  font-weight: 700;
  color: #0f172a;
  letter-spacing: -0.025em;
}

.stat-card.warning .stat-value {
  color: #ea580c;
}

.stat-card.success .stat-value {
  color: #059669;
}

.stat-card.danger .stat-value {
  color: #dc2626;
}

.stat-card.info .stat-value {
  color: #2563eb;
}

.card {
  background: white;
  border-radius: 10px;
  padding: 1.25rem;
  border: 1px solid #e2e8f0;
  margin-bottom: 1.25rem;
}

.card-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 1rem;
  padding-bottom: 0.875rem;
  border-bottom: 1px solid #e2e8f0;
}

.card-title {
  font-size: 1.125rem;
  font-weight: 700;
  color: #0f172a;
  letter-spacing: -0.025em;
}

.table-container {
  overflow-x: auto;
}

table {
  width: 100%;
  border-collapse: collapse;
}

thead {
  background: #f8fafc;
  border-top: 1px solid #e2e8f0;
  border-bottom: 1px solid #e2e8f0;
}

th {
  text-align: left;
  padding: 0.5rem 0.75rem;
  font-weight: 600;
  color: #475569;
  font-size: 0.75rem;
  text-transform: uppercase;
  letter-spacing: 0.05em;
}

td {
  padding: 0.5rem 0.75rem;
  border-top: 1px solid #f1f5f9;
  color: #334155;
  font-size: 0.875rem;
}

tbody tr {
  transition: background-color 0.15s ease;
}

tbody tr:hover {
  background: #f8fafc;
}

.badge {
  display: inline-block;
  padding: 0.313rem 0.75rem;
  border-radius: 6px;
  font-size: 0.75rem;
  font-weight: 600;
  text-transform: uppercase;
  letter-spacing: 0.025em;
}

.badge.success {
  background: #d1fae5;
  color: #065f46;
}

.badge.warning {
  background: #fed7aa;
  color: #92400e;
}

.badge.danger {
  background: #fecaca;
  color: #991b1b;
}

.badge.info {
  background: #dbeafe;
  color: #1e40af;
}

.badge.increasing {
  background: #d1fae5;
  color: #065f46;
}

.badge.decreasing {
  background: #fecaca;
  color: #991b1b;
}

.badge.stable {
  background: #e0e7ff;
  color: #3730a3;
}

.badge.high {
  background: #fecaca;
  color: #991b1b;
}

.badge.medium {
  background: #fed7aa;
  color: #92400e;
}

.badge.low {
  background: #dbeafe;
  color: #1e40af;
}

.loading {
  text-align: center;
  padding: 3rem;
  color: #64748b;
  font-size: 0.938rem;
}

.error {
  background: #fef2f2;
  border: 1px solid #fecaca;
  color: #991b1b;
  padding: 1rem;
  border-radius: 8px;
  margin: 1rem 0;
  font-size: 0.938rem;
}
</style>
