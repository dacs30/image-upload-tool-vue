<template>
    <v-card-title>
        <!-- have the title with a back arrow on the left -->
        <v-row class="v-card-title-component">
            <v-col class="back-btn" cols="4">
                <v-icon @click="goBackFromMasksPage"> mdi-arrow-left </v-icon>
            </v-col>
            <v-col cols="4">
                <div class="cropzone-title">Select/Deselect Masks</div>
            </v-col>
            <v-col class="close-btn" cols="4" align="right">
                <v-icon @click="closeModal"> mdi-close </v-icon>
            </v-col>
        </v-row>
    </v-card-title>

    <div class="parent-container">
        <img class="max-width-img" @click="handleImageClick" :src="generateURL(detectronFiles[selectedIndex])" ref="image"/>
    </div>

    <v-card-actions>
        <div class="modal-actions d-flex">
            <div class="d-flex justify-center action-btn-div">
                <v-btn color="info" block @click="closeModal">Cancel</v-btn>
            </div>
            <div v-if="selectedIndex !== files.length - 1" class="d-flex justify-center action-btn-div">
                <v-btn color="info" block @click="nextFile">Next</v-btn>
            </div>
            <div v-else class="d-flex justify-center action-btn-div">
                <v-btn color="info" block @click="nextFile" :disabled="files.length === 0">Save</v-btn>
            </div>
        </div>
    </v-card-actions>
</template>

<style>
.parent-container {
  max-width: 100%;
}

.max-width-img {
  max-width: 100%;
  height: auto;
}</style>

<script>
import "vue-advanced-cropper/dist/style.css";
import { Cropper } from "vue-advanced-cropper";

export default {
    name: "MaskZoneDialogCard",
    data() {
        return {
            selectedIndex: 0,
            maskVisibilityList: [],
        };
    },
    props: {
        files: {
            type: Array,
            required: true,
        },
        detectronFiles: {
            type: Array,
            required: true,
        },
        predMasksList: {
            type: Array,
            required: true,
        },
        goBackFromMasksPage: {
            type: Function,
            required: true,
        },
        closeModal: {
            type: Function,
            required: true,
        },
        generateURL: {
            type: Function,
            required: true,
        },
        loadAndMaskImage: {
            type: Function,
            required: true,
        },
    },
    methods: { 
      handleImageClick(event) {
      const imageElement = this.$refs.image;

      const image = new Image();
      image.src = this.generateURL(this.files[this.selectedIndex]);


      image.onload = () => {
        const rect = imageElement.getBoundingClientRect(); // Get the position of the image relative to the viewport

        const x = event.clientX - rect.left; // Calculate the X coordinate relative to the image
        const y = event.clientY - rect.top; // Calculate the Y coordinate relative to the image

        const pixelX = Math.round((x / rect.width) * image.naturalWidth); // Calculate the pixel X coordinate
        const pixelY = Math.round((y / rect.height) * image.naturalHeight); // Calculate the pixel Y coordinate

        console.log(`Clicked pixel position (X, Y): (${pixelX}, ${pixelY})`);
        this.updateMasks(pixelX, pixelY);
      };
    },
    async updateMasks(pixelX, pixelY) {
      //first time running, initialize the maskVisibilityList
      if (this.maskVisibilityList.length === 0) {
        let currentPredMask = this.predMasksList[this.selectedIndex];
        let numMasks = Object.keys(currentPredMask).length;
        for (let i = 0; i < numMasks; i++) {
          this.maskVisibilityList.push(true);
        }
      }
      let currentPredMask = this.predMasksList[this.selectedIndex];
      let numMasks = Object.keys(currentPredMask).length;
      //looping through the masks in reverse order to get the topmost mask
      for (let i = numMasks - 1; i >= 0; i--) {
        if (currentPredMask[i][pixelY][pixelX] === true) {
          console.log("clicked on mask " + i);
          this.maskVisibilityList[i] = !this.maskVisibilityList[i];
          const maskedFile = await this.loadAndMaskImage(this.files[this.selectedIndex], currentPredMask, this.maskVisibilityList);
          //change the img element to the masked image
          const imageElement = this.$refs.image;
          if (imageElement) {
            imageElement.src = this.generateURL(maskedFile);
          }
          break;
        }
      }
    }
  }
};
</script>