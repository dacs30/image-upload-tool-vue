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
      <MasksZoneDialogCard
        v-else-if="predMasksPage"
        :goBackFromMasksPage="goBackFromMasksPage" 
        :close-modal="closeModal" 
        :files="files"
        :detectronFiles="detectronFiles"
        :generateURL="generateURL"
        :predMasksList="predMasksList"
        :loadAndMaskImage="loadAndMaskImage"
        :goFromPredMasksPageToCropImagesPage="goFromPredMasksPageToCropImagesPage"
      />
      <UploadZoneDialogCard
        v-else
        :changeSelectedImage="changeSelectedImage"
        :closeModal="closeModal"
        :files="files"
        :generateURL="generateURL"
        :remove="remove"
        :showCropScreen="showCropScreen"
        :showMasksScreen="showMasksScreen"
        :croppedApiResults="croppedApiResults"
        :dataURLToFile="dataURLToFile" 
      />
    </v-card>
  </v-dialog>
</template>

<script>
import CropZoneDialogCard from "../containers/CropZoneDialogCard.vue";
import MasksZoneDialogCard from "../containers/MasksZoneDialogCard.vue";
import UploadZoneDialogCard from "../containers/UploadZoneDialogCard.vue";
import axios from "axios";

export default {
  components: {
    CropZoneDialogCard,
    UploadZoneDialogCard,
    MasksZoneDialogCard
},
  data() {
    return {
      cropImagesPage: false,
      predMasksPage: false,
      isDragging: false,
      model: null,
      files: [],
      selectedImage: null, // have selected image as first file or null
      detectronFiles: [],
      predMasksList: [],
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
    goBackFromMasksPage() {
      this.predMasksPage = false;
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
      this.predMasksPage = false;
      this.predMasksList = [];
    },
    async showMasksScreen(applyDetectron) {
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
          this.croppedApiResults.push(response.data);

          // Get the prediction masks for the current image
          let detectron_pred_masks = response.data.detectron_pred_masks;

          // Get the original image file
          const originalImageFile = this.files[i];

          //generate visibilityList
          let visibilityList = [];
          let numMasks = Object.keys(detectron_pred_masks).length;
          for (let i = 0; i < numMasks; i++) {
            visibilityList.push(true);
          }

          // Load and mask the image
          const maskedFile = await this.loadAndMaskImage(originalImageFile, detectron_pred_masks, visibilityList);

          // Check if there is a file and push it to the detectronFiles array
          if (maskedFile) {
            this.detectronFiles.push(maskedFile);
            this.predMasksList.push(detectron_pred_masks);
          } else {
            this.detectronFiles.push(originalImageFile);
          }
        } catch (error) {
          console.log(error);
        }
      }
      this.isLoading = false;
      this.predMasksPage = !this.predMasksPage;
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

          //generate visibilityList
          let visibilityList = [];
          let numMasks = Object.keys(detectron_pred_masks).length;
          for (let i = 0; i < numMasks; i++) {
            visibilityList.push(true);
          }

          // Load and mask the image
          const maskedFile = await this.loadAndMaskImage(originalImageFile, detectron_pred_masks, visibilityList);

          // Check if there is a file and push it to the detectronFiles array
          if (maskedFile) {
            this.detectronFiles.push(maskedFile);
            this.predMasksList.push(detectron_pred_masks);
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
    async loadAndMaskImage(originalImageFile, predMasks, maskVisibilityList) {
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

          //get the number of masks
          let numMasks = Object.keys(predMasks).length;
          // Iterate over the predMasks array
          for (let i = 0; i < numMasks; i++) {
            //check if the mask is visible
            if(!maskVisibilityList[i]) {
              //skip the current mask if it is not visible
              continue;
            }
            //get the height
            let numPixelsHeight = Object.keys(predMasks[i]).length;
            //iterate over the height
            for(let j = 0; j < numPixelsHeight; j++) {
              //get the width
              let numPixelsWidth = Object.keys(predMasks[i][j]).length;
              //iterate over the width
              for(let k = 0; k < numPixelsWidth; k++) {
                let currentPixel = predMasks[i][j][k];
                //check if current pixel is true (there is a mask there)
                if (currentPixel) {
                  // Calculate the x and y coordinates of the current pixel
                  let x = k; //k iterates over the width
                  let y = j; //j iterates over the height

                  // Set the color for the masked portion from color list
                  let maskColorList = ["black", "red", "green", "blue", "yellow", "purple", "orange", "gray"];
                  context.fillStyle = maskColorList[i % maskColorList.length];

                  // Draw a rectangle for the masked pixel
                  context.fillRect(x, y, 1, 1);
                }
              }
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
    goFromPredMasksPageToCropImagesPage(updatedDetectronFiles) {
      this.detectronFiles = updatedDetectronFiles;
      this.predMasksPage = !this.predMasksPage;
      this.cropImagesPage = !this.cropImagesPage;
    },
  },
};
</script>
