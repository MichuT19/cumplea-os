<script setup>
import { ref } from 'vue'

const categories = ['Todos', 'Clásicos', 'Frutales', 'Especiales', 'Sin Gluten']
const activeCategory = ref('Todos')

const cakes = [
  { name: 'Pastel de Fresa', bgClass: 'bg-accent-mint', image: 'https://storage.googleapis.com/banani-generated-images/generated-images/74ad1de8-4cd5-4709-8d37-d43e1570e792.jpg' },
  { name: 'Pastel de Chocolate', bgClass: 'bg-accent-yellow', image: 'https://storage.googleapis.com/banani-generated-images/generated-images/f09a7f80-8906-4938-a934-bff8adcd7f50.jpg' },
  { name: 'Pastel de Vainilla', bgClass: 'bg-accent-lavender', image: 'https://storage.googleapis.com/banani-generated-images/generated-images/dd03ffb8-559c-441f-b776-2b81ace4f7ef.jpg' },
  { name: 'Red Velvet', bgClass: 'bg-accent-peach', image: 'https://storage.googleapis.com/banani-generated-images/generated-images/d43c7a3f-07ff-43ae-870d-95aa34cd2da3.jpg' },
  { name: 'Pastel de Limón', bgClass: 'bg-secondary', image: 'https://storage.googleapis.com/banani-generated-images/generated-images/e6ea8821-79c4-412b-9e4c-b2a2e85e80d1.jpg' },
  { name: 'Pastel de Zanahoria', bgClass: 'bg-accent-mint', image: 'https://storage.googleapis.com/banani-generated-images/generated-images/72a40a7f-3b01-4047-b6c1-3da7eb0b7793.jpg' },
  { name: 'Pastel de Oreo', bgClass: 'bg-accent-yellow', image: 'https://storage.googleapis.com/banani-generated-images/generated-images/ece390bf-6151-471c-b99e-20abf4b1a538.jpg' },
  { name: 'Pastel de Mango', bgClass: 'bg-accent-lavender', image: 'https://storage.googleapis.com/banani-generated-images/generated-images/adf472ce-c9f2-4f1f-a38b-855c09e54ca0.jpg' }
]
</script>

<template>
  <div class="min-h-full bg-background font-body" style="min-height: 1000px">
    <div class="flex flex-col items-center pt-16 pb-12 px-16">
      <div class="flex items-center gap-4 mb-3">
        <span class="text-5xl">🎂</span>
        <h1 class="font-headings font-bold text-5xl text-foreground tracking-tight" style="letter-spacing: -0.5px">
          Catálogo de Pasteles
        </h1>
        <span class="text-5xl">🎂</span>
      </div>
      <p class="text-muted-foreground text-base font-body mt-2 font-medium">
        Hechos con amor, uno por uno
      </p>
      <div class="mt-6 h-1.5 w-32 rounded-full" style="background: linear-gradient(90deg, #e84fa0 0%, #d6b8f5 50%, #ffe07a 100%);"></div>
    </div>
    
    <div class="flex gap-3 justify-center pb-10 px-16">
      <button
        v-for="cat in categories"
        :key="cat"
        @click="activeCategory = cat"
        :class="['px-6 py-2.5 rounded-full font-body font-semibold text-sm border-2 transition-all cursor-pointer', 
                 activeCategory === cat ? 'bg-primary text-primary-foreground border-primary' : 'bg-card-bg text-muted-foreground border-muted']"
      >
        {{ cat }}
      </button>
    </div>
    
    <div class="px-16 pb-20 grid grid-cols-4 gap-8">
      <div
        v-for="cake in cakes"
        :key="cake.name"
        class="bg-card-bg rounded-2xl overflow-hidden flex flex-col transition-all hover:-translate-y-1 hover:shadow-lg"
        style="box-shadow: 0 8px 32px 0 rgba(232, 79, 160, 0.12); min-height: 280px; cursor: pointer;"
      >
        <div
          :class="[cake.bgClass, 'w-full flex items-center justify-center flex-shrink-0']"
          style="height: 200px; position: relative; overflow: hidden"
        >
          <div style="position: absolute; inset: 0; background: linear-gradient(135deg, rgba(255, 255, 255, 0.1) 0%, transparent 100%);"></div>
          <img
            :src="cake.image"
            :alt="cake.name"
            style="aspect-ratio: 4/3; object-fit: cover"
            class="w-full h-full object-cover"
          />
        </div>
        <div class="px-5 py-5 flex items-center justify-center flex-grow">
          <span class="font-headings font-semibold text-base text-foreground text-center leading-tight">
            {{ cake.name }}
          </span>
        </div>
      </div>
    </div>
    
    <div class="flex items-center justify-center pb-12">
      <span class="text-muted-foreground text-sm font-body font-medium">
        ¿No ves lo que buscas? ¡Pregunta por nuestros pasteles personalizados! 🌸
      </span>
    </div>
  </div>
</template>

<style scoped>
/* Custom extra hover classes to make it feel more interactive */
.hover\:-translate-y-1:hover {
  transform: translateY(-4px);
}
.hover\:shadow-lg:hover {
  box-shadow: 0 12px 40px 0 rgba(232, 79, 160, 0.2) !important;
}
</style>
