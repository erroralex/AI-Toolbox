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

    <!-- Latent Design System Splash Screen -->
    <div
      v-if="store.isLoading && !store.files.length"
      class="loading-overlay-ds"
    >
      <div class="splash-content-ds">
        <div class="splash-logo-container-ds">
          <img src="@/assets/latent-mark.svg" alt="Latent Logo" class="splash-logo-ds" />
          <div class="splash-glow-ds"></div>
        </div>
        <div class="splash-title-ds">Latent Library</div>
        <div class="splash-subtitle-ds">High-Performance AI Asset Manager</div>
        <div class="splash-loader-ds">
          <div class="splash-bar-fill-ds"></div>
        </div>
        <div class="splash-version-ds">v1.1.1</div>
      </div>
    </div>

    <!-- Latent Design System Frameless Titlebar -->
    <Titlebar title="Latent Library" />

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

/* Splash Screen DS */
.loading-overlay-ds {
  position: fixed;
  inset: 0;
  background: var(--color-bg-canvas, #0A0A0D);
  background-image:
    radial-gradient(circle at 50% 40%, rgba(79, 216, 208, 0.08) 0%, transparent 60%),
    radial-gradient(circle at 50% 60%, rgba(155, 126, 245, 0.06) 0%, transparent 60%);
  z-index: 9999;
  display: flex;
  align-items: center;
  justify-content: center;
}

.splash-content-ds {
  display: flex;
  flex-direction: column;
  align-items: center;
  text-align: center;
  gap: 12px;
}

.splash-logo-container-ds {
  position: relative;
  width: 64px;
  height: 64px;
  display: flex;
  align-items: center;
  justify-content: center;
  margin-bottom: 8px;
}

.splash-logo-ds {
  width: 56px;
  height: 56px;
  position: relative;
  z-index: 2;
}

.splash-glow-ds {
  position: absolute;
  inset: -10px;
  background: var(--gradient-brand, linear-gradient(135deg, #4FD8D0 0%, #9B7EF5 100%));
  border-radius: 50%;
  filter: blur(20px);
  opacity: 0.35;
  animation: pulseGlow 2.4s ease-in-out infinite alternate;
}

.splash-title-ds {
  font-family: var(--font-sans, Inter, sans-serif);
  font-size: 26px;
  font-weight: 800;
  letter-spacing: -0.02em;
  background: var(--gradient-brand-text, linear-gradient(90deg, #4FD8D0, #9B7EF5));
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  background-clip: text;
}

.splash-subtitle-ds {
  font-family: var(--font-sans, Inter, sans-serif);
  font-size: 13px;
  font-weight: 500;
  color: var(--color-text-secondary, #9294A3);
}

.splash-loader-ds {
  width: 180px;
  height: 4px;
  border-radius: var(--radius-full, 999px);
  background: var(--color-surface-2, #23252F);
  overflow: hidden;
  margin-top: 16px;
  position: relative;
}

.splash-bar-fill-ds {
  position: absolute;
  top: 0;
  left: 0;
  height: 100%;
  width: 40%;
  background: var(--gradient-brand, linear-gradient(135deg, #4FD8D0 0%, #9B7EF5 100%));
  border-radius: var(--radius-full, 999px);
  animation: splashProgress 1.6s ease-in-out infinite alternate;
}

.splash-version-ds {
  font-family: var(--font-mono, "JetBrains Mono", monospace);
  font-size: 11px;
  color: var(--color-text-tertiary, #6F7180);
  margin-top: 4px;
}

@keyframes pulseGlow {
  0% { opacity: 0.25; transform: scale(0.95); }
  100% { opacity: 0.55; transform: scale(1.1); }
}

@keyframes splashProgress {
  0% { left: 0%; width: 25%; }
  100% { left: 75%; width: 25%; }
}
</style>

