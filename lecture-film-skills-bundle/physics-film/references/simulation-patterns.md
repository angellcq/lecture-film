# 数值仿真模式（sim 工序）

## 基本用法

```json
"sim": {
  "script": "sim/block.py",
  "out": "sim.json"
}
```

`script` 是相对于 `workdir` 的 Python 脚本路径，`out` 是输出的 JSON 文件名（也在 `workdir` 下）。

## sim.json 约定

输出格式自由，但推荐：

```json
{
  "meta": {
    "title": "斜面滑块匀加速",
    "params": { "theta": 0.5236, "mu": 0.1, "g": 9.8 }
  },
  "frames": [
    { "t": 0.0, "x": 0.0, "v": 0.0, "a": 4.051 },
    { "t": 0.1, "x": 0.020, "v": 0.405, "a": 4.051 }
  ]
}
```

- `t` 是时间（秒），建议从 0 开始，步长均匀
- 每帧的物理量都列出来，页面按帧插值或直接取最近帧
- `meta.params` 存原始参数，方便回溯

## 页面读取

```js
let simData = null;

async function loadSim() {
  const res = await fetch('sim.json');
  simData = await res.json();
}

function getFrame(t) {
  // 二分查找或线性插值
  const frames = simData.frames;
  // ...
}
```

## 典型仿真模式

### 1. 匀加速运动（运动学）

```python
import json

g = 9.8
theta = 30 * 3.14159 / 180
mu = 0.1
a = g * (math.sin(theta) - mu * math.cos(theta))

frames = []
for i in range(120):  # 12秒 × 10fps
    t = i * 0.1
    v = a * t
    x = 0.5 * a * t * t
    frames.append({"t": round(t,3), "x": round(x,4), "v": round(v,4), "a": round(a,4)})

with open("sim.json", "w") as f:
    json.dump({"meta": {...}, "frames": frames}, f)
```

### 2. 振动 / 波动

```python
import numpy as np

t = np.linspace(0, 10, 300)
omega = 2 * np.pi * 1.5  # 1.5 Hz
x = 0.1 * np.cos(omega * t)  # 10cm 振幅
v = -0.1 * omega * np.sin(omega * t)
```

### 3. 多体 / 碰撞

逐帧递推，存每帧位置和速度。碰撞检测用简单 AABB 或圆碰撞。

## 设计原则

1. **仿真和渲染分离**：仿真算好所有数据存 JSON，渲染只负责画，不做计算
2. **足够密**：数据点至少 10fps，画面插值才顺滑
3. **可复现**：固定随机种子（如果有随机因素），确保每次运行 sim.json 一样
4. **单位明确**：全部 SI 单位（m, s, kg, N, J, W...），页面上再换算显示
