<template>
  <div >
    <input type="file" id="file" @change="ludeImg" /><br />
    <img
      v-if="src"
      :src="src"
      class="img-ocr"
      alt="תמונה נטענת"
      width="400"
      height="400"
    /><br />
    <button v-if="src" @click="getTxt">סרוק</button> <br />
    <h3 v-if="!txt && start">סורק..</h3>
    <div v-if="txt && !start">
      <textarea
        v-if="txt && !start"
        name=""
        id="textarea-ocr"
        cols="60"
        rows="20"
        v-model="txt"
      ></textarea
      ><br />
      <button @click="copyTxt">העתק</button> {{copy}}
    </div>
  </div>
</template>

<script>
import { createWorker } from "tesseract.js";
import { ref } from "vue";
export default {
  setup() {
    const txt = ref("");
    const src = ref("");
    const start = ref(false);
    const copy = ref("")


    const ludeImg = (event) => {
      const file = event.target.files[0];
      src.value = URL.createObjectURL(file);
    };
    const getTxt = async () => {
      if(txt.value){txt.value = ''}
      const worker = createWorker();
      await worker.load();
      await worker.loadLanguage("heb+eng");
      await worker.initialize("heb+eng");
        start.value = true;

      const {
        data: { text },
      } = await worker.recognize(src.value);
      txt.value = text;
      start.value = false;
      await worker.terminate();
    };

    const copyTxt = () => {
      document.querySelector("textarea").select();
      document.execCommand("copy");
      copy.value = "הועתק !"
      setTimeout(() => {
          copy.value = ""
      }, 5000);
    };
    return { txt, src, start, copy, ludeImg, getTxt, copyTxt };
  },
};
</script>

<style>
#textarea-ocr ,.img-ocr {
  border: 2px solid #2a668d;
  border-radius: 5px;
}
</style>