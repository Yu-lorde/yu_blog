<script setup>
import { ref, onMounted, onBeforeUnmount, computed, nextTick, watch } from "vue";
import { useRoute } from "vue-router";
import { staticData } from "@/store/index.js";
import { storeToRefs } from "pinia";

import TypeWriter from "@/components/TypeWriter/type-writer";
import Waves from "@/components/WelcomeComps/waves.vue";
import { debounce } from "@/utils/tool";
import gsap from "gsap";
import { ScrollTrigger } from "gsap/dist/ScrollTrigger";

const route = useRoute();
const staticStore = staticData();
const { getPageHeaderList, getBlogName, getBlogConfig } = storeToRefs(staticStore);

const saying = ref([]);
const showScrollBottom = ref(true);
const homeRef = ref(null);
const titleAnimated = ref(false);

let bgScrollTrigger = null;

gsap.registerPlugin(ScrollTrigger);

const titleChars = computed(() => getBlogName.value.split(""));
const personalSay = computed(() => getBlogConfig.value?.personal_say || "");

const scrollToContent = () => {
  const content = document.querySelector("#home-content");
  if (content) {
    content.scrollIntoView({ behavior: "smooth", block: "start" });
    return;
  }
  const homeElement = document.querySelector("#home");
  if (homeElement) {
    window.scrollTo({
      top: homeElement.offsetHeight - 90,
      behavior: "smooth",
    });
  }
};

const scrollListener = debounce(() => {
  showScrollBottom.value = document.documentElement.scrollTop < 50;
}, 50);

const initOneSentence = () => {
  fetch("https://wisdom.mrzym.top/api/random")
    .then((res) => res.json())
    .then((res) => {
      if (res.success) {
        saying.value = [res.data.content];
      }
    });
};

const initEntrance = () => {
  gsap.set(".hero-bg-inner", { opacity: 0, scale: 1.06 });
  gsap.set(".hero-mask", { opacity: 0 });
  gsap.set(".hero-char", { opacity: 0, y: 32 });
  gsap.set(".hero-tagline", { opacity: 0, y: 18 });
  gsap.set(".hero-saying", { opacity: 0, y: 14 });
  gsap.set(".scroll-bottom", { opacity: 0, y: 12 });

  const tl = gsap.timeline({ defaults: { ease: "power3.out" } });

  tl.to(".hero-bg-inner", { opacity: 1, scale: 1, duration: 1.1 })
    .to(".hero-mask", { opacity: 1, duration: 0.7 }, "-=0.55")
    .to(
      ".hero-char",
      {
        opacity: 1,
        y: 0,
        stagger: 0.04,
        duration: 0.55,
      },
      "-=0.45"
    )
    .to(".hero-tagline", { opacity: 1, y: 0, duration: 0.65 }, "-=0.15")
    .to(".hero-saying", { opacity: 1, y: 0, duration: 0.55 }, "-=0.25")
    .to(".scroll-bottom", { opacity: 1, y: 0, duration: 0.45 }, "-=0.15");

  titleAnimated.value = true;
};

const initBgParallax = () => {
  const tween = gsap.to(".hero-bg-inner", {
    scrollTrigger: {
      trigger: "#home",
      scrub: true,
      start: "top top",
      end: "bottom",
    },
    scale: 1.28,
  });
  bgScrollTrigger = tween.scrollTrigger;
};

const onMouseMove = (e) => {
  if (window.matchMedia("(max-width: 768px)").matches) return;
  const el = homeRef.value;
  if (!el) return;

  const rect = el.getBoundingClientRect();
  const x = (e.clientX - rect.left - rect.width / 2) / rect.width;
  const y = (e.clientY - rect.top - rect.height / 2) / rect.height;

  gsap.to(".hero-content", {
    x: x * 14,
    y: y * 10,
    duration: 0.55,
    ease: "power2.out",
  });
};

const onMouseLeave = () => {
  gsap.to(".hero-content", {
    x: 0,
    y: 0,
    duration: 0.75,
    ease: "power2.out",
  });
};

const getBgCover = computed(() => {
  const bgList = getPageHeaderList.value;
  const myUrl = "http://img.mrzym.top/FvmVKfygxBKoJbFVXJwzjgAASL9S";
  const index = bgList.findIndex((bg) => bg.route_name == route.name);
  return index == -1 ? myUrl : bgList[index].bg_url;
});

