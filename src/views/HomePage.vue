<template>
  <ion-page>
    <ion-header>
      <ion-toolbar><ion-title>My Photo Gallery</ion-title></ion-toolbar>
    </ion-header>
    <ion-content>
      <main class="page-content">
        <section class="panel camera-panel">
          <h1>Camera</h1>
          <ion-button expand="block" class="take-picture-button" @click="takePicture">
            <ion-icon slot="start" :icon="cameraOutline" /> TAKE PICTURE
          </ion-button>
        </section>
        <section class="panel gallery-panel">
          <h2>Photo Gallery</h2>
          <div v-if="photos.length" class="photo-grid">
            <button v-for="photo in photos" :key="photo.id" class="photo-item" @click="viewPhoto(photo)">
              <img :src="photo.url" alt="Captured photo" />
            </button>
          </div>
          <p v-else class="empty-message">{{ errorMessage || 'Your captured photos will appear here.' }}</p>
        </section>
      </main>
      <ion-modal :is-open="Boolean(selectedPhoto)" @didDismiss="selectedPhoto = null">
        <ion-header><ion-toolbar><ion-title>Photo</ion-title><ion-buttons slot="end"><ion-button @click="selectedPhoto = null">Close</ion-button></ion-buttons></ion-toolbar></ion-header>
        <ion-content v-if="selectedPhoto" class="photo-preview"><img :src="selectedPhoto.url" alt="Selected captured photo" /></ion-content>
      </ion-modal>
    </ion-content>
  </ion-page>
</template>

<script setup lang="ts">
import { onMounted, onUnmounted, ref } from 'vue';
import { Camera, CameraResultType, CameraSource } from '@capacitor/camera';
import { addDoc, collection, onSnapshot, orderBy, query, serverTimestamp } from 'firebase/firestore';
import { getDownloadURL, ref as storageRef, uploadString } from 'firebase/storage';
import { cameraOutline } from 'ionicons/icons';
import { IonButton, IonButtons, IonContent, IonHeader, IonIcon, IonModal, IonPage, IonTitle, IonToolbar } from '@ionic/vue';
import { db, storage } from '../firebase';

type Photo = { id: string; url: string };
const photos = ref<Photo[]>([]);
const selectedPhoto = ref<Photo | null>(null);
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

async function takePicture() {
  try {
    const photo = await Camera.getPhoto({ quality: 90, resultType: CameraResultType.DataUrl, source: CameraSource.Camera });
    if (!photo.dataUrl) return;

    const fileName = `photos/${Date.now()}.jpg`;
    const imageRef = storageRef(storage, fileName);
    await uploadString(imageRef, photo.dataUrl, 'data_url', { contentType: 'image/jpeg' });
    const imageUrl = await getDownloadURL(imageRef);
    await addDoc(collection(db, 'photos'), { imageUrl, fileName, createdAt: serverTimestamp() });
    errorMessage.value = '';
  } catch {
    errorMessage.value = 'The photo could not be saved. Check Firebase rules and try again.';
  }
}

function viewPhoto(photo: Photo) {
  selectedPhoto.value = photo;
}
</script>

