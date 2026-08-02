<script setup>
/**
 * @file ImageBrowserView.vue
 * @description The primary application view for exploring and interacting with the image library.
 *
 * This view serves as the central hub of the application, providing a dual-mode interface for
 * browsing images. It integrates a high-performance virtualized gallery for bulk viewing and
 * a single-image viewer for detailed inspection. The view orchestrates complex interactions
 * between the toolbar, sidebars (metadata and AI tagging), and the core image display.
 *
 * Key functionalities:
 * - Dual View Modes: Seamlessly toggles between a 'Gallery' (grid) and 'Browser' (single) view.
 * - Keyboard Navigation: Implements comprehensive hotkeys for navigation (Arrows/WASD),
 * view switching (G/B), and file management (F2 for rename).
 * - Contextual Actions: Provides a rich right-click menu for collection management,
 * file system operations (Open in Explorer), and deletion.
 * - State Synchronization: Coordinates with the Vuex/Pinia store to maintain selection state,
 * filter criteria, and sidebar visibility across view transitions.
 * - Batch Processing: Supports multi-selection for bulk operations like adding to collections
 * or moving files to the trash.
 */
import {onMounted, onUnmounted, ref, watch} from 'vue';
import {useBrowserStore} from '@/stores/browser';
import {useRoute, useRouter} from 'vue-router';
import api from '@/services/api';
import BrowserToolbar from '@/components/BrowserToolbar.vue';
import MetadataSidebar from '@/components/MetadataSidebar.vue';
import TaggerSidebar from '@/components/TaggerSidebar.vue';
import VirtualGallery from '@/components/VirtualGallery.vue';
import SingleImageViewer from '@/components/SingleImageViewer.vue';
import CustomContextMenu from '@/components/CustomContextMenu.vue';
import {useToast} from 'primevue/usetoast';
import Dialog from 'primevue/dialog';
import InputText from 'primevue/inputtext';
import LButton from '@/components/ds/LButton.vue';
import { Check, X } from 'lucide-vue-next';

const store = useBrowserStore();
const route = useRoute();
const router = useRouter();
const toast = useToast();
const containerRef = ref(null);
const virtualGalleryRef = ref(null);
const cm = ref();
const menuModel = ref([]);
const contextMenuSelection = ref(null);

const showRenameDialog = ref(false);
const newFileName = ref('');
const fileToRename = ref(null);

const handleKeydown = (e) => {
  if (e.target.tagName === 'INPUT' || e.target.tagName === 'TEXTAREA') return;

  if (['ArrowUp', 'ArrowDown', 'ArrowLeft', 'ArrowRight', ' '].includes(e.key)) {
    e.preventDefault();
  }

  const cols = (store.viewMode === 'gallery' && virtualGalleryRef.value) ? virtualGalleryRef.value.gridCols : 1;

  switch (e.key) {
    case 'ArrowLeft':
    case 'a':
    case 'A':
      store.navigate(-1);
      break;

    case 'ArrowRight':
    case 'd':
    case 'D':
      store.navigate(1);
      break;

    case 'ArrowUp':
    case 'w':
    case 'W':
      if (store.viewMode === 'gallery') store.navigate(-cols);
      break;

    case 'ArrowDown':
    case 's':
    case 'S':
      if (store.viewMode === 'gallery') store.navigate(cols);
      break;

    case 'g':
    case 'G':
      store.setViewMode('gallery');
      break;

    case 'b':
    case 'B':
      store.setViewMode('browser');
      break;

    case 'Enter':
      if (store.viewMode === 'gallery') {
        store.setViewMode('browser');
        store.setSidebarOpen(true);
      }
      break;

    case 'F2':
      if (store.selectedFile) {
        openRenameDialog(store.selectedFile);
      }
      break;

    case 'Escape':
      if (store.viewMode === 'browser') store.setViewMode('gallery');
      break;
  }
};

