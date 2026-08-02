<script setup>
const props = defineProps({
  modelValue: {
    type: Number,
    default: 0
  },
  min: {
    type: Number,
    default: 0
  },
  max: {
    type: Number,
    default: 100
  },
  step: {
    type: Number,
    default: 1
  },
  disabled: {
    type: Boolean,
    default: false
  }
});

const emit = defineEmits(['update:modelValue', 'change']);

const handleInput = (e) => {
  const val = Number(e.target.value);
  emit('update:modelValue', val);
  emit('change', val);
};
</script>

<template>
  <div class="l-slider-wrapper" :class="{ disabled }">
    <input
      type="range"
      :value="modelValue"
      :min="min"
      :max="max"
      :step="step"
      :disabled="disabled"
      class="l-slider"
      @input="handleInput"
    />
  </div>
</template>

<style scoped>
.l-slider-wrapper {
  display: inline-flex;
  align-items: center;
  width: 100%;
}

.l-slider {
  appearance: none;
  width: 100%;
  height: 6px;
  border-radius: var(--radius-full, 999px);
  background: var(--color-surface-2, #23252F);
  outline: none;
  cursor: pointer;
  transition: background var(--duration-fast, 120ms) var(--ease-standard);
}

.l-slider::-webkit-slider-thumb {
  appearance: none;
  width: 16px;
  height: 16px;
  border-radius: 50%;
  background: var(--color-accent-primary, #4FD8D0);
  border: 2px solid var(--color-bg-canvas, #0A0A0D);
  box-shadow: 0 1px 3px rgba(0,0,0,0.4);
  cursor: pointer;
  transition: transform var(--duration-fast, 120ms) var(--ease-standard),
              box-shadow var(--duration-fast, 120ms) var(--ease-standard);
}

.l-slider::-webkit-slider-thumb:hover {
  transform: scale(1.15);
  box-shadow: var(--glow-primary, 0 0 0 3px rgba(79, 216, 208, 0.16));
}

.l-slider:disabled {
  opacity: 0.45;
  cursor: not-allowed;
}
</style>
