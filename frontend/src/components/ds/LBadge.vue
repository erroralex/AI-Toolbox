<script setup>
const props = defineProps({
  variant: {
    type: String,
    default: 'neutral', // 'accent' | 'secondary' | 'neutral' | 'success' | 'warning' | 'danger'
    validator: (v) => ['accent', 'secondary', 'neutral', 'success', 'warning', 'danger'].includes(v)
  },
  size: {
    type: String,
    default: 'md', // 'sm' | 'md'
    validator: (v) => ['sm', 'md'].includes(v)
  }
});
</script>

<template>
  <span class="l-badge" :class="[variant, size]">
    <slot />
  </span>
</template>

<style scoped>
.l-badge {
  display: inline-flex;
  align-items: center;
  gap: 4px;
  border-radius: var(--radius-full, 999px);
  font-family: var(--font-sans, Inter, sans-serif);
  font-weight: 600;
  letter-spacing: 0.02em;
  white-space: nowrap;
  user-select: none;
  border: 1px solid transparent;
}

.l-badge.sm {
  padding: 1px 6px;
  font-size: 10px;
}
.l-badge.md {
  padding: 2px 8px;
  font-size: 11px;
}

.l-badge.accent {
  background: var(--color-accent-primary-bg, rgba(79, 216, 208, 0.12));
  color: var(--color-accent-primary, #4FD8D0);
  border-color: rgba(79, 216, 208, 0.25);
}

.l-badge.secondary {
  background: var(--color-accent-secondary-bg, rgba(155, 126, 245, 0.12));
  color: var(--color-accent-secondary, #9B7EF5);
  border-color: rgba(155, 126, 245, 0.25);
}

.l-badge.neutral {
  background: var(--color-surface-2, #23252F);
  color: var(--color-text-secondary, #9294A3);
  border-color: var(--color-border-subtle, rgba(255, 255, 255, 0.06));
}

.l-badge.success {
  background: var(--color-success-bg, rgba(61, 214, 140, 0.12));
  color: var(--color-success, #3DD68C);
  border-color: rgba(61, 214, 140, 0.25);
}

.l-badge.warning {
  background: var(--color-warning-bg, rgba(245, 184, 78, 0.12));
  color: var(--color-warning, #F5B84E);
  border-color: rgba(245, 184, 78, 0.25);
}

.l-badge.danger {
  background: var(--color-danger-bg, rgba(242, 102, 91, 0.12));
  color: var(--color-danger, #F2665B);
  border-color: rgba(242, 102, 91, 0.25);
}
</style>
