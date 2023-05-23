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
        :changeSelectedImage="changeSelectedImage"
        :cropperResults="cropperResults"
        :detectronFiles="detectronFiles"
        :saveCropperResults="saveCropperResults"
        :closeModal="closeModal"
        :croppedImages="croppedImages"
        :files="files"
        :generateURL="generateURL"
        :model="model"
        :selectedImage="selectedImage"
        :uploadFiles="uploadFiles"
        :croppedApiResults="croppedApiResults"
        :goBackFromCroppedPages="goBackFromCroppedPages"
      />
    </v-card>
    <v-card class="v-card" v-else>
      <!-- have a v-progreess-circulare if isLoading is true -->
      <div v-if="isLoading" class="progress-circular">
        <v-progress-circular
          :active="isLoading"
          indeterminate
          color="info"
        ></v-progress-circular>
      </div>
      <UploadZoneDialogCard
        v-else
        :changeSelectedImage="changeSelectedImage"
        :closeModal="closeModal"
        :files="files"
        :generateURL="generateURL"
        :remove="remove"
        :showCropScreen="showCropScreen"
        :croppedApiResults="croppedApiResults"
      />
    </v-card>
  </v-dialog>
</template>

<script>
import CropZoneDialogCard from "../containers/CropZoneDialogCard.vue";
import UploadZoneDialogCard from "../containers/UploadZoneDialogCard.vue";
import axios from "axios";

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
      detectronFiles: [],
      modal: false,
      croppedImages: [], // object to store cropped images
      croppedApiResults: [], // object to store cropped images
      isLoading: false,
      cropperResults: [],
    };
  },
  methods: {
    remove(i) {
      this.files.splice(i, 1);
      this.croppedApiResults.splice(i, 1);
    },
    goBackFromCroppedPages() {
      this.cropImagesPage = false;
    },
    generateURL(file) {
      if (!file) {
        console.log("no file", file);
        // check if it is empty before generating url
        return;
      }
      let fileSrc = URL.createObjectURL(file);
      setTimeout(() => {
        // Revoke the URL after one second
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
      this.croppedApiResults = [];
      this.detectronFiles = [];
    },
    async showCropScreen(applyDetectron) {
      // upload each of the files to the api
      this.isLoading = true;
      for (let i = 0; i < this.files.length; i++) {
        try {
          const formData = new FormData();
          formData.append("file", this.files[i]);
          applyDetectron ? formData.append("applyDetectron", 'true') : formData.append("applyDetectron", 'false');
          console.log(applyDetectron);
          const response = await axios({
            method: "post",
            url: "http://127.0.0.1:5000/", // replace with your API endpoint
            data: formData,
            headers: { "Content-Type": "multipart/form-data" },
          });
          console.log("Printing out the reposnse of the POST request below:");
          console.log(response.data);
          this.croppedApiResults.push(response.data);

          // Get the prediction masks for the current image
          let detectron_pred_masks = response.data.detectron_pred_masks;

          // Get the original image file
          const originalImageFile = this.files[i];

          // Load and mask the image
          const maskedFile = await this.loadAndMaskImage(originalImageFile, detectron_pred_masks);

          // Check if there is a file and push it to the detectronFiles array
          if (maskedFile) {
            this.detectronFiles.push(maskedFile);
          } else {
            this.detectronFiles.push(originalImageFile);
          }

          /*
          //unsure if neeeded
          //TODO: decide whether needed
          let byteCharacters = atob(response.data.detectron);
          let byteNumbers = new Array(byteCharacters.length);

          // for (let i = 0; i < byteCharacters.length; i++) {
          //   byteNumbers[i] = byteCharacters.charCodeAt(i);
          // }

          // let byteArray = new Uint8Array(byteNumbers);

          // let file = new File([byteArray], "detectron.png", {
          //   type: "image/png",
          // });

          // // check if there is a file, if there is not, then push the first file
          // if (file === undefined) {
          //   this.detectronFiles.push(this.files[i]);
          // } else {
          //   this.detectronFiles.push(file);
          // }
          */
        } catch (error) {
          console.log(error);
        }
      }
      this.isLoading = false;
      this.cropImagesPage = !this.cropImagesPage;
    },
    async loadAndMaskImage(originalImageFile, predMasks) {
      return new Promise((resolve, reject) => {
        const originalImage = new Image();

        // load event listener to handle the image loading
        originalImage.addEventListener("load", () => {
          // Create a new canvas element
          const canvas = document.createElement("canvas");
          const context = canvas.getContext("2d");

          // Set the canvas width and height to match the original image
          canvas.width = originalImage.width;
          canvas.height = originalImage.height;

          // Draw the original image on the canvas
          context.drawImage(originalImage, 0, 0);

          // Flatten the predMasks array to simplify the iteration
          const flattenedMasks = predMasks.flat(2);

          // Iterate over the flattenedMasks array to create the mask
          for (let i = 0; i < flattenedMasks.length; i++) {
            const mask = flattenedMasks[i];
            if (mask) {
              // Calculate the x and y coordinates of the current pixel
              const x = i % originalImage.width;
              const y = Math.floor(i / originalImage.width);

              // Set the color for the masked portion
              context.fillStyle = "rgba(255, 0, 0, 1)"; 

              // Draw a rectangle for the masked pixel
              context.fillRect(x, y, 1, 1);
            }
          }

          // Convert the canvas content to a data URL
          const maskedDataURL = canvas.toDataURL("image/png");

          // Create a new File object from the data URL
          const maskedFile = this.dataURLToFile(maskedDataURL, originalImageFile.name);

          // Resolve the Promise with the masked file
          resolve(maskedFile);
        });

        // Set the source of the original image to the URL.createObjectURL of the original image file
        // unsure if needed
        originalImage.src = URL.createObjectURL(originalImageFile);
      });
    },

    // Helper function to convert data URL to File object
    dataURLToFile(dataURL, filename) {
      const arr = dataURL.split(",");
      const mime = arr[0].match(/:(.*?);/)[1];
      const bstr = atob(arr[1]);
      let n = bstr.length;
      const u8arr = new Uint8Array(n);
      while (n--) {
        u8arr[n] = bstr.charCodeAt(n);
      }
      return new File([u8arr], filename, { type: mime });
    },
    changeSelectedImage(file) {
      this.selectedImage = file;
    },
    saveCropperResults(cropperResults) {
      // add cropper results to cropperResults array
      console.log(cropperResults);
      this.cropperResults.push(cropperResults);
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
