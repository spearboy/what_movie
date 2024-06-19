<script>
// Import Swiper core and required modules
import { Navigation, Pagination, Scrollbar, Autoplay, A11y } from 'swiper/modules'
import { Swiper, SwiperSlide } from 'swiper/vue'
import { ref, onMounted } from 'vue'
import { useRouter } from 'vue-router'
import axios from 'axios'

// Import Swiper styles
import 'swiper/css'
import 'swiper/css/autoplay'
import 'swiper/css/navigation'
import 'swiper/css/pagination'
import 'swiper/css/scrollbar'

import 'primeicons/primeicons.css'

export default {
  components: {
    Swiper,
    SwiperSlide
  },
  setup() {
    const latesMovie = ref([])
    const popMovie = ref([])
    const topRatedMovie = ref([])
    const upcomingMovie = ref([])
    const searchResults = ref([])
    const searchQuery = ref('')
    const showResults = ref(false)
    const router = useRouter()
    const api_key = 'ee7184575bd0a8e791229cc0bce088e4'
    const selectedMovie = ref(null)
    const trailerUrl = ref('')
    const showPreview = ref(false)

    const fetchMovies = async () => {
      try {
        const [latesResponse, popResponse, topRatedResponse, upcomingResponse] = await Promise.all([
          axios.get(
            `https://api.themoviedb.org/3/movie/now_playing?api_key=${api_key}&language=ko-KR`
          ),
          axios.get(`https://api.themoviedb.org/3/movie/popular?api_key=${api_key}&language=ko-KR`),
          axios.get(
            `https://api.themoviedb.org/3/movie/top_rated?api_key=${api_key}&language=ko-KR`
          ),
          axios.get(`https://api.themoviedb.org/3/movie/upcoming?api_key=${api_key}&language=ko-KR`)
        ])
        latesMovie.value = latesResponse.data.results
        popMovie.value = popResponse.data.results
        topRatedMovie.value = topRatedResponse.data.results
        upcomingMovie.value = upcomingResponse.data.results
      } catch (error) {
        console.log(error)
      }
    }

    const navigateToDetail = (movieId) => {
      router.push({ name: 'detail', params: { movieID: movieId } })
    }

    const fetchTrailer = async (movieId) => {
      try {
        const response = await axios.get(
          `https://api.themoviedb.org/3/movie/${movieId}/videos?api_key=${api_key}`
        )
        const trailer = response.data.results.find(
          (video) => video.type === 'Trailer' && video.site === 'YouTube'
        )
        trailerUrl.value = trailer ? `https://www.youtube.com/embed/${trailer.key}?autoplay=1` : ''
      } catch (error) {
        console.log(error)
      }
    }

    const openPreview = async (movie) => {
      selectedMovie.value = movie
      await fetchTrailer(movie.id)
      showPreview.value = true
    }

    const closePreview = () => {
      showPreview.value = false
      trailerUrl.value = ''
    }

    const searchMovies = async () => {
      if (searchQuery.value.trim() === '') {
        searchResults.value = []
        return
      }
      try {
        const response = await axios.get(
          `https://api.themoviedb.org/3/search/movie?api_key=${api_key}&language=ko-KR&query=${searchQuery.value}`
        )
        searchResults.value = response.data.results
        showResults.value = true
      } catch (error) {
        console.error('Error searching movies:', error)
      }
    }

    const clearSearch = () => {
      searchQuery.value = ''
      searchResults.value = []
      showResults.value = false
    }

    onMounted(() => {
      fetchMovies()
    })

    const onSwiper = (swiper) => {
      console.log(swiper)
    }
    const onSlideChange = () => {
      console.log('slide change')
    }
    return {
      onSwiper,
      onSlideChange,
      modules: [Navigation, Pagination, Scrollbar, Autoplay, A11y],
      latesMovie,
      popMovie,
      topRatedMovie,
      upcomingMovie,
      navigateToDetail,
      openPreview,
      closePreview,
      showPreview,
      trailerUrl,
      selectedMovie,
      searchQuery,
      searchResults,
      searchMovies,
      clearSearch,
      showResults
    }
  }
}
</script>

