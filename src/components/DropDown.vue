<script setup>
import {
  defineProps,
  defineEmits,
  ref,
  onMounted,
  reactive,
  onUnmounted,
  watch,
} from 'vue';

const props = defineProps({
  items: {
    type: Array,
    required: true,
  },
});

const emit = defineEmits(['valchanged']);

const dropDownItems = ref();

const isCollapsed = ref(true);
const selectedItemIdx = ref(0);
const dropDownItemsStyle = reactive({});

watch(selectedItemIdx, (newVal) => {
  emit('valchanged', props.items[newVal].value);
});

onMounted(() => {
  const clonedItems = dropDownItems.value.cloneNode(true);
  clonedItems.style.display = 'unset';
  clonedItems.style.visibility = 'hidden';

  document.body.append(clonedItems);
  dropDownItemsStyle.height = `${clonedItems.offsetHeight}px`;
  document.body.removeChild(clonedItems);

  window.addEventListener('click', handleWindowClick);
});

onUnmounted(() => {
  window.removeEventListener('click', handleWindowClick);
});

function toggleCollapsed() {
  isCollapsed.value = !isCollapsed.value;
}

function changeSelectedItem(idx) {
  selectedItemIdx.value = idx;
  isCollapsed.value = true;
}

function handleWindowClick(e) {
  if (isCollapsed.value) return;

  if (!e.target.closest('.drop-down')) {
    isCollapsed.value = true;
  }
}
</script>

<template>
  <div class="drop-down" role="list">
    <div
      tabindex="0"
      class="drop-down__preview drop-down__item drop-down__item--selected"
      @click="toggleCollapsed"
      @keydown.enter.space="toggleCollapsed"
    >
      <div class="drop-down__preview-label">
        {{ items[selectedItemIdx].label }}
      </div>
      <i
        class="drop-down__marker pi pi-sort-up-fill"
        :class="{ 'drop-down__marker--rotated': !isCollapsed }"
      ></i>
    </div>
    <transition name="items">
      <ul
        role="listbox"
        ref="dropDownItems"
        class="drop-down__items"
        v-show="!isCollapsed"
        :style="dropDownItemsStyle"
      >
        <li
          role="listitem"
          tabindex="0"
          class="drop-down__item"
          :class="{ 'drop-down__item--selected': idx === selectedItemIdx }"
          v-for="(item, idx) in items"
          @click="changeSelectedItem(idx)"
          @keydown.enter.space="changeSelectedItem(idx)"
        >
          {{ item.label }}
        </li>
      </ul>
    </transition>
  </div>
</template>

<style scoped lang="scss">
:focus-visible {
  box-shadow: inset 0 0 0 1px get-color(neutral-900);
}

.drop-down {
  position: relative;
}

.drop-down__item {
  padding: get-spacing(sm) get-spacing(md);
  background-color: get-color(neutral-100);
  cursor: pointer;

  &:hover:not(&--selected) {
    background-color: get-color(primary-200);
  }

  &--selected {
    background-color: get-color(primary-300);
  }
}

.drop-down__preview {
  background-color: get-color(primary-400);
}

.drop-down__items {
  width: 100%;
  overflow: hidden;
  position: absolute;
}

.drop-down__preview-label,
.drop-down__items .drop-down__item {
  @include text-trunc-ellipsis;
}

.drop-down__preview {
  display: flex;
  align-items: center;
  justify-content: space-between;
}

.drop-down__search {
  width: 100%;
  border: 1px solid;
}

.drop-down__marker {
  font-size: 0.64rem;
  @include transition-fast;

  &--rotated {
    rotate: 180deg;
  }
}

.items-enter-from,
.items-leave-to {
  height: 0 !important;
}

.items-enter-active,
.items-leave-active {
  @include transition-fast;
}
</style>
