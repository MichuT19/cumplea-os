<script setup>
import { ref, computed, onMounted, onUnmounted, nextTick } from 'vue'
import SpecialDayIntro from './SpecialDayIntro.vue'
import PolaroidGallery3D from './PolaroidGallery3D.vue'
import VideoTheater from './VideoTheater.vue'
import BirthdayWishSection from './BirthdayWishSection.vue'
import GalaxyHeart from './GalaxyHeart.vue'

// --- Props & Emits ---
const emit = defineEmits(['backToCatalog'])

// --- Audio Synthesis via Web Audio API ---
const audioEnabled = ref(true)
let audioCtx = null

const initAudio = () => {
  if (!audioCtx) {
    const AudioContextClass = window.AudioContext || window.webkitAudioContext
    if (AudioContextClass) {
      audioCtx = new AudioContextClass()
    }
  }
  if (audioCtx && audioCtx.state === 'suspended') {
    audioCtx.resume()
  }
  return audioCtx
}

// Gentle chime / sparkle arpeggio
const playSparkleSound = () => {
  if (!audioEnabled.value) return
  try {
    const ctx = initAudio()
    if (!ctx) return
    const notes = [523.25, 659.25, 783.99, 1046.50, 1318.51, 1567.98] // C5, E5, G5, C6, E6, G6
    notes.forEach((freq, index) => {
      const osc = ctx.createOscillator()
      const gain = ctx.createGain()
      osc.type = 'triangle'
      osc.frequency.setValueAtTime(freq, ctx.currentTime + index * 0.07)
      gain.gain.setValueAtTime(0.001, ctx.currentTime + index * 0.07)
      gain.gain.exponentialRampToValueAtTime(0.12, ctx.currentTime + index * 0.07 + 0.02)
      gain.gain.exponentialRampToValueAtTime(0.0001, ctx.currentTime + index * 0.07 + 0.45)
      osc.connect(gain)
      gain.connect(ctx.destination)
      osc.start(ctx.currentTime + index * 0.07)
      osc.stop(ctx.currentTime + index * 0.07 + 0.5)
    })
  } catch (e) {
    console.error(e)
  }
}

// Button click chime
const playButtonSound = () => {
  if (!audioEnabled.value) return
  try {
    const ctx = initAudio()
    if (!ctx) return
    const osc = ctx.createOscillator()
    const gain = ctx.createGain()
    osc.type = 'sine'
    osc.frequency.setValueAtTime(587.33, ctx.currentTime) // D5
    osc.frequency.exponentialRampToValueAtTime(880, ctx.currentTime + 0.12) // A5
    gain.gain.setValueAtTime(0.12, ctx.currentTime)
    gain.gain.exponentialRampToValueAtTime(0.001, ctx.currentTime + 0.28)
    osc.connect(gain)
    gain.connect(ctx.destination)
    osc.start()
    osc.stop(ctx.currentTime + 0.28)
  } catch (e) {
    console.error(e)
  }
}

// Soft typewriter key tap
const playTypewriterSound = () => {
  if (!audioEnabled.value) return
  try {
    const ctx = initAudio()
    if (!ctx) return
    const osc = ctx.createOscillator()
    const gain = ctx.createGain()
    osc.type = 'sine'
    const randomFreq = 750 + Math.random() * 350
    osc.frequency.setValueAtTime(randomFreq, ctx.currentTime)
    gain.gain.setValueAtTime(0.025, ctx.currentTime)
    gain.gain.exponentialRampToValueAtTime(0.0001, ctx.currentTime + 0.04)
    osc.connect(gain)
    gain.connect(ctx.destination)
    osc.start()
    osc.stop(ctx.currentTime + 0.04)
  } catch (e) {
    console.error(e)
  }
}

// Fanfare celebration sound
const playCelebrationFanfare = () => {
  if (!audioEnabled.value) return
  try {
    const ctx = initAudio()
    if (!ctx) return
    const chords = [
      { f: [523.25, 659.25, 783.99], t: 0, d: 0.25 },
      { f: [587.33, 739.99, 880.00], t: 0.2, d: 0.25 },
      { f: [659.25, 830.61, 987.77], t: 0.4, d: 0.25 },
      { f: [783.99, 987.77, 1318.51], t: 0.62, d: 0.6 },
      { f: [1046.50, 1318.51, 1567.98], t: 0.9, d: 1.0 }
    ]
    chords.forEach(({ f, t, d }) => {
      f.forEach(freq => {
        const osc = ctx.createOscillator()
        const gain = ctx.createGain()
        osc.type = 'triangle'
        osc.frequency.setValueAtTime(freq, ctx.currentTime + t)
        gain.gain.setValueAtTime(0.001, ctx.currentTime + t)
        gain.gain.exponentialRampToValueAtTime(0.07, ctx.currentTime + t + 0.03)
        gain.gain.exponentialRampToValueAtTime(0.0001, ctx.currentTime + t + d)
        osc.connect(gain)
        gain.connect(ctx.destination)
        osc.start(ctx.currentTime + t)
        osc.stop(ctx.currentTime + t + d + 0.05)
      })
    })
  } catch (e) {
    console.error(e)
  }
}

// --- Active Stage / Page Controller ---
// 1 = Countdown timer page
// 2 = Gift opening animation
// 3 = Happy birthday & typewriter card
const currentStage = ref(1)

// --- Stage 1: Countdown Timer Logic ---
const getTargetBirthday = () => {
  const current = new Date()
  let targetYear = current.getFullYear()
  // Cumpleaños: 1 de Noviembre (mes 10 en base 0)
  let bday = new Date(targetYear, 10, 1, 0, 0, 0)
  // Si ya pasó este año más de 24 horas, apuntar al siguiente año
  if (current.getTime() - bday.getTime() > 24 * 60 * 60 * 1000) {
    bday = new Date(targetYear + 1, 10, 1, 0, 0, 0)
  }
  return bday
}

const defaultTarget = getTargetBirthday()

// Formato local YYYY-MM-DDTHH:mm para datetime-local
const formatLocalDate = (d) => {
  const pad = (n) => String(n).padStart(2, '0')
  return `${d.getFullYear()}-${pad(d.getMonth() + 1)}-${pad(d.getDate())}T${pad(d.getHours())}:${pad(d.getMinutes())}`
}

const targetDate = ref(formatLocalDate(defaultTarget))
const showDatePicker = ref(false)

const remainingTime = ref({
  days: 0,
  hours: 0,
  minutes: 0,
  seconds: 0
})

let timerInterval = null

const updateCountdown = () => {
  const target = new Date(targetDate.value).getTime()
  const current = new Date().getTime()
  const diff = target - current

  if (diff <= 0) {
    remainingTime.value = { days: 0, hours: 0, minutes: 0, seconds: 0 }
    return
  }

  const d = Math.floor(diff / (1000 * 60 * 60 * 24))
  const h = Math.floor((diff % (1000 * 60 * 60 * 24)) / (1000 * 60 * 60))
  const m = Math.floor((diff % (1000 * 60 * 60)) / (1000 * 60))
  const s = Math.floor((diff % (1000 * 60)) / 1000)

  remainingTime.value = {
    days: d,
    hours: h,
    minutes: m,
    seconds: s
  }
}

const padZero = (num) => String(num).padStart(2, '0')

// --- Navigation to Stage 2 ---
const handleContinue = () => {
  initAudio()
  playButtonSound()
  currentStage.value = 2
  // Ready to open gift
  isGiftOpened.value = false
  giftOpening.value = false
  giftShaking.value = false
}

// --- Stage 2: Gift Box Animation Logic ---
const isGiftOpened = ref(false)
const giftShaking = ref(false)
const giftOpening = ref(false)

