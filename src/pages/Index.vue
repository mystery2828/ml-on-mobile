<template>
  <q-page class="flex flex-center">

    <div>
    <input type="file" @change="detectObjects" accept="image/*" />
    <img ref="image" alt="Selected Image" height="200" width="200" />
    <div v-if="detections.length > 0">
      <h2>Object Detection Results:</h2>
      <ul>
        <li v-for="(detection, index) in detections" :key="index">
          {{ detection.class }} (Score: {{ detection.score.toFixed(2) }})
        </li>
      </ul>
    </div>
  </div>
  </q-page>
</template>

<script>
import { defineComponent } from "vue";
require("@tensorflow/tfjs-backend-cpu");
require("@tensorflow/tfjs-backend-webgl");
const cocoSsd = require("@tensorflow-models/coco-ssd");


export default {
 data(){
   return{
     detections:[]
   } ;
 },
 methods: {
    async detectObjects(event) {
      const file = event.target.files[0];
      if (!file) return;

      const imageElement = this.$refs.image;
      const image = new Image();
      image.src = URL.createObjectURL(file);
      image.onload = async () => {
        imageElement.src = image.src;

        const model = await cocoSsd.load();
        const predictions = await model.detect(image);

        this.detections = predictions;
      };
    },
  },
}
</script>
