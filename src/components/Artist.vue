<template>
  <transition appear name="slide-fade">
    <label :data-related="relatedId" :for="id" href="#" class="artist">
      <input class="artist-check" type="checkbox" :id="id" v-model="checked" />
      <div class="artist-img-container">
        <img class="artist-img" :src="image" />
        <div class="img-layer">
          <div class="img-overlay"></div>
          <svg
            aria-hidden="true"
            focusable="false"
            role="img"
            xmlns="http://www.w3.org/2000/svg"
            viewBox="0 0 512 512"
            class="check-svg"
          >
            <path
              fill="currentColor"
              d="M173.898 439.404l-166.4-166.4c-9.997-9.997-9.997-26.206 0-36.204l36.203-36.204c9.997-9.998 26.207-9.998 36.204 0L192 312.69 432.095 72.596c9.997-9.997 26.207-9.997 36.204 0l36.203 36.204c9.997 9.997 9.997 26.206 0 36.204l-294.4 294.401c-9.998 9.997-26.207 9.997-36.204-.001z"
              class=""
            ></path>
          </svg>
        </div>
      </div>
      <span class="artist-name">{{ name }}</span>
    </label>
  </transition>
  <Artist
    v-for="artist in relatedArtists"
    :key="artist.id"
    :id="artist.id"
    :image="artist.images[0] ? artist.images[0].url : '/artist-img.svg'"
    :name="artist.name"
    :relatedId="id"
  >
  </Artist>
</template>

<script>
import { API_URL } from "@/assets/variables.js";

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
        this.fetchPreview(this.id);
        parent.exploreArtists.forEach((artist) => {
          if (artist.id == this.id)
            parent.exploreArtists.splice(
              parent.exploreArtists.indexOf(artist),
              1
            );
        });
        parent.counter++;
      } else {
        parent.removeExploreArtists(this.relatedArtists);
        parent.counter--;
      }
    },
  },
  methods: {
    showRelatedArtists(id) {
      if (this.relatedArtists.length == 0) {
        fetch(API_URL + "artist/related/" + id).then((response) => {
          response.json().then((artists) => {
            artists.forEach((artist) => {
              if (!document.getElementById(artist.id)) {
                this.relatedArtists.push(artist);
                this.fetchLocation(this.relatedArtists.indexOf(artist));
              }
            });
            this.getParent("Home").addExploreArtists(this.relatedArtists);
          });
        });
      } else this.getParent("Home").addExploreArtists(this.relatedArtists);
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
          if (preview.preview_url) {
            this.getParent("Home").togglePreview(preview);
          }
        });
      });
    },
    async fetchLocation(index) {
      let parent = this.getParent("Home");
      if (parent.locationLoading) await this.wait(1000);
      parent.locationLoading = true;
      console.log(this.relatedArtists[index]);
      fetch(
        API_URL + "artist/location/" + this.relatedArtists[index].name
      ).then((response) => {
        response.json().then((location) => {
          this.relatedArtists[index].location = location;
          parent.locationLoading = false;
        });
      });
    },
    wait(milliseconds) {
      return new Promise((resolve) => setTimeout(resolve, milliseconds));
    },
  },
  mounted() {
    this.audio = this.getParent("Home").audio;
  },
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

.img-layer .check-svg {
  position: absolute;
  color: #fff;
  opacity: 0;
  width: 40px;
  height: 40px;
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

.artist-check:checked + .artist-img-container .check-svg {
  opacity: 1;
}

.artist-check:checked ~ .artist-name {
  color: var(--accent-color);
}
</style>