const triggerGiftOpen = () => {
  if (giftOpening.value || isGiftOpened.value) return
  initAudio()
  giftShaking.value = true
  playButtonSound()

  setTimeout(() => {
    giftShaking.value = false
    giftOpening.value = true
    playSparkleSound()
    launchConfettiBurst()

    setTimeout(() => {
      isGiftOpened.value = true
      // Transition to Stage 3 ("Tu Día Especial" with the 6 selfies)
      setTimeout(() => {
        currentStage.value = 3
        playSparkleSound()
        launchConfettiBurst()
      }, 1200)
    }, 1100)
  }, 650)
}

// --- Stage 3: Typewriter Effect Logic ---
const rawLetterText = `Llegar a tus 26 años teniendo el regalo de caminar a tu lado durante muchos años es la mayor bendición de mi vida. Hemos visto al otro crecer, cambiar y convertirse en su mejor versión, pero lo único que no ha cambiado en todos estos años es la certeza de que mi lugar favorito siempre será contigo. Feliz cumpleaños, mi amor gracias por existir y por hacer que estos  años a tu lado hayan sido los más hermosos de mi historia.`

const displayedText = ref('')
const isTypingComplete = ref(false)
const isTyping = ref(false)
const letterScrollRef = ref(null)
let typingTimeout = null

const scrollToBottom = () => {
  nextTick(() => {
    if (letterScrollRef.value) {
      letterScrollRef.value.scrollTop = letterScrollRef.value.scrollHeight
    }
  })
}

const startTypewriter = () => {
  displayedText.value = ''
  isTypingComplete.value = false
  isTyping.value = true
  let charIndex = 0

  const typeNextChar = () => {
    if (charIndex < rawLetterText.length) {
      displayedText.value += rawLetterText.charAt(charIndex)
      if (charIndex % 3 === 0) {
        playTypewriterSound()
      }
      charIndex++
      scrollToBottom()
      const delay = rawLetterText.charAt(charIndex) === '\n' ? 120 : (Math.random() * 22 + 18)
      typingTimeout = setTimeout(typeNextChar, delay)
    } else {
      isTyping.value = false
      isTypingComplete.value = true
      scrollToBottom()
      playSparkleSound()
    }
  }

  typingTimeout = setTimeout(typeNextChar, 400)
}

const skipTypewriter = () => {
  if (typingTimeout) clearTimeout(typingTimeout)
  displayedText.value = rawLetterText
  isTyping.value = false
  isTypingComplete.value = true
  scrollToBottom()
  playSparkleSound()
}

// Transition from Stage 3 ("Tu Día Especial") to Stage 4 (Message Card)
const goToMessage = () => {
  initAudio()
  playSparkleSound()
  launchConfettiBurst()
  currentStage.value = 4
  startTypewriter()
  playCelebrationFanfare()
  window.scrollTo({ top: 0, behavior: 'smooth' })
}

// Stage 5: Go to Polaroid Gallery
const goToGallery = () => {
  initAudio()
  playSparkleSound()
  launchConfettiBurst()
  currentStage.value = 5
  window.scrollTo({ top: 0, behavior: 'smooth' })
}

const backToMessage = () => {
  initAudio()
  playButtonSound()
  currentStage.value = 4
  displayedText.value = rawLetterText
  isTyping.value = false
  isTypingComplete.value = true
  window.scrollTo({ top: 0, behavior: 'smooth' })
}

// Stage 6: Go to Dedicated Video Theater (Only Videos)
const goToVideos = () => {
  initAudio()
  playSparkleSound()
  launchConfettiBurst()
  currentStage.value = 6
  window.scrollTo({ top: 0, behavior: 'smooth' })
}

const backToGallery = () => {
  initAudio()
  playButtonSound()
  currentStage.value = 5
  window.scrollTo({ top: 0, behavior: 'smooth' })
}

// Stage 7: Go to Birthday Wish Section (after video)
const goToWish = () => {
  initAudio()
  playSparkleSound()
  launchConfettiBurst()
  currentStage.value = 7
  window.scrollTo({ top: 0, behavior: 'smooth' })
}

// Stage 8: Go to Galaxy Heart Experience
const goToGalaxy = () => {
  initAudio()
  playSparkleSound()
  launchConfettiBurst()
  currentStage.value = 8
  window.scrollTo({ top: 0, behavior: 'smooth' })
}

// Final celebration button
const showFinalCelebrationModal = ref(false)
const candleBlown = ref(false)

const handleFinalWish = () => {
  initAudio()
  playCelebrationFanfare()
  launchConfettiBurst()
  showFinalCelebrationModal.value = true
}

const blowCandle = () => {
  candleBlown.value = true
  playSparkleSound()
  launchConfettiBurst()
}

const resetAll = () => {
  initAudio()
  playButtonSound()
  currentStage.value = 1
  isGiftOpened.value = false
  giftOpening.value = false
  giftShaking.value = false
  isTypingComplete.value = false
  displayedText.value = ''
  showFinalCelebrationModal.value = false
  candleBlown.value = false
}

// --- Pure Canvas Confetti System ---
const canvasRef = ref(null)
let confettiParticles = []
let animationFrameId = null

const initCanvas = () => {
  const canvas = canvasRef.value
  if (!canvas) return
  canvas.width = window.innerWidth
  canvas.height = window.innerHeight
}

const launchConfettiBurst = () => {
  const canvas = canvasRef.value
  if (!canvas) return
  const colors = ['#FFD1DC', '#FF69B4', '#FFB6C1', '#FFD700', '#FFF0F5', '#E6A8D7', '#FFFFFF', '#FF85A2', '#FAD02C']
  const count = 95
  const centerX = window.innerWidth / 2
  const centerY = window.innerHeight / 2

  for (let i = 0; i < count; i++) {
    const angle = Math.random() * Math.PI * 2
    const velocity = Math.random() * 14 + 5
    confettiParticles.push({
      x: centerX,
      y: centerY,
      vx: Math.cos(angle) * velocity + (Math.random() - 0.5) * 3,
      vy: Math.sin(angle) * velocity - Math.random() * 5 - 2,
      size: Math.random() * 8 + 5,
      color: colors[Math.floor(Math.random() * colors.length)],
      rotation: Math.random() * 360,
      rotationSpeed: (Math.random() - 0.5) * 14,
      opacity: 1,
      shape: Math.random() > 0.4 ? 'circle' : (Math.random() > 0.5 ? 'heart' : 'rect'),
      gravity: 0.2,
      drag: 0.965
    })
  }

  if (!animationFrameId) {
    runConfettiLoop()
  }
}

const runConfettiLoop = () => {
  const canvas = canvasRef.value
  if (!canvas) return
  const ctx = canvas.getContext('2d')
  ctx.clearRect(0, 0, canvas.width, canvas.height)

  for (let i = confettiParticles.length - 1; i >= 0; i--) {
    const p = confettiParticles[i]
    p.x += p.vx
    p.y += p.vy
    p.vy += p.gravity
    p.vx *= p.drag
    p.vy *= p.drag
    p.rotation += p.rotationSpeed
    p.opacity -= 0.007

    if (p.opacity <= 0 || p.y > canvas.height + 50) {
      confettiParticles.splice(i, 1)
      continue
    }

    ctx.save()
    ctx.translate(p.x, p.y)
    ctx.rotate((p.rotation * Math.PI) / 180)
    ctx.globalAlpha = Math.max(0, p.opacity)
    ctx.fillStyle = p.color

    if (p.shape === 'circle') {
      ctx.beginPath()
      ctx.arc(0, 0, p.size / 2, 0, Math.PI * 2)
      ctx.fill()
    } else if (p.shape === 'rect') {
      ctx.fillRect(-p.size / 2, -p.size / 2, p.size, p.size * 0.6)
    } else if (p.shape === 'heart') {
      ctx.beginPath()
      const s = p.size * 0.65
      ctx.moveTo(0, s * 0.3)
      ctx.bezierCurveTo(-s * 0.5, -s * 0.3, -s, s * 0.3, 0, s)
      ctx.bezierCurveTo(s, s * 0.3, s * 0.5, -s * 0.3, 0, s * 0.3)
      ctx.fill()
    }
    ctx.restore()
  }

  if (confettiParticles.length > 0) {
    animationFrameId = requestAnimationFrame(runConfettiLoop)
  } else {
    animationFrameId = null
    ctx.clearRect(0, 0, canvas.width, canvas.height)
  }
}

