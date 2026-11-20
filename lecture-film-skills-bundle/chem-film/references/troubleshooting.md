# 常见问题排查

## 渲染到一半报错「片段超出源素材时长」

**原因**：EAC 入轨时，片段时长比素材多几十毫秒。
**解决**：流水线已自动修（按素材实际时长收 60ms），如果还报错，检查 `duration` 是否精确。

## 字幕被拉伸变形

**原因**：ASS 文件的 PlayRes 和视频画幅不一致（竖屏 ASS 烧横屏）。
**解决**：`burn` 工序有门禁会拦住。按画幅重新生成 ASS，或在配置里设 `captions.playRes`。

## 听不到 BGM（静音一大片）

**原因**：音轨只含旁白，没混入 BGM。
**解决**：走 `mix` 工序。门禁会检查静音占比 >6% 直接停。

## 字幕一行都没有 / 全挤在开头

**原因**：词级时间戳单位是秒却被当成毫秒除了 1000。
**解决**：流水线内置单位探测（>36000 视为毫秒），正常会自动识别。如果识别错了，手动检查 words.json 的数值范围。

## 有 segments 配置却只渲出 10 秒

**原因**：配置里有 `segments` 但跑了单引擎 `render`，把拼接好的成片覆盖了。
**解决**：有 `segments` 时走 `compose` 工序，不要单独跑 `render`。流水线有门禁拦。

## 画中画区域没变化

**原因**：overlay 源视频是静态的，或时间没对齐。
**解决**：检查 `overlay.source` 路径和 `from` / `duration` 字段。

## TSL 页面首帧特别慢（5秒+）

**原因**：走了 WebGL2 后端，GLSL 编译慢。
**解决**：确认 `navigator.gpu` 存在，`renderer.backend.isWebGPUBackend === true`。Edge headless 不需要 flag 就有 WebGPU。

## TSL 同 t 两次画面不同

**原因**：动画型纹理吃了全局时钟。
**解决**：显式传 `time: uniform(0)` 并在 `renderAt(t)` 里赋值。

## 量纲校验报错「N is not defined」

**原因**：sympy 里 N 是数值求值函数，不是量纲符号。
**解决**：内部已改用 dimL/dimT/dimM/dimI/dimTh/dimN/dimCd，正常不会遇到。如果自己写表达式时撞上了，换个变量名。

## 化学计量点 pH 算成 0 或负数

**原因**：用了「过量碱」公式，但恰好中和时只剩盐，应该按盐水解算。
**解决**：用 Kb = Kw/Ka 计算盐水解的 pH。参考 `chem_demo/sim/titrate.py`。
