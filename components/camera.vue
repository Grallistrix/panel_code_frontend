<template>
  <div class="camera-stream">
    <h2>Podgląd kamery</h2>
    <img :src="streamUrl" alt="Stream z kamery" />
  </div>
</template>

<script lang="ts" setup>
import { ref } from 'vue'

const raspberryIp = '192.168.196.220' // <-- Zmień na swój IP
const streamUrl = ref(`http://${raspberryIp}:8080/`)
</script>

<style scoped>
.camera-stream {
  display: flex;
  flex-direction: column;
  align-items: center;
}

.camera-stream img {
  width: 640px;
  height: 480px;
  border: 2px solid #333;
  background: #000;
}
</style>
