<template>
  <div class="spin-container loading">
    <div class="spin"></div>
  </div>
  <canvas class="loading" id="globeCanvas"></canvas>
</template>

<script>
import * as THREE from "three";
import { OrbitControls } from "three/examples/jsm/controls/OrbitControls";

const API_URL = "http://localhost:8080/artists/explore";

let earth;

export default {
  name: "Explore",
  /*methods: {
    fetchArtists(ids) {
      let promises = [];

      for (let id of ids)
        promises.push(fetch(API_URL + "?id=" + id + "&ids=" + ids));

      Promise.allSettled(promises)
        .then((responses) => {
          return Promise.all(
            responses.map((response) => {
              response.value.json().then((artists) => {
                this.addMarkers(artists);
              });
            })
          );
        })
        .catch((err) => {
          console.log(err);
        });
    },
    addMarkers(artists) {
      for (let artist of artists) {
        if (artist.location)
          if (artist.location.latLng) earth.createMarker(artist);
      }
    }
  },*/
  mounted() {
    /**
     * Variables
     */
    const globeCanvas = document.getElementById("globeCanvas");
    var scene, camera, raycaster, renderer, controls;
    let INTERSECTED;
    const mouse = new THREE.Vector2();

    /*Rediriger si il n'y a aucun artistes sélectionnés */
    /*if (!localStorage.selectedArtists) this.$router.push("/");
    else {
      this.fetchArtists(JSON.parse(localStorage.selectedArtists));
      //localStorage.removeItem('selectedArtists');
    }*/

    function Marker(textureUrl) {
      THREE.Object3D.call(this);

      var texture = new THREE.TextureLoader().load(textureUrl);
      texture.flipY = false;

      var material = new THREE.MeshStandardMaterial({
        map: texture,
      });

      var sphere = new THREE.Mesh(
        new THREE.CylinderGeometry(0.02, 0.02, 0, 100),
        material
      );

      sphere.position.y = Math.random() * (0.02 - 0.01) + 0.01 + 0.05;

      this.userData.scaleUp = function () {
        let scale = 1.01;
        let counter = 0;
        let interval = setInterval(function () {
          sphere.scale.multiplyScalar(scale);
          counter++;
          if (counter >= 40) {
            clearInterval(interval);
          }
        }, 0.001);
      };
      this.userData.scaleDown = function () {
        let scale = 0.99;
        let counter = 0;
        let interval = setInterval(function () {
          sphere.scale.multiplyScalar(scale);
          counter++;
          if (counter >= 40) {
            clearInterval(interval);
          }
        }, 0.001);
      };

      this.userData.scale = function (size) {
        sphere.scale.x = size;
        sphere.scale.z = size;
      };

      this.add(sphere);
    }

    Marker.prototype = Object.create(THREE.Object3D.prototype);

    function Earth(mesh) {
      THREE.Object3D.call(this);

      this.userData.radius = 1;

      console.log(mesh);
      this.add(mesh);
    }
    Earth.prototype = Object.create(THREE.Object3D.prototype);
    Earth.prototype.createMarker = function (artist) {
      var marker = new Marker(artist.images[2].url);

      var latRad = artist.location.latLng[1] * (Math.PI / 180);
      var lonRad = -artist.location.latLng[0] * (Math.PI / 180);
      var r = this.userData.radius;

      marker.position.set(
        Math.cos(latRad) * Math.cos(lonRad) * r,
        Math.sin(latRad) * r,
        Math.cos(latRad) * Math.sin(lonRad) * r
      );
      marker.rotation.set(0.0, -lonRad, latRad - Math.PI * 0.5);
      marker.userData.artist = artist;
      this.add(marker);
    };

    init();

    function init() {
      scene = new THREE.Scene();
      scene.background = new THREE.Color(0x131313);
      camera = new THREE.PerspectiveCamera(45, 4 / 3, 0.1, 100);
      camera.position.set(0.0, 1.5, 3.0);
      raycaster = new THREE.Raycaster();
      renderer = new THREE.WebGLRenderer({
        canvas: globeCanvas,
        antialias: true,
      });
      renderer.setSize(window.innerWidth, window.innerHeight);

      controls = new OrbitControls(camera, renderer.domElement);
      controls.enableDamping = true;
      controls.dampingFactor = 0.05;
      controls.enablePan = false;
      controls.screenSpacePanning = false;
      controls.minDistance = 1.5;
      controls.maxDistance = 3;
      controls.autoRotate = true;
      controls.autoRotateSpeed = 0.5;
      controls.rotateSpeed = 0.5;
      controls.zoomSpeed = 0.3;

      scene.add(new THREE.AmbientLight(0x888888, 1));
      let directional = new THREE.DirectionalLight(0xcccccc, 0.2);
      directional.position.set(5.0, 3.0, 5.0);
      scene.add(directional);
      camera.add(new THREE.DirectionalLight(0xcccccc, 0.8));

      loadEarth().then((earthMesh) => {
        scene.add(earthMesh);
        earth = earthMesh;
        animate();
      });
      //earth = new Earth(mesh);

      /*if (localStorage.globeArtists) {
        let globeArtists = JSON.parse(localStorage.globeArtists);
        for (let globeArtist of globeArtists) {
          earth.createMarker(
            globeArtist.location.latLng[0],
            globeArtist.location.latLng[1]
          );
        }
      }*/
      scene.add(camera);
      //scene.add(earth);

      controls.update();
      window.addEventListener("resize", onResize);
      document.addEventListener("mousemove", onDocumentMouseMove);

      //controls.addEventListener("change", globeEvent);

      onResize();
    }

    function animate() {
      requestAnimationFrame(animate);
      controls.update();
      render();
    }

    function onResize() {
      camera.aspect = window.innerWidth / window.innerHeight;
      camera.updateProjectionMatrix();
      renderer.setSize(window.innerWidth, window.innerHeight);
    }

    function onDocumentMouseMove(event) {
      event.preventDefault();
      mouse.x = (event.clientX / window.innerWidth) * 2 - 1;
      mouse.y = -(event.clientY / window.innerHeight) * 2 + 1;
    }

    function render() {
      raycaster.setFromCamera(mouse, camera);
      var intersects = raycaster.intersectObjects(earth.children, true);
      if (
        intersects.length > 0 &&
        intersects[0].object.parent instanceof Marker
      ) {
        if (INTERSECTED != intersects[0].object) {
          if (INTERSECTED) {
            INTERSECTED.parent.userData.scaleDown();
            globeCanvas.classList.remove("marker");
          }
          globeCanvas.classList.add("marker");
          controls.autoRotate = false;
          INTERSECTED = intersects[0].object;
          console.log(INTERSECTED.parent);
          INTERSECTED.parent.userData.scaleUp();
          console.log(INTERSECTED.parent.userData.artist);
        }
      } else {
        if (INTERSECTED) INTERSECTED.parent.userData.scaleDown();
        globeCanvas.classList.remove("marker");
        controls.autoRotate = true;
        INTERSECTED = null;
      }
      renderer.render(scene, camera);
    }

    function loadTexture(url) {
      return new Promise((resolve) => {
        new THREE.TextureLoader().load(url, resolve);
      });
    }

    function loadMaterial() {
      const textures = {
        map: "/earth.jpg",
        specularMap: "/specular.png",
        displacementMap: "/bump2.jpg",
      };
      let params = {};

      const promises = Object.keys(textures).map((key) => {
        return loadTexture(textures[key]).then((texture) => {
          params[key] = texture;
        });
      });

      params.displacementScale = 0.05;

      return Promise.all(promises).then(() => {
        return new THREE.MeshPhongMaterial(params);
      });
    }

    function loadMarkers() {
      let ids = JSON.parse(localStorage.selectedArtists);
      let promises = [];

      for (let id of ids)
        promises.push(fetch(API_URL + "?id=" + id + "&ids=" + ids));

      return Promise.allSettled(promises)
        .then((responses) => {
          return Promise.all(
            responses.map((response) => {
              return response.value.json().then((artists) => {
                console.log(artists);
                return artists;
              });
            })
          );
        })
        .catch((err) => {
          console.log(err);
        });
    }

    function loadEarth() {
      const promises = [loadMaterial(), loadMarkers()];
      return Promise.all(promises).then((result) => {
        document.querySelector(".spin-container").classList.remove("loading");
        document.querySelector("canvas").classList.remove("loading");
        let earth = new Earth(
          new THREE.Mesh(new THREE.SphereBufferGeometry(1, 100, 100), result[0])
        );
        for (let artists of result[1])
          for (let artist of artists)
            if (artist.location)
              if (artist.location.latLng) earth.createMarker(artist);

        return earth;
      });
    }

    /*let previousDistance = 3;
    function globeEvent(){
      let markers = earth.children;
      let currentDistance = Math.round(camera.position.distanceTo(controls.target) * 1000) / 1000;
      if(currentDistance!=previousDistance)
        console.log(currentDistance);
        console.log(previousDistance);
        markers.forEach(marker => {
          if(marker instanceof Marker){
            if(previousDistance > currentDistance){
              console.log(marker);
              marker.userData.scale((currentDistance*0.5)*1/3);
            }else{
              console.log(marker);
              marker.userData.scale((currentDistance*0.5)*1.33);
            }
          }
        });
      previousDistance = currentDistance;
      

    }*/
  },
};
</script>

