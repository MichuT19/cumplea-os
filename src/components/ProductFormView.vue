<script setup>
import { ref } from 'vue'
import { ImagePlus, Loader2, CheckCircle2, AlertCircle } from 'lucide-vue-next'
import { supabase } from '../supabase.js'

const form = ref({
  name: '',
  description: '',
  price: '',
  stock: '',
  category: 'Clásicos'
})

const categories = ['Clásicos', 'Frutales', 'Especiales', 'Sin Gluten']

const isSubmitting = ref(false)
const status = ref({ type: '', message: '' })

const resetForm = () => {
  form.value = { name: '', description: '', price: '', stock: '', category: 'Clásicos' }
}

const selectCategory = (cat) => {
  form.value.category = cat
}

const handleSubmit = async () => {
  isSubmitting.value = true
  status.value = { type: '', message: '' }

  try {
    if (!form.value.name || !form.value.price) {
      throw new Error('El nombre y precio son obligatorios.')
    }

    const { error } = await supabase
      .from('products')
      .insert([
        { 
          name: form.value.name, 
          description: form.value.description, 
          price: parseFloat(form.value.price), 
          stock: parseInt(form.value.stock) || 0,
          category: form.value.category
        }
      ])
      .select()

    if (error) throw error

    status.value = { type: 'success', message: '¡Pastel registrado con éxito!' }
    resetForm()
  } catch (error) {
    console.error(error)
    status.value = { 
      type: 'error', 
      message: error.message || 'Hubo un problema al registrar el pastel.' 
    }
  } finally {
    isSubmitting.value = false
    if (status.value.type === 'success') {
      setTimeout(() => {
        status.value = { type: '', message: '' }
      }, 5000)
    }
  }
}
</script>

<template>
  <div class="cake-view-container animate-fade-in">
    <!-- Header -->
    <div class="header-section">
      <h1 class="title">Registrar nuevo pastel</h1>
      <p class="subtitle">Completa la información para agregar un pastel al catálogo</p>
    </div>

    <!-- Alert -->
    <transition name="slide-fade">
      <div v-if="status.message" :class="['alert', `alert-${status.type}`]">
        <CheckCircle2 v-if="status.type === 'success'" :size="20" />
        <AlertCircle v-else :size="20" />
        <span>{{ status.message }}</span>
      </div>
    </transition>

    <form @submit.prevent="handleSubmit" class="main-content-layout">
      <!-- Left Column: Form Fields -->
      <div class="form-column">
        
        <div class="form-group">
          <label for="name">Nombre del pastel</label>
          <input 
            type="text" 
            id="name" 
            v-model="form.name" 
            placeholder="Ej. Pastel de Fresa con Crema" 
            required 
          />
        </div>

        <div class="form-group">
          <label for="description">Descripción</label>
          <textarea 
            id="description" 
            v-model="form.description" 
            placeholder="Describe los ingredientes, sabores y presentación del pastel..."
          ></textarea>
        </div>

        <div class="form-group">
          <label>Categoría</label>
          <div class="categories-wrapper">
            <button 
              v-for="cat in categories" 
              :key="cat"
              type="button"
              class="category-btn"
              :class="{ active: form.category === cat }"
              @click="selectCategory(cat)"
            >
              {{ cat }}
            </button>
          </div>
        </div>
        
        <div class="actions-row">
          <button type="submit" class="btn-save" :disabled="isSubmitting">
            <Loader2 v-if="isSubmitting" class="animate-spin" :size="20" />
            <span>{{ isSubmitting ? 'Guardando...' : 'Guardar pastel' }}</span>
          </button>
          <button type="button" class="btn-cancel" @click="resetForm" :disabled="isSubmitting">Cancelar</button>
        </div>
      </div>

      <!-- Right Column: Image Dropzone -->
      <div class="image-column">
        <div class="form-group">
          <label>Foto del pastel</label>
          <div class="image-dropzone">
            <div class="icon-circle">
              <ImagePlus :size="26" />
            </div>
            <p>Arrastra una foto aquí</p>
            <button type="button" class="btn-select-file">Seleccionar archivo</button>
          </div>
        </div>
      </div>
    </form>
  </div>
</template>

<style scoped>
@import url('https://fonts.googleapis.com/css2?family=Nunito:wght@400;600;700&display=swap');

.cake-view-container {
  /* Variables extraídas de tu diseño HTML */
  --cake-foreground: #3a1a2e;
  --cake-border: #f0d4e4;
  --cake-input: #fde8f2;
  --cake-primary: #e84fa0;
  --cake-primary-foreground: #ffffff;
  --cake-card-bg: #ffffff;
  --cake-muted: #f7d6ea;
  --cake-muted-foreground: #b07090;

  font-family: 'Nunito', sans-serif;
  color: var(--cake-foreground);
  padding: 1rem 2rem;
  max-width: 1100px;
}

.header-section {
  margin-bottom: 2.5rem;
}

.title {
  font-size: 2rem; /* text-3xl approx */
  font-weight: 700;
  margin-bottom: 0.25rem;
}

.subtitle {
  font-size: 0.875rem; /* text-sm */
  color: var(--cake-muted-foreground);
}

.main-content-layout {
  display: flex;
  gap: 2.5rem; /* gap-10 */
  flex-wrap: wrap;
}

.form-column {
  flex: 1;
  min-width: 300px;
  display: flex;
  flex-direction: column;
  gap: 1.5rem; /* space-y-6 */
}

