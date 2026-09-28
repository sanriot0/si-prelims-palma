<template>
    <v-container class="d-flex flex-column justify-center align-center" style="min-height: 100vh;">
      <v-card width="400" class="pa-6 rounded-xl" >
        <v-card-title class="text-center text-h5">Login</v-card-title>
        <v-text-field
              label="Email address"
              type="email" 
              variant="solo" 
        ></v-text-field>
         <v-text-field
              hint="Enter your password to access this website"
              label="Password"
              type="password"
              variant="solo"
    ></v-text-field>
    <v-card-text>

      <v-btn
      block 
      color="green"
      size="large"
      @click="">
        Sign in 
      </v-btn>
       <v-divider opacity=".7" thickness="3" gradient vertical>OR</v-divider>
      <v-btn
        block
        color="primary"
        size="large"
        prepend-icon="mdi-google"
        @click="loginWithGoogle"
      >
        Sign in with Google
      </v-btn>
    </v-card-text>
      </v-card>
    </v-container>
</template>


<script setup lang="ts">

  definePageMeta({
    layout: false
  })
  
      // @ts-nocheck
    const config = useRuntimeConfig()
      declare global {
      interface Window {
      google: any
 }
}

    const loginWithGoogle = () => {
    const client = window.google.accounts.oauth2.initTokenClient({
    client_id: config.public.googleClientId,
    scope: 'openid email profile',
    callback: async (response: any) => {
    const userInfo = await $fetch(
    'https://www.googleapis.com/oauth2/v3/userinfo',
 {
    headers: {
      Authorization: `Bearer ${response.access_token}`
    }
  }
 )
    localStorage.setItem(
      'google_user',
        JSON.stringify(userInfo)
      )
    localStorage.setItem(
      'google_token',
        response.access_token
      )
        navigateTo('/')
 }
 })
      client.requestAccessToken()
}

  
</script>
