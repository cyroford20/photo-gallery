<template>
  <ion-page>
    <ion-header>
      <ion-toolbar><ion-title>My Photo Gallery</ion-title></ion-toolbar>
    </ion-header>
    <ion-content>
      <main class="page-content">
        <CameraComponent @photo-captured="addPhoto" @error="errorMessage = $event" />
        <PhotoGalleryComponent
          :photos="photos"
          :error-message="errorMessage"
          @select="selectedPhoto = $event"
        />
      </main>
      <ion-modal :is-open="Boolean(selectedPhoto)" @didDismiss="selectedPhoto = null">
        <ion-header>
          <ion-toolbar>
            <ion-title>Photo</ion-title>
            <ion-buttons slot="end"><ion-button @click="selectedPhoto = null">Close</ion-button></ion-buttons>
          </ion-toolbar>
        </ion-header>
        <ion-content v-if="selectedPhoto" class="photo-preview">
          <img :src="selectedPhoto.url" alt="Selected captured photo" />
        </ion-content>
      </ion-modal>
    </ion-content>
  </ion-page>
</template>

<script setup lang="ts">
import { onMounted, onUnmounted, ref } from 'vue';
import { addDoc, collection, onSnapshot, orderBy, query, serverTimestamp } from 'firebase/firestore';
import { getDownloadURL, ref as storageRef, uploadString } from 'firebase/storage';
import { IonButton, IonButtons, IonContent, IonHeader, IonModal, IonPage, IonTitle, IonToolbar } from '@ionic/vue';
import CameraComponent from '../components/CameraComponent.vue';
import PhotoGalleryComponent, { type GalleryPhoto } from '../components/PhotoGalleryComponent.vue';
import { db, storage } from '../firebase';

const photos = ref<GalleryPhoto[]>([]);
const selectedPhoto = ref<GalleryPhoto | null>(null);
const errorMessage = ref('');
let stopListening: (() => void) | undefined;

onMounted(() => {
  const photosQuery = query(collection(db, 'photos'), orderBy('createdAt', 'desc'));
  stopListening = onSnapshot(photosQuery, (snapshot) => {
    photos.value = snapshot.docs.map((photo) => ({
      id: photo.id,
      url: photo.data().imageUrl as string,
    }));
    errorMessage.value = '';
  }, () => {
    errorMessage.value = 'Connect Firebase Firestore to load your photos.';
  });
});

onUnmounted(() => stopListening?.());

async function addPhoto(dataUrl: string) {
  try {
    const fileName = `photos/${Date.now()}.jpg`;
    const imageRef = storageRef(storage, fileName);
    await uploadString(imageRef, dataUrl, 'data_url', { contentType: 'image/jpeg' });
    const imageUrl = await getDownloadURL(imageRef);
    await addDoc(collection(db, 'photos'), { imageUrl, fileName, createdAt: serverTimestamp() });
    errorMessage.value = '';
  } catch {
    errorMessage.value = 'The photo could not be saved. Check Firebase rules and try again.';
  }
}
</script>

<style scoped>
:global(body) { --ion-background-color: #ffffff; --ion-text-color: #222222; }
.page-content { padding: 28px 32px 70px; }
.photo-preview { --background: #111111; text-align: center; }
.photo-preview img { height: auto; max-height: 90vh; max-width: 100%; object-fit: contain; }
@media (min-width: 600px) { .page-content { margin: 0 auto; max-width: 520px; } }
</style>
