<template>
    <v-dialog v-model="showDialog" fullscreen :scrim="false" transition="dialog-top-transition">
        <v-card>
            <h1>Take a Photo Dialog</h1>
            <br>
            <v-btn id="startBtn" @click="openCam">Open Webcam</v-btn>
            <br><br>
            <video id="videoCam"></video>
        </v-card>
    </v-dialog>
</template>


<script>

export default {
    name: "CapturePhotoDialogComponent",
    data() {
        return {
            showDialog: true,
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
            allMediaDevices.getUserMedia({
                audio: false,
                video: {
                    width: { ideal: 1080 },
                    height: { ideal: 720 }
                },
            }).then((vidStream) => {
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
            }).catch((e) => {
                console.log("An error occured with getting webcam feed");
                console.log(e.name + ": " + e.message);
            });
        }
    }
}

</script>