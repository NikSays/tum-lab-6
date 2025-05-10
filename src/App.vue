<script setup>
import { ref, onMounted } from 'vue'

const images = ref([])

onMounted(() => {
  const saved = JSON.parse(localStorage.getItem('photos') || '[]')
  images.value = saved
})

function handleFiles(files) {
  [...files].forEach(file => {
    const reader = new FileReader()
    reader.onload = e => {
      images.value.push(e.target.result)
      localStorage.setItem('photos', JSON.stringify(images.value))
    }
    reader.readAsDataURL(file)
  })
}

function onDrop(e) {
  e.preventDefault()
  if (e.dataTransfer.files && e.dataTransfer.files.length) {
    handleFiles(e.dataTransfer.files)
  }
}

function onChange(e) {
  if (e.target.files && e.target.files.length) {
    handleFiles(e.target.files)
    e.target.value = ''
  }
}
</script>

<template>
  <div class="min-h-screen bg-gray-100 flex flex-col items-center p-6">
    <h1 class="text-3xl font-semibold mb-6">SSTV Photo Uploader</h1>

    <label
      @dragover.prevent
      @drop="onDrop"
      class="w-full max-w-xl h-48 flex flex-col items-center justify-center border-4 border-dashed border-gray-400 rounded-2xl bg-white cursor-pointer hover:bg-gray-50 transition">
      <span class="text-gray-600">Drag & Drop photos here or click to select</span>
      <input type="file" accept="image/*" multiple class="hidden" @change="onChange">
    </label>

    <div v-if="images.length" class="grid grid-cols-2 sm:grid-cols-3 md:grid-cols-4 gap-4 mt-8 w-full max-w-4xl " >
      <div v-for="(src, idx) in images" :key="idx" class="relative group">
        <img :src="src" alt="uploaded" class="object-cover w-full h-40 rounded-xl shadow">
      </div>
    </div>
  </div>
</template>

<style scoped>
</style>
