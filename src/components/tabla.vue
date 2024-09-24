<template>
  <div>
    <table class="table table-bordered">
      <thead>
        <tr>
          <th scope="col" class="titulos">Título</th>
          <th scope="col" class="titulos">Calificación</th>
          <th scope="col" class="titulos">IMDB</th>
          <th scope="col" class="titulos">Acciones</th>
        </tr>
      </thead>
      <tbody>
        <tr v-if="peliculas.length === 0">
          <td colspan="4">No hay películas disponibles.</td>
        </tr>
        <tr v-for="pelicula in peliculas" :key="pelicula.id">
          <td class="fondo" @click="nombre_Pelicula(pelicula)">{{ pelicula.movie }}</td>
          <td class="texto" style="cursor:default;">{{ pelicula.rating }}</td>
          <td class="fondo" style="cursor:default;">
            <a :href="pelicula.imdb_url" target="_blank">{{ pelicula.imdb_url }}</a>
          </td>
          <td class="fondo" style="cursor:default;">
            <v-container>
              <v-row align="center" justify="center">
                <v-col cols="auto">
                  <v-btn density="comfortable" icon @click.stop.prevent="activarAccion('edit', pelicula)">
                    <v-icon>mdi-pencil</v-icon>
                  </v-btn>
                </v-col>
                <v-col cols="auto">
                  <v-btn density="comfortable" icon @click.stop.prevent="activarAccion('delete', pelicula)">
                    <v-icon>mdi-delete</v-icon>
                  </v-btn>
                </v-col>
              </v-row>
            </v-container>
          </td>
        </tr>
      </tbody>
    </table>

    
    <v-dialog v-model="dialogDelete" width="auto">
      <v-card max-width="800">
        <v-card-title>Eliminando película: {{ selectedMovie?.movie }}</v-card-title>
        <v-card-text>¿Está seguro de que desea eliminar esta película?</v-card-text>
        <template v-slot:actions>
          <v-btn text @click="dialogDelete = false">Cancelar</v-btn>
          <v-btn text @click="confirmDelete">Confirmar</v-btn> 
        </template>
      </v-card>
    </v-dialog>

    
    <v-dialog v-model="dialogEdit" width="auto">
      <v-card max-width="800">
        <v-card-title>Editando película: {{ selectedMovie?.movie }}</v-card-title>
        <v-card-text>
          <v-text-field
            label="Nuevo Título"
            v-model="newMovieName"
          ></v-text-field>

          
          <v-text-field
            label="Nueva Calificación"
            v-model.number="newRating"
            type="number"
            min="0"
            max="10"
          ></v-text-field>

         
          <v-text-field
            label="Nueva URL de IMDB"
            v-model="newImdbUrl"
          ></v-text-field>

        </v-card-text>
        <template v-slot:actions>
          <v-btn text @click="dialogEdit = false">Cancelar</v-btn>
          <v-btn text @click="confirmEdit">Confirmar</v-btn> 
        </template>
      </v-card>
    </v-dialog>

  </div>

</template>

<script setup>
import { defineProps, defineEmits, ref } from 'vue';

const dialogEdit = ref(false);
const dialogDelete = ref(false);
const selectedMovie = ref(null);
const newMovieName = ref(''); 
const newRating = ref(null); 
const newImdbUrl = ref(''); 

const props = defineProps({
  peliculas: {
    type: Array,
    required: true,
  },
});

const emits = defineEmits(['nombre_Pelicula2', 'eliminarPelicula']); 

function nombre_Pelicula(pelicula) {
  const payload = {
    movie: pelicula.movie,
  };
  emits('nombre_Pelicula2', payload); 
}

const activarAccion = (action, pelicula) => {
  selectedMovie.value = pelicula; 
  newMovieName.value = pelicula.movie; 
  newRating.value = pelicula.rating; 
  newImdbUrl.value = pelicula.imdb_url; 
  
  if (action === 'edit') {
    dialogEdit.value = true; 
  } else if (action === 'delete') {
    dialogDelete.value = true; 
  }
};

const confirmEdit = () => {
  console.log(`Editando la película: ${selectedMovie.value.movie}`);
  

  selectedMovie.value.movie = newMovieName.value;
  selectedMovie.value.rating = newRating.value;
  selectedMovie.value.imdb_url = newImdbUrl.value;

  emits('nombre_Pelicula2', { movie: selectedMovie.value.movie }); 
  
  dialogEdit.value = false; 
};

const confirmDelete = () => {
  console.log(`Eliminando la película: ${selectedMovie.value.movie}`);
  
  
  emits('eliminarPelicula', selectedMovie.value.id); 
  
  dialogDelete.value = false; 
};
</script>

<style scoped>
table {
  cursor: pointer; 
}

.titulos {
  background-color: rgb(40, 132, 219);
  text-align: center;
  color: rgb(0, 0, 0);
}

.texto {
  text-align: center;
  background-color: rgb(255, 255, 255);
  color: rgb(0, 0, 0);
}

.fondo {
  background-color: rgb(255, 255, 255);
  color: rgb(0, 0, 0);
}

.v-icon {
cursor: pointer; 
}
</style>