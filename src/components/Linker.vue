<script setup>
import { defineProps } from "vue";
import ArrowUpSquare from "@/components/ArrowUpSquare.vue";

const props = defineProps({
  url: {
    type: String,
    required: true,
  },
});
</script>

<template>
  <a
    class="p-1 shadow-sm text-sm bg-[#434343] rounded-lg"
    :href="props.url"
    target="_blank"
  >
    <arrow-up-square class="invert" />
  </a>
</template>
