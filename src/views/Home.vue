<script setup>
import { ref, onMounted, watch } from "vue";
import secrets from "../secrets.json";
import axios from "axios";
import { useRouter } from "vue-router";
import { store } from "@/main.js";

const router = useRouter();

const movies = ref(null);
const moviesInput = ref();
const page = ref(1);
const pages = ref();
const rateLimit = ref(false);

const suggestionAdded = ref(false);
const suggestionExists = ref(false);

const getMovies = async (pageParam) => {
  const config = {
    headers: {
      Authorization: `Bearer ${secrets.token}`,
      "Content-Type": "application/json",
    },
  };
  const res = await axios.get(
    `https://api.themoviedb.org/3/trending/movie/day?page=${pageParam}`,
    config
  );
  console.log(res.data);
  movies.value = res.data.results;
  page.value = pageParam;
  pages.value = res.data.total_pages;
};

const searchMovies = async (e) => {
  e.preventDefault();
  if (!moviesInput.value || !moviesInput.value.length) {
    getMovies(page.value);
    return;
  }
  const config = {
    headers: {
      Authorization: `Bearer ${secrets.token}`,
      "Content-Type": "application/json",
    },
  };
  movies.value = null;
  const res = await axios.get(
    `https://api.themoviedb.org/3/search/movie?page=1&query=${moviesInput.value}`,
    config
  );
  console.log(res.data);
  movies.value = res.data.results;
  page.value = 1;
  pages.value = res.data.total_pages;
  console.log(movies.value);
};

const searchMoviesPages = async (pageParam) => {
  if (!moviesInput.value || !moviesInput.value.length) {
    getMovies(page.value);
    return;
  }

  pageParam = pageParam;
  const config = {
    headers: {
      Authorization: `Bearer ${secrets.token}`,
      "Content-Type": "application/json",
    },
  };
  movies.value = null;
  const res = await axios.get(
    `https://api.themoviedb.org/3/search/movie?page=${pageParam}&query=${moviesInput.value}`,
    config
  );
  console.log(res.data);
  movies.value = res.data.results;
  pages.value = res.data.total_pages;
  console.log(movies.value);
};

const getMovie = async (id) => {
  const config = {
    headers: {
      Authorization: `Bearer ${secrets.token}`,
      "Content-Type": "application/json",
    },
  };

  const res = await axios.get(
    `https://api.themoviedb.org/3/movie/${id}`,
    config
  );
  return res.data;
};

const goToMovie = async (e) => {
  if (e.target.className !== "movie") return;
  const movieId = e.target.id;
  const movie = await getMovie(movieId);
  router.push({
    name: "Movie",
    params: {
      id: movieId,
      movieName: movie.title
        ? movie.title.replace(/ /g, "-")
        : movie.name.replace(/ /g, "-"),
      movieRelease: movie.release_date
        ? movie.release_date.split("-", 1)[0]
        : "N/A",
    },
  });
  console.log(movie);
};

const addSuggestion = async (e) => {
  const data = {
    name: e.target.getAttribute("data-title"),
    release_date: e.target.getAttribute("data-release_date"),
    poster_path: e.target.getAttribute("data-poster_path"),
    vote_average: e.target.getAttribute("data-vote_average"),
    overview: e.target.getAttribute("data-overview"),
    id: e.target.getAttribute("data-id"),
  };

  try {
    const res = await axios.post("https://api.bajmovies.xyz/suggestions", data);
    console.log(res.data);
    suggestionExists.value = false;
    suggestionAdded.value = true;
    setTimeout(() => {
      suggestionAdded.value = false;
    }, 3000);
  } catch (err) {
    console.log(err.response.data.message);

    suggestionAdded.value = false;
    suggestionExists.value = true;
    setTimeout(() => {
      suggestionExists.value = false;
    }, 3000);
  }
};

const logOut = () => {
  store.user = null;
  localStorage.removeItem("user");
};

const logIn = () => {
  router.push("/login");
};

onMounted(() => {
  getMovies(page.value);
});

watch(moviesInput, () => {
  console.log(moviesInput.value);
});

watch(page, () => {
  searchMoviesPages(page.value);
});

const nextPage = () => {
  if (!rateLimit.value) {
    page.value++;
    rateLimit.value = true;
    window.scrollTo({
      top: 0,
      left: 0,
      behavior: "smooth",
    });

    setTimeout(() => {
      rateLimit.value = false;
    }, 2000);
  }
};

