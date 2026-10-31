<script setup>
import { ref, computed, nextTick, onMounted } from 'vue'

const props = defineProps({
  audioEnabled: {
    type: Boolean,
    default: true
  }
})

const emit = defineEmits(['backToGallery', 'sparkleSound', 'burstConfetti', 'continueToWish'])

// Scan all video files in /public/Videos/, /public/videos/ or /public/ dynamically
const videoModules = import.meta.glob([
  '/public/Videos/*.{mp4,webm,mov,MP4,WEBM,MOV}',
  '/public/videos/*.{mp4,webm,mov,MP4,WEBM,MOV}',
  '/public/*.{mp4,webm,mov,MP4,WEBM,MOV}'
], {
  eager: true
})

// Dynamic list of videos populated automatically from /public/Videos/
const allVideos = computed(() => {
  return Object.keys(videoModules)
    .filter(filePath => !filePath.includes('favicon') && !filePath.includes('icons'))
    .map((filePath, index) => {
      const cleanPath = filePath.replace(/^\/public/, '')
      const mod = videoModules[filePath]
      const src = (mod && typeof mod.default === 'string') 
        ? mod.default 
        : (typeof mod === 'string' ? mod : encodeURI(cleanPath))
      const rawName = cleanPath.split('/').pop().replace(/\.[^/.]+$/, '')

      return {
        id: index + 1,
        src,
        title: `Momentos`,
        fileName: rawName,
        label: `Video ${index + 1}`
      }
    })
})

const currentVideoIndex = ref(0)
const videoPlayerRef = ref(null)
const isVideoPlaying = ref(false)

const currentVideo = computed(() => {
  if (!allVideos.value.length) return null
  return allVideos.value[currentVideoIndex.value] || allVideos.value[0]
})

const selectVideo = (index) => {
  if (index < 0 || index >= allVideos.value.length) return
  currentVideoIndex.value = index
  emit('sparkleSound')
  nextTick(() => {
    if (videoPlayerRef.value) {
      videoPlayerRef.value.load()
      videoPlayerRef.value.play().then(() => {
        isVideoPlaying.value = true
      }).catch(() => {
        isVideoPlaying.value = false
      })
    }
  })
}

const onVideoPlay = () => {
  isVideoPlaying.value = true
}

const onVideoPause = () => {
  isVideoPlaying.value = false
}

const onVideoEnded = () => {
  isVideoPlaying.value = false
  // Auto-advance to next video if more than one
  if (allVideos.value.length > 1) {
    const nextIdx = (currentVideoIndex.value + 1) % allVideos.value.length
    selectVideo(nextIdx)
  }
}

onMounted(() => {
  emit('sparkleSound')
  emit('burstConfetti')
})
</script>

