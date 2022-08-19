<template>
  <main>
    <h1>PixGO</h1>
    <p class="subtitle">any pictures you can find!</p>
    <div class="search-wrap">
      <input class="input" type="text" v-model="updateValue"/>
      <button class="button" @click="validate(); fromOne()">search</button>
    </div>
    <ul class="img-wrap">
      <li v-for="(image, index) in images" :key="index">
        <a class="pic-wrap" :href="image.pageURL">
          <img class="image" :src="image.largeImageURL" alt="">
          <h3>{{ image.user }}</h3>
        </a>
      </li>
    </ul>
    <div class="page-wrap" v-if="isShow">
      <button :disabled="prevDisabled" class="button" type="button" @click="pageValue--; search()">prev</button>
      <p>{{ pageValue }}</p>
      <button :disabled="nextDisabled" class="button" type="button" @click="pageValue++; search()">next</button>
    </div>
    <h1 v-if="errorMessage">{{ errorMessage }}</h1>
  </main>
  
</template>

<script setup>
import { ref } from 'vue'
import axios from 'axios'

const images = ref([]);
const updateValue = ref('');
const pageValue = ref(1);
const maxPage = ref(0);
const isShow = ref(false);
const errorMessage = ref('');
const prevDisabled = ref(false);
const nextDisabled = ref(false);

const fromOne = () => {
  pageValue.value = 1;
}

const validate = () => {
  prevDisabled.value = false;
  nextDisabled.value = false;
  if(updateValue.value === '') {
    images.value.splice(0);
    isShow.value = false;
    errorMessage.value = 'There is no result.'
    pageValue.value = 1;
  } else {
    search();
  }
}

const search = () => {
  prevDisabled.value = false;
  nextDisabled.value = false;
  images.value.splice(0);
  errorMessage.value = '';
  if(pageValue.value < 1){
    error();
  } else {
    axiosConnect();
  }
}

const axiosConnect = () => {
  axios.get(`https://pixabay.com/api/?key=29010873-2080efedf3573cca0dbe855c1&q=${updateValue.value}&image_type=photo&page=${pageValue.value}`)
  .then((res) => {
    maxPage.value = Math.ceil(res.data.totalHits / 20);
    //非活性化処理
    if(pageValue.value === 1) {
    prevDisabled.value = true;
    }
    if (pageValue.value === maxPage.value) {
    nextDisabled.value = true;
    }
    //apiの結果の後の処理
    if (res.data.totalHits === 0 || maxPage.value < pageValue.value) {
      error();
    } else {
      isShow.value = true;
      res.data.hits.forEach(function(value){
      images.value.push(value);
      });
    }
  })
}

const error = () => {
  isShow.value = false;
  errorMessage.value = 'There is no result.'
  maxPage.value = null;
  pageValue.value = 1;
}

</script>

<style scoped>

.subtitle {
  margin-top: -30px;
}

.input {
  border: 2px solid rgb(228, 228, 228);
  border-radius: 5px;
}

.input:focus {
  outline: 2px solid #9fa4ff;
}

.button:focus {
  outline: 2px solid #9fa4ff;
}

.search-wrap {
  width: 300px;
  display: flex;
  justify-content: space-around;
}

main {
  max-width: 1280px;
  padding: 100px 100px 0 100px;
  display: flex;
  flex-direction: column;
  align-items: center;
}

.img-wrap {
  display: flex;
  flex-wrap: wrap;
  padding-left: 0;
  margin-top: 150px;
}

.img-wrap>li:nth-child(3n + 1) {
  margin-left: 0;
}

.img-wrap>li:nth-child(-n + 3) {
  margin-top: 0;
}

.img-wrap>li {
  list-style-type: none;
  margin-left: calc((1340px - 1164px) / 2);
  margin-top: 100px;
}

.page-wrap {
  margin-top: 100px;
  display: flex;
  width: 300px;
  justify-content: space-around;
}

.image {
  display: block;
  width: 300px;
  height: 300px;
  border-radius: 10px;
  filter: drop-shadow(0 0 1rem rgb(161, 161, 161));
  transition: all .5s;
}

.image:hover {
  opacity: .7;
}

@media screen and (max-width: 1340px) {
  .img-wrap>li {
  margin-left: calc((100vw - 1164px) / 2);
  }
}

@media screen and (max-width: 1260px) {
  .img-wrap {
  margin-top: 50px;
  flex-wrap: nowrap;
  flex-direction: column;
  }
  
  .img-wrap>li:nth-child(-n + 3) {
  margin-top: 100px;
  }

  .img-wrap>li {
  list-style-type: none;
  margin-left: 0;
  margin-top: 100px;
  }
}

</style>