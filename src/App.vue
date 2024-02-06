<template>
  <h2 class="text-4xl my-10">ocr - סריקת תמונות לטקסט</h2>
  <Ocr />
</template>

<script>

import Ocr from "./components/Ocr.vue";

export default {
  name: "App",
  components: {
    Ocr,
  },
};
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
}
</style>
