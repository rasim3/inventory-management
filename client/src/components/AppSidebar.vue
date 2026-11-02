<script setup>
import { ref, computed, watch, nextTick, onMounted, onBeforeUnmount } from 'vue'
import { useRoute } from 'vue-router'
import { useI18n } from '../composables/useI18n'

const props = defineProps({
  mobileOpen: { type: Boolean, default: false }
})
const emit = defineEmits(['close'])

const { t } = useI18n()
const route = useRoute()

const STORAGE_KEY = 'sidebar-collapsed'

const readCollapsed = () => {
  try {
    return localStorage.getItem(STORAGE_KEY) === 'true'
  } catch (e) {
    return false
  }
}

const collapsed = ref(readCollapsed())

const mqTablet = typeof window !== 'undefined' && window.matchMedia
  ? window.matchMedia('(min-width: 768px) and (max-width: 1279.98px)')
  : null
const mqMobile = typeof window !== 'undefined' && window.matchMedia
  ? window.matchMedia('(max-width: 767.98px)')
  : null

const isTablet = ref(mqTablet ? mqTablet.matches : false)
const isMobile = ref(mqMobile ? mqMobile.matches : false)
// Temporary overlay expansion on tablet; never persisted.
const tabletOpen = ref(false)

const onTabletChange = (e) => {
  isTablet.value = e.matches
  if (!e.matches) tabletOpen.value = false
}
const onMobileChange = (e) => {
  isMobile.value = e.matches
  if (!e.matches && props.mobileOpen) emit('close')
}

const sidebarEl = ref(null)
const focusFirstNavItem = async () => {
  await nextTick()
  const el = sidebarEl.value?.querySelector('.nav-item')
  if (el) el.focus()
}
const isDrawer = computed(() => isMobile.value && props.mobileOpen)

watch(() => props.mobileOpen, (open) => {
  if (open) focusFirstNavItem()
  document.body.style.overflow = open && isMobile.value ? 'hidden' : ''
})
watch(isMobile, (m) => {
  if (!m) document.body.style.overflow = ''
})
watch(tabletOpen, (open) => {
  if (open) focusFirstNavItem()
})

// Icons-only: desktop collapsed preference, or the tablet rail when not overlaid
const iconsOnly = computed(() => {
  if (isMobile.value) return false
  if (isTablet.value) return !tabletOpen.value
  return collapsed.value
})

const toggleCollapsed = () => {
  if (isTablet.value) {
    tabletOpen.value = !tabletOpen.value
    return
  }
  collapsed.value = !collapsed.value
  try {
    localStorage.setItem(STORAGE_KEY, String(collapsed.value))
  } catch (e) {
    // storage unavailable; state stays in memory only
  }
}

const navItems = [
  {
    path: '/',
    labelKey: 'nav.overview',
    exact: true,
    icon: ['M3.5 3.5h5.5v5.5H3.5z', 'M11 3.5h5.5v5.5H11z', 'M3.5 11h5.5v5.5H3.5z', 'M11 11h5.5v5.5H11z']
  },
  {
    path: '/inventory',
    labelKey: 'nav.inventory',
    icon: ['M3 6.5 10 3l7 3.5v7L10 17l-7-3.5z', 'M3 6.5 10 10l7-3.5', 'M10 10v7']
  },
  {
    path: '/orders',
    labelKey: 'nav.orders',
    icon: ['M5 3.5h10a1 1 0 0 1 1 1v11a1 1 0 0 1-1 1H5a1 1 0 0 1-1-1v-11a1 1 0 0 1 1-1z', 'M7 7.5h6', 'M7 10.5h6', 'M7 13.5h3.5']
  },
  {
    path: '/spending',
    labelKey: 'nav.finance',
    icon: ['M3 5.5h14v9H3z', 'M10 12.25a2.25 2.25 0 1 0 0-4.5 2.25 2.25 0 0 0 0 4.5z', 'M5.5 8v4', 'M14.5 8v4']
  },
  {
    path: '/demand',
    labelKey: 'nav.demandForecast',
    icon: ['M3 15l4.5-5 3 3L17 5.5', 'M12.5 5.5H17V10']
  },
  {
    path: '/reports',
    labelKey: 'nav.reports',
    icon: ['M4 16.5h12', 'M5.5 16.5v-5', 'M10 16.5v-10', 'M14.5 16.5v-7']
  }
]

const onBackdropClick = () => {
  emit('close')
  tabletOpen.value = false
}

const onKeydown = (e) => {
  if (e.key !== 'Escape' || e.defaultPrevented) return
  if (props.mobileOpen) emit('close')
  if (tabletOpen.value) tabletOpen.value = false
}

