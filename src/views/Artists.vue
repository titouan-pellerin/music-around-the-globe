<template>
  <div class="artists style-dark">
    <header>
      <h1>Quels sont vos artistes préférés ?</h1>
      <p class="caption">
        Sélectionnez les artistes que vous écoutez pour affiner la recherche.
      </p>
      <div class="input-container">
        <input
          class="search-input"
          placeholder="Rechercher un artiste"
          type="text"
          v-model="query"
        />
        <div v-if="searchedArtists[0]" class="search-results">
          <div
            v-for="artist in searchedArtists"
            :key="artist.id"
            class="artist-result"
            v-on:click="addArtistFromSearch(artist)"
          >
            <img :src="artist.images[0].url" />
            <span>{{ artist.name }}</span>
          </div>
        </div>
      </div>
    </header>
    <main class="artists-list">
      <Artist
        v-for="artist in artists"
        :key="artist.id"
        :id="artist.id"
        :image="artist.images[2].url"
        :name="artist.name"
      ></Artist>
    </main>
    <footer v-if="selectedArtists[0]" class="artists-footer">
      <router-link @click="saveSelectedArtists()" to="explore" class="btn launch-btn">
        Lancer le tour du monde
        <span class="selection-length">{{ selectedArtists.length }}</span>
      </router-link>
    </footer>
  </div>
</template>

<script>
// @ is an alias to /src
import Artist from "@/components/Artist.vue";

const API_URL = "http://localhost:8080/artists";

export default {
  name: "Home",
  components: {
    Artist,
  },
  data() {
    return {
      artists: [],
      selectedArtists: [],
      query: "",
      searchedArtists: [],
      audio: null,
    };
  },
  watch: {
    query(query) {
      let inputContainer = document.querySelector(".input-container");
      if (query.trim().length > 0)
        fetch(API_URL + "/search/" + query).then((response) =>
          response.json().then((artists) => {
            inputContainer.classList.add("open");
            console.log(artists);
            this.searchedArtists = artists.items;
          })
        );
      else inputContainer.classList.remove("open");
    },
  },
  methods: {
    addSelectedArtist(id) {
      this.selectedArtists.push(id);
    },
    removeSelectedArtist(id) {
      this.togglePreview();
      this.selectedArtists.splice(this.selectedArtists.indexOf(id), 1);
    },
    fetchArtists() {
      fetch(API_URL).then((response) =>
        response.json().then((artists) => {
          this.artists = artists;
        })
      );
    },
    addArtistFromSearch(artist) {
      if (!document.getElementById(artist.id)) {
        document.querySelector(".input-container").classList.remove("open");
        this.artists.unshift(artist);
      }
    },
    togglePreview(preview) {
      if (this.audio) this.audio.pause();
      if (preview) {
        this.audio = new Audio(preview.preview_url);
        this.audio.volume = 0.2;
        this.audio.play();
      }
    },
    saveSelectedArtists(){
      localStorage.selectedArtists = JSON.stringify(this.selectedArtists);
    }
  },
  mounted() {
    this.fetchArtists();
  },
};
</script>

<style>
.artists {
  margin-top: 20vh;
  margin-bottom: 100px;
  min-width: 100vh;
  width: 100%;
  display: flex;
  align-items: center;
  justify-content: center;
  flex-direction: column;
}

.artists header {
  display: flex;
  align-items: center;
  justify-content: center;
  flex-direction: column;
  margin-bottom: 30px;
}

.artists h1 {
  font-size: var(--font-size-50);
  margin: 0;
}

.artists p {
  font-size: var(--font-size-20);
  font-weight: 200;
  margin: 0;
}

.input-container {
  margin-top: 20px;
  position: relative;
  background-color: #333333;
  border-radius: 2em;
}

.input-container.open {
  border-radius: 2em 2em 0 0;
}

.search-input {
  font-size: var(--font-size-15);
  text-align: center;
  padding: 12px 30px;
  border: none;
  outline: none;
  background-color: transparent;
  color: #fff;
}

.search-results {
  opacity: 0;
  visibility: hidden;
  position: absolute;
  padding: 0 10px 5px 10px;
  width: 100%;
  box-sizing: border-box;
  z-index: 5;
  background-color: #333333;
  border-radius: 0 0 2em 2em;
  text-align: left;
  cursor: pointer;
}

.open .search-results {
  opacity: 1;
  visibility: visible;
}

.artist-result {
  padding: 5px;
  box-sizing: border-box;
  display: flex;
  align-items: center;
  margin: 10px 0;
  border-radius: 0.5em;
  background-color: #333333;
  transition: background-color ease-in-out 0.1s;
}

.artist-result:hover {
  background-color: #444444;
}

.artist-result img {
  width: 30px;
  height: 30px;
  object-fit: cover;
  margin-right: 10px;
  border-radius: 50%;
}

.artist-result span {
  font-size: var(--font-size-15);
  text-overflow: ellipsis;
  white-space: nowrap;
  overflow: hidden;
}

.artists-list {
  display: flex;
  flex-wrap: wrap;
  justify-content: center;
  max-width: 1800px;

}

.artists-footer {
  position: fixed;
  bottom: 20px;
  display: flex;
  justify-content: center;
}

.launch-btn {
  position: relative;
}

.selection-length {
  position: absolute;
  width: 20px;
  height: 20px;
  top: -10px;
  right: -10px;
  border-radius: 50%;
  background-color: #fff;
  padding: 5px;
  transition: background-color ease-in-out 0.3s;
}

.btn:hover .selection-length {
  background-color: var(--accent-color);
}
</style>
