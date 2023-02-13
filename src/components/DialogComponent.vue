<template>
  <v-dialog v-model="modal" width="unset" style="max-width: 80vw">
    <template v-slot:activator="{ props }">
      <div class="upload-button">
        <v-btn
          outlined
          color="info"
          variant="outlined"
          v-bind="props"
          @click="openModal"
        >
          <v-icon right light> mdi-image </v-icon>
          INSERT IMAGE
        </v-btn>
      </div>
    </template>

    <!-- have different cards depending on cropImagesPage -->
    <v-card class="v-card" v-if="cropImagesPage">
      <CropZoneDialogCard
        :change="change"
        :changeSelectedImage="changeSelectedImage"
        :closeModal="closeModal"
        :croppedImages="croppedImages"
        :files="files"
        :generateURL="generateURL"
        :model="model"
        :selectedImage="selectedImage"
        :uploadFiles="uploadFiles"
      />
    </v-card>
    <v-card class="v-card" v-else>
      <UploadZoneDialogCard
        :changeSelectedImage="changeSelectedImage"
        :closeModal="closeModal"
        :files="files"
        :generateURL="generateURL"
        :remove="remove"
        :showCropScreen="showCropScreen"
      />
    </v-card>
  </v-dialog>
</template>

<script>
import CropZoneDialogCard from "../containers/CropZoneDialogCard.vue";
import UploadZoneDialogCard from "../containers/UploadZoneDialogCard.vue";

export default {
  components: {
    CropZoneDialogCard,
    UploadZoneDialogCard,
  },
  data() {
    return {
      cropImagesPage: false,
      isDragging: false,
      model: null,
      files: [],
      selectedImage: null, // have selected image as first file or null
      modal: false,
      croppedImages: [], // object to store cropped images
    };
  },
  methods: {
    remove(i) {
      this.files.splice(i, 1);
    },
    generateURL(file) {
      if (!file) { // check if it is empty before generating url
        return;
      }
      let fileSrc = URL.createObjectURL(file);
      setTimeout(() => {
        let fileSrc = URL.createObjectURL(file);
        URL.revokeObjectURL(fileSrc);
      }, 1000);
      return fileSrc;
    },
    openModal() {
      this.modal = true;
    },
    closeModal() {
      this.cropImagesPage = false;
      this.croppedImages = [];
      this.files = [];
      this.modal = false;
      this.selectedImage = null;
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
    uploadFiles() {
      this.croppedImages.forEach((image) => {
        // eslint-disable-next-line no-undef
        tinymce.activeEditor.insertContent(
          `<img src="${image}" width="30%" alt="cropped image" />`
        );
      });
      this.closeModal();
    },
  },
};
</script>
