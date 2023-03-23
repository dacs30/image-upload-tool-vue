<template>
  <v-card-title>
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
        <v-slide-group-item v-for="file in files" :key="file.name">
          <div :class="{
            'img-container ma-1': true,
            selected: file === selectedImage,
          }" @click="clickedImage(file)">
            <div>
              <img class="slide-img-preview" :src="generateURL(file)" />
            </div>
          </div>
        </v-slide-group-item>
      </v-slide-group>
    </div>
    <div class="cropzone">
      <cropper class="cropper" ref="cropper" :src="generateURL(selectedImage)" @change="change" />
    </div>
  </div>

  <v-card-actions>
    <div class="modal-actions d-flex">
      <div class="d-flex justify-center action-btn-div">
        <v-btn color="info" block @click="closeModal">Cancel</v-btn>
      </div>
      <div v-if="files.indexOf(selectedImage) !== files.length - 1" class="d-flex justify-center action-btn-div">
        <v-btn color="info" block @click="nextFile">Next</v-btn>
      </div>
      <div v-else class="d-flex justify-center action-btn-div">
        <v-btn color="info" block @click="nextFile" :disabled="files.length === 0">Save</v-btn>
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
    generateURL: {
      type: Function,
      required: true,
    },
    change: {
      type: Function,
      required: true,
    },
    closeModal: {
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
  },
  methods: {
    nextFile() {
      const index = this.$props.files.indexOf(this.selectedImage);

      const { canvas } = this.$refs.cropper.getResult();

      this.$props.croppedImages.push(canvas.toDataURL());

      // if index is last, call uploadFiles
      if (index === this.files.length - 1) {
        this.$props.uploadFiles();
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
      this.$props.changeSelectedImage(file);
      // get index of the file
      const index = this.$props.files.indexOf(file);
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
  },
};
</script>
