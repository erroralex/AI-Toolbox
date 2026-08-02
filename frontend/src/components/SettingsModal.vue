<script setup>
/**
 * @file SettingsModal.vue
 * @description Settings modal dialog for Latent Library aligned with the Latent Design System.
 */
import { ref, onMounted } from 'vue';

const props = defineProps({
  visible: { type: Boolean, required: true },
  isRecursive: { type: Boolean, default: false },
  autoShowLatest: { type: Boolean, default: false }
});

const emit = defineEmits(['update:visible', 'update:isRecursive', 'update:autoShowLatest', 'clearDb', 'reindex', 'clearModels', 'clearTags', 'clearUnorganized', 'clearThumbnails', 'openDataFolder']);

const appVersion = ref('1.1.1');

const openKofi = () => {
  const url = 'https://ko-fi.com/error_alex';
  if (window.electronAPI) {
    window.electronAPI.openExternal(url);
  } else {
    window.open(url, '_blank');
  }
};

const close = () => {
  emit('update:visible', false);
};
</script>

<template>
  <div v-if="visible" class="modal-scrim-ds" @click.self="close">
    <div class="modal-box-ds" role="dialog" aria-modal="true">

      <div class="modal-header-ds">
        <div class="modal-title-group-ds">
          <i class="pi pi-cog modal-header-icon-ds"></i>
          <h2 class="modal-title-ds">Settings</h2>
        </div>
        <button class="win-btn-ds" @click="close" title="Close">
          <span>✕</span>
        </button>
      </div>

      <div class="modal-body-ds">

        <!-- Theme Info -->
        <section class="modal-section-ds">
          <h3 class="modal-section-title-ds">
            <i class="pi pi-palette"></i> Appearance
          </h3>
          <div class="theme-info-box-ds">
            <div class="swatch-ds"></div>
            <div class="theme-desc-ds">
              <span class="theme-name-ds">Latent Design System</span>
              <span class="theme-sub-ds">Unified Dark Theme (Cyan & Violet)</span>
            </div>
            <span class="badge-ds accent">Active</span>
          </div>
        </section>

        <!-- Default Toggles -->
        <section class="modal-section-ds">
          <h3 class="modal-section-title-ds">
            <i class="pi pi-sliders-h"></i> Scanning & Monitoring
          </h3>
          <div class="toggle-list-ds">
            <div class="toggle-row-ds">
              <div class="toggle-info-ds">
                <span class="toggle-name-ds">Deep Scan</span>
                <span class="toggle-desc-ds">Recursively scan all subfolders</span>
              </div>
              <label class="toggle-control-ds">
                <input type="checkbox" :checked="isRecursive"
                       @change="e => emit('update:isRecursive', e.target.checked)" class="sr-only"/>
                <span class="toggle-track-ds" :class="{ checked: isRecursive }"><span class="toggle-thumb-ds"></span></span>
              </label>
            </div>
            <div class="toggle-row-ds">
              <div class="toggle-info-ds">
                <span class="toggle-name-ds">Auto-Show Latest</span>
                <span class="toggle-desc-ds">Automatically focus newly generated images</span>
              </div>
              <label class="toggle-control-ds">
                <input type="checkbox" :checked="autoShowLatest"
                       @change="e => emit('update:autoShowLatest', e.target.checked)" class="sr-only"/>
                <span class="toggle-track-ds" :class="{ checked: autoShowLatest }"><span class="toggle-thumb-ds"></span></span>
              </label>
            </div>
          </div>
        </section>

        <!-- Data Management -->
        <section class="modal-section-ds">
          <h3 class="modal-section-title-ds">
            <i class="pi pi-database"></i> Data Management
          </h3>
          <div class="action-buttons-grid-ds">
            <button class="action-btn-ds" @click="emit('openDataFolder')">
              <i class="pi pi-folder-open"></i> Open Data Folder
            </button>
            <button class="action-btn-ds warning" @click="emit('reindex')">
              <i class="pi pi-refresh"></i> Re-index All
            </button>
            <button class="action-btn-ds danger" @click="emit('clearTags')">
              <i class="pi pi-tag"></i> Clear AI Tags
            </button>
            <button class="action-btn-ds danger" @click="emit('clearDb')">
              <i class="pi pi-trash"></i> Wipe Database
            </button>
          </div>
        </section>

        <!-- Support Section -->
        <section class="modal-section-ds">
          <h3 class="modal-section-title-ds">
            <i class="pi pi-heart"></i> Support
          </h3>
          <button class="kofi-btn-ds" @click="openKofi">
            <svg width="18" height="18" viewBox="0 0 24 24" fill="currentColor">
              <path
                  d="M23.881 8.948c-.773-4.085-4.859-4.593-4.859-4.593H.723c-.604 0-.679.798-.679.798s-.082 7.324-.022 11.822c.164 2.424 2.586 2.672 2.586 2.672s8.267-.023 11.966-.049c2.438-.426 2.683-2.566 2.658-3.734 4.352.24 7.422-2.831 6.649-6.916zm-11.062 3.511c-1.246 1.453-4.011 3.976-4.011 3.976s-.121.119-.31.023c-.076-.057-.108-.09-.108-.09-.443-.441-3.368-3.049-4.034-3.954-.709-.965-1.041-2.7-.091-3.71.951-1.01 3.005-1.086 4.363.407 0 0 1.565-1.782 3.468-.963 1.904.82 1.832 3.011.723 4.311zm6.173.478c-.928.116-1.682.028-1.682.028V7.284h1.77s1.971.551 1.971 2.638c0 1.913-.985 2.667-2.059 3.015z"/>
            </svg>
            Support on Ko-fi
          </button>
        </section>

        <section class="modal-section-ds version-section-ds">
          <span class="version-text-ds">Latent Library v{{ appVersion }}</span>
        </section>

      </div>
    </div>
  </div>
