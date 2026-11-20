# Kit · 物理图元库

物理图元由 `pipeline/kit/physics.js` 提供，是一个纯 Canvas 2D 工具库，不依赖 three.js 或其他框架。

## 加载方式

```html
<script src="../pipeline/kit/physics.js"></script>
<script>
  const kit = window.PhysicsKit;
  // 在 renderAt(t) 里调用 kit.drawXXX(ctx, ...)
</script>
```

## 图元清单

| 函数 | 用途 | 参数概览 |
|---|---|---|
| `drawArrow(ctx, x1, y1, x2, y2, opts)` | 力矢量箭头 | color、width、headSize、label |
| `drawIncline(ctx, cx, cy, angle, length, opts)` | 斜面 + 物块 | angle（弧度）、length、blockSize |
| `drawAxes(ctx, x, y, w, h, opts)` | x-y 坐标系 | xLabel、yLabel、gridStep |
| `drawBlock(ctx, x, y, w, h, opts)` | 物块（方形/小球） | mass、color、labelPos |
| `drawAngleArc(ctx, cx, cy, r, startAngle, endAngle, opts)` | 角度标注弧 | label、color |
| `drawDimensionLine(ctx, x1, y1, x2, y2, opts)` | 尺寸线 | label、offset |
| `drawLineChart(ctx, x, y, w, h, data, opts)` | 折线图（逐帧绘制） | data 为 [{t,v}]、progress 0-1 |
| `drawReadingCard(ctx, x, y, value, unit, opts)` | 数值读数卡 | value、unit、fontSize |

## 设计原则

- **数据驱动**：所有数值从 `sim.json` 读，不要在渲染代码里手算物理量
- **逐帧安全**：每个 draw 函数都是纯函数，给同样的参数出同样的像素
- **风格统一**：默认深蓝主色 + 橙色强调，与包豪斯模板协调

## 扩展图元

需要新图元时，建议：
1. 在 `physics.js` 里加 `drawXxx` 函数
2. 保持纯函数风格（参数 → 绘制）
3. 加 JSDoc 注释说明参数含义和单位
