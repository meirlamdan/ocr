<template>
  <img alt="Vue logo" src="./assets/scan.png" height="250" width="250">
  <HelloWorld msg="Welcome to Your Vue.js App"/>
  <Ocr/>

</template>

<script>
import Ocr from './components/Ocr.vue'

export default {
  name: 'App',
  components: {
    Ocr
  }
}
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
