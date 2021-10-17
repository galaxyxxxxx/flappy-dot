<template>
  <div class="wrapper">
    <img src="../assets/image/bg.jpg" alt="bg" class="bg" />
    <img
      src="../assets/image/mask-light.png"
      alt="mask"
      class="dot"
      id="mask"
      :style="{ top: state.top + 'px' }"
    />
    <div class="dot" :style="{ top: state.top + 'px' }"></div>
    <canvas class="clouds"></canvas>
  </div>
</template>

<script lang="ts">
import cloud1 from "@/assets/image/cloud-1.png";
import cloud2 from "@/assets/image/cloud-2.png";
import cloud3 from "@/assets/image/cloud-3.png";
import { defineComponent, reactive, onMounted } from "@vue/runtime-core";
export default defineComponent({
  name: "Home",
  setup() {
    const RADIUS = 100;
    const TOP = RADIUS;
    const BOTTOM = window.innerHeight - RADIUS * 2;
    const MIDDLE = window.innerHeight / 2 - RADIUS;
    const g = 1.03;
    const ratio = window.innerWidth / 1920;
    let ctx: CanvasRenderingContext2D | null;

    const state = reactive({
      isOver: false,
      isMouseDown: false,
      top: MIDDLE,
      ctx: null,
    });

    /**
     * 云层初始化
     */
    const initClouds = () => {
      const clouds = document.querySelector(".clouds") as HTMLCanvasElement;
      clouds!.width = 3000 + window.innerWidth;
      clouds!.height = window.innerHeight;

      ctx = (clouds as HTMLCanvasElement).getContext("2d");

      let img1 = new Image();
      img1.src = cloud1;
      img1.onload = () => {
        ctx?.drawImage(
          img1,
          0 + window.innerWidth,
          0,
          907 * ratio,
          907 * ratio
        );
      };

      let img2 = new Image();
      img2.src = cloud2;
      img2.onload = () => {
        ctx?.drawImage(
          img2,
          800 + window.innerWidth,
          window.innerHeight - 611 * ratio,
          1298 * ratio,
          611 * ratio
        );
      };

      let img3 = new Image();
      img3.src = cloud3;
      img3.onload = () => {
        ctx?.drawImage(
          img3,
          1200 + window.innerWidth,
          -100,
          1463 * ratio,
          600 * ratio
        );
      };
    };

    /**
     * 蒙版初始化
     */
    const initMask = () => {
      const mask = document.querySelector("#mask") as HTMLDivElement;
      mask!.style.width = window.innerWidth * 2 + "px";
      mask!.style.height = window.innerHeight * 2 + "px";
    };

    /**
     * 碰撞检测
     */
    const check = () => {
      let top = state.top;
      const imageData = ctx?.getImageData(
        window.innerWidth / 2,
        top,
        1,
        1
      ).data;
      console.log(top, imageData);
      // for (let index = 3; index < imageData!.length; index += 4) {
      //   console.log(imageData![index]);
      //   // if (imageData![index] == 0) {
      //   //   console.log("isAlphaBackground");
      //   //   state.isOver = true;
      //   // }
      // }
      // }, 10000);
    };

    onMounted(() => {
      initClouds();
      initMask();
      check();
      setInterval(() => {
        /** 游戏未结束时 */
        if (!state.isOver) {
          if (state.isMouseDown) {
            /** 匀速向上 */
            state.top = Math.max(TOP, state.top * (2 - g));
          } else {
            /** 自由落体 */
            state.top = Math.min(BOTTOM, state.top * g);
          }
        }
      }, 30);
      window.addEventListener("mousedown", () => {
        state.isMouseDown = true;
      });
      window.addEventListener("mouseup", () => {
        state.isMouseDown = false;
      });
    });

    return {
      state,
    };
  },
});
</script>

<style scoped>
.bg {
  z-index: 1;
  position: absolute;
  width: 1300vh;
  height: 100vh;
  animation: 60s rowup linear infinite normal 3s;
}

@keyframes rowup {
  0% {
    transform: translate3d(0, 0, 0);
  }
  100% {
    transform: translate3d(-80%, 0, 0);
  }
}

.dot {
  z-index: 20;
  position: absolute;
  top: 50%;
  left: 0px;
  transform: translate3d(-25%, -50%, 0);
  /* animation: 3s init linear forwards; */
}

@keyframes init {
  0% {
    left: 0px;
  }
  100% {
    left: calc(50% - 100px);
  }
}

.clouds {
  z-index: 30;
  position: fixed;
  animation: 10s rowup linear infinite normal 3s;
}
</style>
