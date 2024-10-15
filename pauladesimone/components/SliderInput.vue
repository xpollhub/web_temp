<script setup>
import { ref, defineProps, defineEmits } from "vue";

const props = defineProps(['title', 'min', 'max', 'step', 'val']);
const title = ref(props.title);
const sliderValue = ref(props.val);
const emits = defineEmits(['update']);

function emitValueUpdate() {
  emits("update", sliderValue.value, title.value);
}
</script>

<template>
  <div>
    <form class="definition-input">
      <label class="paulaSlider" for="range-slider">{{ title }}: {{ sliderValue }}</label>
      <input type="range" class="modern-range custom-slider" :min="min" :max="max" :step="step" :val="val" v-model="sliderValue" @mouseup="emitValueUpdate" />
    </form>
  </div>
</template>

<style scoped>
.custom-slider label {
  color: black;
}

.modern-range {
  -webkit-appearance: none;
  width: 85%;
  height: 5px;
  border-radius: 5px;
  background-color: rgb(192, 159, 118); /* Change background color to beige */
  outline: none;
  margin: 8px 0px;
}

.modern-range::-webkit-slider-thumb {
  -webkit-appearance: none;
  appearance: none;
  width: 10px;
  height: 10px;
  border-radius: 50%;
  background: #000; /* Set thumb color to black */
  cursor: pointer;
}

.modern-range::-moz-range-thumb {
  width: 10px;
  height: 10px;
  border-radius: 50%;
  background: #000; /* Set thumb color to black */
  cursor: pointer;
}

.modern-range::-ms-thumb {
  width: 10px;
  height: 10px;
  border-radius: 50%;
  background: #000; /* Set thumb color to black */
  cursor: pointer;
}

.paulaSlider {
  font-family: 'Roboto Mono', monospace;
  font-size: 1.1rem;
  text-align: left;
  color: #000000;
  margin-bottom: 1rem;
  font-weight: bold;
}
</style>