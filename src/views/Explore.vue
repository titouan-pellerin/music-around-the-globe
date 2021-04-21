<template>
  <div class="explore">
    <div class="spin-container loading"></div>
    <canvas class="loading" id="globeCanvas"></canvas>
    <div class="artist-container">
      <div class="controls-container">
        <a href="#" class="artist-control close-control">
          <svg
            width="14"
            height="14"
            viewBox="0 0 14 14"
            fill="none"
            xmlns="http://www.w3.org/2000/svg"
          >
            <path
              d="M8.48514 7.00022L12.9723 2.51308L13.8976 1.58774C14.0341 1.45123 14.0341 1.22941 13.8976 1.09291L12.9075 0.102816C12.771 -0.0336885 12.5492 -0.0336885 12.4127 0.102816L7.00022 5.5153L1.58774 0.102378C1.45123 -0.0341261 1.22941 -0.0341261 1.09291 0.102378L0.102378 1.09247C-0.0341261 1.22898 -0.0341261 1.4508 0.102378 1.5873L5.5153 7.00022L0.102378 12.4127C-0.0341261 12.5492 -0.0341261 12.771 0.102378 12.9075L1.09247 13.8976C1.22898 14.0341 1.4508 14.0341 1.5873 13.8976L7.00022 8.48514L11.4874 12.9723L12.4127 13.8976C12.5492 14.0341 12.771 14.0341 12.9075 13.8976L13.8976 12.9075C14.0341 12.771 14.0341 12.5492 13.8976 12.4127L8.48514 7.00022Z"
              fill="white"
            />
          </svg>
          <span>Fermer</span>
        </a>
        <a href="#" class="artist-control next-control">
          <span>Suivant</span>
          <svg
            width="8"
            height="14"
            viewBox="0 0 8 14"
            fill="none"
            xmlns="http://www.w3.org/2000/svg"
          >
            <path
              d="M0.772004 0.190662L0.153285 0.809349C0.00684766 0.955787 0.00684766 1.19322 0.153285 1.33969L5.80013 7.00001L0.153285 12.6603C0.00684766 12.8068 0.00684766 13.0442 0.153285 13.1907L0.772004 13.8094C0.918441 13.9558 1.15588 13.9558 1.30235 13.8094L7.84653 7.26519C7.99297 7.11875 7.99297 6.88132 7.84653 6.73485L1.30235 0.190662C1.15588 0.0441934 0.918441 0.0441934 0.772004 0.190662Z"
              fill="white"
            />
          </svg>
        </a>
      </div>
      <div v-if="selectedArtist.name" class="artist-content">
        <header class="artist-header">
          <img
            :src="
              selectedArtist.images[0]
                ? selectedArtist.images[2]
                  ? selectedArtist.images[2].url
                  : selectedArtist.images[0].url
                : '/artist-img.svg'
            "
          />
          <h1>{{ selectedArtist.name }}</h1>
        </header>
      </div>
    </div>
  </div>
</template>


<style>
body {
  overflow: hidden;
}

.explore {
  display: flex;
}

