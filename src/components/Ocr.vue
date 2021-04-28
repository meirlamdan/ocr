<template>
  <div>
    <div class="instructions">
      <h4>הוראות</h4>
      <ol>
        <li>.בחר תמונה מהמחשב</li>
        <li>.לחץ על 'סרוק' והמתן לתוצאות</li>
        <li>.בדוק (וערוך?) את הטקסט והעתק</li>
      </ol>
    </div>
    <div>
      <label for="imput-file" class="but-ocr">בחר קובץ</label>
      <input type="file" id="imput-file" accept="image/*" @change="ludeimg" />
      <p v-if="file">{{ file.name }}</p>
    </div>
    <img v-if="src" :src="src" id="img-ocr" alt="תמונה לסריקה" /><br />
    <button v-if="src && !txt" class="but-ocr" @click="getTxt">סרוק</button>
    <br />
    <h3 v-if="!txt && start">סורק..</h3>
    <div v-if="txt && !start">
      <h3>התוצאה היא :</h3>
      <textarea
        v-if="txt && !start"
        name=""
        id="textarea-ocr"
        cols="60"
        rows="20"
        v-model="txt"
      ></textarea
      ><br />
      <button @click="copyTxt" id="but-copy" class="but-ocr">העתק</button>
      {{ copy }}
    </div>
  </div>
</template>


<script>
import { createWorker } from "tesseract.js";
import { computed, ref } from "vue";
export default {
  components: {
    FileInput,
  },
  setup() {
    const txt = ref("");
    const start = ref(false);
    const copy = ref("");
    const file = ref("");

    const src = computed(() => {
      return file.value ? URL.createObjectURL(file.value) : "";
    });
    const ludeimg = async (event) => {
      file.value = event.target.files[0];
      txt.value = "";
    };

    const getTxt = async () => {
      if (txt.value) {
        txt.value = "";
      }
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
      copy.value = "הועתק!";
      setTimeout(() => {
        copy.value = "";
      }, 5000);
    };
    return { txt, file, start, copy, getTxt, ludeimg, copyTxt, src };
  },
};
</script>

<style>
#img-ocr {
  width: 60%;
  border: 2px solid #2a668d;
  border-radius: 5px;
  display: block;
  margin: 0 auto;
}
#textarea-ocr {
  width: 80%;
  border: 2px solid #2a668d;
  border-radius: 5px;
  display: block;
  margin: 0 auto;
  font-size: 16px;
  font-family: Helvetica, Arial, sans-serif;
}
.instructions {
  text-align: justify;
  width: 40%;
  border: 1px solid rgb(120, 189, 235);
  border-radius: 5px;
  padding: 0;
  margin: 0 auto;
  margin-bottom: 20px;
  background: rgb(233, 237, 240);
}
.instructions h4 {
  text-align: center;
  margin: 5px;
}
.instructions ol {
  padding: 0;
  list-style-position: inside;
}
#imput-file {
  width: 0.1px;
  height: 0.1px;
  opacity: 0;
  overflow: hidden;
  position: absolute;
  z-index: -1;
}
.but-ocr {
  background: rgb(50, 117, 148);
  color: white;
  padding: 8px 16px;
  font-size: 18px;
  border: 1px solid black;
  border-radius: 4px;
}
.but-ocr:hover {
  border: 2px solid black;
}
#but-copy {
  padding: 6px 12px;
  font-size: 14px;
}
</style>