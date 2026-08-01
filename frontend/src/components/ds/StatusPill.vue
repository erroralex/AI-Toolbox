<script setup>
import { computed } from 'vue';

const props = defineProps({
  status: {
    type: String,
    default: 'online', // 'online' | 'starting' | 'offline'
    validator: (v) => ['online', 'starting', 'offline'].includes(v)
  },
  label: {
    type: String,
    default: ''
  }
});

const statusText = computed(() => {
  if (props.label) return props.label;
  switch (props.status) {
    case 'online': return 'Engine: Online';
    case 'starting': return 'Engine: Starting...';
    case 'offline': return 'Engine: Offline';
    default: return 'Engine Status';
  }
});
</script>

<template>
  <div class="status-pill" :class="status">
    <span class="dot" />
    <span class="status-label">{{ statusText }}</span>
  </div>
</template>

<style scoped>
.status-pill {
  display: inline-flex;
  align-items: center;
  gap: 6px;
  padding: 3px 10px;
  border-radius: var(--radius-full, 999px);
  border: 1px solid var(--color-border-subtle, rgba(255, 255, 255, 0.06));
  font-family: var(--font-sans, Inter, sans-serif);
  font-size: 11px;
  font-weight: 600;
  letter-spacing: 0.02em;
  user-select: none;
}

.dot {
  width: 6px;
  height: 6px;
  border-radius: 50%;
  flex-shrink: 0;
}

/* Status variants */
.status-pill.online {
  background: var(--color-success-bg, rgba(61, 214, 140, 0.12));
  color: var(--color-text-secondary, #9294A3);
}
.status-pill.online .dot {
  background: var(--color-success, #3DD68C);
  animation: ds-pulse 2s infinite var(--ease-standard, cubic-bezier(0.4, 0, 0.2, 1));
}

.status-pill.starting {
  background: var(--color-warning-bg, rgba(245, 184, 78, 0.12));
  color: var(--color-text-secondary, #9294A3);
}
.status-pill.starting .dot {
  background: var(--color-warning, #F5B84E);
  animation: ds-pulse 1s infinite var(--ease-standard, cubic-bezier(0.4, 0, 0.2, 1));
}

.status-pill.offline {
  background: var(--color-danger-bg, rgba(242, 102, 91, 0.12));
  color: var(--color-text-secondary, #9294A3);
}
.status-pill.offline .dot {
  background: var(--color-danger, #F2665B);
}
</style>
