<template>
  <div id="webgl"></div>
</template>

<script>
import * as THREE from "three";
import { OrbitControls } from "three/examples/jsm/controls/OrbitControls.js";
import { GLTFLoader } from "three/examples/jsm/loaders/GLTFLoader.js";
import { RGBELoader } from "three/examples/jsm/loaders/RGBELoader.js";
import { RoughnessMipmapper } from "three/examples/jsm/utils/RoughnessMipmapper.js";
// import { GUI } from "three/examples/jsm/libs/dat.gui.module";
// import Stats from "three/examples/jsm/libs/stats.module.js";
export default {
  name: "ThreeTest",
  data() {
    return {
      camera: null,
      scene: null,
      mesh: null,
      renderer: null,
    };
  },
  mounted() {
    var camera, scene, renderer, mixer, controls, clock, url;
    url = window.location.origin;
    function init() {
      const container = document.getElementById("webgl");
      // container.appendChild(container);

      camera = new THREE.PerspectiveCamera(
        45,
        window.innerWidth / window.innerHeight,
        0.25,
        20
      );
      camera.position.set(5.9, 0, 6);
      scene = new THREE.Scene();
      clock = new THREE.Clock();

      new RGBELoader()
        .setPath(`${url}/hdr/`)
        .load("oberer_kuhberg_1k.hdr", function (texture) {
          texture.mapping = THREE.EquirectangularReflectionMapping;
          scene.environment = texture;

          render();

          // model

          // use of RoughnessMipmapper is optional
          const roughnessMipmapper = new RoughnessMipmapper(renderer);

          const loader = new GLTFLoader().setPath(`${url}/glb/`);
          loader.load("scene.gltf", function (gltf) {
            gltf.scene.traverse(function (child) {
              if (child.isMesh) {
                roughnessMipmapper.generateMipmaps(child.material);
              }
            });
            gltf.scene.scale.set(1, 1, 1);
            scene.add(gltf.scene);
            mixer = new THREE.AnimationMixer(gltf.scene);
            mixer.clipAction(gltf.animations[0]).play();

            animate();
            roughnessMipmapper.dispose();

            render();
          });
        });

      renderer = new THREE.WebGLRenderer({ antialias: true, alpha: true });
      renderer.setPixelRatio(window.devicePixelRatio);
      renderer.setSize(window.innerWidth, window.innerHeight);
      renderer.toneMapping = THREE.ACESFilmicToneMapping;
      renderer.toneMappingExposure = 1;
      renderer.outputEncoding = THREE.sRGBEncoding;
      renderer.setClearColor(0x000000, 0); // the default
      container.appendChild(renderer.domElement);
      // stats = new Stats();
      // container.appendChild(stats.dom);
      // const gui = new GUI();
      // const cameraFolder = gui.addFolder("camera");
      // cameraFolder.add(camera.position, "x", 0);
      // cameraFolder.add(camera.position, "y", 0);
      // cameraFolder.add(camera.position, "z", 0);
      // cameraFolder.open();

      controls = new OrbitControls(camera, renderer.domElement);
      controls.addEventListener("change", render); // use if there is no animation loop
      controls.enablePan = false;
      controls.enableDamping = true;
      controls.target.set(0, 0, -0.2);
      controls.maxZoom = 0.5;
      controls.minZoom = 0;
      controls.minDistance = 2;
      controls.maxDistance = 6;
      controls.update();

      window.addEventListener("resize", onWindowResize);
    }

    function onWindowResize() {
      camera.aspect = window.innerWidth / window.innerHeight;
      camera.updateProjectionMatrix();

      renderer.setSize(window.innerWidth, window.innerHeight);

      render();
    }

    function animate() {
      requestAnimationFrame(animate);

      const delta = clock.getDelta();

      mixer.update(delta);

      controls.update();

      // stats.update();

      renderer.render(scene, camera);
    }

    //

    function render() {
      camera.updateProjectionMatrix();
      renderer.render(scene, camera);
    }
    init();
    render();
  },
  unmount() {
    var el = document.querySelector("canvas");
    el.remove();
  },
};
</script>

<style lang="scss">
@import "./index.scss";
</style>
