# CLAUDE.md

## 项目定位

网页版 Balatro（小丑牌）核心循环 Demo，用于验证 PRD 第一轮设计方案的可玩性。技术栈：Vue 3 Composition API + Vite + GSAP。

## 目录约定

| 路径 | 说明 |
|---|---|
| `balatro/src/App.vue` | 游戏全部逻辑（状态、牌型判断、得分计算、动画编排） |
| `balatro/src/components/` | 纯展示组件，不持有游戏状态 |
| `balatro/src/style.css` | 全局 CSS 变量（颜色、字体），组件内用 `var(--xxx)` 引用 |
| `balatro/public/` | 静态资源（牌背图片） |
| `01-第1轮-PRD-*.html` | 产品需求文档（只读参考） |
| `01-第1轮-DESIGN-*.html` | 视觉设计规范（只读参考） |

## 常用命令

```bash
# 开发
cd balatro && npm install
npm run dev          # http://localhost:5173

# 构建 & 预览
npm run build
npm run preview
```

## 关键约定

- 所有游戏状态集中在 `App.vue` 的 `state` reactive 对象，子组件通过 props / emit 通信，不直接修改父状态。
- Joker 效果定义在 `JOKER_POOL` 数组，`effect(cards, chips, mult, handName)` 返回 `{ chips, mult }`，链式叠加。
- 动画时长统一经过 `animDur(ms)` 换算，受设置面板的 `animSpeed` 控制（slow / normal / fast）。
- `balatro/dist/` 已加入 `.gitignore`，不提交构建产物。
