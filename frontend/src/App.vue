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
import Sidebar from '@/components/Sidebar.vue';
import SystemError from '@/components/SystemError.vue';

import { useBrowserStore } from '@/stores/browser';

const router = useRouter();
const route = useRoute();
const store = useBrowserStore();

onMounted(() => {
  store.initialize();
});
</script>

<template>
  <SystemError v-if="store.backendError" />

  <div v-else class="app-layout">
    <Toast position="bottom-right" />
    <ConfirmDialog />

    <!-- Latent Design System Frameless Titlebar -->
    <Titlebar title="Latent Library" />

    <main class="app-body">
      <Sidebar />
      <div class="content-workspace">
        <RouterView />
      </div>
    </main>

  </div>
</template>


<style scoped>
.app-layout {
  height: 100vh;
  width: 100vw;
  overflow: hidden;
  display: flex;
  flex-direction: column;
  background: var(--color-bg-canvas, #0A0A0D);
}

.app-body {
  flex: 1;
  display: flex;
  overflow: hidden;
  position: relative;
}

.content-workspace {
  flex: 1;
  height: 100%;
  overflow: hidden;
  position: relative;
  display: flex;
  flex-direction: column;
}
</style>
