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

const toggle = () => {
  if (props.disabled) return;
  const newValue = !props.modelValue;
  emit('update:modelValue', newValue);
  emit('change', newValue);
};
</script>

<template>
  <label class="l-switch-wrapper" :class="{ disabled }" @click.prevent="toggle">
    <span class="l-switch-track" :class="{ active: modelValue }">
      <span class="l-switch-thumb" />
    </span>
    <span v-if="label || $slots.default" class="l-switch-label">
      <slot>{{ label }}</slot>
    </span>
  </label>
</template>

<style scoped>
.l-switch-wrapper {
  display: inline-flex;
  align-items: center;
  gap: 10px;
  cursor: pointer;
  user-select: none;
  font-family: var(--font-sans, Inter, sans-serif);
  font-size: 13px;
  color: var(--color-text-primary, #F2F3F7);
}

.l-switch-track {
  position: relative;
  width: 36px;
  height: 20px;
  border-radius: var(--radius-full, 999px);
  background: var(--color-surface-2, #23252F);
  border: 1px solid var(--color-border-default, rgba(255, 255, 255, 0.10));
  transition: background var(--duration-base, 180ms) var(--ease-standard),
              border-color var(--duration-base, 180ms) var(--ease-standard);
  flex-shrink: 0;
}

.l-switch-track.active {
  background: var(--color-accent-primary, #4FD8D0);
  border-color: var(--color-accent-primary, #4FD8D0);
}

.l-switch-thumb {
  position: absolute;
  top: 2px;
  left: 2px;
  width: 14px;
  height: 14px;
  border-radius: 50%;
  background: #FFFFFF;
  transition: transform var(--duration-base, 180ms) var(--ease-standard);
}

.l-switch-track.active .l-switch-thumb {
  transform: translateX(16px);
  background: var(--color-text-on-accent, #06101A);
}

.l-switch-wrapper:hover:not(.disabled) .l-switch-track {
  border-color: var(--color-border-strong, rgba(255, 255, 255, 0.18));
}

.l-switch-wrapper.disabled {
  opacity: 0.45;
  cursor: not-allowed;
}
</style>
