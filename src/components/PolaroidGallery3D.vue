<script setup>
import { ref, computed, onMounted, onUnmounted, nextTick, watch } from 'vue'

const props = defineProps({
  audioEnabled: {
    type: Boolean,
    default: true
  }
})

const emit = defineEmits(['sparkleSound', 'burstConfetti', 'goToVideos', 'backToMessage'])

// Scan all image files in /public/Fotos/ or /public/fotos/ dynamically
const imageModules = import.meta.glob([
  '/public/Fotos/*.{jpg,jpeg,png,webp,gif,JPG,JPEG,PNG}'
], {
  eager: true
})

// Palette of tape colors and organic angles for dynamic polaroids
const tapeColors = [
  'rgba(255, 218, 225, 0.85)',
  'rgba(255, 235, 205, 0.85)',
  'rgba(240, 220, 235, 0.85)',
  'rgba(255, 215, 220, 0.85)',
  'rgba(255, 240, 210, 0.85)',
  'rgba(245, 225, 230, 0.85)',
  'rgba(255, 228, 225, 0.85)',
  'rgba(255, 238, 215, 0.85)'
]
const tiltAngles = [-2.8, 3.2, -1.6, 2.7, -3.1, 1.9, -2.4, 3.0, -1.8, 2.5, -2.2, 3.4]
const tapeAngles = [-3, 2, -2, 4, -4, 1, -3, 3]

// Dynamic list of photos populated automatically from /public/Fotos/
const allPhotos = computed(() => {
  return Object.keys(imageModules)
    .filter(filePath => !filePath.includes('favicon') && !filePath.includes('icons'))
    .map((filePath, index) => {
      const cleanPath = filePath.replace(/^\/public/, '')
      const mod = imageModules[filePath]
      const src = (mod && typeof mod.default === 'string') 
        ? mod.default 
        : (typeof mod === 'string' ? mod : encodeURI(cleanPath))

      return {
        id: index + 1,
        src,
        tilt: tiltAngles[index % tiltAngles.length],
        tapeColor: tapeColors[index % tapeColors.length],
        tapeAngle: tapeAngles[index % tapeAngles.length]
      }
    })
})

// --- Progressive Scroll Reveal System for Photos ---
// Initial rows: show 8 photos (2 rows of 4)
const visibleCount = ref(8)
const batchSize = 4 // Reveal 1 full row (4 photos) at a time
const isLoadingNext = ref(false)
const sentinelRef = ref(null)
let observer = null

const visiblePhotos = computed(() => {
  return allPhotos.value.slice(0, visibleCount.value)
})

const isAllLoaded = computed(() => {
  return visibleCount.value >= allPhotos.value.length
})

const revealNextRow = () => {
  if (isLoadingNext.value || isAllLoaded.value) return
  isLoadingNext.value = true

  setTimeout(() => {
    visibleCount.value = Math.min(allPhotos.value.length, visibleCount.value + batchSize)
    isLoadingNext.value = false
    emit('sparkleSound')
    nextTick(() => {
      checkScrollNearBottom()
    })
  }, 180)
}

// Check if user is scrolled near bottom to reveal more photos automatically
const checkScrollNearBottom = () => {
  if (isAllLoaded.value || isLoadingNext.value) return
  if (!sentinelRef.value) return

  const rect = sentinelRef.value.getBoundingClientRect()
  if (rect.top <= window.innerHeight + 450) {
    revealNextRow()
  }
}

const handleWindowScroll = () => {
  checkScrollNearBottom()
}

// Setup IntersectionObserver on the sentinel element
const setupObserver = () => {
  if (observer) observer.disconnect()
  if (!sentinelRef.value) return

  observer = new IntersectionObserver((entries) => {
    const [entry] = entries
    if (entry.isIntersecting && !isAllLoaded.value && !isLoadingNext.value) {
      revealNextRow()
    }
  }, {
    root: null,
    rootMargin: '450px 0px 450px 0px',
    threshold: 0
  })

  observer.observe(sentinelRef.value)
}

// --- Interactive 3D Tilt for Individual Polaroids ---
const cardHoverStates = ref({})