onMounted(() => {
  document.addEventListener('keydown', onKeydown)
  if (mqTablet) mqTablet.addEventListener('change', onTabletChange)
  if (mqMobile) mqMobile.addEventListener('change', onMobileChange)
})
onBeforeUnmount(() => {
  document.removeEventListener('keydown', onKeydown)
  document.body.style.overflow = ''
  if (mqTablet) mqTablet.removeEventListener('change', onTabletChange)
  if (mqMobile) mqMobile.removeEventListener('change', onMobileChange)
})

watch(() => route.path, () => {
  if (props.mobileOpen) emit('close')
  tabletOpen.value = false
})
</script>

<template>
  <div class="sidebar-wrap">
    <div
      class="sidebar-backdrop"
      :class="{ visible: mobileOpen, 'tablet-visible': tabletOpen }"
      aria-hidden="true"
      @click="onBackdropClick"
    ></div>

    <aside
      id="app-sidebar"
      ref="sidebarEl"
      :role="isDrawer ? 'dialog' : undefined"
      :aria-modal="isDrawer ? 'true' : undefined"
      class="sidebar"
      :class="{ collapsed, 'tablet-open': tabletOpen, 'mobile-open': mobileOpen }"
    >
      <div class="brand">
        <div class="brand-mark" aria-hidden="true">CC</div>
        <div class="brand-text">
          <div class="brand-name">{{ t('nav.companyName') }}</div>
          <div class="brand-sub">{{ t('nav.subtitle') }}</div>
        </div>
      </div>

      <nav class="nav" :aria-label="t('nav.mainNav')">
        <router-link
          v-for="item in navItems"
          :key="item.path"
          :to="item.path"
          class="nav-item"
          :active-class="item.exact ? '' : 'active'"
          :exact-active-class="item.exact ? 'active' : ''"
          :title="iconsOnly ? t(item.labelKey) : undefined"
          :aria-label="t(item.labelKey)"
        >
          <svg
            class="nav-icon"
            width="20"
            height="20"
            viewBox="0 0 20 20"
            fill="none"
            stroke="currentColor"
            stroke-width="1.5"
            stroke-linecap="round"
            stroke-linejoin="round"
            aria-hidden="true"
          >
            <path v-for="d in item.icon" :key="d" :d="d" />
          </svg>
          <span class="nav-label">{{ t(item.labelKey) }}</span>
        </router-link>
      </nav>

      <button
        type="button"
        class="collapse-toggle"
        :aria-label="iconsOnly ? t('nav.expand') : t('nav.collapse')"
        :title="iconsOnly ? t('nav.expand') : undefined"
        @click="toggleCollapsed"
      >
        <svg
          class="nav-icon chevron"
          :class="{ flipped: iconsOnly }"
          width="20"
          height="20"
          viewBox="0 0 20 20"
          fill="none"
          stroke="currentColor"
          stroke-width="1.5"
          stroke-linecap="round"
          stroke-linejoin="round"
          aria-hidden="true"
        >
          <path d="M12 5l-5 5 5 5" />
        </svg>
        <span class="nav-label">{{ iconsOnly ? t('nav.expand') : t('nav.collapse') }}</span>
      </button>
    </aside>
  </div>
</template>

<style scoped>
.sidebar-wrap {
  position: relative;
}

.sidebar {
  position: sticky;
  top: 0;
  height: 100vh;
  width: var(--sidebar-w);
  display: flex;
  flex-direction: column;
  background: var(--surface);
  border-right: 1px solid var(--rule);
  padding: var(--space-4) var(--space-3);
  transition: width var(--dur-base) var(--ease);
  overflow: hidden;
  z-index: 200;
}

.brand {
  display: flex;
  align-items: center;
  gap: var(--space-3);
  min-height: 40px;
  padding: 0 var(--space-2);
  margin-bottom: var(--space-6);
}

.brand-mark {
  display: none;
  width: 32px;
  height: 32px;
  border-radius: var(--radius-control);
  border: 1px solid var(--rule);
  background: var(--surface-sunken);
  color: var(--ink);
  font-size: var(--text-sm);
  font-weight: var(--weight-semibold);
  align-items: center;
  justify-content: center;
  flex-shrink: 0;
}

.brand-text {
  min-width: 0;
}

.brand-name {
  font-size: var(--text-md);
  font-weight: var(--weight-semibold);
  color: var(--ink);
  line-height: var(--leading-tight);
  white-space: nowrap;
}

