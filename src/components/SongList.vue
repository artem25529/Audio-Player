<script setup>
import { computed, onMounted, ref, watchEffect } from 'vue';

import Player from './Player.vue';
import PlayIndicator from './PlayIndicator.vue';

const audioModules = import.meta.glob('@/assets/audios/*');
const audioKeys = Object.keys(audioModules);
const audioQty = audioKeys.length;
const imageModules = import.meta.glob('@/assets/images/*');
const imageKeys = Object.keys(imageModules);

const audios = new Map();
const images = new Map();

const root = ref();

const mounted = ref(false);
const currAudioIdx = ref(0);
const currAudioSrc = ref();
const currAudioImg = ref();
const band = ref();
const song = ref();
const isPlaying = ref(false);

const songListStyle = computed(() => {
  if (!mounted.value) return;

  const rootComputedStyle = getComputedStyle(root.value);

  const targetHeight =
    parseFloat(rootComputedStyle.height) -
    (parseFloat(rootComputedStyle.paddingTop) +
      parseFloat(rootComputedStyle.paddingBottom));

  return { display: 'unset', height: `${targetHeight}px` };
});

onMounted(() => {
  mounted.value = true;
});

watchEffect(async () => {
  const audioModule = audioKeys[currAudioIdx.value];
  const name = audioModule.substring(audioModule.lastIndexOf('/') + 1);

  [band.value, song.value] = getSongMeta(name);

  await setCurrAudio(name, audioModule);
  await setCurrImg(name);
});

function getSongMeta(name) {
  const meta = name.split('_');
  meta.splice(1, 1);

  return meta
    .map((e) => e.split('-').map(capitalize).join(' '))
    .map((e, idx) => {
      if (idx === meta.length - 1) {
        return e.substring(0, e.lastIndexOf('.'));
      } else {
        return e;
      }
    });
}

async function setCurrImg(name) {
  const imageName = name.substring(0, name.lastIndexOf('_'));

  if (!images.has(imageName)) {
    const imgKey = imageKeys.find((m) => m.includes(imageName));
    const loadedImage = await imageModules[imgKey]();
    images.set(imageName, loadedImage.default);
  }

  currAudioImg.value = images.get(imageName);
}

async function setCurrAudio(name, audioModule) {
  if (!audios.has(name)) {
    const loadedAudio = await audioModules[audioModule]();
    audios.set(name, loadedAudio.default);
  }

  currAudioSrc.value = audios.get(name);
}

function capitalize(word) {
  return word[0].toUpperCase() + word.substring(1);
}

function next() {
  currAudioIdx.value =
    currAudioIdx.value === audioQty - 1 ? 0 : currAudioIdx.value + 1;
}

function prev() {
  currAudioIdx.value =
    currAudioIdx.value === 0 ? audioQty - 1 : currAudioIdx.value - 1;
}
</script>

<template>
  <div ref="root" class="song-list-wrapper" v-if="audioKeys?.length">
    <player
      :image="currAudioImg"
      :track="currAudioSrc"
      :album="band"
      :song="song"
      @prev="prev"
      @next="next"
      @play="isPlaying = true"
      @pause="isPlaying = false"
    />
    <ul class="song-list" :style="songListStyle">
      <li
        class="song-wrapper"
        :class="{ active: idx === currAudioIdx }"
        v-for="(audio, idx) in audioKeys"
        :key="audio"
        @click="currAudioIdx = idx"
      >
        <div class="song">
          {{
            getSongMeta(audio.substring(audio.lastIndexOf('/') + 1)).join(' - ')
          }}
        </div>

        <PlayIndicator v-show="idx === currAudioIdx" :is-playing="isPlaying" />
      </li>
    </ul>
  </div>
  <div v-else>No audios found</div>
</template>

<style scoped lang="scss">
.song-list-wrapper {
  width: min(100%, 1100px);
  padding: get-spacing(xl);
  background-color: get-color(neutral-50);
  display: grid;
  gap: get-spacing(xl);
  grid-template-columns: repeat(2, 1fr);

  @include box-shadow(sm);
}

@media (max-width: 925px) {
  .song-list-wrapper {
    width: clamp(500px, 100%, 600px);
    grid-template: auto auto / auto;
  }
}

.song-list {
  display: none;
  overflow: auto;
}

.song-wrapper {
  padding: get-spacing(md) get-spacing(lg);
  font-style: italic;
  font-size: 1.1em;
  cursor: pointer;
  display: flex;
  align-items: center;

  @include transition-fast;
  @include border-radius(md);

  &:hover {
    background-color: get-color(neutral-100);
  }

  &.active {
    background-color: get-color(primary-100);
  }
}

.song {
  flex: 1 1;
}
</style>
