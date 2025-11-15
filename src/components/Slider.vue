<script setup>
import {
  defineProps,
  defineEmits,
  onMounted,
  onUnmounted,
  computed,
  watch,
  ref,
} from 'vue';

import { useThrottleFn } from '@vueuse/core';

const props = defineProps({
  width: {
    type: String,
    validator(value) {
      return parseFloat(value) > 0;
    },
  },
  position: {
    type: Number,
    validator(value) {
      return value >= 0 && value <= 1;
    },
  },
});

const emit = defineEmits(['poschanged', 'poschangestart', 'poschangeend']);

const root = ref();
const thumb = ref();

const trackWidth = ref();
const thumbWidth = ref();
const thumbPos = ref(props.position ?? 0);

let disableThumbWatch = false;

watch(
  () => props.position,
  (newVal) => {
    disableThumbWatch = true;
    thumbPos.value = newVal;
  },
);

watch(thumbPos, (newVal) => {
  if (disableThumbWatch) {
    disableThumbWatch = false;
    return;
  }

  emit('poschanged', newVal);
});

const thumbStyle = computed(() => ({
  translate: `${thumbPos.value * trackWidth.value ?? 0}px`,
}));

const passedStyle = computed(() => ({
  translate: `${thumbPos.value * 100 - 100}%`,
}));

const sliderStyle = props.width
  ? {
      width: props.width,
    }
  : null;

function handlePointerDown(e) {
  thumb.value.setPointerCapture(e.pointerId);

  emit('poschangestart', thumbPos.value);
}

function handlePointerMove(e) {
  if (!thumb.value.hasPointerCapture(e.pointerId) || !e.movementX) return;

  const movement = e.movementX / trackWidth.value;
  const newThumbPos = Math.min(Math.max(0, thumbPos.value + movement), 1);

  thumbPos.value = newThumbPos;
}

const handlePointerMoveThrottled = useThrottleFn(handlePointerMove, 10);

function handlePointerUp(e) {
  if (thumb.value.hasPointerCapture(e.pointerId)) {
    thumb.value.releasePointerCapture(e.pointerId);

    emit('poschangeend', thumbPos.value);
  }
}

function handleClick(e) {
  const halfThumbWidth = (thumbWidth.value ?? 0) / 2;

  thumbPos.value = (e.offsetX - halfThumbWidth) / trackWidth.value;
}

function handleKeyDown(e) {
  const step = 0.05;

  switch (e.key) {
    case 'ArrowRight':
    case 'ArrowUp':
      thumbPos.value += step;

      if (thumbPos.value > 1) {
        thumbPos.value = 1;
      }

      break;
    case 'ArrowLeft':
    case 'ArrowDown':
      thumbPos.value -= step;
      if (thumbPos.value < 0) {
        thumbPos.value = 0;
      }

      break;
  }
}

function setTrackWidth() {
  trackWidth.value = root.value.offsetWidth - thumb.value.offsetWidth;
}

function setThumbWidth() {
  thumbWidth.value = thumb.value.offsetWidth;
}

function handleResize() {
  setTrackWidth();
}

onMounted(() => {
  setTrackWidth();
  setThumbWidth();

  window.addEventListener('resize', handleResize);
});

onUnmounted(() => {
  window.removeEventListener('resize', handleResize);
});
</script>

<template>
  <div
    class="slider"
    role="slider"
    aria-label="slider"
    aria-valuemin="0"
    :aria-valuenow="thumbPos * 100"
    aria-valuemax="100"
    ref="root"
    :style="sliderStyle"
    @click.self="handleClick"
  >
    <div class="slider__track">
      <div class="slider__passed" :style="passedStyle"></div>
    </div>
    <div
      class="slider__thumb"
      tabindex="0"
      ref="thumb"
      :style="thumbStyle"
      @pointerdown="handlePointerDown"
      @pointermove="handlePointerMoveThrottled"
      @pointercancel="handlePointerUp"
      @pointerup="handlePointerUp"
      @keydown="handleKeyDown"
    ></div>
  </div>
</template>

<style scoped lang="scss">
$hover-scale: 1.15;

.slider {
  display: grid;

  &:hover {
    .slider__track {
      scale: 1 $hover-scale;
    }

    .slider__thumb {
      scale: $hover-scale;
    }
  }
}

.slider__track {
  height: 0.5rem;
  background-color: get-color(neutral-300);
  overflow: hidden;
  position: relative;
  pointer-events: none;

  @include border-radius(md);
}

.slider__passed {
  background-color: get-color(primary-500);
  width: 100%;
  height: 100%;
  translate: -100%;
  pointer-events: none;
}

.slider__thumb {
  height: 0.8rem;
  aspect-ratio: 1;
  border-radius: 50%;
  background-color: get-color(neutral-300);
  position: absolute;
  align-self: center;
  cursor: grab;

  @include box-shadow-blur(sm);

  &:active {
    cursor: grabbing;
  }

  &:focus {
    background-color: get-color(primary-500);
  }
}
</style>
