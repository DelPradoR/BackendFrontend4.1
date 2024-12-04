<template>
  <v-container>
    <h2>Cursos</h2>
    
    <!-- Campo para buscar por múltiples parámetros -->
    <v-row>
      <v-col>
        <v-text-field v-model="searchTerm" label="Buscar Curso" @input="filterCursos" clearable></v-text-field>
      </v-col>
    </v-row>

    <v-data-table
      :headers="headers"
      :items="filteredCursos"
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
                  <th>Materia</th>
                  <th>Clave de Materia</th>
                  <th style="text-align: center;">Grupo</th>
                  <th>Créditos</th>
                  <th>Acciones</th>
                </tr>
              </thead>
            </table>
          </v-col>
        </v-row>
      </template>
      <!-- Fila para cada curso -->
      <template v-slot:item="{ item }">
        <tr @click="selectCurso(item)">
          <td>{{ item.id }}</td>
          <td>{{ item.materia }}</td>
          <td>{{ item.claveDMateria }}</td>
          <td style="text-align: center;">{{ item.grupo }}</td>
          <td>{{ item.creditos }}</td>
          <td>
            <!-- Botón para eliminar curso -->
            <v-btn icon @click.stop="eliminarCurso(item.id)">
              <v-icon>mdi-delete</v-icon>
            </v-btn>
          </td>
        </tr>
      </template>
    </v-data-table>

    <!-- Sección para inscribir curso -->
    <h3>Inscribir Curso</h3>
    <v-form @submit.prevent="inscribirCurso">
      <v-row>
        <v-col cols="12" md="4">
          <v-text-field v-model="nuevoCurso.materia" label="Materia" required></v-text-field>
        </v-col>
        <v-col cols="12" md="4">
          <v-text-field v-model="nuevoCurso.claveDMateria" label="Clave de Materia" required></v-text-field>
        </v-col>
        <v-col cols="12" md="4">
          <v-text-field v-model="nuevoCurso.grupo" label="Grupo" required></v-text-field>
        </v-col>
        <v-col cols="12" md="4">
          <v-text-field v-model="nuevoCurso.creditos" label="Créditos" type="number" required></v-text-field>
        </v-col>
      </v-row>
      <v-btn type="submit" color="primary">Inscribir Curso</v-btn>
    </v-form>

    <!-- Sección para actualizar curso -->
    <h3>Actualizar Curso</h3>
    <v-form @submit.prevent="actualizarCurso" v-if="selectedCurso">
      <v-row>
        <v-col cols="12" md="4">
          <v-text-field v-model="selectedCurso.materia" label="Materia" required></v-text-field>
        </v-col>
        <v-col cols="12" md="4">
          <v-text-field v-model="selectedCurso.claveDMateria" label="Clave de Materia" required></v-text-field>
        </v-col>
        <v-col cols="12" md="4">
          <v-text-field v-model="selectedCurso.grupo" label="Grupo" required></v-text-field>
        </v-col>
        <v-col cols="12" md="4">
          <v-text-field v-model="selectedCurso.creditos" label="Créditos" type="number" required></v-text-field>
        </v-col>
      </v-row>
      <v-btn type="submit" color="primary">Actualizar Curso</v-btn>
    </v-form>

    <h3>Actualizar Campo</h3>
    <v-form @submit.prevent="actualizarCampo" v-if="selectedCurso">
      <v-row>
        <!-- Campos opcionales para actualizar -->
        <v-col cols="12" md="4">
          <v-text-field v-model="tempCurso.materia" label="Materia (opcional)"></v-text-field>
        </v-col>
        <v-col cols="12" md="4">
          <v-text-field v-model="tempCurso.claveDMateria" label="Clave de Materia (opcional)"></v-text-field>
        </v-col>
        <v-col cols="12" md="4">
          <v-text-field v-model="tempCurso.grupo" label="Grupo (opcional)"></v-text-field>
        </v-col>
        <v-col cols="12" md="4">
          <v-text-field v-model="tempCurso.creditos" label="Créditos (opcional)" type="number"></v-text-field>
        </v-col>
      </v-row>
      <v-btn type="submit" color="primary">Actualizar Campo</v-btn>
    </v-form>
  </v-container>

