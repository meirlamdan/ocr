<template>
  <div >
      <div class="instructions">
          <h4>הוראות</h4>
          <ol>
              <li>.בחר תמונה מהמחשב</li>
              <li>.לחץ על 'סרוק' והמתן לתוצאות</li>
              <li>.בדוק (וערוך?) את הטקסט והעתק</li>
          </ol>

      </div>
    <input type="file" id="file" @change="ludeImg" /><br />
    <img
      v-if="src"
      :src="src"
      class="img-ocr"
      alt="תמונה נטענת"
      width="400"
      height="400"
    /><br />
    <button v-if="src" id="button-scan" @click="getTxt">סרוק</button> <br />
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
      <button  @click="copyTxt">העתק</button> {{copy}}
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
      document.querySelector("textarea").blur();
      copy.value = "הועתק!"
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
  margin-top: 20px;
}
.instructions{
list-style-position: inside;
text-align: justify;
width: 40%;
border: 1px  solid rgb(120, 189, 235);
border-radius: 5px;
margin: 0 auto;
margin-bottom: 20px;
background: rgb(233, 237, 240);


}
.instructions h4 {
text-align:center
}

#button-scan{
    background: rgb(135, 187, 211);
    color: white;
    padding: 10px 24px;
    font-size: 16px;
    border: none;
}
</style>