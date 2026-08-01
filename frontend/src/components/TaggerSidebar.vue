<script setup>
/**
 * @file TaggerSidebar.vue
 * @description Sidebar component for AI Auto-Tagging controls using the WD14 model aligned with the Latent Design System.
 */
import { ref, onMounted, onUnmounted } from 'vue';
import api from '@/services/api';
import LButton from '@/components/ds/LButton.vue';
import LSlider from '@/components/ds/LSlider.vue';
import LProgressBar from '@/components/ds/LProgressBar.vue';
import { useToast } from 'primevue/usetoast';
import { useBrowserStore } from '@/stores/browser';
import { Download, Tag, Tags, Folder, CloudDownload, X } from 'lucide-vue-next';

const store = useBrowserStore();
const toast = useToast();


const modelStatus = ref({
  ready: false,
  downloading: false,
  progress: 0
});

const threshold = ref(35);
const isTagging = ref(false);
let pollInterval = null;

const checkStatus = async () => {
  try {
    const res = await api.get('/tagger/status');
    modelStatus.value = res.data;
  } catch (e) {
    console.error('Failed to check tagger status', e);
  }
};

const downloadModel = async () => {
  try {
    await api.post('/tagger/download');
    modelStatus.value.downloading = true;
    toast.add({ severity: 'info', summary: 'Download Started', detail: 'Downloading WD14 model...', life: 3000 });
  } catch (e) {}
};

const startTagging = async (single = false) => {
  const path = single ? store.selectedFile : store.lastFolderPath;

  if (!path) {
    toast.add({ severity: 'warn', summary: 'No Target', detail: 'Please select a file or folder.', life: 3000 });
    return;
  }

  isTagging.value = true;
  try {
    const res = await api.post('/tagger/tag-folder', null, {
      params: {
        path: path,
        threshold: threshold.value / 100.0
      }
    });

    toast.add({ severity: 'success', summary: 'Started', detail: res.data, life: 3000 });

    if (single) {
      setTimeout(async () => {
        await store.fetchMetadata(path);
        isTagging.value = false;
        toast.add({ severity: 'success', summary: 'Done', detail: 'AI Tags updated', life: 2000 });
      }, 3000);
    } else {
      setTimeout(() => {
        isTagging.value = false;
        toast.add({ severity: 'success', summary: 'Completed', detail: 'Folder tagging finished', life: 3000 });
      }, 8000);
    }
  } catch (e) {
    isTagging.value = false;
  }
};

onMounted(() => {
  checkStatus();
  pollInterval = setInterval(checkStatus, 2000);
});

onUnmounted(() => {
  if (pollInterval) clearInterval(pollInterval);
});
</script>

