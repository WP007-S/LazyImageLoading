# LazyImageLoading
图片懒加载

# 在utils自定义一个lazy指令，utils/lazyLoad.js
export const lazyLoadDirective = {
  mounted(el, binding) {
    const observer = new IntersectionObserver(
      (entries) => {
        entries.forEach((entry) => {
          if (entry.isIntersecting) {
            // 当元素进入视口时，设置图片的 src 属性
            el.src = binding.value;
            // 停止观察该元素
            observer.unobserve(el);
          }
        });
      },
      {
        rootMargin: "50px", // 提前加载的范围
        threshold: 0.1,     // 触发加载的阈值
      }
    );

    // 将 observer 保存到元素上，以便在 unmounted 时清理
    el._observer = observer;
    observer.observe(el);
  },
  unmounted(el) {
    // 清理 observer
    if (el._observer) {
      el._observer.disconnect();
      delete el._observer;
    }
    // 清理图片加载
    if (el.src) {
      el.src = "";
    }
  },
};

# 在Vue项目中注册并使用
import { createApp } from 'vue';
import App from './App.vue';
import { lazyLoadDirective } from './utils/lazyLoad';

const app = createApp(App);

// 注册自定义指令
app.directive('lazy', lazyLoadDirective);

app.mount('#app');

# img标签中使用，可以通过@load="onImageLoad"添加骨架屏
<template>
  <div>
    <div
        v-if="!isImageLoaded"
        class="image-placeholder"
    >
        <div class="skeleton-loading"></div>
    </div>
    <img
      v-lazy="imageUrl"
      @load="onImageLoad"
      alt="Lazy Loaded Image"
      class="lazy-image"
    />
  </div>
</template>

<script setup>
import { ref } from 'vue';

const imageUrl = ref('https://example.com/large-image.jpg');

const onImageLoad = (event) => {
  if (event.target.complete) {
    isImageLoaded.value = true;
    // emit('imageLoaded', {
    //   id: props.item.id,
    //   height: event.target.offsetHeight
    // });
  }
};

</script>

<style>
.lazy-image {
  width: 100%;
  height: auto;
  transition: opacity 0.3s ease-in-out;
}

.skeleton-loading {
  width: 100%;
  height: 100%;
  background: linear-gradient(90deg, #f0f0f0 25%, #e0e0e0 50%, #f0f0f0 75%);
  background-size: 200% 100%;
  animation: shimmer 1.5s infinite;
}

@keyframes shimmer {
  0% {
    background-position: 200% 0;
  }
  100% {
    background-position: -200% 0;
  }
}
</style>
