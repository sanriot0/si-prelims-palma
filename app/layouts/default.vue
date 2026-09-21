<template>
   <v-app>
    
      <v-navigation-drawer
        v-model="drawer" theme="light"
      >
        <v-list>
          <v-list-item
            prepend-avatar="https://randomuser.me/api/portraits/women/85.jpg"
            subtitle="binchi_6767@gmailcom"
            title="bin chiling"
          ></v-list-item>
        </v-list>

        <v-divider></v-divider>

        <v-list density="compact" nav>
          <v-list-item prepend-icon="mdi-folder" title="My Files" value="myfiles"></v-list-item>
          <v-list-item prepend-icon="mdi mdi-qrcode" title="QR Scanner" value="shared" to="/qr-scanner"></v-list-item>
          <v-list-item prepend-icon="mdi mdi-weather-cloudy" title="Weather App" value="starred"></v-list-item>
        </v-list>
      </v-navigation-drawer>

      <v-app-bar>
        <template v-slot:prepend>
          <v-app-bar-nav-icon @click="drawer = !drawer"></v-app-bar-nav-icon>
        </template>
          <v-app-bar-title>Application Bar</v-app-bar-title>
      </v-app-bar>

      <v-main>
        <v-container fluid>
            <slot />
        </v-container>
      </v-main>
</v-app>
   
  
</template>

<script setup>
  const drawer = ref(true)
</script>

