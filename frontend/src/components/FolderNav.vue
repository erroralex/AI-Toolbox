<script setup>
/**
 * @file FolderNav.vue
 * @description A sophisticated navigation sidebar component that provides a unified interface for exploring the local file system and user-defined collections.
 *
 * This component implements a lazy-loading tree structure that allows users to traverse their
 * entire local file system, access pinned folders, and manage image collections. It integrates
 * deeply with the backend to provide real-time folder expansion and system-level operations
 * via a custom context menu.
 *
 * Key functionalities:
 * - **Unified Navigation:** Combines collections, pinned folders, and physical drives into a single tree view.
 * - **Lazy Loading:** Fetches child directories on-demand to ensure high performance even with deep file structures.
 * - **Contextual Management:** Provides a rich right-click menu for pinning folders, editing collections,
 *   and opening directories in external tools (Explorer, Speed Sorter).
 * - **Settings Integration:** Hosts the application's global settings dialog, including theme selection,
 *   database maintenance, and path exclusion rules.
 * - **State Synchronization:** Automatically synchronizes the tree selection with the global browser store
 *   to reflect the currently active folder or collection.
 * - **Manual Pinning:** Allows users to manually pin folders (including WSL and UNC paths)
 *   via a native OS picker, bypassing Java's root drive limitations.
 */
import {ref, onMounted, watch, computed} from 'vue';
import api from '@/services/api';
import {useBrowserStore} from '@/stores/browser';
import {useRouter, useRoute} from 'vue-router';
import Tree from 'primevue/tree';
import CustomContextMenu from './CustomContextMenu.vue';
import {useToast} from 'primevue/usetoast';
import Button from 'primevue/button';
import Dialog from 'primevue/dialog';
import {useConfirm} from 'primevue/useconfirm';
import InputText from 'primevue/inputtext';
import Dropdown from 'primevue/dropdown';
import NavItem from '@/components/ds/NavItem.vue';
import SettingsModal from '@/components/SettingsModal.vue';
import { Image as ImageIcon, Folder as FolderIcon, ArrowLeftRight, Shield, Zap, Copy, Settings as SettingsIcon } from 'lucide-vue-next';


import logoDs from '@/assets/alx_logo.png';
import logoNeon from '@/assets/alx_logo_neon.png';
import logoGold from '@/assets/alx_logo_gold.png';
import logoLight from '@/assets/alx_logo_light.png';
import logoFan from '@/assets/alx_logo_fan.png';
import logoFanLight from '@/assets/alx_logo_fan_light.png';

const store = useBrowserStore();
const router = useRouter();
const route = useRoute();

const mainNavItems = [
  { id: 'gallery', label: 'Gallery', icon: ImageIcon, path: '/' },
  { id: 'collections', label: 'Collections', icon: FolderIcon, path: '/collections' },
  { id: 'comparator', label: 'Comparator', icon: ArrowLeftRight, path: '/comparator' },
  { id: 'scrub', label: 'Scrubber', icon: Shield, path: '/scrub' },
  { id: 'sorter', label: 'Speed Sorter', icon: Zap, path: '/speedsorter' },
  { id: 'dupes', label: 'Duplicates', icon: Copy, path: '/duplicates' }
];

const isNavActive = (path) => {
  if (path === '/') {
    return route.path === '/' || route.path.startsWith('/browser');
  }
  return route.path.startsWith(path);
};

const navigateToPath = (path) => {
  router.push(path);
};

const toast = useToast();
const confirm = useConfirm();

const nodes = ref([]);
const expandedKeys = ref({});
const selectedKey = ref({});
const contextMenuSelection = ref(null);

const cm = ref();
const menuModel = ref([]);

const showSettings = ref(false);
const excludedPaths = ref([]);
const newExcludedPath = ref('');
const appVersion = ref('unknown');

const customPromptNodes = ref([]);
const newPromptNode = ref('');
const customLoraNodes = ref([]);
const newLoraNode = ref('');

