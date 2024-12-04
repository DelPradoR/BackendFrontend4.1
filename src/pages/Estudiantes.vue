<template>
  <v-container>
    <h2>Estudiantes</h2>
    <!-- Campo para buscar por múltiples parámetros -->
    <v-row>
      <v-col>
        <v-text-field v-model="searchTerm" label="Buscar Estudiante" @input="filterEstudiantes" clearable></v-text-field>
      </v-col>
    </v-row>

    <v-data-table :headers="headers" :items="filteredEstudiantes" :loading="loading" class="elevation-1">
      <template v-slot:no-data>
        <v-alert>No hay datos disponibles.</v-alert>
      </template>
      <template v-slot:item="{ item }">
        <tr @click="selectEstudiante(item)">
          <td>{{ item.id }}</td>
          <td>{{ item.nombre }}</td>
          <td>{{ item.matricula }}</td>
          <td>{{ item.semestreIngreso }}</td>
          <td>{{ item.creditosCursados }}</td>
          <td>
            <v-btn icon @click.stop="eliminarEstudiante(item.id)">
              <v-icon>mdi-delete</v-icon>
            </v-btn>
          </td>
        </tr>
      </template>
    </v-data-table>
    

    <!-- Sección para inscribir estudiante -->
    <h3>Inscribir Estudiante</h3>
    <v-form @submit.prevent="inscribirEstudiante">
      <v-row>
        <v-col cols="12" md="4">
          <v-text-field v-model="nuevoEstudiante.nombre" label="Nombre" required></v-text-field>
        </v-col>
        <v-col cols="12" md="4">
          <v-text-field v-model="nuevoEstudiante.matricula" label="Matrícula" required></v-text-field>
        </v-col>
        <v-col cols="12" md="4">
          <v-text-field v-model="nuevoEstudiante.semestreIngreso" label="Semestre de Ingreso" required></v-text-field>
        </v-col>
        <v-col cols="12" md="4">
          <v-text-field v-model="nuevoEstudiante.creditosCursados" label="Créditos" required></v-text-field>
        </v-col>
      </v-row>
      <v-btn type="submit" color="primary">Inscribir Estudiante</v-btn>
    </v-form>

    <!-- Sección para actualizar estudiante -->
    <h3>Actualizar Estudiante</h3>
    <v-form @submit.prevent="actualizarEstudiante" v-if="selectedEstudiante">
      <v-row>
        <v-col cols="12" md="4">
          <v-text-field v-model="selectedEstudiante.nombre" label="Nombre" required></v-text-field>
        </v-col>
        <v-col cols="12" md="4">
          <v-text-field v-model="selectedEstudiante.matricula" label="Matrícula" required></v-text-field>
        </v-col>
        <v-col cols="12" md="4">
          <v-text-field v-model="selectedEstudiante.semestreIngreso" label="Semestre de Ingreso" required></v-text-field>
        </v-col>
        <v-col cols="12" md="4">
          <v-text-field v-model="selectedEstudiante.creditosCursados" label="Créditos" required></v-text-field>
        </v-col>
      </v-row>
      <v-btn type="submit" color="primary">Actualizar Estudiante</v-btn>
    </v-form>

    <!-- Sección para actualizar campo -->
    <h3>Actualizar Campo</h3>
    <v-form @submit.prevent="actualizarCampo" v-if="selectedEstudiante">
      <v-row>
        <!-- Campos opcionales para actualizar -->
        <v-col cols="12" md="4">
          <v-text-field v-model="tempEstudiante.nombre" label="Nombre (opcional)"></v-text-field>
        </v-col>
        <v-col cols="12" md="4">
          <v-text-field v-model="tempEstudiante.matricula" label="Matrícula (opcional)"></v-text-field>
        </v-col>
        <v-col cols="12" md="4">
          <v-text-field v-model="tempEstudiante.semestreIngreso" label="Semestre de Ingreso (opcional)"></v-text-field>
        </v-col>
        <v-col cols="12" md="4">
          <v-text-field v-model="tempEstudiante.creditosCursados" label="Créditos (opcional)"></v-text-field>
        </v-col>
      </v-row>
      <v-btn type="submit" color="primary">Actualizar Campo</v-btn>
    </v-form>

    <!-- Sección para mostrar los cursos del estudiante -->
    <h3>Cursos del Estudiante</h3>
    <v-data-table
      :headers="[
        { text: 'ID', value: 'id' },
        { text: 'Nombre', value: 'nombre' },
        { text: 'Materia', value: 'materia' }
      ]"
      :items="cursosDelEstudiante"
      class="elevation-1"
    >
      <template v-slot:no-data>
        <v-alert type="info">Este estudiante no tiene cursos asignados.</v-alert>
      </template>
      <template v-slot:item="{ item }">
        <tr>
          <td>{{ item.id }}</td>
          <td>{{ item.nombre }}</td>
          <td>{{ item.materia }}</td>
        </tr>
      </template>
    </v-data-table>
  </v-container>


  <div v-if="selectedEstudiante">
    <h3>Profesores del Estudiante</h3>
    <ul v-if="selectedEstudiante.profesores && selectedEstudiante.profesores.length">
      <li v-for="profesor in selectedEstudiante.profesores" :key="profesor.id">
        {{ profesor.nombre }}
      </li>
    </ul>
    <p v-else>Este estudiante no tiene profesores asignados.</p>
  </div>
</template>

<script>
import axios from 'axios';

