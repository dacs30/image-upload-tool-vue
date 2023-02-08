<template>
  <h1>{{ welcome }}</h1>
  <!-- add vuetify button -->
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
      <v-card-title v-if="cropImagesPage"> Crop your images </v-card-title>
      <v-card-title v-else> Upload your images </v-card-title>
      <div class="cropzone-container" v-if="cropImagesPage">
        <div class="img-slider mx-auto" elevation="8">
          <v-slide-group v-model="model" class="pa-4" center-active show-arrows>
            <v-slide-group-item v-for="file in files" :key="file.name" v-slot="{ isSelected, toggle }">
              <!-- add previews of the files -->
              <div class="img-container ma-2">
                <div>
                  <img class="slide-img-preview" :src="generateURL(file)" />
                </div>
              </div>
            </v-slide-group-item>
          </v-slide-group>
        </div>
        <div class="cropzone">
          <cropper ref="cropper" :src="generateURL(files[0])" @change="change" />
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
          <div class="d-flex justify-center action-btn-div">
            <v-btn color="primary" block @click="changeFile" :disabled="files.length === 0">Save</v-btn>
          </div>
        </div>
      </v-card-actions>
      <v-card-actions v-else>
        <div class="modal-actions d-flex">
          <div class="d-flex justify-center action-btn-div">
            <v-btn color="primary" @click="closeModal">Cancel</v-btn>
          </div>
          <div class="d-flex justify-center action-btn-div">
            <v-btn color="primary" @click="cropImages" :disabled="files.length === 0">Crop Images</v-btn>
          </div>
        </div>
      </v-card-actions>
    </v-card>
  </v-dialog>
  <div class="editor">
    <Editor class="editor" id="file-picker" :api-key="apiKey" :init="myInit" />
  </div>
</template>

<style scoped>
.editor {
  width: 50vw;
}
</style>

<script>
import Editor from "@tinymce/tinymce-vue";
import { Cropper } from "vue-advanced-cropper";
import "vue-advanced-cropper/dist/style.css";

export default {
  components: {
    Editor,
  },
  data() {
    return {
      cropImagesPage: false,
      isDragging: false,
      model: null,
      items: [1, 2, 3, 4, 5, 6, 7, 8, 9],
      files: [],
      modal: false,
      apiKey: import.meta.env.VITE_APP_TINYMCE_API_KEY,
      myInit: {
        selector: "textarea#file-picker",
        plugins: "image code",
        toolbar: "undo redo | link image | code",
        /* enable title field in the Image dialog*/
        image_title: true,
        /* enable automatic uploads of images represented by blob or data URIs*/
        automatic_uploads: true,
        /*
          URL of our upload handler (for more details check: https://www.tiny.cloud/docs/configure/file-image-upload/#images_upload_url)
          images_upload_url: 'postAcceptor.php',
          here we add custom filepicker only to Image dialog
        */
        file_picker_types: "image",
        // /* and here's our custom image picker*/
        // eslint-disable-next-line no-unused-vars
        file_picker_callback: function (cb, value, meta) {
          var input = document.createElement("input");
          input.setAttribute("type", "file");
          input.setAttribute("accept", "image/*");

          /*
            Note: In modern browsers input[type="file"] is functional without
            even adding it to the DOM, but that might not be the case in some older
            or quirky browsers like IE, so you might want to add it to the DOM
            just in case, and visually hide it. And do not forget do remove it
            once you do not need it anymore.
          */

          input.onchange = function () {
            var file = this.files[0];
            /*
              This is where we need to intercept the file and upload it to our modal
              and then return the URL of the uploaded file.
            */

            console.log(file);

            var reader = new FileReader();
            reader.onload = function () {
              /*
                we need to register the blob in TinyMCEs image blob
                registry.
              */
              var id = "blobid" + new Date().getTime();
              // eslint-disable-next-line no-undef
              var blobCache = tinymce.activeEditor.editorUpload.blobCache;
              var base64 = reader.result.split(",")[1];
              var blobInfo = blobCache.create(id, file, base64);
              blobCache.add(blobInfo);

              /* call the callback and populate the Title field with the file name */
              cb(blobInfo.blobUri(), { title: file.name });
            };
            reader.readAsDataURL(file);
          };

          input.click();
        },
        content_style:
          "body { font-family:Helvetica,Arial,sans-serif; font-size:14px }",
      },
    };
  },
  methods: {
    onChange() {
      for (let i = 0; i < this.$refs.file.files.length; i++) {
        this.files.push(this.$refs.file.files[i]);
      }
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
    cropImages() {
      this.cropImagesPage = !this.cropImagesPage;
    },
    changeFile() {
      const result = this.$refs.cropper.getResult();

      console.log(this.$refs.cropper.getResult());

      this.files = [result.image.src];

      this.closeModal();
      this.cropImagesPage = !this.cropImagesPage;
    },
  },
};
</script>

<script setup>
import { defineProps } from "vue";

const props = defineProps({
  name: {
    type: String,
    required: true,
  },
});

const welcome = `Image Cropper ${props.name}!`;
</script>
