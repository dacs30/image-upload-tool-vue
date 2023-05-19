<template>
  <v-card-title>
    <!-- have the title with a back arrow on the left -->
    <v-row class="v-card-title-component">
      <v-col class="back-btn" cols="4">
        <v-icon @click="goBackFromCroppedPages"> mdi-arrow-left </v-icon>
      </v-col>
      <v-col cols="4">
        <div class="cropzone-title">Crop your images</div>
      </v-col>
      <v-col class="close-btn" cols="4" align="right">
        <v-icon @click="closeModal"> mdi-close </v-icon>
      </v-col>
    </v-row>
  </v-card-title>

  <div class="cropzone-container">
    <div class="blur-alert">
      <v-alert
        type="info"
        v-model="isBlurred"
        variant="tonal"
        text=" One or more of your images looks to be blurred. You can go back and re-upload them, if you want."
        color="info"
      >
      </v-alert>
    </div>
    <div class="img-slider mx-auto" elevation="8">
      <v-slide-group v-model="model" class="pa-2" center-active show-arrows>
        <v-slide-group-item v-for="(file, index) in files" :key="file.name">
          <div
            :class="{
              'img-container ma-1': true,
              selected: index === selectedIndex,
            }"
            @click="clickedImage(file)"
          >
            <div>
              <img class="slide-img-preview" :src="generateURL(file)" />
            </div>
          </div>
        </v-slide-group-item>
      </v-slide-group>
    </div>
    <div class="d-flex justify-center">
      <v-btn color="info"
        @click="manualCrop"
        variant="outlined"
        >Manual Crop
        <v-tooltip
          activator="parent"
          location="bottom"
        >Middle click and drag to create a box.</v-tooltip>
      </v-btn>
      <div class="pr-2"></div>
      <v-btn
        color="info"
        @click="clearCrop"
        variant="outlined"
        
      >Clear
      <v-tooltip
          activator="parent"
          location="bottom"
        >Clear the manual crop you made.</v-tooltip>
      </v-btn>
      <div class="pr-2"></div>
      <v-btn
        color="info"
        @click="resetToAutomaticCrop"
        variant="outlined"
      >Automatic Crop
      <v-tooltip
          activator="parent"
          location="bottom"
        >Load the automatic crop.</v-tooltip>
      </v-btn>
    </div>
    <div class="d-flex justify-center">
      <div>
        <v-switch
          v-model="switchModel"
          v-if="detectronEnabled"
          hide-details
          inset
          :label="`Personal information detection: ${
            switchModel ? 'Enabled' : 'Disabled'
          }`"
          @change="changeCropper"
        ></v-switch>
      </div>
    </div>
    <div class="cropzone">
      <!-- If for some reason you don't want to re render cropper, use v-once in this component -->
      <cropper
        :model="cropperModel"
        class="cropper"
        ref="cropper"
        @change="checkSaveCropperResults"
        @mousedown.middle="beginDrag"
        @mouseup.middle="endDrag"
        @mousemove="mouseMove"
        :src="generateURL(selectedImage)"
        id="cropper"
        :resizeImage="{ wheel: false }"
      />
    </div>
  </div>
  <v-card-actions>
    <div class="modal-actions d-flex">
      <div class="d-flex justify-center action-btn-div">
        <v-btn color="info" block @click="closeModal">Cancel</v-btn>
      </div>
      <div
        v-if="selectedIndex !== files.length - 1"
        class="d-flex justify-center action-btn-div"
      >
        <v-btn color="info" block @click="nextFile">Next</v-btn>
      </div>
      <div v-else class="d-flex justify-center action-btn-div">
        <v-btn
          color="info"
          block
          @click="nextFile"
          :disabled="files.length === 0"
          >Save</v-btn
        >
      </div>
    </div>
  </v-card-actions>
</template>

<script>
import "vue-advanced-cropper/dist/style.css";
import { Cropper } from "vue-advanced-cropper";
import axios from "axios";

export default {
  name: "CropZoneDialogCard",
  components: {
    Cropper,
  },
  data() {
    return {
      model: null,
      isBlurred: false,
      switchModel: true,
      selectedIndex: 0, // The index of the selected image
      cropperData: null,
      manualCropMode: false, // If the user is manually cropping
      cropperModel: null,
      origDragX: 0, // When manually cropping, the left x-coord of the box
      origDragY: 0, // When manually cropping, the top y-coord of the box
      manualCropping: false, // If the user is currently dragging the box
    };
  },
  props: {
    files: {
      type: Array,
      required: true,
    },
    selectedImage: {
      type: Object,
      required: true,
    },
    changeSelectedImage: {
      type: Function,
      required: true,
    },
    croppedApiResults: {
      type: Array,
      required: true,
    },
    cropperResults: {
      type: Object,
      required: true,
    },
    generateURL: {
      type: Function,
      required: true,
    },
    closeModal: {
      type: Function,
      required: true,
    },
    saveCropperResults: {
      type: Function,
      required: true,
    },
    uploadFiles: {
      type: Function,
      required: true,
    },
    croppedImages: {
      type: Array,
      required: true,
    },
    goBackFromCroppedPages: {
      type: Function,
      required: true,
    },
    detectronFiles: {
      type: Array,
      required: true,
    },
    detectronEnabled: {
      type: Boolean,
      required: true,
    },
  },
  methods: {
    /**
     * Sends a patch request to the API for each manual crop result.
     */
    async patchChanges() {
      for(var i = 0; i < this.$props.files.length; i++) {

        console.log("Sending patch request for file", i);

        if(this.$props.cropperResults[i] !== undefined) {
          // The patch request
          const response = await axios.patch("http://127.0.0.1:5000/",
          {
            "pKey": this.$props.croppedApiResults[i].id,
            "coords": { 
              "xMin": this.$props.cropperResults[i].left,
              "xMax": this.$props.cropperResults[i].left + this.$props.cropperResults[i].width,
              "yMin": this.$props.cropperResults[i].top,
              "yMax": this.$props.cropperResults[i].top + this.$props.cropperResults[i].height,
            }
          }).catch((e) => {
            console.log(e);
          });
        }
      }
      console.log("uploading files");
      this.$props.uploadFiles();
    },
    /**
     * Checks if the cropper results should be saved; if they should, saves them.
     * The cropper results will be saved if the default box has been modified and the user is in the "manual cropping" mode
     */
    checkSaveCropperResults() {
      // Check if the box has not been modified
      if(this.$refs.cropper.coordinates.left === 0
      && this.$refs.cropper.coordinates.top === 0
      && this.$refs.cropper.coordinates.width === 1
      && this.$refs.cropper.coordinates.height === 1) return;

      // Make sure the user is cropping manually
      if(!this.manualCropMode) return;

      this.saveCropperResults(this.$refs.cropper.coordinates, this.selectedIndex);
    },
    /**
     * Initializes everything needed for the click-and-drag functionality of the cropper box
     * @param {*} event The event. Passed by v-on.
     */
    beginDrag(event) {
      if(!this.manualCropMode) return;

      this.manualCropping = true;

      let cropBox = document.getElementById('cropper').getBoundingClientRect();

      this.origDragX = (event.clientX - cropBox.left) * this.$refs.cropper.imageSize.width / cropBox.width;
      this.origDragY = (event.clientY - cropBox.top) * this.$refs.cropper.imageSize.height / cropBox.height;

      this.$refs.cropper.setCoordinates({
        left: this.origDragX,
        top: this.origDragY,
        width: 1,
        height: 1
      });
    },
    /**
     * Updates the cropper box when  the mouse is moved and we are in the click-and-drag state
     * @param {*} event The event. Passed by v-on.
     */
    mouseMove(event) {
      if(this.manualCropping) {

        let cropBox = document.getElementById('cropper').getBoundingClientRect();

        this.$refs.cropper.setCoordinates({
          left: this.origDragX,
          top: this.origDragY,
          width: (event.clientX - cropBox.left) * (this.$refs.cropper.imageSize.width / cropBox.width) - this.origDragX,
          height: (event.clientY - cropBox.top) * (this.$refs.cropper.imageSize.height / cropBox.height) - this.origDragY,
        });
      }
      },
      /**
       * Finishes the click-and-drag cropper box. Sets the final coordinates of the box and saves them.
       * @param {*} event The event. Passed by v-on.
       */
    endDrag(event) {
      if(!this.manualCropMode) return;

      let cropBox = document.getElementById('cropper').getBoundingClientRect();

      this.$refs.cropper.setCoordinates({
        left: this.origDragX,
        top: this.origDragY,
        width: (event.clientX - cropBox.left) * (this.$refs.cropper.imageSize.width / 586) - this.origDragX,
        height: (event.clientY - cropBox.top) * (this.$refs.cropper.imageSize.width / 586) - this.origDragY,
      });

      this.saveCropperResults(this.$refs.cropper.coordinates, this.selectedIndex);

      this.manualCropping = false;

    },
    /**
     * Ititializes the manual crop mode when  the "manual crop" button is pressed. This means setting to cropper box coordinates to the saved coordinates, if there are any.
     */
    manualCrop() {
      if(this.manualCropMode) return;
      console.log("Entering manual crop mode.");
      this.manualCropMode = true;
      
      // There are saved cropper results
      if (this.$props.cropperResults[this.selectedIndex] !== undefined) {

        this.$refs.cropper.setCoordinates({
          left: this.$props.cropperResults[this.selectedIndex].left,
          top: this.$props.cropperResults[this.selectedIndex].top,
          width: this.$props.cropperResults[this.selectedIndex].width,
          height: this.$props.cropperResults[this.selectedIndex].height,
        });
      }
      // There are no saved cropper results
      else {
        this.$refs.cropper.setCoordinates({left: 0, top: 0, width: 1, height: 1});
      }
    },
    /**
     * Clears the cropper box and saves the results
     */
    clearCrop() {
      if(this.manualCropMode) {
        this.$refs.cropper.setCoordinates({left: 0, top: 0, width: 1, height: 1});
      }
      this.saveCropperResults(this.$refs.cropper.coordinates, this.selectedIndex);
    },
    /**
     * Changes the crop mode to automatic crop:
     * - Loads the coordinates that were passed by yolov7
     * Any changes in automatic crop mode will not be saved.
     */
    resetToAutomaticCrop() {

      this.$refs.cropper.setCoordinates({
          left: this.$props.croppedApiResults[this.selectedIndex].xmin,
          top: this.$props.croppedApiResults[this.selectedIndex].ymin,
          width:
            this.$props.croppedApiResults[this.selectedIndex].xmax -
            this.$props.croppedApiResults[this.selectedIndex].xmin,
          height:
            this.$props.croppedApiResults[this.selectedIndex].ymax -
            this.$props.croppedApiResults[this.selectedIndex].ymin,
        });

        console.log("Resetting to automatic cropping.", this.$props.croppedApiResults[this.selectedIndex]);

        this.manualCropMode = false;

    },
    nextFile() {
      let index = 0;

      if (this.switchModel) {
        if (this.$props.detectronFiles.indexOf(this.selectedImage) !== -1) {
          index = this.$props.detectronFiles.indexOf(this.selectedImage);
        } else {
          index = this.$props.files.indexOf(this.selectedImage);
        }
      } else {
        index = this.$props.files.indexOf(this.selectedImage);
      }

      const { canvas } = this.$refs.cropper.getResult();

      console.log("canvas", canvas);

      this.$props.croppedImages.push(canvas.toDataURL());

      this.selectedIndex = index + 1;

      // if index is last, call uploadFiles
      if (index === this.files.length - 1) {
        this.patchChanges();
      }
      else if (this.$props.cropperResults[index + 1] !== undefined || index < this.files.length - 1) {
        if (this.switchModel) {
        this.$props.changeSelectedImage(this.$props.detectronFiles[index + 1]);
      } else {
        this.$props.changeSelectedImage(this.$props.files[index + 1]);
      }
        this.resetToAutomaticCrop();
      }
    },
    clickedImage(file) {
      // get index of the file
      const index = this.$props.files.indexOf(file);

      const { canvas } = this.$refs.cropper.getResult();

      // if switch is true, change selected image to detectron file
      if (this.switchModel) {
        this.$props.changeSelectedImage(this.$props.detectronFiles[index]);
      } else {
        this.$props.changeSelectedImage(this.$props.files[index]);
      }

      this.selectedIndex = index;

      if (canvas) {
        this.$props.croppedImages.push(canvas.toDataURL());
      }

      this.resetToAutomaticCrop();
    },
    changeCropper() {
      if (this.switchModel) {
        let index = this.$props.files.indexOf(this.selectedImage);
        console.log("From switch", index);
        // check if there is a detectron file before changing selected image
        this.changeSelectedImage(this.$props.detectronFiles[index]);
      }
      else {
        let index = this.$props.detectronFiles.indexOf(this.selectedImage);
        if (index === -1) {
          index = this.$props.files.indexOf(this.selectedImage);
        }
        this.changeSelectedImage(this.$props.files[index]);
      }
      this.resetToAutomaticCrop();
    },
  },
  mounted() {
    if (this.$props.detectronFiles[0] !== undefined) {
      this.changeSelectedImage(this.$props.detectronFiles[0]);
    }

    if (this.$props.croppedApiResults.length != 0) {
      // if any of the images are blurred, show the alert
      let filteredBlurred = this.$props.croppedApiResults.filter(
        (item) => item.isBlur === true
      );

      if (filteredBlurred.length > 0) {
        this.isBlurred = true;
      }

      this.$refs.cropper.setCoordinates({
        left: this.$props.croppedApiResults[0].xmin,
        top: this.$props.croppedApiResults[0].ymin,
        width:
          this.$props.croppedApiResults[0].xmax -
          this.$props.croppedApiResults[0].xmin,
        height:
          this.$props.croppedApiResults[0].ymax -
          this.$props.croppedApiResults[0].ymin,
      });
    }
  },
};
</script>
