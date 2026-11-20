---
name: math-film
description: 用本机 lecture-film 流水线出数学题讲解视频。当用户说"做数学题讲解视频 / 几何题讲解 / 阴影面积 / 把这道数学题做成视频 / 出一版数学讲解片 / 数学教学片"时使用。算式 sympy 校验 + 豆包TTS + 句级字幕 + EAC/dsh-motion 双画面引擎 + BGM 闪避混音 + 烧字幕 + 剪映草稿导出。含十道工序、门禁、实测耗时与踩坑清单。
---

# math-film · 数学题讲解视频流水线

一句话：**写一份 JSON 配置，一条命令出成片 + 字幕 + 剪映草稿**，每道工序都有机器可判定的门禁。

## 触发场景

- 做一道数学题讲解视频
- 把这道几何题做成视频
- 阴影面积讲解片
- 出一版数学教学片
- 改一条已有的数学讲解视频

## 先做什么

1. 跑环境自检：`node pipeline/doctor.mjs`（不全绿就先补依赖，别硬跑）
2. 选一份配置当模板（`pipeline/` 下）：
   - `lecture.q20.json` — 画面走 **EAC**（手写 HTML+SVG，适合复杂板书节奏）
   - `motion.9pi.json` — 画面走 **dsh-motion**（场景代码，适合公式/数字反复改）
   - `full.example.json` — 三合一（片头动效 + 主体 + 演算小窗），最接近成品交付
   - `overlay.example.json` — 画中画（主体 + 右上角演算小窗）
   - `hybrid.example.json` — 两种引擎各渲一段，EAC 时间线拼接
3. 新建一份自己的配置（复制最近的模板改 `workdir` / `html` 或 `motion.dir` / `eac.projectId` / 时长）

## 命令

```bash
node pipeline/doctor.mjs                                        # 环境体检
node pipeline/make-lecture.mjs pipeline/<你的配置>.json          # 全流程
node pipeline/make-lecture.mjs pipeline/<配置>.json --stages math,captions,stage   # 只跑前几道（秒级）
node pipeline/make-lecture.mjs pipeline/<配置>.json --stages render,qc             # 重渲+质检
node pipeline/make-lecture.mjs pipeline/<配置>.json --short 20                      # 20 秒样张快检
--force  忽略缓存重跑
```

## 十道工序（顺序固定，失败即停，可断点续跑）

`math → voice → captions → stage → render | compose → mix → burn → export → qc`

| 工序 | 干什么 | 门禁 |
|---|---|---|
| math | sympy 逐条核对讲稿里的算式 | 任一不符即停（注意写 `Rational(6**2,2)` 而不是 `6**2/2`） |
| voice | 旁白与词级时间戳（有则复用） | 没有 words.json 且没配命令 |
| captions | 切句 → SRT + ASS | <3 行；PlayRes ≠ 画幅；时间戳单位自动识别（秒/毫秒） |
| stage | 画面源体检 | 无 renderAt/__seek；场景含 Math.random/Date.now |
| render | EAC render-template 或 dsh-motion CLI | 时长偏差 >0.25s；无视频流 |
| compose | 分片渲染 + EAC 时间线拼接（可带 overlay 画中画） | 每段裁切误差 >0.3s；拼接总长偏差 >0.5s；画中画区域无变化/误伤画面 |
| mix | 旁白 + BGM 侧链闪避 | 静音 >6%（BGM 没铺上）；人声高出底噪 <6dB（抢话） |
| burn | 烧字幕 | ASS PlayRes ≠ 画幅（竖屏 ASS 会拉伸） |
| export | 成片导入项目 → 入轨 → EAC export | 素材未导入；入轨时长超源素材 |
| qc | 终稿严格 / 中间产物只记录 | 无视频、无音轨、静音 |

## 数学专属配置

### math 校验（expr 模式）

```json
"math": [
  { "label": "扇形面积", "expr": "pi*6**2/8", "expect": "Rational(9,2)*pi" },
  { "label": "阴影总面积", "expr": "2*(Rational(9,2)*pi - Rational(36,4))", "expect": "9*pi - 18" }
]
```