<template>
  <div class="video-theater-wrapper">
    <!-- Top Navigation Bar -->
    <header class="theater-top-bar">

      <div class="top-bar-placeholder"></div>
    </header>

    <!-- Main Theater Section -->
    <section class="video-theater-content">
      <!-- Ambient Backlight Glow -->
      <div class="theater-ambient-halo"></div>

      <!-- Section Title & Subtitle -->
      <div class="theater-header-block">
        <div class="theater-badge">
          <span class="badge-camera">🎥</span>
          <span>MOMENTOS JUNTOS</span>
          <span class="badge-sparkle">✦</span>
        </div>

        <h2 class="theater-title">Nuestra Historia en Video</h2>
        <p class="theater-subtitle">
          Cada risa, cada abrazo y cada instante especial que guardamos para siempre.
        </p>
      </div>

      <!-- Framed Central Integrated Player -->
      <div class="framed-player-container">
        <div class="framed-player-card">
          <!-- Frame Topbar with Luxury Hardware Dots -->
          <div class="player-frame-topbar">
            <div class="frame-hardware-dots">
              <span class="hw-dot dot-rose"></span>
              <span class="hw-dot dot-gold"></span>
              <span class="hw-dot dot-cyan"></span>
            </div>

            <div class="frame-topbar-title">
              <span class="play-indicator-dot" :class="{ 'is-active': isVideoPlaying }"></span>
              <span class="topbar-video-name">{{ currentVideo?.title || 'Video Especial' }}</span>
            </div>

            <div class="frame-topbar-badge">
              <span class="badge-quality">HD</span>
              <span class="badge-counter">{{ currentVideoIndex + 1 }} / {{ allVideos.length }}</span>
            </div>
          </div>

          <!-- Central Framed Screen Viewport -->
          <div class="player-viewport">
            <video
              ref="videoPlayerRef"
              :src="currentVideo?.src"
              class="integrated-video-element"
              controls
              playsinline
              preload="metadata"
              @play="onVideoPlay"
              @pause="onVideoPause"
              @ended="onVideoEnded"
            >
              Tu navegador no soporta la reproducción de video HTML5.
            </video>
          </div>

          <!-- Frame Bottom Bar with Quick Controls -->
          <div class="player-frame-bottombar">
            <button 
              v-if="allVideos.length > 1" 
              class="btn-nav-video"
              :disabled="currentVideoIndex === 0"
              @click="selectVideo(currentVideoIndex - 1)"
              title="Video anterior"
            >
              <span>⏮</span> Anterior
            </button>

            <button 
              v-if="allVideos.length > 1" 
              class="btn-nav-video"
              :disabled="currentVideoIndex === allVideos.length - 1"
              @click="selectVideo(currentVideoIndex + 1)"
              title="Siguiente video"
            >
              Siguiente <span>⏭</span>
            </button>
          </div>
        </div>
      </div>

      <!-- Continuar Button after the video -->
      <div class="video-continue-action">
        <button 
          id="btn-continuar-deseo"
          class="btn-open-letter-pill"
          @click="$emit('continueToWish')"
        >
          <span class="btn-heart">♡</span>
          <span class="btn-caption">Continuar</span>
          <span class="btn-heart">♡</span>
        </button>
      </div>
    </section>
  </div>
</template>

<style scoped>
/* ==========================================================================
   DEDICATED VIDEO THEATER WRAPPER
   ========================================================================== */
.video-theater-wrapper {
  position: relative;
  width: 100%;
  max-width: 1100px;
  margin: 0 auto;
  padding: 0 1rem 3rem 1rem;
  box-sizing: border-box;
  display: flex;
  flex-direction: column;
  align-items: center;
  z-index: 10;
  animation: fadeIn 0.6s ease both;
}

.video-continue-action {
  position: relative;
  z-index: 5;
  margin-top: 2.8rem;
  margin-bottom: 2rem;
  display: flex;
  justify-content: center;
  align-items: center;
  width: 100%;
}

@keyframes fadeIn {
  from { opacity: 0; transform: translateY(15px); }
  to { opacity: 1; transform: translateY(0); }
}

/* Top Navigation Bar */
.theater-top-bar {
  width: 100%;
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 1rem 0.5rem 1.5rem 0.5rem;
  box-sizing: border-box;
}

.btn-back-action {
  display: inline-flex;
  align-items: center;
  gap: 0.55rem;
  background: rgba(255, 255, 255, 0.12);
  border: 1px solid rgba(255, 255, 255, 0.28);
  color: #ffffff;
  padding: 0.6rem 1.3rem;
  border-radius: 9999px;
  font-family: 'Montserrat', sans-serif;
  font-size: 0.86rem;
  font-weight: 600;
  cursor: pointer;
  backdrop-filter: blur(10px);
  -webkit-backdrop-filter: blur(10px);
  transition: all 0.25s ease;
  box-shadow: 0 4px 15px rgba(0, 0, 0, 0.15);
}

.btn-back-action:hover {
  background: rgba(255, 255, 255, 0.24);
  transform: translateX(-3px);
  box-shadow: 0 6px 20px rgba(0, 0, 0, 0.25);
}

.back-arrow {
  font-size: 1.1rem;
  transition: transform 0.2s ease;
}

