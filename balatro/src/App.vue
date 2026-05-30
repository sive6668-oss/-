<template>
  <div class="game-layout">
    <!-- ==================== SIDEBAR ==================== -->
    <aside class="sidebar">
      <!-- Logo -->
      <div class="logo-block">
        <div class="logo-text">小丑牌</div>
        <div class="logo-sub">BALATRO</div>
      </div>

      <!-- Blind Panel -->
      <div class="sb-panel blind-panel">
        <div class="panel-label">当前盲注</div>
        <div class="blind-icon">{{ currentBlind.icon }}</div>
        <div class="blind-name">{{ currentBlind.name }}</div>
        <div class="blind-target">
          目标：<span class="gold">{{ currentBlind.target }}</span>
        </div>
        <div class="blind-reward">
          通关奖励：<span class="gold">${{ currentBlind.reward }}</span>
        </div>
      </div>

      <!-- Round Score Panel -->
      <div class="sb-panel score-panel">
        <div class="panel-label">本轮得分</div>
        <div class="score-progress-wrap">
          <div
            class="score-progress-bar"
            :style="{ width: Math.min(100, (state.blindScore / currentBlind.target) * 100) + '%' }"
          ></div>
        </div>
        <div class="score-numbers">
          <span class="blind-score-val">{{ state.blindScore }}</span>
          <span class="score-sep">/</span>
          <span class="score-target">{{ currentBlind.target }}</span>
        </div>
      </div>

      <!-- Hand Score Panel (chips × mult) -->
      <div class="sb-panel hand-score-panel">
        <div class="panel-label hand-type-name">{{ state.currentHandType || '— 选牌出牌 —' }}</div>
        <div class="chips-mult-row">
          <div class="chips-box" ref="chipsBoxRef">
            <div class="cm-label">筹码</div>
            <div class="cm-val chips-val">{{ state.battleChips }}</div>
          </div>
          <div class="cm-times">×</div>
          <div class="mult-box" ref="multBoxRef">
            <div class="cm-label">倍率</div>
            <div class="cm-val mult-val">{{ state.battleMult }}</div>
          </div>
        </div>
        <div v-if="state.settings.showPreview && state.selectedIndices.size > 0" class="preview-formula">
          <span class="preview-text">预览 {{ previewScore }}</span>
        </div>
      </div>

      <!-- Hands / Discards Panel -->
      <div class="sb-panel hd-panel">
        <div class="hd-row">
          <div class="hd-item">
            <div class="hd-val hands-val">{{ state.handsLeft }}</div>
            <div class="hd-label">出牌次数</div>
          </div>
          <div class="hd-divider"></div>
          <div class="hd-item">
            <div class="hd-val discard-val">{{ state.discardsLeft }}</div>
            <div class="hd-label">弃牌次数</div>
          </div>
        </div>
      </div>

      <!-- Money Panel -->
      <div class="sb-panel money-panel">
        <div class="panel-label">金币</div>
        <div class="money-val">${{ state.money }}</div>
      </div>

      <!-- Ante Info -->
      <div class="sb-panel ante-panel">
        <div class="panel-label">关卡进度</div>
        <div class="ante-dots">
          <div
            v-for="(b, i) in BLINDS"
            :key="i"
            class="ante-dot"
            :class="{
              done: i < state.currentBlindIndex,
              current: i === state.currentBlindIndex,
            }"
          ></div>
        </div>
        <div class="ante-name">
          {{ state.currentBlindIndex + 1 }} / {{ BLINDS.length }} — {{ currentBlind.name }}
        </div>
      </div>

      <!-- Restart Button -->
      <button class="px-btn restart sb-restart" @click="restartGame">
        重新开始
      </button>
    </aside>

    <!-- ==================== MAIN AREA ==================== -->
    <main class="main-area">
      <!-- ===== JOKER SECTION ===== -->
      <section class="joker-section" v-if="state.gamePhase === 'playing'">
        <div class="joker-section-title">JOKERS · {{ state.ownedJokers.length }}/5</div>
        <div class="joker-slots">
          <div v-for="i in 5" :key="i" class="joker-slot-wrap">
            <JokerCard
              :joker="state.ownedJokers[i - 1] || null"
              :ref="el => { if (el) jokerRefs[i-1] = el }"
            />
          </div>
        </div>
      </section>
      <section class="joker-section joker-section--empty" v-else></section>

      <!-- ===== PLAY SECTION ===== -->
      <section class="play-section" ref="playSectionRef">
        <template v-if="state.gamePhase === 'playing'">
          <!-- Played Cards Area -->
          <div class="played-area" ref="playedAreaRef">
            <div v-if="state.playedCards.length === 0" class="played-empty-hint">
              选择手牌组成牌型（1-5 张）
            </div>
            <div class="played-cards-row" v-else>
              <PlayingCard
                v-for="card in state.playedCards"
                :key="card.id"
                :card="card"
                :selected="false"
              />
            </div>
          </div>

          <!-- Deck Pile (absolute bottom-right) -->
          <div class="deck-pile" ref="deckPileRef">
            <div class="deck-card deck-back deck-back--3"></div>
            <div class="deck-card deck-back deck-back--2"></div>
            <div class="deck-card deck-back deck-back--1"></div>
            <div class="deck-count">{{ state.deck.length }}</div>
          </div>
        </template>

        <!-- Shop -->
        <ShopOverlay
          v-else-if="state.gamePhase === 'shop'"
          :shopJokers="state.shopJokers"
          :money="state.money"
          :ownedCount="state.ownedJokers.length"
          :boughtIds="shopBoughtIds"
          @buy="buyJoker"
          @skip="skipShop"
        />

        <!-- Result -->
        <ResultOverlay
          v-else-if="state.gamePhase === 'won' || state.gamePhase === 'lost'"
          :isWon="state.gamePhase === 'won'"
          :money="state.money"
          :ownedJokers="state.ownedJokers"
          @restart="restartGame"
        />
      </section>

      <!-- ===== HAND SECTION ===== -->
      <section class="hand-section" v-if="state.gamePhase === 'playing'">
        <!-- Hand Header -->
        <div class="hand-header">
          <span class="hand-title">手牌</span>
          <span class="hand-selected">已选 {{ state.selectedIndices.size }} / 5 张</span>
        </div>

        <!-- Cards Row -->
        <div class="hand-cards-row" ref="handAreaRef">
          <PlayingCard
            v-for="(card, i) in state.hand"
            :key="card.id"
            :card="card"
            :selected="state.selectedIndices.has(i)"
            :ref="el => { if (el) cardRefs[i] = el }"
            @click="toggleSelect(i)"
          />
        </div>

        <!-- Button Row -->
        <div class="hand-btn-row">
          <button
            class="px-btn play"
            :disabled="state.selectedIndices.size === 0 || state.handsLeft === 0 || state.isAnimating"
            @click="handlePlay"
          >
            出牌 ({{ state.selectedIndices.size }})
          </button>

          <button
            class="px-btn discard"
            :disabled="state.selectedIndices.size === 0 || state.discardsLeft === 0 || state.isAnimating"
            @click="handleDiscard"
          >
            弃牌 ({{ state.discardsLeft }})
          </button>

          <button class="px-btn sort" @click="sortByRank" :disabled="state.isAnimating">
            按点排序
          </button>

          <button class="px-btn sort" @click="sortBySuit" :disabled="state.isAnimating">
            按花排序
          </button>

          <button
            class="px-btn ai"
            :class="{ thinking: state.aiThinking }"
            :disabled="state.isAnimating || state.aiThinking || state.handsLeft === 0"
            @click="aiPlay"
          >
            {{ state.aiThinking ? '🤔 AI 思考中…' : '🤖 AI 出牌' }}
          </button>
        </div>
      </section>
      <section class="hand-section hand-section--hidden" v-else></section>
    </main>

    <!-- ==================== SETTINGS GEAR BTN ==================== -->
    <button class="settings-gear-btn" @click="state.showSettings = true">⚙️</button>

    <!-- ==================== SETTINGS MODAL ==================== -->
    <SettingsModal
      v-if="state.showSettings"
      :settings="state.settings"
      @close="state.showSettings = false"
      @update="updateSetting"
    />

    <!-- ==================== FLYING SCORE DIV (portal) ==================== -->
    <div id="fly-layer" style="position:fixed;inset:0;pointer-events:none;z-index:999;"></div>
  </div>
