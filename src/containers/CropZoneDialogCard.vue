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
        text=" One of images look to be blurred. You can got back and re-upload them,
          if you want."
        closable
        close-label="Close Alert"
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
      <div>
        <v-switch
          v-model="switchModel"
          hide-details
          inset
          :label="`${
            switchModel ? 'Disable' : 'Enable'
          } personal information detection`"
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
        @change="saveCropperResults"
        :src="generateURL(selectedImage)"
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
      selectedIndex: 0,
      cropperData: null,
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
  },
  methods: {
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


      console.log("index", index);
      console.log("this.files.length", this.files.length);

      // if index is last, call uploadFiles
      if (index === this.files.length - 1) {
        console.log("uploading files");
        this.$props.uploadFiles();
      } else if (this.$props.cropperResults[index] !== null) {
        this.$props.changeSelectedImage(this.files[index + 1]);
        this.$refs.cropper.setCoordinates({
          x: this.$props.cropperResults[index + 1].x,
          y: this.$props.cropperResults[index + 1].y,
          width: this.$props.cropperResults[index + 1].width,
          height: this.$props.cropperResults[index + 1].height,
        });
      } else if (index < this.files.length - 1) {
        this.$props.changeSelectedImage(this.files[index + 1]);
        this.$refs.cropper.setCoordinates({
          x: this.$props.croppedApiResults[index + 1].xmax,
          y: this.$props.croppedApiResults[index + 1].ymax,
          width:
            this.$props.croppedApiResults[index + 1].xmax -
            this.$props.croppedApiResults[index + 1].xmin,
          height:
            this.$props.croppedApiResults[index + 1].ymax -
            this.$props.croppedApiResults[index + 1].ymin,
        });
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

      if (this.$props.cropperResults[index] !== null) {
        this.$refs.cropper.setCoordinates({
          x: this.$props.cropperResults[index + 1].x,
          y: this.$props.cropperResults[index + 1].y,
          width: this.$props.cropperResults[index + 1].width,
          height: this.$props.cropperResults[index + 1].height,
        });
      } else {
        this.$refs.cropper.setCoordinates({
          x: this.$props.croppedApiResults[index].xmax,
          y: this.$props.croppedApiResults[index].ymax,
          width:
            this.$props.croppedApiResults[index].xmax -
            this.$props.croppedApiResults[index].xmin,
          height:
            this.$props.croppedApiResults[index].ymax -
            this.$props.croppedApiResults[index].ymin,
        });
      }
    },
    changeCropper() {
      if (this.switchModel) {
        let index = this.$props.files.indexOf(this.selectedImage);
        console.log(index);
        return this.changeSelectedImage(this.$props.detectronFiles[index]);
      } else {
        let index = this.$props.detectronFiles.indexOf(this.selectedImage);
        if (index === -1) {
          index = this.$props.files.indexOf(this.selectedImage);
        }
        return this.changeSelectedImage(this.$props.files[index]);
      }
    },
  },
  mounted() {
    if (this.$props.croppedApiResults.length != 0) {
      // if any of the images are blurred, show the alert
      let filteredBlurred = this.$props.croppedApiResults.filter(
        (item) => item.isBlur === true
      );

      if (filteredBlurred.length > 0) {
        this.isBlurred = true;
      }

      this.$refs.cropper.setCoordinates({
        x: this.$props.croppedApiResults[0].xmax,
        y: this.$props.croppedApiResults[0].ymax,
        width:
          this.$props.croppedApiResults[0].xmax -
          this.$props.croppedApiResults[0].xmin,
        height:
          this.$props.croppedApiResults[0].ymax -
          this.$props.croppedApiResults[0].ymin,
      });
    }

    if (this.$props.detectronFiles[0] !== undefined){
      this.changeSelectedImage(this.$props.detectronFiles[0]);
    }
  },
};
</script>
