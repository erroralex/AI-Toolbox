<script setup>
const props = defineProps({
  active: {
    type: Boolean,
    default: false
  },
  label: {
    type: String,
    required: true
  },
  count: {
    type: [Number, String],
    default: null
  }
});

const emit = defineEmits(['click']);
</script>

<template>
  <div
    class="nav-item"
    :class="{ active }"
    @click="$emit('click', $event)"
  >
    <span v-if="$slots.icon" class="nav-icon">
      <slot name="icon" />
    </span>
    <span class="nav-label">{{ label }}</span>
    <span v-if="count !== null" class="nav-count">{{ count }}</span>
  </div>
</template>

<style scoped>
.nav-item {
  display: flex;
  align-items: center;
  gap: 10px;
  padding: 8px 12px;
  border-radius: var(--radius-md, 8px);
  cursor: pointer;
  user-select: none;
  font-family: var(--font-sans, Inter, sans-serif);
  font-size: 13px;
  font-weight: 500;
  color: var(--color-text-secondary, #9294A3);
  transition: all var(--duration-fast, 120ms) var(--ease-standard);
  border: 1px solid transparent;
}

.nav-icon {
  display: flex;
  align-items: center;
  color: var(--color-text-tertiary, #6F7180);
  transition: color var(--duration-fast, 120ms) var(--ease-standard);
}

.nav-label {
  flex-grow: 1;
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
}

.nav-count {
  font-family: var(--font-mono, "JetBrains Mono", monospace);
  font-size: 11px;
  padding: 1px 6px;
  border-radius: var(--radius-full, 999px);
  background: var(--color-surface-2, #23252F);
  color: var(--color-text-tertiary, #6F7180);
}

.nav-item:hover:not(.active) {
  background: var(--color-surface-1, #14151B);
  color: var(--color-text-primary, #F2F3F7);
}

.nav-item:hover:not(.active) .nav-icon {
  color: var(--color-text-primary, #F2F3F7);
}

.nav-item.active {
  background: var(--color-accent-primary-bg, rgba(79, 216, 208, 0.12));
  color: var(--color-accent-primary, #4FD8D0);
  font-weight: 600;
  border-color: rgba(79, 216, 208, 0.2);
}

.nav-item.active .nav-icon {
  color: var(--color-accent-primary, #4FD8D0);
}

.nav-item.active .nav-count {
  background: rgba(79, 216, 208, 0.2);
  color: var(--color-accent-primary, #4FD8D0);
}
</style>
