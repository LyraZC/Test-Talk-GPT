<template>
  <q-page class="flex flex-center column q-pa-md">
    <!-- 1. Reconocimiento de voz -->
    <q-btn color="primary" @click="startListening" label="🎤 Grabar Voz" class="q-mb-md"/>
    <q-btn color="negative" @click="stopListening" label="✋ Detener" class="q-mb-md"/>
    <div class="q-mb-lg">
      <q-banner class="bg-grey-2 text-dark">
        <div><strong>Texto reconocido:</strong></div>
        <div>{{ recognizedText }}</div>
      </q-banner>
    </div>

    <!-- 2. Reproducir audio -->
    <q-btn color="secondary" @click="playAudio" label="▶️ Reproducir Audio" class="q-mb-lg"/>

    <!-- 3. Archivos -->
    <q-btn color="accent" @click="saveFile" label="💾 Guardar Archivo" class="q-mb-md"/>
    <q-btn color="info" @click="readFile" label="📂 Leer Archivo"/>
    <div v-if="fileContent" class="q-mt-md">
      <q-banner class="bg-blue-1 text-dark">
        <div><strong>Contenido del archivo:</strong></div>
        <div>{{ fileContent }}</div>
      </q-banner>
    </div>
  </q-page>
</template>

<script lang="ts" setup>
import { ref } from 'vue'
import { SpeechRecognition } from '@capacitor-community/speech-recognition'
import { Filesystem, Directory, Encoding } from '@capacitor/filesystem'

// 🎤 Texto reconocido por voz
const recognizedText = ref('')
let listening = false

async function startListening () {
  try {
    const available = await SpeechRecognition.available()
    if (!available) {
      alert('El reconocimiento de voz no está disponible en este dispositivo.')
      return
    }

    await SpeechRecognition.requestPermissions()

    await SpeechRecognition.start({
      language: 'es-ES',
      maxResults: 1,
      prompt: 'Habla ahora...'
    })

    await SpeechRecognition.addListener('partialResults', (data) => {
      recognizedText.value = data.matches[0] || ''
    })

    listening = true
  } catch (err) {
    console.error('Error al iniciar reconocimiento de voz:', err)
  }
}

async function stopListening () {
  if (listening) {
    await SpeechRecognition.stop()
    listening = false
  }
}

// ▶️ Reproducir un audio simple (usando HTML5 audio)
async function playAudio () {
  const audio = new Audio('/sounds/example.mp3') // coloca un archivo en "public/sounds/"
  await audio.play()
}

// 📂 Manejo de archivos
const fileContent = ref('')

async function saveFile () {
  try {
    await Filesystem.writeFile({
      path: 'demo.txt',
      data: 'Hola, este es un archivo guardado desde Quasar 🚀',
      directory: Directory.Documents,
      encoding: Encoding.UTF8
    })
    alert('Archivo guardado correctamente ✅')
  } catch (err) {
    console.error('Error al guardar archivo:', err)
  }
}

async function readFile () {
  try {
    const result = await Filesystem.readFile({
      path: 'demo.txt',
      directory: Directory.Documents,
      encoding: Encoding.UTF8
    })
    fileContent.value = result.data as string
  } catch (err) {
    console.error('Error al leer archivo:', err)
    alert('No se encontró el archivo')
  }
}
</script>
