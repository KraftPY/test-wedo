<template>
  <div class="main">
    <div class="name-container">
      <input type="text" name="comp-name" id="comp-name" v-model="compName" autocomplete="off" />
      <svg class="input-right-line">
        <path
          d="M 470,24 L470,40
                  M 470,42 L470,46
                  M 470,49 L470,56
                  M 470,60 L470,68
                  M 470,73 L470,89"
        />
      </svg>
      <svg class="input-bottom-line">
        <path
          d="M 250,40 L298,40
                  M 314,40 L330,40
                  M 338,40 L362,40
                  M 410,40 L414,40
                  M 430,40 L438,40"
        />
      </svg>
      <label for="comp-name">
        <span>Название компании</span>
        <svg class="name-left-line">
          <path d="M 1,1 L1,5 
                    M 1,9 L1,41
                    M 1,45 L1,61" />
        </svg>
        <svg class="name-bottom-line">
          <path d="M 16,41 L112,41" />
        </svg>
      </label>
      <div class="valid-blc none"></div>
    </div>
    <div class="upload-container">
      <label>
        <span>прикрепить файл</span>
        <input type="file" id="up-logo" @change="onChange" accept="image/png, image/jpeg" />
        <div v-if="tempFile" class="icon sel"></div>
        <div v-else class="icon no-sel"></div>
        <svg class="upload-bottom-line">
          <path d="M 144,22 L162,22" />
        </svg>
      </label>
    </div>
    <button :disabled="isDownload" class="btn-download" @click="onDownload">
      <svg class="top-line">
        <path d="M 25,8 L128,8
            M 175,8 L195,8
            M 210,8 L215,8" />
      </svg>
      <svg class="bottom-line">
        <path d="M 8,40 L13,40
            M 28,40 L48,40
            M 95,40 L198,40" />
      </svg>
      <svg class="left-line">
        <path d="M 8,8 L8,32" />
      </svg>
      <svg class="right-line">
        <path d="M 128,16 L128,40" />
      </svg>
      <span>Скачать</span>
    </button>
  </div>
</template>

<script>
import { PDFDocument, rgb, StandardFonts } from "pdf-lib";
import download from "downloadjs";
export default {
  data() {
    return {
      compName: null,
      tempFile: null,
      reader: new FileReader(),
      urlPdfDark: "pdf/master_dark.pdf",
      urlPdfLight: "pdf/master_light.pdf"
    };
  },

  methods: {
    onChange(ev) {
      this.tempFile = ev.target.files[0];
    },

    onDownload() {
      if (this.checkValidName()) {
        this.reader.readAsArrayBuffer(this.tempFile);
      } else {
        console.log("Invalid");
      }
    },

    createNewPdf(ev) {
      const imageBytes = ev.target.result;
      this.addNameAndLogo(imageBytes, this.urlPdfDark, this.compName);
      this.addNameAndLogo(imageBytes, this.urlPdfLight, this.compName);
    },

    async addNameAndLogo(imageBytes, urlPdf, name) {
      const existingPdfBytes = await fetch(urlPdf).then(res =>
        res.arrayBuffer()
      );
      const pdfDoc = await PDFDocument.load(existingPdfBytes);

      // add company logo
      let image = null;
      if (this.tempFile.type == "image/jpeg") {
        image = await pdfDoc.embedJpg(imageBytes);
      } else if (this.tempFile.type == "image/png") {
        image = await pdfDoc.embedPng(imageBytes);
      }

      const pages = pdfDoc.getPages();
      const firstPage = pages[0];
      const { width, height } = firstPage.getSize();

      firstPage.drawImage(image, {
        x: width / 2 - 144,
        y: height - 510 - 170 - 30,
        width: 288,
        height: 170
      });

      //add company name
      const helveticaFont = await pdfDoc.embedFont(StandardFonts.HelveticaBold);
      firstPage.drawText(name, {
        x: width / 2 - (name.length / 2) * 25,
        y: height - 510,
        size: 48,
        font: helveticaFont,
        color: rgb(0.33, 0.38, 0.74)
      });

      // save pdf
      const pdfBytes = await pdfDoc.save();

      if (urlPdf == this.urlPdfDark) {
        download(pdfBytes, "commercial_dark.pdf", "application/pdf");
      } else {
        download(pdfBytes, "commercial_light.pdf", "application/pdf");
      }
      // this.openNewWindow(pdfBytes);
    },

    openNewWindow(pdfBytes) {
      const pdfUrl = URL.createObjectURL(
        new Blob([pdfBytes], { type: "application/pdf" })
      );
      window.open(pdfUrl, "_blank");
    },

    checkValidName() {
      const regex = /[^a-zA-Z0-9\s]{1,}/;
      if (regex.exec(this.compName)) {
        return false;
      }
      return true;
    }
  },

  computed: {
    isDownload() {
      if (this.tempFile && this.compName) return false;
      return true;
    }
  },

  mounted() {
    this.reader.addEventListener("load", this.createNewPdf);
  }
};
</script>

