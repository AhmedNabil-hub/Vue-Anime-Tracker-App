<template>
  <main>
    <h1>My Anime Tracker App</h1>

    <form @submit.prevent="searchAnime">
      <input
        type="text"
        placeholder="Search for an anime..."
        v-model="query"
        @input="handleInput"
      />

      <button type="submit">Search</button>
    </form>

    <div v-if="searchResults.length > 0" class="results">
      <div class="result" v-for="anime in searchResults" :key="anime.id">
        <img :src="anime.images.jpg.image_url" />
        <div class="details">
          <h3>{{ anime.title }}</h3>
          <p :title="anime.synopsis" v-if="anime.synopsis">
            {{ anime.synopsis.slice(0, 120) }}...
          </p>
          <span class="flex-1"></span>
          <button @click="addAnime(anime)">Add to my anime</button>
        </div>
      </div>
    </div>

    <div class="my-anime" v-if="myAnime.length > 0">
      <h2>My Anime</h2>
      <div v-for="anime in myAnimeAsc" class="anime" :key="anime.id">
        <img :src="anime.image" />
        <h3>{{ anime.title }}</h3>
        <span class="flex-1"></span>
        <span class="episodes">
          {{ anime.watchedEpisodes }} / {{ anime.totalEpisodes }}
        </span>
        <button
          v-if="anime.totalEpisodes !== anime.watchedEpisodes"
          @click="increaseWatch(anime)"
        >
          +
        </button>
        <button v-if="anime.watchedEpisodes > 0" @click="decreaseWatch(anime)">
          -
        </button>
      </div>
    </div>
  </main>
</template>

<script setup>
import { computed, onMounted, ref } from "vue";

const query = ref("");
const myAnime = ref([]);
const searchResults = ref([]);

const myAnimeAsc = computed(() => {
  return myAnime.value.sort((a, b) => {
    return a.title.localeCompare(b.title);
  });
});

function searchAnime() {
  const url = `https://api.jikan.moe/v4/anime?q=${query.value}`;

  fetch(url)
    .then((res) => res.json())
    .then((res) => {
      searchResults.value = res.data;
    });
}

function handleInput(event) {
  if (!event.target.value) {
    searchResults.value = [];
  }
}

function addAnime(anime) {
  searchResults.value = [];
  query.value = "";

  myAnime.value.push({
    id: anime.id,
    title: anime.title,
    image: anime.images.jpg.image_url,
    totalEpisodes: anime.episodes,
    watchedEpisodes: 0,
  });

  localStorage.setItem("myAnime", JSON.stringify(myAnime.value));
}

function increaseWatch(anime) {
  anime.watchedEpisodes++;
  localStorage.setItem("myAnime", JSON.stringify(myAnime.value));
}

function decreaseWatch(anime) {
  anime.watchedEpisodes--;
  localStorage.setItem("myAnime", JSON.stringify(myAnime.value));
}

onMounted(() => {
  myAnime.value = JSON.parse(localStorage.getItem("myAnime")) || [];
});
</script>

<style scoped>
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
  font-family: "Fira Sans", sans-serif;
}
body {
  background-color: #eee;
}
main {
  margin: 0 auto;
  max-width: 768px;
  padding: 1.5rem;
}
h1 {
  text-align: center;
  margin-bottom: 1.5rem;
}
form {
  display: flex;
  max-width: 480px;
  margin: 0 auto 1.5rem;
}
form input {
  appearance: none;
  outline: none;
  border: none;
  background: white;
  display: block;
  color: #888;
  font-size: 1.125rem;
  padding: 0.5rem 1rem;
  width: 100%;
}
.button {
  appearance: none;
  outline: none;
  border: none;
  background: none;
  cursor: pointer;
  display: block;
  padding: 0.5rem 1rem;
  background-image: linear-gradient(to right, deeppink 50%, darkviolet 50%);
  background-size: 200%;
  color: white;
  font-size: 1.125rem;
  font-weight: bold;
  text-transform: uppercase;
  transition: 0.4s;
}
.button:hover {
  background-position: right;
}
.results {
  background-color: #fff;
  border-radius: 0.5rem;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
  max-height: 480px;
  overflow-y: scroll;
  margin-bottom: 1.5rem;
}
.result {
  display: flex;
  margin: 1rem;
  padding: 1rem;
  border: 1px solid #ccc;
  border-radius: 5px;
  transition: 0.4s;
}
.result img {
  width: 100px;
  border-radius: 1rem;
  margin-right: 1rem;
}
.details {
  flex: 1 1 0%;
  display: flex;
  align-items: flex-start;
  flex-direction: column;
}
.details h3 {
  font-size: 1.25rem;
  margin-bottom: 0.5rem;
}
.details p {
  font-size: 0.875rem;
  margin-bottom: 1rem;
}
.details .button {
  margin-left: auto;
}
.flex-1 {
  display: block;
  flex: 1 1 0%;
}
.myanime h2 {
  color: #888;
  font-weight: 400;
  margin-bottom: 1.5rem;
}
.myanime .anime {
  display: flex;
  align-items: center;
  margin-bottom: 1.5rem;
  background-color: #fff;
  padding: 1rem;
  border-radius: 0.5rem;
  box-shadow: 0px 2px 4px rgba(0, 0, 0, 0.1);
}
.anime img {
  width: 72px;
  height: 72px;
  object-fit: cover;
  border-radius: 1rem;
  margin-right: 1rem;
}
.anime h3 {
  color: #888;
  font-size: 1.125rem;
}
.anime .episodes {
  margin-right: 1rem;
  color: #888;
}
.anime .button:first-of-type {
  margin-right: 0.5rem;
}
.anime .button:last-of-type {
  margin-right: 0;
}
</style>