const prevPage = () => {
  if (!rateLimit.value) {
    page.value--;
    rateLimit.value = true;
    window.scrollTo({
      top: 0,
      left: 0,
      behavior: "smooth",
    });
    setTimeout(() => {
      rateLimit.value = false;
    }, 2000);
  }
};
</script>

<template>
  <Transition>
    <div v-if="suggestionAdded" class="suggestion-added">
      <span
        ><i class="fa-solid fa-circle-check fa-xl"></i> Suggestion has
        succesfully been added!</span
      >
    </div>
  </Transition>

  <Transition>
    <div v-if="suggestionExists" class="suggestion-exists">
      <span
        ><i class="fa-solid fa-circle-exclamation fa-xl"></i> Suggestion already
        exists!</span
      >
    </div>
  </Transition>

  <div class="auth-div">
    

    <button class="sign-out" @click="logOut" v-if="store.user != null">Sign out</button>
    <button class="sign-in" @click="logIn" v-if="store.user == null">Sign in</button>

    <div v-if="store.user != null" class="user-greet">
      <span>Logged in as: <span style="color: yellow">{{ store.user.name }}</span></span>
    </div>
  </div>

  <div class="search-movies">
    <form @submit="searchMovies">
      <input v-model="moviesInput" type="text" />
      <button type="submit">Search</button>
    </form>
  </div>
  <div v-if="movies !== null && movies.length" class="movies-container">
    <div
      @click="goToMovie"
      v-for="movie in movies"
      :key="movie.id"
      :id="movie.id"
      class="movie"
      :style="{
        backgroundImage: movie.poster_path
          ? `url(https://image.tmdb.org/t/p/original${movie.poster_path})`
          : '',
      }"
    >
      <div class="score">
        <span>{{
          movie.vote_average ? movie.vote_average.toFixed(1) : "N/A"
        }}</span>
      </div>

      <div
        @click="addSuggestion"
        v-if="store.user != null"
        class="add-suggestion-btn"
        :data-title="movie.title"
        :data-release_date="movie.release_date"
        :data-poster_path="movie.poster_path ? movie.poster_path : 'N/A'"
        :data-vote_average="movie.vote_average"
        :data-id="movie.id"
        :data-overview="movie.overview"
      >
        Suggest
      </div>

      <div class="title">
        <span>
          {{
            movie.title
              ? `${movie.title} (${
                  movie.release_date
                    ? movie.release_date.split("-", 1)[0]
                    : "N/A"
                })`
              : movie.name
          }}
        </span>
      </div>
    </div>
  </div>
  <div v-else-if="movies !== null && !movies.length">
    <p>No movies found.</p>
  </div>
  <div v-else>
    <p>Loading...</p>
  </div>

  <div v-if="pages" class="pagination">
    <p>{{ `Page ${page}/${pages}` }}</p>
    <button @click="prevPage" v-if="page > 1">Previous</button>
    <button @click="nextPage" v-if="page < pages">Next</button>
  </div>
</template>

<style lang="scss" scoped>
.v-enter-active,
.v-leave-active {
  transition: 0.3s ease-out;
}

.v-enter-from,
.v-leave-to {
  opacity: 0%;
  margin-right: -300px;
}

.auth-div {
  display: flex;
  margin-bottom: 30px;
  margin-top: -30px;
  flex-direction: column;
  align-items: center;

  & button.sign-out {
    width: 90px;
    padding: 10px 0px;
    margin-bottom: 20px;
        border: none;
    background: none;
    color: white;
    border: 2px solid white;
    border-radius: 5px;
    transition: .2s ease-out;


    &:hover {
            cursor: pointer;
      color: rgb(27, 27, 27);
      background-color: white;

    }
  }

  & button.sign-in {
    padding: 10px 20px;
    border: none;
    background: none;
    color: white;
    border: 2px solid white;
    border-radius: 5px;
    transition: .2s ease-out;

    &:hover {
      cursor: pointer;
      color: rgb(27, 27, 27);
      background-color: white;
    }
  }
}

.suggestion-added {
  background-color: rgb(0, 121, 6);
  position: fixed;
  top: 30px;
  right: 30px;
  padding: 20px;
  border-radius: 10px;
  z-index: 99;
}