const handleMouseMove = (event, id) => {
  const card = event.currentTarget
  const rect = card.getBoundingClientRect()
  const x = event.clientX - rect.left
  const y = event.clientY - rect.top
  const centerX = rect.width / 2
  const centerY = rect.height / 2

  const rotateX = ((y - centerY) / centerY) * -12
  const rotateY = ((x - centerX) / centerX) * 12
  const glareX = (x / rect.width) * 100
  const glareY = (y / rect.height) * 100

  cardHoverStates.value[id] = {
    transform: `perspective(1000px) rotateX(${rotateX.toFixed(2)}deg) rotateY(${rotateY.toFixed(2)}deg) translateZ(32px) scale3d(1.05, 1.05, 1.05)`,
    glare: `radial-gradient(circle at ${glareX}% ${glareY}%, rgba(255,255,255,0.45) 0%, rgba(255,255,255,0) 65%)`
  }
}

const handleMouseLeave = (id) => {
  cardHoverStates.value[id] = null
}

watch(isAllLoaded, (newVal) => {
  if (newVal) {
    emit('sparkleSound')
    emit('burstConfetti')
  }
})

onMounted(() => {
  nextTick(() => {
    setupObserver()
    window.addEventListener('scroll', handleWindowScroll, { passive: true })
    checkScrollNearBottom()
  })
})

onUnmounted(() => {
  if (observer) observer.disconnect()
  window.removeEventListener('scroll', handleWindowScroll)
})
</script>

<template>
  <section class="polaroid-gallery-section" id="galeria-recuerdos">

    <!-- Gallery Header -->
    <div class="gallery-header">
      <div class="gallery-badge">
        <span class="camera-icon">📸</span>
        <span>ÁLBUM DE FOTOS</span>
        <span class="sparkle-dot">✦</span>
      </div>

      <h2 class="gallery-main-title">Nuestra Galería de Recuerdos</h2>
    </div>

    <!-- 3D Polaroid Grid Container -->
    <div class="polaroid-3d-stage">
      <div class="polaroid-grid">
        <article
          v-for="(photo, index) in visiblePhotos"
          :key="photo.id"
          class="polaroid-wrapper"
          :style="{
            '--organic-tilt': `${photo.tilt}deg`,
            '--stagger-delay': `${(index % 4) * 0.08}s`,
            '--tape-color': photo.tapeColor,
            '--tape-angle': `${photo.tapeAngle}deg`
          }"
        >
          <!-- 3D Interactive Card Frame -->
          <div 
            class="polaroid-card"
            :style="cardHoverStates[photo.id]?.transform ? { transform: cardHoverStates[photo.id].transform } : {}"
            @mousemove="handleMouseMove($event, photo.id)"
            @mouseleave="handleMouseLeave(photo.id)"
          >
            <!-- Washi Tape Strip at Top -->
            <div class="washi-tape">
              <span class="tape-shine"></span>
            </div>

            <!-- Instant Photo Area with Sheen -->
            <div class="photo-viewport">
              <img 
                :src="photo.src" 
                :alt="`Foto ${photo.id}`"
                class="photo-img"
                loading="lazy"
              />

              <!-- Authentic Instant Film Sheen / Glare -->
              <div 
                class="film-glare-overlay"
                :style="cardHoverStates[photo.id]?.glare ? { background: cardHoverStates[photo.id].glare } : {}"
              ></div>

              <!-- Vignette & Vintage Texture -->
              <div class="vintage-film-texture"></div>
            </div>

            <!-- Soft multi-layered 3D shadow under card -->
            <div class="polaroid-ambient-shadow"></div>
          </div>
        </article>
      </div>
    </div>

    <!-- Scroll Sentinel & Loader State -->
    <div ref="sentinelRef" class="scroll-sentinel-area">
      <!-- Loading indicator while next row of photos is revealing -->
      <transition name="fade">
        <div v-if="isLoadingNext" class="loading-more-pill">
          <span class="loader-spinner"></span>
          <span class="loader-text">Cargando más fotos...</span>
        </div>
      </transition>

      <!-- All photos completed: Button to transition to dedicated video theater -->
      <transition name="pop-in">
        <button v-if="isAllLoaded" class="btn-open-letter-pill" @click="$emit('goToVideos')">
          <span class="btn-heart">♡</span>
          <span class="btn-caption">Videos Especiales</span>
          <span class="btn-heart">♡</span>
        </button>
      </transition>

    </div>
  </section>
</template>

<style scoped>
/* ==========================================================================
   POLAROID GALLERY 3D SECTION
   ========================================================================== */
.polaroid-gallery-section {
  position: relative;
  width: 100%;
  max-width: 1140px;
  margin: 0.5rem auto 2.5rem auto;
  padding: 0 1rem;
  box-sizing: border-box;
  display: flex;
  flex-direction: column;
  align-items: center;
  z-index: 10;
  animation: fadeIn 0.5s ease both;
}

