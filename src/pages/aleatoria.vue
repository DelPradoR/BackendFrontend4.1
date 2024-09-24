<template>
    <div>
      <h1 class="centered-title">Galería de Imágenes</h1>

  
      
      <v-alert v-if="loading && !error" type="info" dismissible>
        Cargando imágenes...
      </v-alert>
  
     
      <v-alert v-if="!loading && error" type="error" dismissible>
        {{ error }}
      </v-alert>
  
      
      <div class="gallery" v-if="!loading && !error">
        <img 
          :src="`https://picsum.photos/300/400?random=${currentImage}`" 
          alt="Imagen aleatoria" 
          @error="handleImageError"  
        />
      </div>
  
      <!-- Controles para navegar entre imágenes -->
      <div class="controls" v-if="!loading && !error">
        <button @click="prevImage" :disabled="currentImage <= 1">Anterior</button>
        <button @click="nextImage">Siguiente</button>
      </div>
    </div>
  </template>
  
  <script setup>
  import { ref, onMounted } from 'vue';
  
  const totalImages = 20; // Número total de imágenes disponibles
  const currentImage = ref(1); 
  const loading = ref(true); // Estado de carga
  const error = ref(null); // Mensaje de error
  
  onMounted(() => {
    checkConnection(); 
    window.addEventListener('online', checkConnection); 
    window.addEventListener('offline', checkConnection); 
  });
  
  
  const checkConnection = () => {
    if (!navigator.onLine) {
      error.value = "No hay conexión a Internet. Verifica tu conexión.";
      loading.value = false; a
    } else {
      loadImages(); 
    }
  };
  
  
  const loadImages = () => {
    setTimeout(() => {
      loading.value = false; 
    }, 2000); 
  };
  
 
  const handleImageError = () => {
    error.value = "Error al cargar la imagen."; 
  };
  
  
  const nextImage = () => {
    if (currentImage.value < totalImages) {
      currentImage.value++;
    } else {
      currentImage.value = 1; 
    }
  };
  
  const prevImage = () => {
    if (currentImage.value > 1) {
      currentImage.value--;
    } else {
      currentImage.value = totalImages;
    }
  };
  </script>
  
  <style scoped>
  .centered-title {
    text-align: center; 
  }
  
  .gallery {
    display: flex;
    flex-wrap: wrap;
    justify-content: center;
  }
  
  .gallery img {
    width: 300px;
    height: auto; 
  }
  
  .controls {
    text-align: center;
    margin-top: 20px;
  }
  
  .controls button {
    margin: 0 20px; 
  }
  </style>