<template>
  <v-card-title> Upload your images </v-card-title>
  <div>
  <v-checkbox v-model="applyDetectron" label="Use Detectron to block out PII"></v-checkbox>
  </div>
  <div>
    <v-btn variant="outlined" elevation="1" @click="toggleCapturePhotoDialogModal">Take a Photo with Device Camera</v-btn>
  </div>
  <CapturePhotoDialogComponent v-if="capturePhotoDialogModal"/>
  <br>
  OR
  <br>
  <div
    class="dropzone-container"
    @dragover="dragover"
    @dragleave="dragleave"
    @drop="drop"
  >
    <input
      type="file"
      multiple
      name="file"
      id="fileInput"
      class="hidden-input"
      @change="onChange"
      ref="file"
      accept=".pdf,.jpg,.jpeg,.png"
    />
    <label for="fileInput" class="file-label">
      <div v-if="isDragging">Release to drop files here.</div>
      <div v-else>Drop files here or <u>click here</u> to upload.</div>
    </label>
    <div class="preview-container" v-if="files.length">
      <div class="img-container" v-for="file in files" :key="file.name">
        <div>
          <button
            class="close-img-btn"
            type="button"
            @click="remove(files.indexOf(file))"
            title="Remove file"
          >
            <v-icon color="white"> mdi-close </v-icon>
          </button>
        </div>
        <div>
          <img class="preview-img" :src="generateURL(file)" />
        </div>
      </div>
    </div>
  </div>

  <v-card-actions>
    <div class="modal-actions d-flex">
      <div class="d-flex justify-center action-btn-div">
        <v-btn color="info" @click="closeModal">Cancel</v-btn>
      </div>
      <div class="d-flex justify-center action-btn-div">
        <!-- <v-btn
          color="info"
          @click="showCropScreen(this.applyDetectron)"
          :disabled="files.length === 0"
          >Crop Images</v-btn
        > -->
        <v-btn
            color="info"
            @click="showMasksScreen(this.applyDetectron)"
            :disabled="files.length === 0"
            >Show Masks</v-btn
          >
      </div>
    </div>
  </v-card-actions>
</template>

<script>
import CapturePhotoDialogComponent from "../components/CapturePhotoDialogComponent.vue";

export default {
  name: "UploadZoneDialogCard",
  components: {
    CapturePhotoDialogComponent,
  },
  props: {
    closeModal: {
      type: Function,
      required: true,
    },
    croppedApiResults: {
      type: Array,
      required: true,
    },
    showCropScreen: {
      type: Function,
      required: true,
    },
    showMasksScreen: {
      type: Function,
      required: true,
    },
    generateURL: {
      type: Function,
      required: true,
    },
    files: {
      type: Array,
      required: true,
    },
    remove: {
      type: Function,
      required: true,
    },
    changeSelectedImage: {
      type: Function,
      required: true,
    },
  },
  data() {
    return {
      isDragging: false,
      applyDetectron: false,
      capturePhotoDialogModal: false,
    };
  },
  methods: {
    onChange() {
      for (let i = 0; i < this.$refs.file.files.length; i++) {
        this.$props.files.push(this.$refs.file.files[i]);
      }
      console.log(this.$props.files);
      // emit the first file to the parent
      this.$props.changeSelectedImage(this.$props.files[0]);
    },
    dragover(e) {
      e.preventDefault();
      this.isDragging = true;
    },
    dragleave(e) {
      e.preventDefault();
      this.isDragging = false;
    },
    drop(e) {
      e.preventDefault();
      this.$refs.file.files = e.dataTransfer.files;
      this.onChange();
      this.isDragging = false;
    },
    toggleCapturePhotoDialogModal() {
      this.capturePhotoDialogModal = !this.capturePhotoDialogModal;
      console.log("this.captureDialogModal", this.capturePhotoDialogModal);
    },
  },
};
</script>