.btn-back-action:hover .back-arrow {
  transform: translateX(-3px);
}

.top-bar-center-badge {
  display: inline-flex;
  align-items: center;
  gap: 0.45rem;
  padding: 0.35rem 1rem;
  border-radius: 9999px;
  background: rgba(255, 255, 255, 0.08);
  border: 1px solid rgba(255, 255, 255, 0.2);
  font-family: 'Montserrat', sans-serif;
  font-size: 0.74rem;
  font-weight: 700;
  letter-spacing: 0.14em;
  color: #ffccd5;
}

.film-icon {
  font-size: 0.85rem;
}

.top-bar-placeholder {
  width: 140px;
}

@media (max-width: 640px) {
  .top-bar-placeholder {
    display: none;
  }
}

/* Main Content Area */
.video-theater-content {
  position: relative;
  width: 100%;
  display: flex;
  flex-direction: column;
  align-items: center;
}

.theater-ambient-halo {
  position: absolute;
  top: 15%;
  left: 50%;
  transform: translate(-50%, 0);
  width: 90%;
  height: 65%;
  background: radial-gradient(ellipse at center, rgba(255, 117, 140, 0.25) 0%, rgba(255, 215, 0, 0.1) 45%, transparent 75%);
  filter: blur(55px);
  pointer-events: none;
  z-index: 1;
}

/* Header block */
.theater-header-block {
  position: relative;
  z-index: 2;
  text-align: center;
  margin-bottom: 2.2rem;
  display: flex;
  flex-direction: column;
  align-items: center;
}

.theater-badge {
  display: inline-flex;
  align-items: center;
  gap: 0.5rem;
  padding: 0.38rem 1.15rem;
  border-radius: 9999px;
  background: rgba(255, 255, 255, 0.15);
  border: 1px solid rgba(255, 255, 255, 0.35);
  backdrop-filter: blur(14px);
  -webkit-backdrop-filter: blur(14px);
  font-family: 'Montserrat', sans-serif;
  font-size: 0.78rem;
  font-weight: 700;
  letter-spacing: 0.15em;
  color: #ffe6ec;
  text-transform: uppercase;
  margin-bottom: 0.8rem;
  box-shadow: 0 4px 18px rgba(0, 0, 0, 0.18);
}

.badge-camera {
  font-size: 0.95rem;
}

.badge-sparkle {
  color: #ffd54f;
}

.theater-title {
  font-family: 'Playfair Display', Georgia, serif;
  font-size: clamp(2rem, 4.4vw, 3.2rem);
  font-weight: 800;
  color: #ffffff;
  margin: 0 0 0.6rem 0;
  line-height: 1.2;
  text-shadow: 0 4px 25px rgba(45, 10, 20, 0.55);
}

.theater-subtitle {
  font-family: 'Montserrat', sans-serif;
  font-size: clamp(0.92rem, 1.8vw, 1.08rem);
  color: #ffdde3;
  max-width: 640px;
  margin: 0 auto;
  line-height: 1.6;
  opacity: 0.92;
}

/* Framed Player Container & Outer Card */
.framed-player-container {
  position: relative;
  z-index: 3;
  width: 100%;
  display: flex;
  justify-content: center;
  margin-bottom: 2.2rem;
}

.framed-player-card {
  position: relative;
  width: 100%;
  max-width: 940px;
  background: linear-gradient(160deg, rgba(35, 12, 22, 0.94) 0%, rgba(18, 6, 12, 0.97) 100%);
  border-radius: 20px;
  padding: 12px;
  box-sizing: border-box;
  border: 2px solid rgba(255, 215, 225, 0.45);
  box-shadow: 
    0 30px 80px rgba(0, 0, 0, 0.65),
    0 0 45px rgba(255, 117, 140, 0.28),
    inset 0 1px 2px rgba(255, 255, 255, 0.45);
  backdrop-filter: blur(20px);
  -webkit-backdrop-filter: blur(20px);
  display: flex;
  flex-direction: column;
  overflow: hidden;
  transition: transform 0.3s ease, box-shadow 0.3s ease;
}

