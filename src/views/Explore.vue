<template>
  <canvas id="globeCanvas"></canvas>
</template>

<script>
import * as THREE from "three";
import { OrbitControls } from "three/examples/jsm/controls/OrbitControls";
//import { GLTFLoader } from "three/examples/jsm/loaders/GLTFLoader.js";

const API_URL = "http://localhost:8080/artists/explore";

let earth;

export default {
  name: "Explore",
  methods: {
    fetchArtists(ids) {
      let promises = [];
      for (let id of ids)
        promises.push(fetch(API_URL + "?id=" + id + "&ids=" + ids));

      Promise.all(promises)
        .then((responses) => {
          return Promise.all(
            responses.map((response) => {
              return response.json().then((data) => {
                console.log(data);
                this.addMarkers(data);
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
          if (artist.location.latLng)
            earth.createMarker(
              artist.location.latLng[1],
              artist.location.latLng[0],
              artist.images[2].url
            );
      }
    },
  },
  mounted() {
    if (!localStorage.selectedArtists) this.$router.push("/");
    else {
      this.fetchArtists(JSON.parse(localStorage.selectedArtists));
      //localStorage.removeItem('selectedArtists');
    }

    const globeCanvas = document.getElementById("globeCanvas");
    var scene, camera, raycaster, renderer, controls;
    let INTERSECTED;
    const mouse = new THREE.Vector2();

    function Marker(textureUrl) {
      THREE.Object3D.call(this);

      var texture = new THREE.TextureLoader().load(textureUrl);
      texture.flipY = false;

      var material = new THREE.MeshPhongMaterial({
        map: texture,
      });

      var sphere = new THREE.Mesh(
        new THREE.CylinderGeometry(0.02, 0.02, 0, 100),
        material
      );

      /*let self = this;
      const loader = new GLTFLoader();
      loader.load("/visufo.glb", function (glft) {
        console.log(glft.scene);
        let sphere = glft.scene;
        sphere.position.y = 0.05;
        sphere.scale.multiplyScalar(1/100);
        self.userData.scale = function () {
          sphere.scale(2, 2);
        };

        self.add(sphere);
      });*/

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

      this.userData.scale = function(size){
        sphere.scale.x = size;
        sphere.scale.z = size;
      }

      this.add(sphere);
    }
    Marker.prototype = Object.create(THREE.Object3D.prototype);

    function Earth(radius, map, bumpMap, specularMap) {
      THREE.Object3D.call(this);

      this.userData.radius = radius;

      var sphere = new THREE.Mesh(
        new THREE.SphereBufferGeometry(radius, 100, 100),
        new THREE.MeshPhongMaterial({
          map: map,
          specularMap: specularMap,
          displacementMap: bumpMap,
          displacementScale: 0.05,
        })
      );

      this.add(sphere);
    }
    Earth.prototype = Object.create(THREE.Object3D.prototype);
    Earth.prototype.createMarker = function (lat, lon, textureUrl) {
      var marker = new Marker(textureUrl);

      var latRad = lat * (Math.PI / 180);
      var lonRad = -lon * (Math.PI / 180);
      var r = this.userData.radius;

      marker.position.set(
        Math.cos(latRad) * Math.cos(lonRad) * r,
        Math.sin(latRad) * r,
        Math.cos(latRad) * Math.sin(lonRad) * r
      );
      marker.rotation.set(0.0, -lonRad, latRad - Math.PI * 0.5);

      this.add(marker);
    };

    init();
    animate();

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
      controls.enableDamping = true; // an animation loop is required when either damping or auto-rotation are enabled
      controls.dampingFactor = 0.05;
      controls.enablePan = false;
      controls.screenSpacePanning = false;
      controls.minDistance = 1.5;
      controls.maxDistance = 3;
      controls.autoRotate = true;
      controls.autoRotateSpeed = 0.5;
      controls.rotateSpeed = 0.5;
      controls.zoomSpeed = 0.3;
      //controls.addEventListener("change", orbitChange);
      var ambient = new THREE.AmbientLight(0x888888, 1);
      scene.add(ambient);

      var directional = new THREE.DirectionalLight(0xcccccc, 0.2);
      directional.position.set(5.0, 3.0, 5.0);
      scene.add(directional);

      var directional2 = new THREE.DirectionalLight(0xcccccc, 0.8);
      camera.add(directional2);

      let map = new THREE.TextureLoader().load("/earth.jpg");
      let bumpMap = new THREE.TextureLoader().load("/bump2.jpg");
      let specularMap = new THREE.TextureLoader().load("/specular.png");
      earth = new Earth(1, map, bumpMap, specularMap);

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
      scene.add(earth);

      controls.update();
      window.addEventListener("resize", onResize);
      document.addEventListener("mousemove", onDocumentMouseMove);
      //controls.addEventListener("change", globeEvent);
      onResize();
    }
    function animate() {
      requestAnimationFrame(animate);
      /*for (let i = 1, l = scene.children[2].children.length; i < l; i++) {
        scene.children[2].children[i].quaternion.copy(camera.quaternion);
      }*/
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
        }
      } else {
        if (INTERSECTED) INTERSECTED.parent.userData.scaleDown();
        globeCanvas.classList.remove("marker");
        controls.autoRotate = true;

        INTERSECTED = null;
      }

      renderer.render(scene, camera);
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
</style>
