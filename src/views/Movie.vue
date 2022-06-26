<template>
  <div
    v-if="movie"
    :style="{
      backgroundImage: movie.poster_path
        ? `url(https://image.tmdb.org/t/p/original${movie.backdrop_path})`
        : '',
    }"
    class="backdrop"
  ></div>
  <div class="backdrop-overlay"></div>

  <div v-if="movie" class="movie-container">
    <h1>{{ movie.title }}</h1>
    <p class="release-date">
      Release Date: <b>{{ movie.release_date ? movie.release_date : "N/A" }}</b>
    </p>
    <p class="description">{{ movie.overview ? movie.overview : "N/A" }}</p>
    <div
      class="poster"
      :style="{
        backgroundImage: movie.poster_path
          ? `url(https://image.tmdb.org/t/p/original${movie.poster_path})`
          : '',
      }"
    ></div>

    <div v-if="videos && videos.length" class="video">
      <h2>Video:</h2>

      <iframe
        allow="fullscreen;"
        width="560"
        height="315"
        :src="`https://www.youtube.com/embed/${videos[0].key}`"
      >
      </iframe>
    </div>

    <h2>Cast:</h2>
    <div v-if="cast && cast.length" class="cast">
      <div
        v-for="castMember in cast"
        :key="cast.id"
        class="cast-member"
        :style="{
          backgroundImage: castMember.profile_path
            ? `url(https://image.tmdb.org/t/p/original${castMember.profile_path})`
            : '',
        }"
      >
        <span style="text-align: center">
          {{ castMember.name }} <br />
          <span style="color: yellow">as</span> <br />
          <span>{{ castMember.character }}</span>
        </span>
      </div>
    </div>
    <div v-else>
      <p>N/A</p>
    </div>
  </div>
</template>

<script setup>
import { useRoute } from "vue-router";
import { onMounted, ref } from "vue";
import axios from "axios";
import secrets from "../secrets.json";

// import { useHead } from "@vueuse/head";

const route = useRoute();

const movie = ref(null);
const cast = ref(null);
const videos = ref(null);

const getCast = async () => {
  const config = {
    headers: {
      Authorization: `Bearer ${secrets.token}`,
      "Content-Type": "application/json",
    },
  };

  const res = await axios.get(
    `https://api.themoviedb.org/3/movie/${route.params.id}/credits`,
    config
  );

  cast.value = res.data.cast;
  console.log(cast.value);
};

const getVideos = async () => {
  const config = {
    headers: {
      Authorization: `Bearer ${secrets.token}`,
      "Content-Type": "application/json",
    },
  };

  const res = await axios.get(
    `https://api.themoviedb.org/3/movie/${route.params.id}/videos`,
    config
  );

  videos.value = res.data.results;
  console.log(videos.value);
};

const getMovie = async () => {
  const config = {
    headers: {
      Authorization: `Bearer ${secrets.token}`,
      "Content-Type": "application/json",
    },
  };

  const res = await axios.get(
    `https://api.themoviedb.org/3/movie/${route.params.id}`,
    config
  );
  
  movie.value = res.data;
  console.log(movie.value);
};

onMounted(() => {
  getMovie();
  getCast();
  getVideos();
});
</script>

<style lang="scss" scoped>
.video iframe {
  border-radius: 10px;
  border: none;
}
div.backdrop {
  width: 100vw;
  height: 50vh;
  position: absolute;
  top: 0px;
  left: 0px;
  background: #22252c no-repeat center fixed;
  -webkit-background-size: cover;
  -moz-background-size: cover;
  -o-background-size: cover;
  background-size: cover;
}

div.backdrop-overlay {
  width: 100vw;
  height: 100vh;
  position: absolute;
  top: 0px;
  bottom: 100vh;
  left: 0px;
  background-image: linear-gradient(
    rgba(34, 37, 44, 0.205),
    rgb(34, 37, 44),
    #22252c
  );
}

div.movie-container {
  width: 100%;
  position: relative;

  & h1 {
    font-size: 3rem;
    margin: 0;
    max-width: 60%;
    margin-top: 40vh;
  }

  & p.description {
    color: #ffffffad;
    max-width: 60%;
  }

  & .poster {
    width: 250px;
    height: 370px;
    background: url("/image_not_av.jpg") no-repeat center center;
    -webkit-background-size: cover;
    -moz-background-size: cover;
    -o-background-size: cover;
    background-size: cover;
    border-radius: 10px;
    position: absolute;
    top: -200px;
    right: 0;
    display: flex;
    justify-content: center;
    align-items: center;
    font-size: x-large;
    font-weight: 500;
    margin-bottom: 100px;
  }

  & .cast {
    display: flex;
    width: 100%;
    overflow-x: auto;
    overflow-y: hidden;
    height: 370px;

    &::-webkit-scrollbar {
      width: 12px;
      /* width of the entire scrollbar */
    }

    &::-webkit-scrollbar-track {
      background: rgba(0, 0, 0, 0.205);
      border-radius: 100px;
      /* color of the tracking area */
    }

    &::-webkit-scrollbar-thumb {
      background-color: rgb(255, 255, 255);
      /* color of the scroll thumb */
      border-radius: 20px;
      /* roundness of the scroll thumb */
      border: 3px solid rgb(255, 255, 255);
      /* creates padding around scroll thumb */
    }

    & .cast-member {
      min-width: 230px;
      height: 230px;
      background: url("/nocastimg.jpg") no-repeat center center;
      -webkit-background-size: cover;
      -moz-background-size: cover;
      -o-background-size: cover;
      background-size: cover;
      border-radius: 200px;
      margin: 20px 20px;
      display: flex;
      justify-content: center;
      align-items: flex-end;

      & span {
        margin-bottom: -80px;
      }
    }
  }
}

@media (max-width: 700px) {
  div.backdrop {
    width: 100vw;
    height: 50vh;
    position: absolute;
    top: 0px;
    left: 0px;
    background: #22252c center no-repeat static;
    -webkit-background-size: cover;
    -moz-background-size: cover;
    -o-background-size: cover;
    background-size: cover;
  }
  div.movie-container {
    display: flex;
    flex-direction: column;
    align-items: center;
    text-align: center;

    & h1 {
      font-size: 2.5rem;
      text-align: center;
    }

    & p.description {
      text-align: center;
      max-width: 100%;
    }

    & .poster {
      position: initial;
    }
  }

  .video {
    width: 100%;
    max-width: 600px;
    height: 300px;

    & iframe {
      width: 100%;
      height: 100%;
    }
  }
}

@media (max-width: 800px) {
  div.movie-container {
    & .poster {
      position: initial;
      margin: 10px 0px;
    }
  }
}
</style>
