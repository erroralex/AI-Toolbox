<script setup>
/**
 * @file FolderNav.vue
 * @description Dedicated Folder & Collection Navigation Tree Panel placed to the right of the main Sidemenu.
 */
import { ref, onMounted } from 'vue';
import api from '@/services/api';
import { useBrowserStore } from '@/stores/browser';
import { useRouter, useRoute } from 'vue-router';
import Tree from 'primevue/tree';
import CustomContextMenu from './CustomContextMenu.vue';
import { useToast } from 'primevue/usetoast';
import Button from 'primevue/button';
import { Plus } from 'lucide-vue-next';

const store = useBrowserStore();
const router = useRouter();
const route = useRoute();
const toast = useToast();

const nodes = ref([]);
const selectedKey = ref({});
const expandedKeys = ref({});
const cm = ref();
const menuModel = ref([]);
const contextNode = ref(null);

const loadTree = async () => {
  try {
    const res = await api.get('/tree/roots');
    nodes.value = res.data;
    if (store.lastFolderPath) {
      expandPathToNode(store.lastFolderPath);
    }
  } catch (e) {
    console.error('Failed to load tree roots:', e);
  }
};

const onNodeExpand = async (node) => {
  if (node.children && node.children.length > 0 && node.children[0].label !== 'Loading...') return;
  try {
    node.children = [{ key: node.key + '_loading', label: 'Loading...', leaf: true }];
    const res = await api.get('/tree/children', { params: { path: node.data } });
    node.children = res.data;
  } catch (e) {
    node.children = [];
  }
};

const onNodeSelect = (node) => {
  if (node.type === 'separator') return;
  if (node.type === 'collection') {
    store.setCollection(node.data);
  } else if (node.data) {
    store.loadPath(node.data);
  }
};

const onNodeClick = (node) => {
  if (node.type === 'separator') return;
  if (node.children && node.children.length > 0) {
    expandedKeys.value[node.key] = !expandedKeys.value[node.key];
  }
};

const pinNewFolder = async () => {
  let path = null;
  if (window.electronAPI) {
    path = await window.electronAPI.selectFolder();
  }
  if (path) {
    try {
      await api.post('/system/pin-folder', null, { params: { path } });
      toast.add({ severity: 'success', summary: 'Pinned', detail: `Folder pinned: ${path}`, life: 2000 });
      loadTree();
    } catch (e) {
      toast.add({ severity: 'error', summary: 'Error', detail: 'Failed to pin folder', life: 3000 });
    }
  }
};

const onCustomContextMenu = (event, node) => {
  contextNode.value = node;
  const items = [];

  if (node.type === 'pinned') {
    items.push({
      label: 'Unpin Folder',
      icon: 'pi pi-times',
      command: async () => {
        try {
          await api.post('/system/unpin-folder', null, { params: { path: node.data } });
          loadTree();
        } catch (e) {}
      }
    });
  }

  if (node.type === 'collection') {
    items.push({
      label: 'Delete Collection',
      icon: 'pi pi-trash',
      command: async () => {
        try {
          await api.delete(`/collections/${encodeURIComponent(node.data)}`);
          loadTree();
        } catch (e) {}
      }
    });
  }

  if (items.length > 0) {
    menuModel.value = items;
    cm.value.show(event);
  }
};

const expandPathToNode = async (targetPath) => {
  // Path expansion logic
};

onMounted(loadTree);
</script>

<template>
  <aside class="folder-tree-panel h-full flex flex-column">
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
        @node-click="onNodeClick"
        class="clean-tree"
      >
        <template #default="slotProps">
          <div v-if="slotProps.node.type === 'separator'" class="separator-line"></div>
          <div
            v-else
            class="tree-node-row flex align-items-center w-full"
            @contextmenu.prevent.stop="onCustomContextMenu($event, slotProps.node)"
          >
            <span v-if="slotProps.node.icon" :class="['p-treenode-icon mr-2', slotProps.node.icon]"></span>
            <span class="p-treenode-label">{{ slotProps.node.label }}</span>
          </div>
        </template>
      </Tree>
    </div>

    <CustomContextMenu ref="cm" :model="menuModel" />
  </aside>
</template>

<style scoped>
.folder-tree-panel {
  width: 240px;
  min-width: 240px;
  background: var(--color-surface-1, #14151B);
  border-right: 1px solid var(--color-border-subtle, rgba(255, 255, 255, 0.06));
  box-shadow: var(--shadow-card, 0 1px 2px rgba(0,0,0,0.4));
  z-index: 15;
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
</style>
