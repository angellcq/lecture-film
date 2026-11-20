# Kit · 数学图元与工具

数学流水线的画面图元主要靠片子自带的 HTML+SVG 代码（`film.html` / `film.js`），不是从 kit 目录 import 的。这里列出数学片常用的图元模式与复用方式。

## 几何板书图元（SVG 模式）

参考工程：`bauhaus_q20/film.html`、`squares35/`

| 图元 | 实现方式 | 要点 |
|---|---|---|
| 几何图形（圆/方/三角） | SVG `<circle>` `<rect>` `<polygon>` | 用 transform 做位移/旋转，保持数值精确 |
| 阴影面积填充 | SVG `<path>` + fill-opacity | 分段动画用 stroke-dasharray |
| 公式逐句高亮 | DOM `<div>` + 行内 `<span class="highlight">` | 按时间切换 class，CSS transition |
| 辅助线/辅助点 | SVG `<line>` `<circle r="3">` | 虚线用 stroke-dasharray |
| 标注文字 | SVG `<text>` 或 DOM `<div>` | 中文用 Microsoft YaHei，数字用等宽 |
| 逐笔绘制 | stroke-dasharray + stroke-dashoffset 动画 | 长度用 path.getTotalLength() |

## 复用模式

1. **复制最近的片子**当模板（如 `bauhaus_q20/`），改图形和讲稿
2. 共用的数学排版函数（如 `formatFraction`、`formatPi`）直接从别的片子拷
3. 风格色板在 `film.html` 的 CSS 变量里统一改

## TSL 3D 增强（可选）

需要立体几何切块、晶体模型等 3D 真实材质时，用 TSL Textures：

- 纹理选型：`marble`（大理石）、`wood`（木质教具）、`concrete`（石材）
- 接入方式：把 `html` 字段换成 three.js WebGPU 页面，保持 `renderAt(t)` 接口
- 详见 `tsl-textures` skill
