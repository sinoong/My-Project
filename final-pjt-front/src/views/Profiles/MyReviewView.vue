<template>
  <div class="container">
    <h1 class="pageTitle">{{ route.params.username }}님이 작성한 리뷰</h1>
    <h6 class="subtitle">클릭하면 리뷰의 내용을 수정할 수 있어요.</h6>
    <div class="array">
      <ProfileReviewCard 
      v-for="review in reviews"
      :review="review"
      />
    </div>
  </div>
</template>

<script setup>
import { onMounted, ref } from 'vue';
import { useCounterStore } from '@/stores/counter';
import { useRoute } from 'vue-router';
import ProfileReviewCard from '@/components/community/ProfileReviewCard.vue';
import axios from 'axios';
const route = useRoute()
const store = useCounterStore()

const reviews = ref([])
onMounted(() => {
  axios({
    method : 'get',
    url : `http://127.0.0.1:8000/accounts/profile/${route.params.username}/`,
    headers : {
        Authorization:`Token ${store.Token}`
      }
  })
  .then((res) => {
    // console.log(res.data);
    reviews.value = res.data.review_set
    console.log(reviews.value)
  })
  .catch(err=>console.log(err))
  // store.getActors()
  // console.log(store.actors)
})
</script>

<style scoped>
.container {
  padding: 20px;
  background-color: #2b2b2b;
  border-radius: 10px;
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
  margin-top: 10px;
  
}

.pageTitle {
  font-size: 50px;
  color: #d9d8d8;
}

.subtitle {
  font-size: 20px;
  color: #989696;
}

.array {

}
</style>