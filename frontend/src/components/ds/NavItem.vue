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
  <button
    class="nav-item"
    :class="{ active }"
    @click="$emit('click', $event)"
  >
    <span v-if="active" class="active-indicator" />
    <span v-if="$slots.icon" class="nav-icon">
      <slot name="icon" />
    </span>
    <span class="nav-label">{{ label }}</span>
    <span v-if="count !== null" class="nav-count">{{ count }}</span>
  </button>
</template>

<style scoped>
.nav-item {
  position: relative;
  display: flex;
  align-items: center;
  gap: 10px;
  width: 100%;
  text-align: left;
  padding: 9px 12px 9px 14px;
  border-radius: var(--radius-md, 8px);
  border: none;
  cursor: pointer;
  font-family: var(--font-sans, Inter, sans-serif);
  font-size: 13px;
  font-weight: 600;
  background: transparent;
  color: var(--color-text-tertiary, #6F7180);
  transition: background var(--duration-fast, 120ms) var(--ease-standard),
              color var(--duration-fast, 120ms) var(--ease-standard);
  outline: none;
}

.active-indicator {
  position: absolute;
  left: 0;
  top: 6px;
  bottom: 6px;
  width: 2.5px;
  border-radius: 0 2px 2px 0;
  background: var(--gradient-brand, linear-gradient(135deg, #4FD8D0 0%, #9B7EF5 100%));
}

.nav-icon {
  display: flex;
  align-items: center;
  color: inherit;
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

.nav-item.active {
  background: var(--color-accent-primary-bg, rgba(79, 216, 208, 0.12));
  color: var(--color-text-primary, #F2F3F7);
}
</style>
