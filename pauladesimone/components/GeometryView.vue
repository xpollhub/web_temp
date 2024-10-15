<template>
  <div id="viewport">
    <!-- To this element we will append our 3d scene. -->
    <div id="threejs-container"></div>
  </div>
</template>

<script setup>
// Imports;
import { onMounted, watch } from 'vue'
import * as THREE from "three"
import { OrbitControls } from "three/examples/jsm/controls/OrbitControls"
import { Rhino3dmLoader } from "three/addons/loaders/3DMLoader.js"
import { runCompute } from "@/scripts/compute.js"
import { loadRhino } from "@/scripts/compute.js";

// Import wooden texture
// import woodTexture from './assets/wood_txtr.jpg';

const loader = new Rhino3dmLoader()
loader.setLibraryPath('https://cdn.jsdelivr.net/npm/rhino3dm@8.0.0-beta2/')

const props = defineProps(["data", "path", "runCompute"]);
const emits = defineEmits(["updateMetadata"]); 

// TEXTURES

// const woodTexture = new THREE.TextureLoader().load('./assets/wood_txtr.jpg');
// console.log(woodTexture)

// const textureLoader = new THREE.TextureLoader();

watch(() => props.runCompute, (newValue) => {
  if (newValue) {
    compute();
  }
})

let renderer, camera, scene, controls, container

function init() {

  THREE.Object3D.DEFAULT_UP.set(0, 0, 1);
  renderer = new THREE.WebGLRenderer()
  container = document.getElementById("threejs-container")
  renderer.setSize(container.offsetWidth, container.offsetHeight)
  container.appendChild(renderer.domElement)

  camera = new THREE.PerspectiveCamera(75, container.offsetWidth / container.offsetHeight, 0.1, 1000)
  camera.position.set(0, 0, 40)

  scene = new THREE.Scene()
  scene.background = new THREE.Color("#f5f6fa")

  controls = new OrbitControls(camera, renderer.domElement)
  controls.enableRotate = true
  controls.enableRotateX = true
  controls.enableRotateY = true
  controls.enablePan = false
  controls.enableZoom = true

  const ambientlight = new THREE.AmbientLight(0xffffff, 1)
  ambientlight.position.set(0, 0, 0)
  scene.add(ambientlight)

  const directionalLight = new THREE.DirectionalLight(0xffffff, 3)
  directionalLight.position.set(0, 1, 0)
  scene.add(directionalLight)

  animate()
}

async function compute() {
  console.log("Runnning compute... \ndata sent: ", props.data)
  const doc = await runCompute(props.data, props.path)

  if (doc.metadata) {
    emits("updateMetadata", doc.metadata);
  }

  scene.traverse((child) => {
    if (!child.isLight) {
      scene.remove(child)
    }
  })

  const buffer = new Uint8Array(doc.toByteArray()).buffer;
  loader.parse(buffer, function (object) {
    object.traverse((child) => {
      if (child.isLine) {
        if (child.userData.attributes.userStrings != undefined && child.userData.attributes.userStrings.length > 0) {
          const colorData = child.userData.attributes.userStrings[0]
          const col = colorData[1]
          const threeColor = new THREE.Color("rgb(" + col + ")")
          const mat = new THREE.LineBasicMaterial({ color: threeColor })
          child.material = mat
        }
      }
    })


    // TEXTURES OP 01

//     // Load a texture
// textureLoader.load(
//   './assets/wood_txtr.jpg', // Path to the texture file
//     function (texture) {
//         // Texture loaded successfully
//         // You can now use the texture in your scene
//         const material = new THREE.MeshBasicMaterial({ map: texture });
//         const mesh = new THREE.Mesh(geometry, material);
//         scene.add(mesh);
//     },
//     function (xhr) {
//         // Progress callback (optional)
//         console.log((xhr.loaded / xhr.total * 100) + '% loaded');
//     },
//     function (error) {
//         // Error callback
//         console.error('An error happened', error);
//     }
// );


  // TEXTURES OP 02

    // object.traverse((child) => {
    //   if (child.isMesh) {
    //     // Create a material with the wooden texture
    //     const material = new THREE.MeshBasicMaterial({
    //       map: woodTexture, // Apply the wooden texture
    //     });
    //     child.material = material;
    //   }
    // });

    scene.add(object)

    for (let child of scene.children) {
      if (child.type == "Object3D") {
        zoomCameraToSelection(camera, controls, child.children)
      }
    }

    console.log("Compute done")
  });
}

function animate() {
  requestAnimationFrame(animate);
  controls.update();
  renderer.render(scene, camera);
}

window.addEventListener("resize", onWindowResize);
function onWindowResize() {
  const height = container.offsetHeight
  const width = container.offsetWidth
  camera.aspect = width / height
  camera.updateProjectionMatrix();
  renderer.setSize(width, height)
}

function zoomCameraToSelection(camera, controls, selection, fitOffset = 1.1) {
  const box = new THREE.Box3();
  for (const object of selection) {
    if (object.isLight) continue
    box.expandByObject(object);
  }
  const size = box.getSize(new THREE.Vector3());
  const center = box.getCenter(new THREE.Vector3());
  const maxSize = Math.max(size.x, size.y, size.z);
  const fitHeightDistance = maxSize / (2 * Math.atan(Math.PI * camera.fov / 360));
  const fitWidthDistance = fitHeightDistance / camera.aspect;
  const distance = fitOffset * Math.max(fitHeightDistance, fitWidthDistance);
  const direction = controls.target.clone()
    .sub(camera.position)
    .normalize()
    .multiplyScalar(distance);
  controls.maxDistance = distance * 10;
  controls.target.copy(center);
  camera.near = distance / 100;
  camera.far = distance * 100;
  camera.updateProjectionMatrix();
  camera.position.copy(controls.target).sub(direction);
  controls.update();
}

onMounted(async () => {
  init()
  await loadRhino()
})

</script>

<style scoped>
#viewport {
  height: 100%;
  width: 100%;
  min-width: 200px;
  position:inherit;
}

#threejs-container {
  height: 94%;
  width: 80%;
  min-width: 600px;
  border-radius: 80px;
  overflow: hidden;
  margin: 20px;
  position:inherit;
}
</style>