<style scoped>
:global(body) { --ion-background-color: #ffffff; --ion-text-color: #222222; }
.page-content { padding: 28px 32px 70px; }
.panel { background: #ffffff; border: 1px solid #eeeeee; border-radius: 2px; box-shadow: 0 2px 5px rgb(0 0 0 / 18%); padding: 18px; }
.camera-panel { margin-bottom: 18px; }
.panel h1, .panel h2 { color: #222222; font-family: Arial, sans-serif; font-size: 23px; font-weight: 500; margin: 0 0 22px; }
.panel h2 { margin-bottom: 38px; }
.take-picture-button { --background: #1260d8; --background-activated: #0b4cad; --border-radius: 4px; --box-shadow: 0 2px 4px rgb(0 0 0 / 20%); font-size: 16px; font-weight: 600; height: 42px; margin: 0; }
.gallery-panel { min-height: 310px; }
.photo-grid { display: grid; gap: 14px; grid-template-columns: repeat(2, minmax(0, 1fr)); padding: 0 22px; }
.photo-item { aspect-ratio: 3 / 4; background: #f0f0f0; border: 0; border-radius: 4px; cursor: pointer; overflow: hidden; padding: 0; }
.photo-item img { display: block; height: 100%; object-fit: cover; width: 100%; }
.empty-message { color: #777777; font-size: 14px; margin: 0; text-align: center; }
.photo-preview { --background: #111111; text-align: center; }.photo-preview img { height: auto; max-height: 90vh; max-width: 100%; object-fit: contain; }
@media (min-width: 600px) { .page-content { margin: 0 auto; max-width: 520px; } }
.gallery-header { --background: rgba(247, 245, 240, .9); }.gallery-header ion-toolbar { --min-height: 68px; --padding-start: 22px; --padding-end: 18px; }.gallery-header ion-title { font-family: Georgia, serif; font-size: 23px; font-weight: 600; letter-spacing: -.5px; }.gallery-header ion-button { --color: #20302d; }.profile-button { border: 1px solid #d8d4c9; border-radius: 50%; font-size: 11px; height: 34px; min-width: 34px; }
.gallery-shell { max-width: 860px; margin: 0 auto; padding: 30px 22px 100px; }.intro { padding: 10px 0 28px; }.eyebrow { color: #8d735d; font-size: 10px; font-weight: 700; letter-spacing: 2px; margin: 0 0 10px; } h1, h2 { color: #20302d; font-family: Georgia, serif; font-weight: 500; letter-spacing: -1.2px; margin: 0; } h1 { font-size: clamp(38px, 8vw, 60px); line-height: .98; } h1 em { color: #ba6d4d; font-style: italic; }.intro-copy { color: #78807a; font-size: 14px; line-height: 1.5; margin: 18px 0 0; max-width: 310px; }
.search-box { align-items: center; background: #fff; border: 1px solid #e7e3db; border-radius: 13px; box-shadow: 0 5px 18px rgba(47, 56, 48, .05); display: flex; gap: 11px; padding: 13px 15px; }.search-box ion-icon { color: #9b9e98; font-size: 19px; }.search-box input { background: none; border: 0; color: #20302d; flex: 1; font: 14px inherit; outline: 0; }.filter-icon { border-left: 1px solid #e5e1d9; padding-left: 12px; }.category-row { display: flex; gap: 9px; overflow-x: auto; padding: 20px 0 32px; scrollbar-width: none; }.category-row::-webkit-scrollbar { display: none; }.category-chip { background: transparent; border: 1px solid #ddd9d0; border-radius: 20px; color: #7c8179; cursor: pointer; font: 12px inherit; padding: 9px 16px; white-space: nowrap; }.category-chip.active { background: #263b36; border-color: #263b36; color: white; }
.section-heading { align-items: end; display: flex; justify-content: space-between; margin-bottom: 16px; }.section-heading h2 { font-size: 27px; }.section-heading > span { color: #9a9d96; font-size: 12px; }.photo-grid { display: grid; gap: 12px; grid-template-columns: repeat(2, 1fr); }.photo-card { aspect-ratio: 1 / 1.18; background: #ddd; border: 0; border-radius: 12px; cursor: pointer; overflow: hidden; padding: 0; position: relative; }.photo-card img { height: 100%; object-fit: cover; transition: transform .35s ease; width: 100%; }.photo-card:hover img { transform: scale(1.04); }.photo-overlay { align-items: end; background: linear-gradient(transparent, rgba(21, 30, 27, .78)); bottom: 0; color: white; display: flex; font-size: 12px; justify-content: space-between; left: 0; padding: 28px 12px 12px; position: absolute; right: 0; text-align: left; }.photo-overlay ion-icon { font-size: 17px; }.empty-state { color: #7c8179; padding: 40px 0; text-align: center; } ion-fab-button { --background: #ba6d4d; --background-activated: #99553b; --box-shadow: 0 8px 18px rgba(117, 68, 48, .25); }.preview-content { --background: #20302d; text-align: center; }.preview-content img { display: block; height: auto; margin: 0 auto; max-height: 78vh; max-width: 100%; object-fit: contain; }.preview-content p { color: #d7ded8; font-size: 13px; } @media (min-width: 640px) { .photo-grid { grid-template-columns: repeat(3, 1fr); } }
</style>
