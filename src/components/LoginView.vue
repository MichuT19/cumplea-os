<script setup>
import { ref } from 'vue'
import { Lock, Mail, Loader2, UserCircle } from 'lucide-vue-next'
// import { supabase } from '../supabase' // Para usar cuando conectes auth real

const email = ref('')
const password = ref('')
const isSubmitting = ref(false)
const errorMessage = ref('')

const emit = defineEmits(['login-success'])

const handleLogin = async () => {
  isSubmitting.value = true
  errorMessage.value = ''
  
  // Simulación de carga para que se vea premium
  setTimeout(() => {
    if (email.value && password.value) {
      isSubmitting.value = false
      emit('login-success') // Cambia el estado a logueado en App.vue
    } else {
      isSubmitting.value = false
      errorMessage.value = 'Por favor, completa ambos campos.'
    }
  }, 1200)

  // LOGICA REAL CON SUPABASE (Descomentar cuando quieras activarlo)
  /*
  const { data, error } = await supabase.auth.signInWithPassword({
    email: email.value,
    password: password.value,
  })
  if (error) {
    errorMessage.value = error.message
    isSubmitting.value = false
  } else {
    emit('login-success')
  }
  */
}
</script>

<template>
  <div class="login-container animate-fade-in">
    <div class="login-card glass">
      
      <!-- Encabezado del Login -->
      <div class="login-header">
        <div class="logo-wrapper">
          <UserCircle :size="48" class="logo-icon" />
        </div>
        <h1 class="login-title">Bienvenido de nuevo</h1>
        <p class="login-subtitle">Ingresa a tu panel de control</p>
      </div>

      <!-- Formulario -->
      <form @submit.prevent="handleLogin" class="login-form">
        <div class="form-group">
          <label for="email">Correo Electrónico</label>
          <div class="input-wrapper">
            <Mail class="input-icon" :size="18" />
            <input 
              type="email" 
              id="email" 
              v-model="email" 
              placeholder="tu@correo.com" 
              required 
            />
          </div>
        </div>

        <div class="form-group">
          <label for="password">Contraseña</label>
          <div class="input-wrapper">
            <Lock class="input-icon" :size="18" />
            <input 
              type="password" 
              id="password" 
              v-model="password" 
              placeholder="••••••••" 
              required 
            />
          </div>
          <div class="forgot-password">
            <a href="#" @click.prevent>¿Olvidaste tu contraseña?</a>
          </div>
        </div>

        <div v-if="errorMessage" class="error-message">
          {{ errorMessage }}
        </div>

        <button type="submit" class="btn-login" :disabled="isSubmitting">
          <Loader2 v-if="isSubmitting" class="animate-spin" :size="20" />
          <span>{{ isSubmitting ? 'Iniciando sesión...' : 'Ingresar' }}</span>
        </button>
      </form>
      
    </div>
  </div>
</template>

<style scoped>
@import url('https://fonts.googleapis.com/css2?family=Nunito:wght@400;600;700;800&display=swap');

