# 小丑牌 · Balatro Web

基于 [Balatro](https://www.playbalatro.com/) 核心玩法还原的网页扑克 Roguelike 游戏，使用 Vue 3 + Vite + GSAP 开发。

## 核心特性

- 标准 52 张牌 + 9 种经典牌型（同花顺 → 高牌）
- 筹码 × 倍率得分体系，含 Joker 效果叠加
- 小盲注 / 中盲注 / 大盲注 三关递进
- 商店购买 Joker，稀有度分 Common / Rare / Legendary
- GSAP 驱动的发牌、飞牌、得分爆炸动画
- AI 一键出牌（暴力枚举最优子集）
- 设置面板：动画速度 / 音效 / 出牌预览

## 安装与运行

```bash
cd balatro
npm install
npm run dev          # 本地开发，默认 http://localhost:5173
```

## 构建

```bash
cd balatro
npm run build        # 产物输出到 balatro/dist/
npm run preview      # 预览构建产物
```

## 目录结构

```
.
├── balatro/                  # 主游戏工程（Vue 3 + Vite）
│   ├── src/
│   │   ├── App.vue           # 游戏主逻辑与布局
│   │   ├── main.js
│   │   ├── style.css         # 全局 CSS 变量与重置
│   │   └── components/
│   │       ├── PlayingCard.vue
│   │       ├── JokerCard.vue
│   │       ├── ShopOverlay.vue
│   │       ├── ResultOverlay.vue
│   │       └── SettingsModal.vue
│   ├── public/card-back.jpg  # 牌背图片
│   └── package.json
├── 01-第1轮-PRD-小丑牌核心循环.html    # 产品需求文档
└── 01-第1轮-DESIGN-小丑牌核心循环.html # 视觉设计规范
```
