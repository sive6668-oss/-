<template>
  <div
    class="playing-card"
    :class="{
      selected: selected,
      'suit-red': isRed,
      'suit-black': !isRed
    }"
    :data-card-id="card.id"
    @click="$emit('click')"
  >
    <div class="card-corner card-corner--tl">
      <span class="corner-rank">{{ card.rank }}</span>
      <span class="corner-suit">{{ card.suit }}</span>
    </div>
    <div class="card-center-suit">{{ card.suit }}</div>
    <div class="card-corner card-corner--br">
      <span class="corner-rank">{{ card.rank }}</span>
      <span class="corner-suit">{{ card.suit }}</span>
    </div>
  </div>
</template>

<script setup>
import { computed } from 'vue'

const props = defineProps({
  card: { type: Object, required: true },
  selected: { type: Boolean, default: false },
})

defineEmits(['click'])

const isRed = computed(() => props.card.suit === '♥' || props.card.suit === '♦')
</script>

<style scoped>
.playing-card {
  width: 100px;
  height: 145px;
  background: linear-gradient(180deg, #fff8e1, #f7e9c4);
  border: 2px solid #1a0f24;
  border-radius: 8px;
  position: relative;
  cursor: pointer;
  user-select: none;
  transition: transform 0.15s ease, box-shadow 0.15s ease;
  flex-shrink: 0;
  display: flex;
  align-items: center;
  justify-content: center;
}

.playing-card.selected {
  transform: translateY(-28px);
  box-shadow: 0 8px 20px rgba(74, 107, 255, 0.6);
  z-index: 10;
}

.playing-card:hover:not(.selected) {
  transform: translateY(-8px);
  box-shadow: 0 4px 12px rgba(0,0,0,0.3);
}

.card-corner {
  position: absolute;
  display: flex;
  flex-direction: column;
  align-items: center;
  line-height: 1.1;
}

.card-corner--tl {
  top: 5px;
  left: 7px;
}

.card-corner--br {
  bottom: 5px;
  right: 7px;
  transform: rotate(180deg);
}

.corner-rank {
  font-family: 'Inter', sans-serif;
  font-size: 14px;
  font-weight: 800;
  line-height: 1;
}

.corner-suit {
  font-size: 12px;
  line-height: 1;
}

.card-center-suit {
  font-size: 38px;
  line-height: 1;
  pointer-events: none;
}

.suit-red .corner-rank,
.suit-red .corner-suit,
.suit-red .card-center-suit {
  color: #cc2200;
}

.suit-black .corner-rank,
.suit-black .corner-suit,
.suit-black .card-center-suit {
  color: #1a0f24;
}
</style>