const currentTheme = ref('neon');
const themeOptions = ref([
  {label: 'Deep Neon', value: 'neon'},
  {label: 'Clean Light', value: 'light'},
  {label: 'Dark Premium', value: 'gold'},
  {label: 'Fan Friction', value: 'fanfriction'},
  {label: 'Fan Friction Light', value: 'fanfriction-light'}
]);

const currentLogo = computed(() => {
  if (logoDs) return logoDs;
  switch (store.currentTheme) {
    case 'gold':
      return logoGold;
    case 'light':
      return logoLight;
    case 'fanfriction':
      return logoFan;
    case 'fanfriction-light':
      return logoFanLight;
    default:
      return logoNeon;
  }
});


watch(() => store.navRefreshKey, () => {
  loadTree();
});

const loadTree = async () => {
  const rootNodes = [];

  try {
    const colRes = await api.get('/collections');
    const colChildren = colRes.data.map(c => ({
      key: `col-${c.name}`, label: c.name, data: c.name, icon: 'pi pi-folder', type: 'collection', leaf: true
    }));
    rootNodes.push({
      key: 'collections',
      label: 'Collections',
      icon: 'pi pi-list',
      children: colChildren,
      type: 'root',
      leaf: false
    });
  } catch (e) {
    console.error("Error loading collections", e);
  }

  rootNodes.push({key: 'sep-1', type: 'separator', selectable: false});

  try {
    const pinRes = await api.get('/folders/pinned');
    const pinChildren = pinRes.data.map(p => ({
      key: `pinned-${p.path}`, label: p.name || p.path, data: p, icon: 'pi pi-bookmark', type: 'pinned', leaf: false
    }));
    rootNodes.push({
      key: 'pinned',
      label: 'Pinned',
      icon: 'pi pi-bookmark',
      children: pinChildren,
      type: 'root',
      leaf: false
    });
  } catch (e) {
    console.error("Error loading pinned", e);
  }

  rootNodes.push({key: 'sep-2', type: 'separator', selectable: false});

  try {
    const driveRes = await api.get('/folders/roots');
    const driveChildren = driveRes.data.map(d => ({
      key: `drive-${d.path}`, label: d.name || d.path, data: d, icon: 'pi pi-server', type: 'folder', leaf: false
    }));
    rootNodes.push({
      key: 'drives',
      label: 'This PC',
      icon: 'pi pi-desktop',
      children: driveChildren,
      type: 'root',
      leaf: false
    });
  } catch (e) {
    console.error("Error loading drives", e);
  }

  nodes.value = rootNodes;
  expandedKeys.value = {'collections': true, 'pinned': true, 'drives': true};

  syncSelection();
};

const normalizePath = (p) => {
  if (!p) return '';
  return p.replace(/\\/g, '/').toLowerCase().replace(/\/$/, '');
};

const findNodeByPath = (nodesList, path) => {
  if (!nodesList || !path) return null;
  const searchPath = normalizePath(path);
  for (const node of nodesList) {
    if (node.data?.path && normalizePath(node.data.path) === searchPath) return node;
    if (node.children && node.children.length > 0) {
      const found = findNodeByPath(node.children, path);
      if (found) return found;
    }
  }
  return null;
};

const syncSelection = () => {
  let newSelection = {};
  if (store.activeCollection) {
    const root = nodes.value.find(n => n.key === 'collections');
    const node = root?.children?.find(c => c.data === store.activeCollection);
    if (node) newSelection[node.key] = true;
  } else if (store.lastFolderPath) {
    const node = findNodeByPath(nodes.value, store.lastFolderPath);
    if (node) newSelection[node.key] = true;
  }

  if (JSON.stringify(newSelection) !== JSON.stringify(selectedKey.value)) {
    selectedKey.value = newSelection;
  }
};

watch(() => [store.lastFolderPath, store.activeCollection], syncSelection);
watch(nodes, syncSelection);

