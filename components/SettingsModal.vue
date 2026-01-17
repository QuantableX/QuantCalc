<template>
  <div v-if="isOpen" class="modal-overlay" @click.self="close">
    <div class="modal">
      <header class="modal-header">
        <h2>⚙️ Settings</h2>
        <button class="btn btn-icon" @click="close">✕</button>
      </header>

      <div class="modal-content">
        <!-- Version -->
        <div class="version-badge">v{{ appVersion }}</div>

        <!-- Window Position -->
        <div class="setting-group">
          <label class="setting-label">Window Position</label>
          <div class="setting-options">
            <button
              class="btn option-btn"
              :class="{ active: position === 'left' }"
              @click="setPosition('left')"
            >
              ◀ Left
            </button>
            <button
              class="btn option-btn"
              :class="{ active: position === 'right' }"
              @click="setPosition('right')"
            >
              Right ▶
            </button>
          </div>
        </div>

        <!-- Color Theme -->
        <div class="setting-group">
          <label class="setting-label">Color Theme</label>
          <div class="setting-options">
            <button
              class="btn option-btn"
              :class="{ active: theme === 'default' }"
              @click="setTheme('default')"
            >
              🎨 Default
            </button>
            <button
              class="btn option-btn"
              :class="{ active: theme === 'monochrome' }"
              @click="setTheme('monochrome')"
            >
              ⬛ Monochrome
            </button>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import type { WindowPosition, ColorTheme } from "~/composables/useConfig";

const runtimeConfig = useRuntimeConfig();
const appVersion = runtimeConfig.public.appVersion;

const props = defineProps<{
  isOpen: boolean;
  position: WindowPosition;
  theme: ColorTheme;
}>();

const emit = defineEmits<{
  close: [];
  "update:position": [value: WindowPosition];
  "update:theme": [value: ColorTheme];
}>();

function close() {
  emit("close");
}

function setPosition(pos: WindowPosition) {
  emit("update:position", pos);
}

function setTheme(t: ColorTheme) {
  emit("update:theme", t);
}
</script>

<style scoped>
.modal-overlay {
  position: fixed;
  inset: 0;
  background: rgba(0, 0, 0, 0.7);
  display: flex;
  align-items: center;
  justify-content: center;
  z-index: 1000;
}

.modal {
  background: var(--bg-card);
  border: 1px solid var(--border-color);
  border-radius: 12px;
  width: 280px;
  max-height: 80vh;
  overflow: hidden;
}

.modal-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 12px 16px;
  border-bottom: 1px solid var(--border-color);
}

.modal-header h2 {
  font-size: 16px;
  font-weight: 600;
}

.modal-content {
  padding: 16px;
}

.version-badge {
  text-align: center;
  font-size: 12px;
  color: var(--text-secondary);
  margin-bottom: 12px;
  padding: 4px 8px;
  background: var(--input-bg);
  border-radius: 4px;
  display: inline-block;
  width: 100%;
}

.setting-group {
  margin-bottom: 16px;
}

.setting-label {
  display: block;
  font-size: 13px;
  color: var(--text-secondary);
  margin-bottom: 8px;
}

.setting-options {
  display: flex;
  gap: 8px;
}

.option-btn {
  flex: 1;
  background: var(--input-bg);
  border: 1px solid var(--border-color);
  color: #aaa;
}

.option-btn.active {
  background: #444;
  border-color: #555;
  color: white;
}
</style>
