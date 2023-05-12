<template>
  <div>
    <canvas ref="webGl" class="webGl" />
  </div>
</template>

<script lang="ts">
import {
  Scene,
  SphereGeometry,
  MeshBasicMaterial,
  MeshStandardMaterial,
  Mesh,
  PointLight,
  PerspectiveCamera,
  WebGLRenderer,
  TextureLoader,
  HemisphereLight,
} from "three";
import { OrbitControls } from "three/addons/controls/OrbitControls.js";
import { watch, onMounted, ref, computed } from "vue";
import { useWindowSize } from "@vueuse/core";
export default {
  setup() {
    const webGl = ref();
    const img = "../assets/images/earth.jpg";
    const { width, height } = useWindowSize();
    const aspectRatio = computed(() => {
      return width.value / height.value;
    });
    let camera: PerspectiveCamera;
    let renderer: WebGLRenderer;
    let scene: Scene;
    let mesh: Mesh;
    let controls: OrbitControls;
    let light: PointLight;
    const setCanvas = () => {
      // Create Scene
      scene = new Scene();
      scene.background = new TextureLoader().load("/images/galaxy1.avif");

      // Create Object
      const geometry = new SphereGeometry(5, 50, 50);
      const material = new MeshStandardMaterial({
        map: new TextureLoader().load("/images/earth2.jpg"),
      });
      mesh = new Mesh(geometry, material);
      scene.add(mesh);

      // Lights
      light = new PointLight(0xffffff, 1);
      // light = new HemisphereLight(0xffff, 0x080820, 1);
      light.position.set(50, 50, 50);
      scene.add(light);

      // Camera
      camera = new PerspectiveCamera(45, aspectRatio.value, 0.1, 100);
      camera.position.z = 30;
      scene.add(camera);
      camera.add(light);

      // Renderer
      const canvas = webGl.value;
      renderer = new WebGLRenderer({ canvas, antialias: true });
      renderer.setSize(width.value, height.value);
      renderer.render(scene, camera);

      // Controls
      controls = new OrbitControls(camera, canvas);

      controls.enableDamping = true;
    };

    const updateCamera = () => {
      camera.aspect = aspectRatio.value;
      camera.updateProjectionMatrix();
    };

    const updateRenderer = () => {
      renderer.setSize(width.value, height.value);
      renderer.render(scene, camera);
    };

    watch(aspectRatio, (val) => {
      if (val) {
        updateCamera();
        updateRenderer();
      }
    });

    const animate = () => {
      // mesh.rotation.y += 0.01;
      controls.update();
      renderer.render(scene, camera);
      requestAnimationFrame(animate);
    };

    onMounted(() => {
      setCanvas();
      animate();
    });

    return { webGl };
  },
};
</script>
