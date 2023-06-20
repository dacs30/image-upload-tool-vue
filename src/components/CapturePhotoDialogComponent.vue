<template>
    <v-dialog v-model="showDialog" fullscreen :scrim="false" transition="dialog-top-transition">
        <v-card>
            <div class="header"> 
                <v-btn id="cancelBtn" @click="handleCancelBtn">
                    <v-icon start icon="mdi-minus-circle"></v-icon>
                    Cancel
                </v-btn>
                <h2 id="dialogHeading">Take a Photo Dialog</h2>
            </div>
            <div>
                <h3>Instructions: Please place your work/paper within the red bounding box</h3>
            </div>
            <div>
                <v-btn v-if="isCameraOpen" class="openCamBtn" @click="stopWebCamera">Stop Camera</v-btn>
                <v-btn v-else :disabled="isCameraOpenLoading" class="openCamBtn" @click="openCam">Open Camera</v-btn>
            </div>
            <div id="videoContainer">
                <video id="videoCam" ref="videoCam">Video stream not available</video>
                <div v-if="isCameraOpen" :style="boxStyle"></div>
            </div>
            <div>
                <br>
                <v-btn v-if="isCameraOpen" id="captureBtn" @click="capturePhoto">
                    <span v-if="isPhotoTaken">Retake Photo</span>
                    <span v-else>Capture Photo</span>
                </v-btn>
            </div>
            <div>
                <canvas v-if="isCameraOpen" id="intermediateCanvas" ref="intermediateCanvas">Placeholder Text</canvas>
            </div>
            <v-btn v-if="isPhotoTaken" id="saveBtn" @click="savePhoto">Save Photo</v-btn>
        </v-card>
    </v-dialog>
</template>

<style scoped>
#videoCam {
    /* width: 1080px;
    height: 720px; */
    /* width: 99vw; */
    height: 66vh;
    margin: 0 auto;
    border: 1px solid black;
}
.openCamBtn {
    margin: 1% 10%;
    background-color: lightskyblue;
}
#captureBtn{
    margin: 1% 10%;
    background-color: coral;
}
.header {
    position: relative;
}
#cancelBtn {
    position: absolute;
    left: 0;
}
#saveBtn {
    margin: 1% 10%;
    background-color: lightgreen;
}
#dialogHeading {
    text-align: center;
}
#videoContainer {
  position: relative;
  border: 1px solid black;
}
#videoContainer > div {
  position: absolute;
}
</style>


<script>
// import * as cocoSSD from '@tensorflow-models/coco-ssd';
import * as tf from '@tensorflow/tfjs';
import '@tensorflow/tfjs-backend-cpu';
import '@tensorflow/tfjs-backend-webgl';
import { loadGraphModel } from '@tensorflow/tfjs';

