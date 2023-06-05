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
            <v-btn id="openCamBtn" @click="openCam">Open Webcam</v-btn>
            <video id="videoCam"></video>
            <v-btn v-model="showDialog" id="captureBtn">Capture Photo</v-btn>
        </v-card>
    </v-dialog>
</template>

<style scoped>
#videoCam {
    width: 60%;
    height: 40%;
    margin: 0   auto;
}
#openCamBtn {
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
            }
        };
    },
    methods: {
        openCam() {
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
            }).catch((e) => {
                console.log("An error occured with getting webcam feed");
                console.log(e.name + ": " + e.message);
            });
        },
        //utility function to get check the resolution of the webcam video stream
        async getWebCameraResolution() {
            let stream = await navigator.mediaDevices.getUserMedia(this.constraints);

            let stream_settings = stream.getVideoTracks()[0].getSettings();

            // actual width & height of the camera video
            let stream_width = stream_settings.width;
            let stream_height = stream_settings.height;

            console.log('Width: ' + stream_width + 'px');
            console.log('Height: ' + stream_height + 'px');
        },
        handleCancelBtn() {
            this.$emit("close-capture-photo");
        },
    },
}

</script>