.framed-player-card:hover {
  box-shadow: 
    0 35px 90px rgba(0, 0, 0, 0.7),
    0 0 55px rgba(255, 117, 140, 0.35),
    inset 0 1px 3px rgba(255, 255, 255, 0.6);
}

/* Frame Top Bar */
.player-frame-topbar {
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 0.65rem 1rem;
  background: rgba(255, 255, 255, 0.05);
  border-radius: 12px 12px 0 0;
  margin-bottom: 8px;
  border-bottom: 1px solid rgba(255, 255, 255, 0.08);
}

.frame-hardware-dots {
  display: flex;
  align-items: center;
  gap: 7px;
}

.hw-dot {
  width: 11px;
  height: 11px;
  border-radius: 50%;
  display: inline-block;
  box-shadow: 0 0 6px currentColor;
}

.dot-rose {
  background: #ff4d79;
  color: #ff4d79;
}

.dot-gold {
  background: #ffd54f;
  color: #ffd54f;
}

.dot-cyan {
  background: #4fc3f7;
  color: #4fc3f7;
}

.frame-topbar-title {
  display: flex;
  align-items: center;
  gap: 0.55rem;
  font-family: 'Montserrat', sans-serif;
  font-size: 0.88rem;
  font-weight: 600;
  color: #ffffff;
  letter-spacing: 0.03em;
}

.play-indicator-dot {
  width: 8px;
  height: 8px;
  border-radius: 50%;
  background: #9e9e9e;
  transition: all 0.3s ease;
}

.play-indicator-dot.is-active {
  background: #00e676;
  box-shadow: 0 0 10px #00e676;
  animation: pulseGreen 1.8s infinite;
}

@keyframes pulseGreen {
  0%, 100% { opacity: 1; transform: scale(1); }
  50% { opacity: 0.5; transform: scale(0.85); }
}

.topbar-video-name {
  color: #fff;
  font-weight: 600;
}

.frame-topbar-badge {
  display: flex;
  align-items: center;
  gap: 0.5rem;
  font-size: 0.76rem;
  color: #ffccd5;
}

.badge-quality {
  padding: 2px 7px;
  border-radius: 4px;
  background: rgba(255, 215, 0, 0.2);
  border: 1px solid rgba(255, 215, 0, 0.5);
  color: #ffd54f;
  font-weight: 700;
}

.badge-counter {
  font-weight: 600;
}

/* Central Video Viewport (En Grande) */
.player-viewport {
  position: relative;
  width: 100%;
  aspect-ratio: 16 / 9;
  max-height: 520px;
  background: #000000;
  border-radius: 10px;
  overflow: hidden;
  display: flex;
  align-items: center;
  justify-content: center;
  box-shadow: inset 0 0 30px rgba(0, 0, 0, 0.95);
}

.integrated-video-element {
  width: 100%;
  height: 100%;
  object-fit: contain;
  background: #000000;
  outline: none;
  border-radius: 10px;
}

/* Bottom Bar within frame */
.player-frame-bottombar {
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 0.75rem 1rem 0.25rem 1rem;
  margin-top: 6px;
}

.btn-nav-video {
  display: inline-flex;
  align-items: center;
  gap: 0.4rem;
  background: rgba(255, 255, 255, 0.1);
  border: 1px solid rgba(255, 255, 255, 0.25);
  color: #ffffff;
  padding: 0.45rem 1.1rem;
  border-radius: 9999px;
  font-family: 'Montserrat', sans-serif;
  font-size: 0.82rem;
  font-weight: 600;
  cursor: pointer;
  transition: all 0.2s ease;
}

.btn-nav-video:disabled {
  opacity: 0.35;
  cursor: not-allowed;
}

.btn-nav-video:hover:not(:disabled) {
  background: rgba(255, 255, 255, 0.22);
  transform: translateY(-1px);
}

.playing-status-text {
  display: flex;
  align-items: center;
  gap: 0.45rem;
  font-size: 0.82rem;
  color: #ffccd5;
}

