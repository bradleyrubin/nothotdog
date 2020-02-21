<template>
  <div>
    <h1>Hot Dog or Pizza?</h1>
    <div id="webcam-container" />
    <p v-if="!model || !webcam">
      Loading...
    </p>
    <div v-else>
      <p v-if="isPredicting">
        Predicting...
      </p>
      <button v-else-if="!prediction" @click="predict(false)">Predict</button>
      <button v-else @click="reset">Reset</button>
      <h3>{{ resultsString }}</h3>
    </div>
  </div>
</template>

<script>
const URL = 'https://teachablemachine.withgoogle.com/models/M-F7kky7/';
const modelURL = URL + 'model.json';
const metadataURL = URL + 'metadata.json';

import _ from 'lodash';
// import * as tf from '@tensorflow/tfjs';
import * as tmImage from '@teachablemachine/image';

export default {
  name: 'NotHotDog',
  data() {
    return {
      model: null,
      webcam: null,
      maxPredictions: null,
      prediction: null,
      isPredicting: false,
    };
  },
  created() {
    this.setupWebcamAndModel();
  },
  computed: {
    resultsString() {
      return this.prediction ? this.topPredictionString(this.prediction) : '';
    },
  },
  methods: {
    async setupWebcamAndModel() {
      await this.setupWebcam();
      await this.loadModel();
      this.predict(true); // The first prediction takes a while for some reason
    },
    async setupWebcam() {
      this.webcam = new tmImage.Webcam(200, 200, true); // width, height, flip
      await this.webcam.setup(); // request access to the webcam
      await this.webcam.play();
      window.requestAnimationFrame(this.loop);
      document
        .getElementById('webcam-container')
        .appendChild(this.webcam.canvas);
    },
    async loadModel() {
      this.model = await tmImage.load(modelURL, metadataURL);
    },
    loop() {
      this.webcam.update();
      window.requestAnimationFrame(this.loop);
    },
    topPredictionString(prediction) {
      const maxPrediction = _.maxBy(prediction, 'probability');
      return `${maxPrediction.className} (${(
        maxPrediction.probability * 100
      ).toFixed(0)}%)`;
    },
    async predict(ignoreResult = false) {
      if (ignoreResult) {
        await this.model.predict(this.webcam.canvas);
        return;
      }
      this.isPredicting = true;
      this.prediction = await this.model.predict(this.webcam.canvas);
      this.isPredicting = false;
      console.log(JSON.stringify(this.prediction));
    },
    reset() {
      this.prediction = null;
    },
  },
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped></style>
