<template>
  <section class="panel camera-panel">
    <h1>Camera</h1>
    <ion-button expand="block" class="take-picture-button" :disabled="isTakingPicture" @click="takePicture">
      <ion-icon slot="start" :icon="cameraOutline" />
      {{ isTakingPicture ? 'OPENING CAMERA...' : 'TAKE PICTURE' }}
    </ion-button>
  </section>
</template>

<script setup lang="ts">
import { ref } from 'vue';
import { Camera, CameraResultType, CameraSource } from '@capacitor/camera';
import { cameraOutline } from 'ionicons/icons';
import { IonButton, IonIcon } from '@ionic/vue';

const emit = defineEmits<{
  photoCaptured: [dataUrl: string];
  error: [message: string];
}>();

const isTakingPicture = ref(false);

async function takePicture() {
  isTakingPicture.value = true;

  try {
    const photo = await Camera.getPhoto({
      quality: 90,
      resultType: CameraResultType.DataUrl,
      source: CameraSource.Camera,
    });

    if (photo.dataUrl) emit('photoCaptured', photo.dataUrl);
  } catch {
    emit('error', 'The camera was cancelled or could not be opened.');
  } finally {
    isTakingPicture.value = false;
  }
}
</script>

<style scoped>
.panel { background: #ffffff; border: 1px solid #eeeeee; border-radius: 2px; box-shadow: 0 2px 5px rgb(0 0 0 / 18%); padding: 18px; }
.camera-panel { margin-bottom: 18px; }
h1 { color: #222222; font-family: Arial, sans-serif; font-size: 23px; font-weight: 500; margin: 0 0 22px; }
.take-picture-button { --background: #1260d8; --background-activated: #0b4cad; --border-radius: 4px; --box-shadow: 0 2px 4px rgb(0 0 0 / 20%); font-size: 16px; font-weight: 600; height: 42px; margin: 0; }
</style>
