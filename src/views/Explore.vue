<template>
  <div class="explore">
    <div class="spin-container loading">
    </div>
    <canvas class="loading" id="globeCanvas"></canvas>
    <div class="artist-container"></div>
  </div>
</template>

<script>
import * as THREE from "three";
import * as lottie from "lottie-web";
import { OrbitControls } from "three/examples/jsm/controls/OrbitControls";

//const API_URL = "http://localhost:8080/artists/explore";

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
    let previousDistance = 3;
    const mouse = new THREE.Vector2();

    /*Rediriger si il n'y a aucun artistes sélectionnés */
    /*if (!localStorage.selectedArtists) this.$router.push("/");
    else {
      this.fetchArtists(JSON.parse(localStorage.selectedArtists));
      //localStorage.removeItem('selectedArtists');
    }*/

    lottie.loadAnimation({
      container: document.querySelector('.spin-container'), // the dom element that will contain the animation
      renderer: "svg",
      loop: true,
      autoplay: true,
      path: "/data.json", // the path to the animation json
    });

    function Marker(textureUrl) {
      THREE.Object3D.call(this);

      var texture = new THREE.TextureLoader().load(textureUrl);
      texture.flipY = false;
      texture.wrapS = THREE.RepeatWrapping;
      texture.repeat.x = -1;
      var material = new THREE.MeshStandardMaterial({
        map: texture,
      });

      var sphere = new THREE.Mesh(
        new THREE.CylinderGeometry(0.03, 0.03, 0, 100),
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

      //console.log(mesh);
      this.add(mesh);
    }
    Earth.prototype = Object.create(THREE.Object3D.prototype);
    Earth.prototype.createMarker = function (artist) {
      //console.log(artist);
      let url;
      if (artist.images[0]) url = artist.images[2].url;
      else url = "/artist-img.svg";

      var marker = new Marker(url);

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

      scene.add(camera);

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

    function onDocumentMouseDown(event) {
      event.preventDefault();
      document.removeEventListener("mousemove", onDocumentMouseMove);
      console.log(INTERSECTED.parent.userData.artist);
      controls.enabled = false;
      document.removeEventListener("mousedown", onDocumentMouseDown);
    }

    function render() {
      let markers = earth.children;

      raycaster.setFromCamera(mouse, camera);
      var intersects = raycaster.intersectObjects(markers, true);
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
          document.addEventListener("mousedown", onDocumentMouseDown, false);
          INTERSECTED = intersects[0].object;
          INTERSECTED.parent.userData.scaleUp();
          //console.log(INTERSECTED.parent.userData.artist);
        }
      } else {
        if (INTERSECTED) INTERSECTED.parent.userData.scaleDown();
        globeCanvas.classList.remove("marker");
        controls.autoRotate = true;

        INTERSECTED = null;
        document.removeEventListener("mousedown", onDocumentMouseDown);
      }

      let currentDistance =
        Math.round(camera.position.distanceTo(controls.target) * 1000) / 1000;
      if (!INTERSECTED && previousDistance != currentDistance)
        markers.forEach((marker) => {
          if (marker instanceof Marker)
            marker.userData.scale(currentDistance * 0.3 * 1.33);
        });
      previousDistance = currentDistance;
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

    function loadEarth() {
      return Promise.resolve(loadMaterial()).then((result) => {
        document.querySelector(".spin-container").classList.remove("loading");
        document.querySelector("canvas").classList.remove("loading");
        let earth = new Earth(
          new THREE.Mesh(new THREE.SphereBufferGeometry(1, 100, 100), result)
        );
        for (let artist of JSON.parse(localStorage.exploreArtists))
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
              marker.userData.scale((currentDistance*0.3)*1/3);
            }else{
              console.log(marker);
              marker.userData.scale((currentDistance*0.3)*1.33);
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

.explore {
  display: flex;
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
}

.spin-container.loading{
  display: block;
}

.selected .artist-container {
  width: 30vw;
  height: 100vh;
}

.selected canvas {
  width: 70vw;
}
</style>
