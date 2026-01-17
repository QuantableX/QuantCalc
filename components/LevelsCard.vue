<template>
  <div class="card levels-card">
    <div class="card-header">
      <span
        class="direction-indicator"
        :class="{ long: isLong, short: !isLong }"
      >
        {{ isLong ? "📈 LONG" : "📉 SHORT" }}
      </span>
    </div>

    <div class="level-row">
      <label class="level-label">Entry</label>
      <input
        class="input level-input"
        type="text"
        :value="formatPrice(levels.entry)"
        @input="updateLevel('entry', $event)"
      />
      <button
        class="btn btn-icon btn-ghost"
        @click="$emit('copy', String(levels.entry))"
      >
        📋
      </button>
    </div>

    <div class="level-row">
      <label class="level-label">TP</label>
      <input
        class="input level-input"
        type="text"
        :value="formatPrice(levels.tp)"
        @input="updateLevel('tp', $event)"
      />
      <button
        class="btn btn-icon btn-ghost"
        @click="$emit('copy', String(levels.tp))"
      >
        📋
      </button>
    </div>

    <div class="level-row">
      <label class="level-label">SL</label>
      <input
        class="input level-input"
        type="text"
        :value="formatPrice(levels.sl)"
        @input="updateLevel('sl', $event)"
      />
      <button
        class="btn btn-icon btn-ghost"
        @click="$emit('copy', String(levels.sl))"
      >
        📋
      </button>
    </div>
  </div>
</template>

<script setup lang="ts">
interface Levels {
  entry: number;
  tp: number;
  sl: number;
}

const props = defineProps<{
  isLong: boolean;
  levels: Levels;
}>();

const emit = defineEmits<{
  "update:levels": [levels: Levels];
  copy: [value: string];
}>();

function formatPrice(price: number): string {
  if (!price) return "";
  return price.toString();
}

function updateLevel(key: keyof Levels, event: Event) {
  const target = event.target as HTMLInputElement;
  const value = parseFloat(target.value.replace(/,/g, "")) || 0;
  emit("update:levels", { ...props.levels, [key]: value });
}
</script>

<style scoped>
.levels-card {
  margin: 8px 0;
}

.card-header {
  text-align: center;
  margin-bottom: 8px;
}

.direction-indicator {
  font-size: 16px;
  font-weight: 700;
}

.direction-indicator.long {
  color: var(--accent-green);
}

.direction-indicator.short {
  color: var(--accent-red);
}

.level-row {
  display: flex;
  align-items: center;
  gap: 8px;
  margin: 5px 0;
}

.level-label {
  width: 50px;
  font-size: 14px;
  font-weight: 500;
}

.level-input {
  flex: 1;
}
</style>