</template>

<script setup>
import { reactive, computed, ref, nextTick, onMounted } from 'vue'
import { gsap } from 'gsap'
import PlayingCard from './components/PlayingCard.vue'
import JokerCard from './components/JokerCard.vue'
import SettingsModal from './components/SettingsModal.vue'
import ShopOverlay from './components/ShopOverlay.vue'
import ResultOverlay from './components/ResultOverlay.vue'

// ==================== CONSTANTS ====================
const SUITS = ['♠', '♥', '♦', '♣']
const RANKS = ['2','3','4','5','6','7','8','9','10','J','Q','K','A']

function cardValue(rank) {
  if (['J','Q','K'].includes(rank)) return 10
  if (rank === 'A') return 11
  return parseInt(rank)
}

const HAND_TYPES = [
  { name: '同花顺', chips: 100, mult: 8 },
  { name: '四条',   chips: 60,  mult: 7 },
  { name: '葫芦',   chips: 40,  mult: 4 },
  { name: '同花',   chips: 35,  mult: 4 },
  { name: '顺子',   chips: 30,  mult: 4 },
  { name: '三条',   chips: 30,  mult: 3 },
  { name: '两对',   chips: 20,  mult: 2 },
  { name: '对子',   chips: 10,  mult: 2 },
  { name: '高牌',   chips: 5,   mult: 1 },
]

const JOKER_POOL = [
  {
    id: 'jester', name: '小丑', rarity: 'common', price: 3, art: '🃏',
    effect: (cards, c, m, handName) => ({ chips: c, mult: m + 4 })
  },
  {
    id: 'scholar', name: '学者', rarity: 'common', price: 3, art: '📖',
    effect: (cards, c, m, handName) => ({
      chips: c,
      mult: m + cards.filter(x => x.rank === 'A').length * 4
    })
  },
  {
    id: 'hearts', name: '红心收藏家', rarity: 'rare', price: 5, art: '❤️',
    effect: (cards, c, m, handName) =>
      cards.some(x => x.suit === '♥') ? { chips: c, mult: m * 4 } : { chips: c, mult: m }
  },
  {
    id: 'clubs', name: '梅花爱好者', rarity: 'rare', price: 5, art: '♣',
    effect: (cards, c, m, handName) =>
      cards.some(x => x.suit === '♣') ? { chips: c, mult: m * 4 } : { chips: c, mult: m }
  },
  {
    id: 'royal', name: '皇家头牌', rarity: 'rare', price: 5, art: '👑',
    effect: (cards, c, m, handName) =>
      cards.some(x => ['J','Q','K'].includes(x.rank)) ? { chips: c, mult: m * 10 } : { chips: c, mult: m }
  },
  {
    id: 'flush', name: '同花顺大师', rarity: 'legendary', price: 8, art: '🔥',
    effect: (cards, c, m, handName) =>
      handName === '同花顺' ? { chips: c, mult: m + 50 } : { chips: c, mult: m }
  },
]