@keyframes fadeIn {
  from { opacity: 0; transform: translateY(12px); }
  to { opacity: 1; transform: translateY(0); }
}

/* Top Nav */
.gallery-top-nav {
  width: 100%;
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 0.5rem 0.5rem 1.5rem 0.5rem;
  box-sizing: border-box;
}

.btn-back-message, .btn-direct-video {
  display: inline-flex;
  align-items: center;
  gap: 0.5rem;
  background: rgba(255, 255, 255, 0.12);
  border: 1px solid rgba(255, 255, 255, 0.28);
  color: #ffffff;
  padding: 0.55rem 1.2rem;
  border-radius: 9999px;
  font-family: 'Montserrat', sans-serif;
  font-size: 0.84rem;
  font-weight: 600;
  cursor: pointer;
  backdrop-filter: blur(10px);
  -webkit-backdrop-filter: blur(10px);
  transition: all 0.25s ease;
  box-shadow: 0 4px 15px rgba(0, 0, 0, 0.15);
}

.btn-back-message:hover {
  background: rgba(255, 255, 255, 0.22);
  transform: translateX(-3px);
}

.btn-direct-video {
  background: linear-gradient(135deg, rgba(255, 117, 140, 0.25) 0%, rgba(255, 215, 0, 0.2) 100%);
  border-color: rgba(255, 215, 225, 0.4);
}

.btn-direct-video:hover {
  background: linear-gradient(135deg, rgba(255, 117, 140, 0.4) 0%, rgba(255, 215, 0, 0.35) 100%);
  transform: translateX(3px);
  box-shadow: 0 6px 20px rgba(255, 117, 140, 0.3);
}

/* Header & Typography */
.gallery-header {
  text-align: center;
  margin-bottom: 2.8rem;
  display: flex;
  flex-direction: column;
  align-items: center;
  width: 100%;
}

.gallery-badge {
  display: inline-flex;
  align-items: center;
  gap: 0.5rem;
  padding: 0.35rem 1rem;
  border-radius: 9999px;
  background: rgba(255, 255, 255, 0.14);
  border: 1px solid rgba(255, 255, 255, 0.32);
  backdrop-filter: blur(12px);
  -webkit-backdrop-filter: blur(12px);
  font-family: 'Montserrat', sans-serif;
  font-size: 0.76rem;
  font-weight: 600;
  letter-spacing: 0.14em;
  color: #ffe6ec;
  text-transform: uppercase;
  margin-bottom: 1rem;
  box-shadow: 0 4px 15px rgba(0, 0, 0, 0.12);
}

.camera-icon {
  font-size: 0.95rem;
}

.sparkle-dot {
  color: #ffd54f;
}

.gallery-main-title {
  font-family: 'Playfair Display', Georgia, serif;
  font-size: clamp(2rem, 4.8vw, 3.4rem);
  font-weight: 800;
  color: #ffffff;
  margin: 0 0 0.6rem 0;
  line-height: 1.2;
  text-shadow: 0 4px 20px rgba(50, 10, 20, 0.45);
}

/* ==========================================================================
   3D POLAROID STAGE & GRID
   ========================================================================== */
.polaroid-3d-stage {
  width: 100%;
  perspective: 1400px;
}

.polaroid-grid {
  display: grid;
  grid-template-columns: repeat(4, 1fr);
  gap: 2rem 1.4rem;
  width: 100%;
  justify-items: center;
  padding: 1.2rem 0.5rem 2.5rem 0.5rem;
}

/* Single Polaroid Wrapper */
.polaroid-wrapper {
  position: relative;
  width: 100%;
  max-width: 250px;
  animation: polaroidEntrance 0.75s cubic-bezier(0.2, 0.8, 0.2, 1) both;
  animation-delay: var(--stagger-delay);
  transform-style: preserve-3d;
}

@keyframes polaroidEntrance {
  0% {
    opacity: 0;
    transform: translateY(45px) rotateX(20deg) scale(0.92);
  }
  100% {
    opacity: 1;
    transform: translateY(0) rotateX(0deg) scale(1);
  }
}

/* Authentic Polaroid Frame Card */
.polaroid-card {
  position: relative;
  width: 100%;
  background: #fdfbf7;
  border-radius: 6px;
  padding: 10px 10px 24px 10px;
  box-sizing: border-box;
  cursor: default;
  outline: none;
  border: 1px solid rgba(255, 255, 255, 0.8);
  box-shadow: 
    0 10px 24px rgba(40, 10, 18, 0.32),
    0 3px 8px rgba(0, 0, 0, 0.12),
    inset 0 0 0 1px rgba(0, 0, 0, 0.04);
  transform: rotateZ(var(--organic-tilt));
  transform-style: preserve-3d;
  transition: transform 0.28s cubic-bezier(0.25, 1, 0.5, 1), box-shadow 0.28s ease;
  user-select: none;
}

