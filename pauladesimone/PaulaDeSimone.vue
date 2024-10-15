<script setup>
import { ref, onBeforeMount, computed } from "vue"
import { loadRhino } from "@/scripts/compute.js"

// Import other Vue components in order to add them to a template.
import Header from "commonComponents/Header.vue"
import GeometryView from "./components/GeometryView.vue"
// import SliderInput from "commonComponents/SliderInput.vue"
import SliderInput from "./components/SliderInput.vue"
// import ToggleInput from "commonComponents/ToggleInput.vue"
// import ToggleInput from "./components/ToggleInput.vue"
import DropdownSelector from "./components/DropdownSelector.vue"
import ComputeButton from "./components/ComputeButton.vue"
import Upload3dm from "./components/Upload3dm.vue"

import def from './assets/breaker.gh' //import Grasshopper definition for assets



let sliderName2 = ref("xdiv") //must match the Input name in your GH definition!
let sliderValue2 = ref(0.5) //default slider value

let sliderName3 = ref("ydiv") //must match the Input name in your GH definition!
let sliderValue3 = ref(0.5) //default slider value

let sliderName4 = ref("zdiv") //must match the Input name in your GH definition!
let sliderValue4 = ref(0.5) //default slider value

let sliderName5 = ref("thickness") //must match the Input name in your GH definition!
let sliderValue5 = ref(0.01) //default slider value

let sliderName6 = ref("edge") //must match the Input name in your GH definition!
let sliderValue6 = ref(0.01) //default slider value

// let sliderName = ref("Count") //must match the Input name in your GH definition!
// let sliderValue = ref(10) //default slider value


let toggleName = ref("PlaceRandomly")
let toggleValue = ref(true)

let dropdownName = ref("Color")
let dropdownIndex = ref(0)

const dropdownOptions = [
  { label: "Blue", value: 0 },
  { label: "Pink", value: 1 },
  { label: "White", value: 2 },
  { label: "Green", value: 3 },
  { label: "Grey", value: 4 },
  { label: "Yellow", value: 5 },

];

let encodedFile = ref(null);
let isButtonDisabled = ref(true)

///.............................................
let path = def //path to the Grasshopper definition
let data = ref({})
let metadata = ref([])
let compute = ref(false)


function updateValue(newValue, parameterName) {

  // if (parameterName === sliderName.value) {
  //   sliderValue.value = newValue
  // }
  if (parameterName === sliderName2.value) {
    sliderValue2.value = newValue
  }

  else if (parameterName === sliderName3.value) {
    sliderValue3.value = newValue
  }

  else if (parameterName === sliderName4.value) {
    sliderValue4.value = newValue
  }

  else if (parameterName === sliderName5.value) {
    sliderValue5.value = newValue
  }

  else if (parameterName === sliderName6.value) {
    sliderValue6.value = newValue
  }

  else if (parameterName === toggleName.value) {
    toggleValue.value = newValue
  }

  else if (parameterName === dropdownName.value) {
    dropdownIndex.value = newValue
  }

  console.log(parameterName + " : " + newValue)
}

function update3dmData(newData) {
  encodedFile.value = newData
  isButtonDisabled.value = false
  compute.value = true

}

function receiveMetadata(newValue) {
  console.log(newValue)
  metadata.value = newValue
}

function runCompute(newVal) {
  compute.value = newVal
}

// a computed ref. Vue will keep track of this and update it
const computeData = computed(() => {
  data = {
    ["encodedFile"]: encodedFile.value,
    
    [sliderName2.value]: Number(sliderValue2.value),
    [sliderName3.value]: Number(sliderValue3.value),
    [sliderName4.value]: Number(sliderValue4.value),
    [sliderName5.value]: Number(sliderValue5.value),
    [sliderName6.value]: Number(sliderValue6.value),
    // [toggleName.value]: Boolean(toggleValue.value),
    [dropdownName.value]: Number(dropdownIndex.value)
  };

  return data
})

</script>