.status-icon {
  color: #ffd54f;
  font-size: 0.85rem;
}

/* Playlist Strip / Chips Row */
.video-playlist-container {
  position: relative;
  z-index: 3;
  width: 100%;
  max-width: 940px;
  margin-bottom: 2.2rem;
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 0.9rem;
}

.playlist-heading {
  font-size: 0.88rem;
  color: #ffdde3;
  font-weight: 600;
  letter-spacing: 0.04em;
  margin: 0;
}

.playlist-chips-row {
  display: flex;
  flex-wrap: wrap;
  gap: 0.9rem;
  justify-content: center;
  width: 100%;
}

.playlist-chip {
  display: flex;
  align-items: center;
  gap: 0.8rem;
  padding: 0.7rem 1.5rem;
  border-radius: 14px;
  background: rgba(255, 255, 255, 0.08);
  border: 1px solid rgba(255, 255, 255, 0.24);
  color: #ffffff;
  cursor: pointer;
  backdrop-filter: blur(12px);
  -webkit-backdrop-filter: blur(12px);
  transition: all 0.25s cubic-bezier(0.2, 0.8, 0.2, 1);
}

.playlist-chip:hover {
  background: rgba(255, 255, 255, 0.18);
  transform: translateY(-2px);
  box-shadow: 0 8px 24px rgba(0, 0, 0, 0.3);
}

.playlist-chip.chip-active {
  background: linear-gradient(135deg, rgba(255, 117, 140, 0.35) 0%, rgba(216, 43, 87, 0.38) 100%);
  border-color: rgba(255, 215, 225, 0.85);
  box-shadow: 0 0 25px rgba(255, 117, 140, 0.45);
}

.chip-play-icon {
  font-size: 1.15rem;
}

.chip-info {
  display: flex;
  flex-direction: column;
  align-items: flex-start;
  text-align: left;
}

.chip-title {
  font-size: 0.88rem;
  font-weight: 700;
  color: #ffffff;
}

.chip-status {
  font-size: 0.72rem;
  color: #ffd54f;
  font-weight: 600;
}

/* Bottom Actions */
.theater-bottom-actions {
  position: relative;
  z-index: 3;
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 1rem;
  margin-top: 0.5rem;
}

.btn-confetti-action {
  background: linear-gradient(135deg, #ff758c 0%, #ff4d79 50%, #d82b57 100%);
  color: #ffffff;
  border: 1px solid rgba(255, 255, 255, 0.4);
  padding: 0.85rem 2rem;
  border-radius: 9999px;
  font-family: 'Montserrat', sans-serif;
  font-size: 0.95rem;
  font-weight: 700;
  cursor: pointer;
  box-shadow: 0 8px 25px rgba(216, 43, 87, 0.55);
  transition: all 0.25s ease;
}

.btn-confetti-action:hover {
  transform: translateY(-2px) scale(1.04);
  box-shadow: 0 12px 32px rgba(216, 43, 87, 0.75);
}

.btn-back-text {
  background: none;
  border: none;
  color: #ffccd5;
  font-family: 'Montserrat', sans-serif;
  font-size: 0.85rem;
  font-weight: 600;
  cursor: pointer;
  text-decoration: underline;
  opacity: 0.85;
  transition: opacity 0.2s, transform 0.2s;
}

.btn-back-text:hover {
  opacity: 1;
  transform: translateX(-2px);
}

/* Responsive */
@media (max-width: 768px) {
  .framed-player-card {
    padding: 8px;
    border-radius: 16px;
  }

  .player-frame-topbar {
    padding: 0.5rem 0.75rem;
  }

  .player-frame-bottombar {
    flex-wrap: wrap;
    gap: 0.5rem;
    justify-content: center;
  }
}

@media (max-width: 480px) {
  .theater-title {
    font-size: 1.75rem;
  }

  .btn-confetti-action {
    width: 100%;
    padding: 0.8rem 1.4rem;
    font-size: 0.88rem;
  }
}
</style>
