<script setup>
import { ref, onMounted } from 'vue'

const props = defineProps({
  audioEnabled: {
    type: Boolean,
    default: true
  }
})

const emit = defineEmits(['continueToFinal', 'sparkleSound', 'burstConfetti'])

const polaroids = ref([
  {
    id: 1,
    src: '/imagen1.jpg',
    caption: 'Momentos inolvidables ✨',
    rotation: -7,
    tilt: -3
  },
  {
    id: 2,
    src: '/imagen2.jpg',
    caption: 'Tu hermosa sonrisa 💖',
    rotation: 5,
    tilt: 2
  },
  {
    id: 3,
    src: '/imagen3.jpg',
    caption: 'Celebrando tu día 🎂',
    rotation: -4,
    tilt: -1
  }
])

const handleContinue = () => {
  emit('sparkleSound')
  emit('burstConfetti')
  emit('continueToFinal')
}

onMounted(() => {
  emit('sparkleSound')
  emit('burstConfetti')
})
</script>

<template>
  <div class="birthday-wish-wrapper">
    <!-- Starry Romantic Atmosphere Background -->
    <div class="wish-ambient-halo"></div>

    <!-- EL CUADRO (Contenedor Enmarcado) -->
    <div class="wish-frame-card">
      <!-- Topbar with Decorative Glass Hardware Dots -->
      <div class="wish-card-topbar">
        <div class="card-dots">
          <span class="dot dot-rose"></span>
          <span class="dot dot-gold"></span>
          <span class="dot dot-white"></span>
        </div>
      </div>

      <!-- Card Inner Body: Split Layout with Cascade Polaroids on Left -->
      <div class="wish-card-body">
        
        <!-- LADO IZQUIERDO: Collage de 3 Polaroids en Cascada Vertical + Stickers Festivos -->
        <div class="wish-polaroid-side">
          <div class="polaroid-cascade-container">
            
            <!-- Stickers Ilustrados 2D Festivos de Fondo / Alrededor -->
            <div class="sticker-2d sticker-balloon" title="Globos festivos">🎈</div>
            <div class="sticker-2d sticker-sparkles-left" title="Chispas">✨</div>
            <div class="sticker-2d sticker-party-popper" title="Fiesta">🎉</div>
            <div class="sticker-2d sticker-gift" title="Regalito">🎁</div>
            <div class="sticker-2d sticker-cupcake" title="Dulzura">🧁</div>
            <div class="sticker-2d sticker-tape tape-top" title="Washi tape"></div>
            <div class="sticker-2d sticker-heart" title="Amor">💖</div>

            <!-- Las 3 Fotos Polaroid Superpuestas en Cascada -->
            <div 
              v-for="(photo, index) in polaroids" 
              :key="photo.id"
              class="polaroid-item"
              :class="`polaroid-pos-${index + 1}`"
            >
              <div class="polaroid-pin">📌</div>
              <div class="polaroid-frame">
                <div class="polaroid-photo-wrapper">
                  <img :src="photo.src" :alt="photo.caption" class="polaroid-img" />
                  <div class="polaroid-glare"></div>
                </div>
                <div class="polaroid-footer">
                  <span class="polaroid-caption">{{ photo.caption }}</span>
                </div>
              </div>
            </div>

          </div>
        </div>

        <!-- LADO DERECHO: Título y Mensaje de Cumpleaños -->
        <div class="wish-message-side">
          <div class="wish-header-icon">🎂✨</div>
          <h2 class="wish-card-title">Deseo de Cumpleaños</h2>
          <div class="wish-title-divider">
            <span class="divider-line"></span>
            <span class="divider-heart">♡</span>
            <span class="divider-line"></span>
          </div>

          <p class="wish-card-text">
            Te deseo felicidad, buena salud, éxito e infinitas bendiciones en este nuevo capítulo de tu vida. Que todos tus sueños se hagan realidad, que tus días estén llenos de alegría y que tu corazón siempre encuentre paz. Gracias por ser una parte tan maravillosa de mi vida. Espero que este año te traiga todo lo que has estado deseando. Mantente feliz, mantente saludable y sigue brillando.
          </p>

          <div class="wish-signature">
            <span class="sign-heart">♥</span> Con todo mi cariño siempre <span class="sign-heart">♥</span>
          </div>
        </div>

      </div>
    </div>

    <!-- FUERA DEL CONTENEDOR: BOTÓN CONTINUAR -->
    <div class="wish-outer-action">
      <button 
        id="btn-continuar-final"
        class="btn-open-letter-pill"
        @click="handleContinue"
      >
        <span class="btn-heart">♡</span>
        <span class="btn-caption">Continuar</span>
        <span class="btn-heart">♡</span>
      </button>
    </div>
  </div>