<template>

  <div id="sidebar" class="container">
    <img class="deeco-logo-image" alt="sad dino" src="./assets/deecologo.png" style="width: 200px; height: auto;" />
    <h3 class="h3Style">Input your toy volume and we'll create a model to assemble!</h3>
    <!-- <p> Input your toy volume and we'll create an model to assemble!
    </p> -->
    <Upload3dm @encoded3dm="update3dmData" />

    
    <SliderInput :title="sliderName2" :min="0.0" :max="1.0" :step="0.1" :val="5" @update="updateValue"></SliderInput>
    <SliderInput :title="sliderName3" :min="0.0" :max="1.0" :step="0.1" :val="5" @update="updateValue"></SliderInput>
    <SliderInput :title="sliderName4" :min="0.0" :max="1.0" :step="0.1" :val="5" @update="updateValue"></SliderInput>
    <SliderInput :title="sliderName5" :min="0.01" :max="0.05" :step="0.01" :val="5" @update="updateValue"></SliderInput>
    <SliderInput :title="sliderName6" :min="0.10" :max="0.50" :step="0.01" :val="5" @update="updateValue"></SliderInput>
    <!-- <SliderInput :title="sliderName" :min="1" :max="10" :step="1" :val="5" @update="updateValue"></SliderInput> -->

    <!-- <ToggleInput :title="toggleName" :val="true" @update="updateValue"></ToggleInput> -->

    <DropdownSelector :title="dropdownName" :options="dropdownOptions" :val="dropdownIndex" @update="updateValue"
    />
    <ComputeButton title="Compute" @click="runCompute" :isDisabled="isButtonDisabled" />
  </div>


  <div id="viewer" class="container" >

    
    <div class="metadata-container"  v-if="metadata && metadata.length > 0"> 
      <!-- <div id="innerDivision"  -->
        
        <!-- <div class="metadata" style="text-align: left; color: black;"> -->
          <h2 style="text-align: center;">your toy!</h2>
            
              <p id="metadatatitle">01. {{ metadata[0].name }} = {{ metadata[0].value }}</p>
              <p id="metadatatitle02">02. {{ metadata[1].name }} = {{ metadata[1].value }}</p>
              <p id="metadatatitle03">03. {{ metadata[2].name }} = {{ metadata[2].value }}</p>
        <!-- </div> -->
      <!-- </div> -->
    </div>

    <div class="threejs-container1" id="threejs-container1"></div>
    <GeometryView :data="computeData" :path="path" :runCompute="compute" @updateMetadata="receiveMetadata" />
  </div>

</template>
  
 
  
<style scoped>

.metadata-container,
.threejs-container1 {
  flex: 1;
}

.threejs-container1 {
  position: relative;
  background-color: rgb(226, 240, 244);
  border-radius: 100px;  
  order: 1;
  

}

.metadata-container {
  background-color: #f5f6fa;
  padding: 80px;
  border-radius: 80px;
  text-align: left;
  order: 2;
  margin: 20px;
  width: calc(100% - 40px);
  
  
}


#content {
  display: flex;
  height: calc(100vh - 68px);
}

#sidebar {
  width: 310px;
  padding: 10px;
  font-family: Helvetica;
  color: #000000;
  font-size: 12px;
  text-align: center;
}

#viewer {
  display: flex;
  width: 100%;
  border: 1px solid black;
  border-radius: 100px;
  margin: 30px;
  
}


/* #innerDivision {
  margin-top: 20px;
  margin-right: 10px;
  margin-left: 10px;
  width: 200px;
  padding: 20px;
  color: black;
} */

/* .metadata {
  margin-top: 20px;
  margin-right: 10px;
  color: black;
} */

#metadatatitle {
  margin-top: 50px;
  /* margin-right: 50px; */

  color: black;
}
#metadatatitle02 {
  margin-top: 50px;
  /* margin-right: 50px; */
  color: black;
}
#metadatatitle03 {
  margin-top: 50px;
  /* margin-right: 50px; */
  color: black;
}

.container {
  background-color: rgb(226, 240, 244);
  border-width: 1px;
  border-radius: 50px;
  box-shadow: 0 0 10px rgba(0, 0, 0, 0.1);
  /* Adjust the shadow as needed */
  margin: 30px;
  /* Adjust the margin to create the offset */
  
}

.modern-range {
  -webkit-appearance: none;
  width: 100%;
  height: 5px;
  /* Adjust the height as needed */
  background-color: beige;
  /* Change the background color to beige */
  border-radius: 5px;
  /* Adjust the border radius as needed */
  margin: 10px 0px;
}

.modern-range::-webkit-slider-thumb {
  -webkit-appearance: none;
  height: 20px;
  /* Adjust the height of the dot */
  width: 20px;
  /* Adjust the width of the dot */
  border-radius: 50%;
  /* Make the dot circular */
  background-color: black;
  /* Set the color of the dot to black */
  cursor: pointer;
  margin-top: -8px;
  /* Adjust the margin to vertically center the dot */
}

.h3Style {
  color: black;
}
</style>