const navigateToNode = (node) => {
  if (!node || node.type === 'separator' || node.type === 'root') return;

  if (node.type === 'collection') {
    const collectionName = node.data;
    if (router.currentRoute.value.path === '/' && route.query.collection === collectionName) {
      store.loadCollection(collectionName);
    } else {
      router.push({path: '/', query: {collection: collectionName}});
    }
  } else if (node.data?.path) {
    const path = node.data.path;
    store.loadFolder(path);
    if (router.currentRoute.value.path !== '/') {
      router.push('/');
    }
  }
};

const onNodeSelect = (event) => {
  navigateToNode(event.node || event);
};

const onNodeClick = (event) => {
  navigateToNode(event.node);
};

const onNodeExpand = async (node) => {
  const actualNode = node.node || node;
  if (!actualNode.data?.path || actualNode._loaded) return;
  actualNode.loading = true;
  try {
    const res = await api.get('/folders/children', {params: {path: actualNode.data.path}});
    actualNode.children = res.data.map(f => ({
      key: `${actualNode.key}-${f.name}`,
      label: f.name || f.path,
      data: f,
      icon: f.isDirectory ? 'pi pi-folder' : 'pi pi-file',
      type: 'folder',
      leaf: !f.isDirectory,
      children: []
    }));
    actualNode._loaded = true;
    syncSelection();
  } catch (e) {
  } finally {
    actualNode.loading = false;
  }
};

const onCustomContextMenu = (event, node) => {
  if (!node || node.type === 'separator' || node.type === 'root') return;

  contextMenuSelection.value = node;

  menuModel.value = [
    {
      label: 'Pin Folder',
      icon: 'pi pi-bookmark',
      command: () => pinFolder(node.data.path),
      visible: node.type !== 'pinned' && node.type !== 'collection' && node.data?.path
    },
    {
      label: 'Unpin Folder',
      icon: 'pi pi-bookmark-fill',
      command: () => unpinFolder(node.data.path),
      visible: node.type === 'pinned'
    },
    {
      label: 'Edit Collection',
      icon: 'pi pi-pencil',
      command: () => editCollection(node.data),
      visible: node.type === 'collection'
    },
    {
      label: 'Remove Collection',
      icon: 'pi pi-trash',
      command: () => removeCollection(node.data),
      visible: node.type === 'collection'
    },
    {separator: true},
    {
      label: 'Show in Explorer',
      icon: 'pi pi-external-link',
      command: () => openInExplorer(node.data.path),
      visible: !!node.data?.path
    },
    {
      label: 'Open in Speed Sorter',
      icon: 'pi pi-bolt',
      command: () => openInSpeedSorter(node.data.path),
      visible: !!node.data?.path
    }
  ];

  if (cm.value) {
    cm.value.show(event);
  }
};

const pinFolder = async (path) => {
  await api.post('/folders/pin', null, {params: {path}});
  await loadTree();
};

const pinNewFolder = async () => {
  let path = '';
  if (window.electronAPI) {
    path = await window.electronAPI.selectFolder();
  } else {
    path = prompt("Enter absolute path to pin:");
  }

  if (path) {
    try {
      await api.post('/folders/pin', null, {params: {path}});
      toast.add({severity: 'success', summary: 'Success', detail: 'Folder pinned', life: 2000});
      await loadTree();
    } catch (e) {
      console.error("Failed to pin folder", e);
      toast.add({severity: 'error', summary: 'Error', detail: 'Failed to pin folder', life: 3000});
    }
  }
};

const unpinFolder = async (path) => {
  await api.post('/folders/unpin', null, {params: {path}});
  await loadTree();
};
const removeCollection = async (name) => {
  try {
    await api.delete(`/collections/${name}`);
    toast.add({severity: 'success', summary: 'Success', detail: 'Collection removed', life: 2000});
    store.refreshNav();
  } catch (e) {
  }
};
const editCollection = (name) => {
  store.collectionToEdit = name;
  router.push('/collections');
};
const openInSpeedSorter = async (path) => {
  await api.post('/speedsorter/config/input', null, {params: {path}});
  router.push('/speedsorter');
};
const openInExplorer = async (path) => {
  await api.post('/system/open-folder', null, {params: {path}});
};