.polaroid-card:hover,
.polaroid-card:focus-visible {
  z-index: 25;
  box-shadow: 
    0 20px 45px rgba(35, 5, 15, 0.5),
    0 6px 16px rgba(0, 0, 0, 0.22),
    0 0 20px rgba(255, 180, 200, 0.3);
}

/* Washi Tape Strip on Top */
.washi-tape {
  position: absolute;
  top: -11px;
  left: 50%;
  transform: translateX(-50%) rotate(var(--tape-angle));
  width: 72px;
  height: 20px;
  background: var(--tape-color, rgba(255, 220, 225, 0.85));
  backdrop-filter: blur(4px);
  border-left: 2px dashed rgba(0, 0, 0, 0.12);
  border-right: 2px dashed rgba(0, 0, 0, 0.12);
  box-shadow: 0 2px 5px rgba(0, 0, 0, 0.15);
  z-index: 10;
  opacity: 0.92;
  transition: transform 0.25s ease;
}

.polaroid-card:hover .washi-tape {
  transform: translateX(-50%) rotate(calc(var(--tape-angle) * 1.5)) scale(1.05);
}

.tape-shine {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 50%;
  background: linear-gradient(180deg, rgba(255, 255, 255, 0.4) 0%, transparent 100%);
  pointer-events: none;
}

/* Photo Viewport (Square Aspect Ratio) */
.photo-viewport {
  position: relative;
  width: 100%;
  aspect-ratio: 1 / 1;
  background: #1a0f12;
  border-radius: 3px;
  overflow: hidden;
  box-shadow: inset 0 0 10px rgba(0, 0, 0, 0.3);
}

.photo-img {
  width: 100%;
  height: 100%;
  object-fit: cover;
  display: block;
  transition: transform 0.45s cubic-bezier(0.2, 0.8, 0.2, 1), filter 0.45s ease;
  filter: contrast(1.04) saturate(1.05);
}

.polaroid-card:hover .photo-img {
  transform: scale(1.06);
}

/* Dynamic Film Glare */
.film-glare-overlay {
  position: absolute;
  inset: 0;
  pointer-events: none;
  background: linear-gradient(135deg, rgba(255, 255, 255, 0.25) 0%, rgba(255, 255, 255, 0.02) 45%, transparent 60%);
  mix-blend-mode: overlay;
  transition: background 0.15s ease;
}

/* Vintage film texture grain */
.vintage-film-texture {
  position: absolute;
  inset: 0;
  box-shadow: inset 0 0 18px rgba(0, 0, 0, 0.28);
  pointer-events: none;
}

/* ==========================================================================
   SCROLL SENTINEL & PROGRESSIVE STATES
   ========================================================================== */
.scroll-sentinel-area {
  width: 100%;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  min-height: 100px;
  margin-top: 1.5rem;
}

.loading-more-pill {
  display: inline-flex;
  align-items: center;
  gap: 0.75rem;
  padding: 0.75rem 1.6rem;
  border-radius: 9999px;
  background: rgba(255, 255, 255, 0.16);
  border: 1px solid rgba(255, 255, 255, 0.35);
  backdrop-filter: blur(14px);
  box-shadow: 0 10px 25px rgba(0, 0, 0, 0.2);
}

.loader-spinner {
  width: 18px;
  height: 18px;
  border: 2px solid rgba(255, 255, 255, 0.3);
  border-top-color: #ffd54f;
  border-radius: 50%;
  animation: spinLoader 0.75s linear infinite;
}

@keyframes spinLoader {
  to { transform: rotate(360deg); }
}

.loader-text {
  font-size: 0.9rem;
  font-weight: 600;
  color: #ffffff;
  letter-spacing: 0.02em;
}



/* Celebratory Finished Banner: Leads to Videos */
.all-completed-banner {
  position: relative;
  width: 100%;
  max-width: 680px;
  background: linear-gradient(135deg, rgba(255, 255, 255, 0.18) 0%, rgba(255, 255, 255, 0.08) 100%);
  border: 1px solid rgba(255, 255, 255, 0.38);
  backdrop-filter: blur(18px);
  -webkit-backdrop-filter: blur(18px);
  border-radius: 1.8rem;
  padding: 2.2rem 1.8rem;
  text-align: center;
  box-shadow: 0 20px 50px rgba(35, 10, 18, 0.35);
  overflow: hidden;
  margin-top: 1rem;
}

