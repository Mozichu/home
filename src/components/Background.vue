<template>
  <div :class="store.backgroundShow ? 'cover show' : 'cover'">
    <!-- 使用 v-show 而不是 v-if 来避免组件频繁销毁和重建 -->
    <img
      v-show="isImage && store.imgLoadStatus"
      :src="bgUrl || '/images/background1.jpg'"
      class="bg"
      alt="cover"
      @load="imgLoadComplete"
      @error.once="imgLoadError"
      @animationend="imgAnimationEnd"
    />
    <video
      v-show="!isImage && store.imgLoadStatus"
      :src="bgUrl || '/images/background1.jpg'"
      class="bg bg-video"
      autoplay
      muted
      loop
      playsinline
      @loadeddata="imgLoadComplete"
      @error.once="imgLoadError"
      @animationend="imgAnimationEnd"
    ></video>
    <div :class="store.backgroundShow ? 'gray hidden' : 'gray'" />
    <Transition name="fade" mode="out-in">
      <a
        v-if="store.backgroundShow && store.coverType != '3'"
        class="down"
        :href="bgUrl || '/images/background1.jpg'"
        target="_blank"
      >
        下载壁纸
      </a>
    </Transition>
  </div>
</template>

<script setup>
import { mainStore } from "@/store";
import { Error } from "@icon-park/vue-next";
import { ref, computed, defineEmits, watch, onMounted, onBeforeUnmount } from 'vue';
import { ElMessage, h } from 'element-plus';

const store = mainStore();
const bgUrl = ref(null);
const imgTimeout = ref(null);
const emit = defineEmits(["loadComplete"]);

// 壁纸随机数
const bgRandom = Math.floor(Math.random() * 10 + 1);

// 修改后的 isImage 计算属性
const isImage = computed(() => {
  return bgUrl.value?.endsWith('.mp4') === false;
});

// 更换壁纸链接
const changeBg = (type) => {
  console.log('Changing background type to:', type);
  if (type === 0) {
    bgUrl.value = `/images/background${bgRandom}.jpg`;
  } else if (type === 1) {
    bgUrl.value = "https://api.dujin.org/bing/1920.php";
  } else if (type === 2) {
    bgUrl.value = "https://api.vvhan.com/api/wallpaper/views";
  } else if (type === 3) {
    bgUrl.value = "https://api.vvhan.com/api/wallpaper/acg";
  } else if (type === 4) {
    bgUrl.value = "https://t.alcy.cc/acg";
  }
};

// 图片或视频加载完成
const imgLoadComplete = () => {
  console.log('Media loaded:', bgUrl.value);
  imgTimeout.value = setTimeout(
    () => {
      store.setImgLoadStatus(true);
    },
    Math.floor(Math.random() * (600 - 300 + 1)) + 300,
  );
};

// 图片或视频动画完成
const imgAnimationEnd = () => {
  console.log("壁纸或视频加载且动画完成");
  emit("loadComplete");
};

// 图片或视频显示失败
const imgLoadError = () => {
  console.error("壁纸或视频加载失败：", bgUrl.value);
  ElMessage({
    message: "壁纸或视频加载失败，已临时切换回默认",
    icon: h(Error, {
      theme: "filled",
      fill: "#efefef",
    }),
  });
  bgUrl.value = `/images/background${bgRandom}.jpg`;
};

// 监听壁纸切换
watch(
  () => store.coverType,
  (value) => {
    console.log('Cover type changed to:', value);
    changeBg(value);
  },
  { immediate: true } // 立即执行一次，确保初始化时加载正确的壁纸
);

onMounted(() => {
  console.log('Background component mounted');
  // 加载壁纸
  changeBg(store.coverType);
});

onBeforeUnmount(() => {
  clearTimeout(imgTimeout.value);
});
</script>

<style lang="scss" scoped>
.cover {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  transition: 0.25s;
  z-index: -1;

  &.show {
    z-index: 1;
  }

  .bg {
    position: absolute;
    left: 0;
    top: 0;
    width: 100%;
    height: 100%;
    object-fit: cover;
    backface-visibility: hidden;
    filter: blur(20px) brightness(0.3);
    transition:
      filter 0.3s,
      transform 0.3s;
    animation: fade-blur-in 0.8s cubic-bezier(0.25, 0.46, 0.45, 0.94) forwards;
    animation-delay: 0.45s;
  }

  .bg-video {
    width: 100%;
    height: 100%;
    object-fit: cover;
    position: fixed;
    top: 0;
    left: 0;
    z-index: -1;
  }

  .gray {
    opacity: 1;
    position: absolute;
    left: 0;
    top: 0;
    width: 100%;
    height: 100%;
    background-image: radial-gradient(rgba(0, 0, 0, 0) 0, rgba(0, 0, 0, 0.5) 100%),
      radial-gradient(rgba(0, 0, 0, 0) 33%, rgba(0, 0, 0, 0.3) 166%);

    transition: 1.5s;
    &.hidden {
      opacity: 0;
      transition: 1.5s;
    }
  }

  .down {
    font-size: 16px;
    color: white;
    position: absolute;
    bottom: 30px;
    left: 0;
    right: 0;
    margin: 0 auto;
    display: block;
    padding: 20px 26px;
    border-radius: 8px;
    background-color: #00000030;
    width: 120px;
    height: 30px;
    display: flex;
    justify-content: center;
    align-items: center;
    &:hover {
      transform: scale(1.05);
      background-color: #00000060;
    }
    &:active {
      transform: scale(1);
    }
  }
}
</style>
