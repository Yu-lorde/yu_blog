<template>
  <div class="footer_box flex flex-col items-center justify-center">
    <div class="footer-color">&copy; {{ blogName }} {{ currentYear }}</div>
  </div>
</template>

<script setup>
import { computed } from "vue";
import { staticData } from "@/store/index.js";
import { storeToRefs } from "pinia";

const { getBlogName } = storeToRefs(staticData());
const blogName = computed(() => getBlogName.value);
const currentYear = new Date().getFullYear();
</script>

<style lang="scss" scoped>
.footer_box {
  padding: 20px 0;
}

.footer-color {
  color: var(--font-color);
  text-align: center;
}
</style>
