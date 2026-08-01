<script setup>
/**
 * @file ScrubView.vue
 * @description A security and privacy utility for stripping sensitive metadata from image files aligned with the Latent Design System.
 */
import { ref } from 'vue';
import api from '@/services/api';
import LButton from '@/components/ds/LButton.vue';
import LCard from '@/components/ds/LCard.vue';
import LIconButton from '@/components/ds/LIconButton.vue';
import { useToast } from 'primevue/usetoast';
import { Shield, Download, X, Upload } from 'lucide-vue-next';

const toast = useToast();
const fileInput = ref(null);
const uploadedFile = ref(null);
const previewUrl = ref(null);
const isProcessing = ref(false);
const isDragging = ref(false);

const triggerFileInput = () => {
  if (fileInput.value) fileInput.value.click();
};

const handleFileSelect = (event) => {
  const file = event.target.files[0];
  if (file) processSelectedFile(file);
};

const handleDrop = (event) => {
  isDragging.value = false;
  const file = event.dataTransfer.files[0];
  if (file) processSelectedFile(file);
};

const processSelectedFile = async (file) => {
  const formData = new FormData();
  formData.append('file', file);

  try {
    const res = await api.post('/scrub/upload', formData, {
      headers: { 'Content-Type': 'multipart/form-data' }
    });
    uploadedFile.value = res.data.filename;
    previewUrl.value = URL.createObjectURL(file);
    toast.add({ severity: 'info', summary: 'File Uploaded', detail: 'Ready for metadata scrubbing', life: 2000 });
  } catch (error) {
    console.error('Upload failed', error);
  } finally {
    if (fileInput.value) fileInput.value.value = '';
  }
};

const scrubAndDownload = async () => {
  if (!uploadedFile.value) return;
  isProcessing.value = true;

  try {
    const response = await api.post('/scrub/process', null, {
      params: { filename: uploadedFile.value },
      responseType: 'blob'
    });

    const url = window.URL.createObjectURL(new Blob([response.data]));
    const link = document.createElement('a');
    link.href = url;

    const contentDisposition = response.headers['content-disposition'];
    let fileName = 'clean_image.png';
    if (contentDisposition) {
      const fileNameMatch = contentDisposition.match(/filename="?([^"]+)"?/);
      if (fileNameMatch && fileNameMatch.length === 2) {
        fileName = fileNameMatch[1];
      }
    }

    link.setAttribute('download', fileName);
    document.body.appendChild(link);
    link.click();
    document.body.removeChild(link);
    window.URL.revokeObjectURL(url);

    toast.add({ severity: 'success', summary: 'Success', detail: 'Metadata scrubbed & downloaded', life: 3000 });
  } catch (error) {
    console.error('Scrubbing failed', error);
  } finally {
    isProcessing.value = false;
  }
};

const clear = () => {
  uploadedFile.value = null;
  previewUrl.value = null;
  if (fileInput.value) fileInput.value.value = '';
};
</script>

<template>
  <div class="flex flex-column h-full p-4 overflow-hidden scrub-container-ds">
    <div class="flex flex-column align-items-center mb-4 flex-shrink-0">
      <h1 class="view-title-hero mb-2">Metadata Scrubber</h1>
      <p class="view-subtitle">Remove hidden metadata (EXIF, Prompts, Workflow) for total privacy.</p>
    </div>

    <div class="flex-grow-1 flex align-items-center justify-content-center">
      <LCard class="w-full max-w-30rem p-4">
        <div
          v-if="!previewUrl"
          class="drop-zone-ds flex flex-column align-items-center gap-4 py-5 cursor-pointer border-round"
          :class="{ 'drop-zone-active': isDragging }"
          @click="triggerFileInput"
          @dragover.prevent
          @dragenter.prevent="isDragging = true"
          @dragleave.prevent="isDragging = false"
          @drop.prevent="handleDrop"
        >
          <input
            type="file"
            ref="fileInput"
            class="hidden"
            accept="image/png, image/jpeg, image/webp"
            @change="handleFileSelect"
          />

          <div class="shield-box">
            <Shield :size="40" class="text-accent" />
          </div>
          <div class="text-center pointer-events-none">
            <div class="font-bold text-lg mb-1 text-white">Drop Image Here</div>
            <div class="text-secondary text-sm">or click to browse</div>
          </div>
          <span class="text-xs text-secondary pointer-events-none">Supports PNG, JPG, WEBP</span>
        </div>

        <div v-else class="flex flex-column align-items-center gap-4">
          <div class="relative border-round overflow-hidden preview-box" style="max-height: 280px;">
            <img :src="previewUrl" class="block max-w-full h-auto" style="max-height: 280px; object-fit: contain;" alt="Preview" />
          </div>

          <div class="flex gap-3 w-full">
            <LButton
              variant="primary"
              size="md"
              class="flex-grow-1"
              :disabled="isProcessing"
              @click="scrubAndDownload"
            >
              <template #icon><Download :size="16" /></template>
              Export Clean Copy
            </LButton>

            <LIconButton title="Clear" @click="clear">
              <X :size="16" />
            </LIconButton>
          </div>
        </div>
      </LCard>
    </div>
  </div>
</template>

<style scoped>
.scrub-container-ds {
  background: var(--color-bg-canvas, #0A0A0D);
}

.shield-box {
  width: 72px;
  height: 72px;
  border-radius: 50%;
  background: var(--color-surface-2, #23252F);
  border: 1px solid var(--color-border-subtle, rgba(255, 255, 255, 0.06));
  display: flex;
  align-items: center;
  justify-content: center;
}

.drop-zone-ds {
  background: var(--color-surface-2, #23252F);
  border: 2px dashed var(--color-border-default, rgba(255, 255, 255, 0.10));
  border-radius: var(--radius-lg, 12px);
  transition: all var(--duration-fast, 120ms) var(--ease-standard);
}

.drop-zone-ds:hover, .drop-zone-active {
  background: var(--color-surface-3, #2E303E);
  border-color: var(--color-accent-primary, #67E0D8);
}

.text-accent {
  color: var(--color-accent-primary, #67E0D8);
}

.text-secondary {
  color: var(--color-text-secondary, #9294A3);
}

.preview-box {
  border: 1px solid var(--color-border-subtle, rgba(255, 255, 255, 0.06));
  border-radius: var(--radius-md, 8px);
}
</style>
