<template>
  <div class="wrapper">
    <img
      src="../assets/image/bg.jpg"
      alt="bg"
      class="bg"
      :style="{ animationPlayState: state.isOver ? 'paused' : 'running' }"
    />
    <img
      src="../assets/image/mask-light.png"
      alt="mask"
      class="dot"
      id="mask"
      :style="{ top: state.top + 'px' }"
    />
    <canvas
      class="clouds"
      :style="{ animationPlayState: state.isOver ? 'paused' : 'running' }"
    ></canvas>
  </div>
  <div class="over-wrapper" v-show="state.isOver">
    <div class="over-box">
      <div class="content">GAME OVER</div>
      <div class="refresh-btn" @click="reload">PLAY AGAIN</div>
    </div>
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
    /**全局常量 */
    const ratio = window.innerWidth / 1920;
    const RADIUS = 100 * ratio;
    const TOP = RADIUS;
    const BOTTOM = window.innerHeight - RADIUS * 2;
    const MIDDLE = window.innerHeight / 2 - RADIUS;
    const g = 1.03;
    let ctx: CanvasRenderingContext2D | null;
    const v = ((3000 + window.innerWidth) * 0.8) / (10 * 1000); //云朵的运行速度

    /**云层位置数据 */
    const scale = 0.5;
    const cloudsImage = [
      {
        offsetTop: window.innerHeight / 2 - (309 / 2) * ratio,
        offsetLeft: window.innerWidth,
        width: 756 * ratio * scale,
        height: 309 * ratio * scale,
      },
      {
        offsetTop: window.innerHeight - 487 * ratio * scale,
        offsetLeft: 1000 * ratio + window.innerWidth,
        width: 1298 * ratio * scale,
        height: 487 * ratio * scale,
      },
      {
        offsetTop: 0,
        offsetLeft: window.innerWidth + 1200 * ratio,
        width: 1069 * ratio * scale,
        height: 401 * ratio * scale,
      },
    ];

    /** 全局状态 */
    const state = reactive({
      isOver: false,
      isMouseDown: false,
      top: MIDDLE,
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
          cloudsImage[0].offsetLeft,
          cloudsImage[0].offsetTop,
          cloudsImage[0].width,
          cloudsImage[0].height
        );
      };

      let img2 = new Image();
      img2.src = cloud2;
      img2.onload = () => {
        ctx?.drawImage(
          img2,
          cloudsImage[1].offsetLeft,
          cloudsImage[1].offsetTop,
          cloudsImage[1].width,
          cloudsImage[1].height
        );
      };

      let img3 = new Image();
      img3.src = cloud3;
      img3.onload = () => {
        ctx?.drawImage(
          img3,
          cloudsImage[2].offsetLeft,
          cloudsImage[2].offsetTop,
          cloudsImage[2].width,
          cloudsImage[2].height
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
    const checkInterval = () => {
      check();
      setInterval(() => {
        check();
      }, 10000);
    };

    const check = () => {
      let timer;
      clearInterval(timer);
      if (!state.isOver) {
        let left = (window.innerWidth / 2) * ratio;
        console.log("begin!");

        timer = setInterval(() => {
          if (!state.isOver) {
            left += 30 * v;
            let top = state.top;
            /** 碰撞情况枚举 */
            /** 第一朵云 */
            if (
              left >= cloudsImage[0].offsetLeft + RADIUS &&
              left <= cloudsImage[0].offsetLeft + cloudsImage[0].width - RADIUS
            ) {
              console.log("Crossing cloud - 1");
              if (
                top > cloudsImage[0].offsetTop - RADIUS &&
                top < cloudsImage[0].offsetTop + cloudsImage[0].height + RADIUS
              ) {
                state.isOver = true;
              }
            }
            /** 第二朵云 */
            if (
              left >= cloudsImage[1].offsetLeft + RADIUS &&
              left <= cloudsImage[1].offsetLeft + cloudsImage[1].width - RADIUS
            ) {
              console.log("Crossing cloud - 2");
              if (
                top > cloudsImage[1].offsetTop - RADIUS &&
                top < cloudsImage[1].offsetTop + cloudsImage[1].height + RADIUS
              ) {
                state.isOver = true;
              }
            }
            /** 第三朵云 */
            if (
              left >= cloudsImage[2].offsetLeft + RADIUS &&
              left <= cloudsImage[2].offsetLeft + cloudsImage[2].width - RADIUS
            ) {
              // console.log("Crossing cloud - 3");
              if (
                top > cloudsImage[2].offsetTop - RADIUS &&
                top < cloudsImage[2].offsetTop + cloudsImage[2].height + RADIUS
              ) {
                // state.isOver = true;
              }
            }
          }
        }, 30);
      } else {
        clearInterval(timer);
      }
    };

    /**
     * reload
     */
    const reload = () => {
      window.location.reload();
    };

    onMounted(() => {
      initClouds();
      initMask();
      checkInterval();
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
      window.addEventListener("resize", reload);
    });

    return {
      state,
      reload,
    };
  },
});
</script>

<style scoped>
.wrapper {
  z-index: -1;
  position: absolute;
}
.bg {
  z-index: 1;
  position: absolute;
  width: 1300vh;
  height: 100vh;
  animation: 60s rowup linear infinite normal;
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
  animation: 10s rowup linear infinite normal;
}

.over-wrapper {
  z-index: 100;
  width: 100vw;
  height: 100vh;
  position: fixed;
  display: flex;
  justify-content: center;
  align-items: center;
  background: rgba(0, 0, 0, 0.5);
}
.over-box {
  width: 380px;
  height: 200px;
  border-radius: 20px;
  background-color: white;
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  zoom: 0.6;
}

.content {
  color: black;
  font-size: 32px;
  font-weight: bold;
  line-height: 64px;
}

.refresh-btn {
  width: 200px;
  height: 40px;

  display: flex;
  justify-content: center;
  align-items: center;

  border-radius: 60px;
  border: 1px solid black;
  font-size: 16px;

  background-color: black;
  color: white;

  cursor: pointer;
}
.refresh-btn:hover,
.refresh-btn:active {
  opacity: 0.6;
}
</style>
