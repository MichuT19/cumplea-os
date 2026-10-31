<script setup>
import { ref, onMounted, onUnmounted } from 'vue'

const props = defineProps({
  audioEnabled: {
    type: Boolean,
    default: true
  }
})

const emit = defineEmits(['backToWish', 'openCelebrationModal', 'sparkleSound', 'burstConfetti'])

const canvasRef = ref(null)
const heartBeatScale = ref(1)

let animationFrameId = null
let ctx = null
let width = 0
let height = 0

// 3D Camera & Rotation state
let rotX = 0.32
let rotY = 0
let targetRotX = 0.32
let targetRotY = 0
let isDragging = false
let startPointerX = 0
let startPointerY = 0

// Particle arrays
let heartParticles = []
let galaxyParticles = []
let burstParticles = []

// Heartbeat timing
let time = 0

// Parametric 3D Heart Generator
const generateHeartParticles = (count = 1500) => {
  const particles = []
  const heartOffsetY = -55
  const scale = 11.2
  
  for (let i = 0; i < count; i++) {
    // Parameter t around heart curve
    const t = Math.PI * 2 * Math.random()
    
    // Heart 2D parametric equation
    const xBase = 16 * Math.pow(Math.sin(t), 3)
    const yBase = -(13 * Math.cos(t) - 5 * Math.cos(2 * t) - 2 * Math.cos(3 * t) - Math.cos(4 * t))
    
    // Volumetric dispersion (perimeter vs inner volume)
    const isPerimeter = Math.random() < 0.35
    const volumeFactor = isPerimeter ? (0.95 + Math.random() * 0.1) : (Math.pow(Math.random(), 0.55))
    
    // Thickness/depth z diminishes at sharp outer tips
    const maxZ = 8 * Math.sqrt(Math.max(0, 1 - Math.pow(volumeFactor, 2))) + 1
    const zBase = (Math.random() - 0.5) * 2 * maxZ
    
    // Palette: rich glowing rose, magenta, ruby, and gold/white stardust
    const colorRoll = Math.random()
    let color
    if (colorRoll < 0.45) {
      color = `rgba(255, 64, 129, ${0.7 + Math.random() * 0.3})` // Vivid rose
    } else if (colorRoll < 0.75) {
      color = `rgba(255, 117, 140, ${0.7 + Math.random() * 0.3})` // Soft pink
    } else if (colorRoll < 0.90) {
      color = `rgba(255, 215, 0, ${0.8 + Math.random() * 0.2})`   // Gold
    } else {
      color = `rgba(255, 255, 255, ${0.9 + Math.random() * 0.1})` // White starlight
    }

    const xPos = xBase * scale * volumeFactor
    const yPos = yBase * scale * volumeFactor + heartOffsetY
    const zPos = zBase * scale

    particles.push({
      x: xPos,
      y: yPos,
      z: zPos,
      baseX: xPos,
      baseY: yPos,
      baseZ: zPos,
      size: Math.random() * 2.2 + 1.2,
      color,
      pulseOffset: Math.random() * Math.PI * 2,
      orbitSpeed: (Math.random() - 0.5) * 0.008
    })
  }

  return particles
}