const handleResize = () => {
  initCanvas()
}

onMounted(() => {
  initCanvas()
  window.addEventListener('resize', handleResize)
  updateCountdown()
  timerInterval = setInterval(updateCountdown, 1000)
})

onUnmounted(() => {
  window.removeEventListener('resize', handleResize)
  if (timerInterval) clearInterval(timerInterval)
  if (typingTimeout) clearTimeout(typingTimeout)
  if (animationFrameId) cancelAnimationFrame(animationFrameId)
})
</script>

<template>
  <div class="birthday-experience-wrapper">
    <!-- Confetti canvas overlay -->
    <canvas ref="canvasRef" class="confetti-canvas"></canvas>

    <!-- Top floating luxury control bar -->
    <header class="top-nav-bar">
    </header>

    <!-- Date Picker Popover -->
    <transition name="pop-fade">
      <div v-if="showDatePicker" class="date-picker-card">
        <div class="picker-header">
          <h4>Configurar Fecha del Cumpleaños</h4>
          <button class="close-btn" @click="showDatePicker = false">✕</button>
        </div>
        <p class="picker-hint">Selecciona la fecha y hora exacta de la celebración:</p>
        <input 
          type="datetime-local" 
          v-model="targetDate" 
          @change="updateCountdown"
          class="custom-datetime-input"
        />
        <div class="picker-actions">
          <button class="picker-save-btn" @click="showDatePicker = false">Guardar Fecha</button>
        </div>
      </div>
    </transition>

    <!-- Ambient glowing orbs & sparkles -->
    <div class="magic-ambient-glow glow-1"></div>
    <div class="magic-ambient-glow glow-2"></div>
    <div class="magic-ambient-glow glow-3"></div>

    <div class="floating-bokeh">
      <span class="bokeh-star b1">✦</span>
      <span class="bokeh-star b2">✧</span>
      <span class="bokeh-star b3">✦</span>
      <span class="bokeh-star b4">🌸</span>
      <span class="bokeh-star b5">✨</span>
      <span class="bokeh-star b6">💖</span>
    </div>

    <!-- MAIN VIEW CONTAINER -->
    <main class="content-container">

      <!-- ==============================================
           STAGE 1: COUNTDOWN TO SPECIAL DAY
           ============================================== -->
      <transition name="stage-fade" mode="out-in">
        <section v-if="currentStage === 1" key="stage-1" class="stage-section stage-countdown">

          <!-- Typography: "tu dia especial" & "esta llegando" -->
          <div class="titles-wrapper">
            <h1 class="main-title">Tu día especial</h1>
            <p class="subtitle-text">está llegando</p>
          </div>

          <div class="timer-ornament">
            <span class="ornament-line"></span>
            <span class="ornament-diamond">❖</span>
            <span class="ornament-line"></span>
          </div>

          <!-- Timer Cards: DÍAS, HORAS, MINUTOS, SEGUNDOS -->
          <div class="timer-grid">
            <!-- DÍAS -->
            <div class="timer-card">
              <div class="card-glass-glow"></div>
              <div class="card-inner">
                <span class="card-value">{{ padZero(remainingTime.days) }}</span>
                <span class="card-label">DÍAS</span>
              </div>
            </div>

            <!-- HORAS -->
            <div class="timer-card">
              <div class="card-glass-glow"></div>
              <div class="card-inner">
                <span class="card-value">{{ padZero(remainingTime.hours) }}</span>
                <span class="card-label">HORAS</span>
              </div>
            </div>

            <!-- MINUTOS -->
            <div class="timer-card">
              <div class="card-glass-glow"></div>
              <div class="card-inner">
                <span class="card-value">{{ padZero(remainingTime.minutes) }}</span>
                <span class="card-label">MINUTOS</span>
              </div>
            </div>

            <!-- SEGUNDOS -->
            <div class="timer-card highlight-seconds">
              <div class="card-glass-glow"></div>
              <div class="card-inner">
                <span class="card-value">{{ padZero(remainingTime.seconds) }}</span>
                <span class="card-label">SEGUNDOS</span>
              </div>
            </div>
          </div>

          <!-- Continuar Button to advance to gift -->
          <div class="action-btn-container">
            <button 
              id="btn-continuar-especial"
              class="btn-open-letter-pill"
              @click="handleContinue"
            >
              <span class="btn-heart">♡</span>
              <span class="btn-caption">Continuar</span>
              <span class="btn-heart">♡</span>
            </button>
          </div>
        </section>

        <!-- ==============================================
             STAGE 2: GIFT OPENING ANIMATION
             ============================================== -->
        <section v-else-if="currentStage === 2" key="stage-2" class="stage-section stage-gift">
          <div class="gift-scene-header">
            <span class="gift-pretitle">✦ UNA SORPRESA ESPERA POR TI ✦</span>
            <h2 class="gift-instruction">
              {{ isGiftOpened ? '¡Sorpresa revelada!' : '¡Tienes un regalo especial!' }}
            </h2>
          </div>

          <!-- 3D Gift Box Interactive Area -->
          <div 
            class="gift-box-stage" 
            :class="{ 
              'is-shaking': giftShaking, 
              'is-opening': giftOpening, 
              'is-opened': isGiftOpened 
            }"
            @click="triggerGiftOpen"
            title="Toca para abrir"
          >
            <!-- Glowing aura behind the box -->
            <div class="gift-aura"></div>
            <div class="light-beam-burst" v-if="giftOpening || isGiftOpened"></div>

            <div class="gift-box-3d">
              <!-- Lid of the gift box -->
              <div class="gift-lid">
                <div class="lid-ribbon-vertical"></div>
                <!-- 3D Bow on top -->
                <div class="gift-bow">
                  <div class="bow-loop bow-left"></div>
                  <div class="bow-loop bow-right"></div>
                  <div class="bow-center"></div>
                  <div class="bow-ribbon ribbon-tail-left"></div>
                  <div class="bow-ribbon ribbon-tail-right"></div>
                </div>
              </div>

              <!-- Base of the gift box -->
              <div class="gift-base">
                <div class="base-ribbon-horizontal"></div>
                <div class="base-ribbon-vertical"></div>
                <!-- Sparkles burst from inside -->
                <div class="inside-sparkles" v-if="giftOpening || isGiftOpened">
                  <span class="box-sparkle s1">✨</span>
                  <span class="box-sparkle s2">💖</span>
                  <span class="box-sparkle s3">⭐</span>
                  <span class="box-sparkle s4">🌸</span>
                  <span class="box-sparkle s5">✨</span>
                </div>
              </div>

              <!-- Shadow underneath -->
              <div class="gift-shadow"></div>
            </div>
          </div>

          <!-- Interactive button to open -->
          <div class="gift-manual-action">
            <button 
              class="btn-open-letter-pill pulse-pill" 
              @click="triggerGiftOpen"
              v-if="!isGiftOpened"
            >
              <span class="btn-heart">♡</span>
              <span class="btn-caption">Abrir Regalo</span>
              <span class="btn-heart">♡</span>
            </button>
          </div>
        </section>

        <!-- ==============================================
             STAGE 3: TU DÍA ESPECIAL (6 SELFIES & HERO)
             ============================================== -->
        <section v-else-if="currentStage === 3" key="stage-3" class="stage-section stage-special-day">
          <SpecialDayIntro
            :audio-enabled="audioEnabled"
            @sparkle-sound="playSparkleSound"
            @burst-confetti="launchConfettiBurst"
            @continue-to-message="goToMessage"
          />
        </section>

        <!-- ==============================================
             STAGE 4: BIRTHDAY CELEBRATION & TYPEWRITER MODAL
             ============================================== -->
        <section v-else-if="currentStage === 4" key="stage-4" class="stage-section stage-celebration">
          
          <!-- Big celebratory text: "¡Feliz Cumpleaños!" -->
          <div class="celebration-heading-wrap">
            <div class="floating-balloons-left">🎈✨</div>
            <h1 class="celebration-title">¡Feliz Cumpleaños!</h1>
            <div class="floating-balloons-right">🎂🎉</div>
          </div>

          <p class="celebration-subtitle">Un mensaje escrito desde el corazón para ti</p>

          <!-- Typewriter Window / Modal Card -->
          <div class="typewriter-modal-card">
            <!-- Modal Header Bar -->
            <div class="modal-card-topbar">
              <div class="modal-dots">
                <span class="dot dot-rose"></span>
                <span class="dot dot-gold"></span>
                <span class="dot dot-white"></span>
              </div>
              <div class="modal-actions-corner">
              </div>
            </div>

            <!-- Modal Content Area with Typewriter Text -->
            <div class="modal-text-body">
              <div class="letter-scroll-area" ref="letterScrollRef">
                <p class="typewriter-content">
                  {{ displayedText }}
                  <span v-if="!isTypingComplete" class="typewriter-cursor">|</span>
                </p>
              </div>
            </div>

           
          </div>

           <!-- Action below modal: When finished typing, show Recuerdos button with generous spacing -->
            <transition name="button-pop">
              <div v-if="isTypingComplete" class="message-action-container">
                <button 
                  id="btn-recuerdos"
                  class="btn-open-letter-pill"
                  @click="goToGallery"
                >
                  <span class="btn-heart">♡</span>
                  <span class="btn-caption">Recuerdos</span>
                  <span class="btn-heart">♡</span>
                </button>
              </div>
            </transition>

        </section>

        <!-- ==============================================
             STAGE 5: 3D POLAROID PHOTO GALLERY ONLY
             ============================================== -->
        <section v-else-if="currentStage === 5" key="stage-5" class="stage-section stage-gallery">
          <!-- 3D Polaroid Photo Gallery with Progressive Scroll Reveal -->
          <PolaroidGallery3D 
            :audio-enabled="audioEnabled"
            @sparkle-sound="playSparkleSound"
            @burst-confetti="launchConfettiBurst"
            @go-to-videos="goToVideos"
            @back-to-message="backToMessage"
          />
        </section>

        <!-- ==============================================
             STAGE 6: DEDICATED FRAMED VIDEO THEATER ONLY
             ============================================== -->
        <section v-else-if="currentStage === 6" key="stage-6" class="stage-section stage-videos">
          <VideoTheater
            :audio-enabled="audioEnabled"
            @sparkle-sound="playSparkleSound"
            @burst-confetti="launchConfettiBurst"
            @continue-to-wish="goToWish"
            @back-to-gallery="backToGallery"
          />
        </section>

        <!-- ==============================================
             STAGE 7: BIRTHDAY WISH CARD & FINAL CONTINUAR
             ============================================== -->
        <section v-else-if="currentStage === 7" key="stage-7" class="stage-section stage-wish">
          <BirthdayWishSection
            :audio-enabled="audioEnabled"
            @sparkle-sound="playSparkleSound"
            @burst-confetti="launchConfettiBurst"
            @continue-to-final="goToGalaxy"
          />
        </section>

        <!-- ==============================================
             STAGE 8: 3D INTERACTIVE GALAXY HEART
             ============================================== -->
        <section v-else-if="currentStage === 8" key="stage-8" class="stage-section stage-galaxy-heart">
          <GalaxyHeart
            :audio-enabled="audioEnabled"
            @sparkle-sound="playSparkleSound"
            @burst-confetti="launchConfettiBurst"
            @back-to-wish="goToWish"
            @open-celebration-modal="handleFinalWish"
          />
        </section>
      </transition>
    </main>

    <!-- Celebration Wish Modal / Confetti finale modal -->
    <transition name="pop-fade">
      <div v-if="showFinalCelebrationModal" class="wish-modal-overlay" @click.self="showFinalCelebrationModal = false">
        <div class="wish-modal-content">
          <div class="cake-container" @click="blowCandle">
            <div class="candle-flame" :class="{ 'flame-extinguished': candleBlown }">🔥</div>
            <div class="wish-cake-icon">🎂</div>
          </div>
          <h2 class="wish-title">¡Que todos tus deseos se cumplan!</h2>
          <p class="wish-text">
            {{ candleBlown ? '¡Vela soplada! Tu deseo ha sido enviado al universo con amor y bendiciones. ✨' : 'Toca el pastel para soplar la velita y pedir tu deseo más anhelado. 🌸' }}
          </p>
          <div class="wish-buttons">
            <button class="btn-open-letter-pill" @click="launchConfettiBurst">
              <span class="btn-heart">♡</span>
              <span class="btn-caption">¡Lanzar más confeti! 🎊</span>
              <span class="btn-heart">♡</span>
            </button>
            <button class="glass-subtle-btn" @click="showFinalCelebrationModal = false">
              Cerrar
            </button>
          </div>
        </div>
      </div>
    </transition>
  </div>
