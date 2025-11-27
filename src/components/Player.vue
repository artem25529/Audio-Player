<script setup>
import Slider from './Slider.vue';
import DropDown from './DropDown.vue';
import Loader from './Loader.vue';

import { ref, defineProps, defineEmits, computed, watch } from 'vue';
import { injectLocal, useThrottleFn } from '@vueuse/core';

const props = defineProps({
  image: {
    required: true,
  },
  track: {
    required: true,
  },
  song: {
    type: String,
    required: true,
  },
  album: {
    type: String,
    required: true,
  },
});

const emit = defineEmits(['prev', 'next', 'play', 'pause']);

const rateOptions = [1, 0.75, 0.5, 0.25].map((item) => ({
  label: `${item}x`,
  value: item,
}));

const loaderStyle = {
  position: 'absolute',
  inset: '0',
};

const audio = ref();

const currentTime = ref(0);
const duration = ref(0);
const isPlaying = ref(false);
const isSeeking = ref(false);
const isMuted = ref(false);
const volumeSliderPos = ref(1);
const prevVolume = ref(1);
const playBackRate = ref(1);
const prevTimestamp = ref();
const isLoading = ref(false);

const currentTimeSliderPos = computed(() => {
  if (!duration.value) return 0;

  if (isSeeking.value) {
    return currentTimeSliderPos.value;
  }

  return currentTime.value / duration.value;
});

watch(
  () => props.track,
  () => {
    audio.value.addEventListener('canplay', oncanplay, { once: true });
  },
);

function prev() {
  if (performance.now() - prevTimestamp.value < 2000) {
    prevTimestamp.value = performance.now();
    emit('prev');
  } else {
    prevTimestamp.value = performance.now();
    audio.value.currentTime = 0;
  }

  currentTime.value = 0;
}

function next() {
  emit('next');

  currentTime.value = 0;
}

function oncanplay() {
  if (audio.value.playbackRate !== playBackRate.value) {
    audio.value.playbackRate = playBackRate.value;
  }

  if (isPlaying.value && audio.value.paused) {
    audio.value.play();
  }
}

function togglePlay() {
  if (audio.value.paused) {
    audio.value.play();
  } else {
    audio.value.pause();
  }
}

function changeVolume(pos) {
  isMuted.value = pos === 0;
  audio.value.volume = pos;
}

function changeRate(rate) {
  playBackRate.value = rate;
  audio.value.playbackRate = rate;
}

function changeCurrentTime(pos) {
  const newTime = duration.value * pos;
  currentTime.value = newTime;

  if (!isSeeking.value) {
    audio.value.currentTime = newTime;
  }
}

function handleStartSeeking() {
  isSeeking.value = true;
}

function handleEndSeeking(pos) {
  isSeeking.value = false;
  changeCurrentTime(pos);
}

function handlePlay() {
  isPlaying.value = true;
  emit('play');
}

function handlePause() {
  isPlaying.value = false;
  emit('pause');
}

function handleDurationchange() {
  duration.value = audio.value.duration;
}

function handleTimeupdate() {
  if (isSeeking.value) return;

  currentTime.value = audio.value.currentTime;
}

const handleTimeupdateThrottled = useThrottleFn(handleTimeupdate, 100);

function toggleMute() {
  isMuted.value = !isMuted.value;

  if (isMuted.value) {
    prevVolume.value = audio.value.volume;
    volumeSliderPos.value = 0;
    audio.value.volume = 0;
  } else {
    volumeSliderPos.value = prevVolume.value;
    audio.value.volume = prevVolume.value;
  }
}

function formatTime(time) {
  const minutes = Math.floor(time / 60);
  const seconds = Math.floor(time) % 60;
  const secondsWLeadingZero = seconds < 10 ? '0' + seconds : seconds;

  return `${minutes}:${secondsWLeadingZero}`;
}

function onended() {
  isPlaying.value = true;
  next();
}
</script>

