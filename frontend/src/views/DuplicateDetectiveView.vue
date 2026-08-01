<script setup>
/**
 * @file DuplicateDetectiveView.vue
 * @description A specialized view for identifying and resolving visual and exact duplicates aligned with the Latent Design System.
 */
import { ref, onMounted, computed, onUnmounted } from 'vue';
import api, { authenticatedUrl } from '@/services/api';
import LButton from '@/components/ds/LButton.vue';
import Dialog from 'primevue/dialog';
import Dropdown from 'primevue/dropdown';
import { useToast } from 'primevue/usetoast';
import ImageSplitViewer from '@/components/ImageSplitViewer.vue';
import ComparisonMetadataPanel from '@/components/ComparisonMetadataPanel.vue';
import { Search, Trash2, Check, FastForward, RefreshCw } from 'lucide-vue-next';

const toast = useToast();
const splitViewerRef = ref(null);

const pairs = ref([]);
const currentIndex = ref(0);
const status = ref({ missingHashes: 0, totalImages: 0 });
const isScanning = ref(false);
const isResolving = ref(false);

const showResolveDialog = ref(false);
const selectedStrategy = ref('LATEST_SCANNED');
const strategies = [
  { label: 'Keep Latest (Most Recently Scanned)', value: 'LATEST_SCANNED' },
  { label: 'Keep Oldest (First Scanned)', value: 'OLDEST_SCANNED' },
  { label: 'Keep Best Resolution (Highest Pixel Count)', value: 'BEST_RESOLUTION' },
  { label: 'Keep Largest Filesize', value: 'LARGEST_FILESIZE' }
];

const currentPair = computed(() => pairs.value[currentIndex.value] || null);
const leftImage = computed(() => currentPair.value ? authenticatedUrl(`/api/images/content?path=${encodeURIComponent(currentPair.value.left.path)}`) : null);
const rightImage = computed(() => currentPair.value ? authenticatedUrl(`/api/images/content?path=${encodeURIComponent(currentPair.value.right.path)}`) : null);

const loadStatus = async () => {
  try {
    const res = await api.get('/duplicates/status');
    status.value = res.data;
  } catch (e) {
    console.error('Failed to load duplicate status', e);
  }
};

const loadPairs = async () => {
  try {
    const res = await api.get('/duplicates/pairs');
    pairs.value = res.data;
    currentIndex.value = 0;
    await loadStatus();
  } catch (e) {
    console.error('Failed to load duplicate pairs', e);
  }
};

const scanHashes = async () => {
  isScanning.value = true;
  try {
    await api.post('/duplicates/scan');
    toast.add({ severity: 'info', summary: 'Scan Complete', detail: 'Finished calculating image hashes', life: 3000 });
    await loadPairs();
  } catch (e) {
    console.error('Failed to scan hashes', e);
  } finally {
    isScanning.value = false;
  }
};

const keepLeft = async () => {
  if (!currentPair.value) return;
  await deleteFile(currentPair.value.right.path);
  removeCurrentPair();
};

const keepRight = async () => {
  if (!currentPair.value) return;
  await deleteFile(currentPair.value.left.path);
  removeCurrentPair();
};

const deleteFile = async (path) => {
  try {
    await api.post('/images/batch/delete', [path]);
    toast.add({ severity: 'success', summary: 'Resolved', detail: 'Duplicate removed', life: 1000 });
  } catch (e) {
    console.error('Failed to delete file', e);
  }
};

const removeCurrentPair = () => {
  pairs.value.splice(currentIndex.value, 1);
  if (currentIndex.value >= pairs.value.length) {
    currentIndex.value = Math.max(0, pairs.value.length - 1);
  }
  splitViewerRef.value?.resetZoom();
};

const skip = () => {
  if (currentIndex.value < pairs.value.length - 1) {
    currentIndex.value++;
  } else {
    currentIndex.value = 0;
  }
  splitViewerRef.value?.resetZoom();
};

const resolveDuplicates = async () => {
  isResolving.value = true;
  try {
    const res = await api.post('/duplicates/resolve-all', null, {
      params: { strategy: selectedStrategy.value }
    });
    toast.add({ severity: 'success', summary: 'Resolved', detail: res.data, life: 3000 });
    showResolveDialog.value = false;
    await loadPairs();
  } catch (e) {
    console.error('Failed to resolve duplicates', e);
  } finally {
    isResolving.value = false;
  }
};

const handleKeydown = (e) => {
  if (e.target.tagName === 'INPUT' || showResolveDialog.value) return;
  switch (e.key) {
    case '1': keepLeft(); break;
    case '2': keepRight(); break;
    case ' ': skip(); break;
    case 'Escape': splitViewerRef.value?.resetZoom(); break;
  }
};

onMounted(() => {
  loadPairs();
  window.addEventListener('keydown', handleKeydown);
});

onUnmounted(() => {
  window.removeEventListener('keydown', handleKeydown);
});
</script>

