<template>
  <div class="result-overlay">
    <div class="result-card">
      <div class="result-icon">{{ isWon ? '🎉' : '💀' }}</div>
      <div class="result-title" :class="isWon ? 'won' : 'lost'">
        {{ isWon ? '通关全部' : '失败' }}
      </div>

      <div class="result-stats">
        <div class="stat-row">
          <span class="stat-label">最终金币</span>
          <span class="stat-val gold">${{ money }}</span>
        </div>
      </div>

      <div v-if="ownedJokers.length > 0" class="result-jokers-section">
        <div class="result-jokers-title">持有 Joker</div>
        <div class="result-jokers">
          <div v-for="j in ownedJokers" :key="j.id" class="result-joker">
            <span class="rj-art">{{ j.art }}</span>
            <span class="rj-name">{{ j.name }}</span>
          </div>
        </div>
      </div>

      <button class="px-btn restart" style="margin-top:8px" @click="$emit('restart')">
        重新开始
      </button>
    </div>
  </div>
</template>

<script setup>
defineProps({
  isWon:       { type: Boolean, required: true },
  money:       { type: Number, required: true },
  ownedJokers: { type: Array, default: () => [] },
})

defineEmits(['restart'])
</script>

<style scoped>
.result-overlay {
  width: 100%;
  height: 100%;
  background: linear-gradient(180deg, var(--bg-deep) 0%, var(--bg-water) 50%, var(--bg-deep) 100%);
  display: flex;
  align-items: center;
  justify-content: center;
}

.result-card {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 20px;
  padding: 48px 40px;
  background: var(--sb-panel);
  border: 2px solid rgba(74,107,255,.4);
  border-radius: 20px;
  max-width: 480px;
  width: 90%;
}

.result-icon {
  font-size: 64px;
  line-height: 1;
}

.result-title {
  font-family: 'Inter', 'PingFang SC', 'Microsoft YaHei', sans-serif;
  font-size: 32px;
  font-weight: 900;
}
.result-title.won { color: var(--gold); }
.result-title.lost { color: #ef4444; }

.result-stats {
  width: 100%;
  background: rgba(0,0,0,.2);
  border-radius: 10px;
  padding: 16px 20px;
}

.stat-row {
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.stat-label {
  font-family: 'Inter', sans-serif;
  font-size: 14px;
  color: var(--text-dim);
}

.stat-val {
  font-family: 'Inter', sans-serif;
  font-size: 20px;
  font-weight: 800;
}
.stat-val.gold { color: var(--gold); }

.result-jokers-section {
  width: 100%;
}

.result-jokers-title {
  font-family: 'Inter', sans-serif;
  font-size: 13px;
  font-weight: 600;
  color: var(--muted);
  margin-bottom: 10px;
  text-align: center;
}

.result-jokers {
  display: flex;
  flex-wrap: wrap;
  gap: 10px;
  justify-content: center;
}

.result-joker {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 4px;
  background: rgba(255,255,255,.08);
  border-radius: 8px;
  padding: 8px 12px;
}

.rj-art { font-size: 28px; }

.rj-name {
  font-family: 'Inter', 'PingFang SC', 'Microsoft YaHei', sans-serif;
  font-size: 11px;
  color: var(--text-dim);
}
</style>
