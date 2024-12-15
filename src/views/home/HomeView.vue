<script lang="ts" setup>
import { ref, onUnmounted, nextTick } from 'vue';
import { Html5Qrcode } from 'html5-qrcode';

const cameraStarted = ref(false);
const qrCodeResult = ref<string | null>(null);
let html5QrCode: Html5Qrcode | null = null;

const scannerId = 'qr-code-scanner';

const startScanner = async () => {
  qrCodeResult.value = null;
  cameraStarted.value = true;

  await nextTick();

  html5QrCode = new Html5Qrcode(scannerId);

  try {
    const devices = await Html5Qrcode.getCameras();
    if (devices && devices.length) {
      const cameraId = devices[0].id;

      await html5QrCode.start(
        cameraId,
        { fps: 10, qrbox: { width: 250, height: 250 } },
        (decodedText) => {
          qrCodeResult.value = decodedText;
          stopScanner();
        },
        (error) => {
          console.error("Erro de leitura:", error);
        }
      );
    }
  } catch (error) {
    console.error("Erro ao acessar a câmera:", error);
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

onUnmounted(() => {
  stopScanner();
});
</script>

<template>
  <div>
    <h1>Home</h1>

    <button v-if="!cameraStarted" @click="startScanner">
      Abrir Câmera e Ler QR Code
    </button>

    <div v-if="cameraStarted" id="qr-code-scanner" style="width: 100%; height: auto;"></div>

    <div v-if="qrCodeResult">
      <h2>Resultado do QR Code:</h2>
      <p>{{ qrCodeResult }}</p>
    </div>
  </div>
</template>


<style scoped>
#qr-code-scanner {
  width: 300px;
  height: 300px;
  border: 2px solid #000;
  margin: auto;
}
</style>