const openSettings = async () => {
  showSettings.value = true;
  currentTheme.value = store.currentTheme;
  try {
    const res = await api.get('/system/excluded-paths');
    excludedPaths.value = res.data;

    const verRes = await api.get('/system/version');
    if (verRes.data && verRes.data.version) {
        appVersion.value = verRes.data.version;
    }

    const promptRes = await api.get('/system/custom-nodes/prompt');
    customPromptNodes.value = promptRes.data;

    const loraRes = await api.get('/system/custom-nodes/lora');
    customLoraNodes.value = loraRes.data;

  } catch (e) {
    console.error("Failed to load settings data", e);
  }
};

const changeTheme = () => {
  store.setTheme(currentTheme.value);
};

const openDataFolder = async () => {
  try {
    await api.post('/system/open-data-folder');
  } catch (e) {
  }
};

const clearDatabase = () => {
  confirm.require({
    message: 'Are you sure you want to clear the database? All metadata and collections will be lost.',
    header: 'Clear Entire Database',
    icon: 'pi pi-exclamation-triangle',
    acceptClass: 'p-button-danger',
    accept: async () => {
      try {
        await api.post('/system/clear-database');
        toast.add({severity: 'success', summary: 'Success', detail: 'Database cleared', life: 3000});
        store.initialize();
      } catch (e) {
      }
    }
  });
};

const clearThumbnails = () => {
  confirm.require({
    message: 'Are you sure you want to delete all thumbnails? They will be regenerated on demand.',
    header: 'Clear Thumbnails',
    icon: 'pi pi-exclamation-triangle',
    acceptClass: 'p-button-danger',
    accept: async () => {
      try {
        await api.post('/system/clear-thumbnails');
        toast.add({severity: 'success', summary: 'Success', detail: 'Thumbnails cleared', life: 3000});
      } catch (e) {
      }
    }
  });
};

const clearTagModels = () => {
  confirm.require({
    message: 'Are you sure you want to delete the AI tagging models? You will need to download them again to use the auto-tagger.',
    header: 'Clear Tag Models',
    icon: 'pi pi-exclamation-triangle',
    acceptClass: 'p-button-danger',
    accept: async () => {
      try {
        await api.post('/tagger/clear-models');
        toast.add({severity: 'success', summary: 'Success', detail: 'Tag models cleared', life: 3000});
      } catch (e) {
      }
    }
  });
};

const clearAiTags = () => {
  confirm.require({
    message: 'Are you sure you want to clear all AI-generated tags? Your manual collections and ratings will be kept.',
    header: 'Clear AI Tags',
    icon: 'pi pi-exclamation-triangle',
    acceptClass: 'p-button-danger',
    accept: async () => {
      try {
        await api.post('/system/clear-ai-tags');
        toast.add({severity: 'success', summary: 'Success', detail: 'AI tags cleared', life: 3000});
        if (store.selectedFile) store.fetchMetadata(store.selectedFile);
      } catch (e) {
      }
    }
  });
};

const clearUnorganized = () => {
  confirm.require({
    message: 'Are you sure you want to clear all unorganized images from the index? Images in collections or with ratings will be kept.',
    header: 'Clear Unorganized',
    icon: 'pi pi-exclamation-triangle',
    acceptClass: 'p-button-danger',
    accept: async () => {
      try {
        await api.post('/system/clear-unorganized');
        toast.add({severity: 'success', summary: 'Success', detail: 'Unorganized images cleared', life: 3000});
        store.initialize();
      } catch (e) {
      }
    }
  });
};

