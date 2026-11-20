# 量纲校验速查（math dimensions 模式）

## 基本用法

```json
{
  "kind": "dimensions",
  "label": "位移 ½at² 是长度",
  "formula": "a*t**2/2",
  "units": { "a": "m/s**2", "t": "s" },
  "values": { "a": 4.05, "t": 3 },
  "expect": "m"
}
```

`formula` 是量纲表达式，`units` 给出各变量的单位，`expect` 是期望的结果量纲。

## 支持的单位

### 基本量纲

| 量纲 | 符号 | 单位示例 |
|---|---|---|
| 长度 | L | m、km、cm |
| 时间 | T | s、min、h |
| 质量 | M | kg、g |
| 电流 | I | A |
| 温度 | Th | K、°C |
| 物质的量 | N | mol |
| 发光强度 | Cd | cd |

### 派生单位

| 单位 | 展开 |
|---|---|
| N（牛顿） | kg·m/s² → M L T⁻² |
| J（焦耳） | N·m → M L² T⁻² |
| W（瓦特） | J/s → M L² T⁻³ |
| Pa（帕斯卡） | N/m² → M L⁻¹ T⁻² |
| V（伏特） | W/A → M L² T⁻³ I⁻¹ |
| Ω（欧姆） | V/A → M L² T⁻³ I⁻² |

## 写法注意

- 幂次用 `**`，如 `m/s**2`
- 乘法用 `*`，如 `kg*m/s**2`
- 所有单位都用小写字母开头（除了 `A` 安培、`K` 开尔文、`N` 牛顿等 SI 符号本身大写的）

## ⚠️ 坑：变量名不能叫 N 或 I

sympy 里 `N` 是数值求值函数、`I` 是虚数单位，直接用会崩。

内部实现已改用 `dimL / dimT / dimM / dimI / dimTh / dimN / dimCd` 作为量纲符号，正常使用不会遇到。如果你在 `formula` 里写了单字母变量叫 `n` 或 `i`，注意大写形式可能有冲突。

## 验证小技巧

先在 Python 里试：

```python
from sympy import symbols
from sympy.physics.units import *

a = 4.05 * meter / second**2
t = 3 * second
s = 0.5 * a * t**2
print(s)  # 18.225*meter
print(s.dimension)  # length
```
