<script setup>
import { defineProps } from 'vue';

const props = defineProps({
  isPlaying: {
    type: Boolean,
    required: true,
  },
});
</script>

<template>
  <div class="play-indicator" :class="{ paused: !isPlaying }">
    <div></div>
    <div></div>
    <div></div>
  </div>
</template>

<style scoped lang="scss">
.play-indicator {
  height: 1lh;
  width: 1.15lh;
  display: flex;
  align-items: end;
  gap: 0.115em;

  & > div {
    flex: 1;
    background-color: get-color(neutral-600);

    &:first-of-type {
      animation: move-1 0.5s linear infinite both;
    }

    &:nth-of-type(2) {
      animation: move-2 0.5s linear infinite both;
    }

    &:nth-of-type(3) {
      animation: move-3 0.5s linear infinite both;
    }
  }

  &.paused > div {
    animation-play-state: paused;
  }
}

@keyframes move-1 {
  0% {
    height: 50%;
  }

  25% {
    height: 100%;
  }

  75% {
    height: 70%;
  }

  100% {
    height: 50%;
  }
}

@keyframes move-2 {
  0% {
    height: 100%;
  }

  25% {
    height: 70%;
  }

  75% {
    height: 50%;
  }

  100% {
    height: 100%;
  }
}

@keyframes move-3 {
  0% {
    height: 70%;
  }

  25% {
    height: 50%;
  }

  75% {
    height: 100%;
  }

  100% {
    height: 70%;
  }
}
</style>