export default {
  data() {
    return {
      estudiantes: [],
      filteredEstudiantes: [],
      loading: true,
      searchTerm: '',
      headers: [
        { text: 'ID', value: 'id' },
        { text: 'Nombre', value: 'nombre' },
        { text: 'Matrícula', value: 'matricula' },
        { text: 'Semestre de Ingreso', value: 'semestreIngreso' },
        { text: 'Créditos', value: 'creditosCursados' },
        { text: 'Acciones', value: 'acciones' }
      ],
      nuevoEstudiante: {
        nombre: '',
        matricula: '',
        semestreIngreso: '',
        creditosCursados: ''
      },
      selectedEstudiante: null,
      tempEstudiante: {},
      cursosDelEstudiante: [],
      nuevoCursoId: null,
      cursosDisponibles: []
    };
  },
  mounted() {
    this.fetchEstudiantes();
    this.fetchCursosDisponibles();
  },
  methods: {
    async fetchEstudiantes() {
      try {
        const response = await axios.get('https://localhost:4000/Estudiantes');
        this.estudiantes = response.data;
        this.filteredEstudiantes = this.estudiantes;
      } catch (error) {
        console.error('Error al obtener los estudiantes:', error);
      } finally {
        this.loading = false;
      }
    },
    async fetchCursosDisponibles() {
      try {
        const response = await axios.get('https://localhost:4000/Cursos');
        this.cursosDisponibles = response.data;
      } catch (error) {
        console.error('Error al obtener los cursos disponibles:', error);
      }
    },
    filterEstudiantes() {
      if (this.searchTerm) {
        this.filteredEstudiantes = this.estudiantes.filter(estudiante =>
          estudiante.id.toString().includes(this.searchTerm) ||
          estudiante.nombre.toLowerCase().includes(this.searchTerm.toLowerCase()) ||
          estudiante.matricula.toString().includes(this.searchTerm) ||
          estudiante.semestreIngreso.toString().includes(this.searchTerm) ||
          estudiante.creditosCursados.toString().includes(this.searchTerm)
        );
      } else {
        this.filteredEstudiantes = this.estudiantes;
      }
    },
    async inscribirEstudiante() {
      try {
        const response = await axios.post('https://localhost:4000/Estudiantes', this.nuevoEstudiante);
        this.estudiantes.push(response.data);
        this.nuevoEstudiante = { nombre: '', matricula: '', semestreIngreso: '', creditosCursados: '' };
        this.filteredEstudiantes = this.estudiantes;
      } catch (error) {
        console.error('Error al inscribir el estudiante:', error);
      }
    },
    async eliminarEstudiante(id) {
      try {
        await axios.delete(`https://localhost:4000/Estudiantes/${id}`);
        this.estudiantes = this.estudiantes.filter(est => est.id !== id);
        this.filteredEstudiantes = this.estudiantes;
      } catch (error) {
        console.error('Error al eliminar el estudiante:', error);
      }
    },
    async consultarProfesoresEstudiante(estudianteId) {
  try {
    const response = await axios.get(`https://localhost:4000/estudiantes/${estudianteId}/profesores`);
    return response.data.body;
  } catch (error) {
    console.error('Error al consultar profesores del estudiante:', error);
    return [];
  }
},
    selectEstudiante(estudiante) {
      this.selectedEstudiante = { ...estudiante };
      this.tempEstudiante = {};
      this.obtenerCursosEstudiante(estudiante.id);
      this.consultarProfesoresEstudiante(estudiante.id);
    },

    async actualizarEstudiante() {
      try {
        await axios.put(`https://localhost:4000/Estudiantes/${this.selectedEstudiante.id}`, this.selectedEstudiante);
        const index = this.estudiantes.findIndex(est => est.id === this.selectedEstudiante.id);
        if (index !== -1) {
          Object.assign(this.estudiantes[index], this.selectedEstudiante);
          this.filteredEstudiantes = [...this.estudiantes];
          this.selectedEstudiante = null;
        }
      } catch (error) {
        console.error('Error al actualizar el estudiante:', error);
      }
    },
    async actualizarCampo() {
      try {
        const updatedData = {};
        if (this.tempEstudiante.nombre) updatedData.nombre = this.tempEstudiante.nombre;
        if (this.tempEstudiante.matricula) updatedData.matricula = this.tempEstudiante.matricula;
        if (this.tempEstudiante.semestreIngreso) updatedData.semestreIngreso = this.tempEstudiante.semestreIngreso;
        if (this.tempEstudiante.creditosCursados) updatedData.creditosCursados = this.tempEstudiante.creditosCursados;
        if (Object.keys(updatedData).length > 0) {
          await axios.patch(`https://localhost:4000/Estudiantes/${this.selectedEstudiante.id}`, updatedData);
          const index = this.estudiantes.findIndex(est => est.id === this.selectedEstudiante.id);
          if (index !== -1) {
            Object.assign(this.estudiantes[index], updatedData);
            this.filteredEstudiantes = [...this.estudiantes];
            this.tempEstudiante = {};
          }
        }
      } catch (error) {
        console.error('Error al actualizar el campo del estudiante:', error);
      }
    },
    async obtenerCursosEstudiante(estudianteId) {
      try {
        const response = await axios.get(`https://localhost:4000/estudiantes/${estudianteId}/cursos`);
        this.cursosDelEstudiante = response.data.cursos;
      } catch (error) {
        console.error('Error al obtener los cursos del estudiante:', error);
      }
    },
  }
};
</script>

<style scoped>
.v-data-table td:nth-child(4),
.v-data-table th:nth-child(4),
.v-data-table td:nth-child(5),
.v-data-table th:nth-child(5) {
  text-align: center;
}
</style>