# math-film · 数学题讲解视频流水线

> lecture-film 流水线的数学分支。写一份 JSON 配置，一条命令出成片 + 字幕 + 剪映草稿。

## 快速开始

```bash
# 1. 安装（放到 ~/.dsh/skills/ 下，或用 skill 安装器）
# 2. 确认环境
node pipeline/doctor.mjs

# 3. 复制配置模板，改 workdir / html / eac.projectId / duration
cp pipeline/lecture.q20.json pipeline/my-math.json

# 4. 先跑便宜的前几道（秒级）
node pipeline/make-lecture.mjs pipeline/my-math.json --stages math,captions,stage

# 5. 全量出片
node pipeline/make-lecture.mjs pipeline/my-math.json
```

## 它能做什么

- ✅ 几何/代数题板书讲解，公式逐句高亮
- ✅ sympy 算式校验，讲稿里的每条式子都对才往下走
- ✅ 豆包 TTS 旁白 + 词级时间戳，句级字幕精准对齐
- ✅ 两种画面引擎：EAC 手写 HTML+SVG / dsh-motion 场景代码
- ✅ BGM 侧链闪避混音，人声不被音乐盖
- ✅ 烧字幕 + 导出剪映草稿，可直接续剪
- ✅ TSL Textures 可选增强：立体几何切块、晶体模型等 3D 真实材质

## 十道工序

```
math → voice → captions → stage → render | compose → mix → burn → export → qc
```

每道工序都有机器可判定的门禁，失败即停，产物按哈希缓存，可断点续跑。

## 配置模板（最简）

```json
{
  "name": "第20题 · 阴影面积",
  "engine": "eac",
  "workdir": "bauhaus_q20",
  "outDir": "pipeline/out/q20",
  "canvas": { "w": 1920, "h": 1080, "fps": 30 },
  "html": "film.html",
  "duration": 226.42,
  "voice": { "words": "vo/narration.words.json", "wav": "vo/narration.mp3" },
  "bgm": { "wav": "bgm/scifi2_eq.wav", "gainDb": -13 },
  "captions": { "maxChars": 22, "font": "Microsoft YaHei" },
  "eac": { "projectId": "<你的项目ID>", "templateId": "09-bauhaus" },
  "math": [
    { "label": "扇形面积", "expr": "pi*6**2/8", "expect": "Rational(9,2)*pi" }
  ]
}
```

## 示例

| 示例 | 说明 |
|---|---|
| `examples/lecture.q20.json` | EAC 引擎 · 包豪斯几何风 · 阴影面积 |
| `examples/motion.9pi.json` | dsh-motion 引擎 · 9π 公式动效 |
| `examples/full.example.json` | 三合一：片头动效 + 板书主体 + 演算小窗 |

## 依赖

| 层级 | 需要什么 |
|---|---|
| 宿主 | DeepSeek Harness ≥ 0.2.1 |
| 插件 | EAC design world（渲染/导出）、@vlln/dsh-motion（可选第二引擎） |
| 工具 | Node.js ≥ 22、FFmpeg ≥ 7、Python 3.12、Edge/Chrome |
| Python | pillow、numpy、sympy |

## 目录结构

```
math-film/
├── SKILL.md              # Agent 加载的技能主体（含完整操作手册）
├── README.md             # 本文件，人类可读概览
├── package.json          # 技能元数据与触发词
├── LICENSE               # MIT 协议
├── examples/             # 配置模板与示例
│   ├── lecture.q20.json
│   ├── motion.9pi.json
│   └── full.example.json
├── kit/                  # 通用工具脚本
│   └── README.md
└── references/           # 参考资料
    ├── api-cheatsheet.md
    └── troubleshooting.md
```

## 相关技能

- **physics-film**：物理讲解流水线（量纲校验 + 数值仿真 + 力学图元）
- **chem-film**：化学讲解流水线（配平校验 + 结构式 + 滴定图元）
- **tsl-textures**：3D 程序化纹理增强（立体几何/晶体/材质感）
- **lecture-film**：通用教学片流水线（学科未指定时的兜底）
