<template>
  <div class="app-shell">
    <!-- Overlay -->
    <div class="overlay-mask"></div>

    <!-- Video Background -->
    <div class="video-container">
      <div id="player"></div>
    </div>

    <!-- Navigation Bar -->
    <nav class="navbar" :class="{ scrolled: isScrolled }">
      <div class="nav-inner">
        <div class="nav-brand">
          <span class="brand-text">L3SOD<span class="brand-accent">.B</span></span>
        </div>

        <!-- Desktop Nav Links -->
        <ul class="nav-links desktop-only">
          <li v-for="(info, day) in timetable" :key="'nav-' + day">
            <a href="#" @click.prevent="openModal(day, info)">{{ day.slice(0, 3) }}</a>
          </li>
        </ul>

        <div class="nav-right">
          <span class="nav-clock">{{ currentTime }}</span>
          <!-- Hamburger -->
          <button class="hamburger mobile-only" @click="toggleMobileMenu" :aria-label="mobileMenuOpen ? 'Close menu' : 'Open menu'">
            <span class="hamburger-line" :class="{ open: mobileMenuOpen }"></span>
            <span class="hamburger-line" :class="{ open: mobileMenuOpen }"></span>
            <span class="hamburger-line" :class="{ open: mobileMenuOpen }"></span>
          </button>
        </div>
      </div>

      <!-- Mobile Menu -->
      <transition name="menu-slide">
        <div v-if="mobileMenuOpen" class="mobile-menu">
          <a
            v-for="(info, day) in timetable"
            :key="'mob-' + day"
            href="#"
            class="mobile-menu-item"
            @click.prevent="openModal(day, info); toggleMobileMenu()"
          >
            {{ day }}
          </a>
        </div>
      </transition>
    </nav>

    <!-- Main Content -->
    <main class="main-content">
      <!-- Hero Section -->
      <section class="hero-section">
        <h1 class="hero-title">L3SOD<span class="brand-accent">.B</span></h1>
        <p class="hero-subtitle">TERMINAL SCHEDULE SYSTEM</p>
      </section>

      <!-- Day Cards Grid -->
      <section class="cards-section">
        <div class="day-grid">
          <div
            v-for="(info, day) in timetable"
            :key="day"
            class="day-card"
            @click="openModal(day, info)"
            tabindex="0"
            @keydown.enter="openModal(day, info)"
            role="button"
            :aria-label="'View ' + day + ' schedule'"
          >
            <img :src="info.imgUrl" class="day-img" :alt="day + ' background'" loading="lazy" />
            <div class="day-card-overlay"></div>
            <div class="day-label">
              <span class="day-name">{{ day }}</span>
              <span class="day-count">{{ info.subs.length }} classes</span>
            </div>
          </div>
        </div>
      </section>

      <!-- Footer -->
      <footer class="app-footer">
        <p>&copy; 2026 L3SOD.B &mdash; Schedule System</p>
      </footer>
    </main>

    <!-- Modal -->
    <transition name="modal-fade">
      <div v-if="modalOpen" class="modal-overlay" @click="closeModal">
        <div class="modal-card" @click.stop role="dialog" aria-modal="true">
          <div class="modal-header">
            <h2 class="modal-day">{{ selectedDay }}</h2>
            <button class="modal-close" @click="closeModal" aria-label="Close">&times;</button>
          </div>
          <div class="modal-body">
            <div
              v-for="(sub, index) in selectedSubs"
              :key="index"
              :class="['sub-row', { 'practice-highlight': isPractice(sub) }]"
            >
              <span class="sub-text">{{ sub }}</span>
              <span v-if="isPractice(sub)" class="badge">PRACTICE</span>
            </div>
          </div>
          <button class="modal-disconnect" @click="closeModal">DISCONNECT</button>
        </div>
      </div>
    </transition>
  </div>
</template>

<script setup>
import { ref, onMounted, onBeforeUnmount } from 'vue'

const currentTime = ref('00:00:00')
const modalOpen = ref(false)
const selectedDay = ref('')
const selectedSubs = ref([])
const mobileMenuOpen = ref(false)
const isScrolled = ref(false)
let clockInterval
let player

