---
name: chem-film
description: 用本机 lecture-film 流水线出化学讲解视频。当用户说"做化学题讲解视频 / 化学实验动画 / 酸碱滴定 / 化学方程式配平 / 分子结构 / 化学教学片"时使用。配平校验 + 结构式生成 + 豆包TTS + 化学图元库（滴定管/锥形瓶/pH曲线）+ EAC 逐帧渲染 + 混音 + 烧字幕 + 剪映草稿导出。含十一道工序、门禁、实测耗时与踩坑清单。
---

# chem-film · 化学讲解视频流水线

一句话：**讲稿 + 配平校验 + 结构式 + 滴定曲线，一条命令出化学实验讲解片**，每道工序都有机器可判定的门禁。

## 触发场景

- 做一道化学题讲解视频
- 化学实验动画 / 演示
- 酸碱滴定、中和反应
- 化学方程式配平
- 分子结构、有机化学讲解
- 化学教学片

## 先做什么

1. 跑环境自检：`node pipeline/doctor.mjs`（不全绿就先补依赖，别硬跑）
2. 确认 RDKit 已装：`pip install rdkit`（结构式 SVG 生成用）
3. 以 `pipeline/chem.demo.json` 为模板复制一份新配置
4. 改 `workdir` / `html` / `eac.projectId` / `duration` / `sim` / `math`

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
| **math** | sympy 核对：算式 expr + 配平 balance | 任一不符即停；方程式不可配平即停 |
| **speech** | 符号→中文读法（H₂SO₄→硫酸、⇌→可逆、↑→气体逸出） | 仍有危险符号直接失败并列出 |
| **sim** | 数值仿真（numpy 解滴定曲线等）→ sim.json | 脚本执行失败即停 |
| voice | 豆包 TTS 旁白 + 词级时间戳 | 没有 words.json 且没配命令 |
| captions | 切句 → SRT + ASS | <3 行；PlayRes ≠ 画幅 |
| stage | 画面源体检 | 无 renderAt；场景含 Math.random/Date.now |
| render / compose | EAC 逐帧渲染 或 多段拼接 | 时长偏差 >0.25s |
| mix | 旁白 + BGM 侧链闪避 | 静音 >6%；人声底噪 <6dB |
| burn | 烧字幕 | ASS PlayRes ≠ 画幅 |
| export | 剪映草稿 + SRT + LUT | — |
| qc | 终检 | 无视频、无音轨、静音 |

## 化学专属配置

### 1. 配平校验（math.balance）

```json
"math": [
  { "kind": "balance", "label": "醋酸中和氢氧化钠",
    "equation": "CH3COOH + NaOH -> CH3COONa + H2O",
    "expect": { "CH3COOH": 1, "NaOH": 1, "CH3COONa": 1, "H2O": 1 } }
]
```

- 元素矩阵求零空间 → 归一到最小整数系数
- 不写 `expect` 则只判定"可配平"，不对比具体系数
- ⚠️ 化学计量点别用「过量碱」公式，恰好中和时只剩盐，要按水解算（Kb = Kw/Ka）

### 2. 数值仿真（sim）

```json
"sim": { "script": "sim/titrate.py", "out": "sim.json" }
```

- 滴定曲线数值解（pH-V 数据）
- 画面页从 sim.json 读取，逐帧绘制曲线
- 参考脚本：`chem_demo/sim/titrate.py`（弱酸滴定强碱，V_eq = 25.00 mL、pH_eq = 8.722）

### 3. 结构式生成

```bash
python <workdir>/gen_mol.py    # SMILES → assets/*.svg
```

- RDKit 从 SMILES 生成 SVG 结构式
- 页面用 DOM `<img>` 叠加，opacity 接到 `renderAt(t)` 做淡入
- ⚠️ 结构式淡入要自己把 opacity 动画写在 renderAt 里，否则不会随时间出现

### 4. 化学图元库

`pipeline/kit/chem.js` 提供 canvas 2D 图元：

| 图元 | 用途 |
|---|---|
| 滴定管 | 带刻度、液面、活塞，液滴下落动画 |
| 锥形瓶 | 液面上升、颜色变化（酚酞变色） |
| 液滴 | 从滴定管滴入锥形瓶的动画 |
| pH 曲线坐标轴 | pH-V 坐标系，化学计量点标线 |
| 化学计量点 | 虚线标注 V_eq 和 pH_eq |
| 方程式 | 化学方程式逐字显示 |

### 5. speech 符号映射（化学常用）

```json
"speech": {
  "text": "script.txt",
  "map": {
    "H₂SO₄": "硫酸", "HCl": "盐酸", "NaOH": "氢氧化钠",
    "⇌": "可逆", "→": "生成", "↑": "气体逸出", "↓": "沉淀",
    "²": "的平方", "⁺": "正离子", "⁻": "负离子",
    "≈": "约等于", "Δ": "加热"
  }
}
```

