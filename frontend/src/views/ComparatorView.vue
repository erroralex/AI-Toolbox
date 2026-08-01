<script setup>
/**
 * @file ComparatorView.vue
 * @description A side-by-side image comparison tool featuring a draggable slider, synchronized zoom, and pan aligned with the Latent Design System.
 */
import { ref, onUnmounted } from 'vue';
import LButton from '@/components/ds/LButton.vue';
import { useToast } from 'primevue/usetoast';
import ImageSplitViewer from '@/components/ImageSplitViewer.vue';
import ComparisonMetadataPanel from '@/components/ComparisonMetadataPanel.vue';
import api from '@/services/api';
import { Image as ImageIcon, RotateCcw } from 'lucide-vue-next';

const toast = useToast();

const imageA = ref(null);
const imageB = ref(null);
const pathA = ref(null);
const pathB = ref(null);
const metaA = ref(null);
const metaB = ref(null);

const isDraggingOverA = ref(false);
const isDraggingOverB = ref(false);

const fileInputA = ref(null);
const fileInputB = ref(null);

onUnmounted(() => {
  if (imageA.value?.startsWith('blob:')) URL.revokeObjectURL(imageA.value);
  if (imageB.value?.startsWith('blob:')) URL.revokeObjectURL(imageB.value);
});

const fetchMetadata = async (path, target) => {
  if (!path) return;
  try {
    const res = await api.get('/images/metadata', { params: { path } });
    if (target === 'A') metaA.value = res.data;
    else metaB.value = res.data;
  } catch (e) {
    if (target === 'A') metaA.value = null;
    else metaB.value = null;
  }
};

const handleFileSelect = (event, target) => {
  const file = event.target.files[0];
  if (!file) return;
  processFile(file, target);
};

const handleDrop = (event, target) => {
  isDraggingOverA.value = false;
  isDraggingOverB.value = false;
  const file = event.dataTransfer.files[0];
  if (!file) return;
  processFile(file, target);
};

const processFile = (file, target) => {
  try {
    const url = URL.createObjectURL(file);
    if (target === 'A') {
      if (imageA.value?.startsWith('blob:')) URL.revokeObjectURL(imageA.value);
      imageA.value = url;
      pathA.value = file.path;
      fetchMetadata(file.path, 'A');
    } else {
      if (imageB.value?.startsWith('blob:')) URL.revokeObjectURL(imageB.value);
      imageB.value = url;
      pathB.value = file.path;
      fetchMetadata(file.path, 'B');
    }
  } catch (e) {
    console.error('Failed to process file', e);
  }
};

const handleDragLeave = (e, target) => {
  if (!e.currentTarget.contains(e.relatedTarget)) {
    if (target === 'A') isDraggingOverA.value = false;
    else isDraggingOverB.value = false;
  }
};

const reset = () => {
  if (imageA.value?.startsWith('blob:')) URL.revokeObjectURL(imageA.value);
  if (imageB.value?.startsWith('blob:')) URL.revokeObjectURL(imageB.value);
  imageA.value = null;
  imageB.value = null;
  pathA.value = null;
  pathB.value = null;
  metaA.value = null;
  metaB.value = null;
};
</script>

