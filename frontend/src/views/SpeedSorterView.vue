<script setup>
/**
 * @file SpeedSorterView.vue
 * @description A high-efficiency, keyboard-driven tool for rapid image organization aligned with the Latent Design System.
 */
import { ref, onMounted, onUnmounted, computed } from 'vue';
import api, { authenticatedUrl } from '@/services/api';
import LButton from '@/components/ds/LButton.vue';
import { useToast } from 'primevue/usetoast';
import { FolderOpen, Zap, Folder, Trash2, RotateCcw, ArrowRight, ArrowLeft } from 'lucide-vue-next';

const toast = useToast();

const inputDir = ref(null);
const targets = ref([]);
const files = ref([]);
const currentIndex = ref(0);
const history = ref([]);

const currentFile = computed(() => files.value[currentIndex.value] || null);
const currentImageUrl = computed(() => currentFile.value ? authenticatedUrl(`/api/images/content?path=${encodeURIComponent(currentFile.value)}`) : null);
const progress = computed(() => `${currentIndex.value + 1} / ${files.value.length}`);

const loadConfig = async () => {
  try {
    const res = await api.get('/speedsorter/config');
    inputDir.value = res.data.inputDir;
    targets.value = res.data.targets;
    if (inputDir.value) loadFiles();
  } catch (e) {
    console.error('Failed to load config', e);
  }
};

const loadFiles = async () => {
  try {
    const res = await api.get('/speedsorter/files');
    files.value = res.data;
    currentIndex.value = 0;
  } catch (e) {
    console.error('Failed to load files', e);
  }
};

const selectInput = async () => {
  try {
    if (window.electronAPI) {
      const path = await window.electronAPI.selectFolder();
      if (path) {
        await api.post('/speedsorter/config/input', null, { params: { path } });
        await loadConfig();
      }
    } else {
      const path = prompt('Enter absolute path to Input Folder:', inputDir.value || '');
      if (path) {
        await api.post('/speedsorter/config/input', null, { params: { path } });
        await loadConfig();
      }
    }
  } catch (e) {
    console.error('Failed to select input folder', e);
  }
};

const selectTarget = async (index) => {
  try {
    if (window.electronAPI) {
      const path = await window.electronAPI.selectFolder();
      if (path) {
        await api.post('/speedsorter/config/target', null, { params: { index, path } });
        await loadConfig();
      }
    } else {
      const current = targets.value.find(t => t.index == index)?.path || '';
      const path = prompt(`Enter absolute path for Target ${index + 1}:`, current);
      if (path) {
        await api.post('/speedsorter/config/target', null, { params: { index, path } });
        await loadConfig();
      }
    }
  } catch (e) {
    console.error('Failed to select target folder', e);
  }
};

const moveFile = async (targetIndex) => {
  if (!currentFile.value) return;

  const target = targets.value.find(t => t.index == targetIndex);
  if (!target || !target.path) {
    toast.add({
      severity: 'warn',
      summary: 'Target Not Set',
      detail: `Target ${targetIndex + 1} is missing`,
      life: 2000
    });
    return;
  }

  const fileToMove = currentFile.value;
  try {
    const res = await api.post('/speedsorter/move', null, {
      params: { source: fileToMove, targetIndex }
    });

    history.value.push({ source: fileToMove, dest: res.data, isDelete: false });
    files.value.splice(currentIndex.value, 1);

    toast.add({
      severity: 'success',
      summary: 'Moved',
      detail: `Moved to ${target.name || 'Target ' + (targetIndex + 1)}`,
      life: 1000
    });
  } catch (e) {
    console.error('Failed to move file', e);
  }
};

const deleteFile = async () => {
  if (!currentFile.value) return;
  const fileToDelete = currentFile.value;

  try {
    await api.post('/speedsorter/delete', null, { params: { source: fileToDelete } });
    history.value.push({ source: fileToDelete, isDelete: true });
    files.value.splice(currentIndex.value, 1);

    toast.add({ severity: 'info', summary: 'Recycled', detail: 'Moved to Recycle Bin', life: 1000 });
  } catch (e) {
    console.error('Failed to delete file', e);
  }
};

const next = () => {
  if (currentIndex.value < files.value.length - 1) currentIndex.value++;
};

const prev = () => {
  if (currentIndex.value > 0) currentIndex.value--;
};

const undo = async () => {
  if (history.value.length === 0) return;
  const lastAction = history.value.pop();

  try {
    await api.post('/speedsorter/undo', lastAction);
    files.value.splice(currentIndex.value, 0, lastAction.source);
    toast.add({ severity: 'info', summary: 'Undone', detail: 'Action reverted', life: 1000 });
  } catch (e) {
    console.error('Failed to undo', e);
  }
};

const handleKeydown = (e) => {
  if (e.target.tagName === 'INPUT') return;

  if (e.ctrlKey && (e.key === 'z' || e.key === 'Z')) {
    undo();
    return;
  }

  switch (e.key) {
    case '1': moveFile(0); break;
    case '2': moveFile(1); break;
    case '3': moveFile(2); break;
    case '4': moveFile(3); break;
    case '5': moveFile(4); break;
    case 'Delete':
    case 'x':
    case 'X': deleteFile(); break;
    case 'ArrowRight':
    case ' ': next(); break;
    case 'ArrowLeft': prev(); break;
  }
};

