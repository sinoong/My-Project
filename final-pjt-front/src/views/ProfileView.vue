<template>
  <div class="profile">
    <h1>Profile</h1>
    <div class="myprofile">
      <label for="profile-picture" class="profile-picture-label">
        <input
          type="file"
          id="profile-picture"
          accept="image/*"
          @change="handleProfilePictureChange"
          style="display: none"
        />
        <img
          v-if="profilePicture"
          :src="profilePicture"
          alt="프로필 사진"
          class="profile-picture"
        />
        <span v-else="profilePicture"
          :src="profilePicture"
          alt="프로필 사진"
          class="profile-nonpicture">프로필 추가</span>
        </label>
      </div>
      <div class="follow-count">
        <button @click="follow" v-show="followbutton">{{ isFollowing ? '언팔로우' : '팔로우' }}</button>
        <!-- 팔로우 목록 -->
        <div>
          <button class="count-follow" @click.prevent="followingModal = true">팔로우: {{ followings.length }}</button>
          <FollowList :follows="followings" v-show="followingModal" @close-follow="followingModal = false"/>
        </div>
        <!-- 팔로워 목록  -->
        <div>
          <button class="count-following" @click.prevent="followerModal = true">팔로워: {{ followers.length }}</button>
          <FollowList :follows="followers" v-show="followerModal" @close-follow="followerModal = false"/>
        </div>
      </div>
    <div class="category">
      <nav>
        <RouterLink :to="{name:'starrating', params:{username:route.params.username}}">⭐내 평가 ({{ ratingMovies.length }})</RouterLink> |
        <RouterLink :to="{name:'actorlike', params:{username:route.params.username}}">💖 배우 ({{ LikeActors.length }})</RouterLink> |
        <RouterLink :to="{name:'directorlike', params:{username:route.params.username}}">💖 감독 ({{ LikeDirectors.length }})</RouterLink> |
        <RouterLink :to="{name:'hopemovie', params:{username:route.params.username}}">🎬 보고싶어요 ({{ HopeMovies.length }})</RouterLink> |
        <RouterLink :to="{name:'myreview', params:{username:route.params.username}}">💬 나의 리뷰 ({{ myReview.length }})</RouterLink>
      </nav>
    </div>
  </div>
  <RouterView />
</template>

<script setup>
import axios from 'axios';
import { ref, onMounted, computed } from 'vue';
import { RouterLink, RouterView } from 'vue-router';
import { useCounterStore } from '../stores/counter';
import { useRoute } from 'vue-router';
import FollowList from '@/components/FollowList.vue';

const route = useRoute()
const store = useCounterStore()
const profilePicture = ref('');

const followingModal = ref(false)
const followerModal = ref(false)

// 프로필 사진 선택
// const handleProfilePictureChange = (event) => {
//   const file = event.target.files[0];
//   if (file) {
//     const reader = new FileReader();
//     reader.onload = (e) => {
//       profilePicture.value = e.target.result;
//     };
//     reader.readAsDataURL(file);
//     axios({
//       method : 'post',
//       url : `http://127.0.0.1:8000/accounts/profile/${store.tempUsername}/picture/`,
//       headers : {
//           Authorization:`Token ${store.Token}`
//         },
//       data : {
//         profile_picture : file
//       }
//     })
//     .then((res) => {
//       console.log(res);
//     })
//   }
// };


// ------------------
const handleProfilePictureChange = (event) => {
  const file = event.target.files[0];

  if (file) {
    const reader = new FileReader();
    reader.onload = (e) => {
      // Assuming profilePicture is a ref or reactive property
      profilePicture.value = e.target.result;
    };
    reader.readAsDataURL(file);

    // Create FormData to handle file upload
    const formData = new FormData();
    formData.append('profile_picture', file);

    // Add CSRF token if needed
    formData.append('csrfmiddlewaretoken', '{{ csrf_token }}');

    axios({
      method: 'post',
      url: `http://127.0.0.1:8000/accounts/profile/${store.tempUsername}/picture/`,
      headers: {
        Authorization: `Token ${store.Token}`,
        'Content-Type': 'multipart/form-data', // Specify content type for file upload
      },
      data: formData,
    })
      .then((res) => {
        console.log(res);
      })
      .catch((error) => {
        console.error('Error uploading profile picture', error);
      });
  }
};

// ----------------------------


// 팔로우 기능 

const followers = ref([])
const followings = ref([])

const followbutton = computed(() => {
  if (store.tempUsername === route.params.username) {
    return false
  } else {
    return true
  }
})

const isFollowing = computed(() => {
  return followers.value.some(follower => follower.username === store.tempUsername);
})