.image-column {
  width: 320px; /* w-80 approx */
}

.form-group {
  display: flex;
  flex-direction: column;
}

.form-group label {
  font-size: 0.875rem; /* text-sm */
  font-weight: 600;
  margin-bottom: 0.5rem;
}

input, textarea {
  border: 2px solid var(--cake-border);
  border-radius: 0.5rem; /* rounded-lg */
  padding: 0.75rem 1rem; /* px-4 py-3 */
  background-color: var(--cake-input);
  color: var(--cake-foreground);
  font-size: 0.875rem;
  font-family: 'Nunito', sans-serif;
  transition: border-color 0.2s ease, box-shadow 0.2s ease;
}

input::placeholder, textarea::placeholder {
  color: var(--cake-muted-foreground);
}

input:focus, textarea:focus {
  outline: none;
  border-color: var(--cake-primary);
  background-color: #fff;
  box-shadow: 0 0 0 3px rgba(232, 79, 160, 0.1);
}

textarea {
  min-height: 96px;
  resize: vertical;
}

.price-stock-row {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 1.5rem;
}

.categories-wrapper {
  display: flex;
  flex-wrap: wrap;
  gap: 0.5rem;
}

.category-btn {
  padding: 0.5rem 1.25rem; /* px-5 py-2 */
  border-radius: 9999px; /* rounded-full */
  font-weight: 500;
  font-size: 0.875rem;
  border: 2px solid var(--cake-muted);
  background-color: var(--cake-card-bg);
  color: var(--cake-muted-foreground);
  cursor: pointer;
  transition: all 0.2s ease;
}

.category-btn:hover {
  border-color: var(--cake-primary);
  color: var(--cake-primary);
}

.category-btn.active {
  background-color: var(--cake-primary);
  border-color: var(--cake-primary);
  color: var(--cake-primary-foreground);
}

.actions-row {
  display: flex;
  gap: 1rem;
  padding-top: 1rem;
  margin-top: 0.5rem;
}

.btn-save {
  padding: 0.75rem 2rem;
  border-radius: 9999px;
  background-color: var(--cake-primary);
  color: var(--cake-primary-foreground);
  font-weight: 600;
  font-size: 0.875rem;
  border: none;
  cursor: pointer;
  transition: transform 0.2s ease, box-shadow 0.2s ease;
  display: inline-flex;
  align-items: center;
  gap: 0.5rem;
}

.btn-save:hover:not(:disabled) {
  transform: translateY(-2px);
  box-shadow: 0 4px 12px rgba(232, 79, 160, 0.3);
}

.btn-save:disabled {
  opacity: 0.7;
  cursor: not-allowed;
}

.btn-cancel {
  padding: 0.75rem 2rem;
  border-radius: 9999px;
  background-color: var(--cake-card-bg);
  border: 2px solid var(--cake-muted);
  color: var(--cake-muted-foreground);
  font-weight: 600;
  font-size: 0.875rem;
  cursor: pointer;
  transition: all 0.2s ease;
}

.btn-cancel:hover:not(:disabled) {
  background-color: var(--cake-muted);
  color: var(--cake-foreground);
}

.image-dropzone {
  border: 2px dashed var(--cake-border);
  border-radius: 1rem; /* rounded-2xl */
  background-color: var(--cake-input);
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  gap: 0.75rem;
  padding: 2.5rem 1rem; /* py-10 */
  transition: border-color 0.2s ease;
}

.image-dropzone:hover {
  border-color: var(--cake-primary);
}

.icon-circle {
  width: 3.5rem; /* w-14 */
  height: 3.5rem; /* h-14 */
  border-radius: 50%;
  background-color: var(--cake-card-bg);
  display: flex;
  align-items: center;
  justify-content: center;
  color: var(--cake-muted-foreground);
  box-shadow: 0 2px 8px rgba(0,0,0,0.05);
}

.image-dropzone p {
  font-size: 0.875rem;
  color: var(--cake-muted-foreground);
  margin: 0;
}

.btn-select-file {
  padding: 0.5rem 1.25rem;
  border-radius: 9999px;
  background-color: var(--cake-card-bg);
  border: 2px solid var(--cake-muted);
  color: var(--cake-muted-foreground);
  font-size: 0.75rem; /* text-xs */
  font-weight: 600;
  cursor: pointer;
  transition: all 0.2s ease;
}

.btn-select-file:hover {
  border-color: var(--cake-primary);
  color: var(--cake-primary);
}

.animate-spin {
  animation: spin 1s linear infinite;
}

@keyframes spin {
  from { transform: rotate(0deg); }
  to { transform: rotate(360deg); }
}

.alert {
  display: flex;
  align-items: center;
  gap: 0.75rem;
  padding: 1rem;
  border-radius: 0.5rem;
  margin-bottom: 1.5rem;
  font-weight: 600;
  font-size: 0.875rem;
}

.alert-success {
  background-color: rgba(34, 197, 94, 0.1);
  color: #16a34a;
  border: 1px solid rgba(34, 197, 94, 0.2);
}

.alert-error {
  background-color: rgba(239, 68, 68, 0.1);
  color: #dc2626;
  border: 1px solid rgba(239, 68, 68, 0.2);
}

.slide-fade-enter-active,
.slide-fade-leave-active {
  transition: all 0.3s ease;
}

.slide-fade-enter-from,
.slide-fade-leave-to {
  opacity: 0;
  transform: translateY(-10px);
}
</style>
