<template>
      <v-container class="d-flex flex-column justify-center align-center" style="min-height: 100vh;">
        <v-card v-if="user" width="400" class="pa-6 rounded-xl">
      <div class="d-flex align-center ga-4">
      <v-avatar size="64">
        <v-img :src="user.picture" />
      </v-avatar>
      <div>
        <h2>{{ user.name }}</h2>
        <p>{{ user.email }}</p>

        <v-btn color="error" @click="logout">Logout</v-btn>

      </div>
      </div>

          </v-card>
            </v-container>
</template>


    <script setup lang="ts">
      const user = ref<any>(null)

        onMounted(() => {
      const savedUser = localStorage.getItem('google_user')
        if (savedUser) {
          user.value = JSON.parse(savedUser)
 }
})

      const logout = () => {
        localStorage.removeItem('google_user')
        localStorage.removeItem('google_token')
        navigateTo('/login')
}

 

</script>