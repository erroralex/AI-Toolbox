<script setup>
const props = defineProps({
  modelValue: {
    type: [String, Number],
    required: true
  },
  options: {
    type: Array,
    default: () => [] // [{ label: 'Side-by-Side', value: 'split' }]
  },
  size: {
    type: String,
    default: 'md', // 'sm' | 'md'
    validator: (v) => ['sm', 'md'].includes(v)
  }
});

const emit = defineEmits(['update:modelValue', 'change']);

const selectOption = (val) => {
  emit('update:modelValue', val);
  emit('change', val);
};
</script>

<template>
  <div class="segmented-control" :class="size">
    <button
      v-for="opt in options"
      :key="opt.value"
      class="seg-option"
      :class="{ active: modelValue === opt.value }"
      @click="selectOption(opt.value)"
    >
      {{ opt.label }}
    </button>
  </div>
</template>

<style scoped>
.segmented-control {
  display: inline-flex;
  align-items: center;
  padding: 3px;
  background: var(--color-surface-1, #14151B);
  border: 1px solid var(--color-border-subtle, rgba(255, 255, 255, 0.06));
  border-radius: var(--radius-full, 999px);
  gap: 2px;
}

.seg-option {
  border: none;
  background: transparent;
  color: var(--color-text-secondary, #9294A3);
  font-family: var(--font-sans, Inter, sans-serif);
  font-weight: 600;
  border-radius: var(--radius-full, 999px);
  cursor: pointer;
  user-select: none;
  transition: all var(--duration-base, 180ms) var(--ease-standard);
  white-space: nowrap;
}

.segmented-control.sm .seg-option {
  height: 24px;
  padding: 0 10px;
  font-size: 11px;
}

.segmented-control.md .seg-option {
  height: 30px;
  padding: 0 14px;
  font-size: 12px;
}

.seg-option.active {
  background: var(--color-surface-2, #23252F);
  color: var(--color-accent-primary, #4FD8D0);
  box-shadow: 0 1px 3px rgba(0,0,0,0.3);
}

.seg-option:hover:not(.active) {
  color: var(--color-text-primary, #F2F3F7);
}
</style>