<!-- Formulario para inscribir alumno al curso -->
<h3>Inscribir Alumno al Curso</h3>
<v-form @submit.prevent="inscribirAlumno">
  <v-row>
    <v-col cols="12" md="6">
      <v-text-field
        v-model="inscripcionAlumno.cursoId"
        label="ID del Curso"
        type="number"
        required
      ></v-text-field>
    </v-col>
    <v-col cols="12" md="6">
      <v-text-field
        v-model="inscripcionAlumno.estudianteId"
        label="ID del Estudiante"
        type="number"
        required
      ></v-text-field>
    </v-col>
  </v-row>
  <v-btn type="submit" color="primary">Inscribir Alumno</v-btn>
</v-form>

<!-- Formulario para inscribir profesor al curso -->
<h3>Inscribir Profesor al Curso</h3>
<v-form @submit.prevent="inscribirProfesor">
  <v-row>
    <v-col cols="12" md="6">
      <v-text-field
        v-model="inscripcionProfesor.cursoId"
        label="ID del Curso"
        type="number"
        required
      ></v-text-field>
    </v-col>
    <v-col cols="12" md="6">
      <v-text-field
        v-model="inscripcionProfesor.profesorId"
        label="ID del Profesor"
        type="number"
        required
      ></v-text-field>
    </v-col>
  </v-row>
  <v-btn type="submit" color="primary">Inscribir Profesor</v-btn>
</v-form>
<div v-if="selectedCurso">
      <h3>Estudiantes Inscritos</h3>
      <ul v-if="selectedCurso.estudiantes && selectedCurso.estudiantes.length">
        <li v-for="estudiante in selectedCurso.estudiantes" :key="estudiante.id">
          {{ estudiante.nombre }}
        </li>
      </ul>
      <p v-else>No hay estudiantes inscritos en este curso.</p>

      <h3>Profesor Asignado</h3>
      <p v-if="selectedCurso.profesor">{{ selectedCurso.profesor.nombre }}</p>
      <p v-else>No hay profesor asignado a este curso.</p>
    </div>
</template>

<script>
import axios from 'axios';