</template>

<style scoped>
.modal-scrim-ds {
  position: fixed;
  inset: 0;
  z-index: 2000;
  display: flex;
  align-items: center;
  justify-content: center;
  padding: 20px;
  background: var(--color-surface-overlay, rgba(10, 10, 13, 0.72));
  backdrop-filter: var(--blur-glass, blur(20px));
}

.modal-box-ds {
  width: 100%;
  max-width: 440px;
  background: var(--color-surface-1, #14151B);
  border: 1px solid var(--color-border-strong, rgba(255, 255, 255, 0.18));
  border-radius: var(--radius-xl, 16px);
  box-shadow: var(--shadow-panel, 0 20px 60px -20px rgba(0,0,0,0.65));
  display: flex;
  flex-direction: column;
  overflow: hidden;
}

.modal-header-ds {
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 16px 20px;
  border-bottom: 1px solid var(--color-border-subtle, rgba(255, 255, 255, 0.06));
  background: var(--color-surface-2, #23252F);
}

.modal-title-group-ds {
  display: flex;
  align-items: center;
  gap: 10px;
}

.modal-header-icon-ds {
  color: var(--color-accent-primary, #4FD8D0);
  font-size: 1.1rem;
}

.modal-title-ds {
  margin: 0;
  font-size: 16px;
  font-weight: 700;
  color: var(--color-text-primary, #F2F3F7);
  font-family: var(--font-sans, Inter, sans-serif);
}

.modal-body-ds {
  padding: 0;
  max-height: 75vh;
  overflow-y: auto;
}

.modal-section-ds {
  padding: 16px 20px;
  border-bottom: 1px solid var(--color-border-subtle, rgba(255, 255, 255, 0.06));
}

.modal-section-title-ds {
  display: flex;
  align-items: center;
  gap: 8px;
  margin: 0 0 12px;
  font-size: 11px;
  font-weight: 700;
  text-transform: uppercase;
  letter-spacing: 0.05em;
  color: var(--color-text-secondary, #9294A3);
  font-family: var(--font-sans, Inter, sans-serif);
}

.theme-info-box-ds {
  display: flex;
  align-items: center;
  gap: 12px;
  padding: 12px;
  background: var(--color-surface-2, #23252F);
  border: 1px solid var(--color-border-subtle, rgba(255, 255, 255, 0.06));
  border-radius: var(--radius-md, 8px);
}

.swatch-ds {
  width: 16px;
  height: 16px;
  border-radius: 50%;
  background: var(--gradient-brand, linear-gradient(135deg, #4FD8D0 0%, #9B7EF5 100%));
}

.theme-desc-ds {
  flex: 1;
  display: flex;
  flex-direction: column;
}

.theme-name-ds {
  font-size: 13px;
  font-weight: 600;
  color: var(--color-text-primary, #F2F3F7);
}

.theme-sub-ds {
  font-size: 11px;
  color: var(--color-text-tertiary, #6F7180);
}

.toggle-list-ds {
  display: flex;
  flex-direction: column;
  gap: 10px;
}

.toggle-row-ds {
  display: flex;
  align-items: center;
  justify-content: space-between;
}

.toggle-info-ds {
  display: flex;
  flex-direction: column;
}

.toggle-name-ds {
  font-size: 13px;
  font-weight: 600;
  color: var(--color-text-primary, #F2F3F7);
}

.toggle-desc-ds {
  font-size: 11px;
  color: var(--color-text-tertiary, #6F7180);
}

.toggle-control-ds {
  display: flex;
  align-items: center;
  cursor: pointer;
}

.toggle-track-ds {
  position: relative;
  width: 36px;
  height: 20px;
  border-radius: var(--radius-full, 999px);
  background: var(--color-surface-2, #23252F);
  border: 1px solid var(--color-border-default, rgba(255, 255, 255, 0.10));
  transition: background var(--duration-fast, 120ms), border-color var(--duration-fast, 120ms);
}

.toggle-track-ds.checked {
  background: var(--color-accent-primary, #4FD8D0);
  border-color: var(--color-accent-primary, #4FD8D0);
}

.toggle-thumb-ds {
  position: absolute;
  top: 2px;
  left: 2px;
  width: 14px;
  height: 14px;
  border-radius: 50%;
  background: var(--color-text-secondary, #9294A3);
  transition: transform var(--duration-fast, 120ms) var(--ease-standard), background var(--duration-fast, 120ms);
}

.toggle-track-ds.checked .toggle-thumb-ds {
  transform: translateX(16px);
  background: var(--color-text-on-accent, #06101A);
}

.action-buttons-grid-ds {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 8px;
}

.action-btn-ds {
  display: flex;
  align-items: center;
  gap: 6px;
  padding: 8px 10px;
  border-radius: var(--radius-md, 8px);
  font-size: 12px;
  font-weight: 600;
  cursor: pointer;
  background: var(--color-surface-2, #23252F);
  color: var(--color-text-primary, #F2F3F7);
  border: 1px solid var(--color-border-subtle, rgba(255, 255, 255, 0.06));
  transition: all var(--duration-fast, 120ms) var(--ease-standard);
}

.action-btn-ds:hover {
  border-color: var(--color-border-strong, rgba(255, 255, 255, 0.18));
  background: var(--color-surface-3, #262835);
}

.action-btn-ds.warning:hover {
  border-color: var(--color-warning, #F5B84E);
  color: var(--color-warning, #F5B84E);
}

.action-btn-ds.danger:hover {
  border-color: var(--color-danger, #F2665B);
  color: var(--color-danger, #F2665B);
}

.kofi-btn-ds {
  display: inline-flex;
  align-items: center;
  gap: 8px;
  padding: 10px 16px;
  border-radius: var(--radius-md, 8px);
  font-size: 13px;
  font-weight: 700;
  cursor: pointer;
  background: #FF5E5B;
  color: #fff;
  border: none;
  transition: opacity var(--duration-fast, 120ms);
}

.kofi-btn-ds:hover {
  opacity: 0.9;
}

.badge-ds.accent {
  background: var(--color-accent-primary-bg, rgba(79, 216, 208, 0.12));
  color: var(--color-accent-primary, #4FD8D0);
  border: 1px solid rgba(79, 216, 208, 0.25);
  font-size: 11px;
  padding: 2px 8px;
  border-radius: var(--radius-full, 999px);
}

.win-btn-ds {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  width: 28px;
  height: 28px;
  border-radius: var(--radius-sm, 6px);
  border: 1px solid transparent;
  background: transparent;
  color: var(--color-text-tertiary, #6F7180);
  font-size: 13px;
  cursor: pointer;
}

.win-btn-ds:hover {
  background: var(--color-surface-3, #262835);
  color: var(--color-text-primary, #F2F3F7);
}

.version-section-ds {
  border-bottom: none;
  text-align: center;
}

.version-text-ds {
  font-size: 11px;
  color: var(--color-text-tertiary, #6F7180);
  font-family: var(--font-mono, "JetBrains Mono", monospace);
}

.sr-only {
  position: absolute;
  width: 1px;
  height: 1px;
  opacity: 0;
  pointer-events: none;
}
</style>
