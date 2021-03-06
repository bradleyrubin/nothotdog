<template>
  <div>
    <h1>Hot Dog or Not Hot Dog?</h1>
    <div id="webcam-container" />
    <p v-if="!mobilenetModel || !webcam">Loading...</p>
    <div v-else>
      <p v-if="isPredicting">Predicting...</p>
      <button v-else-if="!prediction" @click="predict(false)">Predict</button>
      <button v-else @click="reset">Reset</button>
      <h2 v-if="prediction">{{ notHotDogResults }}</h2>
      <br /><br />
      <h3>{{ resultsString }}</h3>
      <p>{{ fullResults }}</p>
    </div>
  </div>
</template>

<script>
import _ from 'lodash';
// import * as tf from '@tensorflow/tfjs';
import * as tmImage from '@teachablemachine/image';
import * as mobilenet from '@tensorflow-models/mobilenet';

export default {
  name: 'NotHotDog',
  data() {
    return {
      mobilenetModel: null,
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
    fullResults() {
      return this.prediction ? JSON.stringify(this.prediction) : '';
    },
    isHotDog() {
      if (!this.prediction) {
        return false;
      }
      for (let i = 0; i < this.prediction.length; i++) {
        if (this.prediction[i].className.includes('hotdog')) {
          return true;
        }
      }
      return false;
    },
    notHotDogResults() {
      return this.isHotDog ? 'Hot Dog' : 'Not Hot Dog';
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
      this.mobilenetModel = await mobilenet.load();
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
        await this.mobilenetModel.classify(this.webcam.canvas);
        return;
      }
      this.isPredicting = true;
      this.prediction = await this.mobilenetModel.classify(this.webcam.canvas);
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
