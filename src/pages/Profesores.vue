<template>
  <v-container>
    <h2>Profesores</h2>

    <!-- Campo para buscar por múltiples parámetros -->
    <v-row>
      <v-col>
        <v-text-field
          v-model="searchTerm"
          label="Buscar Profesor"
          @input="filterProfesores"
          clearable
        ></v-text-field>
      </v-col>
    </v-row>

    <v-data-table
      :headers="headers"
      :items="filteredProfesores"
      :loading="loading"
      class="elevation-1"
    >
      <template v-slot:no-data>
        <v-alert>No hay datos disponibles.</v-alert>
      </template>

      <!-- Fila adicional con los nombres de cada tipo de dato -->
      <template v-slot:top>
        <v-row>
          <v-col>
            <table style="width: 100%;">
              <thead>
                <tr>
                  <th>ID</th>
                  <th>Nombre</th>
                  <th>Número de Empleado</th>
                  <th style="text-align: center;">Asignatura</th> <!-- Alinear al centro -->
                  <th>Acciones</th> <!-- Nueva columna para acciones -->
                </tr>
              </thead>
            </table>
          </v-col>
        </v-row>
      </template>

      <!-- Fila para cada profesor -->
      <template v-slot:item="{ item }">
        <tr @click="selectProfesor(item)">
          <td>{{ item.id }}</td>
          <td>{{ item.nombreProfesor }}</td>
          <td>{{ item.numEmpleado }}</td>
          <td style="text-align: center;">{{ item.asignatura }}</td> <!-- Alinear al centro -->
          <td>
            <!-- Botón para eliminar profesor -->
            <v-btn icon @click.stop="eliminarProfesor(item.id)">
              <v-icon>mdi-delete</v-icon> <!-- Icono de eliminar -->
            </v-btn>
          </td>
        </tr>
      </template>

    </v-data-table>

    <!-- Sección para inscribir profesor -->
    <h3>Inscribir Profesor</h3>
    <v-form @submit.prevent="inscribirProfesor">
      <v-row>
        <v-col cols="12" md="4">
          <v-text-field v-model="nuevoProfesor.nombreProfesor" label="Nombre" required></v-text-field>
        </v-col>
        <v-col cols="12" md="4">
          <v-text-field v-model="nuevoProfesor.numEmpleado" label="Número de Empleado" required></v-text-field>
        </v-col>
        <v-col cols="12" md="4">
          <v-text-field v-model="nuevoProfesor.asignatura" label="Asignatura" required></v-text-field>
        </v-col>
      </v-row>
      <v-btn type="submit" color="primary">Inscribir Profesor</v-btn>
    </v-form>

    <!-- Sección para actualizar profesor -->
    <h3>Actualizar Profesor</h3>
    <v-form @submit.prevent="actualizarProfesor" v-if="selectedProfesor">
      <v-row>
        <v-col cols="12" md="4">
          <v-text-field v-model="selectedProfesor.nombreProfesor" label="Nombre" required></v-text-field>
        </v-col>
        <v-col cols="12" md="4">
          <v-text-field v-model="selectedProfesor.numEmpleado" label="Número de Empleado" required></v-text-field>
        </v-col>
        <v-col cols="12" md="4">
          <v-text-field v-model="selectedProfesor.asignatura" label="Asignatura" required></v-text-field>
        </v-col>
      </v-row>

      <v-btn type="submit" color="primary">Actualizar Profesor</v-btn>
    </v-form>

    <!-- Sección para actualizar campos opcionales -->
    <h3>Actualizar Campo</h3>
    <v-form @submit.prevent="actualizarCampo" v-if="selectedProfesor">
      <v-row>
        <!-- Campos opcionales para actualizar -->
        <v-col cols="12" md="4">
          <v-text-field v-model="tempProfesor.nombreProfesor" label="Nombre (opcional)"></v-text-field>
        </v-col>
        <v-col cols="12" md="4">
          <v-text-field v-model="tempProfesor.numEmpleado" label="Número de Empleado (opcional)"></v-text-field>
        </v-col>
        <v-col cols="12" md="4">
          <v-text-field v-model="tempProfesor.asignatura" label="Asignatura (opcional)"></v-text-field>
        </v-col>
      </v-row>

      <v-btn type="submit" color="primary">Actualizar Campo</v-btn>
    </v-form>


<!-- Sección para mostrar los cursos del profesor -->
<h3>Cursos del Profesor</h3>
<v-list v-if="selectedProfesor && cursosDeProfesores[selectedProfesor.id]">
  <v-list-item v-for="curso in cursosDeProfesores[selectedProfesor.id]" :key="curso.id">
    <v-list-item-content>
      <v-list-item-title>{{ curso.materia }}</v-list-item-title>
      <v-list-item-subtitle>Grupo: {{ curso.grupo }}</v-list-item-subtitle>
    </v-list-item-content>
  </v-list-item>
</v-list>
<v-alert v-else type="info">Seleccione un profesor para ver sus cursos</v-alert>

  </v-container>
</template>

<script>
import axios from 'axios';