- 分数必须用 `Rational(a,b)`，别用 `a/b`（浮点会误判）
- π 写 `pi`，√ 写 `sqrt`

### 画面风格

- 包豪斯几何风（蓝/黄/红/米白），手写 HTML + SVG 板书
- 公式逐句高亮，几何图形逐笔绘制
- 参考工程：`bauhaus_q20/`、`squares35/`、`bauhaus_q21/`

## 可选增强：TSL Textures 程序化纹理（3D 真实材质）

**什么时候想到它**：题目里有立体几何、3D 几何体、且对「材质感」有要求时 —— 比如：
- 立体几何切块（大理石石材块、木质教具）
- 晶体结构、多面体模型
- 机械零件、齿轮、转轴
- 标题卡/板书需要纸纹、网格底、噪点底

默认板书用 SVG/DOM 就够了；只有需要**真实材质感**时才加 TSL。

### 怎么接进流水线

跟普通 `renderAt(t)` 页面完全一样，不需要改引擎，直接换 `html` 字段：

```json
{
  "workdir": "tsl-geometry",
  "html": "film.html",        // 页面里用 three.js WebGPU + TSL Textures
  "duration": 60,
  "canvas": { "w": 1920, "h": 1080, "fps": 30 },
  "eac": { "projectId": "...", "templateId": "09-bauhaus", "scale": 1 }
}
```

页面骨架四个不可省的点：
1. 用 `three/webgpu`（不是 `three.module.js`）
2. `await renderer.init()` 之后再 render
3. 材质用 `MeshStandardNodeMaterial` + `colorNode` 接纹理函数
4. `window.renderAt = async (t) => { ... }` 逐帧契约

### 数学常用纹理选型

| 场景 | 纹理 |
|---|---|
| 大理石/石材切块 | `marble`、`concrete`、`karstRock` |
| 木质教具、量角器 | `wood`、`processedWood` |
| 陶土、石膏模型 | `roughClay`、`crumpledFabric` |
| 网格纸/等值线底 | `grid`、`isolines`、`isolayers`（注意屏幕空间） |
| 标题卡纸纹 | 直接用 `bake-2d.html` 摊成 1920×1080 静态图，省 3D 开销 |

### 关键注意事项

- **确定性是命门**：动画型纹理默认吃全局时钟，必须显式传 `time: uniform(...)` 并在 `renderAt` 里赋值，否则同 t 两次画面不同，`stage` 门禁直接警告。
- `seed` 参数决定花纹，同 seed 同参数 → 逐像素相同。
- WebGPU 后端实测 54–62 ms/帧（1080p），60 秒片约 100–110 秒渲完，比 EAC HTML 板书慢约 5 倍。
- **先跑 `--stages stage,render --short 3` 验片**，确认有画面、不黑、不白，再全量渲。
- 我看不到画面，靠像素统计验：`meanLuma` 在 35–120 之间（低于 25 是黑片，高于 200 是一片白）。

### 参考

完整说明见 `tsl-textures` skill 和 `tsl-webgpu-test/` 目录。

## 铁律（踩出来的）

- 一条片子**只选一个画面引擎**；混用只能走 `compose`（各自渲 mp4 → EAC 时间线拼），不要在 dsh-motion 里贴视频帧
- 配置里有 `segments` 时**禁止**再跑单引擎 `render`（会覆盖成片）；`burn/export/qc` 会自动沿用 `pipeline/out/<name>/film.mp4`
- 用了 `segments` 又加片头时，字幕要设 `captions.offsetSec`（= 片头时长）
- 竖屏 ASS（PlayRes 1080×1920）不能烧横屏片子
- 渲染 226 秒片约 13 分钟（0.117s/帧），别用短超时调它

## 产物位置

`pipeline/out/<配置名>/` 下：
`captions.srt / captions.ass / film.mp4 / film-subbed.mp4 / mix.wav / export/剪映草稿 / qc.json / report.json`

交付整理到 `交付_<题目名>/` 目录。
