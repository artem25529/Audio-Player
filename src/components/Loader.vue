<script setup></script>

<template>
  <div class="loader"></div>
</template>

<style scoped lang="scss">
.loader {
  aspect-ratio: 1;
  border-radius: 50%;
  border: 3px solid get-color(neutral-400);
  border-top-color: get-color(neutral-900);
  animation: rotate 400ms infinite linear;
}

@keyframes rotate {
  to {
    rotate: 360deg;
  }
}
</style>
