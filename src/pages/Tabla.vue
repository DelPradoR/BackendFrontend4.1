<template>
  <v-container>
    <div class="text-center">
      <v-btn
        prepend-icon="mdi-arrow-left"
        @click="irRoot"
        class="borde"
      >
        <template v-slot:prepend>
          <v-icon color="warning"></v-icon>
        </template>
        Regresar al inicio
      </v-btn>
    </div>

    <!-- Mensaje de error si no hay conexión -->
    <v-alert v-if="error" type="error" dismissible>
      {{ error }}
    </v-alert>

    <!-- Indicador de carga lineal -->
    <v-progress-linear
      v-if="loading"
      color="yellow darken-2"
      indeterminate
    ></v-progress-linear>


    <DetallePelicula :nombre="peliculaSeleccionada" />

    <!-- Centrar la tabla -->
    <v-row justify="center">
      <v-col cols="12" md="8"> 
        <TablaPelis 
          :peliculas="peliculas" 
          @eliminarPelicula="eliminarPelicula"
          @nombre_Pelicula2="peliculaSeleccionada = $event.movie"
        />
      </v-col>
    </v-row>

  </v-container>
</template>

<script setup>

import { onMounted, ref } from "vue";
import { useRouter } from "vue-router";
import TablaPelis from "../components/tabla.vue"; 
import DetallePelicula from "../components/nombrePeli.vue"; 

const router = useRouter();
const peliculas = ref([]);
const peliculaSeleccionada = ref('');
const loading = ref(true); 
const error = ref(null); 

onMounted(async () => {
  try {
    const response = await fetch('https://dummyapi.online/api/movies');
    
    if (!response.ok) {
      throw new Error('Error en la conexión con el servidor');
    }

    const data = await response.json();
    peliculas.value = data; 
  } catch (err) {
    error.value = "No se pudo cargar las películas. " + err.message; 
  } finally {
    loading.value = false; 
  }
});

const irRoot = () => {
  router.replace("/");
};


const eliminarPelicula = (id) => {
  peliculas.value = peliculas.value.filter(pelicula => pelicula.id !== id);
};
</script>

<style scoped>
.borde {
  border: 1px solid white;
}
</style>