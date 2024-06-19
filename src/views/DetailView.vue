<script>
import { ref, onMounted, watch } from 'vue'
import axios from 'axios'
import { useRoute, useRouter } from 'vue-router'

export default {
  setup() {
    const route = useRoute()
    const router = useRouter()
    const movieID = ref(route.params.movieID)
    const movieDetail = ref(null)
    const recommendedMovies = ref([])
    const actorList = ref([])
    const director = ref(null)
    const api_key = 'ee7184575bd0a8e791229cc0bce088e4'

    const fetchMovieDetail = async () => {
      try {
        const response = await axios.get(
          `https://api.themoviedb.org/3/movie/${movieID.value}?api_key=${api_key}&language=ko-KR`
        )
        movieDetail.value = response.data
        fetchMovieCredits()
      } catch (error) {
        console.error('Error fetching movie details:', error)
      }
    }

    const fetchRecommendedMovies = async () => {
      try {
        const response = await axios.get(
          `https://api.themoviedb.org/3/movie/${movieID.value}/recommendations?api_key=${api_key}&language=ko-KR`
        )
        recommendedMovies.value = response.data.results
      } catch (error) {
        console.error('Error fetching recommended movies:', error)
      }
    }

    const fetchMovieCredits = async () => {
      try {
        const response = await axios.get(
          `https://api.themoviedb.org/3/movie/${movieID.value}/credits?api_key=${api_key}&language=ko-KR`
        )
        actorList.value = response.data.cast
        director.value = response.data.crew.find((member) => member.job === 'Director')
      } catch (error) {
        console.error('Error fetching movie credits:', error)
      }
    }

    const navigateToDetail = (id) => {
      router.push({ name: 'detail', params: { movieID: id } }).then(() => {
        movieID.value = id
        window.location.reload()
      })
    }

    const openProfile = (id) => {
      window.open(`https://www.themoviedb.org/person/${id}`, '_blank')
    }

    onMounted(() => {
      fetchMovieDetail()
      fetchRecommendedMovies()
    })

    watch(
      () => route.params.movieID,
      (newMovieID) => {
        movieID.value = newMovieID
        fetchMovieDetail()
        fetchRecommendedMovies()
      }
    )

    return {
      movieDetail,
      recommendedMovies,
      actorList,
      director,
      navigateToDetail,
      openProfile
    }
  }
}
</script>

<template>
  <div v-if="movieDetail" class="detail_wrap">
    <div
      class="detail_backDropPath"
      :style="{
        backgroundImage: `url('https://image.tmdb.org/t/p/w1280${movieDetail.backdrop_path}')`
      }"
    ></div>
    <div class="detail_info_wrap">
      <div class="container">
        <p class="tagline">{{ movieDetail.tagline }}</p>
        <p class="title">{{ movieDetail.title }}</p>
        <div class="type">
          <p>장르:</p>
          <span v-for="genre in movieDetail.genres" :key="genre.id">{{ genre.name }}</span>
        </div>
        <div class="comp">
          <p>제작사:</p>
          <span v-for="company in movieDetail.production_companies" :key="company.id">{{
            company.name
          }}</span>
        </div>
        <div class="director" v-if="director">
          <p>감독:</p>
          <div class="profile_scroll_box">
            <div class="profile_box" @click="openProfile(director.id)">
              <div
                class="img_box"
                :style="{
                  backgroundImage: `url('https://image.tmdb.org/t/p/w1280${director.profile_path}')`
                }"
              ></div>
              <span>{{ director.name }}</span>
            </div>
          </div>
        </div>
        <div class="actor">
          <p>출연자:</p>
          <div class="profile_scroll_box">
            <div
              class="profile_box"
              v-for="actor in actorList"
              :key="actor.cast_id"
              @click="openProfile(actor.id)"
            >
              <div
                class="img_box"
                :style="{
                  backgroundImage: `url('https://image.tmdb.org/t/p/w1280${actor.profile_path}')`
                }"
              ></div>
              <span>{{ actor.name }}</span>
              <span class="actor_character">{{ actor.character }}</span>
            </div>
          </div>
        </div>
        <p class="vote">평점: {{ movieDetail.vote_average }}</p>
        <p class="vote_count">평점 투표수: {{ movieDetail.vote_count }}</p>
        <span v-if="movieDetail.adult">19+</span>
        <p class="rel">개봉일: {{ movieDetail.release_date }}</p>
        <p class="desc">{{ movieDetail.overview }}</p>
      </div>
      <div class="container">
        <div class="other_movie_list">
          <p>이런 영화는 어떠세요?</p>
          <div
            class="movie_card"
            v-for="movie in recommendedMovies"
            :key="movie.id"
            @click="navigateToDetail(movie.id)"
          >
            <div
              class="movie_card_image"
              :style="{
                backgroundImage: `url('https://image.tmdb.org/t/p/w500${movie.poster_path}')`
              }"
            ></div>
            <div class="movie_card_title">{{ movie.title }}</div>
          </div>
        </div>
      </div>
    </div>
  </div>
  <div v-else>
    <p>Loading...</p>
  </div>
