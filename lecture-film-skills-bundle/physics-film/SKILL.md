---
name: physics-film
description: 用本机 lecture-film 流水线出物理讲解视频。当用户说"做物理题讲解视频 / 力学动画 / 斜面滑块 / 自由落体 / 物理实验演示 / 物理教学片"时使用。量纲校验 + 数值仿真 + 豆包TTS + 物理图元库（力箭头/斜面/坐标轴/折线图）+ EAC 逐帧渲染 + 混音 + 烧字幕 + 剪映草稿导出。含十道工序、门禁、实测耗时与踩坑清单。
---

# physics-film · 物理讲解视频流水线

一句话：**讲稿 + 仿真数据 + 物理图元，一条命令出带受力分析和数据曲线的物理讲解片**，每道工序都有机器可判定的门禁。

## 触发场景

- 做一道物理题讲解视频
- 物理实验动画 / 演示
- 斜面滑块、自由落体、平抛运动
- 力学、运动学讲解片
- 物理数据可视化动画

## 先做什么

1. 跑环境自检：`node pipeline/doctor.mjs`（不全绿就先补依赖，别硬跑）
2. 以 `pipeline/physics.demo.json` 为模板复制一份新配置
3. 改 `workdir` / `html` / `eac.projectId` / `duration` / `sim` / `math`

## 命令

```bash
node pipeline/doctor.mjs                                        # 环境体检
node pipeline/make-lecture.mjs pipeline/<你的配置>.json          # 全流程
node pipeline/make-lecture.mjs pipeline/<配置>.json --stages math,speech,sim,captions,stage   # 便宜的前几道
node pipeline/make-lecture.mjs pipeline/<配置>.json --short 20   # 20 秒样张快检
--force  忽略缓存重跑
```

## 十一道工序（顺序固定，失败即停，可断点续跑）

`math → speech → sim → voice → captions → stage → render | compose → mix → burn → export → qc`

| 工序 | 干什么 | 门禁 |
|---|---|---|
| **math** | sympy 核对：算式 expr + 量纲 dimensions | 任一不符即停；量纲不匹配即停 |
| **speech** | 符号→中文读法（θ→西塔、μ→缪、≈→约等于） | 仍有危险符号直接失败并列出 |
| **sim** | 数值仿真（numpy）出 (t, v, s, a) 等序列 | 脚本执行失败即停 |
| voice | 豆包 TTS 旁白 + 词级时间戳 | 没有 words.json 且没配命令 |
| captions | 切句 → SRT + ASS | <3 行；PlayRes ≠ 画幅 |
| stage | 画面源体检 | 无 renderAt；场景含 Math.random/Date.now |
| render / compose | EAC 逐帧渲染 或 多段拼接 | 时长偏差 >0.25s |
| mix | 旁白 + BGM 侧链闪避 | 静音 >6%；人声底噪 <6dB |
| burn | 烧字幕 | ASS PlayRes ≠ 画幅 |
| export | 剪映草稿 + SRT + LUT | — |
| qc | 终检 | 无视频、无音轨、静音 |

## 物理专属配置

### 1. 量纲校验（math.dimensions）

```json
"math": [
  { "kind": "dimensions", "label": "位移 ½at² 是长度",
    "formula": "a*t**2/2",
    "units": { "a": "m/s**2", "t": "s" },
    "values": { "a": 4.05, "t": 3 },
    "expect": "m" }
]
```

- 派生单位 N/J/W/Pa/V/Ω 自动展开成 base 维度
- ⚠️ 量纲符号不能叫 `N` 或 `I`：sympy 里 N 是求值函数、I 是虚数单位，内部已改用 dimL/dimT/dimM/dimI/dimTh/dimN/dimCd

### 2. 数值仿真（sim）

```json
"sim": { "script": "sim/block.py", "out": "sim.json" }
```

- 仿真脚本输出 `sim.json`，画面页 `fetch` 后按帧读取，**数字不手打**
- 参考脚本：`physics_demo/sim/block.py`（斜面滑块 a = g(sinθ−μcosθ)）

### 3. 物理图元库

`pipeline/kit/physics.js` 提供 canvas 2D 图元：