<style>
body {
  overflow: hidden;
}

canvas {
  cursor: grab;
}

canvas:active {
  cursor: grabbing;
}

canvas.marker {
  cursor: pointer;
}

canvas.loading {
  display: none;
}

.spin-container {
  width: 100vw;
  height: 100vh;
  display: none;
  justify-content: center;
  align-items: center;
}

.loading.spin-container {
  display: flex;
}

.spin:before {
  content: "";
  position: absolute;
  top: 0px;
  left: -25px;
  height: 12px;
  width: 12px;
  border-radius: 12px;
  -webkit-animation: loader10g 3s ease-in-out infinite;
  animation: loader10g 3s ease-in-out infinite;
}

.spin {
  position: relative;
  width: 12px;
  height: 12px;
  border-radius: 12px;
  -webkit-animation: loader10m 3s ease-in-out infinite;
  animation: loader10m 3s ease-in-out infinite;
}

.spin:after {
  content: "";
  position: absolute;
  top: 0px;
  left: 25px;
  height: 10px;
  width: 10px;
  border-radius: 10px;
  -webkit-animation: loader10d 3s ease-in-out infinite;
  animation: loader10d 3s ease-in-out infinite;
}

@-webkit-keyframes loader10g {
  0% {
    background-color: rgba(255, 255, 255, 0.2);
  }
  25% {
    background-color: rgba(255, 255, 255, 1);
  }
  50% {
    background-color: rgba(255, 255, 255, 0.2);
  }
  75% {
    background-color: rgba(255, 255, 255, 0.2);
  }
  100% {
    background-color: rgba(255, 255, 255, 0.2);
  }
}
@keyframes loader10g {
  0% {
    background-color: rgba(255, 255, 255, 0.2);
  }
  25% {
    background-color: rgba(255, 255, 255, 1);
  }
  50% {
    background-color: rgba(255, 255, 255, 0.2);
  }
  75% {
    background-color: rgba(255, 255, 255, 0.2);
  }
  100% {
    background-color: rgba(255, 255, 255, 0.2);
  }
}

