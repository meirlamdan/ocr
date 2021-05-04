<template>
  <div>
    <div class="instructions">
      <h4>הוראות</h4>
      <ol>
        <li>בחר תמונה מהמחשב או הדבק כתובת תמונה</li>
        <li>לחץ על 'סרוק' והמתן לתוצאות</li>
        <li>בדוק (וערוך?) את הטקסט והעתק</li>
      </ol>
      <p><b>לע"ע אין תמיכה בקבצי pdf </b></p>
    </div>
    <div>
      <label for="imput-file" class="but-ocr">בחר קובץ</label>
      <input type="file" id="imput-file" accept="image/*" @change="ludeimg" />
      <br />
      <input
        type="text"
        id="input-url"
        v-model="url"
        @input="ludeimg"
        placeholder=" הדבק כתובת תמונה"
      />
      <p v-if="file">{{ file.name }}</p>
    </div>

    <img v-if="src" :src="src" id="img-ocr" alt="תמונה לסריקה" /><br />
    <h4 v-if="err">{{ err }}</h4>
    <div v-if="src && !txt">
      <settings @setLanguage="language = $event" />
      <button class="but-ocr" @click="getTxt(src)">סרוק</button>
    </div>
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
import { ref } from "vue";
import Settings from "./Settings.vue";
export default {
  components: { Settings },
  setup() {
    const txt = ref("");
    const start = ref(false);
    const copy = ref("");
    const file = ref("");
    const url = ref("");
    const src = ref("");
    const err = ref("");
    const language = ref({ lang: "heb", name: "עברית" });

    const ludeimg = async (event) => {
      err.value = "";
      txt.value = "";
      if (event.target.files) {
        file.value = event.target.files[0];
        src.value = URL.createObjectURL(file.value);
        url.value = "";
      } else {
        const img = document.createElement("img");
        const canvas = document.createElement("canvas");
        const ctx = canvas.getContext("2d");
        img.onload = (e) => {
          canvas.width = img.width;
          canvas.height = img.height;
          ctx.drawImage(img, 0, 0);
          src.value = canvas.toDataURL("image/png");
        };
        img.onerror = (e) => {
          err.value =
            "שגיאה! 1. בדוק את הכתובת. 2. בדוק את מדיניות המקור (CORS)";
        };
        img.crossOrigin = "anonymous";
        img.src = url.value;
      }
    };

    const getTxt = async (img) => {
      const worker = createWorker();
      start.value = true;
      await worker.load();
      await worker.loadLanguage(language.value.lang);
      await worker.initialize(language.value.lang);
      start.value = true;
      const {
        data: { text },
      } = await worker.recognize(img);
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
      }, 4000);
    };

    return {
      txt,
      file,
      start,
      copy,
      err,
      language,
      getTxt,
      ludeimg,
      copyTxt,
      src,
      url,
    };
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
  padding: 4px;
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
#input-url {
  height: 20px;
  border: solid 2px rgb(50, 117, 148);
  border-radius: 5px;
  margin: 5px;
}
.but-ocr {
  background: rgb(50, 117, 148);
  color: white;
  padding: 8px 16px;
  font-size: 18px;
  border: 1px solid black;
  border-radius: 4px;
  display: inline-block;
}
.but-ocr:hover {
  border: 2px solid black;
}
#but-copy {
  padding: 6px 12px;
  font-size: 14px;
}
</style>