</template>

<style scoped>
/* ==========================================================================
   SECCIÓN: DESEO DE CUMPLEAÑOS
   Cuadro enmarcado con collage Polaroid en cascada y stickers 2D
   ========================================================================== */
.birthday-wish-wrapper {
  position: relative;
  width: 100%;
  max-width: 980px;
  margin: 1.5rem auto 3rem auto;
  padding: 1rem 1.2rem;
  box-sizing: border-box;
  display: flex;
  flex-direction: column;
  align-items: center;
  z-index: 10;
  animation: wishFadeIn 0.75s cubic-bezier(0.16, 1, 0.3, 1) both;
}

@keyframes wishFadeIn {
  from {
    opacity: 0;
    transform: translateY(25px) scale(0.96);
  }
  to {
    opacity: 1;
    transform: translateY(0) scale(1);
  }
}

.wish-ambient-halo {
  position: absolute;
  top: 15%;
  left: 50%;
  transform: translate(-50%, -10%);
  width: 90%;
  height: 75%;
  background: radial-gradient(circle, rgba(255, 117, 140, 0.22) 0%, rgba(255, 215, 0, 0.1) 45%, transparent 75%);
  filter: blur(55px);
  pointer-events: none;
  z-index: 1;
}

/* El Cuadro Enmarcado (Card) */
.wish-frame-card {
  position: relative;
  z-index: 3;
  width: 100%;
  background: linear-gradient(145deg, rgba(38, 12, 23, 0.95) 0%, rgba(20, 6, 13, 0.97) 100%);
  border: 2px solid rgba(255, 215, 225, 0.45);
  border-radius: 24px;
  box-shadow: 
    0 25px 65px rgba(25, 5, 15, 0.55),
    0 0 40px rgba(255, 117, 140, 0.2),
    inset 0 1px 2px rgba(255, 255, 255, 0.45);
  backdrop-filter: blur(20px);
  -webkit-backdrop-filter: blur(20px);
  overflow: hidden;
  box-sizing: border-box;
  transition: transform 0.3s ease, box-shadow 0.3s ease;
}

.wish-frame-card:hover {
  box-shadow: 
    0 30px 75px rgba(25, 5, 15, 0.65),
    0 0 50px rgba(255, 117, 140, 0.28),
    inset 0 1px 3px rgba(255, 255, 255, 0.65);
}

/* Topbar */
.wish-card-topbar {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 0.75rem 1.4rem;
  background: rgba(255, 255, 255, 0.05);
  border-bottom: 1px solid rgba(255, 255, 255, 0.1);
}

.card-dots {
  display: flex;
  align-items: center;
  gap: 7px;
}

.dot {
  width: 10px;
  height: 10px;
  border-radius: 50%;
  display: inline-block;
  box-shadow: 0 0 6px currentColor;
}

