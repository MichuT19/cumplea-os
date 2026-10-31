<script setup>
import { computed } from 'vue'

const props = defineProps({
  audioEnabled: {
    type: Boolean,
    default: true
  }
})

const emit = defineEmits(['continueToMessage', 'sparkleSound', 'burstConfetti'])

// Scan all selfie images dynamically from /public/Selfis/ or /public/selfis/
const selfiModules = import.meta.glob([
  '/public/Selfis/*.{jpg,jpeg,png,webp,gif,JPG,JPEG,PNG}',
  '/public/selfis/*.{jpg,jpeg,png,webp,gif,JPG,JPEG,PNG}',
  '/public/Selfi/*.{jpg,jpeg,png,webp,gif,JPG,JPEG,PNG}',
  '/public/selfi/*.{jpg,jpeg,png,webp,gif,JPG,JPEG,PNG}'
], {
  eager: true
})

// Dynamic list of selfies from public/Selfis
const allSelfis = computed(() => {
  return Object.keys(selfiModules)
    .filter(filePath => !filePath.includes('favicon') && !filePath.includes('icons'))
    .map((filePath, index) => {
      const cleanPath = filePath.replace(/^\/public/, '')
      const mod = selfiModules[filePath]
      const src = (mod && typeof mod.default === 'string') 
        ? mod.default 
        : (typeof mod === 'string' ? mod : encodeURI(cleanPath))

      return {
        id: index + 1,
        src
      }
    })
})

// Split photos into 3 on top and 3 on bottom (as in the reference design)
const topRowPhotos = computed(() => {
  return allSelfis.value.slice(0, 3)
})

const bottomRowPhotos = computed(() => {
  return allSelfis.value.slice(3, 6)
})

const handleContinue = () => {
  emit('sparkleSound')
  emit('burstConfetti')
  emit('continueToMessage')
}
</script>

<template>
  <div class="special-day-container">
    <!-- Starry Romantic Atmosphere Background -->
    <div class="stars-backdrop">
      <span class="star-spec s1">✦</span>
      <span class="star-spec s2">✧</span>
      <span class="star-spec s3">⋆</span>
      <span class="star-spec s4">✦</span>
      <span class="star-spec s5">✧</span>
      <span class="star-spec s6">⋆</span>
      <span class="star-spec s7">✦</span>
      <span class="star-spec s8">✧</span>
    </div>

    <!-- TOP ROW OF 3 POLAROIDS -->
    <div class="polaroid-row top-row">
      <div 
        v-for="(photo, idx) in topRowPhotos" 
        :key="photo.id"
        class="mini-polaroid-card"
        :class="`tilt-top-${idx}`"
      >
        <div class="polaroid-photo-frame">
          <img :src="photo.src" :alt="`Selfi ${photo.id}`" class="mini-photo-img" />
        </div>
      </div>
    </div>

    <!-- CENTER HERO BLOCK -->
    <div class="special-day-hero">
      <div class="header-star-tag">
        <span class="tag-sparkle">✦</span>
        <span class="tag-text">DISFRUTA DE ESTE DÍA</span>
        <span class="tag-sparkle">✦</span>
      </div>

      <h1 class="special-day-title">Tu Día Especial</h1>
      <p class="special-day-subtitle">creado con amor para ti</p>

      <!-- Pill Button to Continue to the Message -->
      <button 
        id="btn-continuar-mensaje"
        class="btn-open-letter-pill"
        @click="handleContinue"
      >
        <span class="btn-heart">♡</span>
        <span class="btn-caption">Continuar</span>
        <span class="btn-heart">♡</span>
      </button>
    </div>

    <!-- BOTTOM ROW OF 3 POLAROIDS -->
    <div class="polaroid-row bottom-row">
      <div 
        v-for="(photo, idx) in bottomRowPhotos" 
        :key="photo.id"
        class="mini-polaroid-card"
        :class="`tilt-bottom-${idx}`"
      >
        <div class="polaroid-photo-frame">
          <img :src="photo.src" :alt="`Selfi ${photo.id}`" class="mini-photo-img" />
        </div>
      </div>
    </div>
  </div>
</template>

