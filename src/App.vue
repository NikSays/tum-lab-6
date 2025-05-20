<script setup>
import { ref, onMounted } from 'vue'
import {MartinMOne, PD50} from './assets/sstv.js' 

const images = ref([])
const currentOsc = ref(null)
const mode = ref('Martin M1')
const dark = ref(false)

// Themes

function toggleTheme () {
  dark.value = !dark.value
  document.documentElement.classList.toggle('dark', dark.value)
}

// File handling

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

// Playback 

function getEncoder(selected) {
  switch (selected) {
    case 'Martin M1': return new MartinMOne()
    case 'PD50': return new PD50()
  }
}

async function imgToDataArray(src) {
  // decode img
  const img = new Image();
  img.src = src;
  await img.decode();
  // parse to data array
  const width = 320, height = 256; 
  const cvs = new OffscreenCanvas(width, height);
  const g = cvs.getContext('2d');
  g.drawImage(img,0,0,width,height);
  return g.getImageData(0,0,width,height).data;
}

async function playSSTV(src){
  // prepare 
  const imageData = await imgToDataArray(src);
  const encoder = getEncoder(mode.value);
  encoder.prepareImage(imageData);
  // play
  const ctx = new window.AudioContext();
  const osc = ctx.createOscillator();
  osc.type='sine';
  osc.connect(ctx.destination);
  const start = ctx.currentTime+0.1;
  const end = encoder.encodeSSTV(osc,start);
  osc.start(start);
  osc.stop(end);
  // expose global state
  currentOsc.value=osc;
  osc.onended=()=>currentOsc.value=null;
}

function stopPlayback(){
  if(currentOsc.value){
    currentOsc.value.stop(); 
    currentOsc.value=null;
  }
}

</script>

<template>
  <div class="min-h-screen bg-gray-100 flex flex-col items-center p-6 dark:bg-gray-900 transition-colors">
    <button
      @click="toggleTheme"
      class="self-end mb-4 text-2xl select-none">
      {{ dark ? '🌞' : '🌙' }}
    </button>

    <h1 class="text-3xl font-semibold mb-6 text-gray-900 dark:text-gray-100">
      SSTV Photo Uploader
    </h1>

    <!-- SSTV Intro -->
        <div class="mb-6 max-w-2xl text-center text-sm text-gray-700 dark:text-gray-300 flex flex-col gap-1">
          <p>
            <a href="https://en.wikipedia.org/wiki/Slow-scan_television" class="underline hover:text-blue-500">Slow-scan television (SSTV)</a>
            is a picture transmission method, used by amateur radio operators, <br> to transmit and receive static pictures via radio.
          </p>
          <div class="flex justify-around">
            <a href="https://play.google.com/store/apps/details?id=xdsopl.robot36&hl=en" class="underline hover:text-blue-500">
              Android Decoder
            </a> 
            <a href="https://apps.apple.com/us/app/sstv-slow-scan-tv/id387910013" class="underline hover:text-blue-500">
              iOS Decoder (paid)
            </a>
          </div>
        </div>

    <!-- Upload -->
    <label @dragover.prevent @drop="onDrop"
      class="w-xl h-48 flex flex-col items-center justify-center
             border-4 border-dashed border-gray-400 rounded-2xl cursor-pointer
             bg-white hover:bg-gray-50 transition">
      <span class="text-gray-600">Drag & Drop photos here or click to select</span>
      <input type="file" accept="image/*" multiple class="hidden" @change="onChange">
    </label>

    <!-- Encoder -->
    <select v-model="mode" class="m-5 p-2 rounded-lg border bg-white shadow">
      <option>Martin M1</option>
      <option>PD50</option>
    </select>

    <!-- Gallery -->
    <div v-if="images.length" class="grid grid-cols-3 md:grid-cols-4 gap-4 mt-8 w-full">
      <div v-for="(src, idx) in images" :key="idx" class="relative group">
        <img :src="src" class="object-cover w-full h-40 rounded-xl shadow cursor-pointer" @click="playSSTV(src)">
        <!-- Delete button -->
        <button @click="remove(idx)"
          class="absolute top-2 right-2 bg-white rounded-full p-1 opacity-0 group-hover:opacity-100 transition">
          ×
        </button>
      </div>
    </div>

    <!-- Player -->
    <div v-if="currentOsc" 
      @click="stopPlayback" 
      class="fixed inset-0 bg-black/50 flex items-center justify-center text-white text-xl">
        Transmitting… click to stop
    </div>
  </div>
</template>