<template>
  <section class="player">
    <audio
      ref="audio"
      :src="track"
      @progress="isLoading = true"
      @waiting="isLoading = true"
      @canplay="isLoading = false"
      @playing="isLoading = false"
      @suspend="isLoading = false"
      @play="handlePlay"
      @pause="handlePause"
      @durationchange="handleDurationchange"
      @timeupdate="handleTimeupdateThrottled"
      @ended="onended"
    ></audio>

    <img class="player__img" :src="image" alt="logo" />
    <div class="player__info">
      <h1 class="player__info-song">{{ song }}</h1>
      <h2 class="player__info-album">{{ album }}</h2>
    </div>
    <div class="player__time">
      <Slider
        :position="currentTimeSliderPos"
        @poschanged="changeCurrentTime"
        @poschangestart="handleStartSeeking"
        @poschangeend="handleEndSeeking"
      />
      <div class="player__time-values">
        <div>{{ formatTime(currentTime) }}</div>
        <div>{{ formatTime(duration) }}</div>
      </div>
    </div>
    <div class="player__controls">
      <div class="player__controls-left">
        <i
          tabindex="0"
          class="pi muter"
          :class="isMuted ? 'pi-volume-off' : 'pi-volume-down'"
          @click="toggleMute"
          @keydown.space.enter="toggleMute"
        ></i>
        <Slider
          :position="volumeSliderPos"
          @poschanged="changeVolume"
          :style="{ fontSize: '0.9rem' }"
        />
      </div>
      <div class="player__controls-main">
        <button class="prev-track" type="button" @click="prev">
          <i class="pi pi-step-backward-alt"></i>
        </button>
        <button
          class="play-track"
          :class="{ 'play-track--moved': !isPlaying }"
          type="button"
          @click="togglePlay"
        >
          <i class="pi" :class="`pi-${isPlaying ? 'pause' : 'play'}`"></i>
          <Loader :style="loaderStyle" v-show="isLoading" />
        </button>
        <button class="prev-track" type="button" @click="next">
          <i class="pi pi-step-forward-alt"></i>
        </button>
      </div>
      <div class="player__controls-right">
        <DropDown :items="rateOptions" @valchanged="changeRate" />
        <i class="pi pi-stopwatch"></i>
      </div>
    </div>
  </section>
</template>

<style scoped lang="scss">
.player {
  display: flex;
  flex-direction: column;
  gap: 2em;
}

.player__img {
  width: 100%;
  aspect-ratio: 1;
  object-fit: cover;
  box-shadow: 0 0 0.2em get-color(neutral-500);
}

.player__info {
  white-space: nowrap;
  display: flex;
  flex-direction: column;
  text-align: center;
  gap: get-spacing(sm);
}

.player__info-album {
  color: get-color(neutral-700);
}

.player__time {
  display: flex;
  flex-direction: column;
  gap: get-spacing(md);
  color: get-color(neutral-700);
}

.player__time-values {
  padding-inline: get-spacing(sm);
  display: flex;
  justify-content: space-between;

  & > * {
    white-space: nowrap;
  }
}

.pi {
  display: block;
}

.player__controls {
  display: flex;
  align-items: center;
}

.player__controls-left,
.player__controls-right {
  flex: 1;
  padding-inline: 2em;
  display: flex;
  align-items: center;
  gap: get-spacing(md);

  & > :not(.pi) {
    flex: 1;
  }
}

.player__controls-main {
  font-size: 1.2em;
  display: flex;
  align-items: center;
  gap: get-spacing(lg);

  button {
    padding: 0.55em;
    border-radius: 50%;
    transition: all 60ms;

    &:hover {
      background-color: get-color(neutral-100);
    }

    &:active {
      scale: 0.9;
    }

    &:focus-visible {
      outline: 1px solid get-color(primary-600);
    }
  }

  .play-track {
    position: relative;
    padding: 0.85em;
    background-color: get-color(primary-200);

    &:hover {
      background-color: get-color(primary-300);
    }

    &--moved {
      .pi {
        translate: 10%;
      }
    }
  }
}

.muter {
  cursor: pointer;

  &:focus-visible {
    text-shadow: 0 0 0.8em get-color(primary-500);
  }
}
</style>
