<script setup>
/**
 * @file SystemError.vue
 * @description A fallback screen shown when the backend is unreachable or fails to initialize aligned with the Latent Design System.
 */
import LButton from '@/components/ds/LButton.vue';
import { RefreshCw } from 'lucide-vue-next';

const reload = () => {
  window.location.reload();
};
</script>

<template>
  <div
      class="system-error-container flex flex-column align-items-center justify-content-center h-screen p-5 text-center">
    <div class="error-card p-6 border-round-xl max-w-30rem">
      <i class="pi pi-exclamation-triangle text-6xl text-red-500 mb-4"></i>
      <h1 class="text-3xl font-bold mb-3 text-white">Backend Unreachable</h1>
      <p class="text-gray-400 mb-5 line-height-3">
        The application could not connect to the background service. This usually happens if:
      </p>
      <ul class="text-left text-gray-300 mb-5 px-4">
        <li class="mb-2">Java 21 or higher is not installed.</li>
        <li class="mb-2">A firewall is blocking local port communication.</li>
        <li class="mb-2">The backend process failed to start.</li>
      </ul>
      <div class="flex gap-3 justify-content-center">
        <LButton variant="primary" size="md" @click="reload">
          <template #icon><RefreshCw :size="16" /></template>
          Retry Connection
        </LButton>
      </div>
    </div>
  </div>
</template>

<style scoped>
.system-error-container {
  background: var(--color-bg-canvas, #0A0A0D);
}

.error-card {
  background: var(--color-surface-1, #14151B);
  border: 1px solid var(--color-border-strong, rgba(255, 255, 255, 0.18));
  box-shadow: var(--shadow-panel, 0 20px 60px -20px rgba(0,0,0,0.65));
}

.text-red-500 {
  color: var(--color-danger, #F2665B) !important;
}

.text-gray-400,
.text-gray-300 {
  color: var(--color-text-secondary, #9294A3);
}
</style>
