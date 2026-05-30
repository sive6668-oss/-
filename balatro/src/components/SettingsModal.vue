<template>
  <div class="settings-overlay" @click.self="$emit('close')">
    <div class="settings-card">
      <div class="settings-header">
        <span class="settings-title">游戏设置</span>
        <button class="close-btn" @click="$emit('close')">✕</button>
      </div>

      <div class="settings-body">
        <!-- BGM Volume -->
        <div class="settings-row">
          <label class="settings-label">BGM 音量</label>
          <div class="slider-row">
            <input
              type="range" min="0" max="100"
              :value="settings.bgmVolume"
              @input="update('bgmVolume', +$event.target.value)"
            />
            <span class="slider-val">{{ settings.bgmVolume }}</span>
          </div>
        </div>

        <!-- SFX Volume -->
        <div class="settings-row">
          <label class="settings-label">SFX 音量</label>
          <div class="slider-row">
            <input
              type="range" min="0" max="100"
              :value="settings.sfxVolume"
              @input="update('sfxVolume', +$event.target.value)"
            />
            <span class="slider-val">{{ settings.sfxVolume }}</span>
          </div>
        </div>

        <!-- Anim Speed -->
        <div class="settings-row">
          <label class="settings-label">动画速度</label>
          <div class="radio-group">
            <label
              v-for="opt in animOpts"
              :key="opt.value"
              class="radio-label"
              :class="{ active: settings.animSpeed === opt.value }"
            >
              <input
                type="radio"
                :value="opt.value"
                :checked="settings.animSpeed === opt.value"
                @change="update('animSpeed', opt.value)"
              />
              {{ opt.label }}
            </label>
          </div>
        </div>

        <!-- Preview Toggle -->
        <div class="settings-row">
          <label class="settings-label">显示出牌公式预览</label>
          <div
            class="toggle"
            :class="{ on: settings.showPreview }"
            @click="update('showPreview', !settings.showPreview)"
          >
            <div class="toggle-thumb"></div>
          </div>
        </div>
      </div>

      <div class="settings-footer">
        <button class="px-btn settings-color" @click="$emit('close')">确认</button>
      </div>
    </div>
  </div>
</template>

<script setup>
const props = defineProps({
  settings: { type: Object, required: true },
})

const emit = defineEmits(['close', 'update'])

const animOpts = [
  { value: 'slow',   label: '慢' },
  { value: 'normal', label: '普通' },
  { value: 'fast',   label: '快' },
]

function update(key, val) {
  emit('update', key, val)
}
</script>

<style scoped>
.settings-overlay {
  position: fixed;
  inset: 0;
  background: rgba(0,0,0,.55);
  display: flex;
  align-items: center;
  justify-content: center;
  z-index: 200;
}

.settings-card {
  width: 600px;
  max-width: 95vw;
  background: var(--bg-water);
  border: 2px solid var(--sb-blue);
  border-radius: 14px;
  overflow: hidden;
}

.settings-header {
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 20px 24px 16px;
  border-bottom: 1px solid rgba(74,107,255,.3);
}

.settings-title {
  font-family: 'Inter', sans-serif;
  font-size: 20px;
  font-weight: 800;
  color: var(--gold);
}

.close-btn {
  background: none;
  border: none;
  color: var(--text-dim);
  font-size: 20px;
  cursor: pointer;
  padding: 4px 8px;
  border-radius: 6px;
  transition: background 0.1s;
}
.close-btn:hover { background: rgba(255,255,255,.1); }

.settings-body {
  padding: 20px 24px;
  display: flex;
  flex-direction: column;
  gap: 20px;
}

.settings-row {
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: 16px;
}

.settings-label {
  font-family: 'Inter', sans-serif;
  font-size: 14px;
  font-weight: 600;
  color: var(--text-dim);
  white-space: nowrap;
}

.slider-row {
  display: flex;
  align-items: center;
  gap: 10px;
  flex: 1;
  max-width: 260px;
}

.slider-row input[type=range] {
  flex: 1;
  accent-color: var(--sb-blue);
}

.slider-val {
  font-family: 'Inter', sans-serif;
  font-size: 13px;
  font-weight: 600;
  color: var(--text);
  min-width: 32px;
  text-align: right;
}

.radio-group {
  display: flex;
  gap: 8px;
}

.radio-label {
  display: flex;
  align-items: center;
  gap: 4px;
  font-family: 'Inter', sans-serif;
  font-size: 13px;
  font-weight: 600;
  color: var(--text-dim);
  cursor: pointer;
  padding: 6px 12px;
  border-radius: 8px;
  border: 1px solid rgba(74,107,255,.3);
  transition: all 0.1s;
}
.radio-label input[type=radio] { display: none; }
.radio-label.active {
  background: var(--sb-blue);
  color: #fff;
  border-color: var(--sb-blue);
}

.toggle {
  width: 52px;
  height: 28px;
  border-radius: 14px;
  background: rgba(255,255,255,.15);
  border: 2px solid rgba(255,255,255,.2);
  cursor: pointer;
  position: relative;
  transition: background 0.2s, border-color 0.2s;
}
.toggle.on {
  background: var(--sb-blue);
  border-color: var(--sb-blue);
}
.toggle-thumb {
  width: 20px;
  height: 20px;
  border-radius: 50%;
  background: #fff;
  position: absolute;
  top: 2px;
  left: 2px;
  transition: left 0.2s;
  box-shadow: 0 1px 4px rgba(0,0,0,.3);
}
.toggle.on .toggle-thumb { left: 26px; }

.settings-footer {
  padding: 16px 24px 20px;
  border-top: 1px solid rgba(74,107,255,.3);
  display: flex;
  justify-content: flex-end;
}
</style>