const RARITY_COLOR = {
  common: '#6cb4d3',
  rare: '#e34b6f',
  legendary: '#b577ff',
}

const BLINDS = [
  { name: '小盲注', target: 300, reward: 5, icon: '🔵' },
  { name: '中盲注', target: 500, reward: 6, icon: '🟡' },
  { name: '大盲注', target: 800, reward: 8, icon: '🔴' },
]

// ==================== STATE ====================
const SETTINGS_KEY = 'balatro.settings'

function loadSettings() {
  try {
    const s = localStorage.getItem(SETTINGS_KEY)
    if (s) return JSON.parse(s)
  } catch {}
  return null
}

const defaultSettings = {
  bgmVolume: 50,
  sfxVolume: 70,
  animSpeed: 'normal',
  showPreview: true,
}

const state = reactive({
  gamePhase: 'playing',
  currentBlindIndex: 0,
  deck: [],
  hand: [],
  playedCards: [],
  selectedIndices: new Set(),
  blindScore: 0,
  handsLeft: 4,
  discardsLeft: 3,
  money: 0,
  ownedJokers: [],
  shopJokers: [],
  isAnimating: false,
  currentHandType: null,
  battleChips: 0,
  battleMult: 0,
  settings: { ...defaultSettings, ...(loadSettings() || {}) },
  showSettings: false,
  aiThinking: false,
})

// Track shop purchased IDs
const shopBoughtIds = ref([])

// Template refs
const chipsBoxRef = ref(null)
const multBoxRef = ref(null)
const playSectionRef = ref(null)
const playedAreaRef = ref(null)
const deckPileRef = ref(null)
const handAreaRef = ref(null)
const cardRefs = ref([])
const jokerRefs = ref([])

// ==================== COMPUTED ====================
const currentBlind = computed(() => BLINDS[state.currentBlindIndex])

const previewScore = computed(() => {
  if (state.selectedIndices.size === 0) return ''
  const cards = [...state.selectedIndices].map(i => state.hand[i]).filter(Boolean)
  const { score } = calcScore(cards, state.ownedJokers)
  return score
})

// ==================== ANIM HELPER ====================
function animDur(ms) {
  const m = { slow: 1.5, normal: 1, fast: 0.6 }
  return ms * (m[state.settings.animSpeed] || 1) / 1000
}

// ==================== GAME LOGIC ====================
function buildDeck() {
  const deck = []
  let id = 0
  for (const suit of SUITS) {
    for (const rank of RANKS) {
      deck.push({ id: id++, suit, rank })
    }
  }
  return deck
}

function shuffle(arr) {
  const a = [...arr]
  for (let i = a.length - 1; i > 0; i--) {
    const j = Math.floor(Math.random() * (i + 1));
    [a[i], a[j]] = [a[j], a[i]]
  }
  return a
}

function identifyHand(cards) {
  if (!cards || cards.length === 0) return HAND_TYPES[8] // 高牌

  const ranks = cards.map(c => c.rank)
  const suits = cards.map(c => c.suit)
  const n = cards.length

  // Count rank frequencies
  const rankCount = {}
  for (const r of ranks) rankCount[r] = (rankCount[r] || 0) + 1
  const counts = Object.values(rankCount).sort((a, b) => b - a)

  // Check flush (all same suit, need 5 cards)
  const isFlush = n === 5 && suits.every(s => s === suits[0])

  // Check straight (need 5 cards)
  const rankOrder = { '2':2,'3':3,'4':4,'5':5,'6':6,'7':7,'8':8,'9':9,'10':10,'J':11,'Q':12,'K':13,'A':14 }
  let isStraight = false
  if (n === 5) {
    const nums = ranks.map(r => rankOrder[r]).sort((a, b) => a - b)
    // Normal straight
    const isNormalStraight = nums[4] - nums[0] === 4 && new Set(nums).size === 5
    // A-2-3-4-5 (wheel)
    const isWheel = JSON.stringify(nums) === JSON.stringify([2,3,4,5,14])
    isStraight = isNormalStraight || isWheel
  }

  if (isFlush && isStraight) return HAND_TYPES[0] // 同花顺
  if (counts[0] === 4) return HAND_TYPES[1] // 四条
  if (counts[0] === 3 && counts[1] === 2) return HAND_TYPES[2] // 葫芦
  if (isFlush) return HAND_TYPES[3] // 同花
  if (isStraight) return HAND_TYPES[4] // 顺子
  if (counts[0] === 3) return HAND_TYPES[5] // 三条
  if (counts[0] === 2 && counts[1] === 2) return HAND_TYPES[6] // 两对
  if (counts[0] === 2) return HAND_TYPES[7] // 对子
  return HAND_TYPES[8] // 高牌
}

function calcScore(cards, ownedJokers) {
  const { name, chips, mult } = identifyHand(cards)
  let c = chips + cards.reduce((s, card) => s + cardValue(card.rank), 0)
  let m = mult
  for (const joker of ownedJokers) {
    const result = joker.effect(cards, c, m, name)
    c = result.chips
    m = result.mult
  }
  return { handName: name, chips: c, mult: m, score: c * m }
}

