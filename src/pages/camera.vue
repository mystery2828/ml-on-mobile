<template>
  <div>
    <button @click="startCamera">Start Camera</button>
    <video ref="video" autoplay></video>
    <canvas ref="canvas"></canvas>
    <div v-if="detections.length > 0">
      <h2>Object Detection Results:</h2>
      <ul>
        <li v-for="(detection, index) in detections" :key="index">
          {{ detection.class }} (Score: {{ detection.score.toFixed(2) }})
        </li>
      </ul>
    </div>
  </div>
</template>

<script>
require("@tensorflow/tfjs-backend-cpu");
require("@tensorflow/tfjs-backend-webgl");
const cocoSsd = require("@tensorflow-models/coco-ssd");

export default {
  data() {
    return {
      detections: [],
      videoStream: null,
    };
  },
  methods: {
    async startCamera() {
        console.log("HIHIHIHIH");
      try {
        const videoElement = this.$refs.video;
        const canvasElement = this.$refs.canvas;

        this.videoStream = await navigator.mediaDevices.getUserMedia({
          video: true,
        });

        videoElement.srcObject = this.videoStream;

        const model = await cocoSsd.load();
        console.log("model ", model);


          canvasElement.width = videoElement.videoWidth;
          canvasElement.height = videoElement.videoHeight;
        console.log("OUT")

          this.detectObjects(videoElement, canvasElement, model);
        
      } catch (error) {
        console.error("Error accessing the camera:", error);
      }
    },

    async detectObjects(video, canvas, model) {
        console.log("INSIDE")
      const context = canvas.getContext("2d");

      const detectFrame = async () => {
        context.drawImage(video, 0, 0, canvas.width, canvas.height);
        const predictions = await model.detect(canvas);
        this.detections = predictions;
        console.log("pred ", predictions)
        requestAnimationFrame(detectFrame);
      };

      requestAnimationFrame(detectFrame);
    },
  },

  beforeDestroy() {
    if (this.videoStream) {
      this.videoStream.getTracks().forEach((track) => {
        track.stop();
      });
    }
  },
};
</script>

<style scoped>
/* Add your component-specific styles here */
</style>
