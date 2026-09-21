<template>
  <div class="mx-auto d-flex justify-center align-center" style="height: 90vh;">

    <v-card>
      <v-card-text>
        <video class="qr-video"></video>

        <v-btn color="primary" class="mx-5" >Start Scanner</v-btn>
          
        <v-btn color="error">Stop Scanner</v-btn>


      </v-card-text>
    </v-card>

  </div>
</template>

<script lang="ts" setup>
  
</script>

<style scoped>
    .qr-video{
    width: 100%;
    max-width: 400%;
    border-radius: 12px;
    background: rgb(37, 37, 37);
    }
</style>  
