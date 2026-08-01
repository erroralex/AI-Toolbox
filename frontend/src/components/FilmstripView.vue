<script setup>
/**
 * @file FilmstripView.vue
 * @description A horizontal, carousel-style navigation component that displays a "filmstrip" of image thumbnails.
 *
 * This component is optimized for performance by rendering only a slice of the total image set
 * around the currently selected item (windowing). This prevents DOM bloat and network saturation
 * when browsing large libraries.
 *
 * Key functionalities:
 * - **Capped Render Slice:** Only renders ±VISIBLE_BUFFER items around the selection to maintain performance.
 * - **Reactive Centering:** Calculates the exact {@code translateX} offset needed to center the selected item within the container.
 * - **Smooth Transitions:** Uses CSS transitions for fluid movement when the selection changes.
 * - **Responsive Design:** Utilizes {@code ResizeObserver} to adapt centering logic to container size changes.
 * - **Interactive Thumbnails:** Allows users to select images directly from the strip.
 * - **Authenticated Media:** Uses the {@code authenticatedUrl} helper to ensure thumbnails are loaded with the required security token.
 */
import {useBrowserStore} from '@/stores/browser';
import {ref, computed, onMounted, onUnmounted, watch} from 'vue';
import {authenticatedUrl} from '@/services/api';

const store = useBrowserStore();
const containerRef = ref(null);
const containerWidth = ref(0);

const VISIBLE_BUFFER = 20;
const DEFAULT_SLICE_SIZE = 40;

const ITEM_WIDTH = 120;
const ITEM_GAP = 8;
const TOTAL_ITEM_WIDTH = ITEM_WIDTH + ITEM_GAP;

/**
 * Computes a slice of store.files centered around the selected index.
 * This limits the number of <img> elements and network requests.
 */
const visibleFiles = computed(() => {
  if (!store.files || store.files.length === 0) return [];

  const selectedIndex = store.files.findIndex(f => f.path === store.selectedFile);

  if (selectedIndex === -1) {
    return store.files.slice(0, DEFAULT_SLICE_SIZE);
  }

  const start = Math.max(0, selectedIndex - VISIBLE_BUFFER);
  const end = Math.min(store.files.length, selectedIndex + VISIBLE_BUFFER + 1);

  return store.files.slice(start, end);
});

/**
 * Calculates the translateX offset to center the selected item within the filmstrip.
 * The calculation is relative to the visibleFiles slice.
 */
const carouselOffset = computed(() => {
  if (!store.selectedFile || containerWidth.value === 0 || visibleFiles.value.length === 0) {
    return 0;
  }

  const sliceIndex = visibleFiles.value.findIndex(f => f.path === store.selectedFile);
  if (sliceIndex === -1) {
    return 0;
  }

  const selectedItemCenter = (sliceIndex * TOTAL_ITEM_WIDTH) + (TOTAL_ITEM_WIDTH / 2);
  const containerCenter = containerWidth.value / 2;

  return containerCenter - selectedItemCenter;
});

watch(() => store.files.length, () => {
  if (containerRef.value) {
    containerWidth.value = containerRef.value.clientWidth;
  }
});

let resizeObserver;
onMounted(() => {
  if (containerRef.value) {
    resizeObserver = new ResizeObserver(entries => {
      window.requestAnimationFrame(() => {
        if (entries[0]) {
          containerWidth.value = entries[0].contentRect.width;
        }
      });
    });
    resizeObserver.observe(containerRef.value);
    containerWidth.value = containerRef.value.clientWidth;
  }
});

onUnmounted(() => {
  if (resizeObserver) {
    resizeObserver.disconnect();
  }
});

</script>

<template>
  <div ref="containerRef" class="filmstrip-view filmstrip-glass h-10rem flex align-items-center overflow-hidden">
    <div
        class="flex flex-nowrap gap-2 px-2 transition-transform duration-500 ease-in-out"
        :style="{ transform: `translateX(${carouselOffset}px)` }"
    >
      <div v-for="file in visibleFiles" :key="file.path"
           class="filmstrip-item flex-shrink-0 cursor-pointer border-round"
           :class="{ 'selected-item': store.selectedFile === file.path }"
           @click="store.selectFile(file)">

        <div class="relative border-round overflow-hidden flex align-items-center justify-content-center"
             :style="{ width: `${ITEM_WIDTH}px`, height: `${ITEM_WIDTH}px` }">
          <img :src="authenticatedUrl(`/api/images/thumbnail?path=${encodeURIComponent(file.path)}`)"
               loading="lazy"
               class="w-full h-full"
               style="object-fit: contain;"/>
        </div>
      </div>
    </div>
  </div>
</template>

<style scoped>
.filmstrip-glass {
  background: var(--color-surface-1, #14151B);
  border-top: 1px solid var(--color-border-subtle, rgba(255, 255, 255, 0.06));
  box-shadow: var(--shadow-panel, 0 20px 60px -20px rgba(0,0,0,0.65));
}

.filmstrip-item {
  opacity: 0.65;
  border: 1px solid var(--color-border-subtle, rgba(255, 255, 255, 0.06));
  background: var(--color-surface-2, #23252F);
  position: relative;
  z-index: 0;
  border-radius: var(--radius-md, 8px);
  transition: all var(--duration-fast, 120ms) var(--ease-standard);
}

.filmstrip-item:hover {
  opacity: 0.95;
  border-color: var(--color-border-strong, rgba(255, 255, 255, 0.18));
  transform: translateY(-1px);
}

.filmstrip-item.selected-item {
  opacity: 1;
  border-color: var(--color-accent-primary, #4FD8D0);
  box-shadow: var(--glow-primary, 0 0 0 3px rgba(79, 216, 208, 0.16));
  transform: scale(1.04);
  z-index: 1;
}

.transition-transform {
  transition-property: transform;
}

.duration-500 {
  transition-duration: 300ms;
}

.ease-in-out {
  transition-timing-function: var(--ease-standard, cubic-bezier(0.2, 0, 0, 1));
}
</style>

