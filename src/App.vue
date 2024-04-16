<script>
import videojs from "video.js";
import {markRaw} from "vue";

export default {
  mounted() {

    //video through websocket
    const player = videojs('stream-display');
    this.ws.onmessage = function (e) {
      const videoBlob = new Blob([e.data], {type: "video/mp4"})
      const videoUrl = URL.createObjectURL(videoBlob);
      player.src({type: "video/mp4", src: videoUrl});
      player.play();
    }
    //
    // this.ws.onmessage = (e) => {
    //   const imageBlob = new Blob([e.data], {type: "image/png"})
    //   this.frames ++;
    //
    //   // this.test_packageBlobAndDisplay(imageBlob);
    //
    // }
    // setTimeout(this.printCount, 1000);

    this.ws.onerror = console.error;


  },

  data() {
    return {
      test_val: 1,
      ws: new WebSocket('ws://'+window.location.host+'/stream'),
      frames: 0,
    }
  },
  methods: {
    test_fetchVideo() {
      // send message through websocket
      this.ws.send(new Blob([JSON.stringify({id: "vid-123", segment: 0, size: 12})], {type: "application/json"}));
      this.lastTimeUpdate = Date.now()
    },

    test_fetchImageAndDisplay() {
      let frameList = [];
      for (let i = 0; i < 10; i++) {
        const img = new Image();
        img.src = "/frames/output_00" + i + ".png";
        frameList.push(img)
      }
      for (let i = 10; i < 100; i++) {
        const img = new Image();
        img.src = "/frames/output_0" + i + ".png";
        frameList.push(img)
      }
      for (let i = 100; i < 300; i++) {
        const img = new Image();
        img.src = "/frames/output_" + i + ".png";
        frameList.push(img)
      }

      this.test_displayFrames(frameList);

    },

    test_displayFrames(frames) {
      const canvas = document.getElementById('player-c');
      const ctx = canvas.getContext('2d');

      let currentFrameIndex = 0;

      function drawFrame() {
        ctx.clearRect(0, 0, canvas.width, canvas.height);
        ctx.drawImage(frames[currentFrameIndex], 0, 0, canvas.width, canvas.height);

        currentFrameIndex++;
        if (currentFrameIndex >= frames.length) {
          currentFrameIndex = 0;  // 回到第一帧
        }

        setTimeout(drawFrame, 1000 / 30);
      }

      drawFrame();  // 启动帧动画
    },

    test_displayFrame(image) {
      const canvas = document.getElementById('player-c');
      const ctx = canvas.getContext('2d');

      ctx.clearRect(0, 0, canvas.width, canvas.height);
      ctx.drawImage(image, 0, 0, canvas.width, canvas.height);
    },

    async test_testDisplayFrame() {
      await fetch("/frames/output_001.png")
          .then(res => res.blob()).then(blob => this.test_packageBlobAndDisplay(blob));
    },

    test_packageBlobAndDisplay(imageBlob) {
      const blobUrl = URL.createObjectURL(imageBlob);
      const img = new Image();
      img.src = blobUrl;
      console.log("blobUrl", blobUrl)

      img.onload = () => {
        this.test_displayFrame(img);
        URL.revokeObjectURL(blobUrl);
      }
      console.log("image loaded");
    },

    printCount (){
      console.log("frames: ", this.frames);
      this.frames = 0;
      setTimeout(this.printCount, 1000);
    }

  }
}

</script>

<template>
  <div class="display">
        <video id="stream-display"
               class="video-js"
               controls

               style="width: 100%; height: 100%">

        </video>

<!--    <canvas id="player-c" style="width: 100%; height: 100%" width="1920" height="1080"></canvas>-->
  </div>


  <div class="bottom-console">
    <v-btn @click="test_fetchVideo">ABC</v-btn>
    <v-btn @click="">Message</v-btn>
    <v-btn>delay: {{ delay }}</v-btn>
    <v-btn>{{ test_val }}</v-btn>
  </div>
</template>

<style scoped>

.display {
  position: relative;
  left: 10vw;
  top: 10vh;
  height: 80vh;
  width: 80vw;
  background: #282828;
}

.bottom-console {
  width: 100%;
  height: 5%;
  background: #2c3e50;
  position: absolute;
  left: 0;
  bottom: 0;


}
</style>
