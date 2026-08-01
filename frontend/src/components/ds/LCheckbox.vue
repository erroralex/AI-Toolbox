<script setup>
const props = defineProps({
  modelValue: {
    type: Boolean,
    default: false
  },
  label: {
    type: String,
    default: ''
  },
  disabled: {
    type: Boolean,
    default: false
  }
});

const emit = defineEmits(['update:modelValue', 'change']);

const handleChange = (e) => {
  if (props.disabled) return;
  emit('update:modelValue', e.target.checked);
  emit('change', e.target.checked);
};
</script>

<template>
  <label class="l-checkbox-label" :class="{ disabled }">
    <input
      type="checkbox"
      :checked="modelValue"
      :disabled="disabled"
      class="l-checkbox-input"
      @change="handleChange"
    />
    <span class="l-checkbox-box">
      <svg v-if="modelValue" width="10" height="8" viewBox="0 0 10 8" fill="none">
        <path d="M1 4L3.5 6.5L9 1" stroke="currentColor" stroke-width="1.8" stroke-linecap="round" stroke-linejoin="round"/>
      </svg>
    </span>
    <span v-if="label || $slots.default" class="label-text">
      <slot>{{ label }}</slot>
    </span>
  </label>
</template>

<style scoped>
.l-checkbox-label {
  display: inline-flex;
  align-items: center;
  gap: 8px;
  cursor: pointer;
  user-select: none;
  font-family: var(--font-sans, Inter, sans-serif);
  font-size: 13px;
  color: var(--color-text-primary, #F2F3F7);
}

.l-checkbox-input {
  position: absolute;
  opacity: 0;
  width: 0;
  height: 0;
}

.l-checkbox-box {
  width: 18px;
  height: 18px;
  border-radius: var(--radius-sm, 6px);
  background: var(--color-surface-1, #14151B);
  border: 1px solid var(--color-border-default, rgba(255, 255, 255, 0.10));
  display: flex;
  align-items: center;
  justify-content: center;
  color: var(--color-text-on-accent, #06101A);
  transition: all var(--duration-fast, 120ms) var(--ease-standard);
  flex-shrink: 0;
}

.l-checkbox-input:checked + .l-checkbox-box {
  background: var(--color-accent-primary, #4FD8D0);
  border-color: var(--color-accent-primary, #4FD8D0);
}

.l-checkbox-input:focus-visible + .l-checkbox-box {
  box-shadow: var(--glow-primary, 0 0 0 3px rgba(79, 216, 208, 0.16));
}

.l-checkbox-label:hover:not(.disabled) .l-checkbox-box {
  border-color: var(--color-border-strong, rgba(255, 255, 255, 0.18));
}

.l-checkbox-label.disabled {
  opacity: 0.45;
  cursor: not-allowed;
}
</style>