.login-container {
  /* Temática rosada de la aplicación */
  --cake-primary: #e84fa0;
  --cake-primary-hover: #d13d8b;
  --cake-primary-foreground: #ffffff;
  --cake-card-bg: #ffffff;
  --cake-border: #f0d4e4;
  --cake-input: #fde8f2;
  --cake-muted: #f7d6ea;
  --cake-muted-foreground: #b07090;
  --cake-foreground: #3a1a2e;

  min-height: 100vh;
  width: 100vw;
  display: flex;
  align-items: center;
  justify-content: center;
  background: linear-gradient(135deg, #fdf2f8, #fbcfe8); /* Fondo general de la app */
  font-family: 'Nunito', sans-serif;
  padding: 1rem;
}

.login-card {
  width: 100%;
  max-width: 420px;
  background-color: var(--cake-card-bg);
  border-radius: 1.5rem;
  padding: 2.5rem 2rem;
  box-shadow: 0 10px 40px rgba(232, 79, 160, 0.1);
  border: 1px solid var(--cake-border);
}

.login-header {
  text-align: center;
  margin-bottom: 2rem;
}

.logo-wrapper {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  width: 80px;
  height: 80px;
  border-radius: 50%;
  background: linear-gradient(135deg, var(--cake-primary), #ff9ecf);
  color: var(--cake-primary-foreground);
  margin-bottom: 1rem;
  box-shadow: 0 8px 20px rgba(232, 79, 160, 0.25);
}

.login-title {
  font-size: 1.5rem;
  font-weight: 800;
  color: var(--cake-foreground);
  margin-bottom: 0.25rem;
}

.login-subtitle {
  font-size: 0.9rem;
  color: var(--cake-muted-foreground);
  font-weight: 600;
}

.login-form {
  display: flex;
  flex-direction: column;
  gap: 1.25rem;
}

.form-group {
  display: flex;
  flex-direction: column;
  gap: 0.4rem;
}

.form-group label {
  font-size: 0.85rem;
  font-weight: 700;
  color: var(--cake-foreground);
}

.input-wrapper {
  position: relative;
  display: flex;
  align-items: center;
}

.input-icon {
  position: absolute;
  left: 1rem;
  color: var(--cake-muted-foreground);
}

input {
  width: 100%;
  padding: 0.875rem 1rem 0.875rem 2.75rem;
  border-radius: 12px;
  border: 2px solid var(--cake-border);
  background-color: #ffffff;
  font-family: 'Nunito', sans-serif;
  font-size: 0.95rem;
  color: var(--cake-foreground);
  transition: all 0.2s ease;
  outline: none;
}

input:focus {
  border-color: var(--cake-primary);
  background-color: #ffffff;
  box-shadow: 0 0 0 4px var(--cake-input);
}

input::placeholder {
  color: #d8a0c0;
}

.forgot-password {
  display: flex;
  justify-content: flex-end;
  margin-top: 0.25rem;
}

.forgot-password a {
  font-size: 0.8rem;
  color: var(--cake-primary);
  text-decoration: none;
  font-weight: 700;
  transition: color 0.2s;
}

.forgot-password a:hover {
  color: var(--cake-primary-hover);
  text-decoration: underline;
}

.btn-login {
  display: flex;
  justify-content: center;
  align-items: center;
  gap: 0.5rem;
  width: 100%;
  padding: 0.875rem;
  background-color: var(--cake-primary);
  color: var(--cake-primary-foreground);
  border: none;
  border-radius: 12px;
  font-family: 'Nunito', sans-serif;
  font-size: 1rem;
  font-weight: 700;
  cursor: pointer;
  transition: all 0.2s ease;
  margin-top: 0.5rem;
  box-shadow: 0 4px 12px rgba(232, 79, 160, 0.2);
}

.btn-login:hover:not(:disabled) {
  background-color: var(--cake-primary-hover);
  transform: translateY(-2px);
  box-shadow: 0 6px 16px rgba(232, 79, 160, 0.3);
}

.btn-login:disabled {
  opacity: 0.7;
  cursor: not-allowed;
}

.error-message {
  padding: 0.75rem;
  background-color: #fef2f2;
  border: 1px solid #fecaca;
  color: #ef4444;
  border-radius: 8px;
  font-size: 0.85rem;
  font-weight: 600;
  text-align: center;
}

/* Animations */
.animate-fade-in {
  animation: fadeIn 0.5s ease-out;
}

@keyframes fadeIn {
  from { opacity: 0; transform: translateY(10px); }
  to { opacity: 1; transform: translateY(0); }
}

.animate-spin {
  animation: spin 1s linear infinite;
}

@keyframes spin {
  from { transform: rotate(0deg); }
  to { transform: rotate(360deg); }
}
</style>