<template>
  <header id="header" role="banner">
    <div class="header__inner">
      <div class="container">
        <div class="main_menu">
          <h1>
            <svg
              baseProfile="tiny"
              height="60"
              version="1.2"
              width="324.0"
              xmlns="http://www.w3.org/2000/svg"
              xmlns:ev="http://www.w3.org/2001/xml-events"
              xmlns:xlink="http://www.w3.org/1999/xlink"
            >
              <defs />
              <text fill="#ffffff" font-family="Arial" font-size="40" x="60" y="45">
                WhatMovie?!
              </text>
              <g fill="#ffcc00">
                <circle cx="30.0" cy="30.0" r="20.0" />
                <circle cx="30.0" cy="30.0" fill="#333333" r="10.0" />
                <line stroke="#333333" stroke-width="2" x1="30.0" x2="30.0" y1="10.0" y2="50.0" />
                <line stroke="#333333" stroke-width="2" x1="10.0" x2="50.0" y1="30.0" y2="30.0" />
              </g>
            </svg>
          </h1>
          <label class="search_wrapper" for="pageSearch">
            <input
              type="search"
              id="pageSearch"
              v-model="searchQuery"
              @input="searchMovies"
              placeholder="영화 제목을 입력하세요"
            />
            <button @click="clearSearch">검색</button>

            <div v-if="showResults" class="search_results">
              <div
                class="search_result"
                v-for="result in searchResults"
                :key="result.id"
                @click="navigateToDetail(result.id)"
              >
                <img :src="`https://image.tmdb.org/t/p/w500${result.poster_path}`" alt="poster" />
                <div class="result_info">
                  <span class="result_title">{{ result.title }}</span>
                  <span class="result_overview">{{ result.overview }}</span>
                </div>
              </div>
            </div>
          </label>
        </div>
      </div>
    </div>
  </header>
  <article>
    <swiper
      :modules="modules"
      :slides-per-view="1.35"
      :space-between="20"
      navigation
      :autoplay="{ delay: 2000, disableOnInteraction: false }"
      loop
      :loop-additional-slides="2"
      centeredSlides
      @swiper="onSwiper"
      @slideChange="onSlideChange"
    >
      <swiper-slide
        v-for="(movie, index) in latesMovie"
        :key="index"
        @click="navigateToDetail(movie.id)"
      >
        <div
          class="benner_slider"
          :style="{
            backgroundImage: `url('https://image.tmdb.org/t/p/w1280${movie.backdrop_path}')`
          }"
        >
          <div class="bg_cover">
            <div class="info_box">
              <span class="avo">평점 : {{ movie.vote_average }}</span>
              <span class="title">{{ movie.title }}</span>
              <span class="desc">{{ movie.overview }}</span>
            </div>
          </div>
        </div>
      </swiper-slide>
    </swiper>
  </article>
  <main id="main">
    <div class="view__inner container">
      <h3>현재 상영 영화</h3>
      <div class="view__content__wrapper">
        <swiper
          :modules="modules"
          :slides-per-view="3"
          :space-between="20"
          navigation
          loop
          :loop-additional-slides="2"
          centeredSlides
          @swiper="onSwiper"
          @slideChange="onSlideChange"
          :breakpoints="{
            1000: {
              slidesPerView: 3,
              spaceBetween: 20
            },
            800: {
              slidesPerView: 3,
              spaceBetween: 20
            },
            700: {
              slidesPerView: 2,
              spaceBetween: 20
            },
            500: {
              slidesPerView: 1,
              spaceBetween: 10
            }
          }"
        >
          <swiper-slide
            class="view__card style_one"
            v-for="(movie, index) in latesMovie"
            :key="index"
            :style="{
              backgroundImage: `url('https://image.tmdb.org/t/p/w1280${movie.backdrop_path}')`
            }"
          >
            <div class="view__card__title">
              {{ movie.title }}
              <div class="move_detail" @click.stop="navigateToDetail(movie.id)">
                영화페이지 이동
              </div>
              <div class="show_preview" @click.stop="openPreview(movie)">티져보기</div>
            </div>
          </swiper-slide>
        </swiper>
      </div>
    </div>
    <div class="view__inner container">
      <h3>인기 영화</h3>
      <div class="view__content__wrapper">
        <swiper
          :modules="modules"
          :space-between="20"
          navigation
          loop
          :loop-additional-slides="2"
          @swiper="onSwiper"
          @slideChange="onSlideChange"
          :breakpoints="{
            1000: {
              slidesPerView: 4,
              spaceBetween: 20
            },
            800: {
              slidesPerView: 3,
              spaceBetween: 20
            },
            700: {
              slidesPerView: 2,
              spaceBetween: 20
            },
            500: {
              slidesPerView: 1,
              spaceBetween: 10
            }
          }"
        >
          <swiper-slide
            class="view__card style_three"
            v-for="(movie, index) in popMovie"
            :key="index"
            :style="{
              backgroundImage: `url('https://image.tmdb.org/t/p/w1280${movie.poster_path}')`
            }"
          >
            <div class="view__card__title">
              {{ movie.title }}
              <div class="move_detail" @click.stop="navigateToDetail(movie.id)">
                영화페이지 이동
              </div>
              <div class="show_preview" @click.stop="openPreview(movie)">티져보기</div>
            </div>
          </swiper-slide>
        </swiper>
      </div>
    </div>
    <div class="view__inner container">
      <h3>평점 높은 영화</h3>
      <div class="view__content__wrapper">
        <swiper
          :modules="modules"
          :space-between="20"
          navigation
          loop
          :loop-additional-slides="2"
          @swiper="onSwiper"
          @slideChange="onSlideChange"
          :breakpoints="{
            1000: {
              slidesPerView: 4,
              spaceBetween: 20
            },
            800: {
              slidesPerView: 3,
              spaceBetween: 20
            },
            700: {
              slidesPerView: 2,
              spaceBetween: 20
            },
            500: {
              slidesPerView: 1,
              spaceBetween: 10
            }
          }"
        >
          <swiper-slide
            class="view__card style_three"
            v-for="(movie, index) in topRatedMovie"
            :key="index"
            :style="{
              backgroundImage: `url('https://image.tmdb.org/t/p/w1280${movie.poster_path}')`
            }"
          >
            <div class="view__card__title">
              {{ movie.title }}
              <div class="move_detail" @click.stop="navigateToDetail(movie.id)">
                영화페이지 이동
              </div>
              <div class="show_preview" @click.stop="openPreview(movie)">티져보기</div>
            </div>
          </swiper-slide>
        </swiper>
      </div>
    </div>
    <div class="view__inner container">
      <h3>개봉 예정 영화</h3>
      <div class="view__content__wrapper">
        <swiper
          :modules="modules"
          :space-between="20"
          navigation
          loop
          :loop-additional-slides="2"
          @swiper="onSwiper"
          @slideChange="onSlideChange"
          :breakpoints="{
            1000: {
              slidesPerView: 4,
              spaceBetween: 20
            },
            800: {
              slidesPerView: 3,
              spaceBetween: 20
            },
            700: {
              slidesPerView: 2,
              spaceBetween: 20
            },
            500: {
              slidesPerView: 1,
              spaceBetween: 10
            }
          }"
        >
          <swiper-slide
            class="view__card style_three"
            v-for="(movie, index) in upcomingMovie"
            :key="index"
            :style="{
              backgroundImage: `url('https://image.tmdb.org/t/p/w1280${movie.poster_path}')`
            }"
          >
            <div class="view__card__title">
              {{ movie.title }}
              <div class="move_detail" @click.stop="navigateToDetail(movie.id)">
                영화페이지 이동
              </div>
              <div class="show_preview" @click.stop="openPreview(movie)">티져보기</div>
            </div>
          </swiper-slide>
        </swiper>
      </div>
    </div>
  </main>
  <footer>
    <div class="container">
      <p>포트폴리오용 페이지 입니다.</p>
    </div>
  </footer>
  <div class="preview" :class="{ active: showPreview }">
    <div class="preview_container">
      <div class="play_video">
        <iframe
          :src="trailerUrl"
          frameborder="0"
          allow="autoplay; encrypted-media"
          allowfullscreen
        ></iframe>
      </div>
      <div class="info">
        <p class="title">{{ selectedMovie?.original_title }}</p>
        <p>{{ selectedMovie?.overview }}</p>
      </div>
      <div class="close" @click="closePreview">X</div>
    </div>
  </div>
