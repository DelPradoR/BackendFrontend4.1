<template>
     
  <v-layout>
    <v-app-bar style="background-color: rgb(112, 11, 147);" prominent>
      
      <v-app-bar-nav-icon variant="text" @click.stop="drawer = !drawer"></v-app-bar-nav-icon>

      <v-toolbar-title></v-toolbar-title>

      <v-spacer></v-spacer>

      <template v-if="$vuetify.display.mdAndUp">
      </template>

      <v-btn icon="mdi-dots-vertical" variant="text"></v-btn>
    </v-app-bar>

    <v-navigation-drawer
      v-model="drawer"
      :location="$vuetify.display.mobile ? 'bottom' : undefined"
      temporary
    >
    <v-list-item title="My Application" subtitle="Vuetify"></v-list-item>
<v-divider></v-divider>
<v-list-item to="/">Inicio</v-list-item>
       <v-list-item to="/Tabla">Tabla</v-list-item>
        <v-list-item to="/card">Mi carta</v-list-item>
        <v-list-item to="/aleatoria">Imagenes</v-list-item>
        <v-list-item to="/Estudiantes">Estudiantes </v-list-item>
        <v-list-item to="/Profesores">Profesores</v-list-item>
        <v-list-item to="/Cursos">Cursos</v-list-item>
    </v-navigation-drawer>

    <v-main>
      <router-view />
    </v-main>
  </v-layout>

</template>
<script>
export default{
data: () => ({
  drawer:false,
}),
}
</script>