export default {
  data() {
    return {
      cursos: [],
      filteredCursos: [],
      loading: true,
      searchTerm: '',
      headers: [
        { text: 'ID', value: 'id' },
        { text: 'Materia', value: 'materia' },
        { text: 'Clave de Materia', value: 'claveDMateria' },
        { text: 'Grupo', value: 'grupo' },
        { text: 'Créditos', value: 'creditos' }
      ],
      nuevoCurso: {
        materia: '',
        claveDMateria: '',
        grupo: '',
        creditos: ''
      },
      inscripcionAlumno: {
      cursoId: null,
      estudianteId: null
    },
    inscripcionProfesor: {
      cursoId: null,
      profesorId: null
    },
      selectedCurso: null,
      tempCurso: {}
      
    };
  },
  mounted() {
    this.fetchCursos();
  },
  methods: {
    async fetchCursos() {
      try {
        const response = await axios.get('https://localhost:4000/cursos');
        const data = response.data?.body || response.data;
        if (Array.isArray(data)) {
          this.cursos = data;
          this.filteredCursos = [...data];
        } else {
          console.error('Los datos no son un arreglo:', data);
          this.cursos = [];
          this.filteredCursos = [];
        }
      } catch (error) {
        console.error('Error al obtener los cursos:', error);
        this.cursos = [];
        this.filteredCursos = [];
      } finally {
        this.loading = false;
      }
    },
    async inscribirAlumno() {
    try {
      await axios.post('https://localhost:4000/cursos/inscribir-alumno', this.inscripcionAlumno);
      this.inscripcionAlumno = { cursoId: null, estudianteId: null };
      alert('Alumno inscrito exitosamente');
    } catch (error) {
      console.error('Error al inscribir alumno:', error);
      alert('Error al inscribir alumno');
    }
  },
  filterCursos() {
  if (this.searchTerm) {
    const searchLower = this.searchTerm.toLowerCase();
    this.filteredCursos = this.cursos.filter(curso => {
      return (
        (curso.id?.toString().toLowerCase() || '').includes(searchLower) ||
        (curso.materia?.toLowerCase() || '').includes(searchLower) ||
        (typeof curso.claveDMateria === 'number' ? curso.claveDMateria.toString().includes(this.searchTerm) : (curso.claveDMateria?.toLowerCase() || '').includes(searchLower)) ||
        (curso.grupo?.toString().toLowerCase() || '').includes(searchLower) ||
        (typeof curso.creditos === 'number' ? curso.creditos.toString().includes(this.searchTerm) : (curso.creditos?.toString().toLowerCase() || '').includes(searchLower))
      );
    });
  } else {
    this.filteredCursos = this.cursos;
  }
},
    async inscribirProfesor() {
    try {
      await axios.post('https://localhost:4000/cursos/inscribir-profesor', this.inscripcionProfesor);
      this.inscripcionProfesor = { cursoId: null, profesorId: null };
      alert('Profesor inscrito exitosamente');
    } catch (error) {
      console.error('Error al inscribir profesor:', error);
      alert('Error al inscribir profesor');
    }
  },
  async consultarEstudiantesInscritos(cursoId) {
    try {
      const response = await axios.get(`https://localhost:4000/cursos/${cursoId}/estudiantes`);
      return response.data.body; // Asegúrate de que el servidor devuelva los estudiantes en esta propiedad
    } catch (error) {
      console.error('Error al consultar estudiantes inscritos:', error);
      return [];
    }
  },

  async consultarProfesorAsignado(cursoId) {
    try {
      const response = await axios.get(`https://localhost:4000/cursos/${cursoId}/profesor`);
      return response.data.body; // Asegúrate de que el servidor devuelva el profesor en esta propiedad
    } catch (error) {
      console.error('Error al consultar profesor asignado:', error);
      return null;
    }
  },
  async selectCurso(curso) {
    this.selectedCurso = { ...curso };
    this.tempCurso = {};
    try {
      const estudiantesResponse = await axios.get(`https://localhost:4000/cursos/${curso.id}/estudiantes`);
      const profesorResponse = await axios.get(`https://localhost:4000/cursos/${curso.id}/profesor`);
      this.selectedCurso.estudiantes = estudiantesResponse.data.body;
      this.selectedCurso.profesor = profesorResponse.data.body;
    } catch (error) {
      console.error('Error al obtener detalles del curso:', error);
    }
  },
    async inscribirCurso() {
      try {
        const response = await axios.post('https://localhost:4000/cursos', this.nuevoCurso);
        this.cursos.push(response.data);
        this.nuevoCurso = {
          materia: '',
          claveDMateria: '',
          grupo: '',
          creditos: ''
        };
        this.filteredCursos = [...this.cursos];
      } catch (error) {
        console.error('Error al inscribir el curso:', error);
      }
    },
    
    async actualizarCurso() {
      try {
        await axios.put(`https://localhost:4000/cursos/${this.selectedCurso.id}`, this.selectedCurso);
        const index = this.cursos.findIndex(cur => cur.id === this.selectedCurso.id);
        if (index !== -1) {
          Object.assign(this.cursos[index], this.selectedCurso);
          this.filteredCursos = [...this.cursos];
          this.selectedCurso = null;
        }
      } catch (error) {
        console.error('Error al actualizar el curso:', error);
      }
    },
    async actualizarCampo() {
      try {
        const updatedData = {};
        if (this.tempCurso.materia) updatedData.materia = this.tempCurso.materia;
        if (this.tempCurso.claveDMateria) updatedData.claveDMateria = this.tempCurso.claveDMateria;
        if (this.tempCurso.grupo) updatedData.grupo = this.tempCurso.grupo;
        if (this.tempCurso.creditos) updatedData.creditos = this.tempCurso.creditos;

        if (Object.keys(updatedData).length > 0) {
          await axios.patch(`https://localhost:4000/cursos/${this.selectedCurso.id}`, updatedData);
          const index = this.cursos.findIndex(cur => cur.id === this.selectedCurso.id);
          if (index !== -1) {
            Object.assign(this.cursos[index], updatedData);
            this.filteredCursos = [...this.cursos];
            this.tempCurso = {};
          }
        }
      } catch (error) {
        console.error('Error al actualizar el campo del curso:', error);
      }
    },
    async eliminarCurso(id) {
      try {
        await axios.delete(`https://localhost:4000/cursos/${id}`);
        this.cursos = this.cursos.filter(curso => curso.id !== id);
        this.filteredCursos = [...this.cursos];
      } catch (error) {
        console.error('Error al eliminar el curso:', error);
      }
    }
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