<template>
  <div class="duplicate-view-ds h-full flex flex-column p-4 overflow-hidden">
    <div class="flex justify-content-between align-items-center mb-4 flex-shrink-0">
      <div>
        <h1 class="text-3xl font-bold m-0 mb-1 dupe-title-ds">Duplicate Detective</h1>
        <div class="flex align-items-center gap-3">
          <p class="text-secondary m-0" v-if="pairs.length > 0">
            Found {{ pairs.length }} potential duplicate pairs.
          </p>
          <p class="text-secondary m-0" v-else>No duplicates found.</p>

          <span v-if="status.missingHashes > 0" class="missing-badge-ds">
            {{ status.missingHashes }} images missing hashes
          </span>
        </div>
      </div>
      <div class="flex gap-2">
        <LButton variant="secondary" size="md" :disabled="isScanning" @click="scanHashes">
          <template #icon><Search :size="16" /></template>
          Scan for Duplicates
        </LButton>

        <LButton variant="danger" size="md" :disabled="pairs.length === 0" @click="showResolveDialog = true">
          <template #icon><Trash2 :size="16" /></template>
          Resolve Duplicates
        </LButton>
      </div>
    </div>

    <div v-if="currentPair" class="flex-grow-1 flex gap-3 overflow-hidden">
      <ComparisonMetadataPanel
        :metadata="currentPair.left.metadata"
        :path="currentPair.left.path"
        :rating="currentPair.left.rating"
        title="Left (1)"
        actionLabel="Keep Left (1)"
        @action="keepLeft"
      />

      <div class="flex-grow-1 flex flex-column overflow-hidden">
        <ImageSplitViewer ref="splitViewerRef" :imageA="leftImage" :imageB="rightImage" />
        <div class="flex justify-content-center gap-3 mt-3 flex-shrink-0">
          <LButton variant="secondary" size="md" @click="keepLeft">
            <template #icon><Check :size="16" /></template>
            Keep Left (1)
          </LButton>

          <LButton variant="secondary" size="md" @click="skip">
            <template #icon><FastForward :size="16" /></template>
            Skip (Space)
          </LButton>

          <LButton variant="secondary" size="md" @click="keepRight">
            <template #icon><Check :size="16" /></template>
            Keep Right (2)
          </LButton>
        </div>
      </div>

      <ComparisonMetadataPanel
        :metadata="currentPair.right.metadata"
        :path="currentPair.right.path"
        :rating="currentPair.right.rating"
        title="Right (2)"
        actionLabel="Keep Right (2)"
        @action="keepRight"
      />
    </div>

    <div v-else class="flex-grow-1 flex align-items-center justify-content-center">
      <div class="text-center text-secondary">
        <Check :size="64" class="text-accent mb-3" />
        <div class="text-xl text-white font-bold">All clear! No duplicates detected.</div>
        <p v-if="status.missingHashes > 0" class="mt-2 text-sm text-secondary">
          Note: {{ status.missingHashes }} images are missing hashes. Click "Scan for Duplicates" to process them.
        </p>
      </div>
    </div>

    <!-- Batch Resolve Dialog -->
    <Dialog v-model:visible="showResolveDialog" modal header="Batch Resolve Duplicates" style="width: 440px">
      <div class="flex flex-column gap-3 py-2">
        <p class="text-sm text-secondary m-0">
          Automatically resolve all {{ pairs.length }} duplicate pairs using a chosen rule:
        </p>
        <Dropdown
          v-model="selectedStrategy"
          :options="strategies"
          optionLabel="label"
          optionValue="value"
          class="w-full"
        />
      </div>
      <template #footer>
        <div class="flex justify-content-end gap-2">
          <LButton variant="secondary" size="sm" @click="showResolveDialog = false">Cancel</LButton>
          <LButton variant="danger" size="sm" :disabled="isResolving" @click="resolveDuplicates">
            Resolve All Duplicates
          </LButton>
        </div>
      </template>
    </Dialog>
  </div>
</template>

<style scoped>
.duplicate-view-ds {
  background: var(--color-bg-canvas, #0A0A0D);
}

.dupe-title-ds {
  font-family: var(--font-sans, Inter, sans-serif);
  font-size: 24px;
  font-weight: 800;
  background: var(--gradient-brand-text, linear-gradient(90deg, #67E0D8, #9B7EF5));
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  background-clip: text;
}

.missing-badge-ds {
  font-family: var(--font-mono, "JetBrains Mono", monospace);
  font-size: 11px;
  background: var(--color-surface-2, #23252F);
  border: 1px solid var(--color-border-subtle, rgba(255, 255, 255, 0.06));
  color: var(--color-warning, #F5B84E);
  padding: 2px 8px;
  border-radius: var(--radius-sm, 6px);
}

.text-accent {
  color: var(--color-accent-primary, #67E0D8);
}

.text-secondary {
  color: var(--color-text-secondary, #9294A3);
}

.text-white {
  color: var(--color-text-primary, #F2F3F7);
}
</style>