.suggestion-exists {
  background-color: rgb(180, 0, 0);
  position: fixed;
  top: 30px;
  right: 30px;
  padding: 20px;
  border-radius: 10px;
  z-index: 99;
}
.movies-container {
  max-width: calc(250px * 4 + 20px * 8);
  display: flex;
  flex-wrap: wrap;
  margin: auto;
  justify-content: center;
}

.movies-container img {
  width: 100%;
  border-radius: 10px;
}

.movie {
  width: 250px;
  height: 370px;
  border-radius: 8px;
  margin: 20px;
  background: url("/image_not_av.jpg") no-repeat center center;
  -webkit-background-size: cover;
  -moz-background-size: cover;
  -o-background-size: cover;
  background-size: cover;
  transition: 0.2s ease-out;
  cursor: pointer;

  & .add-suggestion-btn {
    width: 80px;
    height: 50px;

    background-color: rgb(27, 27, 27);
    color: white;
    font-size: 15px;
    display: flex;
    justify-content: center;
    align-items: center;
    text-align: center;
    border-bottom-right-radius: 8px;
    border-top-right-radius: 8px;
    border: 2px solid white;
    border-left: none;
    opacity: 0;
    transition: 0.2s ease-out;

    &:hover {
      border-color: yellow;
      width: 100px;
    }
  }

  & .title {
    text-align: center;
    margin-top: 200px;
    transition: 0.2s ease-out;
    position: relative;
    z-index: -1;
    opacity: 0%;
  }

  &:hover {
    margin-top: 0px;

    .score {
      margin-top: 10px;
      opacity: 100%;
    }

    .title {
      margin-top: 280px;
      opacity: 100%;
    }

    .add-suggestion-btn {
      opacity: 100%;
    }
  }
}

.movie .score {
  width: 35px;
  height: 35px;
  background-color: rgb(27, 27, 27);
  display: flex;
  justify-content: center;
  align-items: center;
  margin-left: auto;
  margin-right: 10px;
  margin-top: 20px;
  border-radius: 100px;
  transition: 0.1s ease-out;
  opacity: 0%;
  border: 2px solid yellow;
}

.movie .score span {
  color: white;
}

.search-movies {
  margin-bottom: 40px;
}
.search-movies form {
  transition: 0.2s ease-out;
  border: 2px solid rgba(255, 255, 255, 0.562);
  border-radius: 100px;
  display: flex;
  justify-content: space-between;
  align-items: center;
  height: 40px;
  input {
    color: white;
    width: 100%;
    height: 100%;
    border: none;
    appearance: none;
    outline: none;
    padding: 10px 15px;
    background-color: rgba(255, 0, 0, 0);
    font-size: large;
  }

  &:hover {
    border: 2px solid rgba(255, 255, 255, 0.692);
  }

  button {
    background: none;
    color: inherit;
    border: none;
    padding: 0;
    font: inherit;
    cursor: pointer;
    outline: inherit;

    background-color: rgba(255, 255, 255, 0.685);
    height: 100%;
    padding: 0px 30px;
    border-radius: 0 100px 100px 0;
    margin-left: auto;
    color: black;
    transition: 0.2s ease-out;

    &:hover {
      background-color: white;
    }
  }
}

.pagination {
  text-align: center;

  p {
    margin-bottom: 0;
  }

  & button {
    background: none;
    color: inherit;
    border: none;
    padding: 0;
    font: inherit;
    cursor: pointer;
    outline: inherit;

    border: 2px solid white;
    width: 90px;
    height: 40px;
    margin: 10px;
    border-radius: 100px;
    transition: 0.2s ease-out;

    &:hover {
      background-color: white;
      color: black;
    }
  }
}

@media (max-width: 716px) {
  .movie {
    width: 350px;
    height: 510px;
    margin-bottom: 40px;
    & .score {
      opacity: 100%;
      margin-right: 20px;
      margin-top: 20px;
    }

    & .title {
      opacity: 100%;
      z-index: 0;
      margin-top: 460px;
    }

    &:hover {
      margin-top: 20px;

      .score {
        opacity: 100%;
        margin-right: 20px;
        margin-top: 20px;
      }

      .title {
        opacity: 100%;
        z-index: 0;
        margin-top: 460px;
      }
    }
  }

}
</style>