watch(getBlogName, () => {
  if (titleAnimated.value) {
    nextTick(() => {
      gsap.from(".hero-char", {
        opacity: 0,
        y: 20,
        stagger: 0.03,
        duration: 0.45,
        ease: "power2.out",
      });
    });
  }
});

watch(personalSay, (val, oldVal) => {
  if (val && !oldVal && titleAnimated.value) {
    nextTick(() => {
      gsap.to(".hero-tagline", { opacity: 0.95, y: 0, duration: 0.6, ease: "power3.out" });
    });
  }
});

onMounted(() => {
  initOneSentence();
  window.addEventListener("scroll", scrollListener);

  nextTick(() => {
    initEntrance();
    initBgParallax();
  });
});

onBeforeUnmount(() => {
  window.removeEventListener("scroll", scrollListener);
  bgScrollTrigger?.kill();
});
</script>

<template>
  <div id="home" ref="homeRef" @click="scrollToContent" @mousemove="onMouseMove" @mouseleave="onMouseLeave">
    <div class="hero-bg">
      <div class="hero-bg-inner">
        <el-image class="bg !w-[100%] !h-[100%]" fit="cover" :src="getBgCover"></el-image>
      </div>
    </div>
    <div class="hero-mask"></div>
    <div class="hero-content">
      <h1 class="hero-title">
        <span v-for="(char, index) in titleChars" :key="`${char}-${index}`" class="hero-char">{{
          char === " " ? "\u00A0" : char
        }}</span>
      </h1>
      <p v-if="personalSay" class="hero-tagline">{{ personalSay }}</p>
      <div class="hero-saying">
        <TypeWriter size="1.1em" :typeList="saying"></TypeWriter>
      </div>
    </div>
    <Waves />
    <div v-if="showScrollBottom" class="scroll-bottom">
      <i class="iconfont icon-arrowdown"></i>
    </div>
  </div>
</template>

<style lang="scss" scoped>
#home {
  position: relative;
  width: 100vw;
  height: 100vh;
  overflow: hidden;
  background: var(--home-bg);
  cursor: pointer;
}

.hero-bg {
  position: absolute;
  inset: 0;
  z-index: 1;
  overflow: hidden;

  .hero-bg-inner {
    width: 100%;
    height: 100%;
    transform-origin: center center;
    opacity: 0;
  }

  .bg {
    width: 100%;
    height: 100%;
  }
}

.hero-mask {
  position: absolute;
  inset: 0;
  z-index: 2;
  background-color: var(--home-mask-color);
  pointer-events: none;
  opacity: 0;
}

.hero-content {
  position: absolute;
  inset: 0;
  z-index: 10;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  padding: 0 1.5rem;
  text-align: center;
  will-change: transform;
  transition: transform 0.15s ease;
}

.hero-title {
  margin: 0;
  font-size: clamp(2rem, 5.5vw, 4.2rem);
  font-weight: 700;
  letter-spacing: 0.04em;
  line-height: 1.25;
  color: var(--global-white);
  text-shadow: 0 2px 20px rgba(0, 0, 0, 0.3);
}

.hero-char {
  display: inline-block;
  will-change: transform, opacity;
  opacity: 0;
}

.hero-tagline {
  margin: 0.85rem 0 0;
  font-size: clamp(0.95rem, 2.2vw, 1.25rem);
  letter-spacing: 0.12em;
  color: var(--second-font-color);
  opacity: 0;
}

.hero-saying {
  margin-top: 1rem;
  max-width: 90vw;
  min-height: 1.6em;
  color: var(--menu-color);
  opacity: 0;
}

.scroll-bottom {
  position: absolute;
  cursor: pointer;
  bottom: 70px;
  left: 50%;
  color: var(--menu-color);
  transform: translate(-50%, -50%);
  z-index: 20;
  opacity: 0;

  .icon-arrowdown {
    display: inline-block;
    font-size: 1.5em;
    animation: bounce 2s infinite;
  }
}

@keyframes bounce {
  0%,
  100% {
    transform: translateY(20px);
    opacity: 0.4;
  }

  50% {
    transform: translateY(0);
    opacity: 1;
  }
}
</style>