canvas {
  cursor: grab;
  margin-left: 0;
  transition: all ease-in-out 2s;
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

canvas.reduced {
  margin-left: -40%;
  cursor: pointer;
}

.spin-container {
  width: 100vw;
  height: 100vh;
  display: none;
}

.spin-container.loading {
  display: block;
}

.artist-container {
  width: 40%;
  height: 100vh;
  z-index: 100;
  background: #2e2e2e;
  transition: all ease-in-out 2s;
  position: relative;
  margin-right: -40%;
}

.artist-container.expanded {
  margin-right: 0;
}

.controls-container {
  margin: 20px;
  display: flex;
  justify-content: space-between;
}

.artist-control {
  display: flex;
  align-items: center;
  color: #fff;
  transition-duration: 0.3s;
  text-decoration: none;
}

.artist-control path,
.artist-control svg {
  transition-duration: 0.3s;
}

.close-control:hover {
  color: #ff4141;
}

.close-control:hover path {
  fill: #ff4141;
}

.close-control:hover svg {
  border-color: #ff4141;
}

.close-control {
  left: 20px;
}

.close-control span {
  margin-left: 15px;
}

.expanded .next-control {
  right: 20px;
}

.next-control span {
  margin-right: 15px;
}

.next-control:hover {
  color: var(--accent-color);
}

.next-control:hover path {
  fill: var(--accent-color);
}

.next-control:hover svg {
  border-color: var(--accent-color);
}

.artist-control span {
  font-size: var(--font-size-15);
}

.artist-control svg {
  width: 15px;
  height: 15px;
  border-radius: 50%;
  border: solid 2px #fff;
  padding: 10px;
}

.artist-content{
  margin: 20px;
}

.artist-header{
  display: flex;
  align-items: center;
}
</style>

<script>
import * as THREE from "three";
import * as TWEEN from "@tweenjs/tween.js";
import * as lottie from "lottie-web";
import { OrbitControls } from "@/assets/OrbitControls";

let earth;

export default {
  name: "Explore",
  data() {
    return {
      selectedArtist: {},
    };
  },
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
      container: document.querySelector(".spin-container"),
      renderer: "svg",
      loop: true,
      autoplay: true,
      path: "/data.json",
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
      this.add(mesh);
    }
    Earth.prototype = Object.create(THREE.Object3D.prototype);
    Earth.prototype.createMarker = function (artist) {
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
      controls.minDistance = 1.7;
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

      onResize();
    }

    function animate() {
      requestAnimationFrame(animate);
      controls.update();
      render();
      TWEEN.update();
    }

    function onResize(e, width, height) {
      width = width != null ? width : window.innerWidth;
      height = height != null ? height : window.innerHeight;
      console.log(width);
      console.log(height);

      camera.aspect = width / height;
      camera.updateProjectionMatrix();
      renderer.setSize(width, height);
    }

    function onDocumentMouseMove(event) {
      event.preventDefault();
      mouse.x = (event.clientX / window.innerWidth) * 2 - 1;
      mouse.y = -(event.clientY / window.innerHeight) * 2 + 1;
    }

    let self = this;
    function onDocumentMouseDown(event) {
      event.preventDefault();
      document.removeEventListener("mousemove", onDocumentMouseMove);
      globeCanvas.classList.add("reduced");
      document.querySelector(".artist-container").classList.add("expanded");
      controls.enabled = false;

      self.selectedArtist = INTERSECTED.parent.userData.artist;

      new TWEEN.Tween({
        x: camera.position.x,
        y: camera.position.y,
        z: camera.position.z,
      })
        .to(
          {
            x: INTERSECTED.parent.position.x - 0.5,
            y: INTERSECTED.parent.position.y + 0.5,
            z: INTERSECTED.parent.position.z + 0.5,
          },
          2000
        )
        .easing(TWEEN.Easing.Quadratic.InOut)
        .onUpdate((d) => {
          camera.position.set(d.x, d.y, d.z);
        })
        .start();

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

    function loadCloudsMaterial() {
      const textures = {
        map: "/clouds.png",
      };
      let params = {};

      const promises = Object.keys(textures).map((key) => {
        return loadTexture(textures[key]).then((texture) => {
          params[key] = texture;
        });
      });

      params.transparent = true;

      return Promise.all(promises).then(() => {
        return new THREE.MeshPhongMaterial(params);
      });
    }

    function loadMaterial() {
      const textures = {
        map: "/earth.jpg",
        specularMap: "/specular.png",
        displacementMap: "/bump.jpg",
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
      return Promise.all([loadCloudsMaterial(), loadMaterial()]).then(
        (result) => {
          document.querySelector(".spin-container").classList.remove("loading");
          document.querySelector("canvas").classList.remove("loading");
          let earth = new Earth(
            new THREE.Mesh(
              new THREE.SphereBufferGeometry(1, 100, 100),
              result[1]
            )
          );
          let clouds = new THREE.Mesh(
            new THREE.SphereGeometry(1.03, 100, 100),
            result[0]
          );

          earth.add(clouds);

          for (let artist of JSON.parse(localStorage.exploreArtists))
            if (artist.location)
              if (artist.location.latLng) earth.createMarker(artist);

          return earth;
        }
      );
    }
  },
};
</script>
