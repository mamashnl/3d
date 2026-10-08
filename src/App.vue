<script setup lang="ts">
import { defineAsyncComponent } from 'vue';

const Design = defineAsyncComponent(
  // @ts-ignore - Vue SFC type declarations are provided by the project shims, which are not present here.
  () => import('./components/manual.vue/index.js')
);
</script>

<template>
  <Design />
</template>