<template>
  <aside class="tagger-sidebar-ds h-full flex flex-column p-4" style="width: 380px; min-width: 380px;">
    <div class="flex align-items-center justify-content-between mb-4">
      <button class="icon-btn-ds" title="Close AI Auto-Tagger" @click="store.isTaggerOpen = false">
        <X :size="16" />
      </button>
      <div class="tagger-title-ds flex-grow-1 text-center pr-4">AI Auto-Tagger</div>
    </div>


    <div
      v-if="!modelStatus.ready"
      class="flex-grow-1 flex flex-column align-items-center justify-content-center text-center gap-4"
    >
      <div class="cloud-icon-box">
        <CloudDownload :size="48" class="text-accent" />
      </div>
      <h2 class="text-lg font-bold text-white m-0">Model Required</h2>
      <p class="text-sm text-secondary m-0">The WD14 ONNX model (~300MB) is needed for automatic image tagging.</p>

      <div v-if="modelStatus.downloading" class="w-full px-3">
        <LProgressBar :value="modelStatus.progress" />
        <span class="text-xs text-secondary mt-2 block">Downloading model... {{ modelStatus.progress }}%</span>
      </div>

      <LButton
        v-else
        variant="primary"
        size="md"
        @click="downloadModel"
      >
        <template #icon><Download :size="16" /></template>
        Download Model
      </LButton>
    </div>

    <div v-else class="flex flex-column gap-5">
      <div>
        <div class="flex justify-content-between align-items-center mb-2">
          <span class="section-label-ds">Confidence Threshold</span>
          <span class="value-badge-ds">{{ threshold }}%</span>
        </div>
        <LSlider v-model="threshold" :min="10" :max="90" />
      </div>

      <div class="flex flex-column gap-3">
        <span class="section-label-ds">Actions</span>
        <LButton
          variant="primary"
          size="md"
          :disabled="!store.selectedFile || isTagging"
          @click="startTagging(true)"
        >
          <template #icon><Tag :size="16" /></template>
          Tag Current Image
        </LButton>

        <LButton
          variant="secondary"
          size="md"
          :disabled="!store.lastFolderPath || isTagging"
          @click="startTagging(false)"
        >
          <template #icon><Tags :size="16" /></template>
          Tag Entire Folder
        </LButton>
      </div>

      <div class="mt-4 p-3 ds-card">
        <div class="section-label-ds mb-2">Current Target</div>
        <div class="target-path-text flex align-items-center gap-2 text-sm text-white">
          <Folder :size="16" class="text-accent" />
          <span class="text-overflow-ellipsis overflow-hidden white-space-nowrap">
            {{ store.lastFolderPath || 'None' }}
          </span>
        </div>
      </div>
    </div>
  </aside>
</template>

<style scoped>
.tagger-sidebar-ds {
  background: var(--color-surface-1, #14151B);
  border-left: 1px solid var(--color-border-subtle, rgba(255, 255, 255, 0.06));
  box-shadow: var(--shadow-panel, 0 20px 60px -20px rgba(0,0,0,0.65));
}

.tagger-title-ds {
  font-family: var(--font-sans, Inter, sans-serif);
  font-size: 18px;
  font-weight: 800;
  letter-spacing: -0.02em;
  background: var(--gradient-brand-text, linear-gradient(90deg, #4FD8D0, #9B7EF5));
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  background-clip: text;
}

.cloud-icon-box {
  width: 80px;
  height: 80px;
  border-radius: var(--radius-lg, 12px);
  background: var(--color-surface-2, #23252F);
  border: 1px solid var(--color-border-subtle, rgba(255, 255, 255, 0.06));
  display: flex;
  align-items: center;
  justify-content: center;
}

.text-accent {
  color: var(--color-accent-primary, #4FD8D0);
}

.text-secondary {
  color: var(--color-text-secondary, #9294A3);
}

.section-label-ds {
  font-family: var(--font-sans, Inter, sans-serif);
  font-size: 11px;
  font-weight: 700;
  text-transform: uppercase;
  letter-spacing: 0.05em;
  color: var(--color-text-secondary, #9294A3);
}

.value-badge-ds {
  font-family: var(--font-mono, "JetBrains Mono", monospace);
  font-size: 12px;
  color: var(--color-accent-primary, #4FD8D0);
  font-weight: 600;
}

.ds-card {
  background: var(--color-surface-2, #23252F);
  border: 1px solid var(--color-border-subtle, rgba(255, 255, 255, 0.06));
  border-radius: var(--radius-md, 8px);
}

.icon-btn-ds {
  width: 28px;
  height: 28px;
  border-radius: var(--radius-sm, 6px);
  border: 1px solid transparent;
  background: transparent;
  color: var(--color-text-secondary, #9294A3);
  display: flex;
  align-items: center;
  justify-content: center;
  cursor: pointer;
  transition: all var(--duration-fast, 120ms) var(--ease-standard);
}

.icon-btn-ds:hover {
  background: var(--color-surface-2, #23252F);
  color: var(--color-text-primary, #F2F3F7);
  border-color: var(--color-border-subtle, rgba(255, 255, 255, 0.06));
}
</style>

