<script>
  import { onMount } from 'svelte';

  let videoElement;
  let canvasElement;
  let delayedFrames = [];
  const delayInMilliseconds = 2000; // 2-second delay

  onMount(async () => {
    // Request access to the user's camera
    const stream = await navigator.mediaDevices.getUserMedia({ video: true });
    const videoTrack = stream.getVideoTracks()[0];
    const imageCapture = new ImageCapture(videoTrack);

    // Start playing the live video
    videoElement.srcObject = stream;
    videoElement.play();

    // Capture frames with a delay
    setInterval(async () => {
      const frame = await imageCapture.grabFrame();
      delayedFrames.push(frame);

      // Remove frames older than the delay
      if (delayedFrames.length > delayInMilliseconds / 100) {
        delayedFrames.shift();
      }
    }, 100); // Capture frames every 100ms

    // Display delayed frames
    setInterval(() => {
      if (delayedFrames.length > 0) {
        const context = canvasElement.getContext('2d');
        context.drawImage(delayedFrames[0], 0, 0, canvasElement.width, canvasElement.height);
      }
    }, 100); // Update the canvas every 100ms
  });
</script>

<style>
  video, canvas {
    width: 100%;
    max-width: 600px;
    border: 1px solid #ccc;
    border-radius: 8px;
  }
</style>

<video bind:this={videoElement} autoplay muted style="display: none;"></video>
<canvas bind:this={canvasElement}></canvas>
