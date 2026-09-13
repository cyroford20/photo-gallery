<template>
  <ion-page>
    <ion-header>
      <ion-toolbar>
        <ion-title>Photo-Gallery</ion-title>
      </ion-toolbar>
    </ion-header>
    <ion-content class="ion-padding">
      <CameraComponent @photo-captured="addPhoto" />
      <PhotoGalleryComponent :photos="photos" />
    </ion-content>
  </ion-page>
</template>

<script setup lang="ts">
import { onMounted, ref } from 'vue';
import { IonContent, IonHeader, IonPage, IonTitle, IonToolbar } from '@ionic/vue';
import CameraComponent from '../components/CameraComponent.vue';
import PhotoGalleryComponent from '../components/PhotoGalleryComponent.vue';

const photos = ref<string[]>([]);
const storageKey = 'photo-gallery-photos';

onMounted(() => {
  const savedPhotos = localStorage.getItem(storageKey);
  if (!savedPhotos) return;

  try {
    photos.value = JSON.parse(savedPhotos) as string[];
  } catch {
    localStorage.removeItem(storageKey);
  }
});

function addPhoto(photo: string) {
  photos.value.unshift(photo);
  localStorage.setItem(storageKey, JSON.stringify(photos.value));
}
</script>
