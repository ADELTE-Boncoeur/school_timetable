<template>
  <div class="theme-switcher" :class="{ open: isOpen }">
    <button
      class="theme-toggle-btn"
      @click="togglePanel"
      :aria-label="isOpen ? 'Close theme panel' : 'Open theme panel'"
      :title="isOpen ? 'Close themes' : 'Choose theme'"
    >
      <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" width="20" height="20">
        <circle cx="12" cy="12" r="5" />
        <line x1="12" y1="1" x2="12" y2="3" />
        <line x1="12" y1="21" x2="12" y2="23" />
        <line x1="4.22" y1="4.22" x2="5.64" y2="5.64" />
        <line x1="18.36" y1="18.36" x2="19.78" y2="19.78" />
        <line x1="1" y1="12" x2="3" y2="12" />
        <line x1="21" y1="12" x2="23" y2="12" />
        <line x1="4.22" y1="19.78" x2="5.64" y2="18.36" />
        <line x1="18.36" y1="5.64" x2="19.78" y2="4.22" />
      </svg>
    </button>

    <transition name="panel-slide">
      <div v-if="isOpen" class="theme-panel">
        <div class="theme-panel-header">
          <span>Themes</span>
          <button class="close-btn" @click="togglePanel" aria-label="Close">&times;</button>
        </div>
        <div class="theme-grid">
          <button
            v-for="theme in themes"
            :key="theme.id"
            class="theme-option"
            :class="{ active: currentTheme === theme.id }"
            @click="setTheme(theme.id)"
            :aria-label="'Switch to ' + theme.name + ' theme'"
          >
            <span class="theme-preview" :style="{ background: theme.preview }"></span>
            <span class="theme-name">{{ theme.name }}</span>
          </button>
        </div>
      </div>
    </transition>
  </div>

  <div v-if="isOpen" class="theme-backdrop" @click="togglePanel"></div>
</template>

<script setup>
import { ref, onMounted } from 'vue'

const isOpen = ref(false)
const currentTheme = ref('dark')

const themes = [
  { id: 'light', name: 'Light', preview: 'linear-gradient(135deg, #f0f2f5 50%, #6c5ce7 50%)' },
  { id: 'dark', name: 'Dark', preview: 'linear-gradient(135deg, #0a0a1a 50%, #00f2ff 50%)' },
  { id: 'huawei', name: 'Huawei', preview: 'linear-gradient(135deg, #1a0a0a 50%, #cf0a2c 50%)' },
  { id: 'ocean', name: 'Ocean', preview: 'linear-gradient(135deg, #0a1628 50%, #00b4d8 50%)' },
  { id: 'sunset', name: 'Sunset', preview: 'linear-gradient(135deg, #1a0f1e 50%, #ff6b6b 50%)' },
  { id: 'forest', name: 'Forest', preview: 'linear-gradient(135deg, #0a1a0f 50%, #00c853 50%)' },
  { id: 'cyberpunk', name: 'Cyber', preview: 'linear-gradient(135deg, #0d0221 50%, #ff00ff 50%)' },
]

const togglePanel = () => {
  isOpen.value = !isOpen.value
}

const setTheme = (themeId) => {
  currentTheme.value = themeId
  document.documentElement.setAttribute('data-theme', themeId)
  localStorage.setItem('app-theme', themeId)
}

onMounted(() => {
  const saved = localStorage.getItem('app-theme')
  if (saved) {
    currentTheme.value = saved
    document.documentElement.setAttribute('data-theme', saved)
  } else {
    document.documentElement.setAttribute('data-theme', 'dark')
  }
})
</script>

<style scoped>
.theme-switcher {
  position: fixed;
  top: 1rem;
  left: 1rem;
  z-index: 3000;
}

.theme-toggle-btn {
  width: 44px;
  height: 44px;
  border-radius: 50%;
  border: 1px solid var(--border);
  background: var(--switcher-bg);
  backdrop-filter: blur(12px);
  color: var(--accent);
  display: flex;
  align-items: center;
  justify-content: center;
  transition: all 0.3s ease;
  box-shadow: 0 4px 15px rgba(0, 0, 0, 0.15);
}

.theme-toggle-btn:hover {
  transform: scale(1.1);
  border-color: var(--accent);
  box-shadow: 0 0 20px var(--accent-glow);
}

.theme-panel {
  position: absolute;
  top: 56px;
  left: 0;
  background: var(--switcher-bg);
  backdrop-filter: blur(20px);
  border: 1px solid var(--border);
  border-radius: 16px;
  padding: 1rem;
  min-width: 220px;
  box-shadow: 0 15px 40px rgba(0, 0, 0, 0.25);
}

.theme-panel-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 0.75rem;
  padding-bottom: 0.5rem;
  border-bottom: 1px solid var(--border);
  font-weight: 700;
  font-size: 0.85rem;
  letter-spacing: 1px;
  text-transform: uppercase;
  color: var(--text-secondary);
}

.close-btn {
  background: none;
  border: none;
  color: var(--text-muted);
  font-size: 1.4rem;
  line-height: 1;
  padding: 0;
  transition: color 0.2s;
}

.close-btn:hover {
  color: var(--accent);
}

.theme-grid {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 0.5rem;
}

.theme-option {
  display: flex;
  align-items: center;
  gap: 0.5rem;
  padding: 0.5rem 0.6rem;
  border-radius: 10px;
  border: 1px solid transparent;
  background: transparent;
  color: var(--text-primary);
  font-size: 0.75rem;
  font-weight: 600;
  transition: all 0.2s ease;
  white-space: nowrap;
}

.theme-option:hover {
  background: var(--accent-glow);
  border-color: var(--border);
}

.theme-option.active {
  border-color: var(--accent);
  background: var(--accent-glow);
}

.theme-preview {
  width: 24px;
  height: 24px;
  border-radius: 50%;
  flex-shrink: 0;
  border: 2px solid var(--border);
}

.theme-option.active .theme-preview {
  border-color: var(--accent);
  box-shadow: 0 0 8px var(--accent-glow);
}

.theme-name {
  overflow: hidden;
  text-overflow: ellipsis;
}

.theme-backdrop {
  position: fixed;
  inset: 0;
  z-index: 2999;
}

/* Transition */
.panel-slide-enter-active,
.panel-slide-leave-active {
  transition: all 0.25s ease;
}

.panel-slide-enter-from,
.panel-slide-leave-to {
  opacity: 0;
  transform: translateY(-10px) scale(0.95);
}

/* Mobile: move to bottom sheet style */
@media (max-width: 480px) {
  .theme-panel {
    position: fixed;
    top: auto;
    bottom: 0;
    left: 0;
    right: 0;
    border-radius: 20px 20px 0 0;
    min-width: unset;
    padding: 1.25rem;
    max-height: 60vh;
    overflow-y: auto;
  }

  .theme-grid {
    grid-template-columns: 1fr 1fr 1fr;
  }
}
</style>
