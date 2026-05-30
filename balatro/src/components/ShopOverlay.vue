<template>
  <div class="shop-overlay">
    <div class="shop-header">
      <div class="shop-title">商店</div>
      <div class="shop-subtitle">
        通关奖励到账！金币 ${{ money }} · Joker 槽 {{ ownedCount }}/5
      </div>
    </div>

    <div class="shop-jokers">
      <div
        v-for="joker in shopJokers"
        :key="joker.id"
        class="shop-joker-item"
        :class="{ 'ai-highlight': aiHighlight === joker.id }"
      >
        <JokerCard :joker="joker" />
        <button
          class="px-btn buy-btn"
          :class="getBuyBtnClass(joker)"
          :disabled="isBuyDisabled(joker)"
          @click="$emit('buy', joker)"
        >
          {{ getBuyBtnText(joker) }}
        </button>
      </div>
    </div>

    <div class="shop-footer">
      <button class="px-btn ai" @click="doAiSuggest">
        🤖 AI 建议
      </button>
      <button class="px-btn shop-skip" style="min-width:160px" @click="$emit('skip')">
        跳过 →
      </button>
    </div>
  </div>
</template>

<script setup>
import { ref } from 'vue'
import JokerCard from './JokerCard.vue'

const props = defineProps({
  shopJokers:  { type: Array, required: true },
  money:       { type: Number, required: true },
  ownedCount:  { type: Number, required: true },
  boughtIds:   { type: Array, default: () => [] },
})

const emit = defineEmits(['buy', 'skip'])

const aiHighlight = ref(null)

function isBought(joker) {
  return props.boughtIds.includes(joker.id)
}

function isBuyDisabled(joker) {
  if (isBought(joker)) return true
  if (props.money < joker.price) return true
  if (props.ownedCount >= 5) return true
  return false
}

function getBuyBtnText(joker) {
  if (isBought(joker)) return '已售出'
  if (props.ownedCount >= 5) return '槽满了'
  if (props.money < joker.price) return '钱不够'
  return `购买 $${joker.price}`
}

function getBuyBtnClass(joker) {
  if (isBuyDisabled(joker)) return 'sort'
  return 'play'
}

function doAiSuggest() {
  // Highlight the best value joker (highest effect potential for cheapest price)
  const available = props.shopJokers.filter(j => !isBought(j) && props.money >= j.price && props.ownedCount < 5)
  if (available.length === 0) {
    aiHighlight.value = null
    return
  }
  // Simple heuristic: legendary > rare > common, tie-break by price
  const rarityScore = { legendary: 3, rare: 2, common: 1 }
  const best = available.reduce((a, b) => {
    const sa = rarityScore[a.rarity] || 0
    const sb = rarityScore[b.rarity] || 0
    if (sa !== sb) return sa > sb ? a : b
    return a.price >= b.price ? a : b
  })
  aiHighlight.value = best.id
  setTimeout(() => { aiHighlight.value = null }, 3000)
}
</script>

<style scoped>
.shop-overlay {
  width: 100%;
  height: 100%;
  background: linear-gradient(180deg, var(--bg-deep) 0%, var(--bg-water) 50%, var(--bg-deep) 100%);
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  gap: 32px;
  padding: 40px 24px;
}

.shop-header {
  text-align: center;
}

.shop-title {
  font-family: 'Inter', 'PingFang SC', 'Microsoft YaHei', sans-serif;
  font-size: 28px;
  font-weight: 800;
  color: var(--gold);
  margin-bottom: 8px;
}

.shop-subtitle {
  font-family: 'Inter', 'PingFang SC', 'Microsoft YaHei', sans-serif;
  font-size: 14px;
  color: var(--text-dim);
}

.shop-jokers {
  display: flex;
  gap: 24px;
  align-items: flex-end;
  flex-wrap: wrap;
  justify-content: center;
}

.shop-joker-item {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 12px;
  transition: transform 0.2s;
}

.shop-joker-item.ai-highlight :deep(.joker-card) {
  box-shadow: 0 0 0 3px var(--gold), 0 0 20px rgba(255,200,87,.6);
  transform: scale(1.05);
}

.buy-btn {
  width: 140px;
  min-height: 44px;
  font-size: 14px;
  padding: 10px 16px;
}

.shop-footer {
  display: flex;
  gap: 16px;
  align-items: center;
}
</style>
