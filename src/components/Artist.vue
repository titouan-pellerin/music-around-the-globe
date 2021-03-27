<template>
  <label :data-related="relatedId" :for="id" href="#" class="artist">
    <input class="artist-check" type="checkbox" :id="id" v-model="checked" />
    <div class="artist-img-container">
      <img class="artist-img" :src="image" />
      <div class="img-layer">
        <div class="img-overlay"></div>
        <i class="fas fa-check"></i>
      </div>
    </div>
    <span class="artist-name">{{ name }}</span>
  </label>
  <Artist
    v-for="artist in relatedArtists"
    :key="artist.id"
    :id="artist.id"
    :image="artist.images[0].url"
    :name="artist.name"
    :relatedId="id"
  >
  </Artist>
</template>

<script>
const API_URL = "http://localhost:8080/";

export default {
  name: "Artist",
  props: {
    id: String,
    image: String,
    name: String,
    relatedId: String,
  },
  data() {
    return {
      checked: Number,
      relatedArtists: [],
    };
  },
  watch: {
    checked(checked) {
      let parent = this.getParent("Home");
      if (checked) {
        this.showRelatedArtists(this.id);
        parent.addSelectedArtist(this.id);
        this.fetchPreview(this.id);
      } else {
        parent.removeSelectedArtist(this.id);
      }
    },
  },
  methods: {
    showRelatedArtists(id) {
      if (this.relatedArtists.length == 0) {
        let relatedArtists = [];
        fetch(API_URL + "artist/related/" + id).then((response) => {
          response.json().then((artists) => {
            artists.forEach((artist) => {
              if (!document.getElementById(artist.id))
                relatedArtists.push(artist);
            });
            this.relatedArtists = relatedArtists;
          });
        });
      }
    },
    getParent(name) {
      let p = this.$parent;
      while (typeof p !== "undefined") {
        if (p.$options.name == name) {
          return p;
        } else {
          p = p.$parent;
        }
      }
      return false;
    },
    fetchPreview(id) {
      fetch(API_URL + "artist/preview/" + id).then((response) => {
        response.json().then((preview) => {
          if(preview.preview_url){
            this.getParent('Home').togglePreview(preview);
          }
        });
      });
    },
  },
  mounted(){
    this.audio = this.getParent('Home').audio;
  }
};
</script>

<style>
.artist {
  margin: 10px;
  cursor: pointer;
  width: 150px;
  text-overflow: ellipsis;
  white-space: nowrap;
  overflow: hidden;
}

.artist-img-container {
  border-radius: 50%;
  overflow: hidden;
  width: 100%;
  height: 150px;
  position: relative;
  margin-bottom: 10px;
}

.artist-img {
  position: absolute;
  top: 0;
  left: 0;
  object-fit: cover;
  min-width: 100%;
  height: 100%;
}

.artist-name {
  font-size: var(--font-size-15);
  transition: color ease-in-out 0.3s;
}

.artist-check {
  display: none;
}

.img-layer {
  position: absolute;
  width: 100%;
  height: 100%;
  display: flex;
  justify-content: center;
  align-items: center;
}

.img-layer i {
  position: absolute;
  font-size: var(--font-size-30);
  color: #fff;
  opacity: 0;
  transition: opacity ease-in-out 0.3s;
}

.img-overlay {
  width: 100%;
  height: 100%;
  background-color: var(--accent-color);
  opacity: 0;
  transition: opacity ease-in-out 0.3s;
}

.artist-check:checked + .artist-img-container .img-overlay {
  opacity: 0.5;
}

.artist-check:checked + .artist-img-container i {
  opacity: 1;
}

.artist-check:checked ~ .artist-name {
  color: var(--accent-color);
}
</style>
