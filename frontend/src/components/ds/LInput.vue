<script setup>
const props = defineProps({
  modelValue: {
    type: [String, Number],
    default: ''
  },
  placeholder: {
    type: String,
    default: ''
  },
  type: {
    type: String,
    default: 'text'
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

const emit = defineEmits(['update:modelValue', 'change', 'keydown', 'focus', 'blur']);

const handleInput = (e) => {
  emit('update:modelValue', e.target.value);
};
</script>

<template>
  <div class="l-input-wrapper" :class="[size, { disabled }]">
    <span v-if="$slots['icon-left']" class="icon-left">
      <slot name="icon-left" />
    </span>
    <input
      :type="type"
      :value="modelValue"
      :placeholder="placeholder"
      :disabled="disabled"
      class="l-input"
      @input="handleInput"
      @change="$emit('change', $event)"
      @keydown="$emit('keydown', $event)"
      @focus="$emit('focus', $event)"
      @blur="$emit('blur', $event)"
    />
    <span v-if="$slots['icon-right']" class="icon-right">
      <slot name="icon-right" />
    </span>
  </div>
</template>

<style scoped>
.l-input-wrapper {
  position: relative;
  display: inline-flex;
  align-items: center;
  width: 100%;
}

.l-input {
  width: 100%;
  background: var(--color-surface-1, #14151B);
  color: var(--color-text-primary, #F2F3F7);
  border: 1px solid var(--color-border-default, rgba(255, 255, 255, 0.10));
  border-radius: var(--radius-sm, 6px);
  font-family: var(--font-sans, Inter, sans-serif);
  font-size: 13px;
  outline: none;
  transition: border-color var(--duration-fast, 120ms) var(--ease-standard),
              box-shadow var(--duration-fast, 120ms) var(--ease-standard);
}

.l-input-wrapper.sm .l-input {
  height: 30px;
  padding: 0 10px;
  font-size: 12px;
}
.l-input-wrapper.md .l-input {
  height: 36px;
  padding: 0 12px;
}
.l-input-wrapper.lg .l-input {
  height: 42px;
  padding: 0 14px;
  font-size: 14px;
}

.l-input-wrapper:has(.icon-left) .l-input {
  padding-left: 34px;
}
.l-input-wrapper:has(.icon-right) .l-input {
  padding-right: 34px;
}

.icon-left {
  position: absolute;
  left: 10px;
  display: flex;
  align-items: center;
  color: var(--color-text-tertiary, #6F7180);
  pointer-events: none;
}
.icon-right {
  position: absolute;
  right: 10px;
  display: flex;
  align-items: center;
  color: var(--color-text-tertiary, #6F7180);
}

.l-input::placeholder {
  color: var(--color-text-tertiary, #6F7180);
}

.l-input:focus {
  border-color: var(--color-border-focus, #4FD8D0);
  box-shadow: var(--glow-primary, 0 0 0 3px rgba(79, 216, 208, 0.16));
}

.l-input:disabled {
  opacity: 0.45;
  cursor: not-allowed;
}
</style>
