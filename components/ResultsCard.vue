<template>
  <div class="card results-card">
    <div class="card-title">📊 Results</div>

    <div v-if="error" class="error-msg">{{ error }}</div>

    <div class="result-row">
      <span class="result-label">1x Pos Size</span>
      <span class="result-value">{{
        results ? "$" + formatNumber(results.posSize1x) : "-"
      }}</span>
      <button
        class="btn btn-icon btn-ghost"
        :disabled="!results"
        @click="$emit('copy', formatNumber(results?.posSize1x))"
      >
        📋
      </button>
    </div>

    <div class="result-row">
      <span class="result-label">{{ leverage }}x Pos Size</span>
      <span class="result-value">{{
        results ? "$" + formatNumber(results.posSizeLev) : "-"
      }}</span>
      <button
        class="btn btn-icon btn-ghost"
        :disabled="!results"
        @click="$emit('copy', formatNumber(results?.posSizeLev))"
      >
        📋
      </button>
    </div>

    <div class="result-row">
      <span class="result-label">Quantity</span>
      <span class="result-value">{{
        results ? formatNumber(results.quantity, 6) : "-"
      }}</span>
      <button
        class="btn btn-icon btn-ghost"
        :disabled="!results"
        @click="$emit('copy', formatNumber(results?.quantity, 6))"
      >
        📋
      </button>
    </div>

    <div class="result-row">
      <span class="result-label">Net Profit</span>
      <span class="result-value profit">{{
        results ? "$" + formatNumber(results.netProfit) : "-"
      }}</span>
      <button
        class="btn btn-icon btn-ghost"
        :disabled="!results"
        @click="$emit('copy', formatNumber(results?.netProfit))"
      >
        📋
      </button>
    </div>

    <div class="result-row">
      <span class="result-label">Net Loss</span>
      <span class="result-value loss">{{
        results ? "$" + formatNumber(results.netLoss) : "-"
      }}</span>
      <button
        class="btn btn-icon btn-ghost"
        :disabled="!results"
        @click="$emit('copy', formatNumber(results?.netLoss))"
      >
        📋
      </button>
    </div>

    <div class="result-row">
      <span class="result-label">Risk/Reward</span>
      <span class="result-value">{{
        results ? formatNumber(results.rrRatio) : "-"
      }}</span>
      <button
        class="btn btn-icon btn-ghost"
        :disabled="!results"
        @click="$emit('copy', formatNumber(results?.rrRatio))"
      >
        📋
      </button>
    </div>

    <div class="result-row">
      <span class="result-label">Breakeven</span>
      <span class="result-value">{{
        results ? formatNumber(results.breakeven) : "-"
      }}</span>
      <button
        class="btn btn-icon btn-ghost"
        :disabled="!results"
        @click="$emit('copy', formatNumber(results?.breakeven))"
      >
        📋
      </button>
    </div>
  </div>
</template>

<script setup lang="ts">
import type { CalculatorResults } from "~/composables/useCalculator";

defineProps<{
  results: CalculatorResults | null;
  leverage: number;
  error: string | null;
}>();

defineEmits<{
  copy: [value: string];
}>();

function formatNumber(num: number, decimals = 2): string {
  if (!num && num !== 0) return "-";
  return num.toFixed(decimals);
}
</script>

<style scoped>
.results-card {
  margin: 8px 0;
}

.card-title {
  font-size: 16px;
  font-weight: 700;
  text-align: center;
  margin-bottom: 10px;
}

.error-msg {
  color: var(--accent-red);
  text-align: center;
  font-size: 13px;
  margin-bottom: 8px;
}

.result-row {
  display: flex;
  align-items: center;
  gap: 8px;
  margin: 5px 0;
}

.result-label {
  flex: 1;
  font-size: 13px;
  color: var(--text-secondary);
}

.result-value {
  font-size: 14px;
  font-weight: 500;
  min-width: 80px;
  text-align: right;
}

.result-value.profit {
  color: var(--accent-green);
}

.result-value.loss {
  color: var(--accent-red);
}
</style>