</template>
<style lang="scss">
.preview {
  position: fixed;
  top: 100%;
  left: 0px;
  z-index: 1000;
  width: 100vw;
  height: 100vh;
  transition: top 0.3s ease;
  background: linear-gradient(transparent, #000);
  &.active {
    top: 0;
  }
  .preview_container {
    width: 100%;
    display: flex;
    justify-content: flex-start;
    align-items: flex-start;
    flex-direction: row;
    .play_video {
      position: absolute;
      width: 100%;
      height: 100%;
      iframe {
        width: 100%;
        height: 100%;
      }
    }
    .info {
      position: absolute;
      right: 0px;
      top: 0px;
      width: 100%;
      height: 100%;
      z-index: 10;
      padding: 20px;
      background: linear-gradient(0.42turn, transparent, transparent, #000);
      color: white;
      display: flex;
      flex-direction: column;
      align-items: flex-end;
      justify-content: flex-end;
      p {
        text-align: right;
        width: 30%;
        font-size: 20px;
        &.title {
          font-size: 30px;
          font-weight: bold;
        }
      }
    }
    .close {
      position: absolute;
      top: 13px;
      left: 50%;
      width: 50px;
      height: 50px;
      transform: translateX(-50%);
      z-index: 1111;
      border-radius: 100%;
      background-color: var(--black);
      color: var(--white);
      cursor: pointer;
      font-size: 30px;
      display: flex;
      box-sizing: border-box;
      flex-direction: row;
      align-items: center;
      justify-content: center;
    }
  }
}
.benner_slider {
  width: 100%;
  height: 690px;
  border-radius: 20px;
  background-position: center;
  background-repeat: no-repeat;
  background-size: cover;
  position: relative;
  overflow: hidden;
  @media (max-width: 1400px) {
    height: 400px;
  }
  @media (max-width: 700px) {
    height: 200px;
  }
  .bg_cover {
    background: linear-gradient(transparent, transparent, transparent, transparent, #000);
    position: absolute;
    bottom: 0px;
    left: 0px;
    width: 100%;
    height: 100%;
    .info_box {
      position: absolute;
      bottom: 0px;
      left: 0px;
      width: 100%;
      display: flex;
      flex-direction: column;
      align-items: flex-start;
      justify-content: flex-start;
      padding-left: 20px;
      padding-bottom: 20px;
      span {
        font-size: 20px;
        display: block;
        color: var(--white);
      }
      .avo {
        font-size: 18px;
      }
      .title {
        font-size: 40px;
        font-weight: 600;
      }
      .desc {
        max-width: 500px;
        overflow: hidden;
        text-overflow: ellipsis;
        white-space: nowrap;
      }
      @media (max-width: 1400px) {
        .avo {
          font-size: 16px;
        }
        .title {
          font-size: 30px;
          font-weight: 600;
        }
        .desc {
          max-width: calc(100% - 40px);
          font-size: 14px;
        }
      }
      @media (max-width: 700px) {
        .avo {
          font-size: 16px;
        }
        .title {
          font-size: 20px;
          font-weight: 600;
        }
        .desc {
          max-width: calc(100% - 40px);
          font-size: 14px;
        }
      }
    }
  }
}
#header {
  position: sticky;
  top: 28px;
  left: 0px;
  z-index: 100;
  background-color: var(--black300);
}
.header__inner {
  padding: 6px;
  @media (max-width: 1400px) {
    padding-bottom: 20px;
  }
  .main_menu {
    width: 100%;
    display: flex;
    flex-direction: row;
    align-items: center;
    justify-content: space-between;
    @media (max-width: 1400px) {
      flex-direction: column;
    }
    h1 {
      font-size: 40px;
      color: var(--white);
      font-weight: 900;
    }
    .search_wrapper {
      width: 100%;
      max-width: 300px;
      background-color: var(--black);
      border: 0px;
      border-radius: 50px;
      display: flex;
      flex-direction: row;
      align-items: center;
      justify-content: flex-end;
      position: relative;
      input {
        width: 100%;
        height: 40px;
        background-color: transparent;
        border: 0px;
        outline: 0px;
        color: var(--white);
        appearance: none;
        padding: 10px;
      }
      button {
        min-width: fit-content;
        background-color: transparent;
        color: var (--white);
        border-radius: 20px;
        width: 0px;
        height: 0px;
        visibility: hidden;
        overflow: hidden;
      }
      .search_results {
        width: 100%;
        max-width: 300px;
        max-height: 330px;
        background-color: var(--black200);
        border-radius: 10px;
        padding: 10px;
        position: absolute;
        top: 60px;
        right: 0px;
        overflow: auto;
        overflow-x: hidden;
        z-index: 101;
        .search_result {
          display: flex;
          flex-direction: row;
          align-items: center;
          justify-content: flex-start;
          padding: 10px;
          cursor: pointer;
          width: 100%;
          &:hover {
            background-color: var(--black300);
          }
          img {
            width: 50px;
            height: 75px;
            border-radius: 5px;
            margin-right: 10px;
          }
          .result_info {
            display: flex;
            flex-direction: column;
            align-items: flex-start;
            justify-content: center;
            width: calc(100% - 75px);
            .result_title {
              font-size: 18px;
              font-weight: bold;
              color: var(--white);
              overflow: hidden;
              text-overflow: ellipsis;
              white-space: nowrap;
              max-width: 100%;
            }
            .result_overview {
              font-size: 14px;
              color: var(--white);
              overflow: hidden;
              text-overflow: ellipsis;
              white-space: nowrap;
              max-width: 100%;
            }
          }
        }
      }
    }
  }
}

@media (max-width: 1400px) {
  article {
    padding-top: 50px;
  }
}
.view__inner {
  display: flex;
  flex-direction: row;
  align-items: flex-start;
  justify-content: flex-start;
  flex-wrap: wrap;
  gap: 20px;
  margin-top: 120px;
  @media (max-width: 1400px) {
    width: 100%;
    padding: 0px 20px;
    margin-top: 80px;
  }
  h3 {
    width: 100%;
    font-size: 30px;
    font-weight: 600;
    color: var(--white);
  }
  .view__content__wrapper {
    width: 100%;
    display: flex;
    flex-direction: row;
    align-items: center;
    justify-content: flex-start;
    gap: 20px;
    .view__card {
      position: relative;
      border-radius: 20px;
      overflow: hidden;
      cursor: pointer;
      .view__card__title {
        position: absolute;
        width: 100%;
        height: 100%;
        background: linear-gradient(transparent, #000);
        z-index: 1;
        display: flex;
        flex-direction: column;
        align-items: flex-start;
        justify-content: flex-end;
        padding: 10px;
        color: var(--white);
        font-size: 30px;
        opacity: 0;
        transition: all 0.3s ease;
        .move_detail {
          position: absolute;
          top: 10px;
          right: 10px;
          width: fit-content;
          height: 30px;
          background-color: var(--black300);
          font-size: 18px;
          display: flex;
          flex-direction: row;
          align-items: center;
          justify-content: center;
          padding: 6px 10px;
          border-radius: 4px;
        }
        .show_preview {
          position: absolute;
          top: 50px;
          right: 10px;
          width: fit-content;
          height: 30px;
          background-color: var(--black300);
          font-size: 18px;
          display: flex;
          flex-direction: row;
          align-items: center;
          justify-content: center;
          padding: 6px 10px;
          border-radius: 4px;
        }
      }
      &:hover .view__card__title {
        opacity: 1;
      }
    }
    .view__card.style_one {
      width: 500px;
      height: 200px;
      background-color: azure;
      background-position: center;
      background-repeat: no-repeat;
      background-size: cover;
      @media (max-width: 700px) {
        width: 100%;
      }
    }
    .view__card.style_two {
      width: 300px;
      height: 400px;
      background-position: center;
      background-repeat: no-repeat;
      background-size: cover;
      @media (max-width: 700px) {
        width: 100%;
      }
    }
    .view__card.style_three {
      width: 200px;
      height: 490px;
      background-position: center;
      background-repeat: no-repeat;
      background-size: cover;
      @media (max-width: 700px) {
        width: 100%;
      }
    }
  }
}
footer {
  width: 100%;
  height: 200px;
  margin-top: 150px;
  background-color: var(--black200);
  div {
    height: 100%;
    display: flex;
    flex-direction: row;
    align-items: center;
    justify-content: center;
  }
}
</style>