// 3D Spiral Galaxy Generator (Positioned right under the bottom tip of the heart)
const generateGalaxyParticles = (count = 1800) => {
  const particles = []
  const arms = 3
  const maxRadius = 560
  // Galaxy plane located right at the bottom tip of the heart (~y = +160)
  const galaxyBaseY = 160

  for (let i = 0; i < count; i++) {
    const arm = i % arms
    const armAngle = (arm * 2 * Math.PI) / arms
    
    // Radius distribution: concentrated near the center core and flowing outwards
    const dist = Math.pow(Math.random(), 0.8) * maxRadius + 18
    
    // Core vs Arm stardust: 75% of particles tightly hug the spiral arm spine
    const isArmSpine = Math.random() < 0.78
    const armJitter = isArmSpine
      ? (Math.random() - 0.5) * (0.16 + (dist / maxRadius) * 0.22) // Sharp, defined spiral arm
      : (Math.random() - 0.5) * 0.72                             // Ambient cosmic halo
    
    // Logarithmic-power spiral curve with pronounced winding (starts distinctly spiral)
    const spiralTurns = 3.2
    const spiralAngle = armAngle + Math.log(dist * 0.08 + 1) * spiralTurns + armJitter
    
    const x = Math.cos(spiralAngle) * dist
    const z = Math.sin(spiralAngle) * dist
    
    // Flat disc with a subtle cosmic vortex funnel towards the tip of the heart
    const vortexFunnel = Math.max(0, 1 - dist / maxRadius) * -20
    const verticalSpread = isArmSpine ? (10 + dist * 0.035) : (25 + dist * 0.06)
    const y = galaxyBaseY + vortexFunnel + (Math.random() - 0.5) * verticalSpread

    // Palette: glowing galactic core (gold/white), rose & violet spiral arms, cosmic diamond dust
    const distNorm = dist / maxRadius
    let color
    if (distNorm < 0.18) {
      // Golden luminous galactic core right at the tip of the heart
      color = Math.random() < 0.55 
        ? `rgba(255, 235, 150, ${0.85 + Math.random() * 0.15})`
        : `rgba(255, 255, 255, ${0.9 + Math.random() * 0.1})`
    } else if (distNorm < 0.55) {
      // Inner spiral arms: brilliant rose & magenta
      color = Math.random() < 0.65
        ? `rgba(255, 95, 150, ${0.65 + Math.random() * 0.3})`
        : `rgba(216, 70, 239, ${0.6 + Math.random() * 0.3})`
    } else if (distNorm < 0.82) {
      // Mid spiral arms: vibrant cosmic purple & electric celestial violet
      color = Math.random() < 0.6
        ? `rgba(168, 85, 247, ${0.55 + Math.random() * 0.35})`
        : `rgba(129, 140, 248, ${0.5 + Math.random() * 0.35})`
    } else {
      // Outer stardust rim: shimmering cyan-starlight & crystal white
      color = Math.random() < 0.5
        ? `rgba(56, 189, 248, ${0.45 + Math.random() * 0.35})`
        : `rgba(255, 255, 255, ${0.7 + Math.random() * 0.3})`
    }

    // Differential rotation speed: inner particles rotate faster, continuously coiling into an exquisite spiral
    const orbitSpeed = 0.0022 + (240 / (dist + 95)) * 0.0038

    particles.push({
      x,
      y,
      z,
      radius: dist,
      angle: spiralAngle,
      orbitSpeed,
      size: isArmSpine ? (Math.random() * 2.1 + 1.1) : (Math.random() * 1.4 + 0.7),
      color
    })
  }

  return particles
}

// Spawn burst stardust on touch / click
const spawnBurst = (originX, originY) => {
  emit('sparkleSound')
  emit('burstConfetti')

  for (let i = 0; i < 45; i++) {
    const angle = Math.random() * Math.PI * 2
    const speed = Math.random() * 5 + 2
    burstParticles.push({
      x: 0,
      y: 0,
      z: 0,
      vx: Math.cos(angle) * speed,
      vy: Math.sin(angle) * speed,
      vz: (Math.random() - 0.5) * speed * 2,
      life: 1,
      decay: Math.random() * 0.02 + 0.015,
      size: Math.random() * 3 + 1.5,
      color: Math.random() < 0.6 ? '#ff758c' : '#ffd54f'
    })
  }
}

// Resize Canvas
const handleResize = () => {
  if (!canvasRef.value) return
  width = window.innerWidth
  height = window.innerHeight
  const dpr = Math.min(window.devicePixelRatio || 1, 2)
  canvasRef.value.width = width * dpr
  canvasRef.value.height = height * dpr
  ctx = canvasRef.value.getContext('2d')
  ctx.scale(dpr, dpr)
}

// Pointer Events for 3D exploration
const onPointerDown = (e) => {
  isDragging = true
  startPointerX = e.clientX || (e.touches && e.touches[0].clientX) || 0
  startPointerY = e.clientY || (e.touches && e.touches[0].clientY) || 0
}

