<template>
  <div id="header"><h2>ocr - סריקת מסמכים לטקסט</h2></div>
  <img alt="Vue logo" src="./assets/scan.png" height="250" width="250" />
  <br />
  <br />
  <br />
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
#header h2 {
  background: #deeaf2;
  margin: 0px;
}
</style>
