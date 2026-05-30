<template>
  <div class="joker-card" :class="[joker ? joker.rarity : 'empty']" :data-joker-id="joker?.id">
    <template v-if="joker">
      <div class="rarity-corner rarity-corner--tl"></div>
      <div class="rarity-corner rarity-corner--tr"></div>
      <div class="rarity-corner rarity-corner--bl"></div>
      <div class="rarity-corner rarity-corner--br"></div>
      <div class="joker-art">{{ joker.art }}</div>
      <div class="joker-name">{{ joker.name }}</div>
      <div class="joker-rarity-tag" :class="joker.rarity">{{ rarityLabel }}</div>
    </template>
    <template v-else>
      <div class="empty-slot-plus">+</div>
      <div class="empty-slot-label">空槽</div>
    </template>
  </div>
</template>

<script setup>
import { computed } from 'vue'

const props = defineProps({
  joker: { type: Object, default: null },
})

const rarityLabel = computed(() => {
  const map = { common: '普通', rare: '稀有', legendary: '传说' }
  return map[props.joker?.rarity] || ''
})
</script>

<style scoped>
.joker-card {
  width: 140px;
  height: 200px;
  background: linear-gradient(180deg, #fff8e1, #f7e9c4);
  border: 2px solid #1a0f24;
  border-radius: 10px;
  position: relative;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  gap: 6px;
  flex-shrink: 0;
  overflow: hidden;
  transition: transform 0.15s ease, box-shadow 0.15s ease;
}

.joker-card:not(.empty):hover {
  transform: translateY(-4px);
  box-shadow: 0 8px 20px rgba(0,0,0,0.3);
}

/* Empty slot */
.joker-card.empty {
  background: transparent;
  border: 2px dashed rgba(79, 70, 229, 0.4);
}

.empty-slot-plus {
  font-family: 'Inter', sans-serif;
  font-size: 24px;
  color: var(--muted);
  line-height: 1;
}

.empty-slot-label {
  font-family: 'Inter', sans-serif;
  font-size: 12px;
  color: var(--muted);
}

/* Rarity corner decorations */
.rarity-corner {
  position: absolute;
  width: 16px;
  height: 16px;
  pointer-events: none;
}

.joker-card.common .rarity-corner { border-color: #6cb4d3; }
.joker-card.rare .rarity-corner { border-color: #e34b6f; }
.joker-card.legendary .rarity-corner { border-color: #b577ff; }

.rarity-corner--tl {
  top: 4px; left: 4px;
  border-top: 3px solid;
  border-left: 3px solid;
  border-top-left-radius: 4px;
}
.rarity-corner--tr {
  top: 4px; right: 4px;
  border-top: 3px solid;
  border-right: 3px solid;
  border-top-right-radius: 4px;
}
.rarity-corner--bl {
  bottom: 4px; left: 4px;
  border-bottom: 3px solid;
  border-left: 3px solid;
  border-bottom-left-radius: 4px;
}
.rarity-corner--br {
  bottom: 4px; right: 4px;
  border-bottom: 3px solid;
  border-right: 3px solid;
  border-bottom-right-radius: 4px;
}

.joker-art {
  font-size: 48px;
  line-height: 1;
  margin-bottom: 4px;
}

.joker-name {
  font-family: 'Inter', 'PingFang SC', 'Microsoft YaHei', sans-serif;
  font-size: 13px;
  font-weight: 700;
  color: #1a0f24;
  text-align: center;
  padding: 0 8px;
}

.joker-rarity-tag {
  font-family: 'Inter', sans-serif;
  font-size: 10px;
  font-weight: 600;
  padding: 2px 8px;
  border-radius: 10px;
  color: #fff;
}

.joker-rarity-tag.common { background: #6cb4d3; }
.joker-rarity-tag.rare { background: #e34b6f; }
.joker-rarity-tag.legendary { background: #b577ff; }
</style>
