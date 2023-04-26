<template>
  <div id="app">
    <input type="file" id="file-input" accept="image/*" />
    <div><canvas id="canvas"></canvas></div>
    <div style="margin-top:70px;">{{ text }}</div>
  </div>
</template>

<script>
/* eslint-disable */
import { createWorker, PSM, OEM } from "tesseract.js";
const path = require("path");
const worker = createWorker({
  workerPath: "/tesseract/tesseract.js/dist/worker.min.js",
  corePath: "/tesseract/tesseract.js-core/tesseract-core.wasm.js",
  langPath: "/tesseract/tesseract-lang", 
  logger: (m) => console.log(m),
});

export default {
  name: "app",
  data() {
    return {
      haveInit: false,
      text: "",
    };
  },
  mounted(){
    this.showImg();
  },
  methods: {
    async recognize() {
      const fileInput = document.querySelector("#file-input");
      if (!this.haveInit) {
        await worker.load();
        await worker.loadLanguage("chi_sim");
        await worker.initialize("chi_sim", OEM.LSTM_ONLY);
        await worker.setParameters({
          tessedit_pageseg_mode: PSM.SINGLE_BLOCK,
        });
        this.haveInit = true;
      }

      const img = document.getElementById("canvas");
      const {
        data: { text },
      } = await worker.recognize(img);
      this.text = text;
      console.timeEnd("time:");
    },
    showImg() {
      const fileInput = document.querySelector("#file-input");
      fileInput.addEventListener("change", async () => {
        const file = fileInput.files[0];
        if (file) {
          const reader = new FileReader();
          reader.readAsDataURL(file);
          reader.onload = async () => {
            const img = new Image();
            img.src = reader.result;
            img.onload = () => {
              canvas.width = img.width;
              canvas.height = img.height;
              const ctx = canvas.getContext("2d");
              ctx.drawImage(img, 0, 0);
              this.recognize();
            };
          };
        }
      });
    },
  },
};
</script>

<style>
#app {
  font-family: "Avenir", Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
