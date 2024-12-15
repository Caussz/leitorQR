<script lang="ts" setup>
import { ref, onUnmounted, nextTick } from 'vue';
import { Html5Qrcode } from 'html5-qrcode';

const cameraStarted = ref(false);
const qrCodeResult = ref<string | null>(null);
const cameras = ref<{ id: string; label: string }[]>([]);
const selectedCamera = ref<string | null>(null);
let html5QrCode: Html5Qrcode | null = null;

const scannerId = 'qr-code-scanner';

const fetchCameras = async () => {
  try {
    const devices = await Html5Qrcode.getCameras();
    cameras.value = devices.map((device) => ({
      id: device.id,
      label: device.label || 'Câmera desconhecida',
    }));
    if (devices.length) {
      selectedCamera.value = devices[0].id;
    }
  } catch (error) {
    console.error('Erro ao buscar câmeras:', error);
    cameras.value = [];
  }
};

const startScanner = async () => {
  if (!selectedCamera.value) {
    alert('Nenhuma câmera disponível para iniciar.');
    return;
  }

  qrCodeResult.value = null;
  cameraStarted.value = true;

  await nextTick();

  html5QrCode = new Html5Qrcode(scannerId);

  try {
    await html5QrCode.start(
      selectedCamera.value,
      { fps: 10, qrbox: { width: 250, height: 250 } },
      (decodedText) => {
        qrCodeResult.value = decodedText;
        stopScanner();
      },
      (error) => {
        console.log('Erro de leitura:', error);
      }
    );
  } catch (error) {
    console.error('Erro ao iniciar scanner:', error);
    cameraStarted.value = false;
  }
};

const stopScanner = () => {
  if (html5QrCode) {
    html5QrCode.stop().then(() => {
      html5QrCode?.clear();
      cameraStarted.value = false;
    });
  }
};

fetchCameras();

onUnmounted(() => {
  stopScanner();
});
</script>

<template>
  <div class="container">
    <h1>Leitor de QR Code</h1>

    <div v-if="!cameraStarted" class="camera-select">
      <label for="camera">Escolha a câmera:</label>
      <select id="camera" v-model="selectedCamera">
        <option v-for="camera in cameras" :key="camera.id" :value="camera.id">
          {{ camera.label }}
        </option>
      </select>
    </div>

    <div class="buttons">
      <button v-if="!cameraStarted" @click="startScanner" :disabled="!cameras.length">
        {{ cameras.length ? 'Iniciar Scanner' : 'Nenhuma câmera disponível' }}
      </button>
      <button v-if="cameraStarted" @click="stopScanner" class="stop-button">
        Parar Scanner
      </button>
    </div>

    <div v-if="cameraStarted" id="qr-code-scanner" class="scanner"></div>
    <div v-if="qrCodeResult" class="result">
      <h2>Resultado do QR Code:</h2>
      <p>{{ qrCodeResult }}</p>
    </div>
  </div>
</template>

<style scoped>
.container {
  text-align: center;
  font-family: Arial, sans-serif;
  max-width: 600px;
  margin: 0 auto;
  padding: 20px;
}

h1 {
  font-size: 24px;
  color: #333;
  margin-bottom: 20px;
}

.camera-select {
  margin-bottom: 20px;
}

.camera-select label {
  font-size: 16px;
  margin-right: 10px;
}

.camera-select select {
  padding: 5px 10px;
  font-size: 14px;
}

.buttons {
  margin: 20px 0;
}

button {
  padding: 10px 20px;
  font-size: 16px;
  border: none;
  cursor: pointer;
  background-color: #007bff;
  color: white;
  border-radius: 5px;
  transition: background-color 0.3s;
}

button:disabled {
  background-color: #ccc;
  cursor: not-allowed;
}

button.stop-button {
  background-color: #dc3545;
}

button:hover:not(:disabled) {
  background-color: #0056b3;
}

button.stop-button:hover:not(:disabled) {
  background-color: #c82333;
}

.scanner {
  width: 300px;
  height: 300px;
  margin: 20px auto;
  border: 2px solid #333;
}

.result {
  margin-top: 20px;
  padding: 10px;
  border: 1px solid #007bff;
  background-color: #f8f9fa;
  border-radius: 5px;
}

.result h2 {
  color: #333;
  font-size: 18px;
}

.result p {
  font-size: 16px;
  color: #555;
  word-break: break-word;
}
</style>
