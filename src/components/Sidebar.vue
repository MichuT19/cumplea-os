<script setup>
import { Heart, Package, User, LogOut } from 'lucide-vue-next'
import { inject } from 'vue'

const props = defineProps({
  currentView: {
    type: String,
    required: true
  }
})

const emit = defineEmits(['change-view', 'logout'])
const goToLogin = inject('goToLogin')

const handleLogout = () => {
  emit('logout')
}
</script>

<template>
  <aside class="sidebar animate-fade-in">
    <!-- User Profile (Top) -->
    <div class="user-profile">
      <div class="user-avatar">
        <User :size="20" />
      </div>
      <div class="user-details">
        <span class="user-name">Administrador</span>
        <span class="user-role">Panel de control</span>
      </div>
    </div>

    <!-- Navigation (Middle) -->
    <nav class="nav-menu">
      <div class="nav-section-title">Navegación</div>
      <button 
        @click="emit('change-view', 'birthday')" 
        class="nav-link"
        :class="{ active: currentView === 'birthday' }"
      >
        <Heart :size="20" />
        <span>Tu Día Especial ✨</span>
        <div v-if="currentView === 'birthday'" class="active-indicator"></div>
      </button>
      <button 
        @click="emit('change-view', 'catalog')" 
        class="nav-link"
        :class="{ active: currentView === 'catalog' }"
      >
        <Package :size="20" />
        <span>Catálogo</span>
        <div v-if="currentView === 'catalog'" class="active-indicator"></div>
      </button>
      <button 
        @click="emit('change-view', 'products')" 
        class="nav-link"
        :class="{ active: currentView === 'products' }"
      >
        <Package :size="20" />
        <span>Productos</span>
        <div v-if="currentView === 'products'" class="active-indicator"></div>
      </button>
    </nav>

    <!-- Footer Actions (Bottom) -->
    <div class="sidebar-footer">
  <button @click="goToLogin" class="nav-link login-button">
    <User :size="20" />
    <span>Inicio de sesión</span>
  </button>
      <!-- Logout Button -->
      <button @click="handleLogout" class="btn-logout">
        <LogOut :size="20" />
        <span>Cerrar sesión</span>
      </button>
      
      <div class="footer-credit">
        <p>Made with Vue 3 <Heart :size="10" class="heart-icon" /></p>
      </div>
    </div>
  </aside>
</template>

<style scoped>
@import url('https://fonts.googleapis.com/css2?family=Nunito:wght@400;600;700&display=swap');

.sidebar {
  /* Variables de la temática rosada/pasteles para que coincida con ProductFormView */
  --cake-primary: #e84fa0;
  --cake-primary-foreground: #ffffff;
  --cake-card-bg: #ffffff;
  --cake-border: #f0d4e4;
  --cake-input: #fde8f2;
  --cake-muted: #f7d6ea;
  --cake-muted-foreground: #b07090;
  --cake-foreground: #3a1a2e;

  width: 260px;
  height: 100vh;
  position: fixed;
  top: 0;
  left: 0;
  background-color: var(--cake-card-bg);
  border-right: 1px solid var(--cake-border);
  display: flex;
  flex-direction: column;
  padding: 2.5rem 1.5rem;
  z-index: 50;
  box-sizing: border-box;
  font-family: 'Nunito', sans-serif;
  color: var(--cake-foreground);
}

/* User Profile Styles */
.user-profile {
  display: flex;
  align-items: center;
  gap: 1rem;
  margin-bottom: 3rem; /* Espaciado extra abajo para separar de la navegación */
  padding: 0.5rem;
}

.user-avatar {
  width: 44px;
  height: 44px;
  border-radius: 12px;
  background: linear-gradient(135deg, var(--cake-primary), #ff9ecf);
  color: var(--cake-primary-foreground);
  display: flex;
  align-items: center;
  justify-content: center;
  box-shadow: 0 4px 10px rgba(232, 79, 160, 0.2);
}

.user-details {
  display: flex;
  flex-direction: column;
}

.user-name {
  font-size: 1rem;
  font-weight: 700;
  color: var(--cake-foreground);
}

.user-role {
  font-size: 0.8rem;
  color: var(--cake-muted-foreground);
  font-weight: 600;
}

/* Navigation Menu */
.nav-menu {
  flex: 1;
  display: flex;
  flex-direction: column;
  gap: 0.5rem;
}

.nav-section-title {
  font-size: 0.75rem;
  text-transform: uppercase;
  letter-spacing: 0.05em;
  font-weight: 800;
  color: var(--cake-muted-foreground);
  padding: 0.5rem 0.75rem;
  margin-bottom: 0.25rem;
  opacity: 0.7;
}

.nav-link {
  display: flex;
  align-items: center;
  gap: 0.75rem;
  padding: 0.875rem 1rem;
  border-radius: 12px;
  color: var(--cake-muted-foreground);
  font-weight: 600;
  font-size: 0.95rem;
  cursor: pointer;
  background: transparent;
  border: none;
  transition: all 0.2s ease;
  text-align: left;
  position: relative;
  overflow: hidden;
  font-family: 'Nunito', sans-serif;
}

.nav-link:hover {
  background-color: var(--cake-input);
  color: var(--cake-foreground);
}

.nav-link.active {
  background-color: var(--cake-input);
  color: var(--cake-primary);
  font-weight: 700;
}

.active-indicator {
  position: absolute;
  left: 0;
  top: 25%;
  bottom: 25%;
  width: 4px;
  background-color: var(--cake-primary);
  border-radius: 0 4px 4px 0;
}

/* Footer Section */
.sidebar-footer {
  margin-top: auto;
  display: flex;
  flex-direction: column;
  gap: 1rem;
}

.btn-logout {
  width: 100%;
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 0.75rem;
  padding: 0.75rem;
  background-color: var(--cake-input);
  border: 1px solid var(--cake-border);
  border-radius: 12px;
  color: var(--cake-foreground);
  font-size: 0.875rem;
  font-weight: 600;
  font-family: 'Nunito', sans-serif;
  cursor: pointer;
  transition: all 0.2s ease;
}

.btn-logout:hover {
  background-color: #fff0f5;
  color: #dc2626;
  border-color: #fca5a5;
  transform: translateY(-1px);
}

.footer-credit {
  font-size: 0.75rem;
  color: var(--cake-muted-foreground);
  text-align: center;
  opacity: 0.7;
}

.heart-icon {
  color: var(--cake-primary);
  display: inline;
  vertical-align: middle;
}

@media (max-width: 1024px) {
  .sidebar {
    display: none;
  }
}
</style>