@-webkit-keyframes loader10m {
  0% {
    background-color: rgba(255, 255, 255, 0.2);
  }
  25% {
    background-color: rgba(255, 255, 255, 0.2);
  }
  50% {
    background-color: rgba(255, 255, 255, 1);
  }
  75% {
    background-color: rgba(255, 255, 255, 0.2);
  }
  100% {
    background-color: rgba(255, 255, 255, 0.2);
  }
}
@keyframes loader10m {
  0% {
    background-color: rgba(255, 255, 255, 0.2);
  }
  25% {
    background-color: rgba(255, 255, 255, 0.2);
  }
  50% {
    background-color: rgba(255, 255, 255, 1);
  }
  75% {
    background-color: rgba(255, 255, 255, 0.2);
  }
  100% {
    background-color: rgba(255, 255, 255, 0.2);
  }
}

@-webkit-keyframes loader10d {
  0% {
    background-color: rgba(255, 255, 255, 0.2);
  }
  25% {
    background-color: rgba(255, 255, 255, 0.2);
  }
  50% {
    background-color: rgba(255, 255, 255, 0.2);
  }
  75% {
    background-color: rgba(255, 255, 255, 1);
  }
  100% {
    background-color: rgba(255, 255, 255, 0.2);
  }
}
@keyframes loader10d {
  0% {
    background-color: rgba(255, 255, 255, 0.2);
  }
  25% {
    background-color: rgba(255, 255, 255, 0.2);
  }
  50% {
    background-color: rgba(255, 255, 255, 0.2);
  }
  75% {
    background-color: rgba(255, 255, 255, 1);
  }
  100% {
    background-color: rgba(255, 255, 255, 0.2);
  }
}
</style>
