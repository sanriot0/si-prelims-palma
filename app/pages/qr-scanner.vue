<template>
  <div class="mx-auto d-flex justify-center align-center" style="height: 90vh;">

    <v-card>
      <v-card-text>
        <video ref="videoRef" class="qr-video"></video>

        <v-btn block color="primary" @click="startScanner">Start Scanner</v-btn>
          
        <v-btn block class="mt-2" color="error">Stop Scanner</v-btn>


      </v-card-text>
    </v-card>

  </div>
</template>

<script lang="ts" setup>
    //@ts-nocheck
    import QrScanner from 'qr-scanner';

    const videoRef = ref<HTMLVideoElement | null>(null)
    //const result = ref('')
    let scanner: QrScanner | null = null

    const startScanner = async () => {
      if (!videoRef.value) return

      scanner = new QrScanner(
        videoRef.value,
      (scanResult) => {
        //result.value =scanResult.data
       // defineEmits('scanned' , scanResult.data)

        scanner?.stop()
      },
      {
        preferredCamera: 'environment',
        highlightScanRegion: true,
        highlightCodeOutline: true,
      }
    )

    await scanner.start()
    }
    
</script>

<style scoped>
    .qr-video{
    width: 100%;
    max-width: 400%;
    border-radius: 12px;
    background: rgb(0, 0, 0);
    }
</style>  
