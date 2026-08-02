<script setup>
/**
 * @file ComparisonMetadataPanel.vue
 * @description A reusable sidebar component for displaying and comparing image metadata aligned with the Latent Design System.
 */
import { computed } from 'vue';
import InputText from 'primevue/inputtext';
import LButton from '@/components/ds/LButton.vue';
import api from '@/services/api';
import { Folder, Check } from 'lucide-vue-next';

const props = defineProps({
  metadata: { type: Object, required: false },
  path: { type: String, required: false },
  title: { type: String, required: false },
  actionLabel: { type: String, required: false },
  rating: { type: Number, default: 0 }
});

const emit = defineEmits(['action']);

const fileName = computed(() => props.path ? props.path.split(/[\\/]/).pop() : 'Unknown');
const folderName = computed(() => {
  if (!props.path) return '-';
  const parts = props.path.split(/[\\/]/);
  parts.pop();
  return parts.pop() || 'Root';
});

const fileFormat = computed(() => {
  if (!props.path) return '-';
  const parts = props.path.split('.');
  return parts.length > 1 ? parts.pop().toUpperCase() : 'Unknown';
});

const openFileLocation = async () => {
  if (!props.path) return;
  try {
    await api.post('/system/show-in-explorer', null, { params: { path: props.path } });
  } catch (e) {
    console.error("Failed to open file location", e);
  }
};
</script>

<template>
  <div class="metadata-panel-ds flex flex-column p-3 overflow-y-auto custom-scrollbar">
    <template v-if="metadata">
      <div class="header-title-ds font-bold mb-1 text-overflow-ellipsis overflow-hidden white-space-nowrap" :title="path">
        {{ title || fileName }}
      </div>
      <div
        class="folder-link-ds text-xs mb-3 flex align-items-center gap-1 cursor-pointer"
        @click="openFileLocation"
        title="Click to open folder"
      >
        <Folder :size="12" class="text-accent" />
        <span class="text-overflow-ellipsis overflow-hidden white-space-nowrap">{{ folderName }}</span>
      </div>

      <div class="flex gap-1 mb-3">
        <i
          v-for="i in 5"
          :key="i"
          class="pi text-xs"
          :class="i <= (rating || metadata.rating || 0) ? 'pi-star-fill text-warning' : 'pi-star text-secondary'"
        ></i>
      </div>

      <div class="metadata-grid grid grid-nogutter gap-2 mb-3">
        <div class="col-12">
          <label class="field-label-ds">Model</label>
          <InputText :value="metadata.Model || '-'" readonly class="w-full ds-input" />
        </div>
        <div class="col-6">
          <label class="field-label-ds">Sampler</label>
          <InputText :value="metadata.Sampler || '-'" readonly class="w-full ds-input" />
        </div>
        <div class="col-6">
          <label class="field-label-ds">Scheduler</label>
          <InputText :value="metadata.Scheduler || '-'" readonly class="w-full ds-input" />
        </div>
        <div class="col-6">
          <label class="field-label-ds">Steps</label>
          <InputText :value="metadata.Steps || '-'" readonly class="w-full ds-input" />
        </div>
        <div class="col-6">
          <label class="field-label-ds">Resolution</label>
          <InputText :value="metadata.Resolution || '-'" readonly class="w-full ds-input" />
        </div>
        <div class="col-6">
          <label class="field-label-ds">Size</label>
          <InputText :value="metadata.FileSize || '-'" readonly class="w-full ds-input" />
        </div>
        <div class="col-6">
          <label class="field-label-ds">Format</label>
          <InputText :value="fileFormat" readonly class="w-full ds-input" />
        </div>
        <div class="col-12">
          <label class="field-label-ds">Prompt</label>
          <div class="ds-box p-2 text-xs line-height-2 select-text overflow-y-auto" style="max-height: 100px;">
            {{ metadata.Prompt || 'No prompt' }}
          </div>
        </div>
        <div class="col-12">
          <label class="field-label-ds text-danger font-bold">Negative Prompt</label>
          <div class="ds-box p-2 text-xs line-height-2 select-text overflow-y-auto" style="max-height: 80px;">
            {{ metadata.Negative || 'None' }}
          </div>
        </div>
      </div>

      <LButton
        v-if="actionLabel"
        variant="primary"
        size="md"
        class="mt-auto w-full"
        @click="emit('action')"
      >
        <template #icon><Check :size="16" /></template>
        {{ actionLabel }}
      </LButton>
    </template>

    <div v-else class="flex-grow-1 flex align-items-center justify-content-center text-secondary italic text-sm text-center">
      No metadata available
    </div>
  </div>
</template>

<style scoped>
.metadata-panel-ds {
  width: 300px;
  min-width: 300px;
  background: var(--color-surface-1, #14151B);
  border: 1px solid var(--color-border-subtle, rgba(255, 255, 255, 0.06));
  border-radius: var(--radius-lg, 12px);
  box-shadow: var(--shadow-card, 0 1px 2px rgba(0,0,0,0.4));
}

.header-title-ds {
  font-family: var(--font-sans, Inter, sans-serif);
  font-size: 14px;
  font-weight: 700;
  background: var(--gradient-brand-text, linear-gradient(90deg, #67E0D8, #9B7EF5));
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  background-clip: text;
}

.folder-link-ds {
  color: var(--color-text-secondary, #9294A3);
  transition: color var(--duration-fast, 120ms) var(--ease-standard);
}

.folder-link-ds:hover {
  color: var(--color-text-primary, #F2F3F7);
}

.field-label-ds {
  display: block;
  font-family: var(--font-sans, Inter, sans-serif);
  font-size: 11px;
  font-weight: 700;
  text-transform: uppercase;
  letter-spacing: 0.05em;
  color: var(--color-text-secondary, #9294A3);
  margin-bottom: 4px;
}

.ds-input {
  background: var(--color-surface-2, #23252F) !important;
  border: 1px solid var(--color-border-subtle, rgba(255, 255, 255, 0.06)) !important;
  color: var(--color-text-primary, #F2F3F7) !important;
  border-radius: var(--radius-sm, 6px) !important;
  font-family: var(--font-sans, Inter, sans-serif) !important;
  font-size: 12px !important;
}

.ds-box {
  background: var(--color-surface-2, #23252F);
  border: 1px solid var(--color-border-subtle, rgba(255, 255, 255, 0.06));
  border-radius: var(--radius-md, 8px);
  color: var(--color-text-primary, #F2F3F7);
}

.text-accent {
  color: var(--color-accent-primary, #67E0D8);
}

.text-warning {
  color: var(--color-warning, #F5B84E) !important;
}

.text-danger {
  color: var(--color-danger, #F2665B) !important;
}

.text-secondary {
  color: var(--color-text-secondary, #9294A3) !important;
}
</style>
