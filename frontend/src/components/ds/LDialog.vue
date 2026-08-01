<script setup>
import { onUnmounted, watch } from 'vue';

const props = defineProps({
  visible: {
    type: Boolean,
    default: false
  },
  title: {
    type: String,
    default: ''
  },
  width: {
    type: String,
    default: '500px'
  }
});

const emit = defineEmits(['update:visible', 'close']);

const closeDialog = () => {
  emit('update:visible', false);
  emit('close');
};

const handleKeydown = (e) => {
  if (e.key === 'Escape' && props.visible) {
    closeDialog();
  }
};

watch(() => props.visible, (val) => {
  if (val) {
    window.addEventListener('keydown', handleKeydown);
  } else {
    window.removeEventListener('keydown', handleKeydown);
  }
});

onUnmounted(() => {
  window.removeEventListener('keydown', handleKeydown);
});
</script>

<template>
  <Teleport to="body">
    <Transition name="ds-modal">
      <div v-if="visible" class="l-dialog-scrim" @click.self="closeDialog">
        <div class="l-dialog-container" :style="{ width }">
          <div class="l-dialog-header">
            <h3 class="l-dialog-title">{{ title }}</h3>
            <button class="close-btn" title="Close" @click="closeDialog">
              <svg width="10" height="10" viewBox="0 0 10 10" fill="none">
                <path d="M1 1L9 9M9 1L1 9" stroke="currentColor" stroke-width="1.5" stroke-linecap="round"/>
              </svg>
            </button>
          </div>
          <div class="l-dialog-body">
            <slot />
          </div>
          <div v-if="$slots.footer" class="l-dialog-footer">
            <slot name="footer" />
          </div>
        </div>
      </div>
    </Transition>
  </Teleport>
</template>

<style scoped>
.l-dialog-scrim {
  position: fixed;
  inset: 0;
  background: var(--color-surface-overlay, rgba(10, 10, 13, 0.72));
  backdrop-filter: var(--blur-glass, blur(20px));
  -webkit-backdrop-filter: var(--blur-glass, blur(20px));
  display: flex;
  align-items: center;
  justify-content: center;
  z-index: 2000;
  padding: 20px;
}

.l-dialog-container {
  background: var(--color-surface-1, #14151B);
  border: 1px solid var(--color-border-default, rgba(255, 255, 255, 0.10));
  border-radius: var(--radius-xl, 16px);
  box-shadow: var(--shadow-panel, 0 20px 60px -20px rgba(0,0,0,0.65));
  max-width: 90vw;
  max-height: 85vh;
  display: flex;
  flex-direction: column;
  overflow: hidden;
}

.l-dialog-header {
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 16px 20px;
  border-bottom: 1px solid var(--color-border-subtle, rgba(255, 255, 255, 0.06));
}

.l-dialog-title {
  margin: 0;
  font-family: var(--font-sans, Inter, sans-serif);
  font-size: 15px;
  font-weight: 700;
  color: var(--color-text-primary, #F2F3F7);
}

.close-btn {
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

.close-btn:hover {
  background: var(--color-surface-2, #23252F);
  color: var(--color-text-primary, #F2F3F7);
}

.l-dialog-body {
  padding: 20px;
  overflow-y: auto;
  flex-grow: 1;
}

.l-dialog-footer {
  padding: 14px 20px;
  border-top: 1px solid var(--color-border-subtle, rgba(255, 255, 255, 0.06));
  display: flex;
  align-items: center;
  justify-content: flex-end;
  gap: 10px;
  background: rgba(0, 0, 0, 0.15);
}

/* Transition */
.ds-modal-enter-active,
.ds-modal-leave-active {
  transition: opacity var(--duration-base, 180ms) var(--ease-standard);
}
.ds-modal-enter-from,
.ds-modal-leave-to {
  opacity: 0;
}
</style>
