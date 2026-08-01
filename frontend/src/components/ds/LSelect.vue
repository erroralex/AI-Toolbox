<script setup>
const props = defineProps({
  modelValue: {
    type: [String, Number, Boolean],
    default: ''
  },
  options: {
    type: Array,
    default: () => [] // [{ label: 'All', value: 'all' }]
  },
  disabled: {
    type: Boolean,
    default: false
  },
  size: {
    type: String,
    default: 'md', // 'sm' | 'md' | 'lg'
    validator: (v) => ['sm', 'md', 'lg'].includes(v)
  }
});

const emit = defineEmits(['update:modelValue', 'change']);

const handleChange = (e) => {
  emit('update:modelValue', e.target.value);
  emit('change', e.target.value);
};
</script>

<template>
  <div class="l-select-wrapper" :class="[size, { disabled }]">
    <select
      :value="modelValue"
      :disabled="disabled"
      class="l-select"
      @change="handleChange"
    >
      <option
        v-for="opt in options"
        :key="opt.value"
        :value="opt.value"
        class="l-option"
      >
        {{ opt.label }}
      </option>
    </select>
    <span class="chevron-icon">
      <svg width="10" height="6" viewBox="0 0 10 6" fill="none">
        <path d="M1 1L5 5L9 1" stroke="currentColor" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round"/>
      </svg>
    </span>
  </div>
</template>

<style scoped>
.l-select-wrapper {
  position: relative;
  display: inline-flex;
  align-items: center;
}

.l-select {
  appearance: none;
  width: 100%;
  background: var(--color-surface-1, #14151B);
  color: var(--color-text-primary, #F2F3F7);
  border: 1px solid var(--color-border-default, rgba(255, 255, 255, 0.10));
  border-radius: var(--radius-sm, 6px);
  font-family: var(--font-sans, Inter, sans-serif);
  font-size: 13px;
  font-weight: 500;
  padding-right: 32px;
  outline: none;
  cursor: pointer;
  transition: border-color var(--duration-fast, 120ms) var(--ease-standard),
              box-shadow var(--duration-fast, 120ms) var(--ease-standard);
}

.l-select-wrapper.sm .l-select {
  height: 30px;
  padding-left: 10px;
  font-size: 12px;
}
.l-select-wrapper.md .l-select {
  height: 36px;
  padding-left: 12px;
}
.l-select-wrapper.lg .l-select {
  height: 42px;
  padding-left: 14px;
  font-size: 14px;
}

.chevron-icon {
  position: absolute;
  right: 12px;
  display: flex;
  align-items: center;
  color: var(--color-text-tertiary, #6F7180);
  pointer-events: none;
}

.l-select:focus {
  border-color: var(--color-border-focus, #4FD8D0);
  box-shadow: var(--glow-primary, 0 0 0 3px rgba(79, 216, 208, 0.16));
}

.l-option {
  background: var(--color-surface-1, #14151B);
  color: var(--color-text-primary, #F2F3F7);
  padding: 8px;
}

.l-select:disabled {
  opacity: 0.45;
  cursor: not-allowed;
}
</style>