.banner-sparkle-halo {
  position: absolute;
  top: -50%;
  left: -50%;
  width: 200%;
  height: 200%;
  background: radial-gradient(circle, rgba(255, 215, 0, 0.12) 0%, transparent 60%);
  pointer-events: none;
  animation: rotateHalo 15s linear infinite;
}

@keyframes rotateHalo {
  to { transform: rotate(360deg); }
}

.banner-inner {
  position: relative;
  z-index: 2;
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 0.6rem;
}

.banner-icon {
  font-size: 2.2rem;
  margin-bottom: 0.2rem;
}

.banner-title {
  font-family: 'Playfair Display', Georgia, serif;
  font-size: clamp(1.4rem, 3.2vw, 1.9rem);
  font-weight: 800;
  color: #ffffff;
  margin: 0;
  text-shadow: 0 2px 10px rgba(0, 0, 0, 0.25);
}

.banner-desc {
  font-size: 0.95rem;
  color: #ffdde3;
  line-height: 1.55;
  max-width: 520px;
  margin: 0 0 1.2rem 0;
}

.banner-actions {
  display: flex;
  flex-wrap: wrap;
  gap: 0.9rem;
  justify-content: center;
  align-items: center;
}

.btn-go-to-videos {
  display: inline-flex;
  align-items: center;
  gap: 0.6rem;
  background: linear-gradient(135deg, #ffd54f 0%, #ffb300 100%);
  color: #260812;
  border: 1px solid rgba(255, 255, 255, 0.6);
  padding: 0.85rem 2rem;
  border-radius: 9999px;
  font-family: 'Montserrat', sans-serif;
  font-size: 0.96rem;
  font-weight: 700;
  cursor: pointer;
  box-shadow: 0 8px 25px rgba(255, 179, 0, 0.5);
  transition: all 0.25s ease;
}

.btn-go-to-videos:hover {
  transform: translateY(-2px) scale(1.04);
  box-shadow: 0 12px 32px rgba(255, 179, 0, 0.7);
}

.btn-arrow {
  font-size: 1.15rem;
  transition: transform 0.2s ease;
}

.btn-go-to-videos:hover .btn-arrow {
  transform: translateX(3px);
}

.btn-confetti-gallery {
  background: linear-gradient(135deg, #ff758c 0%, #ff4d79 50%, #d82b57 100%);
  color: #ffffff;
  border: 1px solid rgba(255, 255, 255, 0.4);
  padding: 0.85rem 1.8rem;
  border-radius: 9999px;
  font-family: 'Montserrat', sans-serif;
  font-size: 0.92rem;
  font-weight: 700;
  cursor: pointer;
  box-shadow: 0 8px 24px rgba(216, 43, 87, 0.5);
  transition: all 0.25s ease;
}

.btn-confetti-gallery:hover {
  transform: translateY(-2px) scale(1.04);
  box-shadow: 0 12px 30px rgba(216, 43, 87, 0.7);
}

/* ==========================================================================
   TRANSITIONS
   ========================================================================== */
.fade-enter-active, .fade-leave-active {
  transition: opacity 0.3s ease;
}
.fade-enter-from, .fade-leave-to {
  opacity: 0;
}

.pop-in-enter-active {
  transition: all 0.5s cubic-bezier(0.34, 1.56, 0.64, 1);
}
.pop-in-enter-from {
  opacity: 0;
  transform: translateY(20px) scale(0.95);
}

/* Responsive Breakpoints */
@media (max-width: 1080px) {
  .polaroid-grid {
    grid-template-columns: repeat(3, 1fr);
    gap: 1.8rem 1.2rem;
  }
}

@media (max-width: 768px) {
  .polaroid-grid {
    grid-template-columns: repeat(2, 1fr);
    gap: 1.6rem 1rem;
  }

  .polaroid-wrapper {
    max-width: 230px;
  }
}

@media (max-width: 480px) {
  .polaroid-grid {
    grid-template-columns: repeat(2, 1fr);
    gap: 1.2rem 0.6rem;
  }

  .polaroid-wrapper {
    max-width: 170px;
  }

  .polaroid-card {
    padding: 7px 7px 18px 7px;
  }

  .washi-tape {
    width: 52px;
    height: 15px;
    top: -8px;
  }
}
</style>
