<script setup>
import { ref, onMounted } from 'vue'

const images = ref([])

onMounted(() => {
  images.value = JSON.parse(localStorage.getItem('photos') || '[]')
})

function persist() {
  localStorage.setItem('photos', JSON.stringify(images.value))
}

function handleFiles(files) {
  [...files].forEach(file => {
    const reader = new FileReader()
    reader.onload = e => {
      images.value.push(e.target.result)
      persist()
    }
    reader.readAsDataURL(file)
  })
}

function onDrop(e) {
  e.preventDefault()
  if (e.dataTransfer.files?.length) handleFiles(e.dataTransfer.files)
}

function onChange(e) {
  if (e.target.files?.length) handleFiles(e.target.files)
  e.target.value = ''
}

function remove(idx) {
  images.value.splice(idx, 1)
  persist()
}

</script>

<template>
  <div class="min-h-screen bg-gray-100 flex flex-col items-center p-6">
    <h1 class="text-3xl font-semibold mb-6">SSTV Photo Uploader</h1>

    <!-- Upload -->
    <label
      @dragover.prevent
      @drop="onDrop"
      class="w-full max-w-xl h-48 flex flex-col items-center justify-center border-4 border-dashed border-gray-400 rounded-2xl bg-white cursor-pointer hover:bg-gray-50 transition">
      <span class="text-gray-600">Drag & Drop photos here or click to select</span>
      <input type="file" accept="image/*" multiple class="hidden" @change="onChange">
    </label>

    <!-- Gallery -->
    <div v-if="images.length" class="grid grid-cols-2 sm:grid-cols-3 md:grid-cols-4 gap-4 mt-8 w-full max-w-4xl">
      <div v-for="(src, idx) in images" :key="idx" class="relative group">
        <img :src="src" alt="uploaded" class="object-cover w-full h-40 rounded-xl shadow">
        <!-- Delete button -->
        <button
          @click="remove(idx)"
          class="absolute top-2 right-2 bg-white/80 backdrop-blur-sm rounded-full p-1 opacity-0 group-hover:opacity-100 transition">
        ×
        </button>
      </div>
    </div>
  </div>
</template>

