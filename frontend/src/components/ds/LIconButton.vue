<script setup>
const props = defineProps({
  variant: {
    type: String,
    default: 'secondary', // 'secondary' | 'ghost' | 'danger'
    validator: (v) => ['secondary', 'ghost', 'danger'].includes(v)
  },
  size: {
    type: String,
    default: 'md', // 'sm' | 'md' | 'lg'
    validator: (v) => ['sm', 'md', 'lg'].includes(v)
  },
  disabled: {
    type: Boolean,
    default: false
  },
  title: {
    type: String,
    default: ''
  }
});

const emit = defineEmits(['click']);

const handleClick = (e) => {
  if (!props.disabled) emit('click', e);
};
</script>

<template>
  <button
    class="l-icon-button"
    :class="[variant, size, { disabled }]"
    :disabled="disabled"
    :title="title"
    @click="handleClick"
  >
    <slot />
  </button>
</template>

<style scoped>
.l-icon-button {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  border-radius: var(--radius-md, 8px);
  cursor: pointer;
  user-select: none;
  transition: all var(--duration-fast, 120ms) var(--ease-standard);
  outline: none;
  background: transparent;
  color: var(--color-text-secondary, #9294A3);
}

.l-icon-button.sm {
  width: 28px;
  height: 28px;
}
.l-icon-button.md {
  width: 34px;
  height: 34px;
}
.l-icon-button.lg {
  width: 40px;
  height: 40px;
}

.l-icon-button.secondary {
  background: var(--color-surface-1, #14151B);
  border: 1px solid var(--color-border-subtle, rgba(255, 255, 255, 0.06));
}
.l-icon-button.secondary:hover:not(:disabled) {
  background: var(--color-surface-2, #23252F);
  border-color: var(--color-border-strong, rgba(255, 255, 255, 0.18));
  color: var(--color-text-primary, #F2F3F7);
  box-shadow: var(--glow-primary, 0 0 0 3px rgba(79, 216, 208, 0.16));
}

.l-icon-button.ghost {
  border: 1px solid transparent;
}
.l-icon-button.ghost:hover:not(:disabled) {
  background: var(--color-surface-2, #23252F);
  color: var(--color-text-primary, #F2F3F7);
}

.l-icon-button.danger:hover:not(:disabled) {
  background: var(--color-danger-bg, rgba(242, 102, 91, 0.12));
  color: var(--color-danger, #F2665B);
  box-shadow: var(--glow-danger, 0 0 0 3px rgba(242, 102, 91, 0.16));
}

.l-icon-button:disabled,
.l-icon-button.disabled {
  opacity: 0.45;
  cursor: not-allowed;
}
</style>
