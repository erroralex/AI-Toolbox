<script setup>
const props = defineProps({
  hoverable: {
    type: Boolean,
    default: false
  },
  selected: {
    type: Boolean,
    default: false
  }
});
</script>

<template>
  <div
    class="l-card"
    :class="{ hoverable, selected }"
  >
    <div v-if="$slots.header" class="l-card-header">
      <slot name="header" />
    </div>
    <div class="l-card-body">
      <slot />
    </div>
    <div v-if="$slots.footer" class="l-card-footer">
      <slot name="footer" />
    </div>
  </div>
</template>

<style scoped>
.l-card {
  background: var(--color-surface-1, #14151B);
  border: 1px solid var(--color-border-subtle, rgba(255, 255, 255, 0.06));
  border-radius: var(--radius-lg, 12px);
  box-shadow: var(--shadow-card, 0 1px 2px rgba(0,0,0,0.4), inset 0 1px 0 rgba(255,255,255,0.03));
  overflow: hidden;
  transition: border-color var(--duration-base, 180ms) var(--ease-standard),
              box-shadow var(--duration-base, 180ms) var(--ease-standard),
              transform var(--duration-base, 180ms) var(--ease-standard);
}

.l-card-header {
  padding: 14px 16px;
  border-bottom: 1px solid var(--color-border-subtle, rgba(255, 255, 255, 0.06));
  font-family: var(--font-sans, Inter, sans-serif);
  font-weight: 700;
  font-size: 14px;
  color: var(--color-text-primary, #F2F3F7);
}

.l-card-body {
  padding: 16px;
}

.l-card-footer {
  padding: 12px 16px;
  border-top: 1px solid var(--color-border-subtle, rgba(255, 255, 255, 0.06));
  background: rgba(0, 0, 0, 0.15);
}

.l-card.hoverable:hover {
  border-color: var(--color-border-strong, rgba(255, 255, 255, 0.18));
  box-shadow: var(--shadow-panel, 0 20px 60px -20px rgba(0,0,0,0.65)), var(--glow-primary, 0 0 0 3px rgba(79, 216, 208, 0.16));
}

.l-card.selected {
  border-color: var(--color-accent-primary, #4FD8D0);
  box-shadow: var(--glow-primary, 0 0 0 3px rgba(79, 216, 208, 0.16));
}
</style>
