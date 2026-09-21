<template>
  <NuxtLayout>
    <v-app>
      <NuxtPage />
    </v-app>
  </NuxtLayout>
</template>

<script setup lang="ts">
  useHead({
    script: [
      {
        hid: 'google-gsi',
        src: 'https://accounts.google.com/gsi/client',
        async: true,
      }
    ]
  })
</script>

