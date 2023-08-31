<template>
  <div>
    <button @click="getUserAMedia">测试</button>
    <button @click="snipaste">截取图片</button>

    <video src="#"></video>

    <img v-for="item in imgList" :src="item" alt="">
  </div>
</template>

<script setup lang="ts">
import { ref } from 'vue';

const imgList = ref<string[]>([])

const getUserAMedia = () => {
  const constraints = { audio: false, video: { width: 360, height: 360 } };

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
const snipaste = () => {
  let video: any = document.querySelector("video");
  let canvas = document.createElement("canvas");
  let ctx: any = canvas.getContext("2d");

  canvas.width = video.videoWidth;
  canvas.height = video.videoHeight;

  ctx.drawImage(video, 0, 0);

  imgList.value.push(canvas.toDataURL('image/png'));
}
</script>

<style scoped>
video {
  transform: rotateY(180deg);
}
img {
  width: 40px;
  height: 40px;
}
</style>