async function dealCards(n) {
  const needed = Math.min(n, state.deck.length)
  const newCards = state.deck.splice(0, needed)
  const startIdx = state.hand.length

  // Add cards to hand first (invisible)
  for (const card of newCards) {
    state.hand.push(card)
  }

  // Animate new cards
  await nextTick()

  const deckEl = deckPileRef.value
  if (!deckEl) return

  const deckRect = deckEl.getBoundingClientRect()

  for (let i = 0; i < newCards.length; i++) {
    const cardIdx = startIdx + i
    const cardEl = cardRefs.value[cardIdx]?.$el || cardRefs.value[cardIdx]
    if (!cardEl) continue

    const cardRect = cardEl.getBoundingClientRect()

    // Clone for animation — use card-back image to match deck pile
    const clone = cardEl.cloneNode(true)
    clone.style.cssText = `
      position: fixed;
      width: ${cardRect.width}px;
      height: ${cardRect.height}px;
      top: ${deckRect.top}px;
      left: ${deckRect.left}px;
      z-index: 998;
      pointer-events: none;
      background: url('/card-back.jpg') center / cover no-repeat;
      border: 2px solid #1a0f24;
      border-radius: 8px;
      box-shadow: 0 4px 12px rgba(0,0,0,.6);
    `
    // Hide card face content during flight
    Array.from(clone.children).forEach(c => { c.style.opacity = '0' })
    document.body.appendChild(clone)

    // Hide real card during animation
    cardEl.style.opacity = '0'

    gsap.to(clone, {
      top: cardRect.top,
      left: cardRect.left,
      duration: animDur(400),
      delay: i * animDur(60),
      ease: 'power2.out',
      onComplete: () => {
        clone.remove()
        if (cardEl) cardEl.style.opacity = '1'
      }
    })
  }

  await new Promise(resolve => setTimeout(resolve, animDur(400 + newCards.length * 60) * 1000 + 50))
}

function initGame() {
  state.deck = shuffle(buildDeck())
  state.hand = []
  state.playedCards = []
  state.selectedIndices = new Set()
  state.blindScore = 0
  state.handsLeft = 4
  state.discardsLeft = 3
  state.currentHandType = null
  state.battleChips = 0
  state.battleMult = 0
  state.isAnimating = false
  state.aiThinking = false
  cardRefs.value = []
}

function toggleSelect(idx) {
  if (state.isAnimating) return
  const s = new Set(state.selectedIndices)
  if (s.has(idx)) {
    s.delete(idx)
  } else {
    if (s.size >= 5) return
    s.add(idx)
  }
  state.selectedIndices = s

  // Update hand type preview
  if (s.size > 0) {
    const cards = [...s].map(i => state.hand[i]).filter(Boolean)
    state.currentHandType = identifyHand(cards).name
  } else {
    state.currentHandType = null
  }
}

async function handlePlay() {
  if (state.selectedIndices.size === 0 || state.handsLeft === 0 || state.isAnimating) return

  state.isAnimating = true

  const selectedIdxArr = [...state.selectedIndices].sort((a, b) => a - b)
  const playedCards = selectedIdxArr.map(i => state.hand[i])

  // Step 1: Identify hand type
  const { name: handName, chips: baseChips, mult: baseMult } = identifyHand(playedCards)
  state.currentHandType = handName

  // Step 2: Fly cards to play area
  await flyCardsToPlayArea(selectedIdxArr, playedCards)

  // Step 3: Show in play area
  state.playedCards = playedCards
  // Remove played cards from hand
  const newHand = state.hand.filter((_, i) => !state.selectedIndices.has(i))
  state.hand = newHand
  state.selectedIndices = new Set()

  await nextTick()

  // Step 4: Calculate chips
  let chips = baseChips
  let mult = baseMult

  // Animate initial chips + card values
  state.battleChips = 0
  state.battleMult = 0

  // Show base chips
  await animateCounter('chips', 0, chips, animDur(300))

  // Per-card chips fly
  for (const card of playedCards) {
    const val = cardValue(card.rank)
    await flyChipsText(`+${val}`, 'chips')
    chips += val
    await animateCounter('chips', state.battleChips, chips, animDur(200))
  }

  await animateCounter('mult', 0, mult, animDur(300))

  // Step 5: Joker effects
  for (let ji = 0; ji < state.ownedJokers.length; ji++) {
    const joker = state.ownedJokers[ji]
    const before = { chips, mult }
    const result = joker.effect(playedCards, chips, mult, handName)
    chips = result.chips
    mult = result.mult

    // Animate joker card
    const jokerEl = jokerRefs.value[ji]?.$el || jokerRefs.value[ji]
    if (jokerEl) {
      gsap.to(jokerEl, {
        y: -18, scale: 1.15,
        boxShadow: `0 0 18px 6px var(--gold)`,
        duration: animDur(200),
        ease: 'power2.out',
        onComplete: () => {
          gsap.to(jokerEl, {
            y: 0, scale: 1,
            boxShadow: 'none',
            duration: animDur(400),
            ease: 'power2.in'
          })
        }
      })
    }

    // Fly mult text
    const multDiff = mult - before.mult
    if (multDiff > 0) {
      await flyMultText(`+${multDiff} Mult`)
    } else if (mult !== before.mult) {
      await flyMultText(`×${(mult / before.mult).toFixed(1)}`)
    }

    await animateCounter('mult', state.battleMult, mult, animDur(300))
    await delay(animDur(200) * 1000)
  }

  // Step 6: Final score explosion
  const finalScore = chips * mult
  state.battleChips = chips
  state.battleMult = mult

  await flyFinalScore(chips, mult, finalScore)
  await delay(animDur(800) * 1000)

  // Clear play area after formula fades
  state.playedCards = []

  // Step 7: Accumulate score
  state.blindScore += finalScore

  // Step 8: Decrement hands
  state.handsLeft -= 1

  // Step 9: Check win
  if (state.blindScore >= currentBlind.value.target) {
    await delay(400)
    enterShop()
    state.isAnimating = false
    return
  }

  // Step 10: Check loss
  if (state.handsLeft === 0) {
    await delay(400)
    state.gamePhase = 'lost'
    state.isAnimating = false
    return
  }

  // Refill hand
  await dealCards(8 - state.hand.length)
  state.isAnimating = false
}