export default {
  data() {
    return {
      profesores: [],
      filteredProfesores: [],
      cursosDeProfesores: {},
      loading: true,
      searchTerm: '', // Variable para almacenar el término de búsqueda
      headers: [
        { text: 'ID', value: 'id' },
        { text: 'Nombre', value: 'nombreProfesor' },
        { text: 'Número de Empleado', value: 'numEmpleado' },
        { text: 'Asignatura', value: 'asignatura', align: 'center' }, // Alinear al centro
        { text: 'Acciones', value: 'acciones' } // Nueva columna para acciones
      ],
      nuevoProfesor: {
        nombreProfesor: '',
        numEmpleado: '',
        asignatura: ''
      },
      selectedProfesor: null, // Para almacenar el profesor seleccionado para actualizar
      tempProfesor: {} // Objeto temporal para los campos opcionales a actualizar
    };
  },
  mounted() {
    this.fetchProfesores();
  },
  methods: {
    async fetchProfesores() {
      try {
        const response = await axios.get('https://localhost:4000/profesores'); // Ajusta la URL según tu API
        this.profesores = response.data; // Asegúrate de que la estructura de datos sea correcta
        this.filteredProfesores = this.profesores; // Inicializa la lista filtrada
      } catch (error) {
        console.error('Error al obtener los profesores:', error);
      } finally {
        this.loading = false;
      }
    },

    async fetchCursosDeProfesor(profesorId) {
    try {
      const response = await axios.get(`https://localhost:4000/profesores/${profesorId}/cursos`);
      return response.data;
    } catch (error) {
      console.error('Error al obtener los cursos del profesor:', error);
      return [];
    }
  },
    filterProfesores() {
      if (this.searchTerm) {
        this.filteredProfesores = this.profesores.filter(profesor =>
          profesor.id.toString().includes(this.searchTerm) ||
          profesor.nombreProfesor.toLowerCase().includes(this.searchTerm.toLowerCase()) ||
          profesor.numEmpleado.toString().includes(this.searchTerm) ||
          profesor.asignatura.toLowerCase().includes(this.searchTerm.toLowerCase())
        );
      } else {
        this.filteredProfesores = this.profesores; // Si no hay búsqueda, mostrar todos
      }
    },
    async inscribirProfesor() {
      try {
        const response = await axios.post('https://localhost:4000/profesores', this.nuevoProfesor);
        
       // Agregar el nuevo profesor a la lista local después de la inscripción exitosa
       this.profesores.push(response.data);
       
       // Reiniciar el formulario
       this.nuevoProfesor = { nombreProfesor: '', numEmpleado: '', asignatura: '' };
       
       // Actualizar la lista filtrada también
       this.filteredProfesores = this.profesores;
     } catch (error) {
       console.error('Error al inscribir el profesor:', error);
     }
   },
   async selectProfesor(profesor) {
  this.selectedProfesor = { ...profesor };
  this.tempProfesor = {};
  if (!this.cursosDeProfesores[profesor.id]) {
    this.cursosDeProfesores[profesor.id] = await this.fetchCursosDeProfesor(profesor.id);
  }
},
   async actualizarProfesor() {
     try {
       await axios.put(`https://localhost:4000/profesores/${this.selectedProfesor.id}`, this.selectedProfesor);
       
       // Actualizar la lista local con los nuevos datos del profesor actualizado
       const index = this.profesores.findIndex(prof => prof.id === this.selectedProfesor.id);
       if (index !== -1) {
         Object.assign(this.profesores[index], this.selectedProfesor); // Reemplazar el objeto en la lista local
         this.filteredProfesores = [...this.profesores]; // Actualizar la lista filtrada también
         this.selectedProfesor = null; // Reiniciar después de la actualización exitosa
       }
     } catch (error) {
       console.error('Error al actualizar el profesor:', error);
     }
   },
   async actualizarCampo() {
     try {
       const updatedData = {};
       
       // Solo agregar campos que no estén vacíos en tempProfesor
       if (this.tempProfesor.nombreProfesor) updatedData.nombreProfesor = this.tempProfesor.nombreProfesor;
       if (this.tempProfesor.numEmpleado) updatedData.numEmpleado = this.tempProfesor.numEmpleado;
       if (this.tempProfesor.asignatura) updatedData.asignatura = this.tempProfesor.asignatura;

       if (Object.keys(updatedData).length > 0) { // Verificar si hay campos a actualizar
         await axios.patch(`https://localhost:4000/profesores/${this.selectedProfesor.id}`, updatedData);
         
         const index = this.profesores.findIndex(prof => prof.id === this.selectedProfesor.id);
         if (index !== -1) {
           Object.assign(this.profesores[index], updatedData); // Actualizar solo los campos modificados
           this.filteredProfesores = [...this.profesores]; // Actualizar la lista filtrada también
           this.tempProfesor = {}; // Reiniciar objeto temporal después de la actualización
         }
       }
     } catch (error) {
       console.error('Error al actualizar el campo del profesor:', error);
     }
   },
   async eliminarProfesor(id) {
     try {
       await axios.delete(`https://localhost:4000/profesores/${id}`); // Eliminar el profesor en el servidor

       // Filtrar el profesor eliminado de la lista local
       this.profesores = this.profesores.filter(profesor => profesor.id !== id);
       
       // Actualizar la lista filtrada también
       this.filteredProfesores = this.profesores;
     } catch (error) {
       console.error('Error al eliminar el profesor:', error);
     }
   }
  },
 
};
</script>

<style scoped>
/* Estilo para centrar el contenido de la columna "Asignatura" */
.v-data-table td:nth-child(4),
.v-data-table th:nth-child(4) {
  text-align: center; /* Alinear texto al centro */
}
</style>
