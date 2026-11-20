# Kit · 化学图元库

化学图元由 `pipeline/kit/chem.js` 提供，是一个纯 Canvas 2D 工具库。

## 加载方式

```html
<script src="../pipeline/kit/chem.js"></script>
<script>
  const kit = window.ChemKit;
  // 在 renderAt(t) 里调用 kit.drawXXX(ctx, ...)
</script>
```

## 图元清单

| 函数 | 用途 | 参数概览 |
|---|---|---|
| `drawBurette(ctx, x, y, w, h, opts)` | 滴定管 | fillLevel（0-1）、scaleCount、hasStopcock |
| `drawFlask(ctx, x, y, w, h, opts)` | 锥形瓶 | fillLevel、fillColor、liquidWobble |
| `drawDrop(ctx, x, y, r, opts)` | 液滴 | falling（是否下落动画） |
| `drawPHCurve(ctx, x, y, w, h, data, opts)` | pH-V 曲线图 | data 为 [{V,pH}]、progress 0-1 |
| `drawEquivalenceLine(ctx, x, y, pH, V, opts)` | 化学计量点标线 | dashed、label |
| `drawEquation(ctx, x, y, equation, opts)` | 化学方程式 | 逐字显示进度、fontSize |

## 结构式

结构式不用 Canvas 画，用 RDKit 从 SMILES 生成 SVG：

```bash
python <workdir>/gen_mol.py    # 生成 assets/*.svg
```

页面里用 DOM `<img>` 叠加，把 `opacity` 接到 `renderAt(t)` 做淡入动画。

```js
const molImg = document.getElementById('acetate');
molImg.style.opacity = Math.min(1, (t - 5) / 1.5);  // 第5秒开始淡入，1.5秒完成
```

## 设计原则

- **数据驱动**：滴定曲线从 `sim.json` 读，液滴节奏由仿真数据决定
- **逐帧安全**：每个 draw 函数都是纯函数
- **风格统一**：酚酞粉 + 滴定蓝 + 白底

## 扩展图元

需要新图元（电解池、原电池、分子模型等）时：
1. 在 `chem.js` 里加 `drawXxx` 函数
2. 保持纯函数风格
3. 加 JSDoc 注释