const timetable = ref({
  Monday: {
    imgUrl: 'https://images.unsplash.com/photo-1550745165-9bc0b252726f?auto=format&fit=crop&q=60&w=600',
    subs: ['1-3: UI/UX', '4-6: DJF', '7-8: MATH', '9-10: GD']
  },
  Tuesday: {
    imgUrl: 'https://images.unsplash.com/photo-1581091226825-a6a2a5aee158?auto=format&fit=crop&q=60&w=600',
    subs: ['1-3: DJF', '4-6: DJF Practice', '7-8: CL', '9-10: VC Pract']
  },
  Wednesday: {
    imgUrl: 'https://images.unsplash.com/photo-1441974231531-c6227db76b6e?auto=format&fit=crop&q=60&w=600',
    subs: ['1-3: VC', 'BREAK', '4-6: DGV', 'LUNCH', '7-8: MATH']
  },
  Thursday: {
    imgUrl: 'https://images.unsplash.com/photo-1504384308090-c894fdcc538d?auto=format&fit=crop&q=60&w=600',
    subs: ['1-3: UI/UX', '4-6: DJF Practice', '7-8: PHY', '9-10: DJF Pract']
  },
  Friday: {
    imgUrl: 'https://images.unsplash.com/photo-1470071459604-3b5ec3a7fe05?auto=format&fit=crop&q=60&w=600',
    subs: ['1-3: DGV', '4-6: FR']
  }
})

const updateClock = () => {
  currentTime.value = new Date().toLocaleTimeString([], { hour12: false })
}

const isPractice = (sub) => sub.toLowerCase().includes('pract')

const openModal = (day, info) => {
  selectedDay.value = day
  selectedSubs.value = info.subs
  modalOpen.value = true
  document.body.style.overflow = 'hidden'
}

const closeModal = () => {
  modalOpen.value = false
  document.body.style.overflow = ''
}

const toggleMobileMenu = () => {
  mobileMenuOpen.value = !mobileMenuOpen.value
}

const handleScroll = () => {
  isScrolled.value = window.scrollY > 20
}

const handleKeydown = (e) => {
  if (e.key === 'Escape') {
    if (modalOpen.value) closeModal()
    if (mobileMenuOpen.value) mobileMenuOpen.value = false
  }
}

const initYouTubePlayer = () => {
  const tag = document.createElement('script')
  tag.src = 'https://www.youtube.com/iframe_api'
  const firstScriptTag = document.getElementsByTagName('script')[0]
  firstScriptTag.parentNode.insertBefore(tag, firstScriptTag)

  window.onYouTubeIframeAPIReady = () => {
    player = new YT.Player('player', {
      videoId: 'jfKfPfyJRdk',
      playerVars: { autoplay: 1, controls: 0, loop: 1, playlist: 'jfKfPfyJRdk', mute: 1 },
      events: { onReady: (e) => e.target.playVideo() }
    })
  }
}

onMounted(() => {
  updateClock()
  clockInterval = setInterval(updateClock, 1000)
  initYouTubePlayer()
  window.addEventListener('scroll', handleScroll)
  window.addEventListener('keydown', handleKeydown)
})

onBeforeUnmount(() => {
  if (clockInterval) clearInterval(clockInterval)
  window.removeEventListener('scroll', handleScroll)
  window.removeEventListener('keydown', handleKeydown)
})
</script>

<style scoped>
/* ===== APP SHELL ===== */
.app-shell {
  min-height: 100vh;
  min-height: 100dvh;
  display: flex;
  flex-direction: column;
  position: relative;
  overflow-x: hidden;
}

/* ===== VIDEO BACKGROUND ===== */
.video-container {
  position: fixed;
  inset: 0;
  z-index: -3;
  pointer-events: none;
  overflow: hidden;
  opacity: 0.5;
}

.video-container #player {
  position: absolute;
  top: 50%;
  left: 50%;
  width: 100vw;
  height: 56.25vw;
  min-height: 100vh;
  min-width: 177.77vh;
  transform: translate(-50%, -50%);
  filter: grayscale(0.5) brightness(0.5);
}

/* ===== OVERLAY ===== */
.overlay-mask {
  position: fixed;
  inset: 0;
  background: var(--overlay-gradient);
  z-index: -1;
  pointer-events: none;
}