const onContextMenu = async (payload) => {
  const {event, file} = payload;
  contextMenuSelection.value = file;

  let collections = [];
  try {
    const res = await api.get('/collections');
    collections = res.data;
  } catch (e) {
    console.error("Failed to fetch collections for context menu", e);
  }

  const selectedCount = store.selectedFiles.size;
  const isBatch = selectedCount > 1;
  const batchLabel = isBatch ? ` (${selectedCount} items)` : '';

  const addToCollectionItems = collections.map(col => ({
    label: col.name,
    icon: 'pi pi-folder',
    command: () => addToCollection(col.name, isBatch ? Array.from(store.selectedFiles) : [file.path])
  }));

  addToCollectionItems.unshift({
    label: 'Create New Collection...',
    icon: 'pi pi-plus',
    command: () => createNewCollection()
  });

  if (addToCollectionItems.length > 1) {
    addToCollectionItems.splice(1, 0, {separator: true});
  }

  const items = [];

  items.push({
    label: `Add to Collection${batchLabel}`,
    icon: 'pi pi-plus',
    items: addToCollectionItems
  });

  if (store.activeCollection) {
    items.push({
      label: `Remove from Collection${batchLabel}`,
      icon: 'pi pi-ban',
      command: () => removeFromCollection(store.activeCollection, isBatch ? Array.from(store.selectedFiles) : [file.path])
    });
  }

  items.push({separator: true});

  if (!isBatch) {
    items.push({
      label: 'Rename',
      icon: 'pi pi-pencil',
      command: () => openRenameDialog(file.path)
    });

    items.push({
      label: 'Open in Explorer',
      icon: 'pi pi-external-link',
      command: () => openInExplorer(file.path)
    });
  }

  items.push({
    label: `Delete (Trash)${batchLabel}`,
    icon: 'pi pi-trash',
    command: () => deleteImage(isBatch ? Array.from(store.selectedFiles) : [file.path])
  });

  menuModel.value = items;

  if (cm.value) {
    cm.value.show(event);
  }
};

const addToCollection = async (collectionName, paths) => {
  try {
    await api.post(`/collections/${collectionName}/batch/add`, paths);
    toast.add({
      severity: 'success',
      summary: 'Added',
      detail: `Added ${paths.length} items to ${collectionName}`,
      life: 2000
    });
  } catch (e) {
    // Error handled by api interceptor
  }
};

const createNewCollection = () => {
  store.collectionToEdit = null;
  router.push('/collections');
};

const removeFromCollection = async (collectionName, paths) => {
  try {
    const colDetails = await api.get(`/collections/${collectionName}`);
    const isSmart = colDetails.data.isSmart;

    if (isSmart) {
      await api.post(`/collections/${collectionName}/batch/blacklist`, paths);
    } else {
      await api.post(`/collections/${collectionName}/batch/remove`, paths);
    }

    toast.add({
      severity: 'success',
      summary: 'Removed',
      detail: `Removed ${paths.length} items from ${collectionName}`,
      life: 2000
    });
    store.loadCollection(collectionName);
  } catch (e) {
    // Error handled by api interceptor
  }
};

const openInExplorer = async (path) => {
  try {
    await api.post('/system/show-in-explorer', null, {params: {path}});
  } catch (e) {
    // Error handled by api interceptor
  }
};

const deleteImage = async (paths) => {
  if (!confirm(`Are you sure you want to move ${paths.length} file(s) to the trash?`)) return;

  try {
    await api.post('/images/batch/delete', paths);

    let deletedCount = 0;
    for (const path of paths) {
      const index = store.files.findIndex(f => f.path === path);
      if (index !== -1) {
        store.files.splice(index, 1);
        deletedCount++;
      }
    }

    if (store.selectedFile && paths.includes(store.selectedFile)) {
      if (store.files.length > 0) {
        store.selectFile(store.files[0]);
      } else {
        store.selectedFile = null;
      }
    }
    store.selectedFiles.clear();

    toast.add({severity: 'success', summary: 'Deleted', detail: `Moved ${deletedCount} files to trash`, life: 2000});

  } catch (e) {
    // Error handled by api interceptor
  }
};

const openRenameDialog = (path) => {
  fileToRename.value = path;
  const parts = path.split(/[\\/]/);
  newFileName.value = parts.pop();
  showRenameDialog.value = true;
};

const performRename = async () => {
  if (!fileToRename.value || !newFileName.value) return;

  try {
    await api.post('/images/rename', null, {
      params: {
        path: fileToRename.value,
        newName: newFileName.value
      }
    });

    toast.add({severity: 'success', summary: 'Success', detail: 'File renamed successfully', life: 2000});
    showRenameDialog.value = false;

    if (store.activeCollection) {
      store.loadCollection(store.activeCollection);
    } else if (store.lastFolderPath) {
      store.loadFolder(store.lastFolderPath);
    }

  } catch (e) {
    // Error handled by api interceptor
  }
};

onMounted(async () => {
  window.addEventListener('keydown', handleKeydown);

  if (route.query.collection) {
    if (store.availableModels.length === 0) {
      await store.loadFilters();
    }
    await store.loadCollection(route.query.collection);
  } else {
    await store.initialize();
    if (store.files.length === 0) {
      store.search('');
    }
  }
});

