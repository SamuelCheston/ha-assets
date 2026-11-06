# ha-assets

Minecraft 竞技对局平台的 SVG 视觉素材库，供两个 Wails 桌面前端消费：

- **MatchCoreArena-Client**（竞技客户端）
- **SamuelClient**（游戏启动器）

纯矢量、无位图、无渐变、无阴影、无玻璃拟态。单一强调色驱动的扁平几何风：冷黑底上的浅色线条与色块，用图形、图标、状态色传达信息。

## 目录结构

```
ha-assets/
├── icons/      导航与动作图标（24 网格，16/24/32 三档）
├── states/     空状态与错误插画（240×160）
├── flow/       对局流阶段图形，Stepper 节点（24 网格，16/24/32 三档）
├── tiers/      段位徽章：未定级 + 5 级（160×160，纯装饰递进）
├── modes/      玩法规模图形与占位符（24 网格，16/24/32 三档）
├── patterns/   低透明度背景纹理（可平铺单元）
├── brand/      应用标记（160×160）
├── manifest.json  素材清单（id / 分组 / 尺寸 / 场景 / 可着色部位）
└── preview.html   接触表：浏览器直接打开，按组验收
```

## 命名规则

- 文件名 kebab-case：`career-history.svg`、`no-match-record.svg`。
- 尺寸变体以 `-16` / `-32` 后缀表示，无后缀即 24 基准档：`play.svg`（24）、`play-16.svg`、`play-32.svg`。仅 `icons/`、`flow/`、`modes/` 三组提供变体。
- `manifest.json` 中的 `id` 是稳定引用键：组前缀 + 文件名（如 `icon-play`、`state-no-rank`、`flow-idle`、`tier-3`、`mode-1v1`、`pattern-dots`、`brand-mca`）。

## 尺寸约定

| 组 | 画布 / viewBox | 说明 |
| --- | --- | --- |
| icons / flow / modes | `0 0 24 24`，16/24/32 三档 | 图标网格，内容留 2px 安全边距 |
| states | `0 0 240 160` | 空状态/错误插画，自带圆角面板底 |
| tiers / brand | `0 0 160 160` | 徽章与品牌标记 |
| patterns | 平铺单元（24×24 / 16×16 / 48×41.57） | `background-repeat: repeat` 使用 |

线宽：图标 2px，插画/徽章/标记 1.5px。全部文件带 `viewBox`，无内联 `style`，无外链字体。

## 着色

所有素材只由两类值驱动，便于主题与状态切换：

1. `currentColor` —— 主体线条，跟随宿主文本颜色。
2. `var(--ha-*, 回退值)` —— 强调色与语义色，与 `ha-ui/src/styles/tokens.css` 对齐；回退值保证素材在 `<img>` / 独立打开时仍可渲染。

| 变量 | 回退值 | 用途 |
| --- | --- | --- |
| `--ha-accent` | `#e94560` | 强调色（焦点、关键节点） |
| `--ha-ok` | `#4caf50` | 成功 / 已结算 |
| `--ha-warn` | `#f59e0b` | 警告 / 未实现 / 未登录 |
| `--ha-danger` | `#ef5350` | 错误 / 断连 |
| `--ha-info` | `#3b82f6` | 信息 |
| `--ha-base` / `--ha-panel` / `--ha-raised` | `#0a0c10` / `#10141a` / `#171d26` | 背景层级 |
| `--ha-edge` | `#232b37` | 边框 |
| `--ha-ink` / `--ha-ink-muted` / `--ha-ink-faint` | `#e9edf3` / `#96a1b3` / `#5c6879` | 线条/文字层级 |

每个素材的可着色部位见 `manifest.json` 的 `colorable` 字段。

`patterns/` 在根元素带 `color` 回退（`currentColor` 的来源），因此作为 `background-image` 引用时也有可见颜色；内联到页面后可被 CSS 覆盖。

## React 消费

推荐 `vite-plugin-svgr`（两个客户端均为 Wails + Vite/React）：

```tsx
// vite.config.ts: plugins: [svgr({ svgrOptions: { icon: true } })]
import PlayIcon from "@ha-assets/icons/play.svg?react";
import NetworkFailure from "@ha-assets/states/network-failure.svg?react";

// currentColor 跟随按钮文字色；尺寸用 width/height 或 CSS 控制
<button className="text-ink hover:text-accent">
  <PlayIcon width={24} height={24} />
</button>

// 状态切换示例：排队中换颜色
<QueueIcon className={queued ? "text-accent" : "text-ink-faint"} />

// 空状态插画：直接铺在面板里
<NetworkFailure width={240} height={160} />
```

不使用 SVGR 时，可直接 `fetch` 文件文本后内联（`dangerouslySetInnerHTML`），`currentColor` 与 `--ha-*` 同样生效。**不要**用 `<img>` 引用需要跟随主题色的图标——`<img>` 隔离了 CSS 上下文，只会渲染回退值（`patterns/` 除外，它为此场景设计了自带回退）。

背景纹理用法：

```css
.arena-bg {
  background-color: var(--ha-base);
  background-image: url("@ha-assets/patterns/geodesic.svg");
  background-repeat: repeat;
}
```

## 验收

浏览器直接打开 `preview.html`：深色接触表，按组排布，每个素材展示 id 与两种尺寸。