const reIndexAll = () => {
  confirm.require({
    message: 'Are you sure you want to re-index the entire library? This will clear the database and re-scan your last folder.',
    header: 'Re-index All',
    icon: 'pi pi-refresh',
    acceptClass: 'p-button-warning',
    accept: async () => {
      try {
        await api.post('/system/re-index-all');
        toast.add({severity: 'info', summary: 'Started', detail: 'Full re-index initiated in background', life: 3000});
      } catch (e) {
      }
    }
  });
};

const addExcludedPath = async () => {
  if (!newExcludedPath.value) return;
  try {
    await api.post('/system/excluded-paths', null, {params: {path: newExcludedPath.value}});
    excludedPaths.value.push(newExcludedPath.value);
    newExcludedPath.value = '';
  } catch (e) {
  }
};

const selectExcludedFolder = async () => {
  if (window.electronAPI) {
    const path = await window.electronAPI.selectFolder();
    if (path) {
      newExcludedPath.value = path;
    }
  } else {
    const path = prompt("Enter absolute path to exclude:");
    if (path) {
      newExcludedPath.value = path;
    }
  }
};

const removeExcludedPath = async (path) => {
  try {
    await api.delete('/system/excluded-paths', {params: {path}});
    excludedPaths.value = excludedPaths.value.filter(p => p !== path);
  } catch (e) {
  }
};

const addCustomPromptNode = async () => {
  if (!newPromptNode.value) return;
  try {
    await api.post('/system/custom-nodes/prompt', null, {params: {node: newPromptNode.value}});
    customPromptNodes.value.push(newPromptNode.value);
    newPromptNode.value = '';
  } catch (e) {
  }
};

const removeCustomPromptNode = async (node) => {
  try {
    await api.delete('/system/custom-nodes/prompt', {params: {node}});
    customPromptNodes.value = customPromptNodes.value.filter(n => n !== node);
  } catch (e) {
  }
};

const addCustomLoraNode = async () => {
  if (!newLoraNode.value) return;
  try {
    await api.post('/system/custom-nodes/lora', null, {params: {node: newLoraNode.value}});
    customLoraNodes.value.push(newLoraNode.value);
    newLoraNode.value = '';
  } catch (e) {
  }
};

const removeCustomLoraNode = async (node) => {
  try {
    await api.delete('/system/custom-nodes/lora', {params: {node}});
    customLoraNodes.value = customLoraNodes.value.filter(n => n !== node);
  } catch (e) {
  }
};

const openLink = (url) => {
  if (window.electronAPI) {
    window.electronAPI.openExternal(url);
  } else {
    window.open(url, '_blank');
  }
};

onMounted(loadTree);
</script>