/* ===== NAVBAR ===== */
.navbar {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  z-index: 1000;
  background: transparent;
  backdrop-filter: blur(0px);
  border-bottom: 1px solid transparent;
  transition: all 0.3s ease;
  padding: 0 clamp(1rem, 3vw, 2rem);
}

.navbar.scrolled {
  background: var(--nav-bg);
  backdrop-filter: blur(16px);
  border-bottom-color: var(--border);
  box-shadow: 0 4px 20px rgba(0, 0, 0, 0.1);
}

.nav-inner {
  max-width: 1400px;
  margin: 0 auto;
  display: flex;
  align-items: center;
  justify-content: space-between;
  height: clamp(56px, 8vw, 72px);
}

.nav-brand {
  flex-shrink: 0;
}

.brand-text {
  font-size: clamp(1.2rem, 3vw, 1.6rem);
  font-weight: 900;
  letter-spacing: -1px;
  color: var(--text-primary);
}

.brand-accent {
  color: var(--accent);
}

.nav-links {
  display: flex;
  gap: clamp(0.5rem, 2vw, 1.5rem);
}

.nav-links a {
  font-size: 0.85rem;
  font-weight: 700;
  letter-spacing: 2px;
  text-transform: uppercase;
  color: var(--text-secondary);
  padding: 0.4rem 0.8rem;
  border-radius: 8px;
  transition: all 0.2s ease;
}

.nav-links a:hover {
  color: var(--accent);
  background: var(--accent-glow);
}

.nav-right {
  display: flex;
  align-items: center;
  gap: 1rem;
}

.nav-clock {
  font-family: 'Courier New', monospace;
  font-size: clamp(0.85rem, 2vw, 1.1rem);
  font-weight: 600;
  color: var(--clock-color);
  text-shadow: 0 0 10px var(--accent-glow);
}

/* ===== HAMBURGER ===== */
.hamburger {
  width: 40px;
  height: 40px;
  background: none;
  border: 1px solid var(--border);
  border-radius: 8px;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  gap: 5px;
  padding: 8px;
  transition: border-color 0.2s;
}

.hamburger:hover {
  border-color: var(--accent);
}

.hamburger-line {
  display: block;
  width: 100%;
  height: 2px;
  background: var(--text-primary);
  border-radius: 2px;
  transition: all 0.3s ease;
}

.hamburger-line.open:nth-child(1) {
  transform: translateY(7px) rotate(45deg);
}

.hamburger-line.open:nth-child(2) {
  opacity: 0;
}

.hamburger-line.open:nth-child(3) {
  transform: translateY(-7px) rotate(-45deg);
}

/* ===== MOBILE MENU ===== */
.mobile-menu {
  display: flex;
  flex-direction: column;
  padding: 0.5rem 0 1rem;
}

.mobile-menu-item {
  padding: 0.8rem 1rem;
  font-weight: 700;
  font-size: 1rem;
  letter-spacing: 2px;
  color: var(--text-primary);
  border-radius: 10px;
  transition: all 0.2s;
}

.mobile-menu-item:hover {
  background: var(--accent-glow);
  color: var(--accent);
}

/* ===== VISIBILITY HELPERS ===== */
.desktop-only {
  display: flex;
}

.mobile-only {
  display: none;
}

@media (max-width: 768px) {
  .desktop-only {
    display: none !important;
  }

  .mobile-only {
    display: flex !important;
  }
}

/* ===== HERO SECTION ===== */
.main-content {
  flex: 1;
  display: flex;
  flex-direction: column;
  padding-top: clamp(56px, 8vw, 72px);
}

.hero-section {
  text-align: center;
  padding: clamp(2rem, 6vw, 5rem) clamp(1rem, 4vw, 2rem) clamp(1.5rem, 4vw, 3rem);
}

.hero-title {
  font-size: clamp(2.5rem, 8vw, 5rem);
  font-weight: 900;
  letter-spacing: -3px;
  color: var(--text-primary);
  filter: drop-shadow(var(--header-shadow));
  margin: 0 0 0.5rem;
  line-height: 1.1;
}

.hero-subtitle {
  font-size: clamp(0.65rem, 1.5vw, 0.85rem);
  letter-spacing: clamp(4px, 1vw, 8px);
  color: var(--text-muted);
  text-transform: uppercase;
}