.brand-sub {
  font-size: var(--text-xs);
  color: var(--muted);
  white-space: nowrap;
}

.nav {
  display: flex;
  flex-direction: column;
  gap: var(--space-1);
  flex: 1;
}

.nav-item,
.collapse-toggle {
  position: relative;
  display: flex;
  align-items: center;
  gap: var(--space-3);
  height: 36px;
  padding: 0 var(--space-3);
  border-radius: var(--radius-control);
  color: var(--muted);
  font-family: inherit;
  font-size: var(--text-base);
  font-weight: var(--weight-medium);
  text-decoration: none;
  white-space: nowrap;
  background: transparent;
  border: none;
  cursor: pointer;
  width: 100%;
  text-align: left;
  transition: background var(--dur-fast) var(--ease), color var(--dur-fast) var(--ease);
}

.nav-item:hover,
.collapse-toggle:hover {
  background: var(--surface-sunken);
  color: var(--ink);
}

.nav-item.active {
  background: var(--cobalt-tint);
  color: var(--cobalt);
}

.nav-item.active::before {
  content: '';
  position: absolute;
  left: 0;
  top: 6px;
  bottom: 6px;
  width: 2px;
  border-radius: 1px;
  background: var(--cobalt);
}

.nav-icon {
  flex-shrink: 0;
}

.chevron {
  transition: transform var(--dur-base) var(--ease);
}

.chevron.flipped {
  transform: rotate(180deg);
}

.collapse-toggle {
  margin-top: var(--space-2);
  border-top: 1px solid var(--rule-soft);
  border-radius: var(--radius-control);
}

.sidebar-backdrop {
  display: none;
}

/* Icons-only: desktop collapsed preference */
@media (min-width: 1280px) {
  .sidebar.collapsed {
    width: var(--sidebar-w-collapsed);
  }

  .sidebar.collapsed .brand {
    justify-content: center;
    padding: 0;
  }

  .sidebar.collapsed .brand-mark {
    display: flex;
  }

  .sidebar.collapsed .brand-text,
  .sidebar.collapsed .nav-label {
    display: none;
  }

  .sidebar.collapsed .nav-item,
  .sidebar.collapsed .collapse-toggle {
    justify-content: center;
    padding: 0;
  }
}

/* Tablet: icons-only rail by default, full sidebar as overlay when opened */
@media (min-width: 768px) and (max-width: 1279.98px) {
  .sidebar-wrap {
    width: var(--sidebar-w-collapsed);
  }

  .sidebar {
    width: var(--sidebar-w-collapsed);
  }

  .sidebar:not(.tablet-open) .brand {
    justify-content: center;
    padding: 0;
  }

  .sidebar:not(.tablet-open) .brand-mark {
    display: flex;
  }

  .sidebar:not(.tablet-open) .brand-text,
  .sidebar:not(.tablet-open) .nav-label {
    display: none;
  }

  .sidebar:not(.tablet-open) .nav-item,
  .sidebar:not(.tablet-open) .collapse-toggle {
    justify-content: center;
    padding: 0;
  }

  .sidebar.tablet-open {
    position: fixed;
    left: 0;
    top: 0;
    width: var(--sidebar-w);
    z-index: 300;
    box-shadow: var(--shadow-float);
  }

  .sidebar-backdrop {
    display: block;
    position: fixed;
    inset: 0;
    background: transparent;
    pointer-events: none;
    z-index: 250;
  }

  .sidebar-backdrop.tablet-visible {
    pointer-events: auto;
  }
}

/* Mobile drawer */
@media (max-width: 767.98px) {
  .sidebar-wrap {
    position: static;
  }

  .sidebar {
    position: fixed;
    left: 0;
    top: 0;
    bottom: 0;
    height: 100%;
    width: var(--sidebar-w);
    transform: translateX(-100%);
    transition: transform var(--dur-base) var(--ease), visibility 0s linear var(--dur-base);
    visibility: hidden;
    z-index: 300;
  }

  .sidebar.mobile-open {
    transform: translateX(0);
    visibility: visible;
    transition: transform var(--dur-base) var(--ease), visibility 0s linear 0s;
    box-shadow: var(--shadow-float);
  }

  .collapse-toggle {
    display: none;
  }

  .sidebar-backdrop {
    display: block;
    position: fixed;
    inset: 0;
    background: rgba(15, 23, 42, 0.4);
    opacity: 0;
    pointer-events: none;
    transition: opacity var(--dur-base) var(--ease);
    z-index: 250;
  }

  .sidebar-backdrop.visible {
    opacity: 1;
    pointer-events: auto;
  }
}
</style>