</template>

<style lang="scss">
.detail_wrap {
  .detail_backDropPath {
    position: fixed;
    top: 0px;
    left: 0px;
    width: 100%;
    height: 700px;
    background-size: cover;
    background-position: center;
  }
  .detail_info_wrap {
    position: sticky;
    top: 20px;
    left: 0px;
    width: 100%;
    height: auto;
    margin-top: 0px;
    margin-top: 300px;
    margin-top: 620px;
    background-color: var(--black300);
    color: var(--white);
    padding-top: 100px;
    &::after {
      content: '';
      position: absolute;
      top: -100px;
      left: 0px;
      width: 100%;
      height: 100px;
      background: linear-gradient(transparent, #333);
    }
    .container {
      max-width: 800px;
    }

    .type,
    .comp {
      width: 100%;
      display: flex;
      flex-direction: row;
      align-items: center;
      justify-content: flex-start;
      flex-wrap: wrap;
      gap: 10px;
      margin-bottom: 20px;
      span {
        width: fit-content;
        height: fit-content;
        display: inline-block;
        padding: 4px 10px;
        color: var(--white);
        border-radius: 6px;
        text-align: center;
        background-color: var(--black);
      }
    }
    .actor,
    .director {
      width: 100%;
      display: flex;
      flex-direction: row;
      align-items: stretch;
      justify-content: flex-start;
      flex-wrap: wrap;
      gap: 10px;
      margin-bottom: 20px;
      p {
        width: 100%;
      }
      .profile_scroll_box {
        width: 100%;
        overflow-x: auto;
        display: flex;
        flex-direction: row;
        align-items: stretch;
        justify-content: flex-start;
        gap: 10px;
        .profile_box {
          background-color: var(--black);
          width: 150px;
          min-width: 150px;
          height: 100%;
          display: flex;
          flex-direction: column;
          align-items: stretch;
          justify-content: stretch;
          border-radius: 10px;
          overflow: hidden;
          position: relative;
          cursor: pointer;
          .img_box {
            width: 100%;
            height: 170px;
            min-height: 170px;
            background-position: center;
            background-size: cover;
            background-repeat: no-repeat;
            background-color: #fff;
          }
          span {
            width: 100%;
            height: 100%;
            display: flex;
            justify-content: center;
            align-items: center;
            padding: 4px 10px;
            color: var(--white);
            text-align: center;
          }
          .actor_character {
            position: absolute;
            top: 0px;
            left: 0px;
            width: 100%;
            height: 100%;
            display: flex;
            flex-direction: column;
            align-items: center;
            justify-content: center;
            background-color: rgba(0, 0, 0, 0.7);
            transition: all 0.3s ease;
            opacity: 0;
          }
          &:hover .actor_character {
            opacity: 1;
          }
        }
      }
    }
    .director {
      width: fit-content;
      display: flex;
      flex-direction: row;
      align-items: center;
      justify-content: flex-start;
      flex-wrap: wrap;
      gap: 10px;
      margin-bottom: 20px;
      span {
        display: inline-block;
        border-radius: 6px;
        padding: 4px 10px;
        color: var(--white);
        background-color: var(--black);
      }
    }
    .title {
      font-size: 50px;
      font-weight: 600;
      margin-bottom: 20px;
    }
    .rel {
      display: inline-block;
      border-radius: 6px;
      padding: 4px 10px;
      color: var(--white);
      background-color: var(--black);
      margin-bottom: 100px;
    }
    .desc {
      position: relative;
      font-size: 20px;
      font-weight: 600;
      &::before {
        content: '줄거리';
        position: absolute;
        top: -50px;
        left: 0px;
        font-size: 22px;
        border-radius: 6px;
        padding: 4px 10px;
        color: var(--white);
        background-color: var(--black);
      }
    }
    .vote {
      margin-bottom: 10px;
    }
    .vote_count {
      margin-bottom: 10px;
    }
    .other_movie_list {
      margin-top: 90px;
      display: flex;
      flex-direction: row;
      align-items: flex-start;
      justify-content: flex-start;
      flex-wrap: wrap;
      gap: 25px;

      p {
        width: 100%;
        font-size: 30px;
        font-weight: 600;
      }
      .movie_card {
        display: flex;
        flex-direction: column;
        align-items: center;
        justify-content: center;
        margin-bottom: 20px;
        position: relative;
        overflow: hidden;
        border-radius: 10px;
        cursor: pointer;
        .movie_card_image {
          width: 250px;
          height: 400px;
          background-size: cover;
          background-position: center;
          background-repeat: no-repeat;
        }
        .movie_card_title {
          position: absolute;
          width: 100%;
          height: 100%;
          font-size: 18px;
          font-weight: 600;
          text-align: center;
          display: flex;
          justify-content: center;
          align-items: center;
          padding: 20px;
          background-color: rgba(0, 0, 0, 0.8);
          transition: all 0.3s ease;
          opacity: 0;
        }
        &:hover .movie_card_title {
          opacity: 1;
        }
      }
    }
  }
}
</style>