const onPointerMove = (e) => {
  const currentX = e.clientX || (e.touches && e.touches[0].clientX) || 0
  const currentY = e.clientY || (e.touches && e.touches[0].clientY) || 0

  if (isDragging) {
    const deltaX = currentX - startPointerX
    const deltaY = currentY - startPointerY
    targetRotY += deltaX * 0.006
    targetRotX += deltaY * 0.006
    startPointerX = currentX
    startPointerY = currentY
  } else {
    // Subtle auto-tilt towards cursor
    const normX = (currentX / width - 0.5) * 2
    const normY = (currentY / height - 0.5) * 2
    targetRotY = normX * 0.35
    targetRotX = normY * 0.22 + 0.32
  }
}

const onPointerUp = () => {
  isDragging = false
}

const onCanvasClick = (e) => {
  const rect = canvasRef.value.getBoundingClientRect()
  const clickX = (e.clientX || width / 2) - rect.left - width / 2
  const clickY = (e.clientY || height / 2) - rect.top - height / 2
  spawnBurst(clickX, clickY)
}

// Main 3D Animation Render Loop
const render = () => {
  time += 0.03

  // Smooth rotation easing
  rotX += (targetRotX - rotX) * 0.06
  rotY += (targetRotY - rotY) * 0.06

  // Natural continuous galaxy drift
  targetRotY += 0.0012

  // Realistic human double-beat pulse (lub-dub)
  const beatPrimary = Math.pow(Math.sin(time * 2.2), 63) * 0.14
  const beatSecondary = Math.pow(Math.sin(time * 2.2 + 0.55), 63) * 0.08
  const currentScale = 1 + beatPrimary + beatSecondary
  heartBeatScale.value = currentScale

  // Clear with soft space trail
  ctx.globalCompositeOperation = 'source-over'
  ctx.fillStyle = 'rgba(7, 2, 8, 0.28)' // Deep cosmic black/wine
  ctx.fillRect(0, 0, width, height)

  // Use additive blending for dazzling glowing starlight
  ctx.globalCompositeOperation = 'lighter'

  const fov = 500
  const cameraZ = 750
  const cosY = Math.cos(rotY)
  const sinY = Math.sin(rotY)
  const cosX = Math.cos(rotX)
  const sinX = Math.sin(rotX)

  // 1. Render Spiral Galaxy Particles
  for (let i = 0; i < galaxyParticles.length; i++) {
    const p = galaxyParticles[i]
    p.angle += p.orbitSpeed
    const gx = Math.cos(p.angle) * p.radius
    const gz = Math.sin(p.angle) * p.radius
    const gy = p.y

    // 3D rotation
    const x1 = gx * cosY - gz * sinY
    const z1 = gz * cosY + gx * sinY
    const y1 = gy * cosX - z1 * sinX
    const z2 = z1 * cosX + gy * sinX

    const depth = z2 + cameraZ
    if (depth > 10) {
      const proj = fov / depth
      const screenX = x1 * proj + width / 2
      const screenY = y1 * proj + height / 2
      const size = Math.max(0.5, p.size * proj * 1.5)

      ctx.fillStyle = p.color
      ctx.beginPath()
      ctx.arc(screenX, screenY, size, 0, Math.PI * 2)
      ctx.fill()
    }
  }

  // 2. Render Heart Particles
  for (let i = 0; i < heartParticles.length; i++) {
    const hp = heartParticles[i]
    
    // Pulsing coordinates relative to heart center (y = -55)
    const px = hp.baseX * currentScale
    const py = -55 + (hp.baseY - (-55)) * currentScale
    const pz = hp.baseZ * currentScale

    // 3D rotation
    const x1 = px * cosY - pz * sinY
    const z1 = pz * cosY + px * sinY
    const y1 = py * cosX - z1 * sinX
    const z2 = z1 * cosX + py * sinX

    const depth = z2 + cameraZ
    if (depth > 10) {
      const proj = fov / depth
      const screenX = x1 * proj + width / 2
      const screenY = y1 * proj + height / 2
      const particleSize = Math.max(0.6, hp.size * proj * 1.8)

      // Radial glowing core
      ctx.fillStyle = hp.color
      ctx.beginPath()
      ctx.arc(screenX, screenY, particleSize, 0, Math.PI * 2)
      ctx.fill()
    }
  }

  // 3. Render Burst Sparkles
  for (let i = burstParticles.length - 1; i >= 0; i--) {
    const bp = burstParticles[i]
    bp.x += bp.vx
    bp.y += bp.vy
    bp.z += bp.vz
    bp.life -= bp.decay

    if (bp.life <= 0) {
      burstParticles.splice(i, 1)
      continue
    }

    const x1 = bp.x * cosY - bp.z * sinY
    const z1 = bp.z * cosY + bp.x * sinY
    const y1 = bp.y * cosX - z1 * sinX
    const z2 = z1 * cosX + bp.y * sinX

    const depth = z2 + cameraZ
    if (depth > 10) {
      const proj = fov / depth
      const screenX = x1 * proj + width / 2
      const screenY = y1 * proj + height / 2
      const size = Math.max(0.8, bp.size * bp.life * proj * 2)

      ctx.fillStyle = bp.color
      ctx.beginPath()
      ctx.arc(screenX, screenY, size, 0, Math.PI * 2)
      ctx.fill()
    }
  }

  animationFrameId = requestAnimationFrame(render)
}

