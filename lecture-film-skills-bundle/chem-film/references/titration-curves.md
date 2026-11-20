# 滴定曲线数值计算

## 弱酸滴定强碱（典型示例）

以醋酸（HAc）滴定 NaOH 为例：

- 酸：V₀ = 25.00 mL，c₀ = 0.1000 mol/L
- 碱：c = 0.1000 mol/L
- Ka = 1.8 × 10⁻⁵，pKa = 4.74

### 三个阶段

**1. 滴定前（V = 0）**

只有弱酸，按弱酸电离算：
pH = 0.5 × (pKa - log c) = 0.5 × (4.74 - log 0.1) = 2.87

**2. 化学计量点前（0 < V < V_eq）**

形成缓冲溶液，用 Henderson-Hasselbalch：
pH = pKa + log([Ac⁻]/[HAc])

**3. 化学计量点（V = V_eq）**

⚠️ **别用「过量碱」公式！** 恰好中和时只剩盐 NaAc，要按盐水解算：

Kb = Kw / Ka = 10⁻¹⁴ / (1.8×10⁻⁵) = 5.56×10⁻¹⁰

盐浓度 c' = c₀ × V₀ / (V₀ + V_eq) = 0.05 mol/L

[OH⁻] = √(Kb × c') = √(5.56×10⁻¹⁰ × 0.05) = 5.27×10⁻⁶

pOH = 5.28，pH = 14 - 5.28 = **8.72**

**4. 化学计量点后（V > V_eq）**

过量的 OH⁻ 抑制水解，直接按过量碱浓度算：
[OH⁻] = c × (V - V_eq) / (V₀ + V)

## 数值实现

参考 `chem_demo/sim/titrate.py`，核心逻辑：

```python
import numpy as np

Ka = 1.8e-5
Kw = 1e-14
V0 = 0.025    # L
c0 = 0.1      # mol/L
c  = 0.1      # mol/L
Veq = c0 * V0 / c  # 0.025 L

V_list = np.linspace(0, 0.050, 500)  # 0~50 mL
pH_list = []

for V in V_list:
    if V < Veq * 0.99:
        # 缓冲
        n_acid = c0*V0 - c*V
        n_salt = c*V
        pH = -np.log10(Ka) + np.log10(n_salt / n_acid)
    elif abs(V - Veq) < Veq * 0.01:
        # 化学计量点附近，用精确式
        c_salt = c0 * V0 / (V0 + V)
        Kb = Kw / Ka
        OH = np.sqrt(Kb * c_salt)
        pH = 14 + np.log10(OH)
    else:
        # 过量碱
        OH_excess = (c*V - c0*V0) / (V0 + V)
        pH = 14 + np.log10(OH_excess)
    pH_list.append(pH)
```

## 输出格式

```json
{
  "meta": {
    "title": "弱酸滴定强碱",
    "acid": "CH3COOH",
    "base": "NaOH",
    "Ka": 1.8e-5,
    "Veq_mL": 25.0,
    "pHeq": 8.72
  },
  "points": [
    { "V_mL": 0.0, "pH": 2.87 },
    { "V_mL": 5.0, "pH": 3.58 }
  ]
}
```

## 注意事项

1. **化学计量点 pH ≠ 7**：弱酸滴强碱 pH > 7，强酸滴弱碱 pH < 7
2. **pKa = 半等当点 pH**：V = Veq/2 时，pH = pKa，这是重要的考点
3. **突跃范围**：计量点前后 ±0.1% 的 pH 变化范围，指示剂选择依据
4. **曲线形状**：开始缓慢上升 → 突跃 → 再趋缓，S 形
