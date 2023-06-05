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
                <v-btn v-if="isCameraOpen" class="openCamBtn" @click="stopWebCamera">Stop Camera</v-btn>
                <v-btn v-else :disabled="isCameraOpenLoading" class="openCamBtn" @click="openCam">Open Camera</v-btn>
            </div>
            <div id="videoContainer">
                <video id="videoCam" ref="videoCam">Video stream not available</video>
                <br>
                <v-btn v-if="isCameraOpen" id="captureBtn" @click="capturePhoto">Capture Photo</v-btn>
            </div>
            <div>
                <canvas v-if="isCameraOpen" id="intermediateCanvas" ref="intermediateCanvas">Placeholder Text</canvas>
            </div>
        </v-card>
    </v-dialog>
</template>

<style scoped>
#videoCam {
    /* width: 1080px;
    height: 720px; */
    width: 99vw;
    height: 66vh;
    margin: 0 auto;
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
#dialogHeading {
    text-align: center;
}
</style>


<script>

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
        };
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
                };
                this.getWebCameraResolution();
                this.isCameraOpen = true;
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
        //function to capture a photo from the webcam feed
        capturePhoto() {
            let video = this.$refs.videoCam;
            let canvas = this.$refs.intermediateCanvas;
            let context = canvas.getContext('2d');
            //set the canvas dimensions to the webcam feed dimensions
            canvas.width = video.videoWidth;
            canvas.height = video.videoHeight;
            console.log(canvas.width, canvas.height);
            context.drawImage(video, 0, 0, canvas.width, canvas.height);
            let dataURL = canvas.toDataURL('image/png');
            this.link = dataURL;
            this.isPhotoTaken = true;
        },
    },
}

</script>