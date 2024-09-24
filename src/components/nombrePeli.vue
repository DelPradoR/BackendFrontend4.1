<template>
    <div v-if="nombre">
      <h2>Película Seleccionada: {{ nombre }}</h2>
    </div>
    <div v-else>
      <h2>Selecciona una película para ver su nombre.</h2>
    </div>
  </template>
  
  <script>
  export default {
    name: 'DetallePelicula',
    props: {
      nombre: {
        type: String,
        default: ''
      }
    }
  }
  </script>
  
  <style scoped>
  h2 {
    text-align: center;
    color: rgb(255, 255, 255);
  }
  </style>