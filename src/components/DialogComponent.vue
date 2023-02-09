<template>
  <v-dialog v-model="modal" width="unset" style="max-width: 80vw">
    <template v-slot:activator="{ props }">
      <div class="upload-button">
        <v-btn outlined color="primary" v-bind="props" @click="openModal">
          <v-icon right light> mdi-image </v-icon>
          INSERT IMAGE
        </v-btn>
      </div>
    </template>

    <v-card class="v-card">
      <v-card-title v-if="cropImagesPage">
        <!-- have the title with a back arrow on the left -->
        <v-row class="v-card-title-component">
          <v-col class="back-btn" cols="4">
            <v-icon @click="cropImagesPage = false"> mdi-arrow-left </v-icon>
          </v-col>
          <v-col cols="4">
            <div class="cropzone-title">Crop your images</div>
          </v-col>
          <v-col class="close-btn" cols="4" align="right">
            <v-icon @click="closeModal"> mdi-close </v-icon>
          </v-col>
        </v-row>
      </v-card-title>
      <v-card-title v-else> Upload your images </v-card-title>

      <div class="cropzone-container" v-if="cropImagesPage">
        <div class="img-slider mx-auto" elevation="8">
          <v-slide-group v-model="model" class="pa-2" center-active show-arrows>
            <v-slide-group-item v-for="file in files" :key="file.name">
              <div :class="{
                'img-container ma-1': true,
                selected: file === selectedImage,
              }" @click="changeSelectedImage(file)">
                <div>
                  <img class="slide-img-preview" :src="generateURL(file)" />
                </div>
              </div>
            </v-slide-group-item>
          </v-slide-group>
        </div>
        <div class="cropzone">
          <cropper ref="cropper" :src="generateURL(selectedImage)" @change="change" />
        </div>
      </div>

      <div v-else class="dropzone-container" @dragover="dragover" @dragleave="dragleave" @drop="drop">
        <input type="file" multiple name="file" id="fileInput" class="hidden-input" @change="onChange" ref="file"
          accept=".pdf,.jpg,.jpeg,.png" />
        <label for="fileInput" class="file-label">
          <div v-if="isDragging">Release to drop files here.</div>
          <div v-else>Drop files here or <u>click here</u> to upload.</div>
        </label>
        <div class="preview-container" v-if="files.length">
          <div class="img-container" v-for="file in files" :key="file.name">
            <div>
              <button class="close-img-btn" type="button" @click="remove(files.indexOf(file))" title="Remove file">
                <v-icon color="white"> mdi-close </v-icon>
              </button>
            </div>
            <div>
              <img class="preview-img" :src="generateURL(file)" />
            </div>
          </div>
        </div>
      </div>

      <v-card-actions v-if="cropImagesPage">
        <div class="modal-actions d-flex">
          <div class="d-flex justify-center action-btn-div">
            <v-btn color="primary" block @click="closeModal">Cancel</v-btn>
          </div>
          <div v-if="files.indexOf(selectedImage) !== files.length - 1" class="d-flex justify-center action-btn-div">
            <v-btn color="primary" block @click="nextFile">Next</v-btn>
          </div>
          <div v-else class="d-flex justify-center action-btn-div">
            <v-btn color="primary" block @click="nextFile" :disabled="files.length === 0">Save</v-btn>
          </div>
        </div>
      </v-card-actions>
      <v-card-actions v-else>
        <div class="modal-actions d-flex">
          <div class="d-flex justify-center action-btn-div">
            <v-btn color="primary" @click="closeModal">Cancel</v-btn>
          </div>
          <div class="d-flex justify-center action-btn-div">
            <v-btn color="primary" @click="showCropScreen" :disabled="files.length === 0">Crop Images</v-btn>
          </div>
        </div>
      </v-card-actions>
    </v-card>
  </v-dialog>
</template>

<script>
import "vue-advanced-cropper/dist/style.css";
import { Cropper } from "vue-advanced-cropper";

export default {
  components: {
    Cropper,
  },
  data() {
    return {
      cropImagesPage: false,
      isDragging: false,
      model: null,
      files: [],
      // have selected image as first file or null
      selectedImage: null,
      modal: false,
      // object to store cropped images
      croppedImages: [],
    };
  },
  methods: {
    onChange() {
      for (let i = 0; i < this.$refs.file.files.length; i++) {
        this.files.push(this.$refs.file.files[i]);
      }
      this.selectedImage = this.files[0];
    },
    dragover(e) {
      e.preventDefault();
      this.isDragging = true;
    },
    dragleave() {
      this.isDragging = false;
    },
    drop(e) {
      e.preventDefault();
      this.$refs.file.files = e.dataTransfer.files;
      this.onChange();
      this.isDragging = false;
    },
    remove(i) {
      this.files.splice(i, 1);
    },
    generateURL(file) {
      let fileSrc = URL.createObjectURL(file);
      setTimeout(() => {
        URL.revokeObjectURL(fileSrc);
      }, 1000);
      return fileSrc;
    },
    openModal() {
      this.modal = true;
    },
    closeModal() {
      // clear files array
      this.files = [];
      this.modal = false;
      this.cropImagesPage = false;
    },
    showCropScreen() {
      this.cropImagesPage = !this.cropImagesPage;
    },
    changeFile() {
      const result = this.$refs.cropper.getResult();

      console.log(this.$refs.cropper.getResult());

      this.files = [result.image.src];

      this.closeModal();
      this.cropImagesPage = !this.cropImagesPage;
    },
    changeSelectedImage(file) {
      this.selectedImage = file;
    },
    croppedCanvasReceived(cropData) {
      const { canvas } = cropData;
      this.croppedImages.push(canvas.toDataURL());
    },
    nextFile() {
      const index = this.files.indexOf(this.selectedImage);

      const { canvas } = this.$refs.cropper.getResult();

      this.croppedImages.push(canvas.toDataURL());

      // if index is last, call uploadFiles
      if (index === this.files.length - 1) {
        this.uploadFiles();
      } else if (index < this.files.length - 1) {
        this.changeSelectedImage(this.files[index + 1]);
      }
    },
    uploadFiles() {
      this.croppedImages.forEach((image) => {
        // eslint-disable-next-line no-undef
        tinymce.activeEditor.insertContent(
          `<img src="${image}" alt="cropped image" />`
        );
      });
      this.closeModal();
    },
  },
};
</script>
