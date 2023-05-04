<template>
  <h1>{{ welcome }}</h1>
  <!-- add vuetify button -->
  <DialogComponent />
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
import "vue-advanced-cropper/dist/style.css";
import DialogComponent from "./DialogComponent.vue";

export default {
  components: {
    Editor,
    DialogComponent,
  },
  data() {
    return {
      // have selected image as first file or null
      selectedImage: null,
      modal: false,
      apiKey: import.meta.env.VITE_APP_TINYMCE_API_KEY,
      // object to store cropped images
      croppedImages: [],
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
};
</script>

<script setup>
import { defineProps } from "vue";

const props = defineProps({
  name: {
    type: String,
    required: true,
  },
  components: {
    DialogComponent,
  },
});

const welcome = `Image Cropper ${props.name}!`;
</script>