onUnmounted(() => {
  window.removeEventListener('keydown', handleKeydown);
});

watch(() => route.query.collection, async (newCollection) => {
  if (newCollection) {
    await store.loadCollection(newCollection);
  }
});

watch(() => store.viewMode, (newMode) => {
  if (newMode === 'gallery') {
    store.setSidebarOpen(true);
  } else {
    store.setSidebarOpen(false);
  }
}, {immediate: true});

watch(() => store.imageFocusRequested, (requested) => {
  if (requested) {
    store.imageFocusRequested = false;
    store.setViewMode('browser');
    store.setSidebarOpen(true);
    if (containerRef.value) {
      containerRef.value.focus();
    }
  }
});

</script>

<template>
  <div class="flex flex-column h-full overflow-hidden">
    <BrowserToolbar class="flex-shrink-0"/>

    <div class="flex-grow-1 overflow-hidden relative flex">

      <Transition name="sidebar-slide-left">
        <div v-if="store.isTaggerOpen"
             class="h-full shadow-8 z-5 relative flex-shrink-0">
          <TaggerSidebar/>
        </div>
      </Transition>

      <div class="h-full transition-all duration-300 flex-grow-1"
           ref="containerRef" tabindex="0" style="outline: none;">
        <VirtualGallery v-if="store.viewMode === 'gallery'" ref="virtualGalleryRef" @contextmenu="onContextMenu"/>
        <SingleImageViewer v-else @contextmenu="onContextMenu"/>
      </div>

      <Transition name="sidebar-slide">
        <div v-if="store.isSidebarOpen"
             class="h-full shadow-8 z-5 relative flex-shrink-0">
          <MetadataSidebar/>
        </div>
      </Transition>
    </div>

    <CustomContextMenu ref="cm" :model="menuModel"/>

    <Dialog v-model:visible="showRenameDialog" header="Rename File" :modal="true" class="glass-dialog" style="width: 400px;">
      <div class="flex flex-column gap-3 py-2">
        <span class="text-sm text-secondary block mb-1">Enter new filename:</span>
        <InputText v-model="newFileName" class="w-full glass-input" autofocus @keyup.enter="performRename"/>
      </div>
      <template #footer>
        <div class="flex justify-content-end gap-2 pt-2">
          <LButton variant="secondary" size="sm" @click="showRenameDialog = false">
            <template #icon><X :size="14" /></template>
            Cancel
          </LButton>
          <LButton variant="primary" size="sm" @click="performRename">
            <template #icon><Check :size="14" /></template>
            Rename
          </LButton>
        </div>
      </template>
    </Dialog>
  </div>
</template>

<style scoped>
.sidebar-slide-enter-active,
.sidebar-slide-leave-active {
  transition: transform 0.3s ease;
}

.sidebar-slide-enter-from,
.sidebar-slide-leave-to {
  transform: translateX(100%);
}

.sidebar-slide-left-enter-active,
.sidebar-slide-left-leave-active {
  transition: transform 0.3s ease;
}

.sidebar-slide-left-enter-from,
.sidebar-slide-left-leave-to {
  transform: translateX(-100%);
}

.glass-dialog .p-dialog-header,
.glass-dialog .p-dialog-content,
.glass-dialog .p-dialog-footer {
  background: var(--color-surface-1, #14151B) !important;
  color: var(--color-text-primary, #F2F3F7) !important;
  border-color: var(--color-border-subtle, rgba(255, 255, 255, 0.06)) !important;
}

.glass-input {
  background: var(--color-surface-2, #23252F) !important;
  border: 1px solid var(--color-border-default, rgba(255, 255, 255, 0.10)) !important;
  color: var(--color-text-primary, #F2F3F7) !important;
}

.text-secondary {
  color: var(--color-text-secondary, #9294A3);
}

/* --- Added: Lazy Loading Image Placeholder Styles --- */
/* Uses :deep() to inject styling down into the VirtualGallery child component */
:deep(.image-card) {
  background-color: var(--color-surface-1, #14151B);
  border-radius: var(--radius-md, 8px);
  overflow: hidden;
  position: relative;
}

:deep(.lazy-image) {
  width: 100%;
  height: 100%;
  object-fit: cover;
  opacity: 0;
  transition: opacity 0.3s ease-in;
}

:deep(.lazy-image[src]) {
  opacity: 1;
}
</style>