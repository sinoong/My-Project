<template>
  <div>
    <div class="array">
      <div v-for="movie in movieDetail">
        <span @click="goMovieDetail(movie.movie_id)" @mouseover="openTrailerModal(movie.movie_id)">
          <img :src="movie.poster_path" alt="" />
        </span>
      </div>
    </div>

    <!-- TrailerModal 컴포넌트를 포함시킵니다. -->
    <!-- <TrailerModal v-if="showModal" :movieId="selectedMovieId" /> -->
  </div>
</template>

<script setup>
import axios from 'axios';
import { ref } from 'vue';
import { useRouter } from 'vue-router';
import { useCounterStore } from '../stores/counter';
import TrailerModal from '@/components/TrailerModal.vue';
const store = useCounterStore()
const router = useRouter()
const showModal = ref(false)
// const props = defineProps({
//   movieDetail : Array
// })
const { movieDetail } = defineProps(['movieDetail']);

const goMovieDetail = function (movieId) {
  router.push({name:'moviedetail', params:{ movieId: movieId }})
}

const selectedMovieId = ref(null);

const openTrailerModal = (movieId) => {
  console.log('dmgkgkgkgk')
  selectedMovieId.value = movieId;
  showModal.value = true;
};
</script>

<style scoped>
img {
  width: 200px;
  height: 250px;
  border-radius: 10%;
}

.array {
  display: grid;
  grid-template-columns: repeat(6, 1fr);
  grid-auto-rows: 350px;
  grid-gap: 10px;
  margin-top: 100px;
}

/* 카드에 호버 효과를 추가하는 스타일 */
img:hover {
  /* 여기에 원하는 호버 효과 스타일을 추가하세요 */
  border: 4px solid beige;
  /* 예: 테두리 추가 */
  cursor: pointer; /* 마우스 호버 시 커서를 포인터로 변경 */
}
</style>