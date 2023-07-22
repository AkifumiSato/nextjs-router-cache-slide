<template>
  <h1 class="title">
    <slot />
  </h1>
</template>

<style scoped>
.title {
  display: flex;
  align-items: center;
  font-size: 50px;
  height: 100%;
  background: #FF0080;
  background: linear-gradient(to right, #FF0080 0%, #007CF0 100%);
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  width: max-content;
  line-height: 1.5;
}
</style>
