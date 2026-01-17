<template>
  <div
    class="app-wrapper"
    :class="{ tucked: isTucked, 'position-right': windowPosition === 'right' }"
    @mouseenter="onMouseEnter"
    @mouseleave="onMouseLeave"
  >
    <!-- Trigger zone (left for right-position) -->
    <div v-if="windowPosition === 'right'" class="trigger-zone trigger-left">
      <span class="trigger-arrow">{{ isTucked ? "◀" : "▶" }}</span>
    </div>

    <!-- Main content area -->
    <div class="main-container">
      <!-- Scrollable Content -->
      <div class="scroll-content">
        <!-- Header with Pin and Settings -->
        <header class="header">
          <button
            class="btn btn-icon pin-btn"
            :class="{ active: isPinned }"
            @click="togglePin"
            title="Pin window (keeps it visible)"
          >
            📌
          </button>
          <h1 class="title">⚜️ 𝐐𝐮𝐚𝐧𝐭𝐂𝐚𝐥𝐜 ⚜️</h1>
          <button
            class="btn btn-icon settings-btn"
            @click="showSettings = true"
            title="Settings"
          >
            ⚙️
          </button>
        </header>

        <!-- Capture Controls -->
        <div class="capture-row">
          <button
            class="btn btn-primary capture-btn"
            @click="handleCapture"
            :disabled="isProcessing"
          >
            📸 Capture (F9)
          </button>
          <button
            class="btn btn-icon region-btn"
            :class="{ active: scanRegion }"
            @click="toggleRegion"
            title="Select scan region"
          >
            🔲
          </button>
        </div>

        <!-- Status -->
        <div class="status">{{ status }}</div>

        <!-- Long/Short Toggle -->
        <div class="direction-toggle">
          <button
            class="btn direction-btn long"
            :class="{ active: isLong }"
            @click="setDirection(true)"
          >
            ▲ LONG ▲
          </button>
          <button class="btn btn-icon btn-red" @click="handleClear">✕</button>
          <button
            class="btn direction-btn short"
            :class="{ active: !isLong }"
            @click="setDirection(false)"
          >
            ▼ SHORT ▼
          </button>
        </div>

        <!-- Levels Card -->
        <LevelsCard
          :is-long="isLong"
          :levels="levels"
          @update:levels="updateLevels"
          @copy="copyToClipboard"
        />

        <!-- Calculator Card -->
        <CalculatorCard
          :inputs="inputs"
          @update:inputs="updateInputs"
          @calculate="calculate"
        />

        <!-- Results Card -->
        <ResultsCard
          :results="results"
          :leverage="inputs.leverage"
          :error="error"
          @copy="copyToClipboard"
        />
      </div>
    </div>

    <!-- Trigger zone (right for left-position) -->
    <div v-if="windowPosition === 'left'" class="trigger-zone trigger-right">
      <span class="trigger-arrow">{{ isTucked ? "▶" : "◀" }}</span>
    </div>

    <!-- Settings Modal -->
    <SettingsModal
      :is-open="showSettings"
      :position="config.windowPosition"
      :theme="config.colorTheme"
      @close="showSettings = false"
      @update:position="handlePositionChange"
      @update:theme="handleThemeChange"
    />
  </div>
</template>

<script setup lang="ts">
const { inputs, levels, isLong, results, error, calculate, clear } =
  useCalculator();
const {
  fibPrices,
  isProcessing,
  status,
  scanRegion,
  captureAndExtract,
  getLevelPrices,
  clearFibPrices,
} = useFibExtractor();
const {
  config,
  loadConfig,
  setCalcSettings,
  setScanRegion,
  setWindowPosition,
  setColorTheme,
} = useConfig();

const isPinned = ref(false);
const isTucked = ref(true);
const showSettings = ref(false);
const windowPosition = computed(() => config.value.windowPosition || "left");
let invoke: any = null;

// Check if running in Tauri
const isTauri =
  typeof window !== "undefined" && "__TAURI_INTERNALS__" in window;

// Load saved settings on mount
onMounted(async () => {
  await loadConfig();
  if (config.value.scanRegion) {
    scanRegion.value = config.value.scanRegion;
    status.value = `Region: ${config.value.scanRegion[2]}x${config.value.scanRegion[3]}`;
  }
  applyTheme();
  if (isTauri) {
    const core = await import("@tauri-apps/api/core");
    invoke = core.invoke;
    await invoke("setup_window_size");
    // Start tucked
    await invoke("tuck_window", {
      position: config.value.windowPosition || "left",
    });
    try {
      const { register } = await import("@tauri-apps/plugin-global-shortcut");
      await register("F9", handleCapture);
    } catch (e) {
      console.warn("Failed to register hotkey:", e);
    }
  }
});

async function onMouseEnter() {
  if (isTauri && invoke) {
    await invoke("show_window", { position: windowPosition.value });
  }
  isTucked.value = false;
}

async function onMouseLeave() {
  if (isPinned.value) return;
  if (isTauri && invoke) {
    await invoke("tuck_window", { position: windowPosition.value });
  }
  isTucked.value = true;
}

async function togglePin() {
  isPinned.value = !isPinned.value;
  if (isPinned.value && isTauri && invoke) {
    await invoke("show_window", { position: windowPosition.value });
    isTucked.value = false;
  }
}