async function flyCardsToPlayArea(selectedIdxArr, playedCards) {
  const playedArea = playedAreaRef.value
  if (!playedArea) return

  const destRect = playedArea.getBoundingClientRect()
  const destX = destRect.left + destRect.width / 2
  const destY = destRect.top + destRect.height / 2

  const promises = []
  for (let k = 0; k < selectedIdxArr.length; k++) {
    const idx = selectedIdxArr[k]
    const cardEl = cardRefs.value[idx]?.$el || cardRefs.value[idx]
    if (!cardEl) continue

    const srcRect = cardEl.getBoundingClientRect()
    const clone = cardEl.cloneNode(true)
    clone.style.cssText = `
      position: fixed;
      width: ${srcRect.width}px;
      height: ${srcRect.height}px;
      top: ${srcRect.top}px;
      left: ${srcRect.left}px;
      z-index: 997;
      pointer-events: none;
      border-radius: 8px;
      transform-origin: center center;
    `
    document.body.appendChild(clone)
    cardEl.style.opacity = '0'

    const p = new Promise(resolve => {
      gsap.to(clone, {
        left: destX - srcRect.width / 2 + k * 20 - (selectedIdxArr.length * 10),
        top: destY - srcRect.height / 2,
        duration: animDur(350),
        delay: k * animDur(60),
        ease: 'power2.inOut',
        onComplete: () => {
          clone.remove()
          resolve()
        }
      })
    })
    promises.push(p)
  }

  await Promise.all(promises)
}

async function animateCounter(type, from, to, duration) {
  return new Promise(resolve => {
    const obj = { val: from }
    gsap.to(obj, {
      val: to,
      duration,
      ease: 'power1.out',
      onUpdate: () => {
        if (type === 'chips') state.battleChips = Math.round(obj.val)
        else state.battleMult = Math.round(obj.val)
      },
      onComplete: () => {
        if (type === 'chips') state.battleChips = to
        else state.battleMult = to
        resolve()
      }
    })
  })
}

async function flyChipsText(text, type) {
  const targetEl = chipsBoxRef.value
  if (!targetEl) return

  const rect = targetEl.getBoundingClientRect()
  const flyEl = document.createElement('div')
  flyEl.textContent = text
  flyEl.style.cssText = `
    position: fixed;
    left: ${rect.left + rect.width / 2}px;
    top: ${rect.top}px;
    transform: translateX(-50%);
    font-family: 'Inter', sans-serif;
    font-size: 20px;
    font-weight: 900;
    color: #4dd6ff;
    text-shadow: 0 0 8px rgba(77,214,255,.8);
    z-index: 999;
    pointer-events: none;
    white-space: nowrap;
  `
  document.body.appendChild(flyEl)

  return new Promise(resolve => {
    gsap.to(flyEl, {
      y: -50,
      opacity: 0,
      duration: animDur(500),
      ease: 'power2.out',
      onComplete: () => {
        flyEl.remove()
        resolve()
      }
    })
  })
}

async function flyMultText(text) {
  const targetEl = multBoxRef.value
  if (!targetEl) return

  const rect = targetEl.getBoundingClientRect()
  const flyEl = document.createElement('div')
  flyEl.textContent = text
  flyEl.style.cssText = `
    position: fixed;
    left: ${rect.left + rect.width / 2}px;
    top: ${rect.top}px;
    transform: translateX(-50%);
    font-family: 'Inter', sans-serif;
    font-size: 20px;
    font-weight: 900;
    color: #ff8844;
    text-shadow: 0 0 8px rgba(255,136,68,.8);
    z-index: 999;
    pointer-events: none;
    white-space: nowrap;
  `
  document.body.appendChild(flyEl)

  return new Promise(resolve => {
    gsap.to(flyEl, {
      y: -50,
      opacity: 0,
      duration: animDur(500),
      ease: 'power2.out',
      onComplete: () => {
        flyEl.remove()
        resolve()
      }
    })
  })
}

async function flyFinalScore(chips, mult, score) {
  const targetEl = playSectionRef.value
  if (!targetEl) return

  const rect = targetEl.getBoundingClientRect()
  const flyEl = document.createElement('div')
  flyEl.innerHTML = `<span style="color:#4dd6ff">${chips}</span><span style="color:#c9d2e8;margin:0 8px">×</span><span style="color:#ff8844">${mult}</span><span style="color:#c9d2e8;margin:0 8px">=</span><span style="color:#ffc857">${score}</span>`
  flyEl.style.cssText = `
    position: fixed;
    left: ${rect.left + rect.width / 2}px;
    top: ${rect.top + rect.height / 2}px;
    transform: translate(-50%, -50%) scale(0.7);
    font-family: 'Press Start 2P', monospace;
    font-size: 28px;
    font-weight: 900;
    text-shadow: 0 2px 8px rgba(0,0,0,.8);
    z-index: 999;
    pointer-events: none;
    white-space: nowrap;
    background: rgba(5,8,24,.75);
    padding: 14px 22px;
    border-radius: 12px;
    border: 2px solid rgba(255,200,87,.4);
  `
  document.body.appendChild(flyEl)

  return new Promise(resolve => {
    gsap.fromTo(flyEl,
      { scale: 0.7, opacity: 0 },
      {
        scale: 1.1,
        opacity: 1,
        duration: animDur(300),
        ease: 'back.out(1.7)',
        onComplete: () => {
          gsap.to(flyEl, {
            scale: 1.0,
            opacity: 1,
            duration: animDur(600),
            delay: animDur(600),
            onComplete: () => {
              gsap.to(flyEl, {
                y: -40,
                opacity: 0,
                duration: animDur(300),
                onComplete: () => {
                  flyEl.remove()
                  resolve()
                }
              })
            }
          })
        }
      }
    )
  })
}

