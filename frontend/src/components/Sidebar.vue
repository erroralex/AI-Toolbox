<script setup>
/**
 * @file Sidebar.vue
 * @description Primary left navigation menu for Latent Library aligned with the Latent Design System.
 */
import { ref } from 'vue';
import { useRouter, useRoute } from 'vue-router';
import NavItem from '@/components/ds/NavItem.vue';
import SettingsModal from '@/components/SettingsModal.vue';
import { useBrowserStore } from '@/stores/browser';
import { Image as ImageIcon, Folder as FolderIcon, ArrowLeftRight, Shield, Zap, Copy, Settings as SettingsIcon } from 'lucide-vue-next';
import alxLogoUrl from '@/assets/alx_logo.png';

const router = useRouter();
const route = useRoute();
const store = useBrowserStore();

const showSettings = ref(false);

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
</script>

<template>
  <aside class="sidebar-ds">
    <nav class="sidebar-nav-ds">
      <div class="sidebar-group">
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

      <div class="sidebar-spacer"></div>

      <div class="sidebar-group sidebar-bottom">
        <!-- Settings button is ABOVE the divider line -->
        <NavItem
          label="Settings"
          :active="showSettings"
          @click="showSettings = !showSettings"
        >
          <template #icon>
            <SettingsIcon :size="16" />
          </template>
        </NavItem>

        <div class="divider-line-ds"></div>

        <!-- Developer Credit Logo is BELOW the divider line -->
        <a
          href="https://github.com/erroralex"
          target="_blank"
          rel="noopener noreferrer"
          title="Built by Alexander Nilsson"
          class="dev-credit-link"
        >
          <img :src="alxLogoUrl" alt="Alexander Nilsson" class="dev-logo-img" />
        </a>
      </div>
    </nav>

    <!-- Settings Modal -->
    <SettingsModal
      v-model:visible="showSettings"
      v-model:isRecursive="store.recursiveView"
      v-model:autoShowLatest="store.autoShowLatest"
      @clearDb="store.clearDatabase"
      @reindex="store.reIndexAll"
      @clearModels="store.clearTagModels"
      @clearTags="store.clearAiTags"
      @clearUnorganized="store.clearUnorganized"
      @clearThumbnails="store.clearThumbnails"
    />
  </aside>
</template>

<style scoped>
.sidebar-ds {
  width: 200px;
  min-width: 200px;
  flex-shrink: 0;
  display: flex;
  flex-direction: column;
  padding: 16px 12px;
  border-right: 1px solid var(--color-border-subtle, rgba(255, 255, 255, 0.06));
  background: var(--color-surface-1, #14151B);
  box-shadow: var(--shadow-card, 0 1px 2px rgba(0,0,0,0.4));
  z-index: 20;
}

.sidebar-nav-ds {
  display: flex;
  flex-direction: column;
  height: 100%;
}

.sidebar-group {
  display: flex;
  flex-direction: column;
  gap: 4px;
}

.sidebar-spacer {
  flex: 1;
}

.sidebar-bottom {
  display: flex;
  flex-direction: column;
  gap: 8px;
}

.divider-line-ds {
  height: 1px;
  background: var(--color-border-subtle, rgba(255, 255, 255, 0.06));
  margin: 4px 0;
  width: 100%;
}

.dev-credit-link {
  display: flex;
  justify-content: center;
  align-items: center;
  padding: 6px 0;
  opacity: 0.6;
  transition: opacity var(--duration-fast, 120ms) var(--ease-standard);
}

.dev-credit-link:hover {
  opacity: 1;
}

.dev-logo-img {
  max-width: 120px;
  height: auto;
  max-height: 44px;
  object-fit: contain;
}
</style>
