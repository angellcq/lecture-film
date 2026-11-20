# 算式校验速查（math expr 模式）

## 基本用法

```json
"math": [
  { "label": "扇形面积", "expr": "pi*6**2/8", "expect": "Rational(9,2)*pi" }
]
```

`expr` 是 Python 表达式字符串，`expect` 是期望的 sympy 表达式字符串。两者用 sympy 化简后比较。

## 铁则：用 Rational，别用浮点

```
✅ "Rational(9,2)*pi"      # 精确
❌ "4.5*pi"               # 浮点，可能导致比较失败
❌ "9/2*pi"               # Python 里 9/2 = 4.5，还是浮点
```

## 常用符号

| 数学符号 | sympy 写法 |
|---|---|
| π | `pi` |
| √2 | `sqrt(2)` |
| a² | `a**2` |
| 分数 a/b | `Rational(a,b)` |
| sin/cos | `sin(x)` / `cos(x)` |
| 绝对值 | `Abs(x)` |
| 求和 | `Sum(f, (i, 1, n))` |
| 极限 | `limit(f, x, a)` |

## 常见误判排查

1. **两边都对但不等**：检查是否 `expand` / `simplify` 后才相等。流水线内部会调 `simplify`，但极个别情况需要手动 `expand`。
2. **顺序问题**：`a + b` 和 `b + a` 没问题（加法交换律）；但 `a - b` 和 `-b + a` sympy 会自动规范化。
3. **含 pi 的比较**：确保两边都用 `pi` 而不是数值近似。

## 写配置的小技巧

先在 Python 里试一下：

```python
from sympy import *
expr = sympify("pi*6**2/8")
expect = sympify("Rational(9,2)*pi")
print(simplify(expr - expect))  # 输出 0 就是对的
```
