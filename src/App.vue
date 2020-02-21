<template>
  <div id="app">
    <NotHotDog />
  </div>
</template>

<script>
import NotHotDog from "./components/NotHotDog.vue";

export default {
  name: "App",
  components: {
    NotHotDog
  }
};
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