/* ===== DAY CARDS GRID ===== */
.cards-section {
  flex: 1;
  display: flex;
  align-items: flex-start;
  justify-content: center;
  padding: 0 clamp(0.75rem, 3vw, 2rem) clamp(2rem, 4vw, 4rem);
}

.day-grid {
  display: grid;
  gap: clamp(0.75rem, 2vw, 1.25rem);
  width: 100%;
  max-width: 1400px;

  /* Mobile first: 1 column */
  grid-template-columns: 1fr;
}

/* >=480px: 2 columns */
@media (min-width: 480px) {
  .day-grid {
    grid-template-columns: repeat(2, 1fr);
  }
}

/* >=768px: 3 columns */
@media (min-width: 768px) {
  .day-grid {
    grid-template-columns: repeat(3, 1fr);
  }
}

/* >=1024px: 5 columns */
@media (min-width: 1024px) {
  .day-grid {
    grid-template-columns: repeat(5, 1fr);
  }
}

/* >=1440px: wider gaps */
@media (min-width: 1440px) {
  .day-grid {
    gap: 1.5rem;
  }
}

.day-card {
  position: relative;
  border-radius: clamp(12px, 2vw, 20px);
  overflow: hidden;
  cursor: pointer;
  border: 1px solid var(--border);
  background: var(--card-bg);
  backdrop-filter: blur(10px);
  transition: all 0.4s ease;
  aspect-ratio: 4 / 3;
  outline: none;
}

@media (min-width: 1024px) {
  .day-card {
    aspect-ratio: 3 / 4;
  }
}

.day-card:hover,
.day-card:focus-visible {
  transform: translateY(-8px);
  border-color: var(--accent);
  box-shadow: var(--card-hover-shadow);
}

.day-card:focus-visible {
  outline: 2px solid var(--accent);
  outline-offset: 2px;
}

.day-img {
  position: absolute;
  inset: 0;
  width: 100%;
  height: 100%;
  object-fit: cover;
  z-index: 0;
  opacity: 0.5;
  transition: opacity 0.5s, transform 0.5s;
}

.day-card:hover .day-img {
  opacity: 0.7;
  transform: scale(1.05);
}

.day-card-overlay {
  position: absolute;
  inset: 0;
  background: linear-gradient(to top, rgba(0, 0, 0, 0.8) 0%, transparent 60%);
  z-index: 1;
}

.day-label {
  position: absolute;
  bottom: 0;
  left: 0;
  right: 0;
  padding: clamp(0.75rem, 2vw, 1.25rem);
  z-index: 2;
  display: flex;
  flex-direction: column;
  gap: 0.2rem;
}

.day-name {
  font-weight: 900;
  font-size: clamp(0.85rem, 1.5vw, 1rem);
  letter-spacing: 2px;
  text-transform: uppercase;
  color: #fff;
}

.day-count {
  font-size: clamp(0.65rem, 1vw, 0.75rem);
  color: var(--accent);
  font-weight: 600;
  letter-spacing: 1px;
}

/* ===== MODAL ===== */
.modal-overlay {
  position: fixed;
  inset: 0;
  background: var(--bg-overlay);
  z-index: 2000;
  display: flex;
  justify-content: center;
  align-items: center;
  backdrop-filter: blur(20px);
  padding: 1rem;
}

.modal-card {
  width: 100%;
  max-width: 500px;
  max-height: 85vh;
  max-height: 85dvh;
  display: flex;
  flex-direction: column;
  padding: clamp(1.5rem, 4vw, 2.5rem);
  border-radius: clamp(16px, 3vw, 30px);
  border: 1px solid var(--accent);
  background: var(--bg-modal);
  box-shadow: var(--modal-shadow);
  overflow: hidden;
}

.modal-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 1rem;
  flex-shrink: 0;
}

.modal-day {
  color: var(--accent);
  font-size: clamp(1.5rem, 5vw, 2.5rem);
  font-weight: 900;
  letter-spacing: -1px;
  margin: 0;
}

.modal-close {
  width: 36px;
  height: 36px;
  border-radius: 50%;
  border: 1px solid var(--border);
  background: transparent;
  color: var(--text-secondary);
  font-size: 1.4rem;
  display: flex;
  align-items: center;
  justify-content: center;
  transition: all 0.2s;
  flex-shrink: 0;
}