onMounted(() => {
  handleResize()
  heartParticles = generateHeartParticles(1500)
  galaxyParticles = generateGalaxyParticles(1800)

  window.addEventListener('resize', handleResize)
  window.addEventListener('pointerdown', onPointerDown)
  window.addEventListener('pointermove', onPointerMove)
  window.addEventListener('pointerup', onPointerUp)

  render()
})

onUnmounted(() => {
  if (animationFrameId) cancelAnimationFrame(animationFrameId)
  window.removeEventListener('resize', handleResize)
  window.removeEventListener('pointerdown', onPointerDown)
  window.removeEventListener('pointermove', onPointerMove)
  window.removeEventListener('pointerup', onPointerUp)
})
</script>

<template>
  <div class="galaxy-heart-viewport" @click="onCanvasClick">
    <!-- 3D WebGL/Canvas Cosmic Galaxy -->
    <canvas ref="canvasRef" class="galaxy-canvas"></canvas>

    <!-- Ambient Cosmic Nebula Glow -->
    <div class="cosmic-aura-glow"></div>

    <!-- TITLE ABOVE THE HEART: FELIZ CUMPLEAÑOS -->
    <div 
      class="heart-center-text-anchor"
      :style="{ transform: `translate(-50%, 0) scale(${1 + (heartBeatScale - 1) * 0.4})` }"
    >
      <h1 class="center-galaxy-title">
        <span class="title-text">Feliz Cumpleaños</span>
      </h1>
    </div>
  </div>
</template>

<style scoped>
/* ==========================================================================
   GALAXY HEART 3D VIEWPORT
   Galaxia 3D interactiva con corazón gigante y texto "Feliz Cumpleaños"
   ========================================================================== */
.galaxy-heart-viewport {
  position: fixed;
  inset: 0;
  width: 100vw;
  height: 100vh;
  background-color: #050106;
  overflow: hidden;
  user-select: none;
  z-index: 100;
  display: flex;
  flex-direction: column;
  justify-content: space-between;
  align-items: center;
  cursor: grab;
}

.galaxy-heart-viewport:active {
  cursor: grabbing;
}

.galaxy-canvas {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  display: block;
  z-index: 1;
}

.cosmic-aura-glow {
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  width: 70vw;
  height: 70vh;
  background: radial-gradient(circle, rgba(255, 64, 129, 0.18) 0%, rgba(147, 51, 234, 0.12) 40%, transparent 70%);
  filter: blur(60px);
  pointer-events: none;
  z-index: 2;
}

/* Top Navigation */
.galaxy-top-controls {
  position: relative;
  z-index: 10;
  width: 100%;
  max-width: 1100px;
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 1.2rem 1.5rem;
  box-sizing: border-box;
  pointer-events: auto;
}