onMounted(() => {
  loadConfig();
  window.addEventListener('keydown', handleKeydown);
});

onUnmounted(() => {
  window.removeEventListener('keydown', handleKeydown);
});
</script>

<template>
  <div class="speed-sorter-container flex flex-column h-full overflow-hidden p-3">
    <!-- Top Header -->
    <div class="sorter-header flex align-items-center justify-content-between p-3 border-round mb-3 flex-shrink-0">
      <div class="flex align-items-center gap-3">
        <span class="sorter-title-ds">Speed Sorter</span>
        <LButton variant="secondary" size="sm" @click="selectInput">
          <template #icon><FolderOpen :size="14" /></template>
          Select Input
        </LButton>
        <span class="text-xs text-secondary font-mono">{{ inputDir || 'No input folder selected' }}</span>
      </div>
      <span class="text-sm font-mono font-bold text-accent">{{ progress }}</span>
    </div>

    <!-- Main Image Display -->
    <div class="flex-grow-1 flex gap-3 overflow-hidden min-h-0">
      <div class="flex-grow-1 sorter-display-card border-round flex align-items-center justify-content-center relative overflow-hidden">
        <img v-if="currentImageUrl" :src="currentImageUrl" class="max-w-full max-h-full" style="object-fit: contain;" alt="Speed Sorter Preview" />
        <div v-else class="text-lg text-secondary">
          {{ inputDir ? 'No images found or all processed' : 'Select an input folder to start' }}
        </div>
      </div>
    </div>

    <!-- Bottom Action Bar (Fixed, zero overflow clipping) -->
    <div class="sorter-bottom-bar p-3 border-round flex justify-content-center align-items-center gap-4 mt-3 flex-shrink-0">
      <div v-for="(target, i) in targets" :key="i" class="flex flex-column align-items-center gap-1">
        <span class="key-badge-ds">Key [{{ i + 1 }}]</span>
        <button
          class="target-btn-ds text-overflow-ellipsis"
          :class="{ 'target-btn-empty': !target.path }"
          :title="target.path || 'Click to set target folder'"
          @click="selectTarget(i)"
        >
          {{ target.name || 'Set Folder' }}
        </button>
      </div>

      <div class="divider-v-ds"></div>

      <div class="flex flex-column gap-1 text-xs text-secondary justify-content-center">
        <span><strong class="text-white">DEL / X</strong> : Recycle Bin</span>
        <span><strong class="text-white">Ctrl+Z</strong> : Undo</span>
        <span><strong class="text-white">SPACE</strong> : Skip</span>
      </div>
    </div>
  </div>
</template>

<style scoped>
.speed-sorter-container {
  background: var(--color-bg-canvas, #0A0A0D);
  box-sizing: border-box;
}

.sorter-header {
  background: var(--color-surface-1, #14151B);
  border: 1px solid var(--color-border-subtle, rgba(255, 255, 255, 0.06));
}

.sorter-title-ds {
  font-family: var(--font-sans, Inter, sans-serif);
  font-size: 18px;
  font-weight: 800;
  background: var(--gradient-brand-text, linear-gradient(90deg, #67E0D8, #9B7EF5));
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  background-clip: text;
}

.sorter-display-card {
  background: var(--color-surface-1, #14151B);
  border: 1px solid var(--color-border-subtle, rgba(255, 255, 255, 0.06));
}

.sorter-bottom-bar {
  background: var(--color-surface-1, #14151B);
  border: 1px solid var(--color-border-subtle, rgba(255, 255, 255, 0.06));
}

.key-badge-ds {
  font-family: var(--font-mono, "JetBrains Mono", monospace);
  font-size: 11px;
  color: var(--color-accent-primary, #67E0D8);
  font-weight: 700;
}

.target-btn-ds {
  height: 32px;
  width: 130px;
  background: var(--color-surface-2, #23252F);
  border: 1px solid var(--color-border-default, rgba(255, 255, 255, 0.10));
  border-radius: var(--radius-sm, 6px);
  color: var(--color-text-primary, #F2F3F7);
  font-family: var(--font-sans, Inter, sans-serif);
  font-size: 12px;
  font-weight: 500;
  cursor: pointer;
  padding: 0 8px;
  transition: all var(--duration-fast, 120ms) var(--ease-standard);
}

.target-btn-ds:hover {
  background: var(--color-surface-3, #2E303E);
  border-color: var(--color-border-hover, rgba(255, 255, 255, 0.18));
}

.target-btn-empty {
  border-style: dashed !important;
  color: var(--color-text-secondary, #9294A3) !important;
}

.divider-v-ds {
  width: 1px;
  height: 36px;
  background: var(--color-border-subtle, rgba(255, 255, 255, 0.06));
}

.text-accent {
  color: var(--color-accent-primary, #67E0D8);
}

.text-secondary {
  color: var(--color-text-secondary, #9294A3);
}
</style>