</template>

<style scoped>
/* ==========================================================================
   RESET & FOUNDATION: MATTE / OPAQUE ROSE GOLD PALETTE
   ========================================================================== */
.birthday-experience-wrapper {
  position: relative;
  min-height: 100vh;
  width: 100%;
  display: flex;
  flex-direction: column;
  justify-content: space-between;
  align-items: center;
  overflow-x: hidden;
  font-family: 'Montserrat', sans-serif;
  color: #ffffff;
  /* Rich, velvety opaque / matte rose gold gradient */
  background: radial-gradient(circle at 50% 20%, #b86b78 0%, #994d5a 40%, #763340 75%, #521f28 100%);
  background-attachment: fixed;
  box-sizing: border-box;
  padding: 0.8rem 1.2rem 1.8rem 1.2rem;
  user-select: none;
}

/* Confetti Fullscreen Overlay */
.confetti-canvas {
  position: fixed;
  top: 0;
  left: 0;
  width: 100vw;
  height: 100vh;
  pointer-events: none;
  z-index: 999;
}

/* Ambient glows */
.magic-ambient-glow {
  position: absolute;
  border-radius: 50%;
  pointer-events: none;
  filter: blur(80px);
  opacity: 0.4;
  z-index: 1;
}

.glow-1 {
  width: 450px;
  height: 450px;
  top: -100px;
  left: 8%;
  background: radial-gradient(circle, #e2959f 0%, rgba(226, 149, 159, 0) 70%);
}

.glow-2 {
  width: 550px;
  height: 550px;
  bottom: -150px;
  right: 5%;
  background: radial-gradient(circle, #d47a88 0%, rgba(212, 122, 136, 0) 70%);
}

.glow-3 {
  width: 300px;
  height: 300px;
  top: 45%;
  left: 50%;
  transform: translate(-50%, -50%);
  background: radial-gradient(circle, #f3b6bf 0%, rgba(243, 182, 191, 0) 70%);
  opacity: 0.22;
}

/* Floating Bokeh Stars & Elements */
.floating-bokeh {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  pointer-events: none;
  overflow: hidden;
  z-index: 2;
}

.bokeh-star {
  position: absolute;
  color: rgba(255, 230, 235, 0.55);
  font-size: 1.1rem;
  animation: floatOrb 6s ease-in-out infinite alternate;
}

.b1 { top: 12%; left: 10%; animation-delay: 0s; font-size: 1.4rem; }
.b2 { top: 22%; right: 12%; animation-delay: 1.5s; font-size: 1.1rem; }
.b3 { bottom: 20%; left: 6%; animation-delay: 3s; font-size: 1.6rem; color: #ffe6ec; }
.b4 { top: 60%; right: 8%; animation-delay: 2s; font-size: 1.3rem; opacity: 0.6; }
.b5 { top: 75%; left: 25%; animation-delay: 4s; font-size: 1.4rem; }
.b6 { bottom: 12%; right: 22%; animation-delay: 2.5s; font-size: 1.2rem; opacity: 0.5; }

@keyframes floatOrb {
  0% { transform: translateY(0px) scale(0.95); opacity: 0.3; }
  100% { transform: translateY(-18px) scale(1.08); opacity: 0.75; }
}

/* ==========================================================================
   TOP NAVIGATION & CONTROLS
   ========================================================================== */
.top-nav-bar {
  position: relative;
  z-index: 10;
  width: 100%;
  max-width: 1050px;
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 0.5rem 0;
}

.brand-tag {
  display: flex;
  align-items: center;
  gap: 0.5rem;
  font-size: 0.8rem;
  letter-spacing: 0.15em;
  text-transform: uppercase;
  color: #ffdde2;
  font-weight: 500;
  opacity: 0.9;
}

.sparkle-icon {
  color: #ffd1d9;
  animation: pulseGlow 2s infinite;
}

.nav-controls {
  display: flex;
  align-items: center;
  gap: 0.5rem;
}

.glass-pill-btn {
  background: rgba(255, 255, 255, 0.12);
  border: 1px solid rgba(255, 255, 255, 0.25);
  backdrop-filter: blur(10px);
  color: #ffffff;
  padding: 0.4rem 0.85rem;
  border-radius: 9999px;
  font-size: 0.8rem;
  font-weight: 500;
  cursor: pointer;
  display: flex;
  align-items: center;
  gap: 0.4rem;
  transition: all 0.25s ease;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.08);
}

.glass-pill-btn:hover {
  background: rgba(255, 255, 255, 0.22);
  transform: translateY(-2px);
  border-color: rgba(255, 255, 255, 0.45);
}

.glass-pill-btn.icon-only {
  padding: 0.4rem 0.6rem;
  font-size: 0.95rem;
}

.glass-pill-btn.catalog-btn {
  background: rgba(0, 0, 0, 0.22);
  border-color: rgba(255, 255, 255, 0.18);
  font-size: 0.76rem;
  opacity: 0.78;
}

.glass-pill-btn.catalog-btn:hover {
  opacity: 1;
}

.icon {
  width: 13px;
  height: 13px;
}

/* Date picker popover */
.date-picker-card {
  position: absolute;
  top: 3.8rem;
  right: 1.5rem;
  z-index: 50;
  background: rgba(84, 32, 42, 0.95);
  border: 1px solid rgba(255, 200, 210, 0.35);
  backdrop-filter: blur(16px);
  padding: 1.1rem;
  border-radius: 1.2rem;
  box-shadow: 0 15px 35px rgba(0, 0, 0, 0.4);
  max-width: 320px;
  width: 90%;
}

.picker-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 0.5rem;
}

.picker-header h4 {
  font-size: 0.9rem;
  font-weight: 600;
  color: #ffe6ec;
}

.close-btn {
  background: none;
  border: none;
  color: #ffb8c4;
  cursor: pointer;
  font-size: 1rem;
}

.picker-hint {
  font-size: 0.78rem;
  color: #f7cbd2;
  margin-bottom: 0.7rem;
  line-height: 1.35;
}

.custom-datetime-input {
  width: 100%;
  padding: 0.55rem;
  border-radius: 0.5rem;
  border: 1px solid rgba(255, 255, 255, 0.3);
  background: rgba(255, 255, 255, 0.15);
  color: #ffffff;
  font-size: 0.88rem;
  outline: none;
  box-sizing: border-box;
  margin-bottom: 0.7rem;
}

.picker-actions {
  display: flex;
  justify-content: flex-end;
}

.picker-save-btn {
  background: #ff758c;
  color: white;
  border: none;
  padding: 0.35rem 0.85rem;
  border-radius: 0.45rem;
  font-weight: 600;
  font-size: 0.8rem;
  cursor: pointer;
}

/* ==========================================================================
   MAIN CONTENT CONTAINER
   ========================================================================== */
.content-container {
  position: relative;
  z-index: 5;
  width: 100%;
  max-width: 1200px;
  margin: auto;
  display: flex;
  justify-content: center;
  align-items: center;
  padding: 1rem 0;
}

.stage-section {
  width: 100%;
  display: flex;
  flex-direction: column;
  align-items: center;
  text-align: center;
}

/* ==========================================================================
   STAGE 1: COUNTDOWN DESIGN
   ========================================================================== */
.stage-badge {
  display: inline-flex;
  align-items: center;
  gap: 0.4rem;
  padding: 0.3rem 0.9rem;
  border-radius: 9999px;
  background: rgba(255, 255, 255, 0.12);
  border: 1px solid rgba(255, 255, 255, 0.25);
  backdrop-filter: blur(8px);
  font-size: 0.75rem;
  letter-spacing: 0.1em;
  text-transform: uppercase;
  color: #ffe6ec;
  margin-bottom: 1rem;
}

.badge-dot {
  width: 6px;
  height: 6px;
  border-radius: 50%;
  background: #ff9fb2;
  box-shadow: 0 0 8px #ff758c;
  animation: pulseDot 1.5s infinite;
}

@keyframes pulseDot {
  0%, 100% { opacity: 0.5; transform: scale(0.9); }
  50% { opacity: 1; transform: scale(1.3); }
}

/* Typography: "Tu día especial" & "está llegando" */
.titles-wrapper {
  margin-bottom: 0.6rem;
}

.main-title {
  font-family: 'Playfair Display', Georgia, serif;
  font-size: clamp(2.4rem, 6.2vw, 4.4rem);
  font-weight: 700;
  line-height: 1.15;
  color: #ffffff;
  text-shadow: 0 4px 18px rgba(50, 15, 25, 0.4), 0 2px 4px rgba(0, 0, 0, 0.2);
  margin-bottom: 0.2rem;
}

.subtitle-text {
  font-family: 'Montserrat', sans-serif;
  font-size: clamp(1.05rem, 2.6vw, 1.5rem);
  font-weight: 300;
  letter-spacing: 0.22em;
  text-transform: uppercase;
  color: #ffdde3;
  text-shadow: 0 2px 8px rgba(0, 0, 0, 0.2);
}

/* Ornament Divider */
.timer-ornament {
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 0.8rem;
  width: 100%;
  max-width: 320px;
  margin: 0.8rem 0 1.6rem 0;
  opacity: 0.65;
}

.ornament-line {
  flex: 1;
  height: 1px;
  background: linear-gradient(90deg, transparent, rgba(255, 255, 255, 0.5), transparent);
}

.ornament-diamond {
  color: #ffccd5;
  font-size: 0.8rem;
}

/* Timer Cards Grid */
.timer-grid {
  display: grid;
  grid-template-columns: repeat(4, 1fr);
  gap: clamp(0.6rem, 2vw, 1.4rem);
  width: 100%;
  max-width: 780px;
  margin-bottom: 1.8rem;
}

.timer-card {
  position: relative;
  background: rgba(255, 255, 255, 0.13);
  backdrop-filter: blur(14px);
  -webkit-backdrop-filter: blur(14px);
  border: 1px solid rgba(255, 255, 255, 0.32);
  border-radius: clamp(0.9rem, 1.8vw, 1.4rem);
  padding: clamp(1rem, 2.5vw, 1.8rem) clamp(0.5rem, 1.2vw, 1rem);
  box-shadow: 0 12px 30px rgba(50, 10, 20, 0.25),
              inset 0 1px 0 rgba(255, 255, 255, 0.4);
  transition: transform 0.3s cubic-bezier(0.34, 1.56, 0.64, 1), box-shadow 0.3s ease;
  overflow: hidden;
}

.timer-card:hover {
  transform: translateY(-5px);
  box-shadow: 0 18px 40px rgba(50, 10, 20, 0.35),
              inset 0 1px 0 rgba(255, 255, 255, 0.6);
}

.card-glass-glow {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 50%;
  background: linear-gradient(180deg, rgba(255, 255, 255, 0.15) 0%, rgba(255, 255, 255, 0) 100%);
  pointer-events: none;
}

.card-inner {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
}

.card-value {
  font-family: 'Cinzel', 'Playfair Display', serif;
  font-size: clamp(2.2rem, 5.5vw, 3.8rem);
  font-weight: 700;
  line-height: 1;
  color: #ffffff;
  text-shadow: 0 4px 15px rgba(0, 0, 0, 0.25);
  margin-bottom: 0.4rem;
}

.card-label {
  font-family: 'Montserrat', sans-serif;
  font-size: clamp(0.68rem, 1.3vw, 0.82rem);
  font-weight: 600;
  letter-spacing: 0.18em;
  color: #ffdde3;
  text-transform: uppercase;
}

.highlight-seconds .card-value {
  animation: tickSeconds 1s infinite;
}

@keyframes tickSeconds {
  0%, 100% { opacity: 1; }
  50% { opacity: 0.85; }
}

/* Button: Continuar (Rose/Pink) */
.action-btn-container {
  margin-top: 0.4rem;
}

.btn-continuar-rose {
  position: relative;
  overflow: hidden;
  display: inline-flex;
  align-items: center;
  justify-content: center;
  gap: 0.75rem;
  padding: 0.9rem 2.6rem;
  border-radius: 9999px;
  background: linear-gradient(135deg, #ff758c 0%, #ff5277 50%, #d83159 100%);
  color: #ffffff;
  border: 1px solid rgba(255, 255, 255, 0.4);
  font-family: 'Montserrat', sans-serif;
  font-size: clamp(0.95rem, 1.8vw, 1.15rem);
  font-weight: 700;
  letter-spacing: 0.05em;
  text-transform: uppercase;
  cursor: pointer;
  box-shadow: 0 10px 28px rgba(230, 49, 89, 0.45),
              0 2px 6px rgba(0, 0, 0, 0.15),
              inset 0 1px 0 rgba(255, 255, 255, 0.4);
  transition: all 0.3s cubic-bezier(0.34, 1.56, 0.64, 1);
}

.btn-continuar-rose:hover {
  transform: translateY(-3px) scale(1.02);
  box-shadow: 0 15px 36px rgba(230, 49, 89, 0.65),
              inset 0 1px 0 rgba(255, 255, 255, 0.6);
  background: linear-gradient(135deg, #ff8da1 0%, #ff5e82 50%, #e63962 100%);
}

.btn-sparkle-sweep {
  position: absolute;
  top: 0;
  left: -100%;
  width: 60%;
  height: 100%;
  background: linear-gradient(90deg, transparent, rgba(255, 255, 255, 0.4), transparent);
  transform: skewX(-25deg);
  animation: shineSweep 3.5s infinite;
}

@keyframes shineSweep {
  0% { left: -100%; }
  35%, 100% { left: 160%; }
}

/* ==========================================================================
   STAGE 2: GIFT OPENING ANIMATION DESIGN
   ========================================================================== */
.stage-gift {
  padding: 1rem 0;
}

.gift-scene-header {
  margin-bottom: 1.2rem;
}

.gift-pretitle {
  font-size: 0.8rem;
  letter-spacing: 0.2em;
  color: #ffccd5;
  font-weight: 600;
  display: block;
  margin-bottom: 0.4rem;
}

.gift-instruction {
  font-family: 'Playfair Display', serif;
  font-size: clamp(1.8rem, 4.5vw, 2.8rem);
  font-weight: 700;
  color: #ffffff;
  text-shadow: 0 4px 15px rgba(0, 0, 0, 0.3);
  margin-bottom: 0.2rem;
}

.gift-subinstruction {
  font-size: 0.95rem;
  color: #ffe0e6;
  font-weight: 300;
}

/* 3D Gift Box */
.gift-box-stage {
  position: relative;
  width: 250px;
  height: 250px;
  margin: 1rem auto 2rem auto;
  cursor: pointer;
  perspective: 1000px;
  display: flex;
  justify-content: center;
  align-items: center;
}

.gift-box-stage.is-shaking .gift-box-3d {
  animation: shakeGift 0.65s infinite ease-in-out;
}

@keyframes shakeGift {
  0%, 100% { transform: rotate(0deg) scale(1); }
  20% { transform: rotate(-6deg) scale(1.04); }
  40% { transform: rotate(6deg) scale(1.04); }
  60% { transform: rotate(-4deg) scale(1.03); }
  80% { transform: rotate(4deg) scale(1.03); }
}

.gift-aura {
  position: absolute;
  width: 240px;
  height: 240px;
  border-radius: 50%;
  background: radial-gradient(circle, rgba(255, 182, 193, 0.45) 0%, rgba(255, 105, 180, 0) 70%);
  filter: blur(25px);
  animation: pulseGlow 2.5s infinite alternate;
}

@keyframes pulseGlow {
  0% { transform: scale(0.9); opacity: 0.4; }
  100% { transform: scale(1.2); opacity: 0.85; }
}

.light-beam-burst {
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  width: 420px;
  height: 420px;
  background: conic-gradient(
    from 0deg,
    rgba(255, 235, 180, 0.45),
    rgba(255, 105, 180, 0.25),
    rgba(255, 255, 255, 0.5),
    rgba(255, 182, 193, 0.25),
    rgba(255, 235, 180, 0.45)
  );
  border-radius: 50%;
  filter: blur(18px);
  animation: rotateBeams 5s linear infinite;
  z-index: 1;
}

@keyframes rotateBeams {
  from { transform: translate(-50%, -50%) rotate(0deg); }
  to { transform: translate(-50%, -50%) rotate(360deg); }
}

.gift-box-3d {
  position: relative;
  width: 180px;
  height: 180px;
  transform-style: preserve-3d;
  transition: transform 0.35s ease;
  z-index: 2;
}

.gift-box-stage:hover .gift-box-3d {
  transform: translateY(-6px) scale(1.03);
}

.gift-base {
  position: absolute;
  bottom: 0;
  left: 10px;
  width: 160px;
  height: 125px;
  background: linear-gradient(135deg, #f08ea3 0%, #d85773 60%, #b33954 100%);
  border-radius: 0 0 14px 14px;
  box-shadow: 0 15px 30px rgba(50, 10, 20, 0.4),
              inset 0 -4px 8px rgba(0, 0, 0, 0.25),
              inset 0 4px 6px rgba(255, 255, 255, 0.35);
  overflow: hidden;
}

.base-ribbon-horizontal {
  position: absolute;
  top: 45%;
  left: 0;
  width: 100%;
  height: 26px;
  background: linear-gradient(180deg, #ffe082 0%, #ffd54f 50%, #ffca28 100%);
  box-shadow: 0 2px 5px rgba(0, 0, 0, 0.15),
              inset 0 1px 0 rgba(255, 255, 255, 0.6);
}

.base-ribbon-vertical {
  position: absolute;
  top: 0;
  left: calc(50% - 13px);
  width: 26px;
  height: 100%;
  background: linear-gradient(90deg, #ffe082 0%, #ffd54f 50%, #ffca28 100%);
  box-shadow: 0 2px 5px rgba(0, 0, 0, 0.15),
              inset 1px 0 0 rgba(255, 255, 255, 0.6);
}

.inside-sparkles {
  position: absolute;
  width: 100%;
  height: 100%;
  top: 0;
  left: 0;
  pointer-events: none;
}

.box-sparkle {
  position: absolute;
  font-size: 1.5rem;
  animation: sparklePop 1.2s ease-out infinite;
}

.s1 { top: 20%; left: 20%; animation-delay: 0.1s; }
.s2 { top: 10%; right: 25%; animation-delay: 0.3s; }
.s3 { top: 40%; left: 45%; animation-delay: 0.5s; font-size: 1.8rem; }
.s4 { bottom: 30%; right: 20%; animation-delay: 0.2s; }
.s5 { top: 5%; left: 40%; animation-delay: 0.4s; }

@keyframes sparklePop {
  0% { transform: translateY(20px) scale(0); opacity: 0; }
  50% { transform: translateY(-30px) scale(1.3); opacity: 1; }
  100% { transform: translateY(-65px) scale(0.6); opacity: 0; }
}

.gift-lid {
  position: absolute;
  top: 40px;
  left: 0;
  width: 180px;
  height: 36px;
  background: linear-gradient(135deg, #f79eb1 0%, #e26a85 60%, #c4425e 100%);
  border-radius: 8px;
  box-shadow: 0 6px 15px rgba(0, 0, 0, 0.25),
              inset 0 2px 4px rgba(255, 255, 255, 0.5);
  z-index: 5;
  transition: all 0.85s cubic-bezier(0.2, 0.8, 0.2, 1);
  transform-origin: top center;
}

.gift-box-stage.is-opening .gift-lid,
.gift-box-stage.is-opened .gift-lid {
  transform: translateY(-105px) rotate(-24deg) scale(1.08);
  opacity: 0.85;
}

.lid-ribbon-vertical {
  position: absolute;
  top: 0;
  left: calc(50% - 13px);
  width: 26px;
  height: 100%;
  background: linear-gradient(90deg, #ffe082 0%, #ffd54f 50%, #ffca28 100%);
  box-shadow: 0 1px 3px rgba(0, 0, 0, 0.15);
}

.gift-bow {
  position: absolute;
  top: -22px;
  left: calc(50% - 28px);
  width: 56px;
  height: 28px;
}

.bow-center {
  position: absolute;
  top: 7px;
  left: 19px;
  width: 18px;
  height: 16px;
  background: linear-gradient(135deg, #ffca28 0%, #ffd54f 100%);
  border-radius: 5px;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.2);
  z-index: 3;
}

.bow-loop {
  position: absolute;
  top: 0;
  width: 26px;
  height: 22px;
  border: 4px solid #ffd54f;
  background: linear-gradient(135deg, #ffe082 0%, #ffca28 100%);
  box-shadow: inset 0 2px 4px rgba(0, 0, 0, 0.15), 0 3px 6px rgba(0, 0, 0, 0.15);
}

.bow-left {
  left: 0;
  border-radius: 50% 50% 10% 50%;
  transform: rotate(-25deg);
}

.bow-right {
  right: 0;
  border-radius: 50% 50% 50% 10%;
  transform: rotate(25deg);
}

.bow-ribbon {
  position: absolute;
  top: 16px;
  width: 9px;
  height: 25px;
  background: linear-gradient(180deg, #ffd54f 0%, #ffb300 100%);
  border-radius: 0 0 4px 4px;
  z-index: 1;
}

.ribbon-tail-left { left: 13px; transform: rotate(20deg); }
.ribbon-tail-right { right: 13px; transform: rotate(-20deg); }

.gift-shadow {
  position: absolute;
  bottom: -12px;
  left: 15px;
  width: 150px;
  height: 18px;
  background: radial-gradient(ellipse, rgba(30, 5, 10, 0.45) 0%, rgba(0, 0, 0, 0) 75%);
  border-radius: 50%;
  pointer-events: none;
}

.pulse-btn {
  animation: pulseButton 2s infinite alternate ease-in-out;
}

@keyframes pulseButton {
  0% { transform: scale(1); box-shadow: 0 8px 24px rgba(230, 49, 89, 0.4); }
  100% { transform: scale(1.05); box-shadow: 0 14px 35px rgba(230, 49, 89, 0.7); }
}

/* ==========================================================================
   STAGE 3: BIRTHDAY CELEBRATION & TYPEWRITER MODAL
   ========================================================================== */
.stage-celebration {
  padding: 0.2rem 0 1.5rem 0;
  max-width: 820px;
  width: 100%;
}

.celebration-heading-wrap {
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 1rem;
  margin-bottom: 0.2rem;
}

.floating-balloons-left,
.floating-balloons-right {
  font-size: clamp(1.6rem, 3.8vw, 2.5rem);
  animation: balloonFloat 3s ease-in-out infinite alternate;
}

.floating-balloons-right {
  animation-delay: 1.5s;
}

@keyframes balloonFloat {
  0% { transform: translateY(0) rotate(-4deg); }
  100% { transform: translateY(-10px) rotate(4deg); }
}

.celebration-title {
  font-family: 'Playfair Display', Georgia, serif;
  font-size: clamp(2.3rem, 5.8vw, 4.2rem);
  font-weight: 800;
  line-height: 1.15;
  color: #ffffff;
  text-shadow: 0 4px 25px rgba(255, 180, 195, 0.45),
              0 2px 5px rgba(0, 0, 0, 0.3);
}

.celebration-subtitle {
  font-size: clamp(0.9rem, 1.8vw, 1.1rem);
  color: #ffdde3;
  margin-bottom: 1.2rem;
  font-weight: 400;
  letter-spacing: 0.05em;
}

/* Luxury Typewriter Modal Card */
.typewriter-modal-card {
  position: relative;
  width: 100%;
  max-width: 780px;
  margin: 0 auto;
  background: rgba(255, 255, 255, 0.14);
  backdrop-filter: blur(18px);
  -webkit-backdrop-filter: blur(18px);
  border: 1px solid rgba(255, 255, 255, 0.35);
  border-radius: 1.5rem;
  box-shadow: 0 18px 45px rgba(40, 10, 20, 0.35),
              inset 0 1px 0 rgba(255, 255, 255, 0.5);
  overflow: hidden;
  text-align: left;
}

.modal-card-topbar {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 0.8rem 1.2rem;
  background: rgba(0, 0, 0, 0.12);
  border-bottom: 1px solid rgba(255, 255, 255, 0.18);
}

.modal-dots {
  display: flex;
  gap: 5px;
}

.dot {
  width: 9px;
  height: 9px;
  border-radius: 50%;
}

.dot-rose { background: #ff758c; }
.dot-gold { background: #ffd54f; }
.dot-white { background: rgba(255, 255, 255, 0.7); }

.modal-card-title {
  display: flex;
  align-items: center;
  gap: 0.4rem;
  font-family: 'Montserrat', sans-serif;
  font-size: 0.82rem;
  font-weight: 600;
  letter-spacing: 0.08em;
  text-transform: uppercase;
  color: #ffe4ea;
}

.icon-heart {
  font-size: 1rem;
}

.skip-typing-btn {
  background: rgba(255, 255, 255, 0.15);
  border: 1px solid rgba(255, 255, 255, 0.25);
  color: #ffffff;
  padding: 0.2rem 0.6rem;
  border-radius: 9999px;
  font-size: 0.72rem;
  cursor: pointer;
  font-weight: 500;
  transition: all 0.2s;
}

.skip-typing-btn:hover {
  background: rgba(255, 255, 255, 0.3);
}

.modal-text-body {
  padding: clamp(1.2rem, 3vw, 2rem);
}

.letter-scroll-area {
  max-height: 240px;
  overflow-y: auto;
  scroll-behavior: smooth;
  padding-right: 0.5rem;
}

.letter-scroll-area::-webkit-scrollbar {
  width: 6px;
}
.letter-scroll-area::-webkit-scrollbar-thumb {
  background: rgba(255, 255, 255, 0.3);
  border-radius: 9999px;
}

.typewriter-content {
  font-family: 'Playfair Display', Georgia, serif;
  font-size: clamp(1.05rem, 2.2vw, 1.35rem);
  line-height: 1.75;
  color: #ffffff;
  white-space: pre-line;
  letter-spacing: 0.01em;
  text-shadow: 0 1px 3px rgba(0, 0, 0, 0.2);
}

.typewriter-cursor {
  display: inline-block;
  font-weight: 300;
  color: #ff9fb2;
  margin-left: 2px;
  animation: blinkCursor 0.8s infinite;
}

@keyframes blinkCursor {
  0%, 100% { opacity: 1; }
  50% { opacity: 0; }
}

/* Bottom Action when Typewriter Finishes */
.message-action-container {
  margin-top: 3rem;
  margin-bottom: 2.5rem;
  display: flex;
  justify-content: center;
  align-items: center;
  width: 100%;
}

.modal-bottom-action {
  padding: 1rem 1.2rem 1.4rem 1.2rem;
  background: linear-gradient(180deg, rgba(255, 255, 255, 0.02) 0%, rgba(0, 0, 0, 0.15) 100%);
  border-top: 1px solid rgba(255, 255, 255, 0.15);
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 0.8rem;
}

.btn-celebrar-glow {
  position: relative;
  display: inline-flex;
  align-items: center;
  justify-content: center;
  gap: 0.75rem;
  padding: 1rem 2.5rem;
  border-radius: 9999px;
  background: linear-gradient(135deg, #ff758c 0%, #ff4d79 50%, #d82b57 100%);
  color: #ffffff;
  border: 1px solid rgba(255, 255, 255, 0.45);
  font-family: 'Montserrat', sans-serif;
  font-size: clamp(1rem, 2vw, 1.2rem);
  font-weight: 700;
  letter-spacing: 0.04em;
  cursor: pointer;
  box-shadow: 0 12px 35px rgba(240, 45, 90, 0.55),
              0 2px 6px rgba(0, 0, 0, 0.2),
              inset 0 1px 0 rgba(255, 255, 255, 0.4);
  transition: all 0.3s cubic-bezier(0.34, 1.56, 0.64, 1);
  animation: popPulse 2.5s infinite;
}

.btn-celebrar-glow:hover {
  transform: translateY(-3px) scale(1.03);
  box-shadow: 0 16px 42px rgba(240, 45, 90, 0.75),
              inset 0 1px 0 rgba(255, 255, 255, 0.6);
  background: linear-gradient(135deg, #ff8da1 0%, #ff5e85 50%, #e63564 100%);
}

@keyframes popPulse {
  0%, 100% { box-shadow: 0 10px 30px rgba(240, 45, 90, 0.5); }
  50% { box-shadow: 0 12px 42px rgba(240, 45, 90, 0.85), 0 0 25px rgba(255, 180, 195, 0.5); }
}

/* ==========================================================================
   STAGE 4: DEDICATED 3D POLAROID GALLERY
   ========================================================================== */
.stage-gallery {
  width: 100%;
  max-width: 1200px;
  display: flex;
  flex-direction: column;
  align-items: center;
  padding: 0.2rem 0 3rem 0;
}

.gallery-nav-bar {
  width: 100%;
  max-width: 1140px;
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 0.8rem;
  padding: 0 0.5rem;
  box-sizing: border-box;
}

.gallery-nav-bar .back-btn {
  display: inline-flex;
  align-items: center;
  gap: 0.5rem;
  font-size: 0.85rem;
  padding: 0.45rem 1.15rem;
}

.gallery-nav-bar .back-arrow {
  font-size: 1.1rem;
  transition: transform 0.2s ease;
}

.gallery-nav-bar .back-btn:hover .back-arrow {
  transform: translateX(-3px);
}

.secondary-modal-actions {
  display: flex;
  gap: 1rem;
}

.text-link-btn {
  background: none;
  border: none;
  color: #ffccd5;
  font-size: 0.85rem;
  font-weight: 500;
  cursor: pointer;
  opacity: 0.85;
  transition: opacity 0.2s;
}

.text-link-btn:hover {
  opacity: 1;
  text-decoration: underline;
}

/* ==========================================================================
   WISH MODAL OVERLAY (FINALE)
   ========================================================================== */
.wish-modal-overlay {
  position: fixed;
  top: 0;
  left: 0;
  width: 100vw;
  height: 100vh;
  background: rgba(35, 10, 15, 0.8);
  backdrop-filter: blur(12px);
  display: flex;
  justify-content: center;
  align-items: center;
  z-index: 1000;
  padding: 1.5rem;
}

.wish-modal-content {
  background: linear-gradient(145deg, #783542 0%, #54202a 100%);
  border: 1px solid rgba(255, 200, 215, 0.4);
  border-radius: 2rem;
  padding: clamp(1.8rem, 4vw, 2.8rem) clamp(1.4rem, 3.5vw, 2.2rem);
  text-align: center;
  max-width: 480px;
  width: 100%;
  box-shadow: 0 25px 60px rgba(0, 0, 0, 0.5),
              inset 0 1px 0 rgba(255, 255, 255, 0.4);
}

.cake-container {
  display: inline-block;
  position: relative;
  cursor: pointer;
  margin-bottom: 0.8rem;
  transition: transform 0.2s;
}

.cake-container:hover {
  transform: scale(1.08);
}

.candle-flame {
  font-size: 1.8rem;
  position: absolute;
  top: -16px;
  left: 50%;
  transform: translateX(-50%);
  animation: flicker 0.8s infinite alternate;
  transition: all 0.5s ease;
}

.candle-flame.flame-extinguished {
  opacity: 0;
  transform: translateX(-50%) translateY(-15px) scale(0);
}

@keyframes flicker {
  0% { transform: translateX(-50%) scale(1) rotate(-3deg); }
  100% { transform: translateX(-50%) scale(1.15) rotate(3deg); }
}

.wish-cake-icon {
  font-size: 3.8rem;
}

.wish-title {
  font-family: 'Playfair Display', serif;
  font-size: clamp(1.5rem, 3.5vw, 2rem);
  color: #ffffff;
  margin-bottom: 0.8rem;
  line-height: 1.25;
}

.wish-text {
  font-size: 0.98rem;
  color: #ffdde3;
  line-height: 1.55;
  margin-bottom: 1.8rem;
}

.wish-buttons {
  display: flex;
  flex-direction: column;
  gap: 0.7rem;
  align-items: center;
}

.glass-subtle-btn {
  background: rgba(255, 255, 255, 0.15);
  border: 1px solid rgba(255, 255, 255, 0.3);
  color: #ffffff;
  padding: 0.6rem 1.5rem;
  border-radius: 9999px;
  font-size: 0.9rem;
  font-weight: 600;
  cursor: pointer;
  backdrop-filter: blur(8px);
  transition: all 0.25s ease;
}

.glass-subtle-btn:hover {
  background: rgba(255, 255, 255, 0.28);
  transform: translateY(-2px);
}

/* ==========================================================================
   TRANSITIONS & ANIMATIONS
   ========================================================================== */
.stage-fade-enter-active,
.stage-fade-leave-active {
  transition: opacity 0.45s ease, transform 0.45s cubic-bezier(0.2, 0.8, 0.2, 1);
}

.stage-fade-enter-from {
  opacity: 0;
  transform: translateY(18px) scale(0.98);
}

.stage-fade-leave-to {
  opacity: 0;
  transform: translateY(-18px) scale(0.98);
}

.button-pop-enter-active {
  transition: opacity 0.5s ease, transform 0.5s cubic-bezier(0.34, 1.56, 0.64, 1);
}

.button-pop-enter-from {
  opacity: 0;
  transform: translateY(20px) scale(0.88);
}

.pop-fade-enter-active,
.pop-fade-leave-active {
  transition: opacity 0.25s ease, transform 0.25s cubic-bezier(0.34, 1.56, 0.64, 1);
}

.pop-fade-enter-from,
.pop-fade-leave-to {
  opacity: 0;
  transform: scale(0.94);
}

/* Responsive queries */
@media (max-width: 680px) {
  .timer-grid {
    grid-template-columns: repeat(2, 1fr);
    gap: 0.75rem;
  }
  
  .birthday-experience-wrapper {
    padding: 0.6rem 0.8rem 1.5rem 0.8rem;
  }
  
  .gift-box-stage {
    width: 210px;
    height: 210px;
  }
  
  .gift-box-3d {
    width: 160px;
    height: 160px;
  }

  .gift-base {
    width: 140px;
    height: 110px;
    left: 10px;
  }

  .gift-lid {
    width: 160px;
    height: 30px;
    top: 36px;
  }
}
</style>