<template>
  <aside class="folder-nav-glass h-full flex flex-column">

    <!-- Primary Navigation Routes -->
    <div class="primary-nav-section flex flex-column gap-1 p-2">
      <NavItem
        v-for="item in mainNavItems"
        :key="item.id"
        :label="item.label"
        :active="isNavActive(item.path)"
        @click="navigateToPath(item.path)"
      >
        <template #icon>
          <component :is="item.icon" :size="16" />
        </template>
      </NavItem>
    </div>

    <div class="separator-line px-2"></div>

    <!-- Folder & Collection Navigation Tree -->
    <div class="p-2 px-3 font-bold text-xs uppercase text-gray-400 flex align-items-center justify-content-between">
      <span>Library</span>
      <div class="flex align-items-center gap-1">
        <Button icon="pi pi-plus" class="p-button-text p-button-rounded p-button-sm text-white"
                v-tooltip.bottom="'Pin New Folder'" @click="pinNewFolder"/>
      </div>
    </div>

    <div class="flex-grow-1 overflow-y-auto custom-scrollbar">
      <Tree
          :value="nodes"
          selectionMode="single"
          v-model:selectionKeys="selectedKey"
          v-model:expandedKeys="expandedKeys"
          :lazy="true"
          @node-expand="onNodeExpand"
          @node-select="onNodeSelect"
          @node-click="onNodeClick"
      >
        <template #default="slotProps">
          <div v-if="slotProps.node.type === 'separator'" class="separator-line"></div>
          <div v-else class="w-full h-full flex align-items-center"
               @contextmenu.prevent.stop="onCustomContextMenu($event, slotProps.node)">
            <span v-if="slotProps.node.icon" :class="['p-treenode-icon mr-2', slotProps.node.icon]"></span>
            <span class="p-treenode-label">{{ slotProps.node.label }}</span>
          </div>
        </template>
      </Tree>
    </div>

    <!-- Bottom Sidebar Group (Settings + Dev Logo) -->
    <div class="sidebar-bottom-group border-top-1 border-white-alpha-10 p-2 flex flex-column gap-2 mt-auto">
      <NavItem
        label="Settings"
        :active="showSettings"
        @click="showSettings = !showSettings"
      >
        <template #icon>
          <SettingsIcon :size="16" />
        </template>
      </NavItem>

      <div class="flex justify-content-center pt-1">
        <a href="https://github.com/erroralex" target="_blank" rel="noopener noreferrer" title="Built by Alexander Nilsson">
          <img :src="currentLogo" alt="ALX Logo" class="nav-logo">
        </a>
      </div>
    </div>

    <CustomContextMenu ref="cm" :model="menuModel"/>

    <!-- Latent Design System Settings Modal -->
    <SettingsModal
      v-model:visible="showSettings"
      v-model:isRecursive="store.recursiveView"
      v-model:autoShowLatest="store.autoShowLatest"
      @clearDb="clearDatabase"
      @reindex="reIndexAll"
      @clearModels="clearTagModels"
      @clearTags="clearAiTags"
      @clearUnorganized="clearUnorganized"
      @clearThumbnails="clearThumbnails"
      @openDataFolder="openDataFolder"
    />
  </aside>
</template>