async function handleCapture() {
  await captureAndExtract();
  // Update levels from extracted fib prices
  const { entry, tp, sl } = getLevelPrices(isLong.value);
  if (entry) levels.entry = entry;
  if (tp) levels.tp = tp;
  if (sl) levels.sl = sl;
}

async function toggleRegion() {
  if (scanRegion.value) {
    scanRegion.value = null;
    setScanRegion(null);
    status.value = "Region cleared";
  } else {
    await selectRegion();
  }
}

async function selectRegion() {
  if (!isTauri || !invoke) return;

  // Open the region selector window
  await invoke("open_region_selector");

  // Poll for result (window closes after selection)
  const checkResult = async () => {
    const region = await invoke<[number, number, number, number] | null>(
      "get_selected_region"
    );
    if (region) {
      scanRegion.value = region;
      setScanRegion(region);
      status.value = `Region: ${region[2]}x${region[3]}`;
    } else {
      // Check if selector window still exists
      const { WebviewWindow } = await import("@tauri-apps/api/webviewWindow");
      const selectorWindow = await WebviewWindow.getByLabel("region-selector");
      if (selectorWindow) {
        // Still selecting, check again
        setTimeout(checkResult, 100);
      } else {
        status.value = "Region selection cancelled";
      }
    }
  };

  // Start polling after a short delay
  setTimeout(checkResult, 200);
}

function setDirection(long: boolean) {
  isLong.value = long;
  // Refresh levels if we have fib prices
  if (Object.keys(fibPrices.value).length > 0) {
    const { entry, tp, sl } = getLevelPrices(long);
    if (entry) levels.entry = entry;
    if (tp) levels.tp = tp;
    if (sl) levels.sl = sl;
  }
}

function handleClear() {
  clear();
  clearFibPrices();
}

function updateLevels(newLevels: typeof levels) {
  Object.assign(levels, newLevels);
}

function updateInputs(newInputs: typeof inputs) {
  Object.assign(inputs, newInputs);
  setCalcSettings(newInputs);
}

async function copyToClipboard(value: string) {
  try {
    if (window.__TAURI__) {
      const { writeText } =
        await import("@tauri-apps/plugin-clipboard-manager");
      await writeText(value);
    } else {
      await navigator.clipboard.writeText(value);
    }
  } catch (e) {
    console.warn("Copy failed:", e);
  }
}

async function handlePositionChange(position: "left" | "right") {
  setWindowPosition(position);
  if (isTauri && invoke) {
    await invoke("set_window_position", { position });
  }
}

function handleThemeChange(theme: "default" | "monochrome") {
  setColorTheme(theme);
  applyTheme();
}

function applyTheme() {
  if (config.value.colorTheme) {
    document.documentElement.setAttribute(
      "data-theme",
      config.value.colorTheme
    );
  }
}
</script>

<style scoped>
.app-wrapper {
  height: 100vh;
  width: 340px;
  display: flex;
  flex-direction: row;
  overflow: hidden;
}

/* Tucked state - window shrinks, hide main content */
.app-wrapper.tucked .main-container {
  display: none;
}

.main-container {
  width: 320px;
  flex-shrink: 0;
  display: flex;
  flex-direction: column;
  background: var(--bg-primary);
}

.trigger-zone {
  width: 20px;
  flex-shrink: 0;
  background: #1a1a2e;
  display: flex;
  align-items: center;
  justify-content: center;
  cursor: pointer;
}
.trigger-right {
  border-left: 1px solid #444;
}
.trigger-left {
  border-right: 1px solid #444;
}

.trigger-zone:hover {
  background: #252540;
}

.trigger-arrow {
  color: #888;
  font-size: 14px;
}

.trigger-zone:hover .trigger-arrow {
  color: #fff;
}

.scroll-content {
  flex: 1;
  overflow-y: auto;
  padding: 8px 12px;
}

.header {
  position: relative;
  text-align: center;
  padding: 6px 0;
}

.pin-btn {
  position: absolute;
  left: 0;
  top: 6px;
}
.pin-btn.active {
  background: var(--accent-green-dim);
}

.settings-btn {
  position: absolute;
  right: 0;
  top: 6px;
}

.title {
  font-size: 22px;
  font-weight: 800;
  background: linear-gradient(135deg, #ffd700, #ff8c00);
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
}

.capture-row {
  display: flex;
  justify-content: center;
  gap: 8px;
  margin: 6px 0;
}

.capture-btn {
  min-width: 140px;
}

.region-btn.active {
  background: var(--accent-green-dim);
  color: white;
}

.status {
  text-align: center;
  font-size: 13px;
  color: var(--text-secondary);
  margin: 3px 0;
}

.direction-toggle {
  display: flex;
  justify-content: center;
  align-items: center;
  gap: 8px;
  margin: 6px 0;
}

.direction-btn {
  width: 120px;
  font-weight: 700;
}

.direction-btn.long.active {
  background: var(--accent-green-dim);
  color: white;
}

.direction-btn.short.active {
  background: var(--accent-red-dim);
  color: white;
}

.direction-btn:not(.active) {
  background: #444;
  color: var(--text-secondary);
}
</style>
