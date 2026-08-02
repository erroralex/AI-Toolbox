<script setup>
import { computed } from 'vue';

const props = defineProps({
  value: {
    type: Number,
    default: 0
  },
  max: {
    type: Number,
    default: 100
  },
  showLabel: {
    type: Boolean,
    default: false
  }
});

const percentage = computed(() => {
  if (props.max <= 0) return 0;
  return Math.min(100, Math.max(0, Math.round((props.value / props.max) * 100)));
});
</script>

<template>
  <div class="l-progress-container">
    <div class="l-progress-track">
      <div
        class="l-progress-fill"
        :style="{ width: `${percentage}%` }"
      />
    </div>
    <span v-if="showLabel" class="l-progress-label">{{ percentage }}%</span>
  </div>
</template>

<style scoped>
.l-progress-container {
  display: flex;
  align-items: center;
  gap: 10px;
  width: 100%;
}

.l-progress-track {
  flex-grow: 1;
  height: 6px;
  border-radius: var(--radius-full, 999px);
  background: var(--color-surface-2, #23252F);
  overflow: hidden;
}

.l-progress-fill {
  height: 100%;
  background: var(--gradient-brand, linear-gradient(135deg, #4FD8D0 0%, #9B7EF5 100%));
  border-radius: var(--radius-full, 999px);
  transition: width var(--duration-base, 180ms) var(--ease-standard);
}

.l-progress-label {
  font-family: var(--font-mono, "JetBrains Mono", monospace);
  font-size: 11px;
  color: var(--color-text-secondary, #9294A3);
  min-width: 36px;
  text-align: right;
}
</style>
