<script setup>
/**
 * @file FolderNav.vue
 * @description Dedicated Folder & Collection Navigation Tree Panel placed to the right of the main Sidemenu.
 */
import { ref, onMounted, watch } from 'vue';
import api from '@/services/api';
import { useBrowserStore } from '@/stores/browser';
import { useRouter, useRoute } from 'vue-router';
import Tree from 'primevue/tree';
import CustomContextMenu from './CustomContextMenu.vue';
import { useToast } from 'primevue/usetoast';
import { Plus } from 'lucide-vue-next';

const store = useBrowserStore();
const router = useRouter();
const route = useRoute();
const toast = useToast();

const nodes = ref([]);
const expandedKeys = ref({ 'collections': true, 'pinned': true, 'drives': true });
const selectedKey = ref({});
const contextMenuSelection = ref(null);

const cm = ref();
const menuModel = ref([]);

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

const loadTree = async () => {
  const rootNodes = [];

  // 1. Collections
  try {
    const colRes = await api.get('/collections');
    const colChildren = (colRes.data || []).map(c => ({
      key: `col-${c.name}`,
      label: c.name,
      data: c.name,
      icon: 'pi pi-folder',
      type: 'collection',
      leaf: true
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
    console.error('Error loading collections', e);
  }

  rootNodes.push({ key: 'sep-1', type: 'separator', selectable: false });

  // 2. Pinned
  try {
    const pinRes = await api.get('/folders/pinned');
    const pinChildren = (pinRes.data || []).map(p => ({
      key: `pinned-${p.path}`,
      label: p.name || p.path,
      data: p,
      icon: 'pi pi-bookmark',
      type: 'pinned',
      leaf: false
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
    console.error('Error loading pinned folders', e);
  }

  rootNodes.push({ key: 'sep-2', type: 'separator', selectable: false });

  // 3. This PC (Drives)
  try {
    const driveRes = await api.get('/folders/roots');
    const driveChildren = (driveRes.data || []).map(d => ({
      key: `drive-${d.path}`,
      label: d.name || d.path,
      data: d,
      icon: 'pi pi-server',
      type: 'folder',
      leaf: false
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
    console.error('Error loading drives', e);
  }

  nodes.value = rootNodes;
  expandedKeys.value = { ...expandedKeys.value, 'collections': true, 'pinned': true, 'drives': true };
  syncSelection();
};

const onNodeExpand = async (event) => {
  const actualNode = event.node || event;
  if (!actualNode.data?.path || actualNode._loaded) return;
  actualNode.loading = true;
  try {
    const res = await api.get('/folders/children', { params: { path: actualNode.data.path } });
    actualNode.children = (res.data || []).map(f => ({
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
    console.error('Failed to load children for folder:', e);
  } finally {
    actualNode.loading = false;
  }
};

const navigateToNode = (node) => {
  if (!node || node.type === 'separator' || node.type === 'root') return;

  if (node.type === 'collection') {
    const collectionName = node.data;
    store.loadCollection(collectionName);
    if (router.currentRoute.value.path !== '/') {
      router.push('/');
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

const pinNewFolder = async () => {
  let path = null;
  if (window.electronAPI) {
    path = await window.electronAPI.selectFolder();
  }
  if (path) {
    try {
      await api.post('/folders/pin', null, { params: { path } });
      toast.add({ severity: 'success', summary: 'Pinned', detail: `Folder pinned: ${path}`, life: 2000 });
      await loadTree();
    } catch (e) {
      toast.add({ severity: 'error', summary: 'Error', detail: 'Failed to pin folder', life: 3000 });
    }
  }
};

const onCustomContextMenu = (event, node) => {
  if (!node || node.type === 'separator' || node.type === 'root') return;

  contextMenuSelection.value = node;

  const items = [
    {
      label: 'Pin Folder',
      icon: 'pi pi-bookmark',
      command: async () => {
        if (node.data?.path) {
          await api.post('/folders/pin', null, { params: { path: node.data.path } });
          loadTree();
        }
      },
      visible: node.type !== 'pinned' && node.type !== 'collection' && node.data?.path
    },
    {
      label: 'Unpin Folder',
      icon: 'pi pi-bookmark-fill',
      command: async () => {
        const path = node.data?.path || node.data;
        if (path) {
          await api.post('/folders/unpin', null, { params: { path } });
          loadTree();
        }
      },
      visible: node.type === 'pinned'
    },
    {
      label: 'Remove Collection',
      icon: 'pi pi-trash',
      command: async () => {
        if (node.data) {
          await api.delete(`/collections/${encodeURIComponent(node.data)}`);
          loadTree();
        }
      },
      visible: node.type === 'collection'
    },
    { separator: true },
    {
      label: 'Open in Speed Sorter',
      icon: 'pi pi-bolt',
      command: () => {
        if (node.data?.path) {
          router.push({ path: '/speedsorter', query: { folder: node.data.path } });
        }
      },
      visible: !!node.data?.path
    }
  ];

  const visibleItems = items.filter(item => item.visible !== false);
  if (visibleItems.length > 0 && cm.value) {
    menuModel.value = visibleItems;
    cm.value.show(event);
  }
};

watch(() => store.navRefreshKey, loadTree);
watch(() => [store.lastFolderPath, store.activeCollection], syncSelection);
watch(nodes, syncSelection);

onMounted(loadTree);
</script>

<template>
  <div class="folder-tree-panel flex flex-column">
    <!-- Header -->
    <div class="tree-header p-3 flex align-items-center justify-content-between">
      <span class="tree-header-title">LIBRARY</span>
      <button class="icon-btn-ds" title="Pin New Folder" @click="pinNewFolder">
        <Plus :size="14" />
      </button>
    </div>

    <!-- Tree Content -->
    <div class="flex-grow-1 overflow-y-auto custom-scrollbar p-1">
      <Tree
        :value="nodes"
        selectionMode="single"
        v-model:selectionKeys="selectedKey"
        v-model:expandedKeys="expandedKeys"
        :lazy="true"
        @node-expand="onNodeExpand"
        @node-select="onNodeSelect"
        class="clean-tree"
      >
        <template #default="slotProps">
          <div v-if="slotProps.node.type === 'separator'" class="separator-line"></div>
          <div
            v-else
            class="tree-node-row flex align-items-center w-full"
            @contextmenu.prevent.stop="onCustomContextMenu($event, slotProps.node)"
          >
            <span class="p-treenode-label">{{ slotProps.node.label }}</span>
          </div>

        </template>
      </Tree>
    </div>

    <CustomContextMenu ref="cm" :model="menuModel" />
  </div>
</template>

<style scoped>
.folder-tree-panel {
  width: 100%;
  min-width: 0;
  background: transparent;
  border-right: none;
  box-shadow: none;
}


.tree-header {
  border-bottom: 1px solid var(--color-border-subtle, rgba(255, 255, 255, 0.06));
}

.tree-header-title {
  font-family: var(--font-sans, Inter, sans-serif);
  font-size: 11px;
  font-weight: 700;
  text-transform: uppercase;
  letter-spacing: 0.05em;
  color: var(--color-text-secondary, #9294A3);
}

.icon-btn-ds {
  width: 26px;
  height: 26px;
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

.separator-line {
  height: 1px;
  background: var(--color-border-subtle, rgba(255, 255, 255, 0.06));
  margin: 6px 0;
  width: 100%;
}

.tree-node-row {
  font-family: var(--font-sans, Inter, sans-serif);
  font-size: 12px;
  color: var(--color-text-primary, #F2F3F7);
}

:deep(.p-tree) {
  background: transparent !important;
  border: none !important;
  padding: 0 !important;
}

:deep(.p-treenode) {
  padding: 2px 0 !important;
}

:deep(.p-treenode-content) {
  padding: 6px 8px !important;
  border-radius: var(--radius-md, 8px) !important;
  transition: background var(--duration-fast, 120ms) var(--ease-standard) !important;
}

:deep(.p-treenode-content:hover) {
  background: var(--color-surface-2, #23252F) !important;
}

:deep(.p-treenode-content.p-highlight) {
  background: var(--color-accent-primary-bg, rgba(79, 216, 208, 0.12)) !important;
  color: var(--color-accent-primary, #4FD8D0) !important;
}

:deep(.p-treenode-icon) {
  margin-right: 6px !important;
  color: var(--color-text-primary, #F2F3F7) !important;
  font-size: 13px !important;
}

</style>

