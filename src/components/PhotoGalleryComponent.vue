<template>
  <section class="panel gallery-panel">
    <h2>Photo Gallery</h2>
    <div v-if="photos.length" class="photo-grid">
      <div v-for="(photo, index) in photos" :key="index" class="photo-item">
        <img :src="photo" alt="Captured photo" />
      </div>
    </div>
    <p v-else class="empty-message">Your captured photos will appear here.</p>
  </section>
</template>

<script setup lang="ts">
defineProps<{
  photos: string[];
}>();
</script>

<style scoped>
.panel { background: #ffffff; border: 1px solid #eeeeee; border-radius: 2px; box-shadow: 0 2px 5px rgb(0 0 0 / 18%); padding: 18px; }
.gallery-panel { min-height: 310px; }
h2 { color: #222222; font-family: Arial, sans-serif; font-size: 23px; font-weight: 500; margin: 0 0 38px; }
.photo-grid { display: grid; gap: 14px; grid-template-columns: repeat(2, minmax(0, 1fr)); padding: 0 22px; }
.photo-item { aspect-ratio: 3 / 4; background: #f0f0f0; border: 0; border-radius: 4px; cursor: pointer; overflow: hidden; padding: 0; }
.photo-item img { display: block; height: 100%; object-fit: cover; width: 100%; }
.empty-message { color: #777777; font-size: 14px; margin: 0; text-align: center; }
</style>