<style scoped>
.main {
  width: 500px;
  height: 200px;
  margin: 0 auto;
  padding: 10px 0 0 0;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: space-around;
}

/* Name block */
.name-container {
  position: relative;
  width: 500px;
  display: flex;
  flex-direction: row-reverse;
  align-items: center;
  justify-content: center;
  overflow: hidden;
}

.name-container label {
  position: relative;
  color: #5561bd;
  line-height: 40px;
  width: 210px;
  overflow: hidden;
}

.name-container label span {
  text-transform: uppercase;
  font-weight: 300;
  font-size: 12px;
  letter-spacing: 0.16em;
}

.name-left-line,
.name-bottom-line {
  position: absolute;
  top: 0;
  left: 0;
  transition: all 0.2s linear;
}

.name-left-line {
  top: -45px;
}

.name-left-line,
.name-bottom-line path {
  stroke-width: 1px;
  stroke: #5561bd;
}

.name-container input {
  background: transparent;
  outline: none;
  height: 32px;
  border: none;
  caret-color: #a78733;
  color: #a78733;
  z-index: 10;
}

.input-right-line,
.input-bottom-line {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  transition: all 0.2s linear;
}

.input-bottom-line {
  opacity: 0;
}

.input-right-line,
.input-bottom-line path {
  stroke-width: 1px;
  stroke: #5561bd;
}

.name-container:hover .name-left-line {
  top: 17px;
}

.name-container:hover .name-bottom-line {
  opacity: 0;
}

.name-container:hover .input-right-line {
  top: -18px;
}

.name-container:hover .input-bottom-line {
  opacity: 1;
}

.name-container input:focus ~ label > .name-left-line {
  top: -9px;
}

.name-container input:focus ~ label > .name-bottom-line {
  opacity: 0;
}

.name-container input:focus ~ .input-right-line {
  top: -47px;
}

.name-container input:focus ~ .input-bottom-line {
  opacity: 1;
}

/* Upload block */
.upload-container input {
  display: none;
}

.upload-container label {
  position: relative;
  line-height: 24px;
  width: 200px;
  display: flex;
  flex-direction: row;
  align-items: center;
  justify-content: center;
  border: none;
  cursor: pointer;
}

.upload-container label span {
  letter-spacing: 0.16em;
  text-transform: uppercase;
  font-weight: 300;
  font-size: 12px;
  color: #5561bd;
}

.upload-bottom-line {
  position: absolute;
  width: 100%;
  height: 100%;
  top: 0;
  left: 0;
}

.upload-bottom-line path {
  stroke: #5561bd;
  stroke-width: 1px;
}

.icon {
  margin-left: 10px;
  width: 12px;
  height: 12px;
  background-position: center;
  background-repeat: no-repeat;
  background-size: cover;
  opacity: 0.6;
}

.sel {
  background-image: url("../assets/image/check-solid.svg");
}

.no-sel {
  background-image: url("../assets/image/times-solid.svg");
}

/* Download button */
.btn-download {
  position: relative;
  width: 136px;
  height: 48px;
  color: #a78733;
  outline: none;
  border: none;
  overflow: hidden;
  background-color: transparent;
  text-transform: uppercase;
  letter-spacing: 0.16em;
}

.top-line,
.bottom-line,
.left-line,
.right-line {
  position: absolute;
  top: 0;
  left: 0;
  transition: all 0.2s linear;
}

.bottom-line {
  left: -87px;
}

.top-line path,
.left-line path,
.right-line path,
.bottom-line path {
  stroke: #a78733;
  stroke-width: 1px;
}

.btn-download:hover:enabled > .top-line path,
.btn-download:hover:enabled > .left-line path,
.btn-download:hover:enabled > .right-line path,
.btn-download:hover:enabled > .bottom-line path {
  stroke: #a7925b;
}

.btn-download:disabled > .top-line path,
.btn-download:disabled > .left-line path,
.btn-download:disabled > .right-line path,
.btn-download:disabled > .bottom-line path {
  stroke: transparent;
}

.btn-download:hover > .top-line {
  left: -87px;
}

.btn-download:hover > .bottom-line {
  left: 0px;
}

.btn-download:hover > .left-line {
  top: -8px;
}

.btn-download:hover > .right-line {
  top: 8px;
}

.btn-download:hover:enabled {
  color: #a7925b;
}

.btn-download:disabled {
  opacity: 0.2;
}
</style>