.btn-cosmic-back {
  display: inline-flex;
  align-items: center;
  gap: 0.5rem;
  background: rgba(255, 255, 255, 0.1);
  border: 1px solid rgba(255, 255, 255, 0.25);
  color: #ffffff;
  padding: 0.55rem 1.2rem;
  border-radius: 9999px;
  font-family: 'Montserrat', sans-serif;
  font-size: 0.85rem;
  font-weight: 600;
  cursor: pointer;
  backdrop-filter: blur(12px);
  -webkit-backdrop-filter: blur(12px);
  transition: all 0.25s ease;
}

.btn-cosmic-back:hover {
  background: rgba(255, 255, 255, 0.22);
  transform: translateX(-3px);
}

.cosmic-badge {
  display: inline-flex;
  align-items: center;
  gap: 0.45rem;
  padding: 0.35rem 1.1rem;
  border-radius: 9999px;
  background: rgba(255, 255, 255, 0.08);
  border: 1px solid rgba(255, 255, 255, 0.22);
  backdrop-filter: blur(12px);
  font-family: 'Montserrat', sans-serif;
  font-size: 0.72rem;
  font-weight: 700;
  letter-spacing: 0.16em;
  color: #ffd0dc;
  text-transform: uppercase;
}

.sparkle-star {
  color: #ffd54f;
}

.placeholder-top {
  width: 90px;
}

/* TITLE ABOVE THE HEART: FELIZ CUMPLEAÑOS */
.heart-center-text-anchor {
  position: absolute;
  top: 14%;
  left: 50%;
  transform: translate(-50%, 0);
  z-index: 10;
  pointer-events: none;
  text-align: center;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  width: 90%;
  max-width: 680px;
  transition: transform 0.15s ease-out;
}

.center-galaxy-title {
  margin: 0;
  font-family: 'Playfair Display', Georgia, serif;
  font-size: clamp(2.3rem, 6.5vw, 4.2rem);
  font-weight: 800;
  color: #ffffff;
  letter-spacing: -0.01em;
  line-height: 1.15;
  text-shadow: 
    0 0 10px rgba(255, 255, 255, 0.9),
    0 0 25px rgba(255, 105, 180, 0.95),
    0 0 50px rgba(255, 105, 180, 0.95),
    0 0 85px rgba(255, 105, 180, 0.95);
}

.title-text {
  background: linear-gradient(100deg, #ffffff 0%, #ffffff 45%, #ffffff 100%);
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  filter: drop-shadow(0 0 20px rgba(255, 105, 180, 0.95));
}

.center-galaxy-caption {
  font-family: 'Montserrat', sans-serif;
  font-size: clamp(0.78rem, 2vw, 0.95rem);
  color: #ffe6ec;
  margin-top: 0.8rem;
  letter-spacing: 0.08em;
  text-transform: uppercase;
  font-weight: 600;
  text-shadow: 0 0 15px rgba(255, 117, 140, 0.8);
  opacity: 0.95;
}

/* Bottom Controls */
.galaxy-bottom-controls {
  position: relative;
  z-index: 10;
  width: 100%;
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 1.2rem;
  padding: 0 1rem 2.2rem 1rem;
  box-sizing: border-box;
}

.interaction-hint {
  font-family: 'Montserrat', sans-serif;
  font-size: clamp(0.72rem, 1.8vw, 0.82rem);
  color: rgba(255, 230, 238, 0.85);
  letter-spacing: 0.04em;
  text-align: center;
  text-shadow: 0 0 8px rgba(0, 0, 0, 0.8);
}

/* Mobile Adjustments */
@media (max-width: 640px) {
  .galaxy-top-controls {
    padding: 0.8rem 1rem;
  }

  .heart-center-text-anchor {
    top: 11%;
  }

  .placeholder-top {
    display: none;
  }

  .center-galaxy-title {
    font-size: 2.2rem;
  }

  .center-galaxy-caption {
    font-size: 0.72rem;
  }

  .interaction-hint {
    font-size: 0.68rem;
  }
}
</style>
