<template>
  <div class="default layout">
    <div class="card">
      <div class="message">
        <slot />
      </div>
    </div>
  </div>
</template>

<style scoped>
.layout {
  display: flex;
  justify-content: center;
  align-items: center;
  width: 100%;
  height: 100%;
}

.card {
  /*color: #fff;*/
  background: #f1f3f5;
  /*background: linear-gradient(to right, rgb(74, 222, 128), rgb(6, 182, 212));*/
  border-radius: 10px;
  width: 90%;
  height: 90%;
  display: flex;
  justify-content: center;
  align-items: center;
  padding: 50px;
}

.message {
  font-size: 50px;
  font-weight: bold;
  line-height: 2;
}
</style>
