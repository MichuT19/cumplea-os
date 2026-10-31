<script setup>
import { ref, provide } from 'vue'
import Sidebar from './components/Sidebar.vue'
import ProductFormView from './components/ProductFormView.vue'
import Catalogo from './components/catalogo.vue'
import LoginView from './components/LoginView.vue'
import BirthdayExperience from './components/BirthdayExperience.vue'

const isLoggedIn = ref(true)
const currentView = ref('birthday')

provide('goToLogin', () => { isLoggedIn.value = false })

const handleLoginSuccess = () => {
  isLoggedIn.value = true
}

const handleLogout = () => {
  isLoggedIn.value = false
}

const changeView = (view) => {
  currentView.value = view
}
</script>

<template>
  <div v-if="!isLoggedIn">
    <LoginView @login-success="handleLoginSuccess" />
  </div>
  
  <!-- Full Screen Immersive Birthday Experience -->
  <div v-else-if="currentView === 'birthday'" class="birthday-full-container">
    <BirthdayExperience @back-to-catalog="changeView('catalog')" />
  </div>

  <!-- Standard Admin / Catalog Container -->
  <div v-else class="app-container">
    <!-- Sidebar -->
    <Sidebar :current-view="currentView" @change-view="changeView" @logout="handleLogout" />

    <!-- Main Content Area -->
    <main class="main-content">
      <!-- Floating button to easily return to the Birthday Special -->
      <button 
        class="floating-birthday-badge"
        @click="changeView('birthday')"
        title="Ver la experiencia de Tu Día Especial"
      >
        ✨ Ver Tu Día Especial
      </button>

      <!-- Dynamic View Component Switching -->
      <transition name="fade-view" mode="out-in">
        <component :is="currentView === 'products' ? ProductFormView : (currentView === 'catalog' ? Catalogo : null)" />
      </transition>
    </main>
  </div>
</template>

<style>
/* Full container for Birthday Page */
.birthday-full-container {
  width: 100vw;
  min-height: 100vh;
  margin: 0;
  padding: 0;
}

/* Floating badge in dashboard to jump back to birthday page */
.floating-birthday-badge {
  position: fixed;
  bottom: 1.5rem;
  right: 1.5rem;
  z-index: 100;
  background: linear-gradient(135deg, #ff758c 0%, #ff5277 50%, #d83159 100%);
  color: #ffffff;
  border: 1px solid rgba(255, 255, 255, 0.4);
  padding: 0.65rem 1.25rem;
  border-radius: 9999px;
  font-weight: 700;
  font-size: 0.9rem;
  cursor: pointer;
  box-shadow: 0 8px 25px rgba(216, 49, 89, 0.45);
  transition: all 0.25s cubic-bezier(0.34, 1.56, 0.64, 1);
  display: flex;
  align-items: center;
  gap: 0.4rem;
}

.floating-birthday-badge:hover {
  transform: translateY(-3px) scale(1.05);
  box-shadow: 0 12px 30px rgba(216, 49, 89, 0.65);
}

/* Transition between views */
.fade-view-enter-active,
.fade-view-leave-active {
  transition: opacity 0.25s ease, transform 0.25s ease;
}

.fade-view-enter-from {
  opacity: 0;
  transform: translateY(10px);
}

.fade-view-leave-to {
  opacity: 0;
  transform: translateY(-10px);
}
</style>
