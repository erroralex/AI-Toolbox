<script setup>
const props = defineProps({
  variant: {
    type: String,
    default: 'secondary', // 'primary' | 'secondary' | 'ghost' | 'danger' | 'cta'
    validator: (v) => ['primary', 'secondary', 'ghost', 'danger', 'cta'].includes(v)
  },
  size: {
    type: String,
    default: 'md', // 'sm' | 'md' | 'lg'
    validator: (v) => ['sm', 'md', 'lg'].includes(v)
  },
  disabled: {
    type: Boolean,
    default: false
  }
});

const emit = defineEmits(['click']);

const handleClick = (e) => {
  if (!props.disabled) emit('click', e);
};
</script>

<template>
  <button
    class="l-button"
    :class="[variant, size, { disabled }]"
    :disabled="disabled"
    @click="handleClick"
  >
    <slot name="icon" />
    <slot name="icon-left" />
    <slot />
    <slot name="icon-right" />
  </button>
</template>

<style scoped>
.l-button {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  gap: 8px;
  border-radius: var(--radius-md, 8px);
  font-family: var(--font-sans, Inter, sans-serif);
  font-weight: 600;
  cursor: pointer;
  user-select: none;
  white-space: nowrap;
  transition: background var(--duration-base, 180ms) var(--ease-standard),
              border-color var(--duration-base, 180ms) var(--ease-standard),
              box-shadow var(--duration-base, 180ms) var(--ease-standard),
              opacity var(--duration-base, 180ms) var(--ease-standard);
  outline: none;
}

/* Sizes */
.l-button.sm {
  height: 30px;
  padding: 0 10px;
  font-size: 12px;
}
.l-button.md {
  height: 36px;
  padding: 0 14px;
  font-size: 13px;
}
.l-button.lg {
  height: 42px;
  padding: 0 18px;
  font-size: 14px;
}

/* Variants */
.l-button.primary {
  background: var(--color-accent-primary, #4FD8D0);
  color: var(--color-text-on-accent, #06101A);
  border: 1px solid transparent;
}
.l-button.primary:hover:not(:disabled) {
  background: var(--color-accent-primary-hover, #67E0D8);
  box-shadow: var(--glow-primary, 0 0 0 3px rgba(79, 216, 208, 0.16));
}

.l-button.secondary {
  background: var(--color-surface-1, #14151B);
  color: var(--color-text-primary, #F2F3F7);
  border: 1px solid var(--color-border-default, rgba(255, 255, 255, 0.10));
  box-shadow: var(--shadow-card, 0 1px 2px rgba(0,0,0,0.4));
}
.l-button.secondary:hover:not(:disabled) {
  border-color: var(--color-border-strong, rgba(255, 255, 255, 0.18));
  box-shadow: var(--glow-primary, 0 0 0 3px rgba(79, 216, 208, 0.16));
}

.l-button.ghost {
  background: transparent;
  color: var(--color-text-secondary, #9294A3);
  border: 1px solid transparent;
}
.l-button.ghost:hover:not(:disabled) {
  background: var(--color-surface-2, #23252F);
  color: var(--color-text-primary, #F2F3F7);
}

.l-button.danger {
  background: var(--color-danger-bg, rgba(242, 102, 91, 0.12));
  color: var(--color-danger, #F2665B);
  border: 1px solid rgba(242, 102, 91, 0.25);
}
.l-button.danger:hover:not(:disabled) {
  background: rgba(242, 102, 91, 0.22);
  box-shadow: var(--glow-danger, 0 0 0 3px rgba(242, 102, 91, 0.16));
}

.l-button.cta {
  background: var(--gradient-brand, linear-gradient(135deg, #4FD8D0 0%, #9B7EF5 100%));
  color: #06101A;
  border: 1px solid transparent;
}
.l-button.cta:hover:not(:disabled) {
  box-shadow: var(--glow-primary, 0 0 0 3px rgba(79, 216, 208, 0.16));
  opacity: 0.95;
}

.l-button:focus-visible {
  border-color: var(--color-border-focus, #4FD8D0);
  box-shadow: var(--glow-primary, 0 0 0 3px rgba(79, 216, 208, 0.16));
}

.l-button:disabled,
.l-button.disabled {
  opacity: 0.45;
  cursor: not-allowed;
  box-shadow: none !important;
}
</style>