<template>
  <div class="comparator-view-ds h-full flex flex-column p-4 overflow-hidden">
    <div class="flex flex-column align-items-center mb-4 flex-shrink-0">
      <h1 class="comp-title-ds m-0">Comparator</h1>
      <p class="text-secondary mt-1 m-0">Compare images side-by-side by dropping them into the slots below.</p>
    </div>

    <div v-if="imageA && imageB" class="flex-grow-1 flex gap-3 overflow-hidden">
      <ComparisonMetadataPanel :metadata="metaA" :path="pathA" title="Image A" />

      <div class="flex-grow-1 flex flex-column overflow-hidden">
        <ImageSplitViewer :imageA="imageA" :imageB="imageB" />
        <div class="flex justify-content-center mt-3 flex-shrink-0">
          <LButton variant="secondary" size="md" @click="reset">
            <template #icon><RotateCcw :size="16" /></template>
            Reset / Clear
          </LButton>
        </div>
      </div>

      <ComparisonMetadataPanel :metadata="metaB" :path="pathB" title="Image B" />
    </div>

    <div v-else class="flex-grow-1 flex align-items-center justify-content-center gap-4">
      <!-- Dropzone A -->
      <div
        class="drop-zone-ds p-4 flex flex-column align-items-center justify-content-center cursor-pointer relative border-round"
        :class="{ 'drop-zone-active': isDraggingOverA }"
        @click="fileInputA.click()"
        @dragover.prevent
        @dragenter="isDraggingOverA = true"
        @dragleave="handleDragLeave($event, 'A')"
        @drop.prevent="handleDrop($event, 'A')"
      >
        <input type="file" ref="fileInputA" class="hidden" accept="image/*" @change="handleFileSelect($event, 'A')" />

        <div v-if="imageA" class="w-full h-full absolute top-0 left-0 p-3 pointer-events-none flex align-items-center justify-content-center">
          <img :src="imageA" class="max-w-full max-h-full border-round shadow-4" style="object-fit: contain;" />
        </div>
        <div v-else class="text-center relative z-1 pointer-events-none">
          <ImageIcon :size="48" class="text-secondary mb-3" />
          <div class="font-bold text-xl mb-1 text-white">Image A (Left)</div>
          <div class="text-secondary text-sm">Drop or Click to browse</div>
        </div>
      </div>

      <!-- Dropzone B -->
      <div
        class="drop-zone-ds p-4 flex flex-column align-items-center justify-content-center cursor-pointer relative border-round"
        :class="{ 'drop-zone-active': isDraggingOverB }"
        @click="fileInputB.click()"
        @dragover.prevent
        @dragenter="isDraggingOverB = true"
        @dragleave="handleDragLeave($event, 'B')"
        @drop.prevent="handleDrop($event, 'B')"
      >
        <input type="file" ref="fileInputB" class="hidden" accept="image/*" @change="handleFileSelect($event, 'B')" />

        <div v-if="imageB" class="w-full h-full absolute top-0 left-0 p-3 pointer-events-none flex align-items-center justify-content-center">
          <img :src="imageB" class="max-w-full max-h-full border-round shadow-4" style="object-fit: contain;" />
        </div>
        <div v-else class="text-center relative z-1 pointer-events-none">
          <ImageIcon :size="48" class="text-secondary mb-3" />
          <div class="font-bold text-xl mb-1 text-white">Image B (Right)</div>
          <div class="text-secondary text-sm">Drop or Click to browse</div>
        </div>
      </div>
    </div>
  </div>
</template>

<style scoped>
.comparator-view-ds {
  background: var(--color-bg-canvas, #0A0A0D);
}

.comp-title-ds {
  font-family: var(--font-sans, Inter, sans-serif);
  font-size: 28px;
  font-weight: 800;
  letter-spacing: -0.02em;
  background: var(--gradient-brand-text, linear-gradient(90deg, #67E0D8, #9B7EF5));
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  background-clip: text;
}

.drop-zone-ds {
  width: 300px;
  height: 300px;
  background: var(--color-surface-1, #14151B);
  border: 2px dashed var(--color-border-default, rgba(255, 255, 255, 0.10));
  border-radius: var(--radius-lg, 12px);
  position: relative;
  z-index: 1;
  transition: all var(--duration-fast, 120ms) var(--ease-standard);
}

.drop-zone-ds:hover, .drop-zone-active {
  background: var(--color-surface-2, #23252F);
  border-color: var(--color-accent-primary, #67E0D8);
  transform: translateY(-4px);
}

.text-secondary {
  color: var(--color-text-secondary, #9294A3);
}

.text-white {
  color: var(--color-text-primary, #F2F3F7);
}
</style>