export default {
    name: "CapturePhotoDialogComponent",
    data() {
        return {
            showDialog: true,
            constraints: {
                video: {
                    width: { ideal: 1080 },
                    height: { ideal: 720 }
                }
            },
            isCameraOpen: false,
            isCameraOpenLoading: false,
            isPhotoTaken: false,
            link: '#',
            boxProps: {
                width: 0,
                height: 0,
                top: 0,
                left: 0,
            },
        };
    },
    props: {
        files: {
            type: Array,
            required: true,
        },
        dataURLToFile: {
            type: Function,
            required: true,
        },
    },
    methods: {
        openCam() {
            this.isCameraOpenLoading = true;
            let allMediaDevices = navigator.mediaDevices;
            //if allMediaDevices are null or getUserMedia function DNE
            if (!allMediaDevices || !allMediaDevices.getUserMedia) {
                console.log("getUserMedia() is not supported")
                return;
            }
            console.log("success: allMediaDevices and method not null")
            allMediaDevices.getUserMedia(this.constraints)
            .then((vidStream) => {
                let video = document.getElementById('videoCam')
                if ("srcObject" in video) {
                    video.srcObject = vidStream;
                } else {
                    video.src = window.URL.createObjectURL(vidStream);
                }
                video.onloadedmetadata = () => {
                    video.play()
                    console.log("Succesfully playing live webcam feed");
                    this.getWebCameraResolution();
                    this.isCameraOpen = true;
                    console.log("isCameraOpen: ", this.isCameraOpen);
                    this.detectObjects();
                };
            }).catch((e) => {
                console.log("An error occured with getting webcam feed");
                console.log(e.name + ": " + e.message);
            });
        },
        //utility function to get check the resolution of the webcam video stream
        async getWebCameraResolution() {
            let video = this.$refs.videoCam;
            let videoTrack = video.srcObject.getVideoTracks()[0];
            let settings = videoTrack.getSettings();
            let width = settings.width;
            let height = settings.height;
            console.log("width: ", width, "height: ", height);
        },
        //function to stop the web camera feed
        stopWebCamera() {
            let videoCamSource = this.$refs.videoCam.srcObject;
            if(videoCamSource == null) {
                return;
            }
            let tracks = videoCamSource.getTracks();

            tracks.forEach(track => {
                track.stop();
            });
            this.$refs.videoCam.srcObject = null;
            this.isCameraOpen = false;
            this.isCameraOpenLoading = false;
        },
        handleCancelBtn() {
            //first stop the webcam feed
            this.stopWebCamera();
            //then close the dialog
            this.$emit("close-capture-photo");
        },
        //crop the image to the size of the boxProps
        cropImage() {
            let canvas = this.$refs.intermediateCanvas;
            let context = canvas.getContext('2d');
            let img = new Image();
            img.src = this.link;
            context.drawImage(img, this.boxProps.left, this.boxProps.top, this.boxProps.width, this.boxProps.height, 0, 0, this.boxProps.width, this.boxProps.height);
            let dataURL = canvas.toDataURL('image/png');
            this.link = dataURL;
        },
        //function to capture a photo from the webcam feed
        // capturePhoto() {
        //     let video = this.$refs.videoCam;
        //     let canvas = this.$refs.intermediateCanvas;
        //     let context = canvas.getContext('2d');
        //     //set the canvas dimensions to the webcam feed dimensions
        //     canvas.width = video.videoWidth;
        //     canvas.height = video.videoHeight;
        //     console.log(canvas.width, canvas.height);
        //     context.drawImage(video, 0, 0, canvas.width, canvas.height);
        //     let dataURL = canvas.toDataURL('image/png');
        //     this.link = dataURL;
        //     this.isPhotoTaken = true;
        //     this.cropImage();
        // },
        capturePhoto() {
            let video = this.$refs.videoCam;
            let canvas = this.$refs.intermediateCanvas;
            let context = canvas.getContext('2d');
            // Set the canvas dimensions to the webcam feed dimensions
            canvas.width = video.videoWidth;
            canvas.height = video.videoHeight;
            context.drawImage(video, 0, 0, canvas.width, canvas.height);

            let videoRect = video.getBoundingClientRect();
            let boxRect = document.querySelector('#videoContainer > div').getBoundingClientRect();

            let scaleX = video.videoWidth / videoRect.width;
            let scaleY = video.videoHeight / videoRect.height;

            let boxX = (boxRect.left - videoRect.left) * scaleX;
            let boxY = (boxRect.top - videoRect.top) * scaleY;
            let boxWidth = boxRect.width * scaleX;
            let boxHeight = boxRect.height * scaleY;

            let imageData = context.getImageData(boxX, boxY, boxWidth, boxHeight);

            canvas.width = boxWidth;
            canvas.height = boxHeight;
            context.putImageData(imageData, 0, 0);

            let dataURL = canvas.toDataURL('image/png');
            this.link = dataURL;
            this.isPhotoTaken = true;
        },
        savePhoto() {
            let newPhotoFile = this.dataURLToFile(this.link, "newPhotoFile.png");
            this.files.push(newPhotoFile);
            console.log("pushed new photo file to files list");
            //handleCancelBtn closes the dialog here not cancel
            this.handleCancelBtn();
        },
        async detectObjects() {
            const MODEL_URL = 'src/TFJSModel/model.json'; 
            const model = await loadGraphModel(MODEL_URL);
            const img = tf.browser.fromPixels(this.$refs.videoCam);
            const resized = tf.image.resizeBilinear(img, [640, 480]);
            const casted = resized.cast('int32');
            const expanded = casted.expandDims(0);
            const obj = await model.executeAsync(expanded);

            console.log("obj: ", obj);
            const boxes = await obj[4].array();
            const classes = await obj[5].array();
            const scores = await obj[6].array();
            console.log("boxes: ", boxes);
            console.log("classes: ", classes);
            console.log("scores: ", scores);
            // const predictions = await model.predict(this.$refs.videoCam);
            // console.log("predictions: ", predictions);
        }
    },
    computed: {
        boxStyle() {
            let video = this.$refs.videoCam;
            let videoWidth = 66 * video.videoWidth / 100; // adjusting for 66 vh
            let videoHeight = 66 * video.videoHeight / 100;

            //get the position of the video element
            let videoPos = video.getBoundingClientRect();
            let videoXPos = videoPos.left;
            let videoYPos = videoPos.top;

            //calculate the box size
            let boxWidth = videoWidth * 0.4;
            let boxHeight = videoHeight * 0.8;
            let boxXPos = videoXPos + (videoWidth * 0.3);
            let boxYPos = videoYPos - (videoHeight * 0.2);

            this.boxProps.width = boxWidth;
            this.boxProps.height = boxHeight;
            this.boxProps.top = boxYPos;
            this.boxProps.left = boxXPos;

            return {
                position: 'absolute',
                left: `${boxXPos}px`,
                top: `${boxYPos}px`,
                width: `${boxWidth}px`,
                height: `${boxHeight}px`,
                border: '10px solid red',
            };
        },
    },
}

</script>