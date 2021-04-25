<template>
  <div>
    <h2>ocr -סריקת מסמכים לטקסט</h2>
    <input type="file" id="file" @change="ludeImg" /><br />
    <img
      v-if="src"
      :src="src"
      class="img-ocr"
      alt="תמונה נטענת"
      width="400"
      height="400"
    /><br />
    <button @click="getTxt">סרוק</button> <br />
    <h3 v-if="!txt && start">סורק..</h3>
    <div v-if="txt && !start">
      <textarea
        v-if="txt && !start"
        name=""
        id=""
        cols="60"
        rows="20"
        v-model="txt"
      ></textarea
      ><br />
      <button @click="copyTxt">העתק</button>
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
    };
    return { txt, src, start, ludeImg, getTxt, copyTxt };
  },
};
</script>

<style>
.img-ocr {
  border: 1px solid black;
}
</style>