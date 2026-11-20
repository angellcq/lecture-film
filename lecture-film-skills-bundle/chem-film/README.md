# chem-film · 化学讲解视频流水线

> lecture-film 流水线的化学分支。讲稿 + 配平校验 + 结构式 + 滴定曲线，一条命令出化学实验讲解片。

## 快速开始

```bash
# 1. 安装（放到 ~/.dsh/skills/ 下，或用 skill 安装器）
# 2. 确认环境 + 装 RDKit
pip install rdkit -i https://pypi.tuna.tsinghua.edu.cn/simple
node pipeline/doctor.mjs

# 3. 复制配置模板
cp pipeline/chem.demo.json pipeline/my-chem.json

# 4. 生成结构式 + 跑便宜工序
python chem_demo/gen_mol.py
node pipeline/make-lecture.mjs pipeline/my-chem.json --stages math,speech,sim,captions,stage

# 5. 全量出片
node pipeline/make-lecture.mjs pipeline/my-chem.json
```

## 它能做什么

- ✅ 化学实验讲解（滴定、反应、电解），装置图 + 曲线同步推进
- ✅ sympy 配平校验，元素矩阵求零空间，方程式对不对一眼知道
- ✅ RDKit 结构式生成，SMILES → SVG，页面淡入淡出
- ✅ numpy 数值仿真，pH-V 滴定曲线数值解
- ✅ 化学图元库：滴定管、锥形瓶、液滴、pH 曲线、化学计量点标线
- ✅ speech 符号映射：H₂SO₄→硫酸、⇌→可逆、↑→气体逸出，TTS 念得对
- ✅ 豆包 TTS 旁白 + 句级字幕 + BGM 闪避 + 烧字幕 + 剪映草稿
- ✅ TSL Textures 可选增强：晶体结构 / 分子模型 / 玻璃仪器质感等 3D 真实材质

## 十一道工序

```
math → speech → sim → voice → captions → stage → render | compose → mix → burn → export → qc
```

每道工序都有机器可判定的门禁，失败即停，产物按哈希缓存，可断点续跑。

## 配置模板（最简）

```json
{
  "name": "弱酸滴定强碱",
  "engine": "eac",
  "workdir": "chem_demo",
  "outDir": "pipeline/out/chem_titrate",
  "canvas": { "w": 1920, "h": 1080, "fps": 30 },
  "html": "film.html",
  "duration": 12,
  "sim": { "script": "sim/titrate.py", "out": "sim.json" },
  "math": [
    { "kind": "balance", "label": "醋酸中和氢氧化钠",
      "equation": "CH3COOH + NaOH -> CH3COONa + H2O",
      "expect": { "CH3COOH": 1, "NaOH": 1, "CH3COONa": 1, "H2O": 1 } }
  ],
  "speech": {
    "text": "script.txt",
    "map": {
      "H₂SO₄": "硫酸", "⇌": "可逆", "→": "生成",
      "↑": "气体逸出", "↓": "沉淀", "²": "的平方"
    }
  },
  "eac": { "projectId": "<你的项目ID>", "templateId": "09-bauhaus" }
}
```

## 示例

| 示例 | 说明 |
|---|---|
| `examples/chem.demo.json` | EAC 引擎 · 弱酸滴定强碱 · 12s 样片 |

## 化学图元库

`pipeline/kit/chem.js` 提供的 canvas 2D 图元：

- 滴定管（刻度、液面、活塞、液滴下落）
- 锥形瓶（液面上升、颜色变化）
- 液滴（滴定管口 → 锥形瓶的动画）
- pH 曲线坐标轴 + 化学计量点标线
- 方程式逐字显示

## 依赖

| 层级 | 需要什么 |
|---|---|
| 宿主 | DeepSeek Harness ≥ 0.2.1 |
| 插件 | EAC design world（渲染/导出） |
| 工具 | Node.js ≥ 22、FFmpeg ≥ 7、Python 3.12、Edge/Chrome |
| Python | pillow、numpy、sympy、**rdkit**（结构式 SVG） |

## 目录结构

```
chem-film/
├── SKILL.md              # Agent 加载的技能主体
├── README.md             # 本文件
├── package.json          # 技能元数据与触发词
├── LICENSE               # MIT 协议
├── examples/             # 配置模板
│   └── chem.demo.json
├── kit/                  # 图元与工具
│   └── README.md
└── references/           # 参考资料
    ├── balancing-equations.md
    └── titration-curves.md
```

## 相关技能

- **math-film**：数学讲解流水线（算式校验 + 几何板书）
- **physics-film**：物理讲解流水线（量纲校验 + 数值仿真 + 力学图元）
- **tsl-textures**：3D 程序化纹理增强（晶体/分子模型/玻璃质感）
- **lecture-film**：通用教学片流水线（学科未指定时的兜底）
