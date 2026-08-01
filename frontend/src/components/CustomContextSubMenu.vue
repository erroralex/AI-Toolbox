<script setup>
/**
 * @file CustomContextSubMenu.vue
 * @description A recursive submenu component for the CustomContextMenu aligned with the Latent Design System.
 */
import { ref } from 'vue';

const props = defineProps({
  model: {
    type: Array,
    default: () => []
  }
});

const emit = defineEmits(['execute', 'close']);
const activeSubmenu = ref(null);

const onMouseEnter = (index) => {
  activeSubmenu.value = index;
};

const onMouseLeave = () => {
  activeSubmenu.value = null;
};

const execute = (item) => {
  if (!item.disabled) {
    if (item.command) {
      item.command();
    }
    emit('execute');
  }
};
</script>

<template>
  <ul class="custom-menu-list">
    <template v-for="(item, index) in model" :key="index">
      <li v-if="item.separator" class="menu-separator"></li>

      <li
        v-else-if="item.visible !== false"
        class="menu-item"
        :class="{ 'disabled': item.disabled, 'has-submenu': item.items }"
        @click.stop="execute(item)"
        @mouseenter="onMouseEnter(index)"
        @mouseleave="onMouseLeave"
      >
        <div class="menu-item-content">
          <span v-if="item.icon" :class="['menu-icon', item.icon]"></span>
          <span class="menu-label">{{ item.label }}</span>
          <i v-if="item.items" class="pi pi-angle-right submenu-arrow"></i>
        </div>

        <div v-if="item.items && activeSubmenu === index" class="submenu-wrapper">
          <div class="custom-context-menu submenu">
            <CustomContextSubMenu :model="item.items" @execute="emit('execute')" />
          </div>
        </div>
      </li>
    </template>
  </ul>
</template>

<style scoped>
.custom-menu-list {
  list-style: none;
  padding: 0;
  margin: 0;
}

.menu-separator {
  height: 1px;
  background: var(--color-border-subtle, rgba(255, 255, 255, 0.06));
  margin: 4px 0;
}

.menu-item {
  padding: 8px 12px;
  border-radius: var(--radius-sm, 6px);
  color: var(--color-text-primary, #F2F3F7);
  font-family: var(--font-sans, Inter, sans-serif);
  font-size: 13px;
  cursor: pointer;
  transition: all var(--duration-fast, 120ms) var(--ease-standard);
  display: flex;
  align-items: center;
  position: relative;
}

.menu-item:hover {
  background: var(--color-surface-2, #23252F);
  color: var(--color-accent-primary, #67E0D8);
}

.menu-item.disabled {
  opacity: 0.4;
  cursor: default;
  pointer-events: none;
}

.menu-item-content {
  display: flex;
  align-items: center;
  width: 100%;
}

.menu-icon {
  margin-right: 10px;
  font-size: 14px;
  color: var(--color-text-primary, #F2F3F7);
  width: 16px;
  text-align: center;
  transition: color var(--duration-fast, 120ms) var(--ease-standard);
}

.menu-label {
  flex-grow: 1;
  font-weight: 500;
  color: var(--color-text-primary, #F2F3F7);
  transition: color var(--duration-fast, 120ms) var(--ease-standard);
}

.submenu-arrow {
  font-size: 12px;
  margin-left: 10px;
  color: var(--color-text-secondary, #9294A3);
  transition: color var(--duration-fast, 120ms) var(--ease-standard);
}

.menu-item:hover > .menu-item-content > .menu-icon,
.menu-item:hover > .menu-item-content > .menu-label,
.menu-item:hover > .menu-item-content > .submenu-arrow {
  color: var(--color-accent-primary, #67E0D8) !important;
}

.submenu-wrapper {
  position: absolute;
  top: -4px;
  left: 100%;
  padding-left: 4px;
  z-index: 1000;
  color: var(--color-text-primary, #F2F3F7);
}

</style>
