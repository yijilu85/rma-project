<script setup>
import { ref } from 'vue'

defineProps({})

var dropzoneFile = ref('')
const active = ref(false)
var jsonData = ref(null)

const toggleActive = function () {
  active.value = !active.value
}

const drop = function (e) {
  toggleActive()
  readJSON(e.dataTransfer.files[0])
}

const clearInput = function (event) {
  dropzoneFile.value = ''
  jsonData = ref(null)
  emit('file-deselected')
}

const readJSON = (file) => {
  if (file) {
    dropzoneFile.value = file

    const reader = new FileReader()

    reader.onload = (e) => {
      try {
        const parsedJson = JSON.parse(e.target.result)
        jsonData.value = parsedJson
        emit('file-selected', jsonData.value)
      } catch (error) {
        console.error('Error parsing JSON:', error)
      }
    }

    reader.readAsText(file)
  }
}
const handleFileChange = (event) => {
  const file = event.target.files[0]

  readJSON(file)
}

const emit = defineEmits(['file-selected', 'file-deselected'])
</script>

<template>
  <div
    class="dropzone"
    @dragenter.prevent="toggleActive"
    @dragleave.prevent="toggleActive"
    @dragover.prevent
    @drop.prevent="drop"
    :class="{ 'active-dropzone': active }"
  >
    <span class="bold">Drag and drop JSON file</span>
    <span class="bold">OR</span>
    <label class="button-primary" for="fileInput">Select File</label>
    <input type="file" @change="handleFileChange" id="fileInput" ref="fileInput" />

    <button @click="clearInput" class="clearInput" :disabled="!dropzoneFile">
      Clear<span v-if="dropzoneFile" class="bold"> selected file: </span>{{ dropzoneFile.name }}
    </button>

    <a href="sample.json" target="_blank" download>Sample JSON file</a>
  </div>
</template>

<style scoped>
.dropzone {
  width: 500px;
  height: 300px;
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  gap: 10px;
  border: 2px dashed hsla(160, 100%, 37%, 1);
  background-color: white;
  transition: 0.3s ease all;
}

.active-dropzone {
  border: 2px solid hsla(160, 100%, 37%, 1);
  background-color: hsla(160, 100%, 37%, 0.5);
}

.dropzone label {
  padding: 8px 12px;
  color: white;
  background-color: hsla(160, 100%, 37%, 1);
  transition: 0.3s ease all;
}

.dropzone input {
  display: none;
}

.clearInput {
  background-color: #ffffff;
  border: none;
  color: hsla(160, 100%, 37%, 1);
  font-weight: 800;
}

.clearInput:hover {
  background-color: #ffffff;
  cursor: pointer;
  text-decoration: underline;
}

.clearInput:disabled {
  cursor: default;
  text-decoration: none;
  color: grey;
}

.bold {
  font-weight: 800;
}

@media (min-width: 1024px) {
  .greetings h1,
  .greetings h3 {
    text-align: left;
  }
}
</style>