const follow = function () {
  axios({
    method : 'post',
    url : `http://127.0.0.1:8000/accounts/profile/${route.params.username}/`,
    headers : {
        Authorization:`Token ${store.Token}`
      }
  })
  .then((res) => {
    // console.log(res)
    if (res.data.is_followed === true) {
      followers.value.push(
        { id : res.data.userId,
          username : store.tempUsername
        }
      )
    } else {
      const index = followers.value.findIndex((follow) => follow.username === store.tempUsername)
      followers.value.splice(index, 1)
    }
    // console.log(followers.value)
  })
  .catch(err=>console.log(err))
}
// 영화 감독 배우 조하여
const LikeActors = ref([])
const HopeMovies = ref([])
const LikeDirectors = ref([])

const ratingMovies = ref(0);

// 내 리뷰들 으하하
const myReview = ref([])
onMounted(() => {
  console.log(route.params.username);
  axios({
    method : 'get',
    url : `http://127.0.0.1:8000/accounts/profile/${route.params.username}/`,
    headers : {
        Authorization:`Token ${store.Token}`
      }
  })
  .then((res) => {
    console.log(res);
    myReview.value = res.data.review_set
    followers.value = res.data.followers
    followings.value = res.data.followings
    LikeActors.value = res.data.like_actor
    LikeDirectors.value = res.data.like_director
    HopeMovies.value = res.data.like_movies
    ratingMovies.value = res.data.score_set
    // console.log(LikeActors.value);
  })
  .catch(err=>console.log(err))
  // store.getActors()
  // console.log(store.actors)
});
</script>

<style scoped>
.profile {
  max-width: 800px;
  margin: 0 auto;
  padding: 20px;
  text-align: center;
}

h1 {
  font-family: disney;
  font-size: 100px;
  margin-bottom: 30px;
  /* color: #4caf50; */
}

.myprofile {
  /* background-color: #f5f5f5; */
  padding: 20px;
  border-radius: 8px;
  box-shadow: 0 0 10px rgba(0, 0, 0, 0.1);
  text-align: center;
  margin-bottom: 20px;

  
}

.myprofile img {
  /* 내 사진 */
  width: 300px;
  height: 300px;
  border-radius: 50%;
  object-fit: cover;
}

button {
  padding: 5px 20px;
  font-size: 16px;
  cursor: pointer;
  background-color: #4caf50;
  color: #fff;
  border: none;
  border-radius: 4px;
  transition: background-color 0.3s;
}

button:hover {
  background-color: #d7dbd7;
}
.follow-count {
  position: relative;
}
.count-follow {
  position: absolute;
  bottom: 200px;
  left: 600px;
  font-size: 20px;
}
.count-follow button {
  
}
.count-following {
  position: absolute;
  bottom: 150px;
  left: 600px;
  font-size: 20px;
  background-color: #3498db;
}
.count-following button {
  margin-top: 10px;
}


.category nav {
  margin-top: 50px;
  text-align: center;
  background-color: rgb(245, 244, 244);
}


.category nav a {
  margin: 0 20px;
  text-decoration: none;
  color: rgb(112, 111, 111);
  font-size: 18px;
  font-weight: bold;
  transition: color 0.3s;
}

.category nav a:hover {
  color: #4caf50;
}
.category {
  margin: 20px;
}

nav {
  display: flex;
  justify-content: space-around;
  background-color: white;
  padding: 10px;
  border-radius: 8px;
  border: 3px solid #4caf50;
}

.profile-picture {
  /* 사진 테두리 */
  width: 200px; 
  height: 200px; 
  border-radius: 50%;
  object-fit: cover;
  margin-bottom: 10px;
  border: 2px solid #4caf50; 
  transition: border-color 0.3s;
}

.profile-picture:hover {
  border-color: #45a049; /* 마우스 오버 시 더 어두운 녹색 */
}
.profile-picture-label span {
  /* 사진 없을 때 */
  display: inline-block;
  padding: 10px;
  color: rgb(200, 197, 197);
  cursor: pointer;
  width: 300px;
  height: 300px;
}
.profile-nonpicture {
  border-radius: 300px;
  background-color: rgb(236, 231, 231);
  width: 200px; 
  height: 200px; 
  border-radius: 50%;
  object-fit: cover;
  margin-bottom: 10px;
  border: 4px solid #4caf50;
  line-height: 250px;
  text-align: center;
  font-size: 30px;

}
.profile-nonpicture:hover {
  background-color: #45a049; /* 마우스 오버 시 더 어두운 녹색 */
  color: #fff; /* 마우스 오버 시 흰색 텍스트 */
}
</style>
