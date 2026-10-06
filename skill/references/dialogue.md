# Dialogue（对白规范）

## 核心原则
1. **口型一体生成**：说话镜头必须由视频模型根据中文台词直接生成（嘴型+声音一次成型）。这是用户用"苏巧卧室口播"确立的质量基准。
2. **一人一声**：同一角色全片只用一个声音。原生成功的用原生；工具拒绝（`integrity_check_failed`）的对白才允许 MiniMax 画外音 + 反应镜头。**禁止同一角色混用原生声和配音**——宁可剪掉非关键台词。
3. **无旁白**：不用旁白推进剧情。苍老之声（长生门之声）是画外音角色，保留。
4. **被拒不重试**：被工具拒绝的生成请求不得重复，包括换镜头描述、改台词、换工具。这是硬约束。

## 说话镜头提示词模板
```
<场景>，<人物描述>，<真实五官比例，非网红脸，不磨皮>，
用<语气/状态>说出以下中文台词，嘴型必须与台词逐字同步，
声音与画面同步，画面无任何文字：
「<台词逐字>」
国风仙侠电影质感，写实偏水墨，低饱和青灰色调。
```
- 台词用「」标出，**逐字照抄**，不改写、不"优化"。
- 单句控制在 ~12 秒内；长对白拆成多句。
- 状态词示例：虚弱但温和、少年人恶作剧般的吆喝、压抑的决心。

## 验台词（faster-whisper）
每条必验：中文转写 + 词级时间戳，确认台词一字不差。
```bash
~/workspace/.venvs/whisper/bin/python -c "
import numpy as np, subprocess
from faster_whisper import WhisperModel
m = WhisperModel('/home/hatch/workspace/.venvs/whisper-models/small', device='cpu', compute_type='int8')
raw = subprocess.run(['ffmpeg','-v','error','-i','<clip>.mp4','-ac','1','-ar','16000','-f','f32le','-'],capture_output=True).stdout
audio = np.frombuffer(raw, dtype=np.float32)
segs, _ = m.transcribe(audio, language='zh', beam_size=5, word_timestamps=True)
for s in segs: print(f'[{s.start:.1f}-{s.end:.1f}]: {s.text}')
"
```
- 念错/含糊/多字少字 → 重新生成该条（换一条**新**请求，不是被拒请求的重试）。
- 按转写出的起止时间精确裁剪说话段（前后各留 ~0.2s 气口）。

## 场景空镜提示词模板
```
<场景描述>。国风仙侠电影质感，写实偏水墨，低饱和青灰色调。
同期声：<脚步声/风声/溪水声/衣物摩擦声/鸟鸣虫鸣……>。
无人物，无对白，画面无任何文字。
```
- 同期声是原生音频的唯一来源，描述要具体。
- 有人物无对白时加：`真实五官比例，非网红脸，不磨皮`。

## 对白处理优先级
1. 视频模型原生说话镜头（首选）
2. 工具拒绝 → MiniMax 画外音 + 反应镜头（备选，需如实告知用户）
3. 非关键台词 → 直接剪掉（保"一人一声"）
4. 关键剧情台词被拒 → MiniMax 画外音保留（最后手段）
