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

watch(
  () => props.position,
  (newVal) => {
    if (newVal !== thumbPos.value) {
      thumbPos.value = newVal;
    }
  },
);

const thumbStyle = computed(() => ({
  translate: `${thumbPos.value * trackWidth.value ?? 0}px`,
}));

const trackStyle = computed(() => ({
  width: trackWidth.value ? `${trackWidth.value}px` : '',
}));

const passedStyle = computed(() => ({
  width: `${thumbPos.value * 100}%`,
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

  if (thumbPos.value !== newThumbPos) {
    thumbPos.value = newThumbPos;
    emit('poschanged', newThumbPos);
  }
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
  const newThumbPos = (e.offsetX - halfThumbWidth) / trackWidth.value;

  if (newThumbPos !== thumbPos.value) {
    thumbPos.value = newThumbPos;
    emit('poschanged', newThumbPos);
  }
}

function handleKeyDown(e) {
  const oldVal = thumbPos.value;
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

  if (oldVal !== thumbPos.value) {
    emit('poschanged', thumbPos.value);
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
    <div class="slider__passed" :style="passedStyle"></div>

    <div class="slider__track" :style="trackStyle"></div>
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
$hover-scale: 1.13;

.slider {
  display: grid;
  position: relative;
  background-color: get-color(neutral-300);

  @include border-radius(md);

  &:hover {
    scale: 1 $hover-scale;

    .slider__thumb {
      scale: $hover-scale 1;
    }
  }
}

.slider__track {
  height: 0.5em;
  pointer-events: none;
}

.slider__passed {
  background-color: get-color(primary-500);
  border-radius: inherit;
  position: absolute;
  width: 100%;
  height: 100%;
  pointer-events: none;
}

.slider__thumb {
  height: 0.8em;
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
