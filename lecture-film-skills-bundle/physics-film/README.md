# physics-film · 物理讲解视频流水线

> lecture-film 流水线的物理分支。讲稿 + 量纲校验 + 数值仿真 + 物理图元，一条命令出带受力分析和数据曲线的物理讲解片。

## 快速开始

```bash
# 1. 安装（放到 ~/.dsh/skills/ 下，或用 skill 安装器）
# 2. 确认环境
node pipeline/doctor.mjs

# 3. 复制配置模板
cp pipeline/physics.demo.json pipeline/my-physics.json

# 4. 先跑便宜的（算式 + 量纲 + 仿真 + 字幕 + 源体检）
node pipeline/make-lecture.mjs pipeline/my-physics.json --stages math,speech,sim,captions,stage

# 5. 全量出片
node pipeline/make-lecture.mjs pipeline/my-physics.json
```

## 它能做什么

- ✅ 力学/运动学讲解，受力分析 + v-t/s-t 曲线同步推进
- ✅ sympy 量纲校验，拦住"把时间加到位移上"这类低级错误
- ✅ numpy 数值仿真，画面上的数字和曲线都从 sim.json 读，不手打
- ✅ 物理图元库：力箭头、斜面、物块、坐标轴、折线图、读数卡
- ✅ speech 符号映射：θ→西塔、μ→缪、≈→约等于，TTS 念得对
- ✅ 豆包 TTS 旁白 + 句级字幕 + BGM 闪避 + 烧字幕 + 剪映草稿
- ✅ TSL Textures 可选增强：机械零件 / 地质断层 / 行星模型等 3D 真实材质

## 十一道工序

```
math → speech → sim → voice → captions → stage → render | compose → mix → burn → export → qc
```

每道工序都有机器可判定的门禁，失败即停，产物按哈希缓存，可断点续跑。

## 配置模板（最简）

```json
{
  "name": "斜面滑块匀加速",
  "engine": "eac",
  "workdir": "physics_demo",
  "outDir": "pipeline/out/physics_block",
  "canvas": { "w": 1920, "h": 1080, "fps": 30 },
  "html": "film.html",
  "duration": 12,
  "sim": { "script": "sim/block.py", "out": "sim.json" },
  "math": [
    { "kind": "expr", "label": "a 的精确值",
      "expr": "Rational(98,10)*(Rational(1,2) - Rational(1,10)*sqrt(3)/2)",
      "expect": "Rational(49,10) - Rational(49,100)*sqrt(3)" },
    { "kind": "dimensions", "label": "位移 ½at² 是长度",
      "formula": "a*t**2/2",
      "units": { "a": "m/s**2", "t": "s" },
      "values": { "a": 4.05, "t": 3 },
      "expect": "m" }
  ],
  "speech": {
    "text": "script.txt",
    "map": { "θ": "西塔", "μ": "缪", "≈": "约等于", "²": "的平方" }
  },
  "eac": { "projectId": "<你的项目ID>", "templateId": "09-bauhaus" }
}
```

## 示例

| 示例 | 说明 |
|---|---|
| `examples/physics.demo.json` | EAC 引擎 · 斜面滑块匀加速 · 12s 样片 |
| `examples/physics.spike.json` | 尖峰/碰撞示例（扩展用） |

## 物理图元库

`pipeline/kit/physics.js` 提供的 canvas 2D 图元：

- 力箭头（重力/支持力/摩擦力，带箭头和标注）
- 斜面 + 物块（角度 θ 可调）
- 坐标轴、角度弧、尺寸线
- 折线图（v-t、s-t、a-t 逐帧绘制）
- 读数卡（数值放大显示）

## 依赖

| 层级 | 需要什么 |
|---|---|
| 宿主 | DeepSeek Harness ≥ 0.2.1 |
| 插件 | EAC design world（渲染/导出） |
| 工具 | Node.js ≥ 22、FFmpeg ≥ 7、Python 3.12、Edge/Chrome |
| Python | pillow、numpy、sympy（量纲校验用 units） |

## 目录结构

```
physics-film/
├── SKILL.md              # Agent 加载的技能主体
├── README.md             # 本文件
├── package.json          # 技能元数据与触发词
├── LICENSE               # MIT 协议
├── examples/             # 配置模板
│   └── physics.demo.json
├── kit/                  # 图元与工具
│   └── README.md
└── references/           # 参考资料
    ├── dimensional-analysis.md
    └── simulation-patterns.md
```

## 相关技能

- **math-film**：数学讲解流水线（算式校验 + 几何板书）
- **chem-film**：化学讲解流水线（配平校验 + 结构式 + 滴定图元）
- **tsl-textures**：3D 程序化纹理增强（机械零件/地质/行星）
- **lecture-film**：通用教学片流水线（学科未指定时的兜底）