function delay(ms) {
  return new Promise(resolve => setTimeout(resolve, ms))
}

async function handleDiscard() {
  if (state.selectedIndices.size === 0 || state.discardsLeft === 0 || state.isAnimating) return

  state.isAnimating = true
  const selectedIdxArr = [...state.selectedIndices].sort((a, b) => b - a)
  for (const idx of selectedIdxArr) {
    state.hand.splice(idx, 1)
  }
  state.selectedIndices = new Set()
  state.currentHandType = null
  state.battleChips = 0
  state.battleMult = 0
  state.discardsLeft -= 1

  // Refill with animation
  await dealCards(8 - state.hand.length)
  state.isAnimating = false
}

function sortByRank() {
  if (state.isAnimating) return
  const rankOrder = { '2':2,'3':3,'4':4,'5':5,'6':6,'7':7,'8':8,'9':9,'10':10,'J':11,'Q':12,'K':13,'A':14 }
  state.hand = [...state.hand].sort((a, b) => rankOrder[b.rank] - rankOrder[a.rank])
  state.selectedIndices = new Set()
  state.currentHandType = null
}

function sortBySuit() {
  if (state.isAnimating) return
  const suitOrder = { '♠':0,'♥':1,'♦':2,'♣':3 }
  const rankOrder = { '2':2,'3':3,'4':4,'5':5,'6':6,'7':7,'8':8,'9':9,'10':10,'J':11,'Q':12,'K':13,'A':14 }
  state.hand = [...state.hand].sort((a, b) => {
    const sd = suitOrder[a.suit] - suitOrder[b.suit]
    if (sd !== 0) return sd
    return rankOrder[b.rank] - rankOrder[a.rank]
  })
  state.selectedIndices = new Set()
  state.currentHandType = null
}

async function aiPlay() {
  if (state.isAnimating || state.aiThinking || state.handsLeft === 0) return
  state.aiThinking = true

  await delay(800)

  // Enumerate all subsets of 1-5 cards
  const hand = state.hand
  let bestScore = -1
  let bestIndices = [0]

  for (let mask = 1; mask < (1 << hand.length); mask++) {
    const bits = []
    for (let b = 0; b < hand.length; b++) {
      if (mask & (1 << b)) bits.push(b)
    }
    if (bits.length < 1 || bits.length > 5) continue

    const cards = bits.map(i => hand[i])
    const { score } = calcScore(cards, state.ownedJokers)
    if (score > bestScore) {
      bestScore = score
      bestIndices = bits
    }
  }

  state.selectedIndices = new Set(bestIndices)

  // Update hand type display
  const bestCards = bestIndices.map(i => hand[i])
  state.currentHandType = identifyHand(bestCards).name

  state.aiThinking = false

  await delay(200)
  await handlePlay()
}

function enterShop() {
  // Award money: $5 + hands left * $1
  const reward = currentBlind.value.reward + state.handsLeft * 1
  state.money += reward

  // Random 3 jokers (no duplicates, avoid already owned)
  const ownedIds = state.ownedJokers.map(j => j.id)
  const available = JOKER_POOL.filter(j => !ownedIds.includes(j.id))
  const pool = shuffle(available)
  state.shopJokers = pool.slice(0, Math.min(3, pool.length))
  shopBoughtIds.value = []

  // Check if last blind
  if (state.currentBlindIndex >= BLINDS.length - 1) {
    state.gamePhase = 'won'
    return
  }

  state.gamePhase = 'shop'
}

function buyJoker(joker) {
  if (state.money < joker.price) return
  if (state.ownedJokers.length >= 5) return
  if (shopBoughtIds.value.includes(joker.id)) return

  state.money -= joker.price
  state.ownedJokers.push(joker)
  shopBoughtIds.value = [...shopBoughtIds.value, joker.id]
}

function skipShop() {
  state.currentBlindIndex += 1
  if (state.currentBlindIndex >= BLINDS.length) {
    state.gamePhase = 'won'
    return
  }
  // Reset for next blind
  state.deck = shuffle(buildDeck())
  state.hand = []
  state.playedCards = []
  state.selectedIndices = new Set()
  state.blindScore = 0
  state.handsLeft = 4
  state.discardsLeft = 3
  state.currentHandType = null
  state.battleChips = 0
  state.battleMult = 0
  state.isAnimating = false
  state.aiThinking = false
  state.gamePhase = 'playing'

  nextTick(() => {
    dealCards(8)
  })
}

function restartGame() {
  state.currentBlindIndex = 0
  state.ownedJokers = []
  state.money = 0
  state.gamePhase = 'playing'
  shopBoughtIds.value = []
  initGame()

  nextTick(() => {
    dealCards(8)
  })
}

