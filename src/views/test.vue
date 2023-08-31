<template>
  <div class="about">
    测试2
    <div class="wrappar">
      <video id="myVideo"></video>
      <canvas id="canvas" width="180" height="180"></canvas>
    </div>
    <button @click="getUserMedia">测试</button>
    <button @click="stop">关闭</button>
    <button @click="snipaste">截取图片</button>

    <img v-for="item in imgList" :src="item" alt="">
  </div>
</template>

<script setup lang="ts">
import { onMounted, ref, reactive } from 'vue';
import '../../public/trackingjs/tracking.js';
import '../../public/trackingjs/face.js';

const imgList = ref<string[]>([])

let trackingRect: any = ref({})
let trackerTask: any = ref(null)

onMounted(() => {
  let canvas: any = document.getElementById('canvas');
  let context = canvas.getContext('2d');

  let tracker = new window.tracking.ObjectTracker('face');

  trackerTask.value = window.tracking.track('#myVideo', tracker);

  tracker.on('track', (event: any) => {
    if (event.data.length === 0) {
      // No objects were detected in this frame.
    } else {
      context.clearRect(0, 0, canvas.width, canvas.height);

      event.data.forEach((rect: any) => {
        trackingRect.value = rect
        // rect.x, rect.y, rect.height, rect.width
        context.strokeStyle = '#a64ceb';
        context.strokeRect(rect.x, rect.y, rect.width, rect.height);
        context.font = '11px Helvetica';
        context.fillStyle = "#fff";
        context.fillText('x: ' + rect.x + 'px', rect.x + rect.width + 5, rect.y + 11);
        context.fillText('y: ' + rect.y + 'px', rect.x + rect.width + 5, rect.y + 22);
      });
    }
  });
})

/**
 * 开启摄像头，捕获媒体流
 */
const getUserMedia = () => {
  trackerTask.value.run()
  const constraints = { audio: false, video: { width: 180, height: 180 } };

  //以下是此方法的兼容性写法，
  if (navigator.mediaDevices.getUserMedia === undefined) {
    navigator.mediaDevices.getUserMedia = function (constraints) {
      var getUserMedia =
        (navigator as any).getUserMedia ||
        (navigator as any).webkitGetUserMedia ||
        (navigator as any).mozGetUserMedia ||
        (navigator as any).msGetUserMedia ||
        (navigator as any).oGetUserMedia ||
        MediaDevices;
      if (!getUserMedia) {
        return Promise.reject(
          new Error("getUserMedia is not implemented in this browser")
        );
      }
      return new Promise(function (resolve, reject) {
        getUserMedia.call(navigator, constraints, resolve, reject);
      });
    };
  }

  navigator.mediaDevices.getUserMedia(constraints).then((mediaStream) => {
    let video: any = document.querySelector("video");
      if (video) {
        // 旧的浏览器可能没有 srcObject
        if ("srcObject" in video) {
          video.srcObject = mediaStream;
        } else {
          // 防止在新的浏览器里使用它，应为它已经不再支持了
          video.src = window.URL.createObjectURL(mediaStream as any);
        }
        video.onloadedmetadata = (e: any) => {
          video.play();
        };
      }
  }).catch(() => {

  })
};

/**
 * 停止
 */
const stop = () => {
  let video: any = document.querySelector("video");
  let canvas: any = document.getElementById('canvas');
  let context = canvas.getContext('2d');

  video?.srcObject.getTracks().forEach((track: any) => {
    track.stop()
  });
  trackerTask.value.stop()

  setTimeout(() => {
    context.clearRect(0, 0, canvas.width, canvas.height);
  }, 50);
}

/**
 * 截取帧图像
 */
const snipaste = () => {
  let video = document.querySelector("video");
  let canvas = document.createElement("canvas");
  let ctx: any = canvas.getContext("2d");

  const rect = trackingRect.value

  const compress = 1 // 图像缩放比例
  canvas.width = (rect.width+20)/compress;
  canvas.height = (rect.height+40)/compress;

  ctx.translate((rect.width+20)/compress, 0);
  ctx.scale(-1, 1); //左右镜像翻转

  ctx.drawImage(video, rect.x-10, rect.y-20, rect.width+20, rect.height+40, 0, 0, (rect.width+20)/compress, (rect.height+40)/compress);

  return new Promise<Blob>((resolve, reject) => {
    canvas.toBlob((blob) => {
      if (blob) {
        resolve(blob)
        imgList.value.push(URL.createObjectURL(blob))
      }
    }, 'image/jpeg', 0.7)
  })
}
</script>

<style scoped>
.wrappar {
  height: 180px;
  width: 180px;
  position: relative;
}
video {
  transform: rotateY(180deg);
  height: 180px;
  width: 180px;
}
canvas {
  position: absolute;
  left: 0;
  top: 0;
  transform: rotateY(180deg);
}
img {
  width: 90px;
}
</style>