<style scoped>
.folder-nav-glass {
  background: var(--color-surface-1, #14151B);
  border-right: 1px solid var(--color-border-subtle, rgba(255, 255, 255, 0.06));
  box-shadow: var(--shadow-card, 0 1px 2px rgba(0,0,0,0.4));
  width: 224px !important;
  min-width: 220px !important;
}

.text-gradient {
  background: var(--gradient-brand-text, linear-gradient(90deg, #4FD8D0, #9B7EF5));
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  background-clip: text;
}

.separator-line {
  height: 1px;
  background: var(--color-border-subtle, rgba(255, 255, 255, 0.06));
  margin: 0.5rem 0;
  width: 100%;
}

.glass-dialog .p-dialog-header,
.glass-dialog .p-dialog-content,
.glass-dialog .p-dialog-footer {
  background: var(--color-surface-1, #14151B) !important;
  color: var(--color-text-primary, #F2F3F7) !important;
  border-color: var(--color-border-default, rgba(255, 255, 255, 0.10)) !important;
}

.glass-input {
  background: var(--color-surface-2, #23252F) !important;
  border: 1px solid var(--color-border-default, rgba(255, 255, 255, 0.10)) !important;
  color: var(--color-text-primary, #F2F3F7) !important;
}

.glass-box {
  background: var(--color-surface-2, #23252F);
  border: 1px solid var(--color-border-default, rgba(255, 255, 255, 0.10));
}

.nav-logo {
  max-width: 140px;
  height: auto;
  max-height: 48px;
  object-fit: contain;
  opacity: 0.75;
  transition: opacity var(--duration-fast, 120ms) var(--ease-standard);
  cursor: pointer;
}

.nav-logo:hover {
  opacity: 1;
}


:deep(.p-tree) {
  background: transparent !important;
  border: none !important;
  padding: 0.5rem 0.5rem 0.5rem 1.5rem;
}

:deep(.p-tree .p-tree-container .p-treenode .p-treenode-content) {
  padding: 0.5rem 0.75rem;
  margin: 4px 4px;
  border-radius: 6px;
  color: var(--text-secondary);
  transition: all 0.2s ease;
  border: none;
  font-weight: 600;
  position: relative;
  z-index: 1;
  background: transparent !important;
  overflow: visible !important;
  width: fit-content;
}

/* Hide the default PrimeVue icon to avoid duplication */
:deep(.p-tree .p-tree-container .p-treenode .p-treenode-content > .p-treenode-icon) {
  display: none !important;
}

:deep(.p-tree .p-tree-container .p-treenode .p-treenode-content) {
  min-width: auto;
}

:deep(.p-tree .p-tree-container .p-treenode .p-treenode-content .p-treenode-label) {
  font-size: 0.9rem;
}

:deep(.p-tree .p-tree-container .p-treenode .p-treenode-content:focus),
:deep(.p-tree .p-tree-container .p-treenode:focus),
:deep(.p-tree:focus),
:deep(.p-tree .p-tree-container .p-treenode .p-treenode-content:focus-visible),
:deep(.p-tree .p-tree-container .p-treenode:focus-visible),
:deep(.p-tree .p-tree-container .p-treenode .p-treenode-content.p-highlight:focus),
:deep(.p-tree *) {
  box-shadow: none !important;
  outline: none !important;
  border: none !important;
}

:deep(.p-tree .p-tree-container .p-treenode .p-treenode-content.p-highlight::before),
:deep(.p-tree .p-tree-container .p-treenode .p-treenode-content:hover::before) {
  content: '';
  position: absolute;
  inset: -1px;
  background: var(--grad-hover);
  border-radius: 6px;
  z-index: -2;
  opacity: 0;
  filter: blur(4px);
  transition: opacity 0.3s ease;
}

:deep(.p-tree .p-tree-container .p-treenode .p-treenode-content.p-highlight::after),
:deep(.p-tree .p-tree-container .p-treenode .p-treenode-content:hover::after) {
  content: '';
  position: absolute;
  inset: 0;
  background: var(--bg-btn-inner);
  border-radius: 6px;
  z-index: -1;
  transition: background 0.3s ease;
}

:deep(.p-tree .p-tree-container .p-treenode .p-treenode-content:hover) {
  color: var(--text-primary) !important;
  transform: translateY(-1px);
}

:deep(.p-treenode-content:hover::before) {
  opacity: 0.8;
}

:deep(.p-tree .p-tree-container .p-treenode .p-treenode-content:hover::after) {
  background: var(--bg-btn-inner);
}

:deep(.p-tree .p-tree-container .p-treenode .p-treenode-content:hover .p-treenode-label),
:deep(.p-tree .p-tree-container .p-treenode .p-treenode-content:hover .p-treenode-icon) {
  background: none !important;
  -webkit-text-fill-color: var(--text-primary) !important;
  color: var(--text-primary) !important;
}

:deep(.p-tree .p-tree-container .p-treenode .p-treenode-content.p-highlight) {
  color: transparent !important;
}

:deep(.p-tree .p-tree-container .p-treenode .p-treenode-content.p-highlight .p-treenode-label),
:deep(.p-tree .p-tree-container .p-treenode .p-treenode-content.p-highlight .p-treenode-icon) {
  background-image: var(--grad-text);
  background-clip: text;
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  color: transparent !important;
  display: inline-block;
  position: relative;
  z-index: 2;
}

:deep(.p-tree .p-tree-container .p-treenode .p-treenode-content.p-highlight::before) {
  opacity: 0.8;
}

:deep(.p-tree .p-tree-container .p-treenode .p-treenode-content.p-highlight::after) {
  background: var(--bg-btn-inner);
}

:deep(.p-tree .p-tree-container .p-treenode .p-treenode-content.p-highlight:hover) {
  background: transparent !important;
}

:deep(.p-treenode-icon) {
  color: inherit !important;
  transition: color 0.2s;
}

:deep(.p-tree .p-tree-toggler) {
  color: var(--text-secondary);
  margin-right: 0.25rem;
}

:deep(.p-tree .p-tree-toggler:hover) {
  color: var(--text-primary);
  background: transparent;
}
</style>
