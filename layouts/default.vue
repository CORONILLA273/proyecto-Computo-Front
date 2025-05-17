<template>
  <v-app>
    <v-layout>
      <AppSidebar />
      <v-main class="main-content">
        <v-container fluid class="pa-4">
          <Nuxt />
        </v-container>
      </v-main>
    </v-layout>
  </v-app>
</template>

<script>
import AppSidebar from '~/components/AppSidebar.vue'

export default {
  components: { AppSidebar }
}
</script>

<style scoped>
.main-content {
  margin-left: 250px; /* Igual al ancho del sidebar */
  min-height: 100vh;
  background-color: #ffffff;
}
</style>
