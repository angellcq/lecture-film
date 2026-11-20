# 流水线 API 速查

## 命令总览

```bash
node pipeline/doctor.mjs                                    # 环境自检
node pipeline/make-lecture.mjs <config.json>               # 全流程
node pipeline/make-lecture.mjs <config.json> --stages a,b,c # 指定工序
node pipeline/make-lecture.mjs <config.json> --short 20     # 20 秒样张
node pipeline/make-lecture.mjs <config.json> --force        # 忽略缓存重跑
```

## 工序列表

| 工序 | 产物 | 耗时 |
|---|---|---|
| math | 校验通过 / 失败报告 | 秒级 |
| voice | narration.mp3 + words.json | 取决于 TTS 速度 |
| captions | captions.srt + captions.ass | 秒级 |
| stage | 源体检通过 / 警告 | 秒级 |
| render | film.mp4 | 0.1s/帧（EAC），0.06s/帧（简单页面） |
| compose | film.mp4（多段拼接） | 每段裁切 + 时间线渲染 |
| mix | mix.wav | 秒级 |
| burn | film-subbed.mp4 | 全片重编码，约 50s/226s |
| export | 剪映草稿 / SRT / LUT | 秒级 |
| qc | qc.json | 秒级 |

## 配置字段速查

```jsonc
{
  "name": "题目名",
  "engine": "eac",              // eac 或 dsh-motion
  "workdir": "项目目录",
  "outDir": "输出目录",
  "canvas": { "w": 1920, "h": 1080, "fps": 30 },
  "encode": { "preset": "veryfast", "crf": 20 },
  "html": "film.html",          // eac 引擎用
  "duration": 226.42,
  "voice": { "words": "vo/xxx.json", "wav": "vo/xxx.mp3" },
  "bgm": { "wav": "bgm/xxx.wav", "gainDb": -13 },
  "captions": { "maxChars": 22, "font": "Microsoft YaHei", "offsetSec": 0 },
  "eac": { "projectId": "...", "templateId": "09-bauhaus" },
  "motion": { "dir": "motion" }, // dsh-motion 引擎用
  "segments": [ /* 多段拼接 */ ],
  "math": [ /* 算式校验 */ ],
  "sim": { "script": "sim/xxx.py", "out": "sim.json" }
}
```

## 产物清单

```
pipeline/out/<name>/
├── film.mp4              # 成片（有音轨）
├── film-subbed.mp4       # 烧字幕版
├── mix.wav               # 混音后音轨
├── captions.srt          # 句级字幕
├── captions.ass          # ASS 字幕（含样式）
├── sim.json              # 仿真数据（理化才有）
├── speech.txt            # TTS 安全文本（理化才有）
├── export/
│   └── 剪映草稿/         # 剪映专业版工程
├── qc.json               # 质检报告
├── report.json           # 工序报告
└── cache.json            # 缓存索引
```
