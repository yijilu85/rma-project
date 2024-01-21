<script setup>
import { computed, ref } from 'vue'
import SingleImage from './SingleImage.vue'
import Dropzone from './Dropzone.vue'

defineProps({})

var filters = ref([])
var filteredArr = ref([])
var single = ref()
var arr = []
var selectedFile = ref()
var imported = ref(false)
var singleImageLoaded = ref(false)

const load = function () {
  const files = selectedFile.value
  if (files.arr.length <= 0) {
    return false
  }
  filters.value.push('all')

  for (let i = 0; i < files.arr.length; i++) {
    arr.push(files.arr[i])
    if (!filters.value.includes(files.arr[i].category)) {
      console.log(files.arr[i].category)
      filters.value.push(files.arr[i].category)
    }
  }
  imported = true
  console.log(arr.length)
  loadFilter('all')
}

const loadFilter = function (category) {
  if (category == 'all') {
    filteredArr.value = arr
  } else {
    filteredArr.value = arr.filter((item) => item.category === category)
  }

  let filter_list = document.querySelectorAll('ul.filter-list li')

  for (let i = 0; i < filter_list.length; i++) {
    if (filter_list[i].classList.contains('active')) {
      filter_list[i].classList.remove('active')
    }
    if (filter_list[i].innerText == category) {
      filter_list[i].classList.add('active')
    }
  }
}

const loadImg = function (img) {
  single.value = img
  console.log('single', single)
}

const setFileSelection = (fileData) => {
  selectedFile.value = fileData
}

const clearSelection = () => {
  selectedFile.value = ''
}
</script>

<template>
  <div class="main">
    <h1>Magic Image Gallery</h1>

    <div>
      <SingleImage
        :source-image="single"
        v-if="single"
        @closeModal="() => (single = null)"
      ></SingleImage>
    </div>

    <div v-if="!imported">
      <Dropzone @file-selected="setFileSelection" @file-deselected="clearSelection"></Dropzone>
    </div>
    <div v-if="!imported">
      <a
        class="button-flex"
        :class="selectedFile ? 'button-primary' : 'button-inactive'"
        @click="load"
        :disabled="!selectedFile"
        >Import<img class="feather-white cta" src="../assets/arrow-right-circle-white.svg" />
      </a>
    </div>
    <div class="filters">
      <h3 v-if="imported">Categories</h3>
      <ul class="filter-list">
        <li v-for="filter in filters">
          <a @click="loadFilter(filter)">{{ filter }}</a>
        </li>
      </ul>
    </div>
    <div class="gallery" v-if="!single">
      <div class="gallery-item" v-for="img in filteredArr">
        <div class="img-meta">
          <h2>{{ img.name }}</h2>
          <p>Category: {{ img.category }}</p>
        </div>
        <img :src="img.src" alt="" @click="loadImg(img)" />
      </div>
    </div>
  </div>
</template>

<style scoped>
div.img-meta {
  display: flex;
  flex-flow: row nowrap;
  justify-content: space-between;
  align-items: flex-start;
}
ul.filter-list li.active a {
  color: white !important;
  background-color: hsla(160, 100%, 37%, 1);
  border-radius: 3px;
}

.button-inactive {
  /* background-color: hsla(160, 100%, 37%, 1); */
  color: white;
  padding: 0.5em 1em;
  transition: 0.4s;
  border-radius: 3px;
  background-color: lightgray;
  cursor: default;
}

ul.filter-list li {
  padding: 10px;
}
div.main {
  margin: 0 auto;
  display: flex;
  flex-flow: column nowrap;
  justify-content: flex-start;
  align-items: center;
  gap: 20px;
}
div.single {
  width: 100%;
}
div.gallery {
  margin: 2em 2em;
  display: flex;
  flex-flow: row wrap;
  justify-content: flex-start;
  gap: 20px;
}

div.filters {
  display: flex;
  flex-flow: row nowrap;
  align-items: center;
}
div.filters ul {
  list-style-type: none;
  display: flex;
  flex-flow: row wrap;
  gap: 20px;
}

div.gallery-item {
  flex-grow: 1;
  transition: 0.4s;
  max-width: 32%;
}

div.gallery-item img:hover {
  /* box-shadow: 3px 3px 1px #ccc; */
  box-shadow: 0 3px 10px rgb(0 0 0 / 0.4);
}

div.filters a {
  padding: 0.5em;
}
div.filters a:hover {
  cursor: pointer;
}
img {
  width: 100%;
}

.greetings h1,
.greetings h3 {
  text-align: center;
}

@media (min-width: 1024px) {
  .greetings h1,
  .greetings h3 {
    text-align: left;
  }
}
</style>
