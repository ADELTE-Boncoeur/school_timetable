<template>
  <div class="desktop">
    <!-- Clock -->
    <div id="os-clock" class="os-clock">{{ currentTime }}</div>

    <!-- Video Container -->
    <div id="video-container" class="video-container">
      <div id="player"></div>
    </div>

    <!-- Overlay Mask -->
    <div class="overlay-mask"></div>

    <!-- Header -->
    <div class="header-section">
      <h1>L3SOD<span style="color: var(--accent)">.B</span></h1>
      <p style="letter-spacing: 8px; font-size: 0.8rem; opacity: 0.7; text-align: center;">TERMINAL SCHEDULE SYSTEM</p>
    </div>

    <!-- Grid -->
    <div class="day-grid" id="main-grid">
      <div
        v-for="(info, day) in timetable"
        :key="day"
        class="day-card"
        @click="openModal(day, info)"
      >
        <img :src="info.imgUrl" class="day-img" :alt="day" />
        <div class="day-label">{{ day }}</div>
      </div>
    </div>

    <!-- Modal -->
    <div v-if="modalOpen" class="modal-overlay" @click="closeModal">
      <div class="modal-card" @click.stop>
        <h2 id="mDay" style="color: var(--accent); margin-top: 0; font-size: 2.5rem">{{ selectedDay }}</h2>
        <div id="mContent">
          <div
            v-for="(sub, index) in selectedSubs"
            :key="index"
            :class="['sub-row', { 'practice-highlight': isPractice(sub) }]"
          >
            <span>{{ sub }}</span>
            <span v-if="isPractice(sub)" class="badge">PRACTICE</span>
          </div>
        </div>
        <button @click="closeModal">DISCONNECT</button>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted, onBeforeUnmount } from 'vue'

const currentTime = ref('00:00:00')
const modalOpen = ref(false)
const selectedDay = ref('')
const selectedSubs = ref([])
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
}

const closeModal = () => {
  modalOpen.value = false
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
})

onBeforeUnmount(() => {
  if (clockInterval) clearInterval(clockInterval)
})
</script>

<style scoped>
:root {
  --accent: #00f2ff;
  --glass: rgba(0, 0, 0, 0.65);
  --border: rgba(255, 255, 255, 0.1);
  --practice-glow: rgba(0, 242, 255, 0.3);
}

* {
  box-sizing: border-box;
}

.desktop {
  height: 100vh;
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  gap: 30px;
  backdrop-filter: blur(5px);
  position: relative;
}

.os-clock {
  position: fixed;
  top: 30px;
  right: 40px;
  font-size: 1.8rem;
  font-weight: 200;
  color: var(--accent);
  text-shadow: 0 0 15px var(--accent);
  font-family: monospace;
  z-index: 100;
}

.video-container {
  position: fixed;
  inset: 0;
  z-index: -3;
  pointer-events: none;
  overflow: hidden;
  opacity: 0.6;
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

.overlay-mask {
  position: fixed;
  inset: 0;
  background: linear-gradient(to bottom, rgba(0, 0, 0, 0.4), rgba(0, 0, 0, 0.9));
  z-index: -1;
  pointer-events: none;
}

.header-section h1 {
  font-size: 4rem;
  margin: 0;
  font-weight: 900;
  letter-spacing: -3px;
  filter: drop-shadow(0 0 10px rgba(0, 242, 255, 0.5));
}

.day-grid {
  display: grid;
  grid-template-columns: repeat(5, 1fr);
  gap: 20px;
  width: 92%;
  max-width: 1200px;
  z-index: 10;
}

.day-card {
  height: 220px;
  border-radius: 20px;
  position: relative;
  overflow: hidden;
  cursor: pointer;
  border: 1px solid var(--border);
  background: var(--glass);
  backdrop-filter: blur(10px);
  transition: all 0.4s ease;
}

.day-card:hover {
  transform: translateY(-12px);
  border-color: var(--accent);
  box-shadow: 0 15px 40px rgba(0, 242, 255, 0.3);
}

.day-img {
  position: absolute;
  inset: 0;
  width: 100%;
  height: 100%;
  object-fit: cover;
  z-index: -1;
  opacity: 0.5;
  transition: 0.5s;
}

.day-label {
  position: absolute;
  bottom: 0;
  width: 100%;
  padding: 15px 0;
  background: rgba(0, 0, 0, 0.8);
  text-align: center;
  font-weight: 900;
  letter-spacing: 3px;
  font-size: 0.9rem;
}

.modal-overlay {
  display: flex;
  position: fixed;
  inset: 0;
  background: rgba(0, 0, 0, 0.85);
  z-index: 2000;
  justify-content: center;
  align-items: center;
  backdrop-filter: blur(20px);
}

.modal-card {
  width: 450px;
  padding: 40px;
  border-radius: 30px;
  border: 1px solid var(--accent);
  background: rgba(10, 15, 25, 0.9);
  box-shadow: 0 0 60px rgba(0, 242, 255, 0.2);
}

.sub-row {
  padding: 16px;
  border-bottom: 1px solid rgba(255, 255, 255, 0.05);
  display: flex;
  justify-content: space-between;
  align-items: center;
  border-radius: 8px;
  margin-bottom: 4px;
}

.practice-highlight {
  background: linear-gradient(90deg, rgba(0, 242, 255, 0.2), transparent);
  border-left: 5px solid var(--accent);
  animation: pulse 2s infinite;
}

@keyframes pulse {
  0% {
    box-shadow: inset 0 0 10px rgba(0, 242, 255, 0.1);
  }
  50% {
    box-shadow: inset 0 0 25px rgba(0, 242, 255, 0.3);
  }
  100% {
    box-shadow: inset 0 0 10px rgba(0, 242, 255, 0.1);
  }
}

.badge {
  background: var(--accent);
  color: #000;
  padding: 4px 10px;
  font-size: 0.7rem;
  font-weight: 900;
  border-radius: 4px;
}

button {
  width: 100%;
  margin-top: 30px;
  background: var(--accent);
  border: none;
  padding: 15px;
  border-radius: 12px;
  font-weight: 900;
  cursor: pointer;
  transition: 0.3s;
  color: #000;
}

button:hover {
  letter-spacing: 2px;
  filter: brightness(1.2);
}
</style>