.modal-close:hover {
  background: var(--accent-glow);
  color: var(--accent);
  border-color: var(--accent);
}

.modal-body {
  overflow-y: auto;
  flex: 1;
  -webkit-overflow-scrolling: touch;
}

.sub-row {
  padding: clamp(0.75rem, 2vw, 1rem);
  border-bottom: 1px solid var(--sub-row-border);
  display: flex;
  justify-content: space-between;
  align-items: center;
  gap: 0.5rem;
  border-radius: 8px;
  margin-bottom: 4px;
  transition: background 0.2s;
}

.sub-text {
  font-size: clamp(0.85rem, 1.5vw, 1rem);
  font-weight: 500;
  color: var(--text-primary);
}

.practice-highlight {
  background: var(--practice-gradient);
  border-left: 4px solid var(--practice-border);
  animation: pulse 2s infinite;
}

@keyframes pulse {
  0%, 100% {
    box-shadow: inset 0 0 10px var(--accent-glow);
  }
  50% {
    box-shadow: inset 0 0 25px var(--accent-glow);
  }
}

.badge {
  background: var(--badge-bg);
  color: var(--badge-text);
  padding: 0.2rem 0.6rem;
  font-size: clamp(0.6rem, 1vw, 0.7rem);
  font-weight: 900;
  border-radius: 4px;
  flex-shrink: 0;
  letter-spacing: 0.5px;
}

.modal-disconnect {
  width: 100%;
  margin-top: 1.25rem;
  background: var(--accent);
  border: none;
  padding: clamp(0.75rem, 2vw, 1rem);
  border-radius: 12px;
  font-weight: 900;
  font-size: clamp(0.8rem, 1.5vw, 0.95rem);
  letter-spacing: 2px;
  color: var(--btn-text);
  transition: all 0.3s;
  flex-shrink: 0;
}

.modal-disconnect:hover {
  letter-spacing: 4px;
  filter: brightness(1.15);
}

/* ===== FOOTER ===== */
.app-footer {
  text-align: center;
  padding: clamp(1rem, 3vw, 2rem);
  color: var(--text-muted);
  font-size: clamp(0.7rem, 1.2vw, 0.8rem);
  letter-spacing: 1px;
  border-top: 1px solid var(--border);
}

/* ===== MODAL TRANSITIONS ===== */
.modal-fade-enter-active,
.modal-fade-leave-active {
  transition: opacity 0.3s ease;
}

.modal-fade-enter-active .modal-card,
.modal-fade-leave-active .modal-card {
  transition: transform 0.3s ease;
}

.modal-fade-enter-from,
.modal-fade-leave-to {
  opacity: 0;
}

.modal-fade-enter-from .modal-card {
  transform: scale(0.9) translateY(20px);
}

.modal-fade-leave-to .modal-card {
  transform: scale(0.95) translateY(10px);
}

/* ===== MENU TRANSITIONS ===== */
.menu-slide-enter-active,
.menu-slide-leave-active {
  transition: all 0.3s ease;
}

.menu-slide-enter-from,
.menu-slide-leave-to {
  opacity: 0;
  max-height: 0;
}

.menu-slide-enter-to,
.menu-slide-leave-from {
  max-height: 400px;
}

/* ===== RESPONSIVE FINE-TUNING ===== */

/* Large screens (1920px+) */
@media (min-width: 1920px) {
  .hero-title {
    font-size: 6rem;
  }

  .day-grid {
    max-width: 1600px;
    gap: 2rem;
  }
}

/* Small mobile */
@media (max-width: 360px) {
  .day-grid {
    grid-template-columns: 1fr;
    gap: 0.75rem;
  }

  .day-card {
    aspect-ratio: 16 / 9;
  }
}

/* Tablet landscape */
@media (min-width: 768px) and (max-width: 1023px) and (orientation: landscape) {
  .day-grid {
    grid-template-columns: repeat(3, 1fr);
  }

  .hero-section {
    padding-top: 2rem;
    padding-bottom: 1.5rem;
  }
}

/* Print */
@media print {
  .navbar,
  .video-container,
  .overlay-mask,
  .theme-switcher {
    display: none !important;
  }

  .main-content {
    padding-top: 0;
  }

  .day-card {
    break-inside: avoid;
    page-break-inside: avoid;
  }
}
</style>