<style scoped>
/* ==========================================================================
   TU DÍA ESPECIAL (SPECIAL DAY INTRO SECTION)
   Inspirado exactamente en el diseño de referencia:
   3 fotos arriba, Título y botón central, 3 fotos abajo.
   ========================================================================== */
.special-day-container {
  position: relative;
  width: 100%;
  max-width: 680px;
  min-height: 82vh;
  margin: 0 auto;
  padding: 1.5rem 1rem 2rem 1rem;
  box-sizing: border-box;
  display: flex;
  flex-direction: column;
  justify-content: space-between;
  align-items: center;
  z-index: 10;
  animation: fadeInZoom 0.8s cubic-bezier(0.16, 1, 0.3, 1) both;
}

@keyframes fadeInZoom {
  0% {
    opacity: 0;
    transform: scale(0.94) translateY(20px);
  }
  100% {
    opacity: 1;
    transform: scale(1) translateY(0);
  }
}

/* Starry Atmosphere */
.stars-backdrop {
  position: absolute;
  inset: 0;
  pointer-events: none;
  overflow: hidden;
  z-index: 1;
}

.star-spec {
  position: absolute;
  color: rgba(255, 235, 240, 0.65);
  animation: twinkleStar 4s ease-in-out infinite alternate;
}

.s1 { top: 8%; left: 8%; font-size: 0.9rem; animation-delay: 0s; }
.s2 { top: 22%; right: 10%; font-size: 0.8rem; animation-delay: 1.2s; }
.s3 { top: 38%; left: 12%; font-size: 1.1rem; color: #ffd54f; animation-delay: 2s; }
.s4 { top: 48%; right: 7%; font-size: 0.75rem; animation-delay: 0.5s; }
.s5 { top: 62%; left: 6%; font-size: 0.85rem; animation-delay: 1.8s; }
.s6 { bottom: 25%; right: 14%; font-size: 1rem; color: #ffd54f; animation-delay: 2.5s; }
.s7 { bottom: 10%; left: 15%; font-size: 0.75rem; animation-delay: 3s; }
.s8 { bottom: 12%; right: 8%; font-size: 0.9rem; animation-delay: 1s; }

@keyframes twinkleStar {
  0% { opacity: 0.3; transform: scale(0.85); }
  100% { opacity: 0.9; transform: scale(1.2); }
}

/* ==========================================================================
   POLAROID ROWS (3 TOP, 3 BOTTOM)
   ========================================================================== */
.polaroid-row {
  position: relative;
  z-index: 3;
  width: 100%;
  display: flex;
  justify-content: center;
  align-items: center;
  gap: clamp(0.7rem, 2.5vw, 1.8rem);
}

.top-row {
  margin-bottom: 1.5rem;
}

.bottom-row {
  margin-top: 1.5rem;
}

/* Mini Polaroid Card */
.mini-polaroid-card {
  position: relative;
  background: #ffffff;
  padding: 7px 7px 22px 7px;
  border-radius: 5px;
  box-shadow: 
    0 10px 25px rgba(25, 5, 15, 0.45),
    0 2px 6px rgba(0, 0, 0, 0.2);
  transition: transform 0.3s cubic-bezier(0.2, 0.8, 0.2, 1), box-shadow 0.3s ease;
  cursor: pointer;
  transform-origin: center center;
}

.mini-polaroid-card:hover {
  transform: scale(1.1) rotate(0deg) !important;
  z-index: 10;
  box-shadow: 
    0 18px 35px rgba(25, 5, 15, 0.65),
    0 0 15px rgba(255, 180, 200, 0.35);
}

/* Tilts matching reference image layout */
.tilt-top-0 { transform: rotate(-5deg); }
.tilt-top-1 { transform: rotate(1deg) translateY(-4px); }
.tilt-top-2 { transform: rotate(5deg); }

.tilt-bottom-0 { transform: rotate(4deg); }
.tilt-bottom-1 { transform: rotate(-2deg) translateY(3px); }
.tilt-bottom-2 { transform: rotate(-4deg); }

.polaroid-photo-frame {
  width: clamp(80px, 20vw, 125px);
  aspect-ratio: 1 / 1.15;
  background: #1a0f14;
  border-radius: 3px;
  overflow: hidden;
}

.mini-photo-img {
  width: 100%;
  height: 100%;
  object-fit: cover;
  display: block;
  transition: transform 0.4s ease;
}

.mini-polaroid-card:hover .mini-photo-img {
  transform: scale(1.06);
}

/* ==========================================================================
   CENTER HERO BLOCK (TITLE & ACTION)
   ========================================================================== */
.special-day-hero {
  position: relative;
  z-index: 3;
  display: flex;
  flex-direction: column;
  align-items: center;
  text-align: center;
  margin: 1.2rem 0;
  width: 100%;
}

.header-star-tag {
  display: inline-flex;
  align-items: center;
  gap: 0.6rem;
  font-family: 'Montserrat', sans-serif;
  font-size: clamp(0.72rem, 1.8vw, 0.84rem);
  font-weight: 600;
  letter-spacing: 0.22em;
  color: #ffccd5;
  text-transform: uppercase;
  margin-bottom: 0.6rem;
  opacity: 0.9;
}

.tag-sparkle {
  color: #ffd54f;
  font-size: 0.75rem;
}

.special-day-title {
  font-family: 'Playfair Display', Georgia, serif;
  font-size: clamp(2.4rem, 6.5vw, 4.2rem);
  font-weight: 700;
  color: #ffffff;
  margin: 0;
  line-height: 1.15;
  letter-spacing: -0.01em;
  text-shadow: 0 4px 25px rgba(45, 10, 20, 0.6);
}

.special-day-subtitle {
  font-family: 'Playfair Display', Georgia, serif;
  font-style: italic;
  font-size: clamp(1.05rem, 2.6vw, 1.35rem);
  color: #ffdde3;
  margin: 0.4rem 0 1.8rem 0;
  opacity: 0.92;
  letter-spacing: 0.02em;
  text-shadow: 0 2px 10px rgba(0, 0, 0, 0.3);
}

/* The Reference Pill Button */
.btn-open-letter-pill {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  gap: 0.6rem;
  background: linear-gradient(135deg, #fbe3e8 0%, #ffd0db 50%, #f7b7c5 100%);
  color: #4a1523;
  border: 1px solid rgba(255, 255, 255, 0.75);
  padding: 0.85rem 2.2rem;
  border-radius: 9999px;
  font-family: 'Montserrat', sans-serif;
  font-size: clamp(0.82rem, 2vw, 0.94rem);
  font-weight: 700;
  letter-spacing: 0.12em;
  text-transform: uppercase;
  cursor: pointer;
  box-shadow: 
    0 10px 30px rgba(247, 183, 197, 0.45),
    0 4px 10px rgba(0, 0, 0, 0.15),
    inset 0 1px 2px rgba(255, 255, 255, 0.8);
  transition: all 0.28s cubic-bezier(0.2, 0.8, 0.2, 1);
  outline: none;
}

.btn-open-letter-pill:hover {
  transform: translateY(-2px) scale(1.04);
  box-shadow: 
    0 14px 38px rgba(247, 183, 197, 0.65),
    0 6px 14px rgba(0, 0, 0, 0.2),
    inset 0 1px 3px rgba(255, 255, 255, 0.95);
  background: linear-gradient(135deg, #ffffff 0%, #ffe0e7 50%, #fbc2cf 100%);
}

.btn-open-letter-pill:active {
  transform: translateY(0) scale(0.98);
}

.btn-heart {
  font-size: 1rem;
  color: #c93b58;
}

/* Responsive adjustments */
@media (max-width: 480px) {
  .special-day-container {
    padding: 1rem 0.5rem 1.5rem 0.5rem;
    min-height: 80vh;
  }

  .polaroid-photo-frame {
    width: 78px;
  }

  .mini-polaroid-card {
    padding: 5px 5px 16px 5px;
  }

  .special-day-title {
    font-size: 2.3rem;
  }

  .special-day-subtitle {
    margin-bottom: 1.4rem;
  }

  .btn-open-letter-pill {
    padding: 0.75rem 1.6rem;
    letter-spacing: 0.08em;
  }
}
</style>