function updateSetting(key, val) {
  state.settings[key] = val
  try {
    localStorage.setItem(SETTINGS_KEY, JSON.stringify(state.settings))
  } catch {}
}

// ==================== LIFECYCLE ====================
onMounted(() => {
  initGame()
  nextTick(() => {
    dealCards(8)
  })
})
</script>

<style scoped>
/* ==================== LAYOUT ==================== */
.game-layout {
  display: flex;
  height: 100vh;
  overflow: hidden;
  background: linear-gradient(180deg, #0a1438 0%, #1a2858 50%, #0a1438 100%);
}

/* ==================== SIDEBAR ==================== */
.sidebar {
  width: min(28vw, 480px);
  min-width: 280px;
  height: 100vh;
  background: var(--sb-panel);
  border-right: 1px solid rgba(74,107,255,.2);
  display: flex;
  flex-direction: column;
  gap: 8px;
  padding: 12px 12px;
  overflow-y: auto;
  overflow-x: hidden;
  flex-shrink: 0;
}

/* ==================== LOGO ==================== */
.logo-block {
  text-align: center;
  padding: 8px 0 4px;
}

.logo-text {
  font-family: 'Inter', 'PingFang SC', 'Microsoft YaHei', sans-serif;
  font-size: 26px;
  font-weight: 900;
  color: var(--gold);
  letter-spacing: 2px;
  text-shadow: 0 0 20px rgba(255,200,87,.5);
}

.logo-sub {
  font-family: 'Press Start 2P', monospace;
  font-size: 10px;
  color: var(--sb-blue);
  letter-spacing: 4px;
  margin-top: 2px;
}

/* ==================== SB PANELS ==================== */
.sb-panel {
  background: rgba(5,8,24,.4);
  border: 1px solid rgba(74,107,255,.2);
  border-radius: 10px;
  padding: 10px 12px;
}

.panel-label {
  font-family: 'Inter', 'PingFang SC', 'Microsoft YaHei', sans-serif;
  font-size: 11px;
  font-weight: 600;
  color: var(--muted);
  letter-spacing: 1px;
  text-transform: uppercase;
  margin-bottom: 6px;
}

/* Blind Panel */
.blind-panel {
  display: flex;
  flex-direction: column;
  align-items: flex-start;
  gap: 4px;
}

.blind-icon {
  font-size: 20px;
}

.blind-name {
  font-family: 'Inter', 'PingFang SC', 'Microsoft YaHei', sans-serif;
  font-size: 16px;
  font-weight: 700;
  color: var(--text);
}

.blind-target,
.blind-reward {
  font-family: 'Inter', sans-serif;
  font-size: 13px;
  color: var(--text-dim);
}

.gold {
  color: var(--gold);
  font-weight: 700;
}

/* Score Panel */
.score-panel {
  display: flex;
  flex-direction: column;
  gap: 6px;
}

.score-progress-wrap {
  height: 8px;
  background: rgba(0,0,0,.4);
  border-radius: 4px;
  overflow: hidden;
}

.score-progress-bar {
  height: 100%;
  background: linear-gradient(90deg, var(--chips-from), var(--chips-to));
  border-radius: 4px;
  transition: width 0.4s ease;
}

.score-numbers {
  display: flex;
  align-items: baseline;
  gap: 4px;
}

.blind-score-val {
  font-family: 'Inter', sans-serif;
  font-size: 20px;
  font-weight: 900;
  color: var(--chips-from);
}

.score-sep {
  font-family: 'Inter', sans-serif;
  font-size: 14px;
  color: var(--muted);
}

.score-target {
  font-family: 'Inter', sans-serif;
  font-size: 14px;
  color: var(--text-dim);
}

/* Hand Score Panel */
.hand-score-panel {
  display: flex;
  flex-direction: column;
  gap: 6px;
}

.hand-score-panel .panel-label {
  font-family: 'Inter', 'PingFang SC', 'Microsoft YaHei', sans-serif;
  color: var(--muted);
}

.hand-type-name {
  text-transform: none;
  font-size: 13px;
  font-weight: 700;
  letter-spacing: 0.5px;
  color: var(--muted);
}

.chips-mult-row {
  display: flex;
  align-items: center;
  gap: 8px;
}

.chips-box,
.mult-box {
  display: flex;
  flex-direction: column;
  align-items: center;
  flex: 1;
  padding: 6px 8px;
  border-radius: 8px;
  min-width: 0;
}

.chips-box {
  background: linear-gradient(180deg, rgba(77,214,255,.2), rgba(33,150,243,.15));
  border: 1px solid rgba(77,214,255,.4);
}

.mult-box {
  background: linear-gradient(180deg, rgba(255,136,68,.2), rgba(255,51,68,.15));
  border: 1px solid rgba(255,136,68,.4);
}

.cm-label {
  font-family: 'Inter', sans-serif;
  font-size: 10px;
  font-weight: 600;
  color: var(--muted);
  text-transform: uppercase;
}

.cm-val {
  font-family: 'Inter', sans-serif;
  font-size: 22px;
  font-weight: 900;
}

.chips-val { color: var(--chips-from); }
.mult-val { color: var(--mult-from); }

.cm-times {
  font-family: 'Inter', sans-serif;
  font-size: 20px;
  font-weight: 900;
  color: var(--muted);
}

.preview-formula {
  text-align: center;
}

.preview-text {
  font-family: 'Inter', sans-serif;
  font-size: 12px;
  color: var(--gold);
}

/* Hands/Discards Panel */
.hd-panel {
  padding: 8px 12px;
}

.hd-row {
  display: flex;
  align-items: center;
  gap: 8px;
}

.hd-item {
  flex: 1;
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 2px;
}

.hd-val {
  font-family: 'Inter', sans-serif;
  font-size: 24px;
  font-weight: 900;
}

.hands-val { color: #62d18b; }
.discard-val { color: #fb7185; }

.hd-label {
  font-family: 'Inter', 'PingFang SC', 'Microsoft YaHei', sans-serif;
  font-size: 11px;
  color: var(--muted);
}

.hd-divider {
  width: 1px;
  height: 36px;
  background: rgba(255,255,255,.15);
}

/* Money Panel */
.money-panel {
  display: flex;
  flex-direction: column;
  gap: 4px;
}

.money-val {
  font-family: 'Inter', sans-serif;
  font-size: 24px;
  font-weight: 900;
  color: var(--money);
}

/* Ante Panel */
.ante-panel {
  display: flex;
  flex-direction: column;
  gap: 6px;
}

.ante-dots {
  display: flex;
  gap: 8px;
}

.ante-dot {
  width: 16px;
  height: 16px;
  border-radius: 50%;
  background: rgba(255,255,255,.15);
  border: 2px solid rgba(255,255,255,.2);
  transition: all 0.3s;
}

.ante-dot.done {
  background: var(--btn-green);
  border-color: var(--btn-green);
}

.ante-dot.current {
  background: var(--gold);
  border-color: var(--gold);
  box-shadow: 0 0 8px rgba(255,200,87,.6);
}

.ante-name {
  font-family: 'Inter', 'PingFang SC', 'Microsoft YaHei', sans-serif;
  font-size: 12px;
  color: var(--text-dim);
}

/* Restart in sidebar */
.sb-restart {
  margin-top: auto;
  font-size: 14px;
  min-height: 44px;
  padding: 10px 16px;
}

/* ==================== MAIN AREA ==================== */
.main-area {
  flex: 1;
  display: grid;
  grid-template-rows: 230px 1fr 280px;
  height: 100vh;
  overflow: hidden;
  min-width: 0;
}

/* ==================== JOKER SECTION ==================== */
.joker-section {
  display: flex;
  flex-direction: column;
  gap: 8px;
  padding: 16px 20px 8px;
  border-bottom: 1px solid rgba(74,107,255,.15);
  overflow: hidden;
}

.joker-section--empty {
  border-bottom: 1px solid rgba(74,107,255,.15);
}

.joker-section-title {
  font-family: 'Press Start 2P', monospace;
  font-size: 10px;
  color: var(--muted);
  letter-spacing: 2px;
}

.joker-slots {
  display: flex;
  gap: 12px;
  overflow-x: auto;
  padding-bottom: 4px;
}

.joker-slot-wrap {
  flex-shrink: 0;
}

/* ==================== PLAY SECTION ==================== */
.play-section {
  position: relative;
  display: flex;
  align-items: center;
  justify-content: center;
  padding: 16px 20px;
  border-bottom: 1px solid rgba(74,107,255,.15);
  min-height: 0;
  overflow: hidden;
}

.played-area {
  width: 100%;
  height: 100%;
  display: flex;
  align-items: center;
  justify-content: center;
}

.played-empty-hint {
  font-family: 'Inter', 'PingFang SC', 'Microsoft YaHei', sans-serif;
  font-size: 14px;
  color: var(--muted);
  opacity: 0.55;
  text-align: center;
}

.played-cards-row {
  display: flex;
  gap: 12px;
  align-items: center;
}

/* Deck Pile */
.deck-pile {
  position: absolute;
  bottom: 16px;
  right: 16px;
  display: flex;
  align-items: center;
  justify-content: center;
  width: 72px;
  height: 100px;
}

.deck-card {
  position: absolute;
  width: 60px;
  height: 88px;
  border-radius: 6px;
  background: url('/card-back.jpg') center / cover no-repeat;
  border: 2px solid #1a0f24;
  box-shadow: 0 2px 6px rgba(0,0,0,.5);
}

.deck-back--3 { transform: rotate(-6deg) translate(-2px, 3px); }
.deck-back--2 { transform: rotate(-3deg) translate(-1px, 1px); }
.deck-back--1 { transform: rotate(0deg); }

.deck-count {
  position: absolute;
  bottom: -18px;
  left: 50%;
  transform: translateX(-50%);
  font-family: 'Inter', sans-serif;
  font-size: 11px;
  font-weight: 700;
  color: var(--muted);
  white-space: nowrap;
}

/* ==================== HAND SECTION ==================== */
.hand-section {
  display: flex;
  flex-direction: column;
  gap: 10px;
  padding: 14px 20px 16px;
  overflow: hidden;
}

.hand-section--hidden {
  visibility: hidden;
}

.hand-header {
  display: flex;
  align-items: center;
  justify-content: space-between;
}

.hand-title {
  font-family: 'Inter', 'PingFang SC', 'Microsoft YaHei', sans-serif;
  font-size: 14px;
  font-weight: 700;
  color: var(--text-dim);
  text-transform: uppercase;
  letter-spacing: 1px;
}

.hand-selected {
  font-family: 'Inter', sans-serif;
  font-size: 13px;
  color: var(--muted);
}

.hand-cards-row {
  display: flex;
  gap: 8px;
  align-items: flex-end;
  min-height: 145px;
  overflow-x: auto;
  padding-bottom: 4px;
}

.hand-btn-row {
  display: flex;
  gap: 8px;
  align-items: center;
  flex-wrap: nowrap;
  overflow-x: auto;
}
</style>
