<template>
  <v-app>
    <v-navigation-drawer
      app
      permanent
      color="white"
    >
      <AppSidebar />
    </v-navigation-drawer>

    <v-main class="main-background">
      <v-container fluid>
        <Nuxt />
      </v-container>
    </v-main>
  </v-app>
</template>

<script>
import AppSidebar from '~/components/AppSidebar.vue'

export default {
  components: { AppSidebar }
}
</script>

<style scoped>
.main-background {
  background-color: #ffffff;
}
.layout {
  display: flex;
  height: 100vh;
}
.content {
  flex: 1;
  overflow-y: auto;
  padding: 1rem;
}
</style>
