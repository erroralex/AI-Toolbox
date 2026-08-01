<script setup>
/**
 * @file App.vue
 * @description The root layout shell of Latent Library powered by the Latent Design System.
 */
import { computed, ref, onMounted } from 'vue';
import { RouterView, useRouter, useRoute } from 'vue-router';
import Toast from 'primevue/toast';
import ConfirmDialog from 'primevue/confirmdialog';
import Titlebar from '@/components/ds/Titlebar.vue';
import SegmentedControl from '@/components/ds/SegmentedControl.vue';
import FolderNav from '@/components/FolderNav.vue';
import SystemError from '@/components/SystemError.vue';
import { useBrowserStore } from '@/stores/browser';

const router = useRouter();
const route = useRoute();
const store = useBrowserStore();

const navOptions = [
  { label: 'Gallery', value: '/' },
  { label: 'Collections', value: '/collections' },
  { label: 'Comparator', value: '/comparator' },
  { label: 'Scrubber', value: '/scrub' },
  { label: 'Speed Sorter', value: '/speedsorter' },
  { label: 'Duplicates', value: '/duplicates' }
];

const currentNavValue = computed({
  get() {
    const path = route.path;
    if (path === '/' || path.startsWith('/browser')) return '/';
    const match = navOptions.find(opt => opt.value !== '/' && path.startsWith(opt.value));
    return match ? match.value : '/';
  },
  set(val) {
    router.push(val);
  }
});

onMounted(() => {
  store.initialize();
});
</script>

<template>
  <SystemError v-if="store.backendError" />

  <div v-else class="app-layout">
    <Toast position="bottom-right" />
    <ConfirmDialog />

    <div
      v-if="store.isLoading && !store.files.length"
      class="loading-overlay"
    >
      <div class="loading-box">
        <i class="pi pi-spin pi-spinner text-4xl" style="color: var(--color-accent-primary)" />
        <span class="loading-text">Initializing Latent System...</span>
      </div>
    </div>

    <!-- Latent Design System Frameless Titlebar -->
    <Titlebar title="Latent Library">
      <template #default>
        <SegmentedControl
          v-model="currentNavValue"
          :options="navOptions"
          size="md"
        />
      </template>
    </Titlebar>

    <main class="app-body">
      <FolderNav />
      <div class="content-workspace">
        <RouterView />
      </div>
    </main>
  </div>
</template>

<style scoped>
.app-layout {
  height: 100vh;
  display: flex;
  flex-direction: column;
  overflow: hidden;
  background-color: var(--color-bg-canvas, #0A0A0D);
  background-image:
    radial-gradient(circle at 10% 10%, rgba(79, 216, 208, 0.04) 0%, transparent 40%),
    radial-gradient(circle at 90% 90%, rgba(155, 126, 245, 0.04) 0%, transparent 40%);
  color: var(--color-text-primary, #F2F3F7);
}

.app-body {
  flex-grow: 1;
  display: flex;
  overflow: hidden;
  position: relative;
}

.content-workspace {
  flex-grow: 1;
  overflow: hidden;
  position: relative;
  display: flex;
  flex-direction: column;
}

.loading-overlay {
  position: fixed;
  inset: 0;
  background: var(--color-bg-canvas, #0A0A0D);
  z-index: 9999;
  display: flex;
  align-items: center;
  justify-content: center;
}

.loading-box {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 16px;
}

.loading-text {
  font-family: var(--font-sans, Inter, sans-serif);
  font-size: 16px;
  font-weight: 600;
  color: var(--color-text-secondary, #9294A3);
}
</style>