.dot-rose { background: #ff4d79; color: #ff4d79; }
.dot-gold { background: #ffd54f; color: #ffd54f; }
.dot-white { background: rgba(255, 255, 255, 0.75); color: #ffffff; }

.card-badge {
  display: flex;
  align-items: center;
  gap: 0.45rem;
  font-family: 'Montserrat', sans-serif;
  font-size: 0.74rem;
  font-weight: 600;
  letter-spacing: 0.12em;
  color: #ffccd5;
  text-transform: uppercase;
}

.badge-icon {
  font-size: 0.85rem;
}

/* Body: 2 Column Layout (Polaroids Left, Wish Right) */
.wish-card-body {
  padding: 2.2rem 2rem 2.6rem 2rem;
  display: grid;
  grid-template-columns: 320px 1fr;
  gap: 2.2rem;
  align-items: center;
  box-sizing: border-box;
}

/* ==========================================================================
   POLAROID CASCADE (LADO IZQUIERDO)
   ========================================================================== */
.wish-polaroid-side {
  position: relative;
  display: flex;
  justify-content: center;
  align-items: center;
  min-height: 440px;
  padding: 1rem 0;
}

.polaroid-cascade-container {
  position: relative;
  width: 250px;
  height: 420px;
}

/* Polaroid Card Item */
.polaroid-item {
  position: absolute;
  width: 200px;
  background: #ffffff;
  padding: 10px 10px 16px 10px;
  border-radius: 6px;
  box-shadow: 
    0 12px 28px rgba(0, 0, 0, 0.45),
    0 4px 10px rgba(0, 0, 0, 0.25);
  box-sizing: border-box;
  transition: transform 0.35s cubic-bezier(0.175, 0.885, 0.32, 1.275), z-index 0.2s ease, box-shadow 0.3s ease;
  cursor: pointer;
  border: 1px solid rgba(0, 0, 0, 0.08);
}

.polaroid-item:hover {
  transform: scale(1.09) rotate(0deg) !important;
  z-index: 20 !important;
  box-shadow: 
    0 20px 40px rgba(0, 0, 0, 0.6),
    0 0 25px rgba(255, 117, 140, 0.45);
}

/* Cascada vertical superpuesta con jerarquía de tamaños:
   Foto 1: Arriba (más pequeña, secundaria)
   Foto 2: Al medio (más grande, protagonista destacada)
   Foto 3: Abajo (más pequeña, secundaria) */
   
.polaroid-pos-1 {
  width: 165px;
  top: 15px;
  left: 20px;
  transform: rotate(-7deg);
  z-index: 2;
}

.polaroid-pos-1 .polaroid-photo-wrapper {
  height: 120px;
}

.polaroid-pos-1 .polaroid-caption {
  font-size: 0.88rem;
}

.polaroid-pos-2 {
  width: 215px;
  top: 110px;
  left: 45px;
  transform: rotate(5deg);
  z-index: 5;
  box-shadow: 
    0 16px 36px rgba(0, 0, 0, 0.55),
    0 0 20px rgba(255, 117, 140, 0.25);
}

.polaroid-pos-2 .polaroid-photo-wrapper {
  height: 160px;
}

.polaroid-pos-2 .polaroid-caption {
  font-size: 1.05rem;
}

.polaroid-pos-3 {
  width: 170px;
  top: 255px;
  left: 25px;
  transform: rotate(-4deg);
  z-index: 6;
}

.polaroid-pos-3 .polaroid-photo-wrapper {
  height: 125px;
}

.polaroid-pos-3 .polaroid-caption {
  font-size: 0.9rem;
}

/* Chincheta decorativa en la foto */
.polaroid-pin {
  position: absolute;
  top: -8px;
  left: 50%;
  transform: translateX(-50%);
  font-size: 1.1rem;
  z-index: 10;
  filter: drop-shadow(0 2px 4px rgba(0, 0, 0, 0.3));
}

.polaroid-frame {
  display: flex;
  flex-direction: column;
}

.polaroid-photo-wrapper {
  position: relative;
  width: 100%;
  height: 150px;
  border-radius: 3px;
  overflow: hidden;
  background: #1a1a1a;
}

.polaroid-img {
  width: 100%;
  height: 100%;
  object-fit: cover;
  display: block;
  transition: transform 0.4s ease;
}

.polaroid-item:hover .polaroid-img {
  transform: scale(1.05);
}

.polaroid-glare {
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background: linear-gradient(135deg, rgba(255, 255, 255, 0.25) 0%, transparent 60%);
  pointer-events: none;
}

.polaroid-footer {
  margin-top: 10px;
  text-align: center;
}

.polaroid-caption {
  font-family: 'Caveat', 'Dancing Script', 'Indie Flower', cursive, sans-serif;
  font-size: 1rem;
  font-weight: 700;
  color: #2b1f24;
  letter-spacing: 0.02em;
  display: block;
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
}

/* ==========================================================================
   STICKERS FESTIVOS 2D ALREDEDOR DE LAS FOTOS
   ========================================================================== */
.sticker-2d {
  position: absolute;
  pointer-events: none;
  user-select: none;
  filter: drop-shadow(0 4px 8px rgba(0, 0, 0, 0.35));
  animation: stickerFloat 3.5s ease-in-out infinite alternate;
  z-index: 8;
}

@keyframes stickerFloat {
  0% { transform: translateY(0) rotate(0deg); }
  100% { transform: translateY(-6px) rotate(4deg); }
}

.sticker-balloon {
  top: -20px;
  left: -20px;
  font-size: 2.2rem;
  animation-delay: 0.2s;
}

.sticker-party-popper {
  top: 90px;
  right: -25px;
  font-size: 2rem;
  animation-delay: 0.8s;
  transform: rotate(-15deg);
}

.sticker-sparkles-left {
  top: 180px;
  left: -28px;
  font-size: 1.8rem;
  animation-delay: 1.2s;
}

.sticker-gift {
  bottom: 10px;
  right: -15px;
  font-size: 2rem;
  animation-delay: 0.5s;
}

.sticker-cupcake {
  bottom: -15px;
  left: 30px;
  font-size: 1.9rem;
  animation-delay: 1.5s;
}

.sticker-heart {
  top: 45px;
  right: 15px;
  font-size: 1.5rem;
  animation-delay: 0.4s;
}

.sticker-tape {
  width: 55px;
  height: 18px;
  background: rgba(255, 235, 150, 0.85);
  box-shadow: 0 1px 3px rgba(0, 0, 0, 0.2);
  border-left: 2px dashed rgba(200, 180, 100, 0.4);
  border-right: 2px dashed rgba(200, 180, 100, 0.4);
}

.tape-top {
  top: -6px;
  left: 55px;
  transform: rotate(-5deg);
  z-index: 10;
}

/* ==========================================================================
   LADO DERECHO: MENSAJE DE CUMPLEAÑOS
   ========================================================================== */
.wish-message-side {
  display: flex;
  flex-direction: column;
  align-items: center;
  text-align: center;
  padding: 0 0.5rem;
}

.wish-header-icon {
  font-size: 2.2rem;
  margin-bottom: 0.4rem;
  filter: drop-shadow(0 4px 12px rgba(255, 117, 140, 0.4));
}

.wish-card-title {
  font-family: 'Playfair Display', Georgia, serif;
  font-size: clamp(1.8rem, 3.8vw, 2.6rem);
  font-weight: 800;
  color: #ffffff;
  margin: 0;
  letter-spacing: -0.01em;
  line-height: 1.2;
  text-shadow: 0 4px 20px rgba(45, 10, 20, 0.6);
}

.wish-title-divider {
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 0.8rem;
  width: 100%;
  max-width: 240px;
  margin: 1rem auto 1.4rem auto;
}

.divider-line {
  flex: 1;
  height: 1px;
  background: linear-gradient(90deg, transparent, rgba(255, 215, 225, 0.6), transparent);
}

.divider-heart {
  color: #ff758c;
  font-size: 1rem;
}

.wish-card-text {
  font-family: 'Playfair Display', Georgia, serif;
  font-size: clamp(0.98rem, 1.8vw, 1.15rem);
  line-height: 1.8;
  color: #fff2f5;
  margin: 0;
  letter-spacing: 0.015em;
  text-shadow: 0 2px 8px rgba(0, 0, 0, 0.35);
  font-weight: 400;
}

.wish-signature {
  margin-top: 1.6rem;
  font-family: 'Montserrat', sans-serif;
  font-size: 0.85rem;
  font-weight: 600;
  letter-spacing: 0.08em;
  color: #ffccd5;
  text-transform: uppercase;
  display: flex;
  align-items: center;
  gap: 0.5rem;
}

.sign-heart {
  color: #ff4d79;
}

/* FUERA DEL CONTENEDOR: BOTÓN CONTINUAR */
.wish-outer-action {
  position: relative;
  z-index: 5;
  margin-top: 3rem;
  margin-bottom: 2rem;
  display: flex;
  justify-content: center;
  align-items: center;
  width: 100%;
}

/* ==========================================================================
   RESPONSIVE DESIGN
   ========================================================================== */
@media (max-width: 820px) {
  .wish-card-body {
    grid-template-columns: 1fr;
    gap: 3.4rem;
    padding: 2rem 1.4rem;
  }

  .wish-polaroid-side {
    min-height: 380px;
  }

  .polaroid-cascade-container {
    width: 260px;
    height: 390px;
  }

  .polaroid-pos-1 {
    width: 155px;
    top: 10px;
    left: 20px;
  }
  .polaroid-pos-1 .polaroid-photo-wrapper {
    height: 110px;
  }

  .polaroid-pos-2 {
    width: 195px;
    top: 125px;
    left: 45px;
  }
  .polaroid-pos-2 .polaroid-photo-wrapper {
    height: 145px;
  }

  .polaroid-pos-3 {
    width: 155px;
    top: 280px;
    left: 25px;
  }
  .polaroid-pos-3 .polaroid-photo-wrapper {
    height: 115px;
  }
}

@media (max-width: 640px) {
  .wish-card-body {
    padding: 1.8rem 1rem 2.2rem 1rem;
  }

  .wish-card-title {
    font-size: 1.75rem;
  }

  .wish-card-text {
    font-size: 0.98rem;
    line-height: 1.7;
  }

  .wish-outer-action {
    margin-top: 2.2rem;
  }
}
</style>
