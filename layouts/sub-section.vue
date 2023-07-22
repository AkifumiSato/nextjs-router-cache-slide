<script setup lang="ts">
import { ref } from 'vue'

const props = defineProps({
  breadcrumb: {
    default: 'breadcrumb',
  },
})

const breadcrumb = ref(props.breadcrumb)
</script>

<template>
  <div class="slidev-layout default layout">
    <slot />
    <div class="breadcrumb">{{breadcrumb}}</div>
  </div>
</template>

<style scoped>
.layout {
  display: flex;
  flex-direction: column;
  row-gap: 15px;
  position: relative;
}

.breadcrumb {
  font-size: 22px;
  background: #FF0080;
  background: linear-gradient(to right, #FF0080 0%, #007CF0 100%);
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  width: max-content;
  position: absolute;
  bottom: 30px;
}
</style>
