<template>
  <div id="webgl"></div>
</template>

<script>
import * as THREE from "three";
import { OrbitControls } from "three/examples/jsm/controls/OrbitControls.js";
import { GLTFLoader } from "three/examples/jsm/loaders/GLTFLoader.js";
import { RGBELoader } from "three/examples/jsm/loaders/RGBELoader.js";
import { RoughnessMipmapper } from "three/examples/jsm/utils/RoughnessMipmapper.js";
import { EffectComposer } from "three/examples/jsm/postprocessing/EffectComposer.js";
import { RenderPass } from "three/examples/jsm/postprocessing/RenderPass.js";
import { RGBShiftShader } from "three/examples/jsm/shaders/RGBShiftShader.js";
import { ShaderPass } from "three/examples/jsm/postprocessing/ShaderPass.js";
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
    var camera,
      scene,
      renderer,
      mixer,
      controls,
      clock,
      url,
      composer,
      counter,
      myEffect,
      customPass;
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

          // render();

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
            // post processing
            const vertexShader = `
            varying vec2 vUv;
            void main() {
              vUv = uv;
              gl_Position = projectionMatrix 
                * modelViewMatrix 
                * vec4( position, 1.0 );
            }
            `;
            const fragmentShader = `
            uniform float amount;
            uniform sampler2D tDiffuse;
            varying vec2 vUv;

            float random( vec2 p )
            {
              vec2 K1 = vec2(
                23.14069263277926, // e^pi (Gelfond's constant)
                2.665144142690225 // 2^sqrt(2) (Gelfondâ€“Schneider constant)
              );
              return fract( cos( dot(p,K1) ) * 12345.6789 );
            }

            void main() {

              vec4 color = texture2D( tDiffuse, vUv );
              vec2 uvRandom = vUv;
              uvRandom.y *= random(vec2(uvRandom.y,amount));
              color.rgb += random(uvRandom)*0.15;
              gl_FragColor = vec4( color  );
            }
            `;
            mixer = new THREE.AnimationMixer(gltf.scene);
            mixer.clipAction(gltf.animations[0]).play();
            composer = new EffectComposer(renderer);
            const renderPass = new RenderPass(scene, camera);
            composer.addPass(renderPass);
            const effect1 = new ShaderPass(RGBShiftShader);
            effect1.uniforms["amount"].value = 0.0015;
            composer.addPass(effect1);
            counter = 0.0;
            myEffect = {
              uniforms: {
                tDiffuse: { value: null },
                amount: { value: counter },
              },
              vertexShader: vertexShader,
              fragmentShader: fragmentShader,
            };

            customPass = new ShaderPass(myEffect);
            customPass.renderToScreen = true;
            composer.addPass(customPass);
            roughnessMipmapper.dispose();
            animate();

            // render();
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
      const delta = clock.getDelta();

      mixer.update(delta);

      controls.update();
      counter += 0.01;
      customPass.uniforms["amount"].value = counter;
      composer.render();
      requestAnimationFrame(animate);
    }

    //
    function render() {
      camera.updateProjectionMatrix();
      composer.render();
    }
    init();
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