| 图元 | 用途 |
|---|---|
| 力箭头 | 重力/支持力/摩擦力矢量，带箭头和标注 |
| 坐标轴 | x-y 坐标系，可缩放 |
| 斜面 | 带角度 θ，可放物块 |
| 物块 | 方形/小球，显示质量标注 |
| 角度弧 | 标注角度（θ、α） |
| 尺寸线 | 长度/高度标注 |
| 折线图 | v-t、s-t、a-t 曲线，逐帧绘制 |
| 读数卡 | 数值放大显示（速度、位移、加速度） |

### 4. speech 符号映射（物理常用）

```json
"speech": {
  "text": "script.txt",
  "map": {
    "θ": "西塔", "μ": "缪", "≈": "约等于",
    "²": "的平方", "½": "二分之一",
    "·": "乘以", "−": "减去",
    "Δ": "德尔塔", "∑": "西格玛"
  }
}
```

## 画面风格

白底 + 深蓝主色 + 橙色强调。左侧受力分析图，右侧 v-t / s-t 曲线同步推进，数值读数卡实时更新。

参考工程：`physics_demo/`（12 秒斜面滑块样片）、`physics_film/`

## 实测性能

12 秒 / 1080p30 / 360 帧 → 渲染 **46 秒**（0.085 s/帧），EAC 报告 `deterministic=true, exactPixels=true, meanDifference=0, productionReady=true`。

## 可选增强：TSL Textures 程序化纹理（3D 真实材质）

**什么时候想到它**：物理题里有 3D 实物模型、且对「材质感」有要求时 —— 比如：
- 机械零件、齿轮、滑轮、转轴（金属/塑料质感）
- 地质断层模型、岩石样本（大理石/花岗岩层理）
- 晶体结构、晶格模型
- 水波、液面、光斑效果（`waves` / `caustics`）
- 行星、天体模型（`planet` / `gasGiant` / `stars`）

默认 2D 示意图 + 折线图用 SVG/Canvas 2D 就够了；只有需要**真实材质感的 3D 物体**时才加 TSL。

### 怎么接进流水线

跟普通 `renderAt(t)` 页面完全一样，不需要改引擎，直接换 `html` 字段：

```json
{
  "workdir": "physics_gears",
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

### 物理常用纹理选型

| 场景 | 纹理 |
|---|---|
| 金属零件、齿轮、转轴 | `machinedMetal`、`scratchedMetal`、`brushedAluminum`（库中找对应） |
| 岩石、地质断层 | `marble`、`concrete`、`karstRock`、`sandstone` |
| 木质教具、斜面 | `wood`、`processedWood` |
| 水波、液面、光斑 | `waves`、`caustics`、`waterDrops` |
| 行星、天体 | `planet`、`gasGiant`、`stars`、`photosphere` |
| 数据图底纹 | `grid`、`isolines`（注意屏幕空间） |

### 关键注意事项

- **确定性是命门**：动画型纹理（`waves`/`caustics`/`waterDrops` 等）默认吃全局时钟，必须显式传 `time: uniform(...)` 并在 `renderAt` 里赋值，否则同 t 两次画面不同，`stage` 门禁直接警告。
- `seed` 参数决定花纹，同 seed 同参数 → 逐像素相同。
- WebGPU 后端实测 54–62 ms/帧（1080p），60 秒片约 100–110 秒渲完，比 2D Canvas 慢约 5 倍。
- **物理数据仍然从 sim.json 读**，TSL 只管表面材质，不要把仿真逻辑写进着色器里。
- 先跑 `--stages stage,render --short 3` 验片，确认有画面、不黑、不白，再全量渲。
- 我看不到画面，靠像素统计验：`meanLuma` 在 35–120 之间（低于 25 是黑片，高于 200 是一片白）。

### 参考

完整说明见 `tsl-textures` skill 和 `tsl-webgpu-test/` 目录。

## 铁律（踩出来的）

- 一条片子**只选一个画面引擎**；混用走 `compose`
- 配置里有 `segments` 时禁止再跑单引擎 `render`
- 用了 `segments` 又加片头时，字幕要设 `captions.offsetSec`
- **仿真数据是画面的唯一真相** —— 数字和曲线都从 sim.json 读，不要手写
- speech 工序必须通过，TTS 不会念 θ/μ/≈ 这些符号
- 渲染前先跑 `--stages math,sim,stage` 快检，别渲到一半才发现量纲错了

## 产物位置

`pipeline/out/<配置名>/` 下：
`sim.json / captions.srt / captions.ass / film.mp4 / film-subbed.mp4 / export/剪映草稿 / qc.json`

交付整理到 `交付_<题目名>/` 目录。
