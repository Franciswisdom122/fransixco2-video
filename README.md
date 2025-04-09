<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>FranSixcO2 Live AI Face Call</title>
  <style>
    video, canvas {
      position: absolute;
      top: 0;
      left: 0;
    }
    #controls {
      position: absolute;
      top: 10px;
      left: 10px;
      background: rgba(0,0,0,0.6);
      padding: 10px;
      color: white;
      z-index: 10;
    }
  </style>
</head>
<body>
  <div id="controls">
    <label for="faceSelect">Choose a Face:</label>
    <select id="faceSelect">
      <option value="default">Default</option>
      <option value="anime">Anime Face</option>
      <option value="robot">Robot Face</option>
    </select>
    <button onclick="startVideo()">Start Video</button>
  </div>
  <video id="video" width="640" height="480" autoplay muted></video>
  <canvas id="overlay" width="640" height="480"></canvas>

  <script src="https://cdn.jsdelivr.net/npm/@mediapipe/face_mesh/face_mesh.js"></script>
  <script src="https://cdn.jsdelivr.net/npm/@mediapipe/camera_utils/camera_utils.js"></script>
  <script src="script.js"></script>
</body>
</html>const videoElement = document.getElementById('video');
const canvasElement = document.getElementById('overlay');
const canvasCtx = canvasElement.getContext('2d');

async function startVideo() {
  const stream = await navigator.mediaDevices.getUserMedia({ video: true });
  videoElement.srcObject = stream;
  videoElement.play();

  const faceMesh = new FaceMesh({
    locateFile: (file) => `https://cdn.jsdelivr.net/npm/@mediapipe/face_mesh/${file}`,
  });

  faceMesh.setOptions({
    maxNumFaces: 1,
    refineLandmarks: true,
    minDetectionConfidence: 0.5,
    minTrackingConfidence: 0.5,
  });

  faceMesh.onResults(drawFace);
  const camera = new Camera(videoElement, {
    onFrame: async () => {
      await faceMesh.send({ image: videoElement });
    },
    width: 640,
    height: 480
  });
  camera.start();
}

function drawFace(results) {
  canvasCtx.save();
  canvasCtx.clearRect(0, 0, canvasElement.width, canvasElement.height);
  if (results.multiFaceLandmarks.length > 0) {
    for (const landmarks of results.multiFaceLandmarks) {
      canvasCtx.strokeStyle = 'cyan';
      canvasCtx.lineWidth = 1;
      for (const point of landmarks) {
        canvasCtx.beginPath();
        canvasCtx.arc(point.x * canvasElement.width, point.y * canvasElement.height, 1, 0, 2 * Math.PI);
        canvasCtx.stroke();
      }
    }
  }
  canvasCtx.restore();
}# fransixco2-video
