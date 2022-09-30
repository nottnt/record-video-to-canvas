<script>
    import { onMount } from 'svelte';
    import RecordRTC, { MediaStreamRecorder, WhammyRecorder } from './RecordRTC.js';
    import Webcam from 'webcam-easy';
    let videoRecorder;
    let videoRecorded;
    let videoRecordedURL;
    let recorder;
    let webcamElement;
    let canvasElement;
    let webcam;

    onMount(() => {
        videoRecorder  = document.getElementById("video-recorder");
        videoRecorded = document.getElementById("video-recorded");
        webcamElement = document.getElementById('webcam');
        canvasElement = document.getElementById('canvas');
        webcam = new Webcam(webcamElement, 'user', canvasElement);

	});

    export const handleRecordVideo = () => {
        const recordingTimeMS = 6000;
        console.log(videoRecorder)
        videoRecorder.style.display = "block"; // display video for record
        videoRecorded.style.display = "none"; // hide video recorded
        videoRecorder.style.backgroundColor = "black"
        videoRecorded.style.backgroundColor = "black";

        if (videoRecordedURL) { // check for revoke obj url
            URL.revokeObjectURL(videoRecordedURL)
        }

        captureCamera(function(camera) {
            videoRecorder.muted = true;
            videoRecorder.volume = 0;
            videoRecorder.srcObject = camera;

            recorder = RecordRTC(camera, {
                type: 'video',
                audio: false,
                video: true,
                mimeType: 'video/webm',
                recorderType: MediaStreamRecorder || WhammyRecorder
            });

            recorder.startRecording();

            // release camera on stopRecording
            recorder.camera = camera;

            setTimeout(function () {
                recorder.stopRecording(stopRecordingCallback);
            }, recordingTimeMS)
        })
    }

    const captureCamera = (callback) => {
        const constraints = { video: true };

        navigator.mediaDevices.getUserMedia(constraints).then(function(camera) {
        callback(camera);
        }).catch(function(error) {
        console.error(error);

        if (error.name.indexOf('NotAllowedError') !== -1) {
            onErrorGetAccessCamera()
        }
        });
    }

    const stopRecordingCallback = () => {
        const recordedBlob = recorder.getBlob()
        console.log('recordedBlob', recordedBlob)
        videoRecordedURL = URL.createObjectURL(recordedBlob);
        videoRecorder.muted = true;
        videoRecorded.src = videoRecordedURL
        recorder.camera.stop();
        recorder.destroy();
        recorder = null;

        videoRecorder.style.display = "none"; // hide video recorder
        videoRecorded.style.display = "block"; // display video recorded
        // Assign video file to `input file for prepare to upload`
    }

    const startWebcam = () => {
        webcam.start()
        .then(result =>{
            console.log("webcam started");
        })
        .catch(err => {
            console.log(err);
        });
    }

    const handleTakePicture = () => {
        const picture = webcam.snap();
        console.log('picture =>', picture)
        webcam.stop();
    }

</script>
<link rel="stylesheet" href="video-verify-identity.css" />

<div id="video-recorder-container" class="video-recorder-container">
    <content class="video-recorder-content-container">
        <div class="content-wrapper">
            <div class="video-recorder-header-message">
                <span>Record Video</span>
            </div>
            <div class="video-recorder-wrapper">
                <video id="video-recorder" class="video-recorder" autoplay muted playsinline></video>
                <video id="video-recorded" class="video-recorder" controls style="display: none"></video>
            </div>
            <div class="start-verify-button-wrapper">
                <button class="start-verify-button" on:click={handleRecordVideo}>
                    <span>Start Recording</span>
                </button>
            </div>
            <div class="message-wrapper">
                <div class="hint-message">
                    <span>Note: Recording for 5 seconds when you click start recording </span>
                </div>
            </div>
            <p id="not-verify-message" class="upload-document-error" style="display: none; margin-bottom: 10px;">กรุณายืนยันตัวตน</p>
            <p id="video-verify-success-message" class="upload-success" style="display: none; margin-bottom: 10px;">วิดีโอของท่านได้รับการบันทึกเรียบร้อยแล้ว</p>
        </div>
    </content>
    <content class="video-recorder-content-container">
        <div class="content-wrapper">
            <div class="video-recorder-header-message">
                <span>Web Cam Capture</span>
            </div>
            <div class="video-recorder-wrapper">
                <video id="webcam" autoplay playsinline width="640" height="480"></video>
                <canvas id="canvas" class="d-none"></canvas>
            </div>
            <div class="start-verify-button-wrapper">
                <button class="start-verify-button" on:click={startWebcam}>
                    <span>Start webcam</span>
                </button>
                <button class="start-verify-button" on:click={handleTakePicture}>
                    <span>take a picture</span>
                </button>
            </div>
        </div>
    </content>
</div>