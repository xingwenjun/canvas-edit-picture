<template>
  <div ref="canvas" id="canvas">
    <canvas ref="myCanvas" id="myCanvas" width="800" height="800"></canvas>
    <canvas ref="myCanvas2" id="myCanvas2" width="800" height="800"></canvas>
  </div>
  <div id="menu">
    <div class="btn" @click="getImage">确定</div>
  </div>
</template>
<script setup>
import { reactive, ref, onMounted } from "vue";
const props = defineProps({
  src: {
    type: String,
    default: ""
  },
  option: {
    type: Object,
    default: () => ({})
  },
})
const emits = defineEmits(["result"]);
const canvas = ref(null);
const myCanvas = ref(null);
const myCanvas2 = ref(null);
let ctx,maskCtx;
let w = props.option.w;
let h = props.option.h;
let x=0,y=0;
// 获取设备的像素比
const dpr = window.devicePixelRatio || 1;
onMounted(() => {
  ctx = myCanvas.value.getContext("2d");
  maskCtx = myCanvas2.value.getContext("2d");
  const img = new Image();
  img.crossOrigin = 'anonymous';
  const scale = props.option.scale || 1;
  img.src = "https://www.xhsdyx.top:3002/uploads/user/man.png"
  img.onload = () => {
    w = w || img.width / 2;
    h = h || img.height / 2;
    x=img.width/4,y=img.height/4;
    myCanvas.value.width = img.width;
    myCanvas.value.height = img.height;
    ctx.drawImage(img, 0, 0, img.width, img.height);
    ctx.fillStyle = "rgba(0,0,0,0.8)";
    ctx.fillRect(0, 0, img.height,img.height);
    myCanvas2.value.width = img.width;
    myCanvas2.value.height = img.height;
    drawMask();
  }
  let posX = 0, posY = 0;
  canvas.value.addEventListener("mousedown", (e) => {
    if (e.buttons === 1) { // 检查是否按下鼠标左键
        posX = e.clientX;
        posY = e.clientY;
    }
  })
  canvas.value.addEventListener("mousemove", (e) => {
    if (e.buttons === 1) { // 检查是否按下鼠标左键
        const dx = e.clientX - posX;
        const dy = e.clientY - posY;
        x += dx;
        y += dy;
        posX = e.clientX;
        posY = e.clientY;
        drawMask();
    }
  })
  canvas.value.addEventListener("touchstart", (e) => {
    if (e.isTrusted) { // 检查是否按下鼠标左键
        posX = e.touches[0].clientX;
        posY = e.touches[0].clientY;
    }
  })
  canvas.value.addEventListener("touchmove", (e) => {
    if (e.isTrusted) { // 检查是否按下鼠标左键
        const dx = e.touches[0].clientX - posX;
        const dy = e.touches[0].clientY - posY;
        x += dx;
        y += dy;
        posX = e.touches[0].clientX;
        posY = e.touches[0].clientY;
        drawMask();
    }
  }, { passive: true })
  function drawMask() {
      // 应用蒙版
      maskCtx.clearRect(0, 0, myCanvas2.value.width, myCanvas2.value.height);
      maskCtx.globalCompositeOperation = 'source-over';
      maskCtx.fillStyle = "rgba(0,0,0,1)";
      maskCtx.drawImage(img,x, y, w,h,x,y, w,h);
      maskCtx.globalCompositeOperation = 'destination-out';
  }
})
const getImage = () => {
  const imgData = maskCtx.getImageData(x, y, w,h);
  const drawCanvas = document.createElement("canvas");
  const drawCtx = drawCanvas.getContext("2d");
  drawCanvas.width = imgData.width;
  drawCanvas.height = imgData.height;
  drawCtx.putImageData(imgData, 0, 0);
  const base64 = drawCanvas.toDataURL("image/png");
  emits("result", base64);
}
</script>
<style scoped>
#canvas {
  position: relative;
  width: 100%;
  height: 100%;
  overflow: hidden;
  min-width: 200px;
  min-height: 200px;
  background-color: #000;
}
#myCanvas {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  z-index: 1;
}
#myCanvas2 {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  z-index: 2
}
#menu {
  margin-top: 10px;
  display: flex;
  justify-content: center;
  align-items: center;
  flex-wrap: wrap;
}
.btn {
  width: 80px;
  height: 30px;
  line-height: 30px;
  background-color: #dd524d;
  border-radius: 8px;
  color: #fff;
}
</style>
