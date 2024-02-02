<script setup>
import { ref, onMounted } from "vue";
import { createWorker } from "tesseract.js";

const file = ref("");
const url = ref("");
const err = ref("");

const progress = ref(0);
const show = ref("text");

const worker = ref(null);
const languages = ref([
  { value: "heb", name: "עברית" },
  { value: "eng", name: "אנגלית" },
  { value: "eng+heb", name: "עברית ואנגלית" },
]);
const language = ref("heb");
const loadWorker = async () => {
  worker.value = await createWorker(language.value, null, {
    logger: log => {
      if (log.status === "recognizing text") {
        progress.value = (log.progress * 100).toFixed(2);
      } else {
        console.log(log.status);
      }
    },
  });
};

onMounted(async () => {
  await loadWorker();
});

const src = ref()
const loadImg = async (event) => {
  progress.value = 0;
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

const txt = ref("");
const lines = ref(null);
const inProgress = ref(false);
const getTxt = async () => {
  inProgress.value = true;
  const {
    data: { text, blocks, hocr, tsv },
  } = await worker.value.recognize(src.value);
  txt.value = text;
  const image = new Image();
  image.src = src.value;


  image.onload = () => {
    lines.value = blocks?.[0].paragraphs[0].lines.map(({ text, bbox }) => ({
      text,
      imgPath: imagePartSrc(image, bbox.x0, bbox.y0, bbox.x1 - bbox.x0, bbox.y1 - bbox.y0),
    }));
    console.log(lines.value);
  };
  inProgress.value = false;
  //  await worker.value.terminate();
};

const imagePartSrc = (img, xStart, yStart, w, h) => {
  const canvas = document.createElement('canvas');
  canvas.width = w;
  canvas.height = h;
  const ctx = canvas.getContext('2d');
  ctx.drawImage(img, xStart, yStart, w, h, 0, 0, w, h);


  return canvas.toDataURL('image/png');
};

const copy = ref("");
const copyTxt = (value) => {
  const textarea = document.createElement("textarea");
  textarea.value = value;
  document.body.appendChild(textarea);
  textarea.select();
  document.execCommand("copy");
  document.body.removeChild(textarea);
  copy.value = "הועתק!";
  setTimeout(() => {
    copy.value = "";
  }, 2000);
};

const pasteHendler = (e) => {
  if (e.clipboardData.files[0]) {
    file.value = e.clipboardData.files[0]
    src.value = URL.createObjectURL(file.value);

  }
}

const dropHendler = (e) => {
  if (e.dataTransfer.files[0]) {
    file.value = e.dataTransfer.files[0]
    src.value = URL.createObjectURL(file.value);
  }
}

const dragoverHendler = (e) => {
  e.preventDefault();
  e.dataTransfer.dropEffect = "copy";

}

</script>

<template>
  <div>
    <select v-model="language" @change="loadWorker">
      <option v-for="lang in languages" :key="lang.value" :value="lang.value">{{ lang.name }}</option>
    </select>
    <div>
      <label @paste="pasteHendler" @drop="dropHendler" @dragover="dragoverHendler" for="imput-file"
        class="but-ocr border border-dashed rounded border-black h-24 w-96 bg-gray-50 hover:bg-gray-100  m-auto flex justify-center items-center">
        <div class="text-sm">גרור או הדבק תמונה
          <br />
          או לחץ כאן לבחור תמונה
        </div>
      </label>
      <input type="file" id="imput-file" accept="image/*" @change="loadImg" hidden />
      <!-- <br />
      <input type="text" id="input-url" v-model="url" @input="loadImg" placeholder=" הדבק כתובת תמונה" />
      <p v-if="file">{{ file.name }}</p> -->
    </div>
    <div class="w-72 m-auto my-8"><img v-if="src" :src="src" id="img-ocr" class="w-full" alt="תמונה לסריקה" /><br /></div>

    <h4 v-if="err">{{ err }}</h4>
    <div v-if="src && !txt">
      <button class="bg-blue-500   text-lg px-2 rounded hover:bg-blue-700 text-white" @click="getTxt()">סרוק</button>
    </div>
    <div v-if="txt || inProgress">
      <h3 v-if="inProgress">סורק..</h3>
      <progress :value="progress" max="100" class="h-2 border bg-blue-800"></progress> <span class="text-sm">{{ progress
      }}%</span>
    </div>
    <div v-if="txt" class="mt-10">
      <div class=""><span @click="show = 'text'" class="border p-1"
          :class="show === 'text' ? 'bg-black text-white' : ''">הצגה לפי טקסט</span><span @click="show = 'lines'"
          class="p-1" :class="show === 'lines' ? 'bg-black text-white' : ''">הצגה לפי שורות</span></div>
      <div class="w-fit m-auto border p-2 ">
        <div v-if="show === 'text'"> <textarea v-if="txt && !start" name="" id="textarea-ocr" cols="60"
            :rows="txt.split('\n').length" v-model="txt" focus></textarea><br />
          <button @click="copyTxt(txt)" id="but-copy" class="but-ocr">העתק</button>
          {{ copy }}
        </div>
        <div v-if="show === 'lines'">
          <div v-for="(line, i) in lines" :key="i">
            <div>
              <img :src="line.imgPath" alt="">
            </div>
            <div><input class="w-full" type="text" v-model="line.text" focus></div>
          </div>
          <button @click="copyTxt(lines.map(l => l.text).join('\n'))" id="but-copy" class="but-ocr">העתק</button>
          {{ copy }}
        </div>
      </div>
    </div>
  </div>
</template>
<style>
/* Base styles for the progress element */
progress {
  height: 20px;
  color: #4a90e2;
  background-color: #eee;
  border-radius: 10px;
}

/* For browsers that support the ::-webkit-progress-bar pseudo-element */
progress::-webkit-progress-bar {
  background-color: #eee;
  border-radius: 10px;
}

progress::-webkit-progress-value {
  background-color: #4a90e2;
  border-radius: 10px;
}

/* For browsers that support the ::-moz-progress-bar pseudo-element */
progress::-moz-progress-bar {
  background-color: #4a90e2;
  border-radius: 10px;
}
</style>