## 画面风格

白底 + 酚酞粉 + 滴定蓝。左侧装置图（滴定管+锥形瓶），右侧 pH 曲线逐帧绘制，结构式淡入淡出，方程式在底部。

参考工程：`chem_demo/`（12 秒弱酸滴定强碱样片）

## 额外依赖

| 依赖 | 用途 | 装法 |
|---|---|---|
| RDKit | 化学结构式 SVG 生成 | `pip install rdkit -i https://pypi.tuna.tsinghua.edu.cn/simple` |
| numpy | 滴定曲线数值解 | 已装 |
| sympy | 配平校验 | 已装 |

## 实测性能

12 秒 / 1080p30 / 360 帧 → 渲染 **46 秒**（0.085 s/帧），EAC 报告 `deterministic=true, exactPixels=true, meanDifference=0, productionReady=true`。

画面验证：结构式白底占比 0% → 44.5%（醋酸出现）→ 84.0%（酚酞出现）；pH 曲线蓝色像素 0.09% → 0.79% 渐进绘制。

## 可选增强：TSL Textures 程序化纹理（3D 真实材质）

**什么时候想到它**：化学题里有 3D 分子/晶体/实验装置模型、且对「材质感」有要求时 —— 比如：
- 晶体结构、晶格模型（岩盐、金刚石、金属晶体）
- 分子模型、球棍模型（原子球用不同材质）
- 玻璃仪器质感（烧杯、烧瓶、试管的玻璃反光）
- 溶液/液面效果（`waves` / `caustics`）
- 沉淀、结晶过程的材质表现
- 地质/矿物标本（大理石、石英、黏土）

默认 2D 装置图 + 结构式 + 曲线图用 SVG/Canvas 2D 就够了；只有需要**真实材质感的 3D 物体**时才加 TSL。

### 怎么接进流水线

跟普通 `renderAt(t)` 页面完全一样，不需要改引擎，直接换 `html` 字段：

```json
{
  "workdir": "chem_crystal",
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

### 化学常用纹理选型

| 场景 | 纹理 |
|---|---|
| 矿物/晶体表面 | `marble`、`karstRock`、`concrete`、`quartz`（库中找对应） |
| 玻璃仪器 | 配合 `MeshPhysicalNodeMaterial` 的 transmission + 程序化表面瑕疵 |
| 金属原子球 | `scratchedMetal`、`machinedMetal`（PBR 金属度 1.0） |
| 溶液/液面波动 | `waves`、`caustics`、`waterDrops` |
| 沉淀/颗粒质感 | `roughClay`、`sand`、`camouflage`（颗粒感） |
| 分子模型背景 | `stars`、`staticNoise`（微噪点做气氛底） |

### 关键注意事项

- **确定性是命门**：动画型纹理（`waves`/`caustics`/`waterDrops` 等）默认吃全局时钟，必须显式传 `time: uniform(...)` 并在 `renderAt` 里赋值，否则同 t 两次画面不同，`stage` 门禁直接警告。
- `seed` 参数决定花纹，同 seed 同参数 → 逐像素相同。
- WebGPU 后端实测 54–62 ms/帧（1080p），60 秒片约 100–110 秒渲完，比 2D Canvas 慢约 5 倍。
- **化学数据仍然从 sim.json 读**，TSL 只管表面材质，不要把仿真逻辑写进着色器里。
- **结构式仍然用 RDKit 生成 SVG**，TSL 不负责分子结构表达。
- 先跑 `--stages stage,render --short 3` 验片，确认有画面、不黑、不白，再全量渲。
- 我看不到画面，靠像素统计验：`meanLuma` 在 35–120 之间（低于 25 是黑片，高于 200 是一片白）。

### 参考

完整说明见 `tsl-textures` skill 和 `tsl-webgpu-test/` 目录。

## 铁律（踩出来的）

- 一条片子**只选一个画面引擎**；混用走 `compose`
- 配置里有 `segments` 时禁止再跑单引擎 `render`
- 用了 `segments` 又加片头时，字幕要设 `captions.offsetSec`
- **结构式必须先跑 gen_mol.py**，否则页面里图加载不出来
- **滴定曲线从 sim.json 读**，不要手打点
- speech 工序必须通过，TTS 不会念 H₂SO₄ / ⇌ / ↑ 这些
- 化学计量点 pH 要按盐水解算，别拿「过量碱」公式硬套
- 渲染前先跑 `--stages math,sim,stage` 快检

## 产物位置

`pipeline/out/<配置名>/` 下：
`sim.json / captions.srt / captions.ass / film.mp4 / film-subbed.mp4 / export/剪映草稿 / qc.json`

交付整理到 `交付_<题目名>/` 目录。
