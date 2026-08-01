<script setup>
import { ref, onMounted } from 'vue';
import StatusPill from './StatusPill.vue';
import latentMark from '@/assets/latent-mark.svg';
import api from '@/services/api';

const props = defineProps({
  title: {
    type: String,
    default: 'Latent Library'
  }
});

const engineStatus = ref('online');

const checkEngineHealth = async () => {
  try {
    const res = await api.get('/system/health');
    if (res.status === 200) {
      engineStatus.value = 'online';
    } else {
      engineStatus.value = 'starting';
    }
  } catch (e) {
    engineStatus.value = 'offline';
  }
};

onMounted(() => {
  checkEngineHealth();
  setInterval(checkEngineHealth, 10000);
});

const minimize = () => {
  if (window.electronAPI?.minimizeWindow) {
    window.electronAPI.minimizeWindow();
  }
};

const maximize = () => {
  if (window.electronAPI?.maximizeWindow) {
    window.electronAPI.maximizeWindow();
  }
};

const close = () => {
  if (window.electronAPI?.closeWindow) {
    window.electronAPI.closeWindow();
  }
};
</script>

<template>
  <header class="latent-titlebar">
    <div class="brand-section">
      <img :src="latentMark" alt="Latent Logo" class="brand-icon" />
      <span class="brand-title">{{ title }}</span>
    </div>

    <div class="center-section no-drag">
      <slot>
        <StatusPill :status="engineStatus" label="Spring Boot" />
      </slot>
    </div>


    <div class="window-controls no-drag">
      <button class="win-btn min" title="Minimize" @click="minimize">
        <svg width="10" height="1" viewBox="0 0 10 1"><rect width="10" height="1" fill="currentColor"/></svg>
      </button>
      <button class="win-btn max" title="Maximize" @click="maximize">
        <svg width="9" height="9" viewBox="0 0 9 9" fill="none"><rect x="0.5" y="0.5" width="8" height="8" stroke="currentColor"/></svg>
      </button>
      <button class="win-btn close" title="Close" @click="close">
        <svg width="9" height="9" viewBox="0 0 9 9"><path d="M0.5 0.5L8.5 8.5M8.5 0.5L0.5 8.5" stroke="currentColor" stroke-width="1.2"/></svg>
      </button>
    </div>
  </header>
</template>

<style scoped>
.latent-titlebar {
  height: 52px;
  min-height: 52px;
  background: var(--color-bg-canvas, #0A0A0D);
  border-bottom: 1px solid var(--color-border-subtle, rgba(255, 255, 255, 0.06));
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 0 16px;
  user-select: none;
  -webkit-app-region: drag;
  z-index: 1000;
}

.brand-section {
  display: flex;
  align-items: center;
  gap: 10px;
}

.brand-icon {
  width: 20px;
  height: 20px;
}

.brand-title {
  font-family: var(--font-sans, Inter, sans-serif);
  font-size: 13px;
  font-weight: 700;
  color: var(--color-text-primary, #F2F3F7);
  letter-spacing: -0.01em;
}

.no-drag {
  -webkit-app-region: no-drag;
}

.center-section {
  display: flex;
  align-items: center;
}

.window-controls {
  display: flex;
  align-items: center;
  gap: 4px;
}

.win-btn {
  width: 28px;
  height: 28px;
  border-radius: var(--radius-sm, 6px);
  border: none;
  background: transparent;
  color: var(--color-text-tertiary, #6F7180);
  display: flex;
  align-items: center;
  justify-content: center;
  cursor: pointer;
  transition: all var(--duration-fast, 120ms) var(--ease-standard);
}

.win-btn:hover {
  background: var(--color-surface-2, #23252F);
  color: var(--color-text-primary, #F2F3F7);
}

.win-btn.close:hover {
  background: var(--color-danger, #F2665B);
  color: #FFFFFF;
}
</style>
