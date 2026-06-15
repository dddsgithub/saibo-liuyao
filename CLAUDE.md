# 赛博六爻 (Cyber I Ching)

## 项目概述
金钱卦 PWA —— 三枚铜钱，六次起卦。故宫风视觉（红墙青砖金顶），Web Audio 合成铜钱音效，64 卦完整数据。纯静态单文件，部署于 GitHub Pages。

## 关键 URL
- **线上地址**: `https://dddsgithub.github.io/saibo-liuyao/`
- **Git 仓库**: `https://github.com/dddsgithub/saibo-liuyao`
- **本地路径**: `D:\赛博六爻`
- **唯一源文件**: `index.html`（~1260 行）

## 技术架构

```
index.html (纯静态单文件)
├── <style>        CSS (~530 行) — 故宫风设计系统
├── <body>         HTML (~90 行) — 三个屏幕：welcome / casting / result
└── <script>       JS (~560 行) — 见下方分模块
```

### JS 模块（按出现顺序）

| 模块 | 行号范围 | 职责 |
|------|---------|------|
| AudioEngine | 689–760 | Web Audio 铜钱音效，`async _ensure()` 等 iOS resume |
| HapticEngine | 762–788 | 震动反馈（iOS switch hack + Android vibrate） |
| HEXAGRAMS | 794–860 | 64 卦完整数据（卦辞/彖传/大象/爻辞/用九用六） |
| 核心函数 | 862–889 | `getCoinResult`, `buildBinary`, `transformBinary`, `getHexagram` |
| App State | 892–904 | `state` 对象（screen, question, currentLine, coinStates, lines） |
| DOM Refs | 906–920 | `$()` 速记，所有 DOM 元素引用 |
| Screen | 923–930 | `showScreen()` 三屏切换 |
| Toast | 935–950 | `showToast()` 提示 |
| Coin UI | 954–1010 | `renderCoins`, `updateLiveResult`, `showPlaceholder`, `enableConfirm`, `getRandomBytes` |
| Line Locking | 1013–1055 | `lockCurrentLine`, `advanceLine`, `confirmLine`, `randomThrow` |
| Result | 1100–1170 | `showResult` — 构建卦象展示 HTML |
| Prompt Copy | 1177–1240 | `copyPrompt` — 生成解卦 Prompt 复制到剪贴板 |
| Event Handlers | 1243–1271 | 按钮/键盘事件绑定 |
| PWA SW | 1273–1278 | 内联 Service Worker（缓存策略 cache-first） |

### 核心数据流

```
用户操作 → coinStates[3] → getCoinResult() → line 对象 {value, name, line, changing, symbol}
                                                         ↓
6 次掷爻 → state.lines[6] → buildBinary() → '101101' → getHexagram() → 显示卦象
                              ↓ (有动爻时)
                         transformBinary() → 变卦
```

### 关键状态机

```
welcome → (点击开始) → casting → (6 次掷爻完成) → result → (重新起卦) → welcome
```

`state.currentLine`: 0–5（对应第 1–6 爻），`advanceLine()` 递增到 6 时触发 `showResult()`。

## 已知坑与教训

1. **iOS AudioContext**: 创建后始终 `suspended`，`resume()` 是异步的，必须 `await`，否则静默无声。
2. **PWA 缓存**: Service Worker 版本号在 SW 代码内（`liuyao-vN`），每次改动需升级版本号否则旧缓存不更新。
3. **HapticEngine 局限性**: iOS switch hack 在 iOS 16+ 已基本失效，Android 用 `navigator.vibrate`。
4. **randomThrow 防重复**: 必须在入口守卫 `lines.length >= 6` 和 `lines.length !== currentLine`，并在执行期间禁用按钮，否则第 7 次掷爻会污染数据导致 `showResult` 崩溃。
5. **confirmLine 守卫**: 用 `!==` 而非 `>` 做等值判断，防止同一爻被重复锁定。

## 部署流程

```bash
cd D:\赛博六爻
git add index.html
git commit -m "<描述>"
git push
# GitHub Pages 1-2 分钟后自动生效
```

## 设计约束

- 所有资源内联（CSS/JS/SVG/Manifest/Service Worker），零外部依赖
- 零 HTTP 请求（manifest 和 icon 用 data: URI）
- 离线可用（SW cache-first）
- iOS 适配（safe-area-inset, viewport-fit=cover, apple-mobile-web-app-capable）
- 触觉反馈（iOS switch hack + Android vibrate）
- 真随机（`crypto.getRandomValues`，硬件熵源）
