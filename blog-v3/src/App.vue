<script setup>
import { ref, onMounted, h, onBeforeUnmount } from "vue";
import { isMobile } from "@/utils/tool";
import { addView, getAllPageHeader, homeGetConfig } from "@/api/config";
import { useRoute, useRouter } from "vue-router";
import { ElNotification } from "element-plus";

import { user, staticData } from "@/store/index.js";

import BackTop from "@/components/BackTop/index";
import ContextMenu from "@/components/ContextMenu/index.vue";

const userStore = user();
const router = useRouter();
const route = useRoute();
const isPc = ref(true);
const ContextMenuRef = ref(null);

const goBack = () => {
  router.go(-1);
};

// 获取所有的网站页面背景图
const getAllPageHeaderBg = async () => {
  const res = await getAllPageHeader();
  if (res.code == 0) {
    staticData().setPageHeaderLIst(res.result);
  } else {
    ElNotification({
      offset: 60,
      title: "错误提示",
      message: h("div", { style: "color: #f56c6c; font-weight: 600;" }, res.message),
    });
  }
};

const loadBlogConfig = async () => {
  const res = await homeGetConfig();
  if (res.code == 0 && typeof res.result != "string") {
    staticData().setBlogConfig(res.result);
    userStore.setBlogAvatar(res.result.blog_avatar);
    if (res.result.blog_name) {
      document.title = res.result.blog_name;
    }
  }
};

const handleContextMenu = (e) => {
  ContextMenuRef.value?.show(e);
};

const handleClick = () => {
  ContextMenuRef.value?.hide();
};

onMounted(async () => {
  isPc.value = !isMobile();

  // 上传访问量
  await addView();
  if (window.name == "") {
    getAllPageHeaderBg();
    loadBlogConfig();
  }

  document.addEventListener("contextmenu", handleContextMenu);
  document.addEventListener("click", handleClick);
});

onBeforeUnmount(() => {
  document.removeEventListener("contextmenu", handleContextMenu);
  document.removeEventListener("click", handleClick);
});
</script>

<template>
  <div class="app">
    <router-view></router-view>
    <BackTop v-if="route.path !== '/'" :right="isPc ? 3 : 0" />
    <i
      v-if="!isPc && ['home', '/'].includes(route.path)"
      class="iconfont icon-fanhui"
      @click="goBack"
    ></i>
    <ContextMenu ref="ContextMenuRef" />
  </div>
</template>

<style lang="scss">
.app {
  width: 100%;
  box-sizing: border-box;
}

.icon-fanhui {
  position: fixed;
  left: 5px;
  top: 60px;
  font-size: 2.2rem;
  color: var(--font-color);
  z-index: 999